# 🏷️ Projeto 13 - Gerenciamento de Recursos por Tags com AWS CLI

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![CLI](https://img.shields.io/badge/AWS-CLI-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Bash-232F3E?style=for-the-badge&logo=linux&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Utilizar a AWS CLI para localizar, filtrar e gerenciar instâncias EC2 por meio de tags, aplicando queries JMESPath para refinar resultados, automatizando a atualização de tags em lote via script Bash e controlando o ciclo de vida de instâncias de desenvolvimento com um script PHP (Stopinator).

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon EC2 | Instâncias tagueadas com Project, Environment e Version |
| AWS CLI | Filtragem, consulta e atualização de recursos por tags |
| AWS SDK for PHP | Base do script Stopinator para parar/iniciar instâncias por tag |
| Linux Bash | Script de atualização de tags em lote (`change-resource-tags.sh`) |

---

## 🏗️ Arquitetura da solução

```
┌──────────────────────────────────────────────────────────────┐
│                        AWS Account                           │
│                                                              │
│   Instâncias EC2 tagueadas (Projeto ERPSystem):              │
│                                                              │
│   ┌─────────────────────┐   ┌─────────────────────┐         │
│   │  EC2 — production   │   │  EC2 — development  │         │
│   │  Project=ERPSystem  │   │  Project=ERPSystem  │         │
│   │  Environment=prod   │   │  Environment=dev    │         │
│   │  Version=1.0        │   │  Version=1.0 → 1.1  │         │
│   └─────────────────────┘   └──────────┬──────────┘         │
│                                        │                    │
│                     ┌──────────────────┘                    │
│                     ▼                                        │
│   ┌──────────────────────────────────────────────────────┐   │
│   │               Command Host EC2                       │   │
│   │                                                      │   │
│   │  AWS CLI                                             │   │
│   │  ├── describe-instances (filter + --query JMESPath)  │   │
│   │  ├── create-tags (atualização em lote)               │   │
│   │  │                                                   │   │
│   │  change-resource-tags.sh                             │   │
│   │  └── atualiza Version=1.1 em todas as instâncias dev │   │
│   │                                                      │   │
│   │  stopinator.php (AWS SDK PHP)                        │   │
│   │  ├── Para instâncias dev do ERPSystem                │   │
│   │  └── Reinicia instâncias dev do ERPSystem            │   │
│   └──────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

---

## 📋 Etapas de implementação

1. Conexão ao **Command Host** via SSH
2. Filtragem de instâncias do projeto `ERPSystem` com `aws ec2 describe-instances`
3. Refinamento progressivo dos resultados com **JMESPath** (`--query`)
4. Inclusão das tags `Project`, `Environment` e `Version` no output da query
5. Filtro combinado por `Project=ERPSystem` + `Environment=development`
6. Execução do script `change-resource-tags.sh` para atualizar `Version=1.1` em lote
7. Validação da alteração confirmando que instâncias de produção não foram afetadas
8. Execução do **Stopinator** para parar instâncias de desenvolvimento
9. Validação no console EC2 das instâncias paradas
10. Reinício das instâncias com o Stopinator usando o flag `-s`

---

## ⚙️ Comandos AWS CLI executados

```bash
# Listar todas as instâncias do projeto ERPSystem
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem"

# Filtrar retornando apenas IDs
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].InstanceId'

# Incluir ID e Availability Zone
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].{ID:InstanceId,AZ:Placement.AvailabilityZone}'

# Incluir valores das tags Project, Environment e Version
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
  --query 'Reservations[*].Instances[*].{
    ID:InstanceId,
    AZ:Placement.AvailabilityZone,
    Project:Tags[?Key==`Project`] | [0].Value,
    Environment:Tags[?Key==`Environment`] | [0].Value,
    Version:Tags[?Key==`Version`] | [0].Value}'

# Filtrar apenas instâncias de desenvolvimento
aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
           "Name=tag:Environment,Values=development" \
  --query 'Reservations[*].Instances[*].{
    ID:InstanceId,
    AZ:Placement.AvailabilityZone,
    Project:Tags[?Key==`Project`] | [0].Value,
    Environment:Tags[?Key==`Environment`] | [0].Value,
    Version:Tags[?Key==`Version`] | [0].Value}'
```

---

## 📜 Scripts utilizados

**change-resource-tags.sh** — Atualiza a tag Version em lote nas instâncias de desenvolvimento:

```bash
#!/bin/bash

ids=$(aws ec2 describe-instances \
  --filter "Name=tag:Project,Values=ERPSystem" \
           "Name=tag:Environment,Values=development" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

aws ec2 create-tags --resources $ids --tags 'Key=Version,Value=1.1'
```

**stopinator.php** — Para ou inicia instâncias por tag usando AWS SDK para PHP:

```bash
# Parar instâncias de desenvolvimento do ERPSystem
./stopinator.php -t"Project=ERPSystem;Environment=development"

# Reiniciar as mesmas instâncias
./stopinator.php -t"Project=ERPSystem;Environment=development" -s
```

---

## 🔍 JMESPath — Sintaxe utilizada

| Objetivo | Sintaxe |
|---|---|
| Iterar todas as instâncias | `Reservations[*].Instances[*]` |
| Selecionar múltiplos campos com alias | `{Alias1:Campo1, Alias2:Campo2}` |
| Ler valor de uma tag específica | `Tags[?Key==\`Project\`] \| [0].Value` |
| Retornar como texto simples | `--output text` |

---

## 📸 Evidências

Instances per tag:
<img width="494" height="556" alt="instances-per-tags" src="https://github.com/user-attachments/assets/d3481f76-65c9-4140-aa58-929de7a31568" />

Stopping instances:
<img width="817" height="447" alt="stopping-instances" src="https://github.com/user-attachments/assets/5d4f759d-dd5c-4f7c-95a1-2ac10919dbd5" />

Instances stopped:
<img width="900" height="87" alt="instances-stopped" src="https://github.com/user-attachments/assets/d7f4bfa1-b9e8-4229-9731-64e50e7a571c" />

Stopping instances without tags:
<img width="891" height="287" alt="stopping-instances-without-tags" src="https://github.com/user-attachments/assets/f62e9481-9aed-444e-b2a3-1c9afa3212eb" />

---

## 💡 Aprendizados

- ✅ Como usar `--filter` na AWS CLI para localizar recursos por **tags**
- ✅ Uso de **JMESPath** com `--query` para refinar e formatar o output da CLI
- ✅ Atualização de tags em **lote** com script Bash + `aws ec2 create-tags`
- ✅ Automação do ciclo de vida de instâncias com o script **Stopinator** (PHP SDK)
- ✅ Diferença entre `--output json` e `--output text` e quando usar cada um
- ✅ Importância das **tags** como base para automação, governança e controle de custos

---

## 🔗 Referências

- [AWS CLI — describe-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html)
- [JMESPath Query Language](https://jmespath.org/)
- [AWS CLI — output formats](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-output-format.html)
- [AWS SDK for PHP](https://docs.aws.amazon.com/sdk-for-php/)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
