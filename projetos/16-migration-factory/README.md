# 🏭 Projeto 16 - AWS Cloud Migration Factory

![AWS](https://img.shields.io/badge/AWS-CloudFormation-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![Cognito](https://img.shields.io/badge/AWS-Cognito-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)
![DynamoDB](https://img.shields.io/badge/AWS-DynamoDB-527FFF?style=for-the-badge&logo=amazondynamodb&logoColor=white)
![Status](https://img.shields.io/badge/Status-Em_Andamento-yellow?style=for-the-badge)

---

## 🎯 Objetivo

Implantar e explorar a solução **AWS Cloud Migration Factory** - uma arquitetura serverless que automatiza e orquestra pipelines de migração em escala para a AWS, utilizando CloudFormation para provisionamento completo da infraestrutura e integrando serviços como API Gateway, Lambda, DynamoDB, Cognito, SNS e CloudWatch.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| AWS CloudFormation | Provisionamento de toda a stack (130+ recursos) via template único |
| Amazon API Gateway | Interface de comunicação entre o frontend e os Lambdas |
| AWS Lambda | Orquestração serverless dos pipelines de migração |
| Amazon DynamoDB | Armazenamento dos dados de migração e configurações |
| Amazon Cognito | Autenticação e controle de acesso à interface web |
| Amazon CloudFront | Distribuição da interface web da solução |
| Amazon SQS | Fila de mensagens para processamento assíncrono |
| Amazon SNS | Notificações por e-mail sobre eventos de migração |
| Amazon CloudWatch | Logs e monitoramento dos Lambdas e API Gateway |
| AWS IAM | Roles e permissões geradas automaticamente pela stack |

---

## 🏗️ Arquitetura da solução

```
                        Usuários
                            │
                            ▼
                  ┌──────────────────┐
                  │   CloudFront     │
                  │  Interface Web   │
                  │  Migration Fct.  │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │   Amazon Cognito │
                  │   Autenticação   │
                  └────────┬─────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │   API Gateway    │
                  └────────┬─────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │  Lambda  │ │  Lambda  │ │  Lambda  │
        │ Pipeline │ │  Tasks   │ │  Scripts │
        └────┬─────┘ └────┬─────┘ └────┬─────┘
             │             │            │
             ▼             ▼            ▼
        ┌──────────────────────────────────┐
        │          DynamoDB                │
        │  (pipelines, tasks, servers)     │
        └──────────────────────────────────┘
             │                      │
             ▼                      ▼
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

---

## 📋 Etapas realizadas

### 1. Deploy da Stack

- Stack criada via CloudFormation: `CloudMigrationFactory-MarceloCarrara`
- Status final: **CREATE_COMPLETE**
- **130 recursos** provisionados, incluindo API Gateway, Lambda, DynamoDB, Cognito, SQS, SNS e CloudWatch

---

### 2. Abordagens de deploy testadas

| Abordagem | Descrição | Resultado |
|---|---|---|
| Role customizada | Criação manual da role `CFN-MigrationFactory-Role` com permissões específicas (CloudFront, API Gateway, SQS etc.) | Requer ajuste fino de permissões |
| Role automática | CloudFormation gera as permissões automaticamente sem role prévia | Mais simples, deploy bem-sucedido |

> **Conclusão:** Demonstrei entendimento das duas formas de trabalhar - com role customizada (mais seguro e controlado) e com role automática (mais simples e rápido para laboratório).

> **Observação:** O Migration Factory cria dezenas de roles IAM automaticamente, o que é esperado e removido no delete da stack.

---

### 3. Outputs e Endpoint

- O endpoint da aplicação foi disponibilizado via **CloudFront**
- Interface web acessível com tela de login do Migration Factory

---

### 4. Exploração dos recursos

- Interface web acessível via CloudFront ✅
- Login não concluído por ausência de credenciais configuradas no Cognito
- Recursos explorados via console: CloudFormation, API Gateway, DynamoDB, IAM Roles

---

### 5. Testes planejados *(próxima etapa)*

| Teste | Descrição | Status |
|---|---|---|
| Rehost | Instalar agente MGN em EC2 e simular lift-and-shift | ⬜ Planejado |
| Replatform | Recriar EC2 via CloudFormation com ajustes | ⬜ Planejado |
| SNS | Validar notificações por e-mail nos eventos de migração | ⬜ Planejado |
| CloudWatch | Validar logs de Lambda e API Gateway | ⬜ Planejado |

---

### 6. Limitações desta versão

- Login no Cognito não concluído → pipeline de migração não executado
- Rehost e Replatform não realizados nesta etapa
- Documentação focada no deploy da stack e exploração dos recursos provisionados

> **Observação:** Alguns recursos não foram removidos automaticamente após o delete da stack e precisaram ser excluídos manualmente.

---

### 7. Próximos passos

- Configurar usuário no Cognito com senha permanente
- Validar criação de pipeline de migração pela interface web
- Executar testes de Rehost e Replatform
- Documentar logs no CloudWatch e notificações via SNS

---

## 📸 Evidências

Diagrama da arquitetura:
<img width="1536" height="1024" alt="diagrama-resumido" src="https://github.com/user-attachments/assets/0d2a8cad-4006-4448-93b1-d1b86eb06970" />

Tags no CloudFormation:
<img width="1294" height="398" alt="tags-cloudformation" src="https://github.com/user-attachments/assets/e2efac3a-0f18-4cc9-bfda-85e884a621f6" />

Nova IAM Role criada:
<img width="1908" height="802" alt="newrole-policy" src="https://github.com/user-attachments/assets/f302efd2-3332-489e-a489-d0e460bf740c" />

CloudFormation create in progress:
<img width="1910" height="886" alt="cloudformation-create-in-progress" src="https://github.com/user-attachments/assets/f0cfe984-a7d5-4611-82d8-e070d1f88a53" />

CloudFormation create complete:
<img width="1901" height="645" alt="cloudformation-complete" src="https://github.com/user-attachments/assets/009db316-ca9a-43a1-b93c-89ed1c623135" />

Endpoint online e funcionando nessa URL:
<img width="1797" height="546" alt="url-online-api-endpoint" src="https://github.com/user-attachments/assets/01abdc7a-ed5f-4675-b5c8-edb6cff92320" />

Não deletou tudo então tive que ir manualmente nos serviços:
<img width="1439" height="665" alt="delete-skipped-failed" src="https://github.com/user-attachments/assets/227ac909-4b8e-483f-9132-21d86c1006d8" />

---

## 💡 Aprendizados

- ✅ Como implantar uma **solução AWS completa** via CloudFormation com 130+ recursos
- ✅ Diferença entre deploy com **role customizada** (controle granular) e **role automática** (simplicidade)
- ✅ Arquitetura **serverless orientada a eventos** com API Gateway + Lambda + DynamoDB
- ✅ Uso do **Amazon Cognito** como camada de autenticação em aplicações web
- ✅ Como o **Migration Factory** orquestra pipelines de migração em escala (Rehost e Replatform)
- ✅ Gestão de recursos residuais após delete de stack no CloudFormation

---

## 🔗 Referências

- [AWS Cloud Migration Factory — Implementation Guide](https://docs.aws.amazon.com/solutions/latest/cloud-migration-factory-on-aws/solution-overview.html)
- [AWS Application Migration Service (MGN)](https://docs.aws.amazon.com/mgn/)
- [Amazon Cognito](https://docs.aws.amazon.com/cognito/)
- [AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
