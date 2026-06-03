# 📊 Projeto 17 - FinOps + Resource Optimization Center

![AWS](https://img.shields.io/badge/AWS-Lambda-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![SNS](https://img.shields.io/badge/AWS-SNS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## ⚠️ Problema

Equipes que operam na AWS sem visibilidade de recursos descobrem problemas tarde demais: instâncias ociosas consumindo custo, recursos sem tags impossibilitando rastreabilidade financeira e nenhum mecanismo de alerta proativo para desvios operacionais.

O desafio era: como construir um sistema de análise contínua que identificasse automaticamente instâncias subutilizadas, recursos fora de conformidade de tags e variações de custo e notificasse a equipe antes que o problema virasse fatura?

---

## 🎯 Objetivo

Construir um **Resource Optimization Center** serverless na AWS, integrando CloudWatch para monitoramento de métricas, três funções Lambda especializadas para análise de recursos, custos e conformidade de tags, SNS para alertas por e-mail e CloudWatch Logs para investigação de falhas, simulando a rotina real(supostamente) de um analista de operações cloud.

---

## 🏗️ Solução

```
EC2 (3 instâncias)
  │
  ├── web-dev (com tags ✅)
  ├── legacy-server (sem tags ❌)
  └── test-no-tags (sem tags ❌)
  │
  ▼
CloudWatch Metrics
  │
  ├── Dashboard NOC (4 widgets)
  ├── Alarme: CPU < 2% por 1h → SNS
  └── Alarme: CPU > 80% por 5min → SNS
  │
  ▼
EventBridge (agendamento)
  │
  ├──→ Lambda: resource-analyzer
  │     ├── Lista EC2 running
  │     ├── Verifica tags obrigatórias
  │     ├── Verifica CPU média (1h)
  │     └── Envia relatório via SNS
  │
  ├──→ Lambda: cost-analyzer
  │     ├── Consulta Cost Explorer (custo por serviço/dia)
  │     └── Salva JSON no S3
  │
  └──→ Lambda: tag-compliance-checker
        ├── Lista EC2 running
        ├── Verifica: Project, Environment, Owner
        └── Envia alerta via SNS se não conforme
  │
  ▼
SNS → resource-alerts
  │
  ▼
E-mail (relatório diário + alertas)
  │
  ▼
CloudWatch Logs (investigação e troubleshooting)
```

### Etapas de implementação

1. Criação das 3 instâncias EC2: `web-dev` (com tags), `legacy-server` e `test-no-tags` (sem tags)
2. Criação do bucket S3 `optimization-lab-XXXX` para armazenar dados de custo
3. Criação do Dashboard CloudWatch `OptimizationLab-NOC` com 4 widgets
4. Configuração dos alarmes de **CPU Baixa** (`< 2% por 1h`) e **CPU Alta** (`> 80% por 5min`)
5. Criação do tópico SNS `resource-alerts` com assinatura por e-mail confirmada
6. Criação da Role IAM `lambda-resource-analyzer-role` com permissões específicas
7. Deploy das 3 funções Lambda com agendamento via **EventBridge**
8. Testes: remoção de tag, ARN inválido para simular falha, nova EC2 sem tags
9. Investigação de logs no **CloudWatch Logs** - análise de erros e execuções

---

## 🛠️ Ferramentas

| Serviço | Função |
|---|---|
| Amazon EC2 | 3 instâncias com diferentes configurações de tags para teste |
| Amazon S3 | Armazenamento dos dados de custo exportados pelo Lambda |
| Amazon CloudWatch | Dashboard NOC, alarmes de CPU e análise de logs |
| Amazon EventBridge | Agendamento automático das 3 funções Lambda |
| AWS Lambda | Análise de recursos, custos e conformidade de tags |
| AWS Cost Explorer | Fonte dos dados de custo por serviço e por dia |
| Amazon SNS | Notificações por e-mail para alertas e relatórios |
| AWS IAM | Role com permissões específicas por responsabilidade |

---

## ⚙️ Detalhes técnicos

### Lambda 1- resource-analyzer

Lista instâncias EC2 em execução, verifica tags obrigatórias, mede CPU média da última hora e envia relatório diário via SNS.

```python
import boto3
import datetime

ec2 = boto3.client('ec2', region_name='us-east-1')
cw  = boto3.client('cloudwatch', region_name='us-east-1')
sns = boto3.client('sns', region_name='us-east-1')

SNS_TOPIC_ARN = 'arn:aws:sns:us-east-1:XXXX:resource-alerts'
REQUIRED_TAGS = ['Project', 'Environment', 'Owner']

def lambda_handler(event, context):
    report = []
    reservations = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )['Reservations']

    for r in reservations:
        for i in r['Instances']:
            iid     = i['InstanceId']
            tags    = {t['Key']: t['Value'] for t in i.get('Tags', [])}
            name    = tags.get('Name', 'sem-nome')
            missing = [k for k in REQUIRED_TAGS if k not in tags]
            launch  = i['LaunchTime']
            uptime  = (datetime.datetime.now(datetime.timezone.utc) - launch).seconds // 3600

            cpu = cw.get_metric_statistics(
                Namespace='AWS/EC2',
                MetricName='CPUUtilization',
                Dimensions=[{'Name': 'InstanceId', 'Value': iid}],
                StartTime=datetime.datetime.utcnow() - datetime.timedelta(hours=1),
                EndTime=datetime.datetime.utcnow(),
                Period=3600,
                Statistics=['Average']
            )
            cpu_avg = round(cpu['Datapoints'][0]['Average'], 2) if cpu['Datapoints'] else 0
            report.append({'instance': iid, 'name': name, 'uptime': uptime,
                           'cpu_avg': cpu_avg, 'missing_tags': missing})

    body = "📊 Relatório de Recursos — OptimizationLab\n\n"
    for r in report:
        body += f"Instância: {r['name']} ({r['instance']})\n"
        body += f"  CPU (1h): {r['cpu_avg']}% | Uptime: {r['uptime']}h\n"
        body += f"  Tags ausentes: {', '.join(r['missing_tags'])}\n\n" if r['missing_tags'] else "  ✅ Tags OK\n\n"

    sns.publish(TopicArn=SNS_TOPIC_ARN, Subject='[FinOps] Relatório Diário', Message=body)
    return {'report': report}
```

**Agendamento:** `cron(0 12 * * ? *)` — todo dia ao meio-dia UTC

---

### Lambda 2- cost-analyzer

Coleta dados de custo do Cost Explorer por serviço e salva como JSON no S3.

```python
import boto3, datetime, json

ce = boto3.client('ce', region_name='us-east-1')
s3 = boto3.client('s3')
BUCKET = 'optimization-lab-XXXX'

def lambda_handler(event, context):
    today = datetime.date.today()
    start = today.replace(day=1).strftime('%Y-%m-%d')
    end   = today.strftime('%Y-%m-%d')

    response = ce.get_cost_and_usage(
        TimePeriod={'Start': start, 'End': end},
        Granularity='DAILY',
        Metrics=['UnblendedCost'],
        GroupBy=[{'Type': 'DIMENSION', 'Key': 'SERVICE'}]
    )

    key = f"costs/{today.year}/{today.month:02d}/{today}.json"
    s3.put_object(Bucket=BUCKET, Key=key,
                  Body=json.dumps({'collected_at': str(datetime.datetime.utcnow()),
                                   'data': response['ResultsByTime']}, default=str),
                  ContentType='application/json')
    return {'statusCode': 200, 'key': key}
```

**Agendamento:** `cron(0 9 * * ? *)` — todo dia às 09h UTC

---

### Lambda 3- tag-compliance-checker

Verifica conformidade de tags em todas as instâncias e envia alerta via SNS para as não conformes.

```python
import boto3

ec2 = boto3.client('ec2', region_name='us-east-1')
sns = boto3.client('sns', region_name='us-east-1')

SNS_TOPIC_ARN = 'arn:aws:sns:us-east-1:XXXX:resource-alerts'
REQUIRED_TAGS = ['Project', 'Environment', 'Owner']

def lambda_handler(event, context):
    reservations = ec2.describe_instances(
        Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
    )['Reservations']

    non_compliant = []
    for r in reservations:
        for i in r['Instances']:
            iid     = i['InstanceId']
            tags    = {t['Key']: t['Value'] for t in i.get('Tags', [])}
            missing = [k for k in REQUIRED_TAGS if k not in tags]
            if missing:
                non_compliant.append({'id': iid, 'name': tags.get('Name','sem-nome'), 'missing': missing})

    if non_compliant:
        msg = "⚠️ Instâncias fora de conformidade:\n\n"
        for nc in non_compliant:
            msg += f"{nc['name']} ({nc['id']})\nTags ausentes: {', '.join(nc['missing'])}\n\n"
        sns.publish(TopicArn=SNS_TOPIC_ARN, Subject='[Compliance] Tags ausentes', Message=msg)

    return {'non_compliant': non_compliant, 'total': len(non_compliant)}
```

**Agendamento:** `cron(0 20 ? * MON-FRI *)` - dias úteis às 20h UTC

---

### CloudWatch Dashboard - 4 Widgets

| Widget | Métrica | Instância |
|---|---|---|
| Widget 1 | CPUUtilization | web-dev |
| Widget 2 | CPUUtilization | legacy-server |
| Widget 3 | NetworkIn | ambas |
| Widget 4 | NetworkOut | ambas |

### Alarmes configurados

| Alarme | Condição | Destino |
|---|---|---|
| `CPU-Baixa-legacy-server` | CPUUtilization < 2% por 1h | SNS → E-mail |
| `CPU-Alta-web-dev` | CPUUtilization > 80% por 5min | SNS → E-mail |

---

## 🧪 Testes realizados

**Teste 1- Remoção de tag**
- Removida a tag `Owner` da instância `web-dev`
- Lambda `tag-compliance-checker` executada manualmente
- Alerta recebido por e-mail com a instância e a tag ausente
<img width="1463" height="693" alt="teste-mostrando-instancia-semtags" src="https://github.com/user-attachments/assets/74458054-22d4-497e-8943-91ee4fd887bf" />

**Teste 2- CPU ociosa**
- `legacy-server` mantida em idle sem carga
- Alarme `CPU-Baixa-legacy-server` disparado após período configurado
- Notificação SNS recebida por e-mail
<img width="1676" height="628" alt="alarm-legacy-cpu" src="https://github.com/user-attachments/assets/4381ca25-680c-4f12-aa06-bb3f0a0d4a3e" />

**Teste 3- Nova instância sem tags**
- EC2 `test-no-tags` criada sem nenhuma tag
- Lambda detectou e incluiu no relatório de conformidade
<img width="1467" height="669" alt="nova-ec2-semtags-detectado" src="https://github.com/user-attachments/assets/0ab4c5a8-8671-4e88-b75d-0a7d1410447a" />

**Teste 4- Falha intencional para análise de logs**
- ARN do SNS alterado para valor inválido na Lambda `resource-analyzer`
- Função executada → erro gerado
- Investigação no **CloudWatch Logs**: localizado `ERROR`, `START RequestId` e `END RequestId`
- ARN corrigido e função reexecutada com sucesso

---

## 🔍 CloudWatch Logs- Investigação de falhas

```
/aws/lambda/resource-analyzer
/aws/lambda/cost-analyzer
/aws/lambda/tag-compliance-checker
```

| Linha no log | Significado |
|---|---|
| `START RequestId: xxx` | Início da execução |
| `END RequestId: xxx` | Fim da execução |
| `REPORT RequestId: xxx` | Duração, memória usada, custo de billing |
| Linhas com `print()` | Output customizado da função |
| Linhas com `ERROR` | Falhas a investigar e corrigir |

> Simular falhas intencionais e investigar os logs é rotina. Saber localizar um erro em CloudWatch Logs em produção é uma das habilidades mais cobradas em entrevistas para essas posições.

---

## ✅ Resultado

Três funções Lambda especializadas operando de forma independente, cada uma com sua responsabilidade, agendamento e permissões IAM distintas. Dashboard CloudWatch com visão em tempo real das instâncias, alarmes proativos disparando antes do problema virar fatura, e pipeline de conformidade de tags detectando desvios automaticamente, com logs completos para auditoria e troubleshooting.

---

## 📸 Evidências

Add trigger Lambda:
<img width="1027" height="430" alt="add-trigger-lambda" src="https://github.com/user-attachments/assets/043193bf-5698-474d-952d-cb7624d2f329" />

Alerta e-mail avisando sem tags:
<img width="971" height="647" alt="alerta-email-ec2-semtags" src="https://github.com/user-attachments/assets/f5b2c960-e721-4380-ad88-f63feca34346" />

Lambda resource analyzer:
<img width="1459" height="774" alt="lambda-resourceanalyzer-succeeded" src="https://github.com/user-attachments/assets/b1215fbe-9d3e-427c-bd3c-bb3c60a33f04" />

Mais imagens na pasta evidencias.

---

## 💡 Aprendizados

**Separar responsabilidades entre Lambdas não é burocracia é arquitetura**
Meu primeiro instinto era colocar tudo em uma função só. Mantê-las separadas me forçou a pensar em permissões IAM distintas, agendamentos independentes e falhas isoladas. Uma função com erro não derruba as outras. Esse princípio se aplica a qualquer arquitetura serverless.

**Dashboard CloudWatch é um mini NOC**
Criar widgets para CPU, rede e instâncias diferentes em um painel único foi o momento em que entendi o que é observabilidade na prática.

**Alarme em 100% é tarde demais, em 2% é cedo o suficiente**
Configurar o alarme de CPU baixa me fez pensar em custo de forma diferente: uma instância idle pagando a hora inteira é desperdício silencioso. O alarme não serve para apagar incêndio, serve para evitar que ele comece.

**CloudWatch Logs é a caixa preta da Lambda**
Quando alterei o ARN do SNS para um valor inválido, a Lambda falhou silenciosamente no console. Os logs foram o único lugar onde o erro apareceu. Saber navegar em Log Groups, localizar o Request ID e ler o stack trace não é só debug, é investigação.

**Tag sem enforcement é tag que some**
As instâncias `legacy-server` e `test-no-tags` não tinham tags porque ninguém as adicionou na criação. Em escala, isso é inevitável sem automação. A Lambda de compliance existe porque confiar em processo manual não funciona, precisa de código verificando.

**IAM com menor privilégio protege o sistema de si mesmo**
A role da Lambda tem acesso apenas ao que ela precisa. Se alguém comprometer a função, o raio de impacto é limitado. Permissão excessiva em Lambda é tão perigosa quanto porta 22 aberta para o mundo.

---

## 🔗 Referências

- [AWS Cost Explorer](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-what-is.html)
- [Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
- [AWS Lambda](https://docs.aws.amazon.com/lambda/)
- [Amazon EventBridge — Scheduled Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html)
- [CloudWatch Logs — Lambda](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html)

---

<div align="center">

**Marcelo Carrara** · AWS Certified Cloud Practitioner | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
