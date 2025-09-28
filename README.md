# 🌐 WordPress em Alta Disponibilidade na AWS  

![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20RDS%20%7C%20EFS%20%7C%20ALB-orange?logo=amazon-aws&logoColor=white)  ![Docker](https://img.shields.io/badge/Docker-Containers-blue?logo=docker&logoColor=white)  ![WordPress](https://img.shields.io/badge/WordPress-CMS-blue?logo=wordpress&logoColor=white)  ![MySQL](https://img.shields.io/badge/Database-MySQL-blue?logo=mysql&logoColor=white)  ![Security](https://img.shields.io/badge/Security-SG%20Rules-red)  

---

## 📖 Visão Geral  

Este projeto implementa uma infraestrutura **escalável, resiliente e segura** para hospedar **WordPress** na **AWS**, utilizando **Docker** e serviços gerenciados.  

---

## 🚀 Etapas da Implementação  

### ✅ Etapa 1: VPC  
- VPC personalizada (10.0.0.0/16)  
- 2 subnets públicas (Load Balancer + EC2)  
- 4 subnets privadas (EFS + RDS)  
- Internet Gateway + NAT Gateways  

---

### 🔒 Etapa 2: Segurança (Security Groups)  
- **SG do Load Balancer (SG-ALB)** 
  - Regras de entrada: 
    - HTTP  → Porta 80  → Origem 0.0.0.0/0
    - HTTPS → Porta 443 → Origem 0.0.0.0/0
  - Regras de saída: Todo o tráfego → 0.0.0.0/0 

- **SG da EC2 (SG-EC2)**  
  - Regras de entrada: 
    - HTTP  → Porta 80  → Origem: SG-ALB
    - HTTPS → Porta 443 → Origem: SG-ALB
  - Regras de saída: todo o tráfego → 0.0.0.0/0

- **SG do RDS (SG-RDS)**
  - Regras de entrada: 
    - MYSQL/Aurora → Porta 3306 → SG-EC2
  - Regras de saída: Todo o tráfego → 0.0.0.0/0
 
- **SG do EFS (SG-EFS)** 
  - Regras de entrada: 
    - NFS → Porta 2049 → Origem: SG-EC2
  - Regras de saída: Todo o tráfego → 0.0.0.0/0

---

### 💾 Etapa 3: Armazenamento (EFS)
Pontos importantes:  
- Sistema de arquivos **compartilhado** entre instâncias  
- Montado automaticamente via **User Data Script**  
- Garante persistência para uploads e temas/plugins  

  1- Criação do EFS
    - Dê um nome para o sistema de arquivos

    - Selecione a VPC criada anteriormente

    - Clique em Customize

    - Clique em Next

    - Crie um Mount Target para cada uma das quatro subnets privadas, associando o Security Group do EFS

    - Clique em Next nas demais etapas e finalize a criação

---

### 🗄️ Etapa 4: Banco de Dados (RDS)  
Pontos importantes:
- Engine: **MySQL**  
- Isolado em subnets privadas  
- Autenticação por senha  
- Banco inicial: `wordpress`  

  1- Criando o Banco de Dados RDS (MySQL)
    - Acesse o serviço RDS

    - Clique em Create database

    - Escolha Standard create

  2- Escolha a engine:

    - Selecione MySQL

  3- Selecione o template:

    - Dev/Test → marque Single-AZ em Availability & Durability

    - Free Tier → essa opção já vem como padrão

  4- Configurações principais:

    - DB instance identifier → defina o nome da instância

    - Master username → escolha o usuário administrador

    - Senha → pode deixar a AWS gerar ou criar a sua própria

  5- Configuração da instância:

    - Em DB instance class, escolha:
    - Burstable classes (includes t classes) → db.t3.micro

  6- Conectividade:

    - Selecione a VPC criada anteriormente

    - Garanta que o DB Subnet Group usa apenas subnets privadas

    - Em Public access, escolha No (não queremos DB exposto)

    - Associe o Security Group do RDS configurado na VPC

  7- Configurações adicionais:

    - Em Additional configuration → defina o nome do banco inicial (ex: wordpress)

    ✅ Pronto! Sua instância RDS MySQL e o banco inicial foram criados com sucesso.


---

### 🐳 Etapa 5: Template de Lançamento (Launch Template)
Pontos importantes:
- AMI: Ubuntu 22.04 LTS  
- Script de inicialização instala Docker, monta EFS e inicia containers  
- Base para o **Auto Scaling Group**  


  1- Criando o Launch Template

    - Em EC2, vá até Launch Templates

    - Clique em Create launch template

    - Defina um nome sugestivo

    - (Opcional) Adicione uma descrição

  2- Configurações principais

  - AMI → vá em Quick Start e selecione:
    - Amazon Linux 2023 kernel-6.12 AMI (arquitetura x86_64)
    - Instance type → escolha t3.micro

  - Key pair →

    - Crie um novo par de chaves

    - Dê um nome sugestivo

    - Tipo: RSA

    - Formato: .pem

    - ⚠️ Guarde o arquivo em local seguro (será usado para SSH)

  3- Rede e Segurança

    - Em Network settings, deixe as subnets em branco

    - Associe o Security Group criado para EC2
      (ou crie um novo seguindo os mesmos critérios de entrada e saída)

  4- Tags

    - Em Resource tags, adicione as tags necessárias (ex: projeto, ambiente, dono)

  5- User Data (Script de inicialização)

  No final da seção Advanced details, insira o script User Data.

  ⚠️ Lembre-se de substituir as variáveis de credenciais (usuário, senha, endpoint do RDS) no código antes de salvar.

  ```bash
  #!/bin/bash
  set -e

  # Atualiza sistema e instala dependências
  apt-get update -y
  apt-get install -y apt-transport-https ca-certificates curl software-properties-common nfs-common

  # Instala Docker
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
  apt-get update -y
  apt-get install -y docker-ce docker-ce-cli containerd.io

  # Configura Docker
  systemctl start docker
  systemctl enable docker
  usermod -a -G docker ubuntu

  # Instala Docker Compose
  DOCKER_COMPOSE_VERSION="v2.20.2"
  curl -L "https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  chmod +x /usr/local/bin/docker-compose

  # Monta EFS
  EFS_ID="<SEU_EFS_ID>"
  EFS_REGION="<SUA_REGIAO>"
  EFS_MOUNT_POINT="/mnt/efs/wordpress"
  mkdir -p ${EFS_MOUNT_POINT}
  mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport ${EFS_ID}.efs.${EFS_REGION}.amazonaws.com:/ ${EFS_MOUNT_POINT}
  echo "${EFS_ID}.efs.${EFS_REGION}.amazonaws.com:/ ${EFS_MOUNT_POINT} nfs4 defaults,_netdev 0 0" >> /etc/fstab

  # Cria configuração Docker Compose
  cat <<EOF > /home/ubuntu/docker-compose.yml
  version: '3.8'
  services:
    wordpress:
      image: wordpress:latest
      container_name: wordpress
      restart: always
      ports:
        - "80:80"
      environment:
        WORDPRESS_DB_

  ```

✅ Pronto! Seu Launch Template foi criado e já contém o script de automação (User Data), garantindo que cada instância nova seja provisionada com Docker + WordPress pronto para rodar.

---

### ⚖️ Etapa 6: Load Balancing & Auto Scaling  
Pontos importantes:
- **Application Load Balancer** distribui tráfego entre instâncias  
- **Target Group** com health checks na porta 80  
- **Auto Scaling Group**:  
  - Min: 1  
  - Desejado: 2  
  - Max: 3  

  1- Criação do Target Group

    - No menu EC2, vá até Target Groups

    - Clique em Create target group

    - Defina:

      - Target type: Instances

      - Protocol: HTTP

      - Port: 80

      - VPC: selecione a mesma usada no projeto

    - Finalize a criação


  2- Criação do Application Load Balancer (ALB)

  - No menu EC2, vá até Load Balancers

  - Clique em Create Load Balancer → Application Load Balancer

  - Defina:

    - Name: sugestivo (ex: wordpress-alb)

    - Scheme: Internet-facing

    - VPC: selecione a criada anteriormente

    - Subnets: escolha as duas públicas

    - Security Group: associe o SG do ALB (porta 80 liberada)

    - Listener: HTTP na porta 80, direcionando para o Target Group criado

  3- Criação do Auto Scaling Group (ASG)

  - No menu EC2, vá até Auto Scaling Groups

  - Clique em Create Auto Scaling group

  - Defina:

    - Name: sugestivo (ex: wordpress-asg)

    - Launch Template: selecione o que você criou com o User Data

    - VPC/Subnets: escolha a mesma usada anteriormente (subnets públicas)

    - Load Balancer: associe ao ALB criado, selecionando o Target Group

    - Capacidade: Minimum: 1, Desired: 2, Maximum: 3

  - Finalize a criação

  ✅ Pronto! O ALB já estará ativo, distribuindo tráfego HTTP entre as instâncias do Auto Scaling Group.
---

## 📊 Status do Projeto  

- ✅ Infraestrutura provisionada  
- ✅ WordPress funcional via **ALB**  
- ✅ Banco de dados persistente no **RDS**  
- ✅ Uploads compartilhados no **EFS**  
- ✅ Escalabilidade garantida pelo **ASG**  
- ✅ Alta disponibilidade em múltiplas AZs  

---

---
✨ Projeto em constante evolução   
🔧 Mantido por **Rayane Vitória Valadares**  
📌 Versão atual: **1.1.1**  

---
![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat-square)
![Version](https://img.shields.io/badge/version-1.1.1-blue?style=flat-square)
![Maintainer](https://img.shields.io/badge/maintainer-Rayane%20Vitória%20Valadares-purple?style=flat-square)

