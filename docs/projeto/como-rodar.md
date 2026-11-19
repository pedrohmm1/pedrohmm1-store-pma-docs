# Como rodar

Tudo a partir do root [EP-PC/store-pma](https://github.com/EP-PC/store-pma):

```bash
git clone --recurse-submodules https://github.com/EP-PC/store-pma.git
cd store-pma/api
cp .env.example .env      # preencher JWT_SECRET_KEY, ACCOUNT_ADMIN_EMAIL, ACCOUNT_ADMIN_HASH
```

## Local (Docker Compose)

```bash
./build.sh                       # Windows: .\build.ps1  (precisa do JDK 25 e do Maven)
docker compose up --build -d     # db, account, auth, product, order, order-redis, gateway
curl http://localhost:8080/products
docker compose down              # SEM -v: o volume store_db-data guarda contas e produtos
```

Os Dockerfiles copiam `target/*.jar`, por isso o build vem antes: `mvn install` nos contratos
(`product`, `account`, `order`) e `mvn package` nos services. O script não roda `clean` porque o
OneDrive trava a pasta `target/` no Windows.

```yaml title="api/compose.yaml (trecho)"
name: store

services:
  db:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

  auth:
    build: ./auth-service
    environment:
      JWT_SECRET_KEY: ${JWT_SECRET_KEY:?defina JWT_SECRET_KEY no .env}   # (1)!
    depends_on:
      - account

  gateway:
    build: ./gateway-service
    ports:
      - 8080:8080                                                       # (2)!
```

1. `:?` faz o compose parar com uma mensagem clara se a variável faltar, em vez de subir um
   serviço quebrado.
2. Só o gateway publica porta. Os outros serviços só são alcançáveis pela rede interna do
   compose, o que garante que ninguém de fora chame o Account sem passar pelo gateway.

!!! warning "Memória"

    São 5 JVMs + Postgres + Redis. Num notebook de 16 GB o Docker travou depois de dias com o
    compose de pé e pouca RAM livre; feche programas pesados antes do `up`.

## Minikube

Hands-on 5 / exercício [MiniKube](https://insper.github.io/platform/2026.2/exercises/minikube/).
O banco fica em `api/postgres/` (como no repositório do professor) e cada serviço tem **um**
`k8s/k8s.yaml` (Deployment + Service).

```bash
minikube start --driver=docker --memory=3072 --cpus=4

kubectl apply -f api/postgres/configmap.yaml
kubectl apply -f api/postgres/secrets.yaml        # credencial de desenvolvimento (store/store)
kubectl apply -f api/postgres/deployment.yaml
kubectl apply -f api/postgres/service.yaml

# segredos reais, criados na hora (nunca em arquivo versionado)
kubectl create secret generic account-secrets \
  --from-literal=ACCOUNT_ADMIN_EMAIL="..." --from-literal=ACCOUNT_ADMIN_HASH="..."
kubectl create secret generic auth-secrets --from-literal=JWT_SECRET_KEY="..."

kubectl apply -f api/account-service/k8s/k8s.yaml
kubectl apply -f api/auth-service/k8s/k8s.yaml
kubectl apply -f api/product-service/k8s/k8s.yaml
kubectl apply -f api/gateway-service/k8s/k8s.yaml

minikube tunnel        # em outro terminal: o Service LoadBalancer do gateway vira 127.0.0.1:8080
```

O mesmo smoke test do compose passou pelo gateway no Minikube em 11/11/2026.

!!! note "Reinícios logo depois do `minikube start`"

    Account e Product às vezes reiniciam algumas vezes com `UnknownHostException: postgres`: o
    CoreDNS ainda está subindo. O Kubernetes recupera sozinho; não é erro dos manifestos.

## AWS (EKS)

```powershell
aws sso login --profile grupo05
powershell -NoProfile -ExecutionPolicy Bypass -File .\aws\up.ps1     # ~30 min
# ... no fim imprime http://<load-balancer>:8080
powershell -NoProfile -ExecutionPolicy Bypass -File .\aws\down.ps1   # ~20 min
```

Detalhes, decisões e evidências em [AWS e EKS](aws-eks.md).
