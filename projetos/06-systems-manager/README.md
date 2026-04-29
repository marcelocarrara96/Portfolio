# 🖥️ Projeto 06 — Gerenciamento de Instâncias com AWS Systems Manager<img width="746" height="386" alt="widget-online" src="https://github.com/user-attachments/assets/2c59c452-71b2-4a56-bf69-6e755570929f" />


![AWS](https://img.shields.io/badge/AWS-Systems_Manager-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![SSM](https://img.shields.io/badge/SSM-Run_Command-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Session](https://img.shields.io/badge/Session_Manager-Acesso_Seguro-1A9C3E?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Utilizar o AWS Systems Manager para gerenciar instâncias EC2 sem necessidade de acesso SSH, coletando inventário automatizado, instalando aplicações via Run Command, gerenciando configurações com Parameter Store e acessando o terminal da instância de forma segura com Session Manager.

---

## 🛠️ Serviços utilizados

![Uploading widget-online.png…]()

| Serviço | Função |
|---|---|
| AWS Systems Manager | Gerenciamento centralizado de instâncias e configurações |
| Fleet Manager (Inventory) | Coleta automatizada de informações de software e SO |
| Run Command | Execução remota de scripts e comandos sem SSH |
| Parameter Store | Armazenamento seguro de configurações e parâmetros da aplicação |
| Session Manager | Acesso interativo ao terminal da instância sem abrir portas |
| Amazon EC2 | Instância gerenciada pelo Systems Manager |

---

## 🏗️ Arquitetura da solução

```
                        AWS Cloud
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
  ┌───────────────┐ ┌──────────────┐ ┌──────────────────┐
  │ Fleet Manager │ │ Run Command  │ │ Parameter Store  │
  │  (Inventory)  │ │              │ │                  │
  │  Coleta info  │ │ Instala app  │ │ /dashboard/      │
  │  de software  │ │ Apache + PHP │ │ show-beta-featur │
  │  e configs    │ │ + Dashboard  │ │ es = True        │
  └───────┬───────┘ └──────┬───────┘ └────────┬─────────┘
          │                │                  │
          └────────────────┼──────────────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │       VPC            │
                │  ┌────────────────┐  │
                │  │  EC2 Instance  │  │
                │  │ (SSM Agent ✅) │  │
                │  │                │  │
                │  │ Apache + PHP   │  │
                │  │ Widget Dashboard│  │
                │  └────────────────┘  │
                └──────────────────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Session Manager │
                  │ Acesso seguro   │
                  │ sem SSH / sem   │
                  │ portas abertas  │
                  └─────────────────┘
```

---

## 📋 Etapas de implementação

1. Configuração do **Fleet Manager** para coleta de inventário da instância EC2
2. Criação de uma **Inventory Association** apontando para a instância gerenciada
3. Instalação da aplicação **Widget Manufacturing Dashboard** via **Run Command**
4. Validação da aplicação acessando o IP público da instância no navegador
5. Criação do parâmetro `/dashboard/show-beta-features` no **Parameter Store**
6. Verificação da ativação de funcionalidade beta na aplicação via parâmetro
7. Acesso ao terminal da instância pelo **Session Manager** sem uso de SSH
8. Execução de comandos para listar arquivos e descrever instâncias EC2 via CLI

---

## ⚙️ Run Command — Aplicação Instalada

O documento de Run Command executou automaticamente um script de instalação que provisionou os seguintes componentes na instância:

| Componente | Descrição |
|---|---|
| Apache HTTP Server | Servidor web da aplicação |
| PHP | Linguagem de backend da aplicação |
| AWS SDK | Integração da aplicação com serviços AWS |
| Widget Manufacturing Dashboard | Aplicação web customizada |

---

## 🗂️ Parameter Store — Parâmetro Criado

| Campo | Valor |
|---|---|
| Nome | `/dashboard/show-beta-features` |
| Descrição | Display beta features |
| Tipo | String |
| Valor | `True` |
| Efeito | Ativa o terceiro gráfico (beta) na aplicação |

> A aplicação consulta automaticamente o Parameter Store a cada carregamento. Ao encontrar o parâmetro com valor `True`, exibe funcionalidades ainda em fase beta — uma técnica conhecida como **dark features** ou **feature flags**.

---

## 🔒 Session Manager — Comandos Executados

```bash
# Listar arquivos da aplicação instalada
ls /var/www/html

# Obter a região atual via metadata da instância
AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`
export AWS_DEFAULT_REGION=${AZ::-1}

# Listar detalhes da instância EC2 em formato JSON
aws ec2 describe-instances
```

---

## 📸 Evidências

Run command - AWS Systems Manager:
<img width="1616" height="429" alt="runcommand2-systemsmanager" src="https://github.com/user-attachments/assets/1d509093-561f-49b4-9fe7-c04911d0087a" />

Widget primário ---> Widget atualizado:
<img width="1094" height="391" alt="widgetatualizado-online" src="https://github.com/user-attachments/assets/5d84220d-da4f-41e0-82b7-80ff2b919b4b" />

Acesso na instancia com Systems Manager:
<img width="1533" height="720" alt="acesso-instancia-systemsmanager" src="https://github.com/user-attachments/assets/c753ad40-1b0b-474f-92ec-25d182545c71" />


---

## 💡 Aprendizados

- ✅ Como usar o **Fleet Manager** para coletar inventário de instâncias sem SSH
- ✅ Execução de scripts remotos com **Run Command** de forma segura e auditável
- ✅ Uso do **Parameter Store** para gerenciar configurações de aplicação de forma centralizada
- ✅ Conceito de **feature flags** (dark features) para ativar funcionalidades via parâmetro
- ✅ Acesso seguro ao terminal de instâncias com **Session Manager**, sem abrir portas ou gerenciar chaves SSH
- ✅ Auditoria e controle de acesso via **IAM** e **CloudTrail** integrados ao Systems Manager

---

## 🔗 Referências

- [Documentação AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/)
- [Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/execute-remote-commands.html)
- [Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html)

---

*Projeto realizado — Abril 2026*
