# ⚖️ Projeto 08 — Alta Disponibilidade com Auto Scaling e Load Balancer

![AWS](https://img.shields.io/badge/AWS-Auto_Scaling-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![ELB](https://img.shields.io/badge/AWS-Elastic_Load_Balancer-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Implementar uma arquitetura de alta disponibilidade e escalabilidade automática na AWS, criando uma AMI personalizada, configurando um Application Load Balancer, definindo um Launch Template e provisionando um Auto Scaling Group que distribui instâncias EC2 entre duas Availability Zones.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon EC2 AMI | Imagem base para lançamento das instâncias do Auto Scaling |
| Application Load Balancer (ALB) | Distribui o tráfego entre instâncias em múltiplas AZs |
| Target Group | Agrupa as instâncias que recebem tráfego do load balancer |
| Launch Template | Define as configurações das instâncias lançadas pelo Auto Scaling |
| Auto Scaling Group | Gerencia o número de instâncias conforme a demanda |
| Amazon VPC | Rede isolada com subnets públicas e privadas |

---

## 🏗️ Arquitetura da solução

```
Internet
    │
    │  HTTP
    ▼
┌──────────────────────────────────────────────────────────┐
│                        Lab VPC                           │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │               Public Subnets                       │  │
│  │                                                    │  │
│  │        ┌──────────────────────────┐                │  │
│  │        │  Application Load        │                │  │
│  │        │  Balancer — LabELB       │                │  │
│  │        │  Web Security Group      │                │  │
│  │        └────────────┬─────────────┘                │  │
│  └─────────────────────┼──────────────────────────────┘  │
│                        │ Distribui tráfego               │
│  ┌─────────────────────┼──────────────────────────────┐  │
│  │               Private Subnets                      │  │
│  │                     │                              │  │
│  │       ┌─────────────┴──────────────┐               │  │
│  │       │                            │               │  │
│  │  ┌────▼──────────────┐  ┌──────────▼────────────┐  │  │
│  │  │  Lab Instance     │  │  Lab Instance         │  │  │
│  │  │  AZ 1             │  │  AZ 2                 │  │  │
│  │  │  Private Subnet 1 │  │  Private Subnet 2     │  │  │
│  │  │  10.0.1.0/24      │  │  10.0.3.0/24          │  │  │
│  │  └───────────────────┘  └───────────────────────┘  │  │
│  │                                                    │  │
│  │        Auto Scaling Group (min: 2 | max: 4)        │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Criação de uma **AMI personalizada** (`Web Server AMI`) a partir da instância `Web Server 1`
2. Criação do **Application Load Balancer** `LabELB` nas subnets públicas das duas AZs
3. Criação do **Target Group** `lab-target-group` para registrar as instâncias
4. Criação do **Launch Template** `lab-app-launch-template` com a AMI, tipo `t3.micro` e Web Security Group
5. Criação do **Auto Scaling Group** `Lab Auto Scaling Group` nas subnets privadas das duas AZs
6. Configuração da **política de escalabilidade** baseada em CPU (target: 50%)
7. Validação do health check das instâncias no Target Group
8. Teste de acesso à aplicação via DNS do Load Balancer

---

## ⚙️ Configurações do Auto Scaling Group

| Parâmetro | Valor |
|---|---|
| Launch Template | `lab-app-launch-template` |
| AMI | Web Server AMI |
| Tipo de instância | `t3.micro` |
| VPC | Lab VPC |
| Subnets | Private Subnet 1 e Private Subnet 2 |
| Capacidade desejada | 2 instâncias |
| Capacidade mínima | 2 instâncias |
| Capacidade máxima | 4 instâncias |
| Política de scaling | Target Tracking — CPU médio em 50% |
| Health check | ELB |
| Tag das instâncias | `Name: Lab Instance` |

---

## 🔁 Política de Escalabilidade

| Campo | Valor |
|---|---|
| Tipo | Target Tracking Scaling Policy |
| Métrica | Average CPU Utilization |
| Target | 50% |
| Efeito | Adiciona ou remove instâncias automaticamente para manter a CPU próxima de 50% |

---

## 📸 Evidências



---

## 💡 Aprendizados

- ✅ Como criar uma **AMI personalizada** a partir de uma instância existente
- ✅ Configuração de um **Application Load Balancer** com Target Groups em múltiplas AZs
- ✅ Uso de **Launch Templates** para padronizar o lançamento de instâncias
- ✅ Como o **Auto Scaling Group** mantém a disponibilidade com capacidade mínima e máxima
- ✅ Política de **Target Tracking** para escalabilidade automática baseada em métricas de CPU
- ✅ Diferença entre subnets **públicas** (load balancer) e **privadas** (instâncias de aplicação)

---

## 🔗 Referências

- [Documentação Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/)
- [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [Launch Templates](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html)

---

<div align="center">

**Marcelo Carrara** · Aspiring Solutions Architect · Paraná, Brasil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
