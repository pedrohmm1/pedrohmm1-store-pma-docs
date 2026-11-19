# Atividades individuais

A página de apresentação pede a documentação "of the exercises done and also of the project".
As atividades da seção *Exercises → Individual* do site e os hands-on, e onde cada uma está:

## Exercícios individuais

| Atividade | Situação | Onde |
|---|---|---|
| [Product API](https://insper.github.io/platform/2026.2/exercises/product/) | **feita por mim** | [Meus serviços](projeto/servicos.md#product) · [EP-PC/Product](https://github.com/EP-PC/Product) · [EP-PC/Product-service](https://github.com/EP-PC/Product-service) |
| [Jenkins](https://insper.github.io/platform/2026.2/exercises/jenkins/) | **feita por mim**: 6 jobs, push multi-arch e deploy no EKS | [CI/CD](projeto/cicd.md) · `Jenkinsfile` de cada service |
| [MiniKube](https://insper.github.io/platform/2026.2/exercises/minikube/) | **feita por mim**: Postgres + 4 serviços | [Como rodar](projeto/como-rodar.md#minikube) · `k8s/k8s.yaml` de cada service · `store-pma/api/postgres/` |
| [Bottlenecks](https://insper.github.io/platform/2026.2/exercises/bottlenecks/) | **2 feitos por mim** | [Bottlenecks](projeto/bottlenecks.md) |
| [Order API](https://insper.github.io/platform/2026.2/exercises/order/) | feita no projeto pelo Enzo | [EP-PC/Order-service](https://github.com/EP-PC/Order-service) |
| [Exchange API](https://insper.github.io/platform/2026.2/exercises/exchange/) | feita no projeto pelo Pedro Garcia | [EP-PC/Exchange-service](https://github.com/EP-PC/Exchange-service) |

## Hands-on aplicados no projeto

| Hands-on | O que virou no projeto |
|---|---|
| 1. CRUD (controller, service, containerization, repository) | **Account** (contrato + service, Flyway, hash SHA-256) e **Product** no mesmo padrão; Dockerfile de cada um |
| 2. Gateway | **Gateway** com as rotas de todos os serviços |
| 3. Security (JWT, open routes, authorization) | **Auth** (JWT HS256 no cookie `HttpOnly`), `RouterValidator` (rotas abertas) e `AuthorizationFilter`; regras de papel no Account e no Product |
| 4. DevOps | Jenkins do projeto (`store-pma/jenkins/`) e Jenkinsfiles |
| 5. Orchestration | manifestos Kubernetes, Minikube e depois EKS |

## O que eu fiz, em resumo

- **4 microsserviços** (Gateway, Auth, Account, Product) e os 2 contratos (`store:account`,
  `store:product`), com as regras de autorização por papel.
- **Root `store-pma`**: submódulos, compose integrado, scripts de build, README.
- **Kubernetes**: Minikube e depois EKS, com os mesmos manifestos.
- **AWS**: VPC, IAM, EKS e RDS em CloudFormation; scripts `up.ps1`/`down.ps1`; usuário IAM de
  menor privilégio para o Jenkins.
- **CI/CD**: build, push multi-arch e deploy no EKS pelo Jenkins.
- **Teste de carga** com Locust e **HPA**; 2 bottlenecks medidos antes e depois.
- **Integração do grupo**: documento de passagem para os colegas (padrões, contrato do Exchange,
  como subir cada ambiente) e ajustes no gateway para o Order e o Exchange.
