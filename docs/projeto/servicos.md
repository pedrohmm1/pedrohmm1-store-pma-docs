# Meus serviços

Implementei **Gateway**, **Auth**, **Account** e **Product**, seguindo os hands-on 1 (CRUD),
2 (Gateway) e 3 (Security) e o exercício [Product API](https://insper.github.io/platform/2026.2/exercises/product/).
Todos em Java 25 + Spring Boot 4.1.1, com o padrão **contrato + implementação** do hands-on:
o contrato (interface Feign + DTOs) é um jar separado que os outros serviços usam.

| Serviço | Repositório | Contrato | Chama |
|---|---|---|---|
| Gateway | [EP-PC/Gateway](https://github.com/EP-PC/Gateway) | — | auth (`/auth/solve`) e roteia o resto |
| Auth | [EP-PC/Auth-service](https://github.com/EP-PC/Auth-service) | usa `store:account` | account (Feign) |
| Account | [EP-PC/Account-service](https://github.com/EP-PC/Account-service) | [`store:account:1.0.0`](https://github.com/EP-PC/Account) | — (schema `accounts`) |
| Product | [EP-PC/Product-service](https://github.com/EP-PC/Product-service) | [`store:product:1.0.0`](https://github.com/EP-PC/Product) | — (schema `products`) |

Todos os endpoints abaixo são chamados **pelo gateway** (`http://<host>:8080`).

## Gateway

Spring Cloud Gateway (WebFlux). É o **único** serviço com porta publicada (compose) e o único
Service `LoadBalancer` (Kubernetes). Faz três coisas:

1. **Roteia** `/auth/**`, `/accounts/**`, `/products/**`, `/orders/**` e `/exchange/**` para
   `http://<serviço>:8080`, sem cortar o prefixo.
2. **Apaga** os headers `id-account` e `id-role` que vierem do cliente (`default-filters`), para
   ninguém forjar a identidade.
3. **Autentica** toda rota que não está na lista de rotas abertas: lê o cookie, chama
   `POST /auth/solve` e injeta `id-account`/`id-role` com o que o Auth devolveu.

```java title="RouterValidator.java: rotas abertas"
private static final List<String> OPEN_API_ENDPOINTS = List.of(
    "POST /auth/register",
    "POST /auth/login",
    "GET /auth/logout",
    "GET /auth/health-check",
    "GET /accounts/health-check",
    "GET /orders/health-check",
    "GET /exchange/health-check",
    "GET /products/**"          // vitrine pública (inclui /products/health-check)
);
```

```java title="AuthorizationFilter.java (trecho)"
final HttpCookie cookie = request.getCookies().getFirst(AUTH_COOKIE_TOKEN);
if (cookie == null || cookie.getValue().isBlank()) {
    return Mono.error(new ResponseStatusException(HttpStatus.UNAUTHORIZED, "Missing token"));
}
return solve(cookie.getValue().trim())                       // POST http://auth:8080/auth/solve
    .flatMap(solved -> chain.filter(authorize(exchange, solved)));  // injeta id-account / id-role
```

Decisões que eu tomei (e sei defender):

| Decisão | Por quê |
|---|---|
| Auth com erro 4xx → **401**; Auth fora do ar ou 5xx → **503** | token ruim é culpa do cliente; Auth caído não é |
| `WebClient` criado uma vez (o hands-on criava um por requisição) | menos alocação por requisição |
| Corrigi o `matches` do hands-on (o `replaceAll` apagava o `-`) | `/auth/health-check` nunca casava |
| `/auth/solve` bloqueado de fora (rota com `SetStatus=404`) | endpoint é só para o gateway |
| Health-check do Order e do Exchange abertos (18/11) | o site exige `GET /<serviço>/health-check` sem login |

## Auth

Não tem banco: cria e valida contas chamando o Account via Feign (`http://account:8080`).

| Endpoint | Entrada | Resposta |
|---|---|---|
| `POST /auth/register` | `{name, email, password}` | 201 `{id, name, email, role}`; 400/409 |
| `POST /auth/login` | `{email, password}` | 200 + cookie `__store_jwt_token`; 401 |
| `GET /auth/logout` | — | 200 + cookie expirado (`Max-Age=0`) |
| `GET /auth/me` | header `id-account` | 200 `{id, name, email, role}`; 401 |
| `POST /auth/solve` | `{token}` | 200 `{idAccount, role}`; 401 (uso interno do gateway) |
| `GET /auth/health-check` | — | 200 |

```java title="JwtService.java: o token"
// nunca colocar senha nem hash no token: o payload e so Base64, qualquer um le
return Jwts.builder()
    .id(account.id())                       // jti = id da conta
    .subject(account.name())
    .claim(CLAIM_EMAIL, account.email())
    .claim(CLAIM_ROLE, account.role())      // USER ou ADMIN
    .issuer(ISSUER)
    .notBefore(now)
    .expiration(new Date(now.getTime() + duration))   // 1 h (JWT_DURATION)
    .signWith(key)                          // chave de 32 bytes -> HS256
    .compact();
```

- O token vai **só no cookie `HttpOnly`** (o JavaScript do navegador não lê), nunca no corpo.
- A chave vem de `JWT_SECRET_KEY` (`.env` / Secret do Kubernetes). Sem ela, o serviço não sobe.
- **Problema real no EKS:** o cookie tinha `Secure; SameSite=None` fixo, e o navegador não guarda
  cookie `Secure` em HTTP (o Load Balancer do projeto é HTTP). Criei `JWT_COOKIE_SECURE`
  (padrão `true`; `false` → sem `Secure` e `SameSite=Lax`) e usei `"false"` no `k8s.yaml`.

## Account

| Endpoint | Pelo gateway (`id-account`) | Interno (Auth, sem `id-account`) |
|---|---|---|
| `POST /accounts`, `POST /accounts/login` | 403 | permitido |
| `GET /accounts` | só `ADMIN` | permitido |
| `GET /accounts/{id}`, `DELETE /accounts/{id}` | dono ou `ADMIN` | permitido |
| `GET /accounts/health-check` | permitido | permitido |

- Senha guardada como **SHA-256 em Base64** (hands-on 1); a resposta nunca devolve o hash.
- O primeiro **ADMIN** vem de uma migration do Flyway que lê `ACCOUNT_ADMIN_EMAIL` e
  `ACCOUNT_ADMIN_HASH` (variáveis de ambiente, fora do Git). Todo cadastro novo é `USER`.

```java title="AccountAuthorization.java (trecho)"
// sem id-account: veio da rede interna (Auth), nao do Gateway
public boolean isInternal() { return idAccount() == null; }

// ler uma conta: interna libera; externa so o dono ou ADMIN
public void requireOwnerOrAdmin(String id) {
    if (isInternal() || isAdmin()) return;
    if (!idAccount().equals(id)) throw forbidden();
}
```

A regra "sem `id-account` = interno" só é segura porque o gateway **sempre** injeta o header nas
rotas protegidas e apaga o que vem de fora; o Account não tem porta publicada.

## Product

| Endpoint | Resposta | Quem pode |
|---|---|---|
| `GET /products`, `GET /products/{id}` | 200 (404 se não existe) | todos, sem login |
| `GET /products?name=` (busca, fora do contrato) | 200 | todos |
| `POST /products` `{name, price, unit}` | 201 + `Location` | só `ADMIN` (403 para `USER`) |
| `DELETE /products/{id}` | 204 | só `ADMIN` |
| `GET /products/health-check` | 200 | todos |

- O contrato `store:product:1.0.0` (repo `EP-PC/Product`) tem o `@FeignClient` que o **Order**
  usa para validar cada item e pegar o preço. Por isso o contrato ficou **congelado**: a regra de
  papel é lida no service (`ProductAuthorization`), não na interface.
- `Location` usa o host público (o gateway repassa o `Host` original), não `http://product:8080`.

```java title="ProductAuthorization.java"
// criar/apagar produto: so ADMIN; USER ou sem header -> 403
public void requireAdmin() {
    if (!ROLE_ADMIN.equals(header(HEADER_ID_ROLE))) {
        throw new ResponseStatusException(HttpStatus.FORBIDDEN, "Access denied");
    }
}
```

## Smoke test (o mesmo nos três ambientes)

Rodei esta sequência no compose, no Minikube e no EKS (pela URL pública), sempre com sucesso:

| Passo | Esperado |
|---|---|
| `GET /products` sem cookie | 200 |
| `POST /auth/register` → `POST /auth/login` | 201 → 200 + cookie `HttpOnly` |
| `GET /auth/me` | 200, `role: USER` |
| `POST /products` como USER | **403** |
| login do admin → `POST /products` → `DELETE /products/{id}` | 200 → **201** → **204** |
| `POST /auth/solve` de fora | 401 sem cookie / 404 com cookie |
| `GET /auth/logout` → `GET /auth/me` | 200 → **401** |

Os outros serviços (Order, do Enzo, e Exchange, do Pedro Garcia) estão nas documentações deles;
a integração do Order com o Product e com o meu gateway foi conferida em 18/11/2026 (pedido
criado, preço congelado no item, 404 para pedido de outro usuário, 422 para moeda inválida).
