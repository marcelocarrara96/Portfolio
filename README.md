# 🖥️ Projeto 01, Servidor Web com Amazon EC2

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Apache](https://img.shields.io/badge/Apache-HTTP_Server-D22128?style=for-the-badge&logo=apache&logoColor=white)
![Linux](https://img.shields.io/badge/Amazon_Linux-232F3E?style=for-the-badge&logo=linux&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Implantar um servidor web em uma instância Amazon EC2 a partir do 0, configurando regras de segurança, automatizando a instalação do Apache via User Data e explorando o redimensionamento de recursos da instância e do volume EBS.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon EC2 | Instância de servidor virtual na nuvem |
| Security Group | Controle de tráfego de entrada e saída |
| Amazon EBS | Armazenamento persistente da instância |
| Apache HTTP Server | Servidor web instalado via User Data |

---

## 🏗️ Arquitetura da solução

```
Internet
    │
    │  HTTP (porta 80) / HTTPS (porta 443)
    ▼
┌─────────────────────────────────────┐
│           Security Group            │
│   Permite tráfego HTTP e HTTPS      │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│         Amazon EC2 (t2.micro)       │
│         Amazon Linux 2              │
│                                     │
│  ┌─────────────────────────────┐    │
│  │     Apache HTTP Server      │    │
│  │   /var/www/html/index.html  │    │
│  └─────────────────────────────┘    │
│                                     │
│  User Data: script de instalação    │
│  Proteção contra término: ✅ ativa  │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│           Amazon EBS                │
│     Armazenamento persistente       │
└─────────────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Criação de uma instância Amazon EC2 na AWS
2. Seleção da Amazon Machine Image (AMI), Amazon Linux 2
3. Escolha do tipo de instância para testes
4. Configuração do Security Group permitindo acesso HTTP e HTTPS
5. Ativação da **Proteção contra término** para evitar exclusão acidental
6. Inserção do script Bash no campo **User Data** para instalação automática do Apache
7. Redimensionamento da instância e do volume EBS
8. Inicialização da instância e validação do servidor web

---

## 📄 Script User Data

```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Marcelo Carrara fazendo projetos!</h1></html>' > /var/www/html/index.html
```

> O script é executado automaticamente no boot da instância, instalando e iniciando o Apache sem necessidade de acesso manual via SSH.

---

## 📸 Evidências

> 

---

## 💡 Aprendizados

- ✅ Processo de criação e configuração de uma instância EC2
- ✅ Uso do **User Data** para automatizar a instalação de serviços no boot
- ✅ Papel dos **Security Groups** no controle de acesso à instância
- ✅ Importância do **Amazon EBS** como armazenamento persistente
- ✅ Como redimensionar tipo de instância e volume de armazenamento
- ✅ Proteção contra término acidental de instâncias em produção

---

## 🔗 Referências

- [Documentação Amazon EC2](https://docs.aws.amazon.com/ec2/)
- [Apache HTTP Server](https://httpd.apache.org/)
- [Amazon EBS](https://docs.aws.amazon.com/ebs/)

---

*Projeto realizado no AWS Sandbox — Março 2026*
