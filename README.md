## Configuração de Servidor Linux

Este projeto faz parte do [UDACITY Full-Stack ND](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004) sobre **configuração de servidor linux**.  
O objetivo do projeto é acessar e configurar um servidor linux para manter um servidor web de banco de dados e hospedar tal aplicação.


### Acessar

[ec2-54-85-25-170.compute-1.amazonaws.com](ec2-54-85-25-170.compute-1.amazonaws.com)

### Endereços

* Endereço IP: 54.85.25.170

* Porta SSH: 2200

* URL: ec2-54-85-25-170.compute-1.amazonaws.com

### Softwares e Configurações

#### Softwares

* [Python 2.7.15](https://www.python.org/) Aplicação
* [Apache](https://www.apache.org/) Servidor web
* [PostgreSQL](https://www.postgresql.org/) Database
* [Git](https://git-scm.com/)

*Python Libraries*

* virtualenv
* Flask
* sqlalchemy
* Flask-SQLAlchemy
* psycopg2
* httplib2
* requests

#### Configurações

**1. Update and Upgrade**

Após logar no servidor:  
`sudo apt-get update && sudo apt-get upgrade`

**2. Alterar porta do SSH de 22 para 2200**

* Logado no servidor alterei a porta de 22 para 2200 no seguinte arquivo:  
`sudo nano /etc/ssh/ssh_config`  
`sudo service ssh restart`


* No site do Amazon Lightsail, na máquina do projeto, na seção redes, adicionei a porta 2200 e a 123 no firewall e removi a 22.

* No servidor configurei o UFW:
    ```
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow ssh
    sudo ufw allow 2200/tcp (Note: Changed SSH above to port 2200)
    sudo ufw allow www
    sudo ufw allow 123/udp
    sudo ufw enable
    ```


**3. Criar Usuário grader**

Logado no servidor, adicionei o usuário grader:  
`sudo adduser grader`

**4. Garantir Permissão sudo para grader**

Adicionei arquivo em /etc/sudoers.d, copiando configurações do usuário padrão da instância:  
`sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`

Alterei somente o nome do usuário de dentro do arquivo de ubuntu para grader:  
`sudo nano /etc/sudoers.d/grader`

**5. Criar Chaves de Autenticação**

Na máquina local:  
`ssh-keygen`

Agora temos as duas chaves, pública e privada.

**6. Adicionar Chave Pública ao Servidor**

Loguei na máquina como usuário grader:  
`sudo su - grader`

Adicionei os arquivos para chave:  
`sudo mkdir .ssh`  
`touch .ssh/authorized_keys`

Copiei, por fim, o conteúdo da chave pública para dentro do arquvio authorized_keys

**7. Garantir Permissões Para Arquivos SSH**

Como usuário grader:  
`chmod 700 .ssh`  
`chmod 664 .ssh/authorized_keys`

**8. Desabilitar Login por Senha e Login Remoto do root**

`sudo nano /etc/ssh/sshd_config`

Modificar seguintes configurações: `Password Authentication no` e `PermitRootLogin no`  

**9. Configurar Fuso Horário**

`dpkg-reconfigure tzdata`

**10. Instalar Apache e Aplicação Python mod_wsgi**

`sudo apt-get install apache2`  
`sudo apt-get install libapache2-mod-wsgi`


**11. Clonar Projeto dos Catalogos**

Criando pasta catalog no diretório www e clonando:
```
cd /var/www
sudo mkdir catalog
cd catalog
git clone 'git-repository'
```

Impossibilitando de acessar o diretorio .git adicionando RedirectMatch 404 /\\\.git em arquivo htaccess em:
/var/www

**12. Configurando PostgreSQL**

conectar no database como postgres:
`sudo su - postgres psql`

Criar novo usuário:  
`CREATE USER catalog WITH PASSWORD 'senha';`

Atualizando permisões do usuário catalog:  
`ALTER ROLE catalog WITH LOGIN;`  
`ALTER ROLE catalog CREATEDB;`

Criando o database:
`CREATE DATABASE catalog WITH OWNER catalog;`

Configurando database: 
```
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
```

Na aplicação foi necessário alterar a configuração do banco de dados de sqlite para postgresql

**13. Configurar Virtual Environment e Virtual Host** 

Assim cada aplicação tem seu própria versão dos frameworks
```
sudo apt-get install python-pip

cd /var/www/catalog/catalog
sudo pip install virtualenv

source venv/bin/activate

sudo pip install Flask
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install psycopg2
sudo apt-get install python-psycopg2 
sudo pip install httplib2
sudo pip install requests
```

Configurar o arquivo WSGI

`cd /var/www/catalog`  
`sudo nano catalog.wsgi`

Após configurado 

`sudo a2ensite catalog`  
`sudo service apache2 restart`

**14. Acessar aplicação pela URL**

`sudo nano /etc/apache2/sites-enabled/catalog.conf`

Adicionado seguinte linha a arquivo

`ServerAlias ec2-54-85-25-170.compute-1.amazonaws.com`

### Documentação fonte

[Fuso Horário](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442)  
[Git inacessível](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)  
[PostgreSQL config](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)  
[Deploy FLask](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)  
[Aplicação pela URL](https://httpd.apache.org/docs/2.4/vhosts/name-based.html)