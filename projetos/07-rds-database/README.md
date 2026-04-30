# 🗄️ Projeto 07 — Banco de Dados Gerenciado com Amazon RDS

![AWS](https://img.shields.io/badge/AWS-RDS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-Database-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![VPC](https://img.shields.io/badge/AWS-VPC-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Provisionar um banco de dados relacional gerenciado com o Amazon RDS para MySQL em uma arquitetura Multi-AZ, configurando security groups, subnet groups e integrando o banco a uma aplicação web rodando em uma instância EC2.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon RDS | Banco de dados relacional gerenciado (MySQL) |
| Multi-AZ Deployment | Alta disponibilidade com réplica em outra AZ |
| DB Subnet Group | Define as subnets privadas disponíveis para o RDS |
| Security Group | Controle de acesso à porta 3306 do banco |
| Amazon VPC | Isolamento de rede para o banco e o servidor web |
| Amazon EC2 | Servidor web que consome o banco de dados |

---

## 🏗️ Arquitetura da solução

```
Internet
    │
    │  HTTP
    ▼
┌─────────────────────────────────────────────────────┐
│                      Lab VPC                        │
│                                                     │
│  ┌──────────────────────────────────────────────┐   │
│  │            Public Subnet                     │   │
│  │  ┌────────────────────────────────────┐      │   │
│  │  │  EC2 — Web Server                  │      │   │
│  │  │  Web Security Group                │      │   │
│  │  │  Address Book App                  │      │   │
│  │  └──────────────┬─────────────────────┘      │   │
│  └─────────────────┼────────────────────────────┘   │
│                    │ MySQL (porta 3306)              │
│  ┌─────────────────┼────────────────────────────┐   │
│  │         Private Subnets                      │   │
│  │                 │                            │   │
│  │  ┌──────────────▼──────────┐                 │   │
│  │  │  RDS MySQL — Primary    │                 │   │
│  │  │  AZ 1 — 10.0.1.0/24    │                 │   │
│  │  │  DB Security Group      │                 │   │
│  │  └──────────────┬──────────┘                 │   │
│  │                 │ Replicação síncrona         │   │
│  │  ┌──────────────▼──────────┐                 │   │
│  │  │  RDS MySQL — Standby    │                 │   │
│  │  │  AZ 2 — 10.0.3.0/24    │                 │   │
│  │  └─────────────────────────┘                 │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Criação do **Security Group** `DB Security Group` permitindo tráfego MySQL (porta 3306) originado do `Web Security Group`
2. Criação do **DB Subnet Group** com as subnets privadas das duas Availability Zones (`10.0.1.0/24` e `10.0.3.0/24`)
3. Provisionamento da instância **Amazon RDS MySQL** em modo **Multi-AZ** com a classe `db.t3.medium`
4. Configuração da aplicação web **Address Book** com o endpoint do banco
5. Validação da persistência de dados adicionando, editando e removendo contatos na aplicação

---

## ⚙️ Configurações do Banco de Dados

| Parâmetro | Valor |
|---|---|
| Engine | MySQL |
| Template | Dev/Test |
| Disponibilidade | Multi-AZ DB Instance |
| Identificador | `lab-db` |
| Master username | `main` |
| Classe da instância | `db.t3.medium` (Burstable) |
| Storage type | General Purpose SSD |
| VPC | Lab VPC |
| Security Group | DB Security Group |
| Database name | `lab` |
| Backup automático | ❌ Desativado (ambiente de lab) |

---

## 🔒 Regra do Security Group

| Campo | Valor |
|---|---|
| Nome | DB Security Group |
| Tipo | MySQL/Aurora |
| Porta | 3306 |
| Origem | Web Security Group |

> O banco de dados só aceita conexões de instâncias EC2 associadas ao `Web Security Group`, garantindo isolamento e segurança na camada de rede.

---

## 📸 Evidências

Database RDS online:
<img width="1599" height="429" alt="rds-online" src="https://github.com/user-attachments/assets/2e8abe60-5a0b-4b03-bbc8-369827338cf9" />

Interaction with RDS:
<img width="1148" height="416" alt="interact-RDS" src="https://github.com/user-attachments/assets/5a37dfdf-81de-4b92-aaae-8f912696aa98" />



---

## 💡 Aprendizados

- ✅ Como provisionar um banco de dados gerenciado com **Amazon RDS MySQL**
- ✅ Conceito de **Multi-AZ**: replicação síncrona para alta disponibilidade e failover automático
- ✅ Uso de **DB Subnet Groups** para definir subnets privadas para o banco
- ✅ Isolamento de acesso ao banco via **Security Groups** na porta 3306
- ✅ Integração entre **EC2** e **RDS** dentro de uma VPC
- ✅ Diferença entre banco gerenciado (RDS) e auto-gerenciado em EC2

---

## 🔗 Referências

- [Documentação Amazon RDS](https://docs.aws.amazon.com/rds/)
- [RDS Multi-AZ Deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [DB Subnet Groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html)

---

<div align="center">

**Marcelo Carrara** · Aspiring Solutions Architect · Paraná, Brasil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>


*Projeto realizado — Abril 2026*

