# 🔍 Projeto 15 — Investigação de Segurança com AWS CloudTrail e Athena

![AWS](https://img.shields.io/badge/AWS-CloudTrail-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Athena](https://img.shields.io/badge/AWS-Athena-EF9F27?style=for-the-badge&logo=amazonaws&logoColor=white)
![IAM](https://img.shields.io/badge/AWS-IAM-DD344C?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Investigar uma invasão ao servidor web do Café utilizando AWS CloudTrail para auditoria de ações na conta, analisar os logs com grep e AWS CLI, consultar os logs via Amazon Athena com SQL para identificar o hacker, remover o acesso indevido, corrigir as vulnerabilidades de segurança e restaurar o site comprometido.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| AWS CloudTrail | Registro de todas as ações realizadas na conta AWS |
| Amazon S3 | Armazenamento dos logs gerados pelo CloudTrail |
| Amazon Athena | Consulta SQL sobre os logs JSON armazenados no S3 |
| AWS IAM | Remoção do usuário malicioso `chaos` da conta AWS |
| Amazon EC2 | Instância web server comprometida e recuperada |
| AWS KMS | Criptografia dos logs do CloudTrail |
| Linux / SSH | Acesso à instância para remoção do invasor e correção do SSH |

---

## 🏗️ Arquitetura da solução

```
┌──────────────────────────────────────────────────────────────────┐
│                         AWS Account                              │
│                                                                  │
│  Todas as ações na conta                                         │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────────────┐                                         │
│  │   AWS CloudTrail    │──────→ S3: monitoring####/              │
│  │   Trail: monitor    │        (logs JSON.gz a cada 5 min)      │
│  └─────────────────────┘                                         │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  Análise dos Logs                                        │    │
│  │                                                          │    │
│  │  Método 1: grep + AWS CLI (terminal SSH)                 │    │
│  │  ├── gunzip *.gz                                         │    │
│  │  ├── grep sourceIPAddress / eventName                    │    │
│  │  └── aws cloudtrail lookup-events                        │    │
│  │                                                          │    │
│  │  Método 2: Amazon Athena (SQL)                           │    │
│  │  ├── CREATE TABLE sobre S3                               │    │
│  │  ├── SELECT userName, eventtime, eventname               │    │
│  │  └── WHERE eventsource + LIKE '%Security%'               │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                  │
│  🔴 Invasão identificada: usuário IAM chaos                      │
│  ├── Abriu porta 22 (SSH) para 0.0.0.0/0 no Security Group      │
│  ├── Conectou via SSH com senha (PasswordAuthentication)         │
│  ├── Substituiu imagem do site                                   │
│  └── Criou OS user chaos-user na instância EC2                   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Abertura da porta 22 no Security Group para o IP próprio e observação do site
2. Criação do **CloudTrail Trail** `monitor` com logs armazenados no S3 + criptografia KMS
3. Detecção da invasão: regra SSH `0.0.0.0/0` adicionada e imagem do site substituída
4. Download e extração dos logs CloudTrail via AWS CLI (`aws s3 cp` + `gunzip`)
5. Análise dos logs com **grep** filtrando por `sourceIPAddress` e `eventName`
6. Uso do **AWS CLI CloudTrail** (`lookup-events`) para filtrar por Security Group
7. Criação da tabela **Athena** apontando para o bucket S3 do CloudTrail
8. Identificação do hacker via **query SQL** no Athena
9. Remoção do OS user `chaos-user` e kill do processo ativo na instância
10. Correção das configurações de SSH (`PasswordAuthentication no`)
11. Remoção da regra SSH `0.0.0.0/0` do Security Group
12. Restauração da imagem original do site
13. Exclusão do **usuário IAM** `chaos` da conta AWS

---

## 🔎 Queries Athena utilizadas

```sql
-- Visualizar os primeiros registros
SELECT *
FROM cloudtrail_logs_monitoring####
LIMIT 5;

-- Filtrar colunas relevantes
SELECT useridentity.userName, eventtime, eventsource, eventname, requestparameters
FROM cloudtrail_logs_monitoring####
LIMIT 30;

-- Identificar todos os usuários ativos nas últimas 24h e suas ações
SELECT DISTINCT useridentity.userName, eventName, eventSource
FROM cloudtrail_logs_monitoring####
WHERE from_iso8601_timestamp(eventtime) > date_add('day', -1, now())
ORDER BY eventSource;

-- Filtrar ações relacionadas a Security Groups (EC2)
SELECT useridentity.userName, eventtime, eventname, requestparameters
FROM cloudtrail_logs_monitoring####
WHERE eventsource = 'ec2.amazonaws.com'
AND eventname LIKE '%Security%';
```

---

## ⚙️ Comandos Linux executados na instância

```bash
# Baixar logs do S3
mkdir ctraillogs && cd ctraillogs
aws s3 cp s3://<monitoring####>/ . --recursive

# Extrair logs
gunzip *.gz

# Analisar sourceIPAddress nos logs
for i in $(ls); do echo $i && cat $i | python -m json.tool | grep sourceIPAddress; done

# Analisar eventName nos logs
for i in $(ls); do echo $i && cat $i | python -m json.tool | grep eventName; done

# Verificar logins recentes no OS
sudo aureport --auth

# Ver usuários atualmente conectados
who

# Remover o chaos-user (forçar desconexão do processo)
sudo userdel -r chaos-user
sudo kill -9 <ProcNum>
sudo userdel -r chaos-user

# Verificar usuários com login no sistema
sudo cat /etc/passwd | grep -v nologin

# Corrigir configuração SSH (desativar PasswordAuthentication)
sudo vi /etc/ssh/sshd_config
# Comentar: PasswordAuthentication yes  →  #PasswordAuthentication yes
# Descomentar: #PasswordAuthentication no  →  PasswordAuthentication no
sudo service sshd restart

# Restaurar imagem do site
cd /var/www/html/cafe/images/
sudo mv Coffee-and-Pastries.backup Coffee-and-Pastries.jpg
```

---

## 🔐 Vulnerabilidades encontradas e corrigidas

| Vulnerabilidade | Como foi explorada | Correção aplicada |
|---|---|---|
| Regra SSH `0.0.0.0/0` no Security Group | Hacker adicionou acesso SSH irrestrito via AWS CLI | Regra removida do Security Group |
| `PasswordAuthentication yes` no sshd | Acesso SSH com usuário/senha sem precisar de key pair | Configuração alterada para `no` |
| Usuário IAM `chaos` com permissões EC2 | Usado para modificar Security Group programaticamente | Usuário IAM deletado |
| OS user `chaos-user` na instância | Criado pelo hacker para acesso persistente | Usuário removido com `userdel` |
| Imagem do site substituída | Acesso ao sistema de arquivos via SSH | Backup restaurado |

---

## 📸 Evidências

Site com imagem alterada:
<img width="995" height="892" alt="site-alterado" src="https://github.com/user-attachments/assets/29b9451d-c27d-4c11-a85c-94603538b7e2" />

Grupo de segurança com porta 22 (SSH) liberada para o público:
<img width="1629" height="351" alt="porta22-ssh-liberada-publico" src="https://github.com/user-attachments/assets/5d4a7697-9a84-400e-8c9f-7baba9f9c56e" />

Command "who":
<img width="656" height="410" alt="who-command-ssh" src="https://github.com/user-attachments/assets/e8109331-5aab-4644-9941-569f725e0880" />

Site com imagem corrigida:
<img width="991" height="798" alt="website-fixed" src="https://github.com/user-attachments/assets/e18399df-a94a-44c5-97ad-9b6338c42aa3" />

Remove wrong IAM user:
<img width="599" height="470" alt="remove-user-from-IAM" src="https://github.com/user-attachments/assets/5f5de810-3b30-4293-863a-d4895aa57d6c" />

---

## 💡 Aprendizados

- ✅ Como criar e configurar um **CloudTrail Trail** para auditoria completa da conta AWS
- ✅ Análise de logs JSON com **grep** e **AWS CLI** (`lookup-events`) via terminal
- ✅ Uso do **Amazon Athena** para consultas SQL sobre logs CloudTrail armazenados no S3
- ✅ Identificação de ações maliciosas por **usuário, IP, horário e método** nos logs
- ✅ Remoção de usuários ativos em sessão SSH com `kill` + `userdel`
- ✅ Importância de desabilitar **PasswordAuthentication** no SSH em ambientes cloud
- ✅ Como Security Groups mal configurados (`0.0.0.0/0`) representam vetores de ataque
- ✅ Princípio do **menor privilégio**: usuários IAM com permissões desnecessárias são riscos reais

---

## 🔗 Referências

- [AWS CloudTrail](https://docs.aws.amazon.com/cloudtrail/)
- [Analisar logs com Athena](https://docs.aws.amazon.com/athena/latest/ug/cloudtrail-logs.html)
- [CloudTrail Record Structure](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-event-reference.html)
- [AWS CloudTrail Partners](https://aws.amazon.com/cloudtrail/partners/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
