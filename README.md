# b2a_uf_infra

## Théo

### Sécuriser l'accès SSH

https://community.gladysassistant.com/t/tutoriel-securiser-lacces-ssh-sur-votre-raspberry/2157

-   Demander le mot de passe pour l'utilisation de `sudo`

    ```
    $ cd /etc/sudoers.d
    $ sudo mv 010_pi-nopasswd 010_pi-passwd
    $ sudo vi 010_pi-passwd
    ```

    Modifier la ligne :

    ```
    pi ALL=(ALL) NOPASSWD: ALL
    ```

    En :

    ```
    pi ALL=(ALL) PASSWD: ALL
    ```

-   Copier le fichier de configuration SSH (au cas ou hein)

    ```
    $ cd /etc/ssh
    $ sudo cp sshd_config sshd_config.orig
    ```

-   Dans le fichier de config `/etc/ssh/sshd_config`

    -   Modifier le port d'ecoute de SSH _(22 par défaut)_ :  
        Port SSH passé sur le port **2222**

    -   Établir un temps pour se connecter :  
        `LoginGraceTime 30` (30sec pour se connecter avant fermeture du SSH)

    -   Nombre de tentative de mot de passe avant fermeture SSH :  
        `MaxAuthTries 1` (1 erreur et SSH fermé)

    -   Temps d'inactivité avant déconnexion :  
        `ClientAliveInterval 900` (15min d'inactivité et SSH raccroche)

    -   Redemarrer le service SSH
        ```
        sudo systemctl restart ssh
        ```

-   Authentification par clés SSH

    -   Sur **HOST** :

        -   Créer une paire de clés publique et privé sur le host

            ```
            ssh-keygen -t rsa -b 2048 -C "Un commentaire pour vous aider à identifier votre clé"
            ```

        -   Copier la clé publique sur la raspberry
            ```
            cd ~/.ssh
            scp id_rsa.pub pi@my_raspberry_ip:~/.ssh/authorized_keys
            ```
        -   Possibilité de se connecter a la raspberry en utilisant sa clé privé
            ```
            ssh pi@my_raspberry_ip -p <ssh_port> -i ~/.ssh/id_rsa
            ```

    -   Sur **RASPBERRY** :
        -   Dans le fichier `/etc/ssh/sshd_config`
            ```
            PasswordAuthentication no
            ChallengeResponseAuthentication no # this should be the default anyway
            UsePAM no
            ```
        -   Redemarrer le service SSH
            ```
            sudo service ssh restart
            ```

    On est maintenant obligé de se connecté au serveur avec nos clés privé, sinon l'accès est refusé

## Maxime

### I- Se connecter à la raspberry à distance

### II- Installer Git sur la raspberry

### III- Installer Gitea

-   Création gitea use :
    ```
    sudo adduser --disabled-login --gecos 'Gitea' git
    ```
-   Installation gitea :
    ```
    wget -O gitea https://dl.gitea.io/gitea/1.4.0/gitea-1.10.0-linux-arm-6
    ```
-   Le rendre executable :
    ```
    chmod +x gitea
    ```
-   Configuration du service gitea :

    ```
    [Unit]
    Description=Gitea
    After=syslog.target
    After=network.target
    After=mariadb.service mysqld.service postgresql.service memcached.service redis.service

    [Service]
    # Modify these two values and uncomment them if you have
    # repos with lots of files and get an HTTP error 500 because
    # of that
    ###
    #LimitMEMLOCK=infinity
    #LimitNOFILE=65535
    Type=simple
    User=git
    Group=git
    WorkingDirectory=/home/git
    ExecStart=/home/git/gitea web
    Restart=always
    Environment=USER=git HOME=/home/git

    [Install]
    WantedBy=multi-user.target
    ```

-   Lancement de gitea :
    ```
    sudo systemctl enable gitea
    sudo systemctl start gitea
    ```
-   Installer Nginx :
    ```
    sudo apt-get install nginx -y
    ```
-   Modifications fichiers de configuration :

    ```
    /etc/nginx/sites-available/gitea
    ```

    ```
    server {
        listen 443 ssl;
        server_name rpi-git myraspberry.sytes.net;
        ssl_certificate     /etc/letsencrypt/live/myraspberry.sytes.net/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/myraspberry.sytes.net/privkey.pem;

        location / {
            client_max_body_size 364M;
            proxy_pass http://localhost:3000;
            proxy_connect_timeout 600;
            proxy_send_timeout 600;
            proxy_read_timeout 600;
            send_timeout 600;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }

    server {
        listen 80;
        server_name rpi-git myraspberry.sytes.net;
        return 301 https://$host$request_uri;
    }
    ```

-   Activer gitea avec nginx :
    ```
    sudo ln -s /etc/nginx/sites-available/gitea /etc/nginx/sites-enabled/gitea
    sudo rm /etc/nginx/sites-enabled/default
    sudo service nginx restart
    ```
-   Activation du certificat ssl :
    ```
    wget https://dl.eff.org/certbot-auto
    chmod a+x certbot-auto
    sudo ./certbot-auto certonly --standalone -d myraspberry.sytes.net
    ```
-   Reload certificat ssl grâce à cron tache :
    ```
     0 1 2 * * sudo service nginx stop && sudo /home/pi/certbot-auto renew && sudo service nginx start
    ```
-   Activation nginx :
    ```
    sudo service nginx restart
    ```
