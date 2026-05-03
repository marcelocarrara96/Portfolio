# 🌐 Projeto 09 — Alta Disponibilidade com Failover Routing no Amazon Route 53

![AWS](https://img.shields.io/badge/AWS-Route_53-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![SNS](https://img.shields.io/badge/AWS-SNS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Configurar failover routing no Amazon Route 53 para uma aplicação web, garantindo que o tráfego seja redirecionado automaticamente para uma instância secundária em outra Availability Zone caso a instância primária se torne indisponível, com alertas via e-mail configurados pelo SNS.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon Route 53 | Roteamento DNS com política de failover entre AZs |
| Route 53 Health Checks | Monitoramento contínuo da disponibilidade do endpoint primário |
| Amazon EC2 | Duas instâncias com LAMP stack rodando a aplicação café |
| Amazon SNS | Envio de alertas por e-mail quando o health check falha |
| AWS CloudFormation | Provisionamento inicial do ambiente de laboratório |

---

## 🏗️ Arquitetura da solução

```
                        Usuários
                            │
                            │  HTTP
                            ▼
                ┌───────────────────────┐
                │     Amazon Route 53   │
                │   Hosted Zone (DNS)   │
                │                       │
                │  www.dominio.training │
                │  TTL: 15 segundos     │
                └───────────┬───────────┘
                            │
              ┌─────────────┴──────────────┐
              │  Failover Routing Policy   │
              └──────┬─────────────────────┘
                     │
        ┌────────────┴─────────────┐
        │                          │
        ▼                          ▼
┌───────────────────┐    ┌───────────────────┐
│  A Record         │    │  A Record         │
│  PRIMARY          │    │  SECONDARY        │
│  FailoverPrimary  │    │  FailoverSecondary│
│                   │    │                   │
│ ✅ Health Check   │    │  (sem health      │
│ Primary-Website   │    │   check)          │
└────────┬──────────┘    └────────┬──────────┘
         │                        │
         ▼                        ▼
┌─────────────────┐    ┌─────────────────────┐
│  CafeInstance1  │    │  CafeInstance2      │
│  AZ 1 (primary) │    │  AZ 2 (secondary)   │
│  us-west-2a     │    │  us-west-2b         │
│  LAMP + Café    │    │  LAMP + Café        │
└─────────────────┘    └─────────────────────┘
         │
         ▼
┌─────────────────────────────┐
│  Route 53 Health Check      │
│  Endpoint: /cafe            │
│  Interval: 10s (Fast)       │
│  Failure threshold: 2       │
│                             │
│  Se UNHEALTHY → SNS Alert   │
│  + Failover para Instance2  │
└─────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Confirmação das duas instâncias EC2 rodando a aplicação café em AZs distintas
2. Criação do **Health Check** `Primary-Website-Health` monitorando o endpoint `/cafe` da instância primária
3. Configuração do **SNS** para envio de alertas por e-mail quando o health check falhar
4. Confirmação da assinatura do tópico SNS via e-mail
5. Criação do **registro A primário** (`FailoverPrimary`) apontando para `CafeInstance1` com política de failover
6. Criação do **registro A secundário** (`FailoverSecondary`) apontando para `CafeInstance2`
7. Validação do DNS acessando `www.dominio.training/cafe` no navegador
8. Simulação de falha parando a `CafeInstance1` e verificando o redirecionamento automático para `CafeInstance2`
9. Confirmação do alerta de e-mail enviado pelo SNS com detalhes do alarme

---

## ⚙️ Configuração do Health Check

| Parâmetro | Valor |
|---|---|
| Nome | `Primary-Website-Health` |
| O que monitorar | Endpoint |
| Especificar por | Endereço IP |
| IP monitorado | IP público da CafeInstance1 |
| Path | `/cafe` |
| Request interval | Fast (10 segundos) |
| Failure threshold | 2 falhas consecutivas |
| Alarme SNS | Sim — novo tópico `Primary-Website-Health` |

---

## 🗂️ Registros DNS Configurados

| Record Name | Tipo | Valor | Routing Policy | Failover Type | Health Check |
|---|---|---|---|---|---|
| `www` | A | IP da CafeInstance1 | Failover | Primary | Primary-Website-Health |
| `www` | A | IP da CafeInstance2 | Failover | Secondary | — |

> TTL configurado em **15 segundos** para acelerar a propagação das mudanças DNS durante o teste de failover.

---

## 🔁 Fluxo de Failover Testado

```
1. CafeInstance1 parada manualmente (simulação de falha)
         │
         ▼
2. Health Check detecta endpoint indisponível
   (2 falhas consecutivas em intervalos de 10s)
         │
         ▼
3. Status muda para UNHEALTHY
         │
         ├──→ SNS dispara e-mail de alerta:
         │    "ALARM: Primary-Website-Health-awsroute53-..."
         │
         └──→ Route 53 redireciona tráfego DNS
              para o registro SECONDARY (CafeInstance2)
         │
         ▼
4. Aplicação responde normalmente pela AZ 2 (us-west-2b)
```

---

## 📸 Evidências

Primary healthcheck:
<img width="1607" height="595" alt="primary-healthcheck" src="https://github.com/user-attachments/assets/5c95470c-bc25-471e-91fb-622393a11c90" />

Hosted Zones record:
<img width="1215" height="698" alt="hostedzones-record" src="https://github.com/user-attachments/assets/e5768205-bae0-41ce-8cdc-0d534e466468" />

Healthcheck unhealthy:
<img width="1303" height="319" alt="healthcheck-unhealthy" src="https://github.com/user-attachments/assets/3e9652b7-6e8d-4b83-b4a8-893914ad1633" />


---

## 💡 Aprendizados

- ✅ Como configurar **Failover Routing** no Route 53 com registros primário e secundário
- ✅ Uso de **Health Checks** para monitoramento contínuo de endpoints HTTP
- ✅ Integração entre **Route 53 e SNS** para alertas automáticos por e-mail
- ✅ Importância do **TTL baixo** para agilizar a propagação de mudanças DNS durante failover
- ✅ Como simular e validar um cenário de **falha e recuperação** entre Availability Zones
- ✅ Diferença entre registros DNS **Primary** e **Secondary** em políticas de failover

---

## 🔗 Referências

- [Documentação Amazon Route 53](https://docs.aws.amazon.com/route53/)
- [Failover Routing Policy](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-failover.html)
- [Route 53 Health Checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/health-checks-creating.html)
- [Amazon SNS](https://docs.aws.amazon.com/sns/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
