# 📊 Projeto 12 - Monitoramento e Conformidade com CloudWatch e AWS Config

![AWS](https://img.shields.io/badge/AWS-CloudWatch-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![Systems Manager](https://img.shields.io/badge/AWS-Systems_Manager-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![SNS](https://img.shields.io/badge/AWS-SNS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Config](https://img.shields.io/badge/AWS-Config-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Implementar monitoramento completo de uma instância EC2 com o agente CloudWatch, coletando logs de aplicação e métricas de sistema, criando filtros e alarmes para detecção de erros em tempo real, configurando notificações via SNS e auditando conformidade de recursos com AWS Config.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| AWS Systems Manager | Instalação e configuração do agente CloudWatch via Run Command e Parameter Store |
| Amazon CloudWatch Agent | Coleta de logs do Apache e métricas de CPU, disco e memória da instância |
| CloudWatch Logs | Armazenamento e análise dos logs de acesso e erro do servidor web |
| CloudWatch Metrics | Visualização de métricas de sistema coletadas pelo agente |
| CloudWatch Alarms | Alarme disparado ao detectar 5+ erros 404 em 1 minuto |
| Amazon SNS | Envio de notificações por e-mail para alarmes e mudanças de estado |
| AWS Config | Auditoria contínua de conformidade de tags e volumes EBS |
| Amazon EC2 | Instância Web Server monitorada pelo agente CloudWatch |

---

## 🏗️ Arquitetura da solução

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Account                          │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              EC2 — Web Server (Apache)               │   │
│  │                                                      │   │
│  │  CloudWatch Agent (instalado via SSM Run Command)    │   │
│  │  ├── Logs: /var/log/httpd/access_log → HttpAccessLog │   │
│  │  ├── Logs: /var/log/httpd/error_log  → HttpErrorLog  │   │
│  │  ├── Metrics: CPU (idle, iowait, user, system)       │   │
│  │  ├── Metrics: Disk (used%, inodes_free, io_time)     │   │
│  │  └── Metrics: Mem (used%) · Swap (used%)             │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                   │
│           ┌─────────────┴──────────────┐                    │
│           ▼                            ▼                    │
│  ┌─────────────────┐        ┌─────────────────────┐         │
│  │ CloudWatch Logs │        │ CloudWatch Metrics  │         │
│  │                 │        │ (CWAgent namespace) │         │
│  │  HttpAccessLog  │        │  CPU · Disk · Mem   │         │
│  │  HttpErrorLog   │        └─────────────────────┘         │
│  └────────┬────────┘                                        │
│           │                                                 │
│           ▼                                                 │
│  ┌─────────────────────────────┐                            │
│  │  Metric Filter: 404Errors   │                            │
│  │  Pattern: status_code=404   │                            │
│  │  Namespace: LogMetrics      │                            │
│  └────────────┬────────────────┘                            │
│               │                                             │
│               ▼                                             │
│  ┌─────────────────────────────┐                            │
│  │  CloudWatch Alarm           │                            │
│  │  "404 Errors"               │                            │
│  │  Condição: >= 5 em 1 min    │                            │
│  └────────────┬────────────────┘                            │
│               │                                             │
│               ▼                                             │
│  ┌────────────────────┐   ┌──────────────────────────────┐  │
│  │  Amazon SNS        │   │  AWS Config                  │  │
│  │  → E-mail Alarme   │   │  Regras de conformidade:     │  │
│  │  → E-mail Stop EC2 │   │  ├── required-tags (project) │  │
│  └────────────────────┘   │  └── ec2-volume-inuse-check  │  │
│                           └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Instalação do **CloudWatch Agent** na instância via **SSM Run Command** (`AWS-ConfigureAWSPackage`)
2. Criação do parâmetro `Monitor-Web-Server` no **Parameter Store** com a configuração do agente (logs + métricas)
3. Inicialização do agente via **Run Command** (`AmazonCloudWatch-ManageAgent`) referenciando o parâmetro
4. Geração de dados de log acessando o servidor web e forçando erros 404
5. Validação dos logs no CloudWatch Logs (`HttpAccessLog` e `HttpErrorLog`)
6. Criação do **Metric Filter** `404Errors` com pattern `status_code=404`
7. Criação do **CloudWatch Alarm** disparado ao atingir ≥ 5 erros 404 em 1 minuto
8. Confirmação da assinatura SNS e validação do alarme por e-mail
9. Exploração das métricas do CWAgent (CPU, disco, memória) no CloudWatch Metrics
10. Parada da instância EC2 e recebimento de notificação SNS em tempo real
11. Configuração do **AWS Config** com regras `required-tags` e `ec2-volume-inuse-check`
12. Análise dos resultados de conformidade (recursos compliant vs. non-compliant)

---

## ⚙️ Configuração do CloudWatch Agent (Parameter Store)

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "log_group_name": "HttpAccessLog",
            "file_path": "/var/log/httpd/access_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          },
          {
            "log_group_name": "HttpErrorLog",
            "file_path": "/var/log/httpd/error_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          }
        ]
      }
    }
  },
  "metrics": {
    "metrics_collected": {
      "cpu":    { "measurement": ["cpu_usage_idle","cpu_usage_iowait","cpu_usage_user","cpu_usage_system"], "metrics_collection_interval": 10 },
      "disk":   { "measurement": ["used_percent","inodes_free"], "metrics_collection_interval": 10, "resources": ["*"] },
      "diskio": { "measurement": ["io_time"], "metrics_collection_interval": 10, "resources": ["*"] },
      "mem":    { "measurement": ["mem_used_percent"], "metrics_collection_interval": 10 },
      "swap":   { "measurement": ["swap_used_percent"], "metrics_collection_interval": 10 }
    }
  }
}
```

---

## 🔔 Alarme e Metric Filter configurados

| Campo | Valor |
|---|---|
| Filter name | `404Errors` |
| Filter pattern | `[ip, id, user, timestamp, request, status_code=404, size]` |
| Metric namespace | `LogMetrics` |
| Metric name | `404Errors` |
| Alarm name | `404 Errors` |
| Condição | `>= 5` ocorrências em **1 minuto** |
| Ação | SNS → notificação por e-mail |

---

## 🛡️ Regras AWS Config aplicadas

| Regra | O que verifica | Resultado |
|---|---|---|
| `required-tags` | Recursos sem a tag `project` | EC2 Web Server: ✅ Compliant · Demais recursos: ❌ Non-compliant |
| `ec2-volume-inuse-check` | Volumes EBS não anexados a instâncias | Volume em uso: ✅ Compliant · Volume solto: ❌ Non-compliant |

---

## 📸 Evidências

Arquitetura do projeto:
<img width="760" height="243" alt="arquitetura-projeto" src="https://github.com/user-attachments/assets/873c0c11-5cc1-45a0-beaa-909d1935d044" />

Agente do CloudWatch instalado:
<img width="1598" height="479" alt="clouwatch-agent-installed" src="https://github.com/user-attachments/assets/6969c815-1c48-4a8e-a003-4967f59e1f3b" />

Logs do CloudWatch:
<img width="1643" height="375" alt="logs-cloudwatch" src="https://github.com/user-attachments/assets/cf27c2d7-2eae-4a89-9171-f62a5e8c3c3b" />

Red alarm CloudWatch:
<img width="1616" height="620" alt="red-alarm-cloudwatch" src="https://github.com/user-attachments/assets/0cd0b69e-af4f-44a5-92b6-37be0e2f67e5" />

Email alarm CloudWatch:
<img width="958" height="725" alt="email-alarm-cloudwatch" src="https://github.com/user-attachments/assets/020e8948-f5cc-4b74-a035-096f8c3b6305" />

Rules do AWS Config:
<img width="1308" height="384" alt="awsconfig-rules" src="https://github.com/user-attachments/assets/1434f7d2-bf31-46cb-9fbf-b557bb1a73b9" />

---

## 💡 Aprendizados

- ✅ Instalação e configuração do **CloudWatch Agent** via SSM Run Command e Parameter Store
- ✅ Coleta de **logs de aplicação** (Apache access/error) e envio ao CloudWatch Logs
- ✅ Coleta de **métricas de sistema** (CPU, disco, memória) invisíveis ao CloudWatch padrão
- ✅ Criação de **Metric Filters** para transformar dados de log em métricas mensuráveis
- ✅ Configuração de **alarmes** com thresholds e notificações automáticas via SNS
- ✅ Monitoramento de **mudanças de estado de infraestrutura** em tempo real com SNS
- ✅ Auditoria contínua de conformidade de recursos com **AWS Config**
- ✅ Diferença entre métricas padrão do CloudWatch e métricas coletadas pelo **CWAgent**

---

## 🔗 Referências

- [CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/)
- [CloudWatch Metric Filters](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/MonitoringLogData.html)
- [AWS Config](https://docs.aws.amazon.com/config/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
