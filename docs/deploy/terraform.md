# Terraform

Gerenciamento de infraestrutura como codigo (IaC) do TepConfina.

## Estrutura de Modulos

O projeto utiliza 11 modulos Terraform organizados por responsabilidade:

```
infra/terraform/
  modules/
    vpc/            # VPC, Subnets, NAT Gateway, Internet Gateway
    alb/            # Application Load Balancer, Target Groups
    ecs/            # ECS Cluster, Task Definition, Service
    ecr/            # Elastic Container Registry
    rds/            # RDS PostgreSQL
    elasticache/    # ElastiCache Redis
    cloudwatch/     # Alarms, Dashboards, Log Groups
    monitoring/     # SNS Topics, Subscription
    backup/         # AWS Backup Plans
    security/       # Security Groups, IAM Roles
    networking/     # Route53, ACM Certificates
  environments/
    dev.tfvars      # Variaveis para desenvolvimento
    staging.tfvars  # Variaveis para staging
    prod.tfvars     # Variaveis para producao
  main.tf           # Orquestracao dos modulos
  variables.tf      # Declaracao de variaveis
  outputs.tf        # Outputs do Terraform
  backend.tf        # Configuracao do state remoto
```

## Modulos

| Modulo          | Recursos Gerenciados                              |
|-----------------|---------------------------------------------------|
| `vpc`           | VPC, Subnets, NAT Gateway, Internet Gateway, Route Tables |
| `alb`           | Load Balancer, Listeners, Target Groups           |
| `ecs`           | Cluster, Task Definition, Service, Auto-scaling   |
| `ecr`           | Repositorio de imagens Docker                     |
| `rds`           | Instancia PostgreSQL, Subnet Group, Parameter Group |
| `elasticache`   | Cluster Redis, Subnet Group, Parameter Group      |
| `cloudwatch`    | Log Groups, Alarms, Dashboards                    |
| `monitoring`    | SNS Topics, Email Subscriptions                   |
| `backup`        | Backup Plans, Backup Vault                        |
| `security`      | Security Groups, IAM Roles, IAM Policies          |
| `networking`    | Route53 Records, ACM Certificates                 |

## State Management

O estado do Terraform e armazenado remotamente no S3 com lock via DynamoDB:

```hcl
terraform {
  backend "s3" {
    bucket         = "tepconfina-terraform-state"
    key            = "terraform.tfstate"
    region         = "sa-east-1"
    dynamodb_table = "tepconfina-terraform-locks"
    encrypt        = true
  }
}
```

!!! warning "State remoto obrigatorio"
    Nunca armazene o state localmente em ambientes compartilhados. O state contem informacoes sensiveis e deve ser centralizado.

## Multi-Ambiente

Cada ambiente possui seu arquivo `.tfvars`:

```hcl
# environments/staging.tfvars
environment        = "staging"
ecs_cpu            = 512
ecs_memory         = 1024
ecs_desired_count  = 1
rds_instance_class = "db.t3.micro"
redis_node_type    = "cache.t3.micro"
rds_multi_az       = false
```

```hcl
# environments/prod.tfvars
environment        = "production"
ecs_cpu            = 1024
ecs_memory         = 2048
ecs_desired_count  = 2
rds_instance_class = "db.r6g.large"
redis_node_type    = "cache.r6g.large"
rds_multi_az       = true
```

## Comandos

Utilize o script auxiliar para executar comandos Terraform:

```bash
./scripts/terraform.sh <ambiente> <comando>
```

| Comando  | Descricao                                         |
|----------|----------------------------------------------------|
| `init`   | Inicializa o Terraform e configura o backend       |
| `plan`   | Mostra as alteracoes que serao aplicadas           |
| `apply`  | Aplica as alteracoes na infraestrutura             |
| `destroy`| Remove toda a infraestrutura (use com cautela)     |

### Exemplos

```bash
# Inicializar Terraform para staging
./scripts/terraform.sh staging init

# Ver o plano de alteracoes para producao
./scripts/terraform.sh prod plan

# Aplicar alteracoes em staging
./scripts/terraform.sh staging apply
```

!!! danger "Cuidado com destroy"
    O comando `destroy` remove **todos** os recursos. Nunca execute em producao sem aprovacao da equipe.

## Workspace-based Isolation

Cada ambiente utiliza um workspace separado do Terraform, garantindo isolamento completo do state:

```bash
# Listar workspaces
terraform workspace list

# Selecionar workspace
terraform workspace select staging
```

## Boas Praticas

!!! tip "Sempre faca plan antes de apply"
    Revise cuidadosamente o output do `terraform plan` antes de aplicar. Verifique especialmente recursos que serao destruidos e recriados.

- Execute `plan` antes de cada `apply`
- Utilize variaves para valores que mudam entre ambientes
- Mantenha modulos pequenos e focados
- Documente outputs relevantes para outros modulos
- Faca code review em alteracoes de infraestrutura
