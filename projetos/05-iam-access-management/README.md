<img width="1306" height="436" alt="password-policy" src="https://github.com/user-attachments/assets/cf957f39-4a99-4988-b4e7-f83283976309" />
<img width="1178" height="353" alt="users-iam" src="https://github.com/user-attachments/assets/e6870631-7e84-4df6-99f0-ecb15eff3a25" />
<img width="1661" height="412" alt="add-user-groups" src="https://github.com/user-attachments/assets/b5813cc2-514a-4539-91bc-15913794c571" />
# 🔐 Projeto 05 — Gerenciamento de Identidade e Acesso com AWS IAM

![AWS](https://img.shields.io/badge/AWS-IAM-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Security](https://img.shields.io/badge/Segurança-Políticas_de_Acesso-1A9C3E?style=for-the-badge&logo=springsecurity&logoColor=white)
![Linux](https://img.shields.io/badge/IAM-Usuários_e_Grupos-232F3E?style=for-the-badge&logo=amazoniam&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Configurar o gerenciamento de identidade e acesso na AWS utilizando o IAM, aplicando uma política de senha personalizada para a conta, explorando usuários e grupos com permissões distintas e atribuindo os usuários aos grupos corretos de acordo com suas funções na empresa.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| AWS IAM | Gerenciamento centralizado de identidade e acesso |
| IAM Users | Identidades individuais para acesso à conta AWS |
| IAM User Groups | Agrupamento de usuários com permissões compartilhadas |
| IAM Managed Policies | Políticas pré-construídas pela AWS, reutilizáveis entre grupos |
| IAM Inline Policies | Políticas aplicadas diretamente a um único usuário ou grupo |
| Password Policy | Requisitos de complexidade e expiração de senhas da conta |

---

## 🏗️ Arquitetura da solução

```
┌──────────────────────────────────────────────────────────────┐
│                        AWS Account                           │
│                                                              │
│  🔒 Password Policy (aplicada a toda a conta)               │
│  ├── Mínimo 10 caracteres                                    │
│  ├── Letras maiúsculas, minúsculas, números e símbolos       │
│  ├── Expiração em 90 dias                                    │
│  └── Bloqueio das últimas 5 senhas                           │
│                                                              │
│  ┌──────────────┐  ┌───────────────┐  ┌───────────────┐     │
│  │  S3-Support  │  │  EC2-Support  │  │   EC2-Admin   │     │
│  │              │  │               │  │               │     │
│  │  📋 Managed  │  │  📋 Managed   │  │  📝 Inline    │     │
│  │  S3          │  │  EC2          │  │  EC2 Describe │     │
│  │  ReadOnly    │  │  ReadOnly     │  │  Start / Stop │     │
│  │              │  │               │  │               │     │
│  │  👤 user-1   │  │  👤 user-2    │  │  👤 user-3    │     │
│  └──────────────┘  └───────────────┘  └───────────────┘     │
└──────────────────────────────────────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Criação de uma **política de senha personalizada** para a conta AWS
2. Exploração dos usuários `user-1`, `user-2` e `user-3` pré-criados no IAM
3. Análise dos grupos `S3-Support`, `EC2-Support` e `EC2-Admin` e suas políticas
4. Compreensão da diferença entre **Managed Policies** e **Inline Policies**
5. Atribuição dos usuários aos grupos corretos conforme o cenário de negócio
6. Validação de que cada grupo ficou com exatamente **1 usuário** associado

---

## 🔑 Política de Senha Configurada

| Requisito | Configuração |
|---|---|
| Tamanho mínimo | 10 caracteres |
| Letras maiúsculas | ✅ Obrigatório |
| Letras minúsculas | ✅ Obrigatório |
| Números | ✅ Obrigatório |
| Caracteres especiais | ✅ Obrigatório |
| Expiração de senha | 90 dias |
| Histórico de senhas | Últimas 5 senhas bloqueadas |
| Reset obrigatório por admin | ❌ Não habilitado |

---

## 👥 Cenário de Negócio — Atribuição de Usuários

| Usuário | Grupo | Tipo de Política | Permissões |
|---|---|---|---|
| user-1 | S3-Support | Managed Policy | Leitura no Amazon S3 |
| user-2 | EC2-Support | Managed Policy | Leitura no Amazon EC2, ELB, CloudWatch e Auto Scaling |
| user-3 | EC2-Admin | Inline Policy | Visualizar, iniciar e parar instâncias EC2 |

---

## 🔍 Estrutura de uma IAM Policy

Todo statement de uma política IAM é composto por três elementos principais:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::meu-bucket/*"
}
```

| Elemento | Descrição |
|---|---|
| `Effect` | Define se a permissão é `Allow` (permitir) ou `Deny` (negar) |
| `Action` | Especifica a chamada de API permitida ou negada (ex: `ec2:StartInstances`) |
| `Resource` | Escopo dos recursos cobertos pela regra (ex: bucket S3, instância EC2 ou `*` para todos) |

---

## 📸 Evidências

> _Adicione aqui os prints das etapas concluídas_

---

## 💡 Aprendizados

- ✅ Como criar e aplicar uma **política de senha** personalizada no nível da conta AWS
- ✅ Diferença entre **Managed Policies** (reutilizáveis) e **Inline Policies** (específicas)
- ✅ Como **grupos IAM** centralizam a gestão de permissões de forma eficiente
- ✅ Estrutura de um **statement IAM**: `Effect`, `Action` e `Resource`
- ✅ Princípio do **menor privilégio**: cada usuário recebe apenas o acesso necessário para sua função
- ✅ Como atribuir usuários a grupos e validar as permissões herdadas

---

## 🔗 Referências

- [Documentação AWS IAM](https://docs.aws.amazon.com/iam/)
- [Boas práticas de segurança IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [Políticas gerenciadas pela AWS](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html)

---

*Projeto realizado no AWS Sandbox — Abril 2026*
