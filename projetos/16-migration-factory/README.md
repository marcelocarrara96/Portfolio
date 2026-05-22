# 🏭 Projeto 16 — AWS Cloud Migration Factory: Deploy, Exploração e Descomissionamento

![AWS](https://img.shields.io/badge/AWS-CloudFormation-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![Cognito](https://img.shields.io/badge/AWS-Cognito-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)
![DynamoDB](https://img.shields.io/badge/AWS-DynamoDB-527FFF?style=for-the-badge&logo=amazondynamodb&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## ⚠️ Problema

Migrações em escala para a nuvem são complexas: envolvem dezenas de servidores, múltiplas ondas de migração, diferentes estratégias (Rehost, Replatform) e times que precisam de visibilidade centralizada do progresso. Gerenciar esse processo manualmente gera erros, retrabalho e custos imprevistos.

O desafio era: como orquestrar uma migração corporativa completa de forma automatizada, rastreável e com governança integrada desde o deploy até o descomissionamento da infraestrutura de laboratório?

---

## 🎯 Objetivo

Implantar, explorar e descomissionar com segurança a solução **AWS Cloud Migration Factory** — uma arquitetura serverless enterprise que automatiza pipelines de migração em escala, integrando CloudFormation, Lambda, DynamoDB, Cognito, API Gateway e CloudShell para gerenciar o ciclo de vida completo de uma migração AWS.

---

## 🏗️ Solução

A solução foi implantada via CloudFormation com 130+ recursos em Nested Stacks, acessada via interface web protegida por Cognito, explorada em profundidade via painel administrativo e descomissionada com processo de FinOps aplicado — garantindo custo zero ao final do laboratório.

```
                        Usuários
                            │
                            ▼
                  ┌──────────────────────┐
                  │      CloudFront      │
                  │   Interface Web      │
                  │  Migration Factory   │
                  │      v5.0.1          │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │   Amazon Cognito     │
                  │   Autenticação       │
                  │   (bypass via CLI)   │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │    API Gateway       │
                  └──────────┬───────────┘
                             │
           ┌─────────────────┼─────────────────┐
           ▼                 ▼                 ▼
     ┌──────────┐      ┌──────────┐      ┌──────────┐
     │  Lambda  │      │  Lambda  │      │  Lambda  │
     │ Pipeline │      │  Tasks   │      │  Scripts │
     └────┬─────┘      └────┬─────┘      └────┬─────┘
          │                 │                  │
          ▼                 ▼                  ▼
     ┌────────────────────────────────────────────┐
     │                 DynamoDB                   │
     │   Schema flexível: waves, apps, servers    │
     │   app_id, wave_ids, multivalue-relationship│
     └───────────────────┬────────────────────────┘
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        ┌──────────┐         ┌──────────────┐
        │   SQS    │         │  CloudWatch  │
        │  Queue   │         │     Logs     │
        └────┬─────┘         └──────────────┘
             │
             ▼
        ┌──────────┐
        │   SNS    │
        │  E-mail  │
        └──────────┘
```

### Etapas de implementação

1. Deploy da stack `CloudMigrationFactory-MarceloCarrara` via CloudFormation com parâmetros customizados
2. Aplicação de **tags de governança** na stack (Project, Owner, Environment)
3. Bypass de autenticação do **Cognito** via AWS CloudShell (`admin-set-user-password`)
4. Acesso ao **dashboard operacional** do Migration Factory (v5.0.1)
5. Exploração do **schema NoSQL** do DynamoDB via aba Attributes do painel admin
6. Cadastro de Wave e Application via formulários da interface web
7. Identificação e documentação de **bug de renderização** na v5.0.1
8. **Descomissionamento seguro**: esvaziamento manual dos buckets S3 + delete da stack
9. Validação de remoção completa dos 130+ recursos sem recursos órfãos

---

## 🛠️ Ferramentas

| Serviço | Função |
|---|---|
| AWS CloudFormation | Provisionamento de 130+ recursos via Nested Stacks |
| Amazon API Gateway | Interface entre frontend e Lambdas de orquestração |
| AWS Lambda | Orquestração serverless dos pipelines de migração |
| Amazon DynamoDB | Schema NoSQL flexível para inventário de migração |
| Amazon Cognito | Autenticação da interface web (bypass via CloudShell) |
| Amazon CloudFront | Distribuição da interface web da solução |
| Amazon SQS | Fila de mensagens para processamento assíncrono |
| Amazon SNS | Notificações por e-mail sobre eventos de migração |
| Amazon CloudWatch | Logs e monitoramento dos Lambdas e API Gateway |
| Amazon S3 | Armazenamento de scripts e logs de migração |
| AWS CloudShell | Execução de comandos CLI diretamente no console AWS |
| AWS IAM | Dezenas de roles geradas automaticamente pela stack |

---

## ⚙️ Detalhes técnicos

### Parâmetros de deploy configurados

| Parâmetro | Valor | Justificativa |
|---|---|---|
| WPM (Wave Planning Manager) | `true` | Gerenciamento visual das ondas de migração |
| Deploy Bedrock Guardrail | `false` | Controle de custos — foco na infra base |
| Replatform EC2 | `true` | Suporte a estratégias de replataforma automatizada |
| Deployment Type | `Default (Public)` | Acesso via internet protegido por Cognito |
| Additional Identity Provider | `false` | Simplicidade para ambiente de laboratório |

### Tags de governança aplicadas

```
Project:     CloudMigrationFactory
Owner:       MarceloCarrara
Environment: Test
```

### Bypass de autenticação Cognito via CloudShell

```bash
aws cognito-idp admin-set-user-password \
  --user-pool-id <UserPoolId> \
  --username <username> \
  --password <NewPassword> \
  --permanent
```

### Descomissionamento seguro — processo FinOps

```
1. Identificar todos os buckets S3 vinculados à stack
        │
        ▼
2. Esvaziar manualmente cada bucket (Empty)
   → Remove objetos, logs e scripts de migração
        │
        ▼
3. Executar Delete Stack na stack raiz do CloudFormation
        │
        ▼
4. CloudFormation remove 130+ recursos de forma encadeada
   → IAM Roles, Lambdas, DynamoDB, Cognito, API Gateway
        │
        ▼
5. Validar: zero recursos órfãos, custo zero pós-lab ✅
```

---

## ✅ Resultado

Stack com 130+ recursos provisionada, interface web acessível e autenticada via Cognito, schema NoSQL do DynamoDB explorado em profundidade, bug de renderização identificado e documentado, e descomissionamento completo executado com processo FinOps — sem recursos órfãos e sem custos residuais.

---

## 📸 Evidências do Projeto

Para garantir a transparência técnica e a rastreabilidade de cada etapa descrita, todas as capturas de tela e logs gerados durante o ciclo de vida deste laboratório foram centralizados.

👉 [Clique aqui para acessar a pasta com todas as evidências visuais do projeto](./evidencias)

### Mapeamento dos Arquivos na Pasta:
- **`01-parametros-deploy.png`**: Parâmetros iniciais de deploy e chaves do CloudFormation.
- **`02-tags-governanca.png`**: Aplicação de tags de propriedade (`Owner: MarceloCarrara`).
- **`03-stack-complete.png`**: Infraestrutura e *Nested Stacks* em status `CREATE_COMPLETE`.
- **`04-cognito-bypass.png`**: Execução do comando de redefinição de senha via AWS CLI no CloudShell.
- **`05-dashboard-factory.png`**: Dashboard principal do AWS Cloud Migration Factory (v5.0.1) operando.
- **`06-nosql-attributes.png`**: Análise e mapeamento do esquema dinâmico e tabelas de atributos no DynamoDB.
- **`07-s3-finops-clean.png`**: Comprovação de esvaziamento manual dos buckets antes do decommissioning.

---

## 💡 Aprendizados

**Nested Stacks organizam o que um template único não consegue**
Com 130+ recursos, o CloudFormation organiza a stack em componentes isolados — rede, segurança, computação e banco de dados — cada um com seu próprio ciclo de vida. Isso permite atualizar partes da arquitetura sem recriar tudo.

**Cognito pode ser gerenciado programaticamente quando o front falha**
A interface web não permitia login por ausência de senha permanente no Cognito. A solução foi usar o CloudShell para forçar a senha via CLI — mostrando que operações administrativas de identidade podem (e devem) ser feitas de forma programática em produção.

**DynamoDB sem schema rígido é uma vantagem em migração**
O schema flexível do DynamoDB permite que administradores adicionem atributos dinâmicos ao inventário de servidores sem migrations de banco. Isso é essencial em cenários de migração onde cada cliente tem metadados diferentes.

**Descomissionamento é parte do projeto, não o fim**
O encerramento anterior da stack deixou buckets S3 com arquivos residuais, gerando custos. O processo correto — esvaziar buckets antes do delete — garantiu remoção completa de todos os recursos. Em ambientes corporativos, isso é a diferença entre um lab limpo e uma fatura surpresa.

**Documentar bugs é tão profissional quanto documentar sucessos**
O travamento de renderização na v5.0.1 foi identificado, isolado e documentado como limitação de interface — não como falha do operador. Saber distinguir erro de configuração de bug de produto é uma habilidade real de suporte e operações.

---

## 🔗 Referências

- [AWS Cloud Migration Factory — Implementation Guide](https://docs.aws.amazon.com/solutions/latest/cloud-migration-factory-on-aws/solution-overview.html)
- [AWS Application Migration Service (MGN)](https://docs.aws.amazon.com/mgn/)
- [Amazon Cognito — Admin Set User Password](https://docs.aws.amazon.com/cognito-user-identity-pools/latest/APIReference/API_AdminSetUserPassword.html)
- [CloudFormation Nested Stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
