# devops-ubuntuserver
configuração servidor ubuntu server com integração com github

## 1. Atualize o sistema
    sudo apt update && sudo apt upgrade -y

## 2. Instale a stack LAMP
### 2.1. Instale o Apache
    sudo apt install apache2 -y
### 2.2. Instale o MySQL
    sudo apt install mysql-server -y
    sudo mysql_secure_installation
### 2.3. Instale o PHP
    sudo apt install php libapache2-mod-php php-mysql -y
### 2.4. Teste o PHP
    sudo nano /var/www/html/info.php

#### Adicione o seguinte conteúdo:
    <?php
      phpinfo();
    ?>
    
#### Acesse http://your_server_ip/info.php para verificar a instalação.

## 3. Instale o phpMyAdmin
    sudo apt install phpmyadmin -y
#### Habilite o phpMyAdmin no Apache:
    sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
    sudo systemctl restart apache2
## 4. Instale o Git e configure SSH
### 4.1. Instale o Git
    sudo apt install git -y

### 4.2. Configure SSH
#### Gere uma chave SSH:
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
#### Adicione a chave pública ao GitHub. Copie a chave com o comando:
    cat ~/.ssh/id_rsa.pub
## 5. Instale e configure o Flask com Apache e WSGI
### 5.1. Instale dependências
    sudo apt install python3-pip python3-dev libapache2-mod-wsgi-py3 -y
### 5.2. Configure o ambiente Flask
#### Crie um diretório para sua aplicação Flask:
    sudo mkdir /var/www/flaskapp
    cd /var/www/flaskapp
    python3 -m venv venv
    source venv/bin/activate
    pip install flask

#### Crie um arquivo app.py:
    sudo nano /var/www/flaskapp/app.py

##### Adicione o seguinte código Flask básico:
    from flask import Flask
    app = Flask(__name__)
    
    @app.route('/')
    def hello_world():
        return 'Hello, Flask!'
    
    if __name__ == '__main__':
        app.run()

### 5.3. Configure o Apache para rodar o Flask na porta 80
    sudo nano /etc/apache2/sites-available/flaskapp.conf
#### Adicione o seguinte conteúdo:
    <VirtualHost *:80>
        ServerName your_server_ip
    
        WSGIDaemonProcess flaskapp python-path=/var/www/flaskapp:/var/www/flaskapp/venv/lib/python3.8/site-packages
        WSGIProcessGroup flaskapp
        WSGIScriptAlias / /var/www/flaskapp/flaskapp.wsgi
    
        <Directory /var/www/flaskapp>
            Require all granted
        </Directory>
    
        ErrorLog ${APACHE_LOG_DIR}/flaskapp_error.log
        CustomLog ${APACHE_LOG_DIR}/flaskapp_access.log combined
    </VirtualHost>


#### Crie o arquivo WSGI:
    sudo nano /var/www/flaskapp/flaskapp.wsgi
#### Adicione o seguinte conteúdo:
    import sys
    import logging
    sys.path.insert(0, "/var/www/flaskapp")
    
    from app import app as application
    application.logger.setLevel(logging.INFO)
#### Ative o site Flask e reinicie o Apache:
    sudo a2ensite flaskapp
    sudo systemctl restart apache2
## 6. Configure a integração com o GitHub
### 6.1. Clone seu repositório Flask
#### Navegue até o diretório /var/www:
    cd /var/www/flaskapp
    git clone git@github.com:seuusuario/seurepositorio.git .
### 6.2. Configure deploy automático via Webhooks
#### No GitHub, vá para Settings > Webhooks e adicione um novo webhook com o seguinte payload URL:
    http://your_server_ip/github-webhook/
#### No servidor, instale o flask-github-webhook:
    pip install flask-github-webhook
#### Crie um script Flask para o webhook:
    sudo nano /var/www/flaskapp/github_webhook.py
#### Adicione o seguinte código:
    from flask import Flask, request
    from github_webhook import Webhook
    import os
    
    app = Flask(__name__)
    webhook = Webhook(app)
    
    @app.route('/')
    def hello_world():
        return 'Hello from Flask!'
    
    @webhook.hook()  # Com isso, ele será acessível no endpoint /github-webhook/
    def on_push(data):
        os.system('cd /var/www/flaskapp && git pull origin main')
        return 'Updated!'
    
    if __name__ == '__main__':
        app.run()


Configure o Apache para lidar com esse novo arquivo github_webhook.py da mesma forma que fizemos para o app.py. Reinicie o Apache após configurar o webhook.

## 7. Teste e finalize
Agora, cada vez que você fizer push para a branch main no GitHub, o servidor automaticamente puxará as mudanças e refletirá as atualizações na aplicação Flask rodando na porta 80.

Este setup cria uma stack LAMP com phpMyAdmin e uma stack Flask totalmente integrada com GitHub e rodando na porta 80, com deploys automáticos usando webhooks do GitHub.








