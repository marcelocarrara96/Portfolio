# 🌐 Projeto 02, Criando uma VPC e implementando um Servidor Web com Acesso SSH

<div align="center">

![AWS](https://img.shields.io/badge/AWS-VPC-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-HTTP_SERVER-D22128?style=for-the-badge&logo=apache&logoColor=white)
![PuTTY](https://img.shields.io/badge/PuTTY-SSH-007ACC?style=for-the-badge&logoColor=white)
![Status](https://img.shields.io/badge/Status-CONCLUÍDO-1a9c3e?style=for-the-badge)

</div>

---

## 🎯 Objetivo

Compreender na prática como funciona a comunicação entre recursos dentro de uma VPC, além de aplicar regras de segurança e acesso remoto a uma instância EC2.

---

## 🛠️ Serviços Utilizados

| Serviço | Função |
|---|---|
| Amazon VPC | Rede virtual isolada na AWS |
| Amazon EC2 | Instância de servidor virtual na nuvem |
| Internet Gateway | Conectividade entre a VPC e a internet |
| NAT Gateway | Saída de tráfego para recursos em subnet privada |
| Security Groups | Firewall virtual, regras HTTP e SSH |
| PuTTY | Cliente SSH para acesso remoto à instância |
| Apache HTTP Server | Servidor web instalado na EC2 via User Data |

---

## 🏗️ Arquitetura da Solução

```
Internet
    |
    | HTTP (porta 80) / SSH (porta 22)
    ▼
┌──────────────────────────────┐
│       Internet Gateway        │
│   Conecta a VPC à internet   │
└──────────────────────────────┘
    |
    ▼
┌──────────────────────────────────────────────┐
│              VPC (10.0.0.0/16)               │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │       Subnet Pública (10.0.1.0/24)     │  │
│  │                                        │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │         Security Group           │  │  │
│  │  │   Porta 80 — HTTP   ✅           │  │  │
│  │  │   Porta 22 — SSH    ✅           │  │  │
│  │  └──────────────────────────────────┘  │  │
│  │                                        │  │
│  │  ┌──────────────────────────────────┐  │  │
│  │  │     Amazon EC2 (t2.micro)        │  │  │
│  │  │     Amazon Linux 2023            │  │  │
│  │  │   Apache HTTP Server             │  │  │
│  │  │   Acesso SSH via PuTTY (.ppk)    │  │  │
│  │  └──────────────────────────────────┘  │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  Route Table → 0.0.0.0/0 → Internet Gateway │
└──────────────────────────────────────────────┘
```

A infraestrutura foi construída dentro de uma rede virtual isolada utilizando o serviço Amazon VPC. Dentro da VPC foi criada uma subnet pública, onde foi implantada uma instância do Amazon EC2 responsável por hospedar a aplicação web. A conectividade com a internet é realizada por meio de um Internet Gateway, enquanto o roteamento do tráfego externo é controlado por uma Route Table associada à subnet pública. O acesso à instância é protegido por Security Groups, que definem as regras de segurança permitindo acesso SSH para administração do servidor e tráfego HTTP para acesso à aplicação web.

---

## 📋 Etapas de Implementação

1. Criação de uma VPC personalizada para isolar a infraestrutura
2. Configuração de subnets públicas para permitir acesso externo
3. Criação e associação de um Internet Gateway para habilitar conectividade com a internet
4. Configuração de um NAT Gateway para permitir saída de recursos privados
5. Definição de tabelas de rotas para direcionar o tráfego da rede
6. Criação de Security Groups controlando acesso HTTP e SSH
7. Lançamento de uma instância EC2 dentro da VPC criada
8. Configuração de acesso SSH utilizando chave `.ppk` via PuTTY
9. Implantação de uma aplicação web acessível via navegador

---

## 📝 Script User Data

```bash
#!/bin/bash
# Install Apache Web Server and PHP
yum install -y httpd mysql php

# Download Lab files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip
unzip lab-app.zip -d /var/www/html/

# Turn on web server
chkconfig httpd on
service httpd start
```

> **Nota:** Este script foi escrito para **Amazon Linux 2**. Se estiver usando **Amazon Linux 2023**, rode os comandos abaixo manualmente via SSH após conectar:

```bash
sudo dnf install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```

---

## 🔑 Conectando via SSH com PuTTY

1. Baixe e instale o [PuTTY](https://www.putty.org/)
2. Abra o **PuTTYgen** e carregue o arquivo `.pem` gerado na criação da EC2
3. Clique em **Save private key** e salve como `.ppk`
4. Abra o **PuTTY** e configure:
   - **Host Name:** `ec2-user@SEU_IP_PUBLICO`
   - **Port:** `22`
   - **Connection > SSH > Auth > Credentials:** selecione o arquivo `.ppk`
5. Clique em **Open** e aceite a chave do host na primeira conexão

---

## ⚠️ Troubleshooting — Amazon Linux 2023

Se o site não carregar após o launch da instância:

| Verificação | O que checar |
|---|---|
| Security Group | Porta 80 (HTTP) liberada para `0.0.0.0/0` |
| Status da instância | Status `running` + checks `2/2 passed` |
| Protocolo | Acessar via `http://` e não `https://` |
| IP correto | IP público copiado do console EC2 |
| User Data | Script colado corretamente antes do launch |
| AL2023 | Usar `dnf` e `systemctl` em vez de `yum` e `chkconfig` |

---

## 📸 Evidências

| # | Evidência |
|---|---|
| 1 | Criando a VPC personalizada no console AWS |
| 2 | Acesso SSH via PuTTY conectado com sucesso à instância EC2 |
| 3 | Servidor web rodando e acessível via navegador |

> Screenshots disponíveis na pasta [`/evidencias`](./evidencias/)

---

## 💡 Aprendizado

- Como estruturar uma rede isolada dentro da AWS com VPC, subnets e tabelas de roteamento
- A importância dos **Security Groups** no controle de acesso (HTTP e SSH)
- Diferença entre **subnet pública** e **privada**, e o papel do **Internet Gateway**
- O processo de acesso remoto seguro via **SSH** usando chave `.ppk` no PuTTY
- Diferença entre **Amazon Linux 2** (`yum`) e **Amazon Linux 2023** (`dnf`)
- Como implantar uma aplicação web simples e validar o servidor via navegador

---

## 🔗 Portfólio

- 📁 [Voltar ao Portfólio Principal](https://github.com/marcelocarrara96/Portfolio)

---

<div align="center">

**Marcelo Carrara** · Aspiring Solutions Architect · Paraná, Brasil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
