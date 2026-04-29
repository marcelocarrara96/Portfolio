# Projeto 03 - Provisionamento de Infraestrutura na AWS via CLI

#### OBJETIVO
Provisionar uma infraestrutura básica na AWS utilizando exclusivamente a linha de comando, compreendendo a relação e dependência entre os recursos de rede e computação dentro da nuvem.

_O laboratório foi executado no ambiente sandbox do programa AWS re/Start, realizando a criação dos recursos manualmente para compreender cada etapa do processo de configuração._

#### SERVIÇOS UTILIZADOS
* Amazon VPC
* Amazon EC2
* Internet Gateway
* Route Tables
* Security Groups
* AWS CLI
* PowerShell
* PuTTY

#### IMPLEMENTAÇÃO
Toda a infraestrutura foi provisionada utilizando AWS CLI no PowerShell, sem utilização da console da AWS.

Etapas realizadas:
1. Criação de uma VPC dedicada, sem utilizar a VPC default.
2. Criação de uma subnet pública dentro da VPC.
3. Criação de um Internet Gateway e associação à VPC.
4. Criação de uma Route Table.
5. Criação de uma rota para acesso à internet (0.0.0.0/0) apontando para o Internet Gateway.
6. Associação da Route Table à subnet pública, permitindo tráfego externo.
7. Criação de Security Groups com regra permitindo acesso SSH (porta 22).
8. Lançamento de uma instância Amazon EC2 utilizando Amazon Linux.
9. Associação da instância à subnet e ao Security Group configurado.
10. Execução de comando via CLI para obter o endereço IP público da instância.
11. Validação da conectividade com acesso remoto via SSH.

##### ACESSO E VALIDAÇÃO

Após a criação da infraestrutura, foi realizada a conexão remota com a instância EC2 para validar o funcionamento da rede e das regras de segurança.

Formas de acesso utilizadas:

* Conexão SSH via PowerShell utilizando chave .pem (OpenSSH)
* Conexão SSH via PuTTY utilizando chave .ppk

Esses testes confirmaram a correta configuração da infraestrutura e a comunicação entre os recursos criados.

#### ARQUITETURA DA SOLUÇÃO

_A infraestrutura foi projetada utilizando uma VPC dedicada com subnet pública conectada à internet por meio de um Internet Gateway.
Uma instância EC2 com Amazon Linux foi implantada dentro da subnet e protegida por um Security Group permitindo acesso SSH.
O acesso administrativo foi realizado via SSH utilizando PowerShell (OpenSSH) e PuTTY._

#### EVIDÊNCIA
Acessar pasta de evidencias para screenshots do projeto.


#### COMANDOS UTILIZADOS
```bash
# Criar VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Criar subnet pública
aws ec2 create-subnet \
--vpc-id vpc-08cfaca6d74f943e2 \
--cidr-block 10.0.1.0/24

# Criar Internet Gateway
aws ec2 create-internet-gateway

# Associar Internet Gateway à VPC
aws ec2 attach-internet-gateway \
--internet-gateway-id igw-0a5deee9519b89ae8 \
--vpc-id vpc-08cfaca6d74f943e2

# Criar tabela de roteamento
aws ec2 create-route-table \
--vpc-id vpc-08cfaca6d74f943e2

# Criar rota para acesso à internet
aws ec2 create-route \
--route-table-id rtb-04fb10c03f3271a4c \
--destination-cidr-block 0.0.0.0/0 \
--gateway-id igw-0a5deee9519b89ae8

# Associar tabela de rotas à subnet
aws ec2 associate-route-table \
--route-table-id rtb-04fb10c03f3271a4c \
--subnet-id subnet-091fca42df094adb1

# Criar Security Group
aws ec2 create-security-group \
--group-name meusegurancalab \
--description "acesso ssh" \
--vpc-id vpc-08cfaca6d74f943e2

# Criar regra de entrada para SSH
aws ec2 authorize-security-group-ingress \
--group-id sg-051e70cb2f606cfd9 \
--protocol tcp \
--port 22 \
--cidr 0.0.0.0/0

# Consultar informações da instância
aws ec2 describe-instances \
--query "Reservations[*].Instances[*].{ID:InstanceId,Estado:State.Name,IP_Publico:PublicIpAddress}" \
--output table
```

#### APRENDIZADO

_Este laboratório permitiu compreender na prática o provisionamento de infraestrutura utilizando AWS CLI, além de reforçar o entendimento sobre a dependência entre os recursos de rede na AWS, como VPC, Internet Gateway e tabelas de roteamento.
Também possibilitou praticar a configuração de acesso remoto seguro via SSH em instâncias Linux, utilizando tanto o OpenSSH no PowerShell com chave .pem quanto o PuTTY com chave .ppk, validando a conectividade da infraestrutura criada._

---

<div align="center">

**Marcelo Carrara** · Aspiring Solutions Architect · Paraná, Brasil

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/marcelo-carrara-tech/)
[![GitHub](https://img.shields.io/badge/GitHub-FF9900?style=flat&logo=github&logoColor=white)](https://github.com/marcelocarrara96)

</div>

