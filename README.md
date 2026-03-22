# Projeto 01, Introdução ao Amazon EC2

## Objetivo
Implantar um servidor web Apache em uma instância EC2 com Security Group, User Data e EBS.

## Serviços AWS
- Amazon EC2
- Security Group
- Amazon EBS
- Apache HTTP Server

## O que foi feito
- Criação de instância EC2 com Amazon Linux 2023
- Configuração de Security Group (HTTP/HTTPS)
- Instalação automática do Apache via User Data
- Proteção contra término ativada
- Redimensionamento de instância e volume EBS

## Script User Data
```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo 'Marcelo Carrara fazendo projetos!' > /var/www/html/index.html
```

## Documentação
Ver arquivo `projeto01-ec2-passo-a-passo.docx`
