# ☁️ Projeto 04 — Infraestrutura como Código com AWS CloudFormation

![AWS](https://img.shields.io/badge/AWS-CloudFormation-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![S3](https://img.shields.io/badge/AWS-S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![RDS](https://img.shields.io/badge/AWS-RDS-527FFF?style=for-the-badge&logo=amazonaws&logoColor=white)
![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Provisionar 100% de forma automatizada uma infraestrutura em nuvem segura, escalável e otimizada para custos (Free Tier) utilizando AWS CloudFormation, sustentando um ambiente completo de Cloud com separação de camadas de rede, computação, dados e observabilidade.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| AWS CloudFormation | Provisionamento automatizado de toda a infraestrutura via IaC (YAML) |
| Amazon S3 | Hospedagem do portfólio serverless em HTML/CSS (camada global) |
| Amazon VPC | Rede isolada com Internet Gateway e tabelas de roteamento customizadas |
| Amazon EC2 | Servidor de processamento Linux na subnet pública |
| AWS Systems Manager | Busca dinâmica da AMI mais recente via Parameter Store |
| Amazon RDS MySQL | Banco de dados relacional isolado em subnets privadas |
| AWS CloudWatch | Alarmes de monitoramento de utilização de CPU |

---

## 🏗️ Arquitetura da solução

```
Internet
    │
    │  HTTP
    ▼
┌──────────────────────────────────────────────────────────┐
│                        VPC                               │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │                  Public Subnet                     │  │
│  │                                                    │  │
│  │  ┌──────────────────────────────────────────────┐  │  │
│  │  │  EC2 — Amazon Linux 2 (t2.micro)             │  │  │
│  │  │  AMI via SSM Parameter Store (dinâmica)      │  │  │
│  │  │  Web Server Security Group                   │  │  │
│  │  └──────────────────┬───────────────────────────┘  │  │
│  └─────────────────────┼──────────────────────────────┘  │
│                        │ MySQL (porta 3306)               │
│  ┌─────────────────────┼──────────────────────────────┐  │
│  │               Private Subnets                      │  │
│  │                     │                              │  │
│  │  ┌──────────────────▼───────────────────────────┐  │  │
│  │  │  RDS MySQL — Single-AZ (Free Tier)           │  │  │
│  │  │  Acesso restrito ao Security Group do EC2    │  │  │
│  │  └──────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
         │                              │
         ▼                              ▼
┌─────────────────┐          ┌─────────────────────┐
│   Amazon S3     │          │   AWS CloudWatch     │
│  Portfólio      │          │  Alarmes de CPU      │
│  HTML/CSS       │          │  Observabilidade     │
│  (Serverless)   │          └─────────────────────┘
└─────────────────┘
```

---

## 📋 Etapas de implementação

1. Criação do template **CloudFormation** em YAML com todos os recursos parametrizados
2. Upload do template via console AWS → CloudFormation → Create Stack
3. Revisão preventiva com **Change Sets** antes do deploy
4. Provisionamento automático de VPC, subnets, Internet Gateway e tabelas de roteamento
5. Lançamento da instância **EC2** com AMI buscada dinamicamente via **SSM Parameter Store**
6. Criação do banco **RDS MySQL** isolado nas subnets privadas com acesso restrito por Security Group
7. Configuração dos **alarmes CloudWatch** para monitoramento de CPU
8. Upload do portfólio (`index.html` e `perfil.jpg`) para o bucket **S3** via AWS CLI
9. Validação de todos os recursos após status `CREATE_COMPLETE`

---

## 💡 Diferenciais técnicos aplicados

| Prática | Descrição |
|---|---|
| Segurança em camadas | O RDS não possui IP público — aceita conexões apenas do Security Group do EC2 |
| Previsibilidade com Change Sets | Evolução da arquitetura validada preventivamente antes de cada deploy |
| FinOps / Free Tier | Recursos parametrizados dentro dos limites do Free Tier (`t2.micro`, discos de 8–20GB, Single-AZ) |
| AMI dinâmica | A AMI do Amazon Linux 2 é buscada automaticamente via SSM Parameter Store, sem hardcode |

---

## 🚀 Como fazer o deploy

```bash
# 1. Clone o repositório
git clone https://github.com/seu-usuario/seu-repositorio.git

# 2. Acesse o console AWS → CloudFormation → Create Stack
#    Faça upload do arquivo template-infra.yaml
#    Preencha os parâmetros: Nome do KeyPair e Nome único do Bucket

# 3. Após o status CREATE_COMPLETE, envie os arquivos para o S3
aws s3 cp index.html s3://nome-do-seu-bucket/
aws s3 cp perfil.jpg s3://nome-do-seu-bucket/
```

---

## 📸 Evidências

CloudFormation - YAML:
<img width="1060" height="844" alt="template-cloudformation-yaml" src="https://github.com/user-attachments/assets/36bc154f-6e96-4da4-aa52-c888b4e332c4" />

CloudFormation - Console:
<img width="1550" height="777" alt="cloudformation-criado" src="https://github.com/user-attachments/assets/a0c246f9-a1dd-4c1e-ab5e-0598d8e8201e" />

Métrica do CloudWatch:
<img width="1541" height="797" alt="metric-cloudwatch" src="https://github.com/user-attachments/assets/86d83704-ea27-4601-b5e4-7e87e36cc799" />

Portfólio online pelo S3:
<img width="1909" height="1025" alt="siteestatico-online" src="https://github.com/user-attachments/assets/74eeab4c-f39d-40fb-9a08-e91e0ae58678" />


---

## 💡 Aprendizados

- ✅ Provisionamento completo de infraestrutura com **CloudFormation** (IaC em YAML)
- ✅ Uso de **Change Sets** para validar mudanças antes de aplicar em produção
- ✅ Busca dinâmica de AMI via **SSM Parameter Store**, eliminando hardcode no template
- ✅ Isolamento de banco de dados em **subnets privadas** com acesso controlado por Security Groups
- ✅ Hospedagem de site estático serverless no **Amazon S3**
- ✅ Monitoramento de infraestrutura com alarmes do **CloudWatch**
- ✅ Boas práticas de **FinOps** para otimização de custos dentro do Free Tier

---

## 🔗 Referências

- [Documentação AWS CloudFormation](https://docs.aws.amazon.com/cloudformation/)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [SSM Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [Amazon RDS](https://docs.aws.amazon.com/rds/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
