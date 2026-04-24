# Projeto 04: Infraestrutura Unificada e Portfólio Estático na AWS

## 🌐 Visão Geral
Este projeto consiste na implantação de uma infraestrutura em nuvem completa e automatizada utilizando **AWS CloudFormation**. A arquitetura abrange desde a hospedagem de um site estático de alta disponibilidade até uma camada de backend com processamento, banco de dados e monitoramento.

O objetivo principal foi consolidar conhecimentos em **Infraestrutura como Código (IaC)**, criando um ambiente seguro, escalável e seguindo as melhores práticas do *AWS Well-Architected Framework*.

## 🏗️ Arquitetura do Projeto

A infraestrutura foi dividida em 5 camadas principais:

1.  **Camada de Armazenamento (S3):** Hospedagem de portfólio estático com configuração de site e políticas de acesso público.
2.  **Camada de Rede (VPC):** Criação de uma rede isolada com sub-redes públicas e privadas distribuídas em múltiplas Zonas de Disponibilidade (AZs) para garantir resiliência.
3.  **Camada de Computação (EC2):** Servidor Linux (T3 Micro) provisionado em sub-rede pública com script de automação (User Data) para instalação do Apache.
4.  **Camada de Dados (RDS):** Banco de dados MySQL gerenciado, isolado em sub-redes privadas e acessível apenas pela camada de computação através de Security Groups.
5.  **Monitoramento (CloudWatch):** Configuração de alarmes de utilização de CPU para garantir a observabilidade da instância EC2.

## 🛠️ Tecnologias Utilizadas
-   **AWS CloudFormation:** Automação de toda a infraestrutura.
-   **Amazon S3:** Hospedagem de site estático.
-   **Amazon EC2:** Servidor de aplicação.
-   **Amazon RDS (MySQL):** Banco de dados relacional.
-   **Amazon VPC:** Isolamento de rede, Subnets, Internet Gateway e Tabelas de Roteamento.
-   **AWS CloudWatch:** Monitoramento e alertas.

## 🚀 Como Executar o Projeto

1.  **Pré-requisitos:**
    * Possuir uma conta ativa na AWS.
    * Ter um **Key Pair** criado na região onde o stack será executado.

2.  **Passo a Passo:**
    * Faça o download do arquivo `s3fulltest.yaml`.
    * No console da AWS, acesse o serviço **CloudFormation**.
    * Clique em **Create Stack** > **With new resources**.
    * Faça o upload do arquivo YAML.
    * Preencha os parâmetros:
        * `KeyPairName`: Selecione sua chave SSH.
        * `NomeDoBucket`: Defina um nome único para o seu bucket de portfólio.
    * Avance e clique em **Submit**.

3.  **Acessando os Resultados:**
    * Após o status `CREATE_COMPLETE`, verifique a aba **Outputs** para obter as URLs do site no S3 e o endpoint da EC2.

## 📄 Conclusão
Este laboratório demonstra a capacidade de provisionar uma stack tecnológica completa em poucos minutos, garantindo que a infraestrutura seja replicável e livre de erros manuais. É um passo fundamental na minha transição para a carreira de **Solutions Architect**. Todas as imagens registradas do projeto estão na pasta "Evidencias".

---
**Marcelo Carrara** *Aspiring Solutions Architect | AWS re/Start Graduate*
