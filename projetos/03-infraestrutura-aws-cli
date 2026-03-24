# 💻 Projeto 03: Provisionando Infraestrutura AWS via CLI (Linha de Comando)

<div align="center">

![AWS](https://img.shields.io/badge/AWS-CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![VPC](https://img.shields.io/badge/AWS-VPC-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-5391FE?style=for-the-badge&logo=powershell&logoColor=white)
![Status](https://img.shields.io/badge/Status-CONCLUÍDO-1a9c3e?style=for-the-badge)

</div>

---

## 🎯 Objetivo

Provisionar uma infraestrutura básica na AWS utilizando exclusivamente a linha de comando (**AWS CLI**), compreendendo a relação e dependência entre os recursos de rede e computação dentro da nuvem. 

O laboratório foi executado no ambiente sandbox do programa **AWS re/Start**, realizando a criação dos recursos manualmente para compreender cada etapa do processo de configuração de forma granular.

---

## 🛠️ Serviços Utilizados

| Serviço | Função |
|---|---|
| **Amazon VPC** | Rede virtual isolada na AWS |
| **Amazon EC2** | Instância de servidor virtual (Amazon Linux) |
| **Internet Gateway** | Conectividade entre a VPC e a internet |
| **Route Tables** | Controle de tráfego e roteamento da rede |
| **Security Groups** | Firewall virtual para permissão de acesso SSH |
| **AWS CLI** | Interface de linha de comando para provisionamento |
| **PowerShell** | Terminal para execução dos comandos e SSH |

---

## 🏗️ Arquitetura da Solução

A infraestrutura foi projetada utilizando uma **VPC dedicada** (não-default) com uma **subnet pública** conectada à internet por meio de um **Internet Gateway**. Uma instância EC2 com Amazon Linux foi implantada dentro desta subnet e protegida por um **Security Group** permitindo acesso SSH (porta 22). O acesso administrativo foi validado via terminal.

```text
Internet
    |
    | Porta 22 (SSH)
    ▼
┌──────────────────────────────┐
│       Internet Gateway       │
└──────────────────────────────┘
    |
    ▼
┌──────────────────────────────────────────────┐
│             VPC (10.0.0.0/16)                │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │       Subnet Pública (10.0.1.0/24)     │  │
│  │                                        │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │      Security Group (SSH)        │  │  │
│  │  └──────────────────────────────────┘  │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │   Amazon EC2 (Amazon Linux)      │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  Route Table → 0.0.0.0/0 → IGW               │
└──────────────────────────────────────────────┘
