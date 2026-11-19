# AWS e EKS

Páginas da disciplina: [AWS](https://insper.github.io/platform/2026.2/exercises/project/aws/) e
[EKS](https://insper.github.io/platform/2026.2/exercises/project/eks/). Conta do grupo
`pmapi-bcc-grupo05`, região **us-east-2 (Ohio)**, acesso pelo portal SSO.

Toda a infraestrutura é **código** (CloudFormation em `store-pma/aws/`) e sobe ou desce com
**um comando**, por qualquer integrante com acesso SSO. Assim o grupo não depende de mim nem de
cliques no console, e nada fica ligado gastando dinheiro sem querer.

## Stacks

| Stack | Template | O que cria |
|---|---|---|
| `eks-store-vpc` | `amazon-eks-vpc-private-subnets.yaml` (do professor, sem alteração) | VPC, 2 subnets públicas + 2 privadas, 2 NAT Gateways |
| `eks-store-iam` | `iam.yaml` | role do cluster (`AmazonEKSClusterPolicy`) e role dos nós (worker, CNI, ECR read-only) |
| `eks-store-cluster` | `eks.yaml` | EKS 1.37 + node group (2 × `t3.medium`, mín. 1, máx. 4, subnets privadas) + add-on **metrics-server** |
| `eks-store-rds` | `rds.yaml` | RDS PostgreSQL 17, `db.t4g.micro`, 20 GB gp3, privado, sem backup, `DeletionPolicy: Delete` |
| `eks-store-jenkins-iam` | `jenkins-iam.yaml` | usuário IAM `store-jenkins` (permanente; só `eks:DescribeCluster`) |
| `eks-store-jenkins-access` | `jenkins-access.yaml` | access entry do Jenkins no cluster: `AmazonEKSEditPolicy` **só no namespace `default`** |

```yaml title="rds.yaml (trecho)"
DBSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Postgres do store - so a partir do cluster EKS
    VpcId: !Ref VpcId
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 5432
        ToPort: 5432
        SourceSecurityGroupId: !Ref ClusterSecurityGroupId   # (1)!
```

1. O banco não aceita conexão de nenhum IP: só de quem está no security group do cluster.
   Junto com `PubliclyAccessible: false`, o RDS é inalcançável de fora da VPC.

## `up.ps1`: subir do zero

1. Stacks na ordem **VPC → IAM → cluster → RDS** (`aws cloudformation deploy`, com
   `--no-fail-on-empty-changeset`: rodar de novo com tudo igual não é erro, o script é
   **idempotente**).
2. Gera a senha do RDS na primeira vez (em `aws/.env`, ignorado pelo Git).
3. `aws eks update-kubeconfig --alias eks-store`.
4. Cria no cluster a `postgres-configmap` (host do RDS), a `postgres-secrets` e os Secrets
   `account-secrets`/`auth-secrets` a partir do `api/.env` (`kubectl create ... --dry-run=client
   -o yaml | kubectl apply -f -`: nada de segredo em arquivo).
5. Aplica os `k8s/k8s.yaml` dos serviços (**os mesmos do Minikube**), espera os rollouts e
   imprime a URL pública do gateway.

Os manifestos não mudaram entre Minikube e EKS: só a `postgres-configmap`/`postgres-secrets`
apontam para o RDS em vez do Postgres do cluster.

## `down.ps1`: derrubar

Apaga primeiro o **Service `gateway`** (para a AWS remover o Load Balancer, que não pertence a
nenhuma stack) e depois as stacks na ordem inversa. Nunca apaga o usuário IAM do Jenkins.

!!! failure "Bug que eu encontrei e corrigi"

    Na primeira versão, a função que testava se uma stack existia tratava **qualquer erro** como
    "não existe". Com o token SSO expirado, o script "pulou" as stacks de IAM e VPC e terminou
    dizendo que estava tudo apagado. Agora só a mensagem `does not exist` conta como inexistente;
    qualquer outro erro para o script.

    ```powershell
    function StackExists($stack) {
        $out = aws cloudformation describe-stacks @aws --stack-name $stack 2>&1
        if ($LASTEXITCODE -eq 0) { return $true }
        if ("$out" -match 'does not exist') { return $false }
        throw "erro ao consultar a stack ${stack}: $out"
    }
    ```

!!! warning "Lição de custo"

    Em 12/11 o `down.ps1` parou no meio (a internet caiu e depois o login SSO expirou) e o
    cluster, os nós e os NAT Gateways ficaram ligados a noite toda. Desde então, depois de todo
    `down.ps1` eu confiro o inventário da região (stacks, EKS, NAT, EIP, ELB, RDS, volumes) antes
    de considerar encerrado.

## Evidências (11/11/2026)

```text
eks-store      ACTIVE  1.37
eks-store-nodes  ACTIVE  t3.medium  2
eks-store-rds-db-...  postgres  17.9  db.t4g.micro  PubliclyAccessible=False  available

NAME                 TYPE           EXTERNAL-IP                                      PORT(S)
service/account      ClusterIP      <none>                                           8080/TCP
service/auth         ClusterIP      <none>                                           8080/TCP
service/gateway      LoadBalancer   a8403ba7...us-east-2.elb.amazonaws.com           8080:30152/TCP
service/product      ClusterIP      <none>                                           8080/TCP

Flyway: Successfully applied 3 migrations to schema "accounts" (PostgreSQL 17.9, RDS)
Flyway: Successfully applied 2 migrations to schema "products"
```

**Smoke test pela URL pública, com cookie real:** `GET /products` 200 · register 201 · login
USER 200 (`HttpOnly; SameSite=Lax`) · `/auth/me` USER · `POST /products` USER **403** · login
ADMIN · `POST` **201** · `DELETE` **204** · `/auth/solve` 401/404 · logout → `/auth/me` **401**.
Os dados de teste foram apagados do RDS com um pod temporário `postgres:17` (`psql ...
sslmode=require`), já que o banco não é acessível de fora.
