# aws-ec2-static-site
Documentation for deploying a static website on AWS EC2.

# DevOps Project – Static Website Deployment on AWS EC2

![Ubuntu](https://img.shields.io/badge/Ubuntu-222222?style=for-the-badge&logo=ubuntu&logoColor=E95420)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=FF9900)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Certbot](https://img.shields.io/badge/Certbot-183A61?style=for-the-badge&logo=let's-encrypt&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-000000?style=for-the-badge&logo=openssh&logoColor=white)


---

## 🇧🇷 Deploy de Site Estático em AWS EC2

Este repositório documenta o processo completo de provisionamento de um servidor de produção na AWS para hospedar um site estático ([https://devathylla.site](https://devathylla.site)) usando Ubuntu e Nginx, com domínio customizado e HTTPS automatizado pelo Certbot. O objetivo é aplicar práticas DevOps avançadas, incluindo **Infraestrutura como Código (IaC)**, **hardening do servidor** e **automação de pipeline**.  

O ambiente final consiste em:

- Instância EC2 Ubuntu 22.04 LTS (AMI oficial)  
- Nginx servindo conteúdo estático  
- Registro DNS apontando para Elastic IP fixo  
- Certificado Let’s Encrypt via Certbot  

O site em produção pode ser acessado em [https://devathylla.site](https://devathylla.site)

### Stack Usada

- **Infraestrutura:** AWS EC2 (t2.micro ou superior)  
- **Webserver:** Nginx configurado manualmente  
- **Domínio/DNS:** devathylla.site, gerenciado via GoDaddy  
- **Certificado SSL:** Let’s Encrypt + Certbot  
- **Firewall e Segurança:** UFW + Security Groups AWS  
- **Extras:** Git, futura automação com Ansible/Terraform  

### Estrutura do Ambiente

- **VPC e Rede:** Sub-rede pública com Internet Gateway  
- **EC2 e Elastic IP:** IP fixo, portas mínimas abertas (22, 80, 443)  
- **Configuração Nginx:** `/etc/nginx/sites-available/devathylla.conf` → `/var/www/html/devathylla`  
- **DNS:** Registro A apontando para Elastic IP  
- **Segurança:** SSH restrito, UFW ativo  

### Etapas de Provisionamento

1. Provisionar EC2 Ubuntu 22.04 LTS  
2. Associar Elastic IP  
3. Conectar via SSH (`ssh -i chave.pem ubuntu@<Elastic-IP>`)  
4. Atualizar sistema (`sudo apt update && sudo apt upgrade -y`)  
5. Instalar Nginx (`sudo apt install nginx -y`)  
6. Configurar Nginx (`/etc/nginx/sites-available/devathylla.conf`)  
7. Criar diretório `/var/www/html/devathylla` e ajustar permissões  
8. Remover página padrão (`sudo rm /var/www/html/index.nginx-debian.html`)  
9. Testar configuração (`sudo nginx -t && sudo systemctl reload nginx`)  
10. Copiar site estático (`scp` ou `git clone`)  
11. Configurar DNS e verificar propagação  

### Certificado HTTPS com Certbot

1. `sudo apt install certbot python3-certbot-nginx -y`  
2. `sudo certbot --nginx -d devathylla.site -d www.devathylla.site`  
3. Aceitar termos e redirecionar HTTP → HTTPS  
4. Certbot recarrega Nginx automaticamente  
5. Renovação automática (`sudo certbot renew --dry-run`)  

### Checklist de Segurança Aplicada

- Atualizações automáticas  
- SSH com chave, root desabilitado  
- UFW ativo  
- Security Groups AWS restritivos  
- Logs e monitoramento  
- Hardening TLS/Nginx  
- Backups periódicos  

### Possibilidades de Automação Futura

- Terraform (IaC completo)  
- Ansible (configuração automática)  
- Docker + Nginx (containerização)  
- CI/CD com GitHub Actions  
- Infraestrutura imutável com Packer  
- Escalabilidade e alta disponibilidade  

---

## 🇺🇸 Static Website Deployment on AWS EC2

This repository documents the full provisioning process of a production server on AWS to host a static website ([https://devathylla.site](https://devathylla.site)) using Ubuntu and Nginx, with custom domain and HTTPS automated via Certbot. The goal is to apply advanced DevOps practices: **Infrastructure as Code (IaC)**, **server hardening**, and **pipeline automation**.  

Production environment:

- EC2 Ubuntu 22.04 LTS (official AMI)  
- Nginx serving static content  
- DNS record pointing to fixed Elastic IP  
- Let’s Encrypt certificate via Certbot  

The site is accessible at [https://devathylla.site](https://devathylla.site).

### Stack Used

- **Infrastructure:** AWS EC2 (t2.micro or higher)  
- **Webserver:** Nginx manually configured  
- **Domain/DNS:** devathylla.site managed via GoDaddy  
- **SSL Certificate:** Let’s Encrypt + Certbot  
- **Firewall & Security:** UFW + AWS Security Groups  
- **Extras:** Git, future automation with Ansible/Terraform  

### Environment Structure

- **VPC & Network:** Public subnet with Internet Gateway  
- **EC2 & Elastic IP:** Fixed IP, minimal ports open (22, 80, 443)  
- **Nginx Config:** `/etc/nginx/sites-available/devathylla.conf` → `/var/www/html/devathylla`  
- **DNS:** A record pointing to Elastic IP  
- **Security:** SSH restricted, UFW active  

### Provisioning Steps

1. Provision EC2 Ubuntu 22.04 LTS  
2. Associate Elastic IP  
3. Connect via SSH (`ssh -i key.pem ubuntu@<Elastic-IP>`)  
4. Update system (`sudo apt update && sudo apt upgrade -y`)  
5. Install Nginx (`sudo apt install nginx -y`)  
6. Configure Nginx (`/etc/nginx/sites-available/devathylla.conf`)  
7. Create directory `/var/www/html/devathylla` and set permissions  
8. Remove default page (`sudo rm /var/www/html/index.nginx-debian.html`)  
9. Test configuration (`sudo nginx -t && sudo systemctl reload nginx`)  
10. Copy static site (`scp` or `git clone`)  
11. Configure DNS and verify propagation  

### HTTPS Certificate with Certbot

1. `sudo apt install certbot python3-certbot-nginx -y`  
2. `sudo certbot --nginx -d devathylla.site -d www.devathylla.site`  
3. Accept terms and redirect HTTP → HTTPS  
4. Certbot reloads Nginx automatically  
5. Automatic renewal (`sudo certbot renew --dry-run`)  

### Applied Security Checklist

- Automatic updates  
- SSH key auth, root disabled  
- UFW active  
- AWS Security Groups restrictive  
- Logging & monitoring  
- TLS/Nginx hardening  
- Periodic backups  

### Future Automation Possibilities

- Terraform (full IaC)  
- Ansible (automated setup)  
- Docker + Nginx (containerization)  
- CI/CD with GitHub Actions  
- Immutable infrastructure with Packer  
- Scalability & high availability  

---

## 🧑‍💻 Author / Responsável Técnico

**Athylla Rodrigues – DevOps Engineer Jr**  
GitHub: [athyllarodrig](https://github.com/athyllarodrig)  
Website: [https://devathylla.site](https://devathylla.site)  



---

## 📚 References

1. [Ubuntu EC2 Hardening Guide](https://cipherprojects.com/blog/posts/complete-guide-hardening-ubuntu-ec2-instances)  
2. [Hosting a website with AWS EC2](https://littlebigtech.net/posts/hosting-a-website-on-aws-for-free/)  
3. [Secure Static Website Deployment using AWS EC2, Route 53, and Certbot](https://medium.com/@uwadon1/a-comprehensive-walkthrough-to-deploying-a-secure-high-performance-static-website-using-aws-ec2-637377eb995e)  
4. [DigitalOcean – Secure Nginx with Let's Encrypt](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)  
5. [Terraform in Action](https://dev.to/sre_panchanan/terraform-in-actiondeploying-an-ubuntu-ec2-instance-with-nginx-on-aws-2fdp)  
6. [Ansible Automated Web Stack](https://dev.to/hritikraj8804/automated-web-stack-deployment-using-ansible-and-nginx-on-aws-ikf)  
7. [Docker CMD/ENTRYPOINT](https://dev.to/kalkwst/interlude-1-cmd-entrypoint-and-run-2926)  
8. [CI/CD Static HTML with GitHub Actions](https://joy-joel.medium.com/ci-cd-for-beginners-deploy-a-static-html-website-automatically-to-aws-ec2-with-github-actions-abbe95b39bb7)
