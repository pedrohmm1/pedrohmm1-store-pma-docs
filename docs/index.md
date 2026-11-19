# store-pma · documentação individual

???+ info inline end "Edição"

    Platforms, Microservices, DevOps and APIs
    Insper · 2026.2

## Aluno e grupo

**Pedro Machado** ([pedrohmm1](https://github.com/pedrohmm1)), responsável por **Gateway**,
**Auth**, **Account** e **Product**, pelo repositório root (submódulos e compose), pelo
Kubernetes no Minikube, pela infraestrutura na **AWS (EKS + RDS)**, pelo **Jenkins (CI/CD com
deploy no EKS)** e pelo **teste de carga com HPA**.

**Grupo 05** (organização [EP-PC](https://github.com/EP-PC) no GitHub, conta AWS `pmapi-bcc-grupo05`):

| Integrante | GitHub | O que fez no projeto |
|---|---|---|
| **Pedro Machado** | [pedrohmm1](https://github.com/pedrohmm1) | Gateway, Auth, Account e Product; root com submódulos e compose; Minikube; EKS na AWS (`up.ps1`/`down.ps1`); Jenkins e CI/CD; teste de carga com HPA |
| Enzo Perego | [EnzoPerego](https://github.com/EnzoPerego) | Order (contrato + service), Redis do Order, Prometheus e Grafana |
| Pedro Garcia | [garciapp2](https://github.com/garciapp2) | Exchange (Python, PTAX do BCB), cache e varredura de vulnerabilidades, custos/PaaS/SLA, vídeo de apresentação |

## Entregas do projeto

Tabela de notas da [página do projeto](https://insper.github.io/platform/2026.2/exercises/project/):

| Item | Peso | Status | Quem | Onde |
|---|---|---|---|---|
| Gateway | 5% | :material-check: | **Pedro Machado** | [Serviços implementados](projeto/servicos.md#gateway) |
| Auth | 5% | :material-check: | **Pedro Machado** | [Serviços implementados](projeto/servicos.md#auth) |
| Account | 5% | :material-check: | **Pedro Machado** | [Serviços implementados](projeto/servicos.md#account) |
| Exchange | 5% | :material-check: | Pedro Garcia | [Arquitetura](projeto/arquitetura.md) |
| Bottlenecks (≥ 2 por integrante) | 20% | :material-check: | todos | [Bottlenecks](projeto/bottlenecks.md) |
| AWS | 5% | :material-check: | **Pedro Machado** | [AWS e EKS](projeto/aws-eks.md) |
| EKS | 10% | :material-check: | **Pedro Machado** | [AWS e EKS](projeto/aws-eks.md) |
| Jenkins (CI/CD) | 10% | :material-check: | **Pedro Machado** | [CI/CD](projeto/cicd.md) |
| Load Testing | 15% | :material-check: | **Pedro Machado** | [Teste de carga](projeto/load-test.md) |
| PaaS, Custos e SLA | 10% | :material-check: | Pedro Garcia | [Custos](projeto/custos.md) |
| MkDocs | 10% | :material-check: | cada um | esta documentação |

Além da tabela: **Product** (Pedro Machado) e **Order** (Enzo), o **vídeo de apresentação** do grupo
([Apresentação](apresentacao.md)) e a **declaração de uso de IA** ([Uso de IA](ia.md)).

## Linha do tempo da parte de Pedro Machado

| Data | Entrega |
|---|---|
| até 08/11/2026 | Product, Account, Auth e Gateway (contratos + implementações, testes locais) |
| 09/11/2026 | root `store-pma`: 6 submódulos, compose integrado, scripts de build, smoke test |
| 10–11/11/2026 | Kubernetes no Minikube: Postgres + 4 serviços, smoke test pelo gateway |
| 11/11/2026 | Jenkins: 6 jobs, push multi-arch no Docker Hub; EKS + RDS por CloudFormation |
| 12/11/2026 | deploy no EKS pelo Jenkins, `readinessProbe`, teste de carga com HPA |
| 13/11/2026 | scripts de subir/derrubar validados, teste local final |
| 18/11/2026 | gateway: health-check do Order/Exchange abertos (contrato combinado), integração do Order conferida |

## Repositórios

| Repositório | O que tem |
|---|---|
| [EP-PC/store-pma](https://github.com/EP-PC/store-pma) | root: submódulos, compose, `aws/` (CloudFormation + scripts), `jenkins/`, `load-test/` |
| [EP-PC/Gateway](https://github.com/EP-PC/Gateway) | gateway (Spring Cloud Gateway) |
| [EP-PC/Auth-service](https://github.com/EP-PC/Auth-service) | autenticação (JWT) |
| [EP-PC/Account](https://github.com/EP-PC/Account) · [EP-PC/Account-service](https://github.com/EP-PC/Account-service) | contrato e implementação do Account |
| [EP-PC/Product](https://github.com/EP-PC/Product) · [EP-PC/Product-service](https://github.com/EP-PC/Product-service) | contrato e implementação do Product |
| [EP-PC/Order](https://github.com/EP-PC/Order) · [EP-PC/Order-service](https://github.com/EP-PC/Order-service) | contrato e implementação do Order (Enzo) |
| [EP-PC/Exchange-service](https://github.com/EP-PC/Exchange-service) | Exchange em Python (Pedro Garcia) |
| [pedrohmm1/pedrohmm1-store-pma-docs](https://github.com/pedrohmm1/pedrohmm1-store-pma-docs) | esta documentação |

Imagens públicas no Docker Hub: `pedrohmm1/account`, `pedrohmm1/auth`, `pedrohmm1/product` e
`pedrohmm1/gateway` (multi-arch `amd64` + `arm64`, publicadas pelo Jenkins).
