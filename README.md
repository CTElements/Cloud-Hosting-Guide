# Hospedagem de API Node.js em Servidor Linux na Nuvem

Este guia descreve o processo completo para configurar e hospedar uma API Node.js em um servidor Linux na nuvem, incluindo a configuração de firewall, a criação de um serviço do sistema para a aplicação, a configuração do Nginx como um proxy reverso e a adição de um certificado SSL.

## 1. Configuração Inicial do Servidor

### 1.1. Adicionar o Projeto ao Servidor

Execute os seguintes comandos no terminal para preparar o servidor e adicionar o projeto:

```bash
# Navegar para o diretório apropriado
cd ../
cd ../
cd /var

# Criar o diretório onde o projeto será armazenado
sudo mkdir www
cd www

# Instalar Git
sudo apt update
sudo apt install git

# Clonar o repositório do projeto
sudo git clone [project link]

# Instalar Node.js e npm
sudo apt install nodejs npm

# Navegar para o diretório do projeto e instalar dependências
cd project_name
sudo npm install

# Iniciar a aplicação
node index.js
````

## 1.2. Habilitar a Porta 3000 no Firewall do Linux
Para permitir que o servidor escute na porta 3000, execute os seguintes comandos:

````bash
# Instalar o firewall (se necessário)
sudo apt install firewalld

# Abrir a porta 3000
sudo firewall-cmd --permanent --add-port=3000/tcp

# Recarregar as configurações do firewall
sudo firewall-cmd --reload
````
## 2. Configurar a Aplicação como um Serviço do Sistema 
Para garantir que a API seja executada automaticamente após reinicializações, podemos configurá-la como um serviço do sistema:

### 2.1. Criar o Serviço
````bash
cd /etc/systemd/system
sudo nano gc.service
````

### 2.2. Adicionar o Código de Configuração do Serviço
Copie e cole o seguinte conteúdo no arquivo `gc.service`:
````bash
[Unit]
Description=Send Order
After=network.target

[Service]
ExecStart=/home/ubuntu/.nvm/versions/node/v20.16.0/bin/node /var/www/sendOrderToOperator/index.js
Restart=always
User=nobody
Group=nogroup
Environment=PATH=/home/ubuntu/.nvm/versions/node/v20.16.0/bin:/usr/bin:/usr/local/bin
Environment=NODE_ENV=production
WorkingDirectory=/var/www/sendOrderToOperator

[Install]
WantedBy=multi-user.target

````
### 2.3. Ativar e Iniciar o Serviço
````bash 
# Salvar o arquivo e sair do editor (Ctrl + O, Enter, Ctrl + X)

# Recarregar o daemon para reconhecer o novo serviço
sudo systemctl daemon-reload

# Ativar o serviço para iniciar automaticamente na inicialização
sudo systemctl enable gc

# Iniciar o serviço
sudo systemctl start gc

# Verificar o log do serviço
journalctl -u gc
````

## 3. Configurar o Nginx como Proxy Reverso
O Nginx pode ser usado como um proxy reverso para redirecionar as requisições para o servidor Node.js.
### 3.1. Instalar e Configurar o Nginx
````bash
# Atualizar a lista de pacotes e instalar o Nginx
sudo apt update
sudo apt install nginx
````
### 3.2. Configurar o Subdomínio
````bash
sudo mkdir -p /var/www/subdominio
sudo chown -R $USER:$USER /var/www/subdominio
sudo chmod -R 755 /var/www/subdominio
````
### 3.3. Criar um Arquivo de Configuração do Nginx para o Subdomínio
````bash
sudo nano /etc/nginx/sites-available/subdominio.seudominio.com
````
Adicione o seguinte conteúdo de exemplo ao arquivo:
````bash
server {
    listen 80;
    server_name sendOrderToOperator.nelsondev.com.br;

    root /var/www/subdominio;  
    index index.html;  

    location / {
        proxy_pass http://localhost:3030;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /sendXML {
        proxy_pass http://localhost:3030/sendXML;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Configuração de logs (opcional)
    error_log /var/log/nginx/sendOrderToOperator_error.log;
    access_log /var/log/nginx/sendOrderToOperator_access.log;
}
````
### 3.4. Ativar a Configuração e Reiniciar o Nginx
````bash
# Criar um link simbólico para ativar a configuração
sudo ln -s /etc/nginx/sites-available/sendOrderToOperator.nelsondev.com.br /etc/nginx/sites-enabled/

# Testar a configuração do Nginx
sudo nginx -t

# Reiniciar o Nginx
sudo systemctl restart nginx
````
### 4. Configurar o DNS do Domínio
Certifique-se de que o DNS do seu domínio (sendOrderToOperator.nelsondev.com.br) aponte para o endereço IP do seu servidor. Isso é feito no painel de controle do provedor do seu domínio.

## 5. Configurar Certificado SSL (Opcional, mas Recomendado)
Para configurar um certificado SSL gratuito usando o Let's Encrypt:

````bash
# Instalar Certbot e o plugin do Nginx
sudo apt install certbot python3-certbot-nginx

# Executar o Certbot para configurar o SSL
sudo certbot --nginx -d sendOrderToOperator.nelsondev.com.br
````

### 5.1. Renovação Automática de Certificado

````bash
sudo certbot renew --dry-run
````


## 4. Configurar Variáveis de Ambiente 
Para definir variáveis de ambiente no servidor: 
### 4.1. Adicionar Variáveis de Ambiente no ~/.bashrc
Abra o arquivo ~/.bashrc para edição:
````bash
nano ~/.bashrc
````
Adicione as variáveis de ambiente desejadas no final do arquivo:

````bash
export MONGO_USERNAME='seu_username'
export MONGO_PASSWORD='sua_senha'
export MONGO_DATABASE='seu_database'
export MONGO_HOST='seu_host'
````
Recarregue o ~/.bashrc para aplicar as alterações:
````bash
source ~/.bashrc
````

### 4.2. Verificar se as Variáveis Foram Definidas 
Para verificar se as variáveis de ambiente foram definidas corretamente, você pode usar os seguintes comandos:
````bash
echo $MONGO_USERNAME
echo $MONGO_PASSWORD
echo $MONGO_DATABASE
echo $MONGO_HOST
````
