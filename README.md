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

# 🚀 Deploy de Site Estático com HTTPS na AWS EC2

Este repositório contém um guia passo a passo completo para provisionar, configurar e publicar um site estático na AWS, utilizando uma instância EC2 com Nginx, um domínio customizado e segurança com HTTPS.

O guia está disponível em dois idiomas.

---

### 🌐 Navegue para a sua língua de preferência:

*   [🇧🇷 Português do Brasil](#-guia-completo-deploy-de-site-estático-com-https-na-aws-ec2)
*   [🇺🇸 English](#-complete-guide-static-website-deployment-with-https-on-aws-ec2)

---
<br>

## 🇧🇷 Guia Completo: Deploy de Site Estático com HTTPS na AWS EC2

Este repositório contém um guia passo a passo exato para provisionar, configurar e publicar um site estático simples ("Hello World") na AWS EC2, com domínio customizado e HTTPS via Certbot. Siga exatamente estas instruções para replicar o ambiente. O site final estará acessível em `https://SEU-DOMINIO.site` (substitua pelo seu).

### ⚙️ Ambiente Final

| Componente          | Especificação                                                    |
| ------------------- | ---------------------------------------------------------------- |
| **Instância EC2**   | Ubuntu 22.04 LTS (t2.micro, gratuito no tier free)               |
| **Webserver**       | Nginx servindo `/var/www/html/index.html` com "Hello World"      |
| **Domínio**         | Registrado na GoDaddy, apontando para um Elastic IP              |
| **HTTPS**           | Certificado Let's Encrypt via Certbot, com redirect automático   |
| **Segurança**       | UFW + AWS Security Groups (portas 22, 80, 443 liberadas)         |

### 📋 Pré-requisitos

*   **Conta AWS gratuita** (crie em [aws.amazon.com](https://aws.amazon.com)).
*   **Chave SSH gerada localmente** (use `ssh-keygen` no seu terminal).
*   **Computador com terminal** (Linux/Mac) ou PuTTY (Windows).
*   **Domínio registrado** (veja Etapa 2).

### 🌐 URL do Site em Produção
**URL:** [https://devathylla.site](https://devathylla.site)  
**Autor:** Athylla Rodrigues – DevOps Engineer Jr | **GitHub:** [athyllarodrig](https://github.com/athyllarodrig)

---

### Etapa 1: 💸 Gestão Financeira na AWS

Configure alertas para evitar custos inesperados. O tier free cobre a `t2.micro` por 750h/mês, mas o monitoramento é crucial.

1.  Acesse o console AWS e busque por **"Billing"**.
2.  No menu esquerdo, clique em **Budgets > Create budget**.
3.  Selecione **Cost budget**, defina o nome "Meu Budget Mensal", período "Monthly" e um limite (ex: $10 USD).
4.  Em **Alert threshold**, adicione alertas para 80% ($8) e 100% ($10) do orçamento, enviando para seu e-mail.
5.  Clique em **Create budget**.

![Navegando para Budgets no Billing Dashboard.](/images/pasted_file_xKrjEL_image.png)

![Tela de criação de Budget na AWS.](/images/pasted_file_1op0w5_image.png)

6.  No Billing Dashboard, ative o **Cost Explorer** para visualizar os custos.

---

### Etapa 2: 🌐 Aquisição de Domínio na GoDaddy

1.  Acesse [godaddy.com](https://godaddy.com) e crie ou faça login em sua conta.
2.  Busque por um nome de domínio profissional e adicione uma extensão disponível (ex: `.site`, `.com`) ao carrinho.
3.  Complete a compra.
4.  Após a compra, vá para **My Products > Domains** para confirmar que o domínio está ativo.

![Tela de "My Products" na GoDaddy mostrando o domínio ativo.](/images/Yrj3MGN2QlkS.png)

**Nota:** A configuração do DNS será feita na Etapa 5.

---

### Etapa 3: 💻 Provisionamento de Servidor (EC2)

1.  No console AWS, busque por **"EC2"** e clique em **Launch Instance**.
2.  **Nomeie a instância:** `MeuServidorWeb`.
3.  **AMI:** Selecione "Ubuntu Server 22.04 LTS" (free tier eligible).
4.  **Tipo de instância:** `t2.micro` (gratuito).
5.  **Key pair (login):** Crie um novo par de chaves com o nome `minha-chave-ssh`, tipo `.pem`, e baixe o arquivo. Proteja-o localmente:
    ```bash
    chmod 400 minha-chave-ssh.pem
    ```
6.  **Network settings:** Crie um novo **Security Group** chamado `WebServerSG` com as seguintes regras:
    *   **SSH** (porta 22) de `My IP`.
    *   **HTTP** (porta 80) de `Anywhere` (0.0.0.0/0).
    *   **HTTPS** (porta 443) de `Anywhere` (0.0.0.0/0).

![Regras de entrada para o Security Group na AWS.](/images/OiWA82jTg3Q0.png)

7.  **Armazenamento:** Mantenha o padrão de 8 GiB.
8.  Clique em **Launch instance**.
9.  **Alocar Elastic IP:** No painel do EC2, vá para **Elastic IPs**, aloque um novo endereço e associe-o à sua instância `MeuServidorWeb`. Anote este IP.

---

### Etapa 4: 🛠️ Configuração do Servidor Web

1.  Conecte-se à sua instância via SSH:
    ```bash
    ssh -i minha-chave-ssh.pem ubuntu@SEU-ELASTIC-IP
    ```
    *Substitua `SEU-ELASTIC-IP` pelo IP que você anotou.*

2.  Atualize o sistema e instale o Nginx:
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install nginx -y
    ```

3.  Inicie e habilite o Nginx e o firewall UFW:
    ```bash
    sudo systemctl start nginx && sudo systemctl enable nginx
    sudo ufw allow OpenSSH
    sudo ufw allow 'Nginx Full'
    sudo ufw --force enable
    ```

![`sudo ufw status verbose` mostrando as regras ativas.](/images/ZLGHeIbVx6tn.png)

4.  Crie o diretório do site e ajuste as permissões:
    ```bash
    sudo mkdir -p /var/www/html/devathylla
    sudo chown -R $USER:$USER /var/www/html/devathylla
    sudo chmod -R 755 /var/www
    ```

5.  Crie a página `index.html`:
    ```bash
    echo '<!DOCTYPE html><html><head><title>Hello World</title></head><body><h1>Hello World from AWS EC2!</h1><p>Site estático deployado com sucesso.</p></body></html>' | sudo tee /var/www/html/devathylla/index.html
    ```

6.  Crie o arquivo de configuração do Nginx para o seu site:
    ```bash
    sudo nano /etc/nginx/sites-available/devathylla
    ```
    Cole o seguinte conteúdo, substituindo `devathylla.site` pelo seu domínio:
    ```nginx
    server {
        listen 80;
        server_name devathylla.site www.devathylla.site;
        root /var/www/html/devathylla;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

7.  Ative o site, teste a configuração e recarregue o Nginx:
    ```bash
    sudo ln -s /etc/nginx/sites-available/devathylla /etc/nginx/sites-enabled/
    sudo rm /etc/nginx/sites-enabled/default
    sudo nginx -t
    sudo systemctl reload nginx
    ```

![`nginx -t` mostrando que a sintaxe está correta.](/images/58kHMA5ZYP2G.png)

Neste ponto, você já deve conseguir acessar `http://SEU-ELASTIC-IP` e ver a página "Hello World".

---

### Etapa 5: 🚀 Publicação do Site (DNS e HTTPS)

1.  **(Opcional) Copie o site do seu PC para o servidor via SCP:**
    ```bash
    scp -i minha-chave-ssh.pem index.html ubuntu@SEU-ELASTIC-IP:/var/www/html/devathylla/
    ```

2.  **Configure o DNS na GoDaddy:**
    *   Acesse a gestão de DNS do seu domínio.
    *   Adicione um registro **A** com `Nome` **@** apontando para o seu `SEU-ELASTIC-IP`.
    *   Adicione um registro **CNAME** com `Nome` **www** apontando para **@**.

![Tabela de registros DNS na GoDaddy.](/images/PadqjL27iDAJ.png)

3.  **Instale o Certbot para gerar o certificado HTTPS:**
    ```bash
    sudo apt install certbot python3-certbot-nginx -y
    ```

4.  **Gere o certificado:**
    ```bash
    sudo certbot --nginx -d devathylla.site -d www.devathylla.site
    ```
    *Siga as instruções: aceite os termos, compartilhe seu e-mail e escolha a opção de redirecionar todo o tráfego HTTP para HTTPS.*

![Certbot emitindo o certificado com sucesso.](/images/ipjHN9jKdEEw.jpg)

5.  **Teste a renovação automática:**
    ```bash
    sudo certbot renew --dry-run
    ```

Agora, acesse `https://seu-dominio.site` no navegador. Você deverá ver o site com um cadeado verde, indicando que o HTTPS está funcionando!

![Site final rodando com HTTPS válido.](/images/YLOQRAUeRvDd.jpg)

---

### 🔒 Checklist de Segurança Aplicada

-   [x] **Acesso SSH:** Apenas com chave privada (`PasswordAuthentication no`).
-   [x] **Usuário Root:** Desabilitado para login por padrão no Ubuntu.
-   [x] **Firewall:** UFW e Security Groups restringindo o tráfego.
-   [x] **Atualizações Automáticas:** Configuradas com `unattended-upgrades`.
-   [x] **Logs:** Monitoramento via `journalctl -u nginx`.
-   [ ] **Backups:** Crie um script de backup (ex: `backup.sh`).

### 🧩 Desafios e Soluções

*   **Propagação de DNS:** A demora foi monitorada com o comando `dig seu-dominio.site` até que o IP correto fosse exibido.
*   **Erro no Certbot:** A porta 80 estava bloqueada. A solução foi garantir que o Security Group e o UFW permitiam tráfego na porta 80.
*   **Controle de Custos:** O alerta de budget notificou um gasto mínimo, o que validou a eficácia do monitoramento.

### 🔮 Possibilidades Futuras

*   **Automação com Terraform:** Provisionar a infraestrutura como código (IaC).
*   **Gerenciamento de Configuração com Ansible:** Automatizar a configuração do servidor.
*   **CI/CD com GitHub Actions:** Implementar um pipeline de integração e entrega contínua.

### 📚 Referências

*   [Documentação do AWS EC2](https://docs.aws.amazon.com/ec2/)
*   [Guia do Certbot para Nginx](https://certbot.eff.org/instructions?ws=nginx&os=ubuntu-22)
*   [Documentação de DNS da GoDaddy](https://www.godaddy.com/help/manage-dns-zone-files-680)

<br>
<br>

---

## 🇺🇸 Complete Guide: Static Website Deployment with HTTPS on AWS EC2

This repository provides an exact step-by-step guide to provision, configure, and publish a simple static "Hello World" website on AWS EC2 with a custom domain and HTTPS via Certbot. Follow these instructions exactly to replicate the environment. The final site will be accessible at `https://YOUR-DOMAIN.site` (replace with yours).

### ⚙️ Final Environment

| Component       | Specification                                                    |
| --------------- | ---------------------------------------------------------------- |
| **EC2 Instance**| Ubuntu 22.04 LTS (t2.micro, free tier eligible)                  |
| **Webserver**   | Nginx serving `/var/www/html/index.html` with "Hello World"      |
| **Domain**      | Registered on GoDaddy, pointing to an Elastic IP                 |
| **HTTPS**       | Let's Encrypt certificate via Certbot, with automatic redirect   |
| **Security**    | UFW + AWS Security Groups (ports 22, 80, 443 only)               |

### 📋 Prerequisites

*   **Free AWS account** (create at [aws.amazon.com](https://aws.amazon.com)).
*   **Locally generated SSH key** (use `ssh-keygen` in your terminal).
*   **Computer with a terminal** (Linux/Mac) or PuTTY (Windows).
*   **Registered domain** (see Step 2).

### 🌐 Production Site URL
**URL:** [https://devathylla.site](https://devathylla.site)  
**Author:** Athylla Rodrigues – DevOps Engineer Jr | **GitHub:** [athyllarodrig](https://github.com/athyllarodrig)

---

### Step 1: 💸 Financial Management in AWS

Set up alerts to avoid unexpected costs. The free tier covers the `t2.micro` for 750h/month, but monitoring is crucial.

1.  Access the AWS console and search for **"Billing"**.
2.  In the left sidebar, click **Budgets > Create budget**.
3.  Select **Cost budget**, set the name to "My Monthly Budget", period to "Monthly", and a limit (e.g., $10 USD).
4.  Under **Alert threshold**, add alerts for 80% ($8) and 100% ($10) of the budget, sent to your email.
5.  Click **Create budget**.

![Navigating to Budgets in the Billing Dashboard.](/images/pasted_file_xKrjEL_image.png)

![AWS Budget creation screen.](/images/pasted_file_1op0w5_image.png)

6.  In the Billing Dashboard, enable **Cost Explorer** to visualize costs.

---

### Step 2: 🌐 Domain Acquisition on GoDaddy

1.  Go to [godaddy.com](https://godaddy.com) and create or log in to your account.
2.  Search for a professional domain name and add an available extension (e.g., `.site`, `.com`) to your cart.
3.  Complete the purchase.
4.  After purchase, go to **My Products > Domains** to confirm the domain is active.

!["My Products" screen on GoDaddy showing the active domain.](/images/Yrj3MGN2QlkS.png)

**Note:** DNS configuration will be done in Step 5.

---

### Step 3: 💻 Server Provisioning (EC2)

1.  In the AWS console, search for **"EC2"** and click **Launch Instance**.
2.  **Name the instance:** `MyWebServer`.
3.  **AMI:** Select "Ubuntu Server 22.04 LTS" (free tier eligible).
4.  **Instance type:** `t2.micro` (free tier).
5.  **Key pair (login):** Create a new key pair named `my-ssh-key`, type `.pem`, and download the file. Secure it locally:
    ```bash
    chmod 400 minha-chave-ssh.pem
    ```
6.  **Network settings:** Create a new **Security Group** named `WebServerSG` with the following rules:
    *   **SSH** (port 22) from `My IP`.
    *   **HTTP** (port 80) from `Anywhere` (0.0.0.0/0).
    *   **HTTPS** (port 443) from `Anywhere` (0.0.0.0/0).

![Inbound rules for the Security Group in AWS.](/images/OiWA82jTg3Q0.png)

7.  **Storage:** Keep the default 8 GiB.
8.  Click **Launch instance**.
9.  **Allocate Elastic IP:** In the EC2 dashboard, go to **Elastic IPs**, allocate a new address, and associate it with your `MyWebServer` instance. Note this IP.

---

### Step 4: 🛠️ Web Server Configuration

1.  Connect to your instance via SSH:
    ```bash
    ssh -i my-ssh-key.pem ubuntu@YOUR-ELASTIC-IP
    ```
    *Replace `YOUR-ELASTIC-IP` with the IP you noted.*

2.  Update the system and install Nginx:
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install nginx -y
    ```

3.  Start and enable Nginx and the UFW firewall:
    ```bash
    sudo systemctl start nginx && sudo systemctl enable nginx
    sudo ufw allow OpenSSH
    sudo ufw allow 'Nginx Full'
    sudo ufw --force enable
    ```

![`sudo ufw status verbose` showing active rules.](/images/ZLGHeIbVx6tn.png)

4.  Crie o diretório do site e ajuste as permissões:
    ```bash
    sudo mkdir -p /var/www/html/devathylla
    sudo chown -R $USER:$USER /var/www/html/devathylla
    sudo chmod -R 755 /var/www
    ```

5.  Crie a página `index.html`:
    ```bash
    echo '<!DOCTYPE html><html><head><title>Hello World</title></head><body><h1>Hello World from AWS EC2!</h1><p>Static site deployed successfully.</p></body></html>' | sudo tee /var/www/html/devathylla/index.html
    ```

6.  Crie o arquivo de configuração do Nginx para o seu site:
    ```bash
    sudo nano /etc/nginx/sites-available/devathylla
    ```
    Paste the following content, replacing `devathylla.site` with your domain:
    ```nginx
    server {
        listen 80;
        server_name devathylla.site www.devathylla.site;
        root /var/www/html/devathylla;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
    ```

7.  Enable the site, test the configuration, and reload Nginx:
    ```bash
    sudo ln -s /etc/nginx/sites-available/devathylla /etc/nginx/sites-enabled/
    sudo rm /etc/nginx/sites-enabled/default
    sudo nginx -t
    sudo systemctl reload nginx
    ```

![`nginx -t` showing the syntax is ok.](/images/58kHMA5ZYP2G.png)

At this point, you should be able to access `http://YOUR-ELASTIC-IP` and see the "Hello World" page.

---

### Step 5: 🚀 Site Publication (DNS and HTTPS)

1.  **(Opcional) Copy the site from your PC to the server via SCP:**
    ```bash
    scp -i my-ssh-key.pem index.html ubuntu@YOUR-ELASTIC-IP:/var/www/html/devathylla/
    ```

2.  **Configure DNS on GoDaddy:**
    *   Access the DNS management for your domain.
    *   Add an **A** record with `Name` **@** pointing to your `YOUR-ELASTIC-IP`.
    *   Add a **CNAME** record with `Name` **www** pointing to **@**.

![DNS records table on GoDaddy.](/images/PadqjL27iDAJ.png)

3.  **Install Certbot to generate the HTTPS certificate:**
    ```bash
    sudo apt install certbot python3-certbot-nginx -y
    ```

4.  **Generate the certificate:**
    ```bash
    sudo certbot --nginx -d devathylla.site -d www.devathylla.site
    ```
    *Follow the prompts: agree to the terms, share your email, and choose the option to redirect all HTTP traffic to HTTPS.*

![Certbot successfully issuing the certificate.](/images/ipjHN9jKdEEw.jpg)

5.  **Test automatic renewal:**
    ```bash
    sudo certbot renew --dry-run
    ```

Now, access `https://your-domain.site` in your browser. You should see the site with a green lock, indicating that HTTPS is working!

![Final site running with a valid HTTPS certificate.](/images/YLOQRAUeRvDd.jpg)

---

### 🔒 Applied Security Checklist

-   [x] **SSH Access:** Only with a private key (`PasswordAuthentication no`).
-   [x] **Root User:** Disabled for login by default on Ubuntu.
-   [x] **Firewall:** UFW and Security Groups restricting traffic.
-   [x] **Automatic Updates:** Configured with `unattended-upgrades`.
-   [x] **Logs:** Monitoring via `journalctl -u nginx`.
-   [ ] **Backups:** Create a backup script (e.g., `backup.sh`).

### 🧩 Challenges and Solutions

*   **DNS Propagation:** The delay was monitored with the `dig your-domain.site` command until the correct IP was displayed.
*   **Certbot Error:** Port 80 was blocked. The solution was to ensure the Security Group and UFW allowed traffic on port 80.
*   **Cost Control:** The budget alert notified a minimal expense, which validated the effectiveness of the monitoring.

### 🔮 Possibilities Futuras

*   **Automação com Terraform:** Provisionar a infraestrutura como código (IaC).
*   **Gerenciamento de Configuração com Ansible:** Automatizar a configuração do servidor.
*   **CI/CD com GitHub Actions:** Implementar um pipeline de integração e entrega contínua.

### 📚 Referências

*   [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
*   [Guia do Certbot para Nginx](https://certbot.eff.org/instructions?ws=nginx&os=ubuntu-22)
*   [Documentação de DNS da GoDaddy](https://www.godaddy.com/help/manage-dns-zone-files-680)
