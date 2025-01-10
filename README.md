# SAULO ADONES GABRIEL GUIMARÃES DA SILVEIRA
# Documentação Estagio DevOps

Documentação para criação dos VirtualHost e Wordpress e Docker Container

## Começando

### Pré-requisitos

- Nginx
- php-fpm
- Wordpress
- MariaDB
- Docker
- AWS Linux
- Fedora

### Primeiros Passos

**Acesando a maquina virtual**

```bash
    ssh -i yourpemkey.pem usuario@IP-MAQUINA
```

**Criando o usuario com o shell "BASH" e com o repositorio com seu nome no /home** 

```bash
    useradd -m -s /bin/bash nginx
```

**Criando a senha para o usuario**

```bash
    passwd nginx
```
**Gerando acesso para o usuario no seu diretorio**
```bash
    chown nginx /home/nginx
    chmod 770 
```

**Configurando sudoers para dar certas permissões ao usuario nginx**


```bash
    echo "nginx ALL=(ALL) NOPASSWD: /bin/yum install *
    nginx ALL=(ALL) NOPASSWD: /usr/bin/nano
    nginx ALL=(ALL) NOPASSWD: /bin/systemctl
    nginx ALL=(ALL) NOPASSWD: /usr/sbin/nginx
    nginx ALL=(ALL) NOPASSWD: /usr/bin/mv" | sudo tee -a  /etc/sudoers.d/nginx_sudoers    
```

## Instalando os serviços necessarios

**Como usuario root instale as dependencias do mysql**

```bash
    sudo dnf -y update
    sudo dnf search mariadb
    sudo dnf install mariadb105-server mariadb105-server-utils -y
    sudo dnf install mysql-community-server -y
```

**Inicie e Ative o mariadb**

```bash
   sudo systemctl start mariadb
   sudo systemctl enable mariadb
```
**Set as configurações inicias, Recomendo colocar uma senha forte**

```bash
   sudo mysql_secure_installation
```

## Configuração do Banco MariaDB

**Crie um usuario para o usuario Nginx poder acessar o banco**

```bash
    mysql -u root -p
```
```bash
    CREATE USER 'nginx'@'%' IDENTIFIED BY 'nginx';
    GRANT ALL PRIVILEGES ON *.* TO 'nginx'@'%';
    FLUSH PRIVILEGES;
```
**Precisamos garantir que mariadb server esta configurado para acesso remoto acesse o arquivo e descomente o bind-address**

```bash
    sudo nano /etc/my.cnf.d/mariadb-server.cnf 
```

## Usuario Nginx

**No usuario nginx instale os pacotes necessarios**

```bash
    sudo su - nginx
    yum install -y nginx php-fpm wordpress docker
```

## Nginx com PHP-FPM Service VirtualHost

**Vamos precisar criar uma direito para armazena nosso site em php**

```bash
    mkdir -p /var/www/html/site-exemplo
    sudo nano index.php
```

**Agora criaremos um diretorio para configurar o arquivo do VirtualHost**

```bash
    mkdir -p /etc/nginx/conf.d
    sudo nano default.conf
```

**Dentro do arquivo cole a seguinte informação**

```bash
server {
    listen 80;
    server_name yourdns www.yourdns;

    root /var/www/html/site-exemplo;  
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php-fpm.sock;  
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_index index.php;
    }

    location ~ /\.ht {
        deny all;
        }
    }

```
**Depois utilize o comando do nginx para testar as configuraçoes**

```bash
    nginx -t
```

**Agora reinicie o serviço do nginx ele automaticamente ira criar um link de referencia**

```bash
    sudo systemctl restart nginx
    sudo systemctl enable nginx
```

## Wordpress

**Crie um diretorio para armazena o conteudo do wordpress**

```bash
    mkdir -p /var/www/html/wordpress-exemplo
    mv wordpress /var/www/html/wordpress-exemplo
```

**Agora criaremos um novo arquivo dentro do mesmo diretorio que esta a configuração do php-fpm**

```bash
    sudo nano wordpress.conf
```

**Dentro do arquivo cole a seguinte informação**
```bash
    server {
    listen 80;
    server_name yourdns www.your-dns;

    root /var/www/html/wordpress-exemplo; 
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params
        fastcgi_pass unix:/var/run/php-fpm.sock;
        
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_index index.php;
    }

    location ~ /\.ht {
        deny all;
    }
}
  
```

## Docker

**Primeiro como usuario root vamos permite que o usuario nginx tenha acesso as funções do docker inserindo ele dentro do grupo de docker**

```bash
    usermod -aG docker nginx
```

**Como usuario nginx instale uma imagem do nginx basica dando o docker push**

```bash
    docker push nginx
    docker run --name nginx-service -p -d 8080:80 nginx
```

## Links

### Docker: http://docker-saulo-adones.dreamsquad.com.br:8080
### Wordpress: http://blog-saulo-adones.dreamsquad.com.br
### Site PHP: http://php-saulo-adones.dreamsquad.com.br
