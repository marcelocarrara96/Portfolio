# 🌐 Projeto 18 - WordPress Production Architecture on AWS

![AWS](https://img.shields.io/badge/AWS-Solutions_Architecture-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![VPC](https://img.shields.io/badge/VPC-Networking-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![WordPress](https://img.shields.io/badge/WordPress-Production-21759B?style=for-the-badge&logo=wordpress&logoColor=white)
![CloudFront](https://img.shields.io/badge/CloudFront-CDN_Global-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## ⚠️ Problema

Hospedar um site WordPress em uma única instância EC2 com IP público exposto é inseguro, não escalável e sem observabilidade. Qualquer pico de tráfego derruba a aplicação, o banco de dados fica exposto à internet, não há cache global e nenhum mecanismo de alerta quando algo sai do ar.

O desafio era: como construir uma arquitetura WordPress de nível produção na AWS - com rede isolada, banco privado, distribuição global de conteúdo e monitoramento ativo - seguindo os princípios do Well-Architected Framework?

---

## 🎯 Objetivo

Implementar uma arquitetura completa de múltiplas camadas para hospedar WordPress na AWS, integrando VPC customizada com subnets públicas e privadas, Application Load Balancer, EC2, RDS MySQL em subnet privada, S3, CloudFront como CDN global com HTTPS, CloudWatch com alarmes de CPU e SNS para notificações - documentando cada decisão arquitetural e evidenciando a implementação para portfólio.

---

## 🏗️ Solução

A solução foi construída em 9 fases, partindo da rede e subindo camada por camada até a distribuição global. O tráfego público entra pelo CloudFront (HTTPS), passa pelo ALB (HTTP interno), chega à EC2 com WordPress e consulta o RDS MySQL em subnet privada, sem nenhuma exposição direta do banco à internet.

```
                          Internet
                              │
                              ▼
                   ┌─────────────────────┐
                   │     CloudFront      │
                   │  d30taqzz5a71zi     │
                   │  .cloudfront.net    │
                   │  HTTPS → CDN Global │
                   └──────────┬──────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │   wordpress-prod-vpc   │
                 │      10.0.0.0/16       │
                 │                        │
                 │  ┌──────────────────┐  │
                 │  │  Subnets Públicas│  │
                 │  │  10.0.1.0/24 (1a)│  │
                 │  │  10.0.2.0/24 (1b)│  │
                 │  │                  │  │
                 │  │ ┌──────────────┐ │  │
                 │  │ │  ALB (HTTP)  │ │  │
                 │  │ │wordpress-alb │ │  │
                 │  │ └──────┬───────┘ │  │
                 │  │        │         │  │
                 │  │ ┌──────▼───────┐ │  │
                 │  │ │     EC2      │ │  │
                 │  │ │ t3.micro     │ │  │
                 │  │ │ Amazon Linux │ │  │
                 │  │ │ 2023         │ │  │
                 │  │ │ WordPress +  │ │  │
                 │  │ │ Apache + PHP │ │  │
                 │  │ └─────────────┘ │  │
                 │  └──────────────────┘  │
                 │                        │
                 │  ┌──────────────────┐  │
                 │  │ Subnets Privadas │  │
                 │  │ 10.0.11.0/24(1a) │  │
                 │  │ 10.0.12.0/24(1b) │  │
                 │  │                  │  │
                 │  │ ┌──────────────┐ │  │
                 │  │ │  RDS MySQL   │ │  │
                 │  │ │ db.t3.micro  │ │  │
                 │  │ │ wordpressdb  │ │  │
                 │  │ │ Sem acesso   │ │  │
                 │  │ │ público      │ │  │
                 │  │ └─────────────┘ │  │
                 │  └──────────────────┘  │
                 └────────────────────────┘
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
      ┌─────────────────┐      ┌────────────────────┐
      │   CloudWatch    │      │        SNS         │
      │ wordpress-cpu-  │─────▶│ wordpress-alerts   │
      │ high (> 80%)    │      │ Email notification │
      │ wordpress-cpu-  │      └────────────────────┘
      │ low  (< 5%)     │
      └─────────────────┘
                 │
                 ▼
      ┌─────────────────┐
      │       S3        │
      │ wordpress-assets│
      │ Static assets / │
      │ evidências      │
      └─────────────────┘
```

### Etapas de implementação

1. Criação da **VPC customizada** `wordpress-prod-vpc` (10.0.0.0/16) com 4 subnets em 2 AZs, 2 públicas e 2 privadas
2. Configuração do **Internet Gateway**, route tables pública e privada com isolamento correto
3. Criação dos **3 Security Groups** em camadas: SG-ALB - SG-EC2 - SG-RDS
4. Criação do **RDS MySQL** `wordpressdb` em subnet privada, sem acesso público, com DB Subnet Group dedicado
5. Criação do **bucket S3** para assets estáticos e armazenamento de evidências
6. Lançamento da **EC2** `wordpress-server` (Amazon Linux 2023 + Apache + PHP) em subnet pública
7. Instalação e configuração do **WordPress** com wp-config.php apontando para o endpoint do RDS
8. Criação do **Target Group** e **Application Load Balancer** com health check em `/wp-login.php`
9. Criação dos **alarmes CloudWatch** (CPU high > 80% e CPU low < 5%) integrados ao **tópico SNS** com notificação por email
10. Criação da **distribuição CloudFront** com origin no ALB, CachingDisabled e redirect HTTP para HTTPS

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon VPC | Rede isolada com subnets públicas e privadas em 2 AZs |
| Internet Gateway | Saída para internet das subnets públicas |
| Route Tables | Roteamento separado para camada pública e privada |
| Security Groups | Controle de tráfego em camadas: ALB → EC2 → RDS |
| Amazon EC2 | Servidor WordPress (Amazon Linux 2023, t3.micro) |
| Amazon RDS | Banco de dados MySQL em subnet privada (db.t3.micro) |
| Amazon S3 | Armazenamento de assets estáticos e evidências do projeto |
| Application Load Balancer | Distribuição de tráfego com health check na EC2 |
| Amazon CloudFront | CDN global com HTTPS e redirect automático |
| Amazon CloudWatch | Monitoramento de CPU com alarmes configurados |
| Amazon SNS | Notificações por email integradas aos alarmes |

---

## ⚙️ Detalhes técnicos

### Configuração de rede

| Recurso | Nome | CIDR / AZ |
|---|---|---|
| VPC | wordpress-prod-vpc | 10.0.0.0/16 |
| Subnet pública A | public-subnet-1a | 10.0.1.0/24 — us-east-1a |
| Subnet pública B | public-subnet-1b | 10.0.2.0/24 — us-east-1b |
| Subnet privada A | private-subnet-1a | 10.0.11.0/24 — us-east-1a |
| Subnet privada B | private-subnet-1b | 10.0.12.0/24 — us-east-1b |
| Internet Gateway | wordpress-igw | Anexado à VPC |
| Route Table pública | rtb-public | 0.0.0.0/0 → IGW |
| Route Table privada | rtb-private | Sem rota para IGW |

### Security Groups - regras de inbound

| Security Group | Porta | Protocolo | Source |
|---|---|---|---|
| SG-ALB | 80 | HTTP | 0.0.0.0/0 |
| SG-ALB | 443 | HTTPS | 0.0.0.0/0 |
| SG-EC2 | 80 | HTTP | SG-ALB |
| SG-EC2 | 22 | SSH | My IP |
| SG-RDS | 3306 | MySQL | SG-EC2 |

> Usar Security Group como source (em vez de CIDR) garante que apenas recursos com aquele SG específico podem se comunicar, princípio de menor privilégio aplicado na camada de rede.

### EC2 - Stack de aplicação

| Componente | Versão / Detalhe |
|---|---|
| AMI | Amazon Linux 2023 |
| Instance type | t3.micro (Free Tier eligible) |
| Apache HTTP Server | Instalado via dnf |
| PHP | php, php-mysqlnd, php-fpm, php-xml, php-gd, php-json, php-mbstring, php-curl |
| WordPress | Versão mais recente (wordpress.org/latest.tar.gz) |

### RDS MySQL

| Campo | Valor |
|---|---|
| DB identifier | wordpressdb |
| Engine | MySQL 8.0.x |
| Instance class | db.t3.micro |
| Storage | 20 GiB gp2 |
| Multi-AZ | Não (Free Tier - trade-off documentado) |
| Public access | No |
| Initial database name | wordpress |
| Endpoint | wordpressdb.cibikss4svrp.us-east-1.rds.amazonaws.com |

### CloudFront

| Campo | Valor |
|---|---|
| Origin | wordpress-alb-921039624.us-east-1.elb.amazonaws.com |
| Origin protocol | HTTP only |
| Viewer protocol policy | Redirect HTTP to HTTPS |
| Cache policy | CachingDisabled |
| Origin request policy | AllViewer |
| Distribution domain | d30taqzz5a71zi.cloudfront.net |

> `CachingDisabled` foi escolhido intencionalmente para o lab. Com cache ativo (CachingOptimized), o CloudFront armazena cookies de sessão e causa loop infinito no wp-admin.

### CloudWatch — Alarmes

| Alarme | Métrica | Threshold | Ação |
|---|---|---|---|
| wordpress-cpu-high | CPUUtilization | > 80% por 2 períodos de 5 min | SNS - wordpress-alerts |
| wordpress-cpu-low | CPUUtilization | < 5% por 2 períodos de 5 min | SNS - wordpress-alerts |

---

## ✅ Resultado

WordPress em produção acessível via `https://d30taqzz5a71zi.cloudfront.net` com arquitetura de múltiplas camadas: tráfego HTTPS distribuído globalmente pelo CloudFront, balanceado pelo ALB em 2 AZs, servido pela EC2 em subnet pública e persistido no RDS MySQL em subnet privada sem nenhuma exposição à internet. Observabilidade ativa via CloudWatch com notificações por email via SNS.

---

## 🔍 Well-Architected Framework - Análise

| Pilar | Status | Observação |
|---|---|---|
| Operational Excellence | ✅ Parcial | CloudWatch + SNS ativos. Deploy manual - CI/CD seria o próximo passo |
| Security | ✅ Bom | SGs em camadas, RDS privado, SSH restrito, CloudFront na frente |
| Reliability | ✅ Parcial | ALB em 2 AZs. RDS Single-AZ por restrição Free Tier (trade-off documentado) |
| Performance Efficiency | ✅ Bom | CloudFront reduz latência global e descarrega o backend |
| Cost Optimization | ✅ Bom | Free Tier em EC2 e RDS. ALB é o único custo fixo (~$16/mês) |
| Sustainability | ✅ Atendido | CloudFront reduz requisições ao backend, menor compute ativo |

> **Trade-off documentado:** RDS em Single-AZ e sem Auto Scaling são decisões de laboratório para evitar custos. Em produção, a arquitetura exigiria Multi-AZ RDS e Auto Scaling Group com mínimo de 2 instâncias em AZs distintas.

---

## 📸 Evidências

| 1 | VPC criada com CIDR 10.0.0.0/16 e status Available |
<img width="1484" height="396" alt="vpc-criada-ipv4-CIDR" src="https://github.com/user-attachments/assets/750af2ff-ea76-4f95-ab1d-7505ba48e370" />

| 2 | 4 subnets listadas com AZs e CIDRs corretos |
<img width="1687" height="283" alt="4-subnets-CIDR-AZ" src="https://github.com/user-attachments/assets/eda81372-5a50-4458-a347-67ea949d85c1" />

| 3 | RDS wordpressdb com status Available e endpoint visível |
<img width="1630" height="746" alt="rds-created-endpoint-available" src="https://github.com/user-attachments/assets/782fd89e-0cb8-466b-b9ae-895baf13cb64" />

| 4 | Bucket S3 wordpress-assets criado em us-east-1 |
<img width="1013" height="316" alt="S3-bucket-created" src="https://github.com/user-attachments/assets/439b9ad2-1c71-4d10-a1d2-822b94e25cfd" />

| 5 | EC2 wordpress-server Running com 2/2 checks passed |
<img width="1667" height="426" alt="EC2-instance-created-publicIP" src="https://github.com/user-attachments/assets/ec6f48a4-6135-4bd4-aca8-3cf96722e185" />

| 6 | ALB wordpress-alb Active com target Healthy |
<img width="1497" height="694" alt="ALB-status-active" src="https://github.com/user-attachments/assets/4e2769e9-c3ec-42a9-b732-78985b18d556" />

| 7 | WordPress Projeto 18 acessível via DNS do ALB |
<img width="1911" height="851" alt="wordpress-working-ALB-DNS" src="https://github.com/user-attachments/assets/a8ac5a28-fbcf-471b-8776-eebf576960d4" />

| 8 | 2 alarmes CloudWatch criados (cpu-high e cpu-low) |
<img width="1672" height="371" alt="cloudwatch-alarms-created" src="https://github.com/user-attachments/assets/dc111e92-3f74-4319-b311-0be8e7b61c29" />

| 9 | CloudFront distribution Enabled com domain visível |
<img width="1909" height="280" alt="cloudfront-distribution-enabled-domainname" src="https://github.com/user-attachments/assets/48c4b05b-4d87-4f5c-ad3a-1ef4e8b90836" />

| 10 | WordPress Projeto 18 acessível via CloudFront com HTTPS |
<img width="1898" height="994" alt="wordpress-online-cloudfront" src="https://github.com/user-attachments/assets/b638ff29-9fa7-425c-b36e-1bf4ccc1a6a6" />


> Pasta completa de evidências ao lado

---

## 💡 Aprendizados

**Rede é a base de tudo, e errar aqui quebra tudo**
Criar VPC do zero revelou o que o console esconde quando você usa a VPC default. Cada subnet, route table e IGW tem um propósito específico. O momento em que o browser não alcançava a EC2 porque o SG-EC2 só permitia tráfego do SG-ALB, que ainda não existia, foi o aprendizado mais concreto de dependência entre recursos.

**Security Groups em camadas são a implementação real do menor privilégio**
Usar um SG como source em vez de um CIDR não é apenas mais seguro, é mais inteligente. O banco de dados só aceita conexão de recursos que carregam o SG-EC2. Nem o meu IP, nem um atacante externo, apenas a instância WordPress. Esse padrão é o que separa uma arquitetura amadora de uma profissional.

**O RDS exige preparação, não é só criar o banco**
Antes do RDS vem o DB Subnet Group. Antes do Subnet Group vêm as subnets privadas. A sequência de dependências do RDS ensinou que em cloud você projeta de baixo para cima: rede - segurança - dados - aplicação - entrega.

**CloudFront não é só cache, é a camada de segurança e HTTPS**
Sem certificado no ALB, HTTPS seria impossível sem custo adicional. O CloudFront fornece HTTPS gratuitamente e coloca uma camada de proteção entre a internet e o backend. O problema do loop de login (CachingOptimized + wp-admin) foi o aprendizado mais inesperado: cache mal configurado quebra aplicações stateful.

**Observabilidade não é opcional, é o que diferencia lab de produção**
Criar os alarmes de CPU antes de precisar deles é o raciocínio de quem opera sistemas, não de quem apenas os cria. O SNS com email confirmado fecha o ciclo: o sistema avisa antes que o usuário reclame.

**Troubleshooting é a habilidade real**
504 no CloudFront, timeout no browser, permission denied no SSH, cada erro resolvido foi mais valioso que qualquer configuração que funcionou de primeira. Um Cloud Analyst não é contratado para seguir tutoriais, mas para diagnosticar e resolver problemas em produção.

---

## 🔗 Referências

- [Documentação Amazon VPC](https://docs.aws.amazon.com/vpc/)
- [Documentação Amazon RDS](https://docs.aws.amazon.com/rds/)
- [Documentação Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Documentação Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/)
- [Documentação Amazon CloudWatch](https://docs.aws.amazon.com/cloudwatch/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [WordPress on AWS, Best Practices](https://aws.amazon.com/wordpress/)

---

<div align="center">

**Marcelo Carrara** · AWS Certified Cloud Practitioner | Cloud Analyst · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
