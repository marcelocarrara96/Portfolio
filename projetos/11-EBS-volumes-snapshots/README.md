# 💾 Projeto 11 - Armazenamento Persistente com Amazon EBS: Volumes e Snapshots

![AWS](https://img.shields.io/badge/AWS-EBS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![S3](https://img.shields.io/badge/AWS-S3-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![Status](https://img.shields.io/badge/Status-Concluído-1A9C3E?style=for-the-badge)

---

## 🎯 Objetivo

Criar e gerenciar volumes Amazon EBS, anexando-os a uma instância EC2, formatando e montando o sistema de arquivos via terminal Linux, criando snapshots para backup e restaurando dados a partir de um snapshot em um novo volume.

---

## 🛠️ Serviços utilizados

| Serviço | Função |
|---|---|
| Amazon EBS | Criação, anexação e gerenciamento de volumes de armazenamento persistente |
| Amazon EC2 | Instância Linux que recebe e utiliza os volumes EBS |
| Amazon S3 | Armazenamento durável dos snapshots EBS nos bastidores |
| EC2 Instance Connect | Acesso ao terminal da instância sem chave SSH local |

---

## 🏗️ Arquitetura da solução

```
┌──────────────────────────────────────────────────────┐
│                  Availability Zone                   │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │              EC2 Instance — Lab                │  │
│  │                                                │  │
│  │  /dev/nvme0n1p1  →  /          (8 GiB - root) │  │
│  │  /dev/sdb        →  /mnt/data-store  (1 GiB)  │  │
│  │  /dev/sdc        →  /mnt/data-store2 (restore) │  │
│  └────────────────────────────────────────────────┘  │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐   │
│  │  My Volume   │  │  Snapshot    │  │ Restored  │   │
│  │  1 GiB gp2   │  │  My Snapshot │  │ Volume    │   │
│  │  /dev/sdb    │──│  (backup)    │──│ /dev/sdc  │   │
│  └──────────────┘  └──────────────┘  └───────────┘   │
│                           │                          │
└───────────────────────────┼──────────────────────────┘
                            │ Armazenado em
                            ▼
                    ┌───────────────┐
                    │  Amazon S3    │
                    │  (snapshots)  │
                    └───────────────┘
```

---

## 📋 Etapas de implementação

1. Criação do volume EBS **My Volume** (1 GiB, gp2) na mesma AZ da instância EC2
2. Anexação do volume à instância Lab via console (`/dev/sdb`)
3. Conexão à instância via **EC2 Instance Connect**
4. Formatação do volume como **ext3** e montagem em `/mnt/data-store`
5. Configuração da montagem automática via `/etc/fstab`
6. Criação de arquivo de teste no volume montado
7. Criação do snapshot **My Snapshot** a partir do volume
8. Exclusão do arquivo de teste para simular perda de dados
9. Restauração do snapshot em um novo volume **Restored Volume**
10. Anexação e montagem do volume restaurado em `/mnt/data-store2`
11. Validação da recuperação do arquivo `file.txt`

---

## ⚙️ Comandos executados no terminal Linux

```bash
# Verificar armazenamento disponível antes da anexação
df -h

# Formatar o novo volume como ext3
sudo mkfs -t ext3 /dev/sdb

# Criar ponto de montagem
sudo mkdir /mnt/data-store

# Montar o volume
sudo mount /dev/sdb /mnt/data-store

# Persistir montagem após reboot (adicionar ao fstab)
echo "/dev/sdb   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab

# Verificar configuração do fstab
cat /etc/fstab

# Confirmar montagem com espaço disponível
df -h

# Criar arquivo de teste no volume
sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"

# Confirmar escrita
cat /mnt/data-store/file.txt

# Simular perda de dados (deletar arquivo)
sudo rm /mnt/data-store/file.txt

# Confirmar exclusão
ls /mnt/data-store/file.txt

# --- Após restauração do snapshot ---

# Criar segundo ponto de montagem
sudo mkdir /mnt/data-store2

# Montar volume restaurado
sudo mount /dev/sdc /mnt/data-store2

# Confirmar recuperação do arquivo
ls /mnt/data-store2/file.txt
```

---

## 🗂️ Volumes criados

| Nome | Tipo | Tamanho | Device | Mount Point | Origem |
|---|---|---|---|---|---|
| My Volume | gp2 | 1 GiB | `/dev/sdb` | `/mnt/data-store` | Novo |
| Restored Volume | gp2 | 1 GiB | `/dev/sdc` | `/mnt/data-store2` | Restaurado do snapshot |

---

## 📸 Evidências



---

## 💡 Aprendizados

- ✅ Como criar e anexar um **volume EBS** a uma instância EC2 em execução
- ✅ Formatação de volume com **ext3** e montagem em diretório Linux
- ✅ Persistência da montagem via **/etc/fstab** para sobreviver a reboots
- ✅ Criação de **snapshots EBS** como estratégia de backup incremental
- ✅ Como os snapshots são armazenados de forma durável no **Amazon S3** nos bastidores
- ✅ Restauração de dados a partir de um snapshot em um **novo volume EBS**
- ✅ Diferença entre volume raiz e volumes adicionais em uma instância EC2

---

## 🔗 Referências

- [Documentação Amazon EBS](https://docs.aws.amazon.com/ebs/)
- [EBS Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html)
- [Montar volumes EBS no Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html)

---

<div align="center">

**Marcelo Carrara** · Transitioning into Cloud | Cloud Analyst Jr & NOC Support · Paraná, Brazil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>
