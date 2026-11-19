# Apresentação

A [página de apresentação](https://insper.github.io/platform/2026.2/exercises/project/presentation/)
pede **um vídeo por grupo**, de 1 a 2 minutos, apresentando o projeto e os bottlenecks.

## Vídeo do grupo (1 min 58 s)

<video controls preload="metadata" width="100%">
  <source src="../video/store-pma.mp4" type="video/mp4">
  Seu navegador não toca vídeo: <a href="../video/store-pma.mp4">baixe o arquivo</a>.
</video>

Montado e narrado pelo **Pedro Garcia**. As cenas de terminal e navegador são gravações reais do
projeto: a loja pelo gateway (imagens `pedrohmm1/*` do Docker Hub), os `k8s.yaml` com o HPA
escalando, o Grafana do Order e o cache e a varredura do Exchange.

## Onde a parte de Pedro Machado aparece no vídeo

| Tempo | Parte | O que é de Pedro Machado |
|---|---|---|
| 0:13 | Arquitetura | gateway como porta única, Auth/Account/Product, EKS + RDS na AWS, Jenkins |
| 0:35 | Demonstração | 401 sem login e 403 para `USER` (filtro do gateway + regras de papel do Product) |
| 0:53 | Kubernetes e teste de carga | health-check e HPA do gateway de 1 para 4 réplicas (`hpa-gateway.yaml`) |
| 1:05 | Bottlenecks | HPA do gateway e `readinessProbe` |
| 1:39 | Custos e disponibilidade | ambiente descartável com `up.ps1`/`down.ps1` |

O vídeo do teste de carga com HPA está em [Teste de carga e HPA](projeto/load-test.md#video-do-teste-com-hpa).
