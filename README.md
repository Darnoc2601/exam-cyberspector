
# Rapport de Mise en Place d'un Serveur Linux Sécurisé pour Héberger un Site WordPress

## Introduction
Ce rapport décrit les étapes suivies pour l'installation et la sécurisation d'un serveur Linux  qui est destiné à héberger un site WordPress. L'objectif principal est de garantir un hébergement sécurisé, en limitant les vulnérabilités, en controllant l'accessibilité au site et en assurant que le site ne soit accessible que de manière sécurisée via HTTPS.

## 1. Préparation et Endurcissement du Serveur

### 1.1 Check-list de Sécurisation du Serveur Linux
- **Mise à jour du système**:
  - Commandes exécutées :
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt dist-upgrade -y
    ```

- **Configuration du pare-feu (UFW)**:
  - Commandes exécutées :
    ```bash
    sudo apt install ufw
    sudo ufw allow OpenSSH
    sudo ufw allow 80/tcp
    sudo ufw allow 443/tcp
    sudo ufw enable
    ```

- **Installation de Fail2ban pour la protection contre les attaques par force brute**:
  - Commandes exécutées :
    ```bash
    sudo apt install fail2ban
    sudo systemctl enable fail2ban
    ```
- **Configuration de l'Audit des Logs**:
   - Commandes exécutées :
     ```bash
     sudo apt install auditd
     sudo systemctl start auditd
     sudo auditctl -w /etc/passwd -p wa -k identity
     sudo auditctl -w /etc/shadow -p wa -k identity
     ```

- **Installation de ClamAV pour la détection de malwares**:
  - Commandes exécutées :
    ```bash
    sudo apt install clamav clamav-daemon
    sudo freshclam
    sudo systemctl start clamav-daemon
    ```
   ### 1.2 **Tests Réalisés**:
  - Vérification des mises à jour : succès.
  - Vérification du pare-feu avec `ufw status`: tout est correct.
  - Test de Fail2ban, ClamAV, auditd : protection effective.
 
  - ![Vérification des mise à jour et l'activation du pare-feu](/updfiwe.png)
  - ![Protection contre les attaques](/securiteeeeee.png)



## 2. Mise en Place du Serveur Web Sécurisé

### 2.1. Installation et Configuration de WordPress

- **Installation des prérequis**:
  - Commandes exécutées :
    ```bash
    sudo apt install apache2
    sudo apt install mysql-server
    sudo apt install php libapache2-mod-php php-mysql
    ```
- **Configuration de MySQL**:
  - Sécurisation de l'installation MySQL :
    ```bash
    sudo mysql_secure_installation
    ```
  - Création de la base de données et de l'utilisateur :
    ```sql
    CREATE DATABASE mabasededonne;
    CREATE USER 'mon-user'@'localhost' IDENTIFIED BY 'mot_de_passe_';
    GRANT ALL PRIVILEGES ON mabasededonne.* TO 'mon-user'@'localhost';
    FLUSH PRIVILEGES;
    ```
- **Installation de WordPress**:
  - Téléchargement et extraction de WordPress :
    ```bash
    wget https://wordpress.org/latest.tar.gz
    tar -xzvf latest.tar.gz
    sudo cp -R wordpress/* /var/www/html/wordpress
    ```
  - Configuration des permissions :
    ```bash
    sudo chown -R www-data:www-data /var/www/html/wordpress
    sudo chmod -R 755 /var/www/html/wordpress
    ```

### 2.2. Sécurisation du Serveur Web

- **Configuration d'Apache pour SSL** :
  - Commandes pour installer et configurer Let's Encrypt :
    ```bash
    sudo apt install certbot python3-certbot-apache
    sudo certbot --apache
    ```
  - Fichier de configuration Apache modifié : `/etc/apache2/sites-available/hounkpevi-website.cyberspector.xyz.conf`
    ```apache
    <VirtualHost *:443>
        ServerName hounkpevi-website.cyberspector.xyz
        DocumentRoot /var/www/html/wordpress

        SSLEngine on
        SSLCertificateFile /etc/ssl/certs/your_domain.crt
        SSLCertificateKeyFile /etc/ssl/private/your_domain.key

        <Directory /var/www/html/wordpress>
            AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```

    - Activation du site et des modules nécessaires :
     ```bash
     sudo a2enmod ssl
     sudo a2ensite hounkpevi-website.cyberspector.xyz.conf
     sudo systemctl restart apache2
     ```

- **Restriction de l'accès via le pare-feu** :
  - Commande pour restreindre l'accès HTTP non sécurisé :
    ```bash
    sudo ufw delete allow 80/tcp
    sudo ufw allow 443/tcp
    ```
- **Mise en place de Fail2ban pour Apache** :
  - Configuration ajoutée dans `/etc/fail2ban/jail.local` :
    ```plaintext
    [apache]
    enabled = true
    port = http,https
    logpath = /var/log/apache2/access.log
    maxretry = 5
    bantime = 600
    ```
 - **Controler l'accessibilité au site.** :
    - Installer l'utilitaire Apache pour créer le fichier .htpasswd, avec la commande suivante pour installer apache2-utils:
    - 
    ```plaintext
    sudo apt-get install apache2-utils
    ```
      
    - Créer le fichier .htpasswd et ajouter un utilisateur :
      
    ```plaintext
    sudo htpasswd -c /etc/apache2/.htpasswd utilisateur
    ```
        
    - Configurer l'authentification dans le fichier de configuration Apache pour votre site `/etc/apache2/sites-available/wordpress.conf` :
    ```plaintext
    <Directory /var/www/html/wordpress>
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
    </Directory>
    ```
    - Redemarrer le serveur Apache
    ```plaintext
    sudo systemctl restart apache2
    ```
