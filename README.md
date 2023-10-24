# Réplication de la base de données "radars" depuis un conteneur Docker avec MariaDB dans une machine locale

Ce fichier readme (technique) explique les différentes étapes effectuées pour configurer le serveur source (Master) et le replica dans la machine locale (Slave).

## Configuration du source/master (Conteneur Docker) :

### Connexion au Serveur :
1. Je me suis connecté au serveur distant via SSH. (les informations du serveur sont masquées pour question de sécurité)
![1](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/fb1a0179-3382-49b4-b78b-f70903caf956)
### Gestion des Conteneurs Docker :
2. J'ai listé la liste des conteneurs Docker : `sudo docker ps`
![2](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/b04755e5-ca41-44fb-a9ef-6c1d774694a2)
3. J'ai copié le nom du conteneur requis et exécutez une session interactive : `sudo docker exec -it mymariadb /bin/bash`
### Configuration de MariaDB :
4. Jai modifié la configuration de MariaDB :
   - J'ai ajouté `server_id = 1` et `log_bin = /var/log/mysql/mysql-bin.log` dans '/etc/mysql/mariadb.conf.d/50-server.cnf'.
![Capture d’écran du 2023-10-21 21-12-48](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/daf32785-7dfb-43ec-97c5-9ed24d91c560)
   - J'ai ajouté `log-bin = mysql-bin` et `server_id = 1` dans le fichier '/etc/mysql/my.cnf'.
![Capture d’écran du 2023-10-21 21-13-20](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/9438326c-942f-4225-a514-639b30bbe87e)
5. Redémarrage de MariaDB : `sudo systemctl restart mariadb`
### Configuration de la Réplication :
6. La connexion à MariaDB : `mariadb -p` (Mot de passe : "MyMariaDB")
7. J'ai vérifié l'id du serveur actuel avec `SELECT @server_id;`.
8. J'ai définit le server_id à 1 avec `SET GLOBAL server_id = 1;`.
9. J'ai crée un utilisateur de réplication avec la requete: `CREATE USER 'repl'@'%' IDENTIFIED BY 'pass';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';`
10. J'ai éxécuté `SHOW MASTER STATUS;` pour afficher les détails du fichier binaire et de sa position.
![3](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/489c9c09-32ca-483c-8e57-3fdb0bed4f5e)

## Configuration Slave (Base de Données Locale) :

### Installation de MariaDB Localement :
1. J'ai installé MariaDB localement avec la commande: `sudo apt-get install mariadb-server mariadb-client -y`.
2. J'ai vérifié l'installationde MariaDB : `mariadb -V`
![Capture d’écran du 2023-10-21 21-16-17](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/5cab3bb4-444c-453c-9c0b-e10eb3995a71)
3. J'ai vérifié l'état de MariaDB : `sudo systemctl status mariadb`, ca doit être "active(running)".
![Capture d’écran du 2023-10-21 21-17-06](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/f9fc69b1-f059-4c3e-a5f7-7d7ddb0ccf22)
### Configuration de la Réplication sur le Slave :
1. J'ai configuré le fichier '/etc/mysql/mariadb.conf.d/50-server.cnf' en ajoutant les 2 variables pour le log_bin et relay_log:
2. Accédez à la base de données locale : `mariadb -p`.
3. Démarrez la réplication en modifiant les paramètres du maître.<br />
`CHANGE MASTER TO`<br />
  `MASTER_HOST = 'XX.XXX.XX.XXX',`<br /> 
  `MASTER_USER = 'utilisateurRepl',`<br />
  `MASTER_PASSWORD = 'test',`<br />
  `MASTER_LOG_FILE = 'mysql-bin.xxxxxx',`<br />
  `MASTER_LOG_POS = 16201,`<br />
  `MASTER_PORT = 3307;`
4. Démarrez l'esclave : `START SLAVE;`
5. J'ai éxécuté `SHOW SLAVE STATUS\G` pour voir si la connextion est établie entre la BDD locale et la BDD du conteneur Docker. J'ai vérifié que "Slave_IO_Running: Yes" et "Slave_SQL_Running: Yes" s'affichent, ca indique qu'il y a une connexion au conteneur Docker.
![Capture d’écran du 2023-10-21 21-24-06](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/945e33e5-2848-43dc-88e5-8e31f2687388)
6. Pour confirmer la réplication, j'ai fait une requête de comptage dans la table radars à la fois dans la source et l'esclave : `SELECT count(*) from radars;`, il faut qu'on recoit le même résultat, par exemple: 480. 
![Capture d’écran du 2023-10-21 21-25-14](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/0a1399e7-a4dc-4ab8-b56e-314c73dc5ccd)
7. Pour que je sois certain, j'ai inséré des données aléatoires dans la base de données du conteneur, puis j'ai fait un nouveau comptage dans la base de données locale. cela confirme la réplication réussie entre les deux bases de données.
![Capture d’écran du 2023-10-21 21-26-14](https://github.com/AymaneLk/Replication-de-la-base-de-donnees-radars-depuis-un-conteneur-Docker-avec-MariaDB/assets/24440328/908ee1b9-b8a3-442b-a903-ea662057c9f6)


