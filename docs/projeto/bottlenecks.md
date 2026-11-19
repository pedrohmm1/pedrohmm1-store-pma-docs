# Bottlenecks

A [página de apresentação](https://insper.github.io/platform/2026.2/exercises/project/presentation/)
pede **pelo menos 2 bottlenecks por integrante**. Os meus dois saíram de problemas que eu
**medi** no EKS, não de uma lista: cada um tem o antes, a correção e o depois.

| Integrante | Bottleneck 1 | Bottleneck 2 |
|---|---|---|
| **Pedro Machado** (eu) | **Escala / load balancing**: HPA do gateway | **Disponibilidade no deploy**: `readinessProbe` |
| Enzo Perego (Order) | Caching: Redis para preços e cotações | Observability: Prometheus + Grafana |
| Pedro Garcia (Exchange) | Caching: cotações com fallback | Vulnerability scanning: pip-audit + Trivy |

## 1. Gateway saturando a CPU → HPA

**Problema.** No teste de carga com 800 usuários, o gateway bateu o limite de CPU do pod
(493m de 500m). O Kubernetes passou a estrangular a CPU (*throttling*) e as requisições fizeram
fila: o p95 ficou **3,5×** maior e o p99 **4×** maior que com 300 usuários. Não houve erro,
mas a experiência piorou para todo mundo, porque **toda** requisição passa pelo gateway.

**Por que o gateway.** Ele recebe 100% do tráfego e, em toda rota autenticada, ainda faz uma
chamada HTTP ao Auth (`/auth/solve`). Os outros serviços estavam com folga (product ~310m,
account ~300m, auth ~110m).

**Solução.** `HorizontalPodAutoscaler` pela CPU média (alvo 50% do request), de 1 a 6 réplicas,
com o **metrics-server** instalado como add-on do EKS no CloudFormation. O gateway é *stateless*
(não guarda sessão: a identidade vem do JWT a cada requisição), então qualquer réplica atende
qualquer requisição e o Service distribui entre elas.

**Resultado** (800 usuários, mesmo cenário):

| | Sem HPA | Com HPA | Diferença |
|---|---|---|---|
| Réplicas do gateway | 1 | 1 → 2 → 4 em ~40 s | |
| Vazão | 566 req/s | 597 req/s | +5% |
| p95 | 770 ms | 320 ms | **−58%** |
| p99 | 1.400 ms | 910 ms | **−35%** |
| Falhas | 0 | 0 | |

O p50 quase não mudou (~170 ms) porque é dominado pela rede Brasil → Ohio. Detalhes e vídeo em
[Teste de carga](load-test.md).

**Limite que sobra:** sem Cluster Autoscaler, o HPA só cresce até onde cabem pods nos 2 nós.

## 2. Deploy derrubando a API → `readinessProbe`

**Problema.** Quando coloquei o deploy automático no Jenkins, medi com uma requisição por
segundo pelo Load Balancer durante o rollout: **~40 s de HTTP 503 a cada deploy**. Sem probe, o
Kubernetes considera o pod novo pronto assim que o container inicia e já tira o antigo, mas o
Spring Boot leva ~40 s para subir. Com deploy contínuo, isso é indisponibilidade a cada push.

**Solução.** `readinessProbe` nos 4 serviços. O pod novo só recebe tráfego (e o antigo só é
desligado) depois que a probe passa:

```yaml title="k8s.yaml (gateway, account, auth, product)"
readinessProbe:
  tcpSocket:
    port: 8080             # (1)!
  initialDelaySeconds: 20  # o Spring não sobe em menos que isso
  periodSeconds: 5
  failureThreshold: 6      # até +30 s de tolerância antes de marcar como falho
```

1. A porta 8080 só abre no **fim** da inicialização do Spring, então o teste TCP é suficiente e
   não precisou mudar código nem adicionar dependência.

**Resultado** (1 requisição/s durante o rollout, deploy pelo Jenkins):

| Serviço | Sem probe | Com probe |
|---|---|---|
| account | ~40 s de 503 | 100 de 100 respondidas |
| gateway | — | 90 de 90 respondidas |
| auth | — | 99 de 100 (1 cortada) |
| product | — | 89 de 90 (1 cortada) |

A requisição que ainda falha é a que estava **em andamento** quando o pod antigo foi desligado.
A próxima melhoria seria `preStop` com uma pausa curta + *graceful shutdown* do Spring, para o
pod antigo terminar o que está atendendo antes de sair.

!!! note "Evolução: probe no health-check"

    O site passou a pedir que as probes chamem o `GET /<serviço>/health-check`. O Order e o
    Exchange já usam `httpGet` nesse endpoint. Nos meus quatro serviços o endpoint existe e
    responde 200 (`/accounts/health-check`, `/auth/health-check`, `/products/health-check` e
    `/health-check` no gateway); trocar `tcpSocket` por `httpGet` é uma mudança de 3 linhas por
    manifesto, com os mesmos tempos.

## Dos outros integrantes

Detalhados nas documentações de cada um:

- **Enzo (Order):** Redis com TTL de 60 s para preços do Product e cotações do Exchange
  (`GET /orders/{id}?currency=BRL` p50 de 335 ms → 10 ms) e Prometheus + Grafana no Order.
- **Pedro Garcia (Exchange):** cache das cotações do PTAX com *single flight* e fallback (p50
  −98%) e varredura de vulnerabilidades no pipeline (pip-audit + Trivy). Documentação:
  [garciapp2.github.io/store-pma-docs](https://garciapp2.github.io/store-pma-docs/projeto/bottlenecks/).
