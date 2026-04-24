# Projeto 04: Infraestrutura Unificada e Portfólio Estático na AWS

## 🌐 Visão Geral
Este projeto consiste na implantação de uma infraestrutura em nuvem completa e automatizada utilizando **AWS CloudFormation**. A arquitetura abrange desde a hospedagem de um site estático de alta disponibilidade até uma camada de backend com processamento, banco de dados e monitoramento.

O objetivo principal foi consolidar conhecimentos em **Infraestrutura como Código (IaC)**, criando um ambiente seguro, escalável e seguindo as melhores práticas da AWS.

## 🏗️ Arquitetura do Projeto
A infraestrutura foi dividida em 5 camadas principais:

1.  **Camada de Armazenamento (S3):** Hospedagem de portfólio estático com configuração de site e políticas de acesso público.
2.  **Camada de Rede (VPC):** Criação de uma rede isolada com sub-redes públicas e privadas distribuídas em múltiplas Zonas de Disponibilidade (AZs).
3.  **Camada de Computação (EC2):** Servidor Linux (T3 Micro) provisionado em sub-rede pública com script de automação (User Data).
4.  **Camada de Dados (RDS):** Banco de dados MySQL gerenciado, isolado em sub-redes privadas e acessível apenas pela EC2.
5.  **Monitoramento (CloudWatch):** Configuração de alarmes de utilização de CPU para garantir a observabilidade da instância.

## 🛠️ Tecnologias Utilizadas
* **AWS CloudFormation:** Automação total da infraestrutura.
* **Amazon S3:** Hospedagem de site estático.
* **Amazon EC2:** Servidor de aplicação.
* **Amazon RDS (MySQL):** Banco de dados relacional.
* **Amazon VPC:** Isolamento de rede e conectividade.
* **AWS CloudWatch:** Monitoramento e alertas.

## 🚀 Guia Detalhado de Execução

### 1. Preparação do Ambiente
* **Key Pair:** Certifique-se de ter um par de chaves (.pem ou .ppk) criado no console da EC2 na região onde pretende rodar o projeto.
* **Arquivos do Site:** Tenha prontos os arquivos `index.html`, `style.css` e sua imagem de perfil em uma pasta local.

### 2. Deploy da Infraestrutura (CloudFormation)
1.  Acesse o console da **AWS** e vá para o serviço **CloudFormation**.
2.  Clique em **Create stack** > **With new resources (standard)**.
3.  Em *Specify template*, selecione **Upload a template file** e escolha o arquivo `s3fulltest.yaml`.
4.  No passo **Specify stack details**:
    * **Stack name:** `ProjetoUnificadoMarcelo`.
    * **KeyPairName:** Selecione sua chave SSH.
    * **NomeDoBucket:** Digite um nome exclusivo (ex: `portfolio-marcelo-carrara-aws`).
5.  Avance as telas mantendo as configurações padrão e clique em **Submit**.
6.  Aguarde até que o status mude para `CREATE_COMPLETE`.

### 3. Upload do Portfólio para o S3
1.  Vá para a aba **Outputs** do seu Stack no CloudFormation e copie o link gerado em `URLPortfolio`.
2.  Acesse o serviço **S3** e entre no bucket que você acabou de criar.
3.  Clique em **Upload** e envie os arquivos `index.html`, `style.css` e sua foto.
4.  Certifique-se de que a política do bucket permite o acesso público conforme configurado no template.
   


## 📄 Conclusão
Este projeto demonstra a capacidade de provisionar uma stack tecnológica completa em poucos minutos através de IaC. É um passo fundamental na minha transição de carreira para **Cloud Analyst Jr**.


📸 Evidências do Projeto
Você pode conferir as capturas de tela das etapas de criação e dos recursos ativos na pasta ao lado:

Visualizar Evidências: [Clique aqui para abrir a pasta de evidências](./evidencias/)


---
**Marcelo Carrara** *Aspiring Solutions Architect | AWS re/Start Cloud Practicioner*
