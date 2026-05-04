# 💰 Projeto 10 — FinOps na AWS: Monitoramento e Governança de Custos

![AWS](https://img.shields.io/badge/AWS-Cost_Explorer-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![Athena](https://img.shields.io/badge/AWS-Athena-EF9F27?style=for-the-badge&logo=amazonaws&logoColor=white)
![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Implementar uma pipeline completa de FinOps na AWS, coletando dados de custo automaticamente via Lambda, armazenando no S3, consultando com Athena, configurando alertas de budget e automatizando governança de instâncias EC2 sem tags — tudo dentro do Free Tier.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon EC2 | Instância de laboratório com tags de custo aplicadas |
| Amazon S3 | Armazenamento dos dados de custo exportados pelo Lambda |
| AWS Lambda | Coleta diária de dados do Cost Explorer e governança de instâncias |
| Amazon EventBridge | Agendamento automático das funções Lambda |
| AWS Cost Explorer | Fonte dos dados de custo e uso por serviço e tag |
| Amazon Athena | Consulta SQL dos dados de custo armazenados no S3 |
| AWS Budgets | Alertas proativos de custo com múltiplos thresholds |
| Amazon CloudWatch | Alarmes de billing e monitoramento de instâncias ociosas |
| Amazon SNS | Notificações por e-mail dos alarmes e alertas |
| AWS IAM | Controle de acesso com roles e políticas específicas por função |

---

## 🏗️ Arquitetura da solução

```
┌──────────────────────────────────────────────────────────────────┐
│                          AWS Account                             │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │              Pipeline de Coleta de Dados                │     │
│  │                                                         │     │
│  │  EventBridge (cron diário 09h UTC)                      │     │
│  │       │                                                 │     │
│  │       ▼                                                 │     │
│  │  Lambda: finops-cost-collector                          │     │
│  │       │  (Python 3.12 — Role: lambda-finops-role)       │     │
│  │       │                                                 │     │
│  │       ├──→ Cost Explorer API                            │     │
│  │       │    (custo por serviço + tag Project)            │     │
│  │       │                                                 │     │
│  │       └──→ S3: seu-bucket-finops/                       │     │
│  │                costs/YYYY/MM/YYYY-MM-DD.json            │     │
│  └─────────────────────────────────────────────────────────┘     │
│                                                                  │
│  ┌──────────────────────────┐   ┌──────────────────────────┐     │
│  │     Consulta de Dados    │   │   Alertas & Governança   │     │
│  │                          │   │                          │     │
│  │  Athena → finops_db      │   │  AWS Budgets             │     │
│  │  Queries SQL sobre       │   │  → 50% / 80% / 100%      │     │
│  │  os JSONs do S3          │   │  → SNS → E-mail          │     │
│  │                          │   │                          │     │
│  │  Results em:             │   │  CloudWatch Alarm        │     │
│  │  s3://.../athena-results │   │  → Billing > $2          │     │
│  └──────────────────────────┘   │  → EC2 CPU < 1% (ociosa) │     │
│                                 └──────────────────────────┘     │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │           Fase 6 — Automação de Governança              │     │
│  │                                                         │     │
│  │  EventBridge (cron dias úteis 20h UTC / 17h Brasília)   │     │
│  │       │                                                 │     │
│  │       ▼                                                 │     │
│  │  Lambda: finops-governance-bot                          │     │
│  │  → Lista instâncias EC2 running                         │     │
│  │  → Verifica tags: Project, Environment, Owner           │     │
│  │  → Para instâncias sem tags obrigatórias                │     │
│  └─────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────┘
```

---

## 📋 Fases de implementação

| Fase | Nome | Tempo |
|---|---|---|
| 0 | Preparação da conta AWS (IAM, MFA, Cost Explorer) | 10–15 min |
| 1 | Simulação de ambiente com EC2 + S3 tagueados | 20–30 min |
| 2 | Exportação de dados de custo com Lambda + EventBridge | 30–45 min |
| 3 | Consulta dos dados com Amazon Athena | 20–30 min |
| 4 | Alertas de custo com AWS Budgets | 10 min |
| 5 | Monitoramento com CloudWatch (billing + CPU ociosa) | 15 min |
| 6 ✦ | Automação — Lambda para parar instâncias sem tag | 45 min |

---

## 🏷️ Tags aplicadas na EC2 (base do FinOps)

```bash
Project     = FinOpsLab
Environment = Dev
Owner       = SeuNome
CostCenter  = Laboratorio
```

> Tags são a base do FinOps. Sem elas, não é possível filtrar custos por projeto no Cost Explorer.

---

## ⚙️ Lambda 1 — finops-cost-collector

Coleta dados diários do Cost Explorer e salva como JSON no S3.

```python
import boto3
import datetime
import json

ce = boto3.client('ce', region_name='us-east-1')
s3 = boto3.client('s3')

BUCKET = 'seu-bucket-finops-XXXX'

def lambda_handler(event, context):
    today = datetime.date.today()
    start = today.replace(day=1).strftime('%Y-%m-%d')
    end   = today.strftime('%Y-%m-%d')

    response = ce.get_cost_and_usage(
        TimePeriod={'Start': start, 'End': end},
        Granularity='DAILY',
        Metrics=['UnblendedCost', 'UsageQuantity'],
        GroupBy=[
            {'Type': 'DIMENSION', 'Key': 'SERVICE'},
            {'Type': 'TAG',       'Key': 'Project'}
        ]
    )

    payload = {
        'collected_at': str(datetime.datetime.utcnow()),
        'period': {'start': start, 'end': end},
        'data': response['ResultsByTime']
    }

    key = f"costs/{today.year}/{today.month:02d}/{today}.json"
    s3.put_object(
        Bucket=BUCKET,
        Key=key,
        Body=json.dumps(payload, default=str),
        ContentType='application/json'
    )

    return {'statusCode': 200, 'key': key}
```

**Agendamento EventBridge:** `cron(0 9 * * ? *)` → todo dia às 09h UTC (06h Brasília)

---

## ⚙️ Lambda 2 — finops-governance-bot

Para automaticamente instâncias EC2 sem as tags obrigatórias.

```python
import boto3

ec2 = boto3.client('ec2', region_name='us-east-1')
REQUIRED_TAGS = ['Project', 'Environment', 'Owner']

def lambda_handler(event, context):
    reservations = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )['Reservations']

    stopped = []
    for r in reservations:
        for i in r['Instances']:
            iid  = i['InstanceId']
            tags = {t['Key']: t['Value'] for t in i.get('Tags', [])}

            missing = [k for k in REQUIRED_TAGS if k not in tags]
            if missing:
                print(f"STOP {iid} — missing tags: {missing}")
                ec2.stop_instances(InstanceIds=[iid])
                stopped.append({'id': iid, 'missing_tags': missing})

    return {'stopped': stopped, 'total': len(stopped)}
```

**Agendamento EventBridge:** `cron(0 20 ? * MON-FRI *)` → dias úteis às 20h UTC (17h Brasília)

---

## 🗂️ Queries Athena

```sql
-- Criar banco de dados
CREATE DATABASE finops_db;

-- Criar tabela externa apontando para o S3
CREATE EXTERNAL TABLE finops_db.cost_data (
  collected_at string,
  period       struct<start:string, end:string>,
  data         string
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION 's3://seu-bucket-finops-XXXX/costs/'
TBLPROPERTIES ('has_encrypted_data'='false');

-- Ver registros coletados
SELECT * FROM finops_db.cost_data LIMIT 10;

-- Verificar datas coletadas
SELECT collected_at, period.start, period.end
FROM finops_db.cost_data
ORDER BY collected_at DESC;
```

---

## 🔔 Configuração de Alertas

**AWS Budgets — thresholds múltiplos:**

| Threshold | Ação |
|---|---|
| 50% do orçamento | E-mail de aviso antecipado |
| 80% do orçamento | E-mail principal de alerta |
| 100% do orçamento | E-mail de limite atingido |

**CloudWatch Alarms:**
- Billing total estimado > $2 → SNS → E-mail
- EC2 CPU < 1% por 1h → instância ociosa candidata a desligamento

---

## 📸 Evidências



---

## 💡 Aprendizados

- ✅ Conceito de **FinOps**: visibilidade, otimização e governança de custos em nuvem
- ✅ Uso do **Cost Explorer** para análise de custos por serviço e tag
- ✅ Criação de **pipeline serverless** com Lambda + EventBridge + S3
- ✅ Consulta de dados com **Amazon Athena** usando SQL sobre arquivos JSON no S3
- ✅ Configuração de **AWS Budgets** com alertas proativos em múltiplos thresholds
- ✅ Monitoramento de billing e instâncias ociosas com **CloudWatch**
- ✅ Automação de governança com **Lambda** parando instâncias sem tags obrigatórias
- ✅ Importância das **tags** como base para rastreabilidade de custos por projeto

---

## 🔗 Referências

- [AWS Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [Amazon Athena](https://docs.aws.amazon.com/athena/)
- [AWS Lambda](https://docs.aws.amazon.com/lambda/)
- [Amazon EventBridge](https://docs.aws.amazon.com/eventbridge/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
