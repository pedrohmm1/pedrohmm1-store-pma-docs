# Declaração de uso de IA

A [página do projeto](https://insper.github.io/platform/2026.2/exercises/project/) permite o uso de
IA, limitado a tarefas específicas, e exige que ele seja declarado: o que foi feito com IA e o
que foi feito pelo integrante. Esta declaração cobre a parte de Pedro Machado. Os outros
integrantes declaram o uso deles nas próprias documentações.

**Ferramenta:** Claude Code (Anthropic), como assistente no terminal e no VS Code.

## O que foi feito com ajuda de IA

| Parte | Uso da IA |
|---|---|
| Compose integrado, `build.ps1`/`build.sh`, README do root | rascunho a partir do compose do Gateway e dos hands-on |
| Manifestos do Postgres e `k8s.yaml` dos serviços | adaptação a partir do repositório do professor (`repo-classes/pma262`) |
| Templates CloudFormation (`iam.yaml`, `eks.yaml`, `rds.yaml`, Jenkins) e `up.ps1`/`down.ps1` | rascunho dos templates e dos scripts; diagnóstico do bug do `down.ps1` |
| Estágio "Deploy to EKS" dos Jenkinsfiles e `readinessProbe` | rascunho do código e dos comandos de medição |
| Teste de carga (`locustfile.py`, `hpa-gateway.yaml`) | rascunho, execução dos comandos e organização dos números |
| Smoke tests e limpeza dos dados de teste | execução dos comandos `curl`/`psql` |
| Esta documentação | rascunho do texto das páginas a partir dos READMEs e das evidências |

## O que foi feito por Pedro Machado

- implementação dos serviços Gateway, Auth, Account e Product, seguindo os hands-on;
- as decisões: layout do root, um cluster EKS para o grupo, banco no RDS, ambiente descartável,
  menor privilégio para o Jenkins, HPA e `readinessProbe` como bottlenecks;
- a correção do cookie `Secure` no Auth para funcionar em HTTP no EKS;
- toda a operação na AWS e no Jenkins (login SSO, credenciais, criação dos jobs, builds),
  conferindo cada passo no console;
- revisão de todo o código, dos commits e do texto antes de publicar;
- coordenação com o grupo (contrato do Exchange com o Enzo, passagem para os colegas).

A disciplina exige que cada integrante saiba explicar e defender tudo o que entregou. Cada
arquivo gerado com ajuda de IA foi explicado e revisado por Pedro Machado antes do commit.
