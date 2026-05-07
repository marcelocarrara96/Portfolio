# 💸 Projeto 14 — Otimização de Recursos e Estimativa de Custos com AWS Pricing Calculator

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![RDS](https://img.shields.io/badge/AWS-RDS-527FFF?style=for-the-badge&logo=amazonaws&logoColor=white)
![EBS](https://img.shields.io/badge/AWS-EBS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![CLI](https://img.shields.io/badge/AWS-CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Otimizar uma instância EC2 para redução de custos removendo um banco de dados local descomissionado e redimensionando o tipo de instância via AWS CLI, e utilizar o AWS Pricing Calculator para estimar e comparar os custos mensais antes e depois da otimização.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon EC2 | Instância da aplicação Café otimizada de t3.small para t3.micro |
| Amazon EBS | Volume reduzido de 40 GB para 20 GB após remoção do banco local |
| Amazon RDS | Banco de dados MariaDB gerenciado (db.t3.micro) mantido após migração |
| AWS CLI | Automação do stop, resize e start da instância via terminal |
| AWS Pricing Calculator | Estimativa de custos mensais antes e depois da otimização |

---

## 🏗️ Arquitetura da solução

```
          ANTES da otimização                  DEPOIS da otimização
┌─────────────────────────────┐      ┌─────────────────────────────┐
│      CafeInstance           │      │      CafeInstance           │
│      t3.small               │      │      t3.micro  ✅           │
│                             │      │                             │
│  ┌───────────────────────┐  │      │  ┌───────────────────────┐  │
│  │  Apache + PHP + Café  │  │      │  │  Apache + PHP + Café  │  │
│  ├───────────────────────┤  │      │  └───────────────────────┘  │
│  │  MariaDB local        │  │      │                             │
│  │  (descomissionado) ❌ │  │      │  EBS: 20 GB gp2  ✅         │
│  └───────────────────────┘  │      └──────────────┬──────────────┘
│  EBS: 40 GB gp2             │                     │
└──────────────┬──────────────┘                     │
               │                                    │
               ▼                                    ▼
┌─────────────────────────────┐      ┌─────────────────────────────┐
│  Amazon RDS                 │      │  Amazon RDS                 │
│  MariaDB — db.t3.micro      │      │  MariaDB — db.t3.micro      │
│  20 GB gp2                  │      │  20 GB gp2                  │
└─────────────────────────────┘      └─────────────────────────────┘

  Custo estimado: ~$35.60/mês          Custo estimado: ~$25.18/mês
```

---

## 📋 Etapas de implementação

1. Conexão à **CafeInstance** via SSH
2. Parada e remoção do **MariaDB local** com `systemctl` e `yum`
3. Obtenção do Instance ID da CafeInstance via **AWS CLI**
4. **Stop** da instância via CLI Host
5. **Redimensionamento** do tipo de instância de `t3.small` para `t3.micro`
6. **Start** da instância e validação do novo estado via CLI
7. Teste da aplicação Café no novo DNS público gerado
8. Cálculo do custo **antes** da otimização no AWS Pricing Calculator
9. Cálculo do custo **depois** da otimização com EC2 e EBS reduzidos
10. Comparação e registro da economia mensal projetada

---

## ⚙️ Comandos AWS CLI executados

```bash
# Parar o banco de dados local na CafeInstance
sudo systemctl stop mariadb
sudo yum -y remove mariadb-server

# Obter o Instance ID da CafeInstance
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=CafeInstance" \
  --query "Reservations[*].Instances[*].InstanceId"

# Parar a instância
aws ec2 stop-instances --instance-ids <CafeInstance-ID>

# Redimensionar para t3.micro
aws ec2 modify-instance-attribute \
  --instance-id <CafeInstance-ID> \
  --instance-type "{\"Value\": \"t3.micro\"}"

# Iniciar a instância
aws ec2 start-instances --instance-ids <CafeInstance-ID>

# Verificar estado e novo IP/DNS
aws ec2 describe-instances \
  --instance-ids <CafeInstance-ID> \
  --query "Reservations[*].Instances[*].[InstanceType,PublicDnsName,PublicIpAddress,State.Name]"
```

---

## 💰 Comparativo de Custos — AWS Pricing Calculator

| Recurso | Antes | Depois |
|---|---|---|
| EC2 | t3.small / 40 GB EBS | t3.micro / 20 GB EBS |
| RDS | db.t3.micro / 20 GB | db.t3.micro / 20 GB |
| **Custo EC2 estimado** | ~$20.89/mês | ~$10.47/mês |
| **Custo RDS estimado** | ~$14.71/mês | ~$14.71/mês |
| **Total estimado** | **~$35.60/mês** | **~$25.18/mês** |
| **💸 Economia mensal** | — | **~$10.42/mês** |

> Valores calculados com o **AWS Pricing Calculator** usando configuração On-Demand, região US East (N. Virginia), Linux. Preços de referência — consulte o site da AWS para valores atualizados.

---

## 📸 Evidências

AWS Configure and remove MariaDB:
<img width="657" height="418" alt="awsconfigure-remove-mariadb" src="https://github.com/user-attachments/assets/8ae75c34-9309-4a35-a5d9-145054ce53ed" />

Describe and stop instance:
<img width="656" height="489" alt="describe-and-stop-instance" src="https://github.com/user-attachments/assets/b94c2535-945b-4f07-acb8-a9e24cb29316" />

Change instance size:
<img width="653" height="326" alt="change-t3micro-start-instance" src="https://github.com/user-attachments/assets/aa5db074-243b-4f25-8a98-158a8a493843" />

Pricing calculator estimate MariaDB + EC2:
<img width="1874" height="771" alt="pricing-calculator-mariadb-ec2" src="https://github.com/user-attachments/assets/83004870-2763-4393-b5ff-7fd3d67ab642" />

Estimated updated após trocar o tamanho da instancia EC2:
<img width="1891" height="771" alt="estimated-updated" src="https://github.com/user-attachments/assets/bc30fd4a-c951-45b2-b314-384706fce160" />

---

## 💡 Aprendizados

- ✅ Como **redimensionar uma instância EC2** em execução via AWS CLI (stop → modify → start)
- ✅ Remoção de serviços desnecessários para **reduzir CPU e armazenamento**
- ✅ Uso do **AWS Pricing Calculator** para estimar e comparar custos mensais
- ✅ Conceito de **FinOps**: identificar e eliminar desperdícios de recursos na nuvem
- ✅ Impacto direto do tipo de instância e tamanho do EBS no custo mensal
- ✅ Como usar `modify-instance-attribute` para alterar o tipo de instância via CLI

---

## 🔗 Referências

- [AWS Pricing Calculator](https://calculator.aws)
- [EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [modify-instance-attribute — AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/ec2/modify-instance-attribute.html)
- [Amazon RDS Pricing](https://aws.amazon.com/rds/pricing/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
