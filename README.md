# b2a_uf_infra

## Théo
### Sécuriser l'accès SSH
https://community.gladysassistant.com/t/tutoriel-securiser-lacces-ssh-sur-votre-raspberry/2157  
+ Demander le mot de passe pour l'utilisation de `sudo`  
    ```
    $ cd /etc/sudoers.d
    $ sudo mv 010_pi-nopasswd 010_pi-passwd
    $ sudo vi 010_pi-passwd
    ```
    Modifier la ligne : `pi ALL=(ALL) NOPASSWD: ALL`
    En : `pi ALL=(ALL) PASSWD: ALL`  

+ Copier le fichier de configuration SSH (au cas ou hein)  
    ```
    $ cd /etc/ssh
    $ sudo cp sshd_config sshd_config.orig
    ```  

+ Dans le fichier de config : `sudo nano /etc/ssh/sshd_config`
    + Modifier le port d'ecoute de SSH (22 par défaut) 
        Port SSH passé sur le port 2222  

    + Établir un temps pour se connecter
        `LoginGraceTime 30` (30sec pour se connecter avant fermeture du SSH)  
    
    + Nombre de tentative de mot de passe avant fermeture SSH
        `MaxAuthTries 1` (1 erreur et SSH fermé)
    
    + Temps d'inactivité avant déconnexion  
        `ClientAliveInterval 900` (15min d'inactivité et SSH raccroche)

    + Redemarrer le service SSH  
        `sudo systemctl restart ssh`

+ Authentification par clés SSH  

    + Sur HOST
        + Créer une paire de clés publique et privé sur le host  

            `ssh-keygen -t rsa -b 2048 -C "Un commentaire pour vous aider à identifier votre clé"`

        + Copier la clé publique sur la raspberry  
            ```
            cd ~/.ssh
            scp id_rsa.pub pi@my_raspberry_ip:~/.ssh/authorized_keys
            ```  
        + Possibilité de se connecter a la raspberry en utilisant sa clé privé
            `ssh pi@my_raspberry_ip -p <ssh_port> -i ~/.ssh/id_rsa`
    + Sur RASPBERRY  
        + Dans le fichier /etc/ssh/sshd_config 
            ```
            PasswordAuthentication no
            ChallengeResponseAuthentication no # this should be the default anyway
            UsePAM no
            ```
        + Redemarrer le service SSH  
            ```
            sudo service ssh restart
            ```  

    On est maintenant obligé de se connecté au serveur avec nos clés privé, sinon l'accès est refusé