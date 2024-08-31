# Este guia fornece um exemplo de configuração de um servidor Ubuntu para rodar uma aplicação Node.js em um subdomínio, utilizando Nginx como proxy reverso e SSL para segurança.

Este documento fornece um guia passo a passo para configurar um servidor Ubuntu para rodar uma aplicação Node.js em um subdomínio usando Nginx como proxy reverso e um certificado SSL usando Certbot (Let's Encrypt).

## Passo 1: Configurar o Diretório do Projeto

1. Navegue até o diretório `/var`:
   ```bash
   cd /var
   ```

2. Crie o diretório `www` para hospedar a aplicação:
   ```bash
   sudo mkdir www
   cd www
   ```

3. Instale o Git:
   ```bash
   sudo apt install git
   ```

4. Clone o repositório do projeto:
   ```bash
   sudo git clone [project link]
   ```

## Passo 2: Instalar Node.js e NPM

1. Atualize os pacotes do sistema:
   ```bash
   sudo apt update
   ```

2. Instale o `npm` e o `nodejs`:
   ```bash
   sudo apt install npm
   sudo apt install nodejs
   ```

3. Instale o Node Version Manager (NVM):
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
   source ~/.bashrc
   ```

4. Instale a versão desejada do Node.js:
   ```bash
   nvm list-remote
   nvm install v20.16.0
   ```

## Passo 3: Configurar o Firewall

1. Instale o `firewalld`:
   ```bash
   sudo apt install firewalld
   ```

2. Abra as portas necessárias no firewall:
   ```bash
   sudo firewall-cmd --permanent --add-port=3000/tcp
   sudo firewall-cmd --permanent --add-port=80/tcp
   sudo firewall-cmd --permanent --add-port=443/tcp
   sudo firewall-cmd --reload
   ```

## Passo 4: Instalar PM2 para Gerenciamento de Processos

1. Instale o PM2 para gerenciar o processo da aplicação Node.js:
   ```bash
   npm install -g pm2
   ```

2. Inicie a aplicação com o PM2:
   ```bash
   pm2 start index.js
   ```

## Passo 5: Configurar Nginx como Proxy Reverso

1. Instale o Nginx:
   ```bash
   sudo apt install -y nginx
   ```

2. Crie um arquivo de configuração para o seu subdomínio:
   ```bash
   sudo nano /etc/nginx/sites-available/app.nelsondev.com.br
   ```

   Adicione o seguinte conteúdo ao arquivo de configuração:

   ```nginx
   server {
       listen 80;
       server_name app.nelsondev.com.br;

       location / {
           proxy_pass http://localhost:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. Verifique a configuração do Nginx:
   ```bash
   sudo nginx -t
   ```

4. Ative a nova configuração:
   ```bash
   sudo ln -s /etc/nginx/sites-available/app.nelsondev.com.br /etc/nginx/sites-enabled/
   ```

5. Reinicie o Nginx:
   ```bash
   sudo systemctl restart nginx
   ```

## Passo 6: Instalar Certificado SSL com Let's Encrypt

1. Instale o Certbot:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

2. Obtenha e instale o certificado SSL para o seu subdomínio:
   ```bash
   sudo certbot --nginx -d app.nelsondev.com.br -d www.app.nelsondev.com.br
   ```

3. Verifique a renovação automática do certificado:
   ```bash
   sudo certbot renew --dry-run
   ```

## Passo 7: Configurar o PATH do Ambiente

1. Configure o ambiente para usar o caminho correto do Node.js:
   ```bash
   Environment=PATH=/home/ubuntu/.nvm/versions/node/v20.16.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/ubuntu/.nvm/versions/node/v20.16.0/bin:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
   ```

