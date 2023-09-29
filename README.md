<h1 align="center">
    Módulo Terraform AWS Grupo de Segurança 
</h1>

<p align="center" style="font-size: 1.2rem;"> 
    Esse módulo terraform cria os recursos de Grupo de Segurança and Regras de Grupo de Segurança em várias combinações.
     </p>

<p align="center">


## Exemplos

Alguns exemplos de como usar este módulo para criar recursos que usam grupo de segurança:

### Básico
  ```hcl
    module "security_group" {
      source          = "git@github.com/terraform-modules//security-group/"
      version         = "1.0.0"
      name            = "microserivce-app"
      vpc_id          = module.vpc.vpc_id
      tags.cicle       = "vendas"
      tags.owner       = "squad-vendas"
      tags.environment = "dev"

      ## INGRESS Rules
      new_sg_ingress_rules_with_cidr_blocks = [{
        rule_count  = 1
        from_port   = 8080
        protocol    = "tcp"
        to_port     = 8080
        cidr_blocks = [module.vpc.vpc_cidr_block, "172.16.0.0/16"]
        description = "Allow incoming traffic."
        },
        {
          rule_count  = 2
          from_port   = 6379
          protocol    = "tcp"
          to_port     = 6379
          cidr_blocks = ["172.16.0.0/16"]
          description = "Allow Redis traffic."
        }
      ]

      ## EGRESS Rules
      new_sg_egress_rules_with_cidr_blocks = [{
        rule_count  = 1
        from_port   = 8080
        protocol    = "tcp"
        to_port     = 8080
        cidr_blocks = [module.vpc.vpc_cidr_block, "172.16.0.0/16"]
        description = "Allow outbound traffic."
        },
        {
          rule_count  = 2
          from_port   = 6379
          protocol    = "tcp"
          to_port     = 6879
          cidr_blocks = ["172.16.0.0/16"]
          description = "Allow Redis outbound traffic."
        }]
    }
  ```

  ### ONLY RULES
  ```hcl
    module "security_group_rules" {
      source        = "git@github.com/terraform-modules//security-group/"
      version       = "1.0.0"
      name           = "microservice-app"
      vpc_id         = "vpc-xxxxxxxxx"
      new_sg         = false
      existing_sg_id = "sg-xxxxxxxxx"

      ## INGRESS Rules
      existing_sg_ingress_rules_with_cidr_blocks = [{
        rule_count  = 1
        from_port   = 8080
        protocol    = "tcp"
        to_port     = 8080
        cidr_blocks = ["10.9.0.0/16"]
        description = "Allow incoming traffic."
        },
        {
          rule_count  = 2
          from_port   = 6379
          protocol    = "tcp"
          to_port     = 6379
          cidr_blocks = ["10.9.0.0/16"]
          description = "Allow Redis traffic."
        }
      ]

      existing_sg_ingress_rules_with_self = [{
        rule_count  = 1
        from_port   = 8080
        protocol    = "tcp"
        to_port     = 8080
        description = "Allow incoming traffic."
        },
        {
          rule_count  = 2
          from_port   = 6379
          protocol    = "tcp"
          to_port     = 6379
          description = "Allow Redis traffic."
        }
      ]

      existing_sg_ingress_rules_with_source_sg_id = [{
        rule_count               = 1
        from_port                = 9300
        protocol                 = "tcp"
        to_port                  = 9300
        source_security_group_id = "sg-xxxxxxxxx"
        description              = "Allow ERP traffic."
        },
        {
          rule_count               = 2
          from_port                = 1433
          protocol                 = "tcp"
          to_port                  = 1433
          source_security_group_id = "sg-xxxxxxxxx"
          description              = "Allow SQL traffic."
        }]

      ## EGRESS Rules
      existing_sg_egress_rules_with_cidr_blocks = [{
        rule_count  = 1
        from_port   = 9300
        protocol    = "tcp"
        to_port     = 9300
        cidr_blocks = ["10.9.0.0/16"]
        description = "Allow ERP outbound traffic."
        },
        {
          rule_count  = 2
          from_port   = 1433
          protocol    = "tcp"
          to_port     = 1433
          cidr_blocks = ["10.9.0.0/16"]
          description = "Allow SQL outbound traffic."
      }]

      existing_sg_egress_rules_with_self = [{
        rule_count  = 1
        from_port   = 9300
        protocol    = "tcp"
        to_port     = 9300
        description = "Allow ERP outbound traffic."
        },
        {
          rule_count  = 2
          from_port   = 1433
          protocol    = "tcp"
          to_port     = 1433
          description = "Allow SQL outbound traffic."
      }]

      existing_sg_egress_rules_with_source_sg_id = [{
        rule_count               = 1
        from_port                = 8081
        protocol                 = "tcp"
        to_port                  = 8081
        source_security_group_id = "sg-xxxxxxxxx"
        description              = "Allow outbound traffic."
        },
        {
          rule_count               = 2
          from_port                = 5432
          protocol                 = "tcp"
          to_port                  = 5432
          source_security_group_id = "sg-xxxxxxxxx"
          description              = "Allow postgres/aurora outbound traffic."
        }]
    }
  ```

  ### PREFIX LIST
  ```hcl
    module "security_group" {
      source              = "git@github.com/terraform-modules//security-group/"
      version             = "1.0.0"
      name                = "microservice-app"
      vpc_id              = module.vpc.vpc_id
      prefix_list_enabled = true
      entry = [{
        cidr = "10.19.0.0/16"
      }]

      ## INGRESS Rules
      new_sg_ingress_rules_with_prefix_list = [{
        rule_count  = 1
        from_port   = 8080
        protocol    = "tcp"
        to_port     = 8080
        description = "Allow incoming traffic."
        }
      ]
      ## EGRESS Rules
      new_sg_egress_rules_with_prefix_list = [{
        rule_count  = 1
        from_port   = 3306
        protocol    = "tcp"
        to_port     = 3306
        description = "Allow mysql/aurora outbound traffic."
        }
      ]
    }
  ```


## Entradas (Inputs)

| Nome | Descrição | Tipo | Padrão | Obrigatório |
|------|-------------|------|---------|:--------:|
| enable | Flag para controlar a criação do módulo.. | `bool` | `true` | no |
| entry | Pode ser especificado várias vezes para cada entrada da lista de prefixos. | `list(any)` | `[]` | no |
| existing\_sg\_egress\_rules\_with\_cidr\_blocks | Regras de entrada apenas com bloco de cidr. Deve ser usado quando existe um grupo de segurança. | `any` | `{}` | no |
| existing\_sg\_egress\_rules\_with\_prefix\_list | Regras de saída apenas com IDs de lista de prefixos. Deve ser usado quando existe um grupo de segurança. | `any` | `{}` | no |
| existing\_sg\_egress\_rules\_with\_self | Regras de saída governadas no mesmo SG . Deve ser usado quando existe um grupo de segurança. | `any` | `{}` | no |
| existing\_sg\_egress\_rules\_with\_source\_sg\_id | Regras de saída apenas com o ID do grupo de segurança de origem. Deve ser usado quando existe um grupo de segurança. | `any` | `{}` | no |
| existing\_sg\_id | Forneça o ID do grupo de segurança existente para atualizar a regra existente. | `string` | `null` | no |
| existing\_sg\_ingress\_rules\_with\_cidr\_blocks | Regras de entrada apenas com blocos de cidr. Deve ser usado quando existe um grupo de segurança. | `any` | `{}` | no |
| existing\_sg\_ingress\_rules\_with\_prefix\_list | Regras de entrada apenas com prefix\_list. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| existing\_sg\_ingress\_rules\_with\_self | Regras de entrada apenas com o ID do grupo de segurança de origem. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| existing\_sg\_ingress\_rules\_with\_source\_sg\_id | Regras de entrada apenas com IDs de lista de prefixos. Deve ser usado quando existe um grupo de segurança. | `any` | `{}` | no |
| label\_order | Label order, e.g. `name`,`application`. | `list(any)` | <pre>[<br>  "name",<br>  "environment"<br>]</pre> | no |
| tags | cicle, owner e environment; exemplo 'departamento'. | `string` | `"financeiro"` | no |
| max\_entries | O número máximo de entradas que esta lista de prefixos pode conter. | `number` | `5` | no |
| name | Nome  (e.g. `app` or `service`). | `string` | `""` | no |
| new\_sg | Flag para controlar a criação de um novo grupo de segurança. | `bool` | `true` | no |
| new\_sg\_egress\_rules\_with\_cidr\_blocks | Regras de saída apenas com cidr\_blockd. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| new\_sg\_egress\_rules\_with\_prefix\_list | Regras de saída apenas com IDs de lista de prefixos. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| new\_sg\_egress\_rules\_with\_self | Regra de saída governada apenas por ela mesma. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| new\_sg\_egress\_rules\_with\_source\_sg\_id | Regras de saída apenas com o ID do grupo de segurança de origem. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| new\_sg\_ingress\_rules\_with\_cidr\_blocks | Regras de entrada apenas com blocos cidr. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| new\_sg\_ingress\_rules\_with\_prefix\_list | Regras de entrada apenas com IDs de lista de prefixos. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| new\_sg\_ingress\_rules\_with\_self | Regras de entrada governada apenas por ela mesma. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| new\_sg\_ingress\_rules\_with\_source\_sg\_id | Regras de entrada apenas com o ID do grupo de segurança de origem. Deve ser usado quando um novo grupo de segurança for implantado. | `any` | `{}` | no |
| prefix\_list\_address\_family | (Obrigatório para novo recurso) A família de endereços (IPv4 ou IPv6) da lista de prefixos. | `string` | `"IPv4"` | no |
| prefix\_list\_enabled | Enable prefix\_list. | `bool` | `false` | no |
| prefix\_list\_ids | O ID da lista de prefixos. | `list(string)` | `[]` | no |
| sg\_description | Descrição do grupo de segurança. O padrão é Gerenciado pelo Terraform. Não pode ser uma string vazia. NOTA: Este campo é mapeado para o atributo AWS GroupDescription, para o qual não há API de atualização. Se quiser classificar seus grupos de segurança de uma forma que possa ser atualizada, use tags. | `string` | `null` | no |
| vpc\_id | O ID da VPC ao qual o grupo de segurança da instância pertence. | `string` | `""` | no |

## Saídas (Outputs)

| Nome | Descrição |
|------|-------------|
| prefix\_list\_id | O ID da lista de prefixos. |
| security\_group\_arn | IDs nos grupos de segurança da AWS associados à instância. |
| security\_group\_id | IDs nos grupos de segurança da AWS associados à instância. |
| security\_group\_tags | Um mapeamento de tags públicas a serem atribuídas ao recurso. |


