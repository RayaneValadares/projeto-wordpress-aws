# projeto-wordpress-aws
wordpress-aws-# 🌐 WordPress em Alta Disponibilidade na AWS  

![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20RDS%20%7C%20EFS%20%7C%20ALB-orange?logo=amazon-aws&logoColor=white)  
![Docker](https://img.shields.io/badge/Docker-Containers-blue?logo=docker&logoColor=white)  
![WordPress](https://img.shields.io/badge/WordPress-CMS-blue?logo=wordpress&logoColor=white)  
![MySQL](https://img.shields.io/badge/Database-MySQL-blue?logo=mysql&logoColor=white)  
![IaC](https://img.shields.io/badge/IaC-UserData-green)  
![Security](https://img.shields.io/badge/Security-SG%20Rules-red)  

---

## 📖 Visão Geral  

Este projeto implementa uma infraestrutura **escalável, resiliente e segura** para hospedar **WordPress** na **AWS**, utilizando **Docker** e serviços gerenciados.  

---

## 🚀 Etapas da Implementação  

### ✅ Etapa 1: Networking (VPC)  
- VPC personalizada (10.0.0.0/16)  
- 2 subnets públicas (ALB + EC2)  
- 4 subnets privadas (EFS + RDS)  
- Internet Gateway + NAT Gateways  

---

### 🔒 Etapa 2: Segurança (Security Groups)  
- **SG-ALB** → portas 80/443 (0.0.0.0/0)  
- **SG-EC2** → recebe tráfego do ALB e acessa EFS/RDS  
- **SG-RDS** → porta 3306, acessível apenas pelo SG-EC2  
- **SG-EFS** → porta 2049, acessível apenas pelo SG-EC2  

---

### 💾 Etapa 3: Armazenamento (EFS)  
- Sistema de arquivos **compartilhado** entre instâncias  
- Montado automaticamente via **User Data Script**  
- Garante persistência para uploads e temas/plugins  

---

### 🗄️ Etapa 4: Banco de Dados (RDS)  
- Engine: **MySQL**  
- Isolado em subnets privadas  
- Autenticação por senha  
- Banco inicial: `wordpress`  

---

### 🐳 Etapa 5: Template de Lançamento  
- AMI: Ubuntu 22.04 LTS  
- Script de inicialização instala Docker, monta EFS e inicia containers  
- Base para o **Auto Scaling Group**  

---

### ⚖️ Etapa 6: Load Balancing & Auto Scaling  
- **Application Load Balancer** distribui tráfego entre instâncias  
- **Target Group** com health checks na porta 80  
- **Auto Scaling Group**:  
  - Min: 1  
  - Desejado: 2  
  - Max: 3  

---

## 📊 Status do Projeto  

- ✅ Infraestrutura provisionada  
- ✅ WordPress funcional via **ALB**  
- ✅ Banco de dados persistente no **RDS**  
- ✅ Uploads compartilhados no **EFS**  
- ✅ Escalabilidade garantida pelo **ASG**  
- ✅ Alta disponibilidade em múltiplas AZs  

---

📅 **Última atualização**: 28/09/2025  
👨‍💻 **Desenvolvido por**: Rayane Vitória Valadares  
🏷️ **Versão**: 1.0.0  
