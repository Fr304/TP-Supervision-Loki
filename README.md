# TP-Supervision-Loki

## Module 1 : Les Fondamentaux

### Exercice 1 : Déploiement mono-nœud et vérification du flux de logs

Pour réaliser ce TP, j’ai installé Docker sur un serveur Ubuntu. Ensuite, j’ai déployé les trois services : **Grafana**, **Loki** et **Grafana Alloy** grâce à un fichier **Docker Compose**.

Pour ce déploiement, j’ai créé deux fichiers de configuration :

- **loki/loki.yml** pour configurer Loki en stockage local
- **alloy/config.alloy** pour collecter les logs locaux ainsi que les logs Docker

Ensuite, j’ai connecté la datasource **Loki** à Grafana pour pouvoir visualiser le contenu des logs et créer des dashboards.

Ci-dessous, une capture d’écran qui valide le flux de collecte des logs vers Grafana :

<img width="1498" height="843" alt="image" src="https://github.com/user-attachments/assets/fcf2edd1-f933-43f4-a25a-c5cda8f879d2" />

### Exercice 2 : Comprendre les labels et le mécanisme de Relabeling

Dans le fichier de configuration Grafana Alloy, j’ai ajouté le label statique « **environment=development** » et le label dynamique « **loglevel** ».

Cette ligne dans le fichier de configuration Alloy permet d’ajouter le label statique :

```language
loki.process "labels_and_relabel" {
  stage.static_labels {
    values = {
      environment = "development",
    }
```

Cette ligne dans le fichier de configuration Alloy permet d’extraire une métadonnée depuis un contenu de log. Grâce à une expression régulière, chaque fichier de log qui inclut les termes « level », « **lvl** » ou « **severity** » est capturé et labellisé en loglevel.

```language
stage.regex {
  expression = "(?i)(?:level|lvl|severity)=(?P<loglevel>[a-z]+)"
}

stage.labels {
  values = {
    loglevel = "",
  }
}
```
Cette ligne du fichier de configuration permet de retirer le label natif « **filename** »

```language
stage.label_drop {
  values = ["filename"]
}
```

### Exercice 2 : Rotation des logs et découverte dynamique

Pour faire cet exercice, j'ai créé le répertoire **/var/log/apps** car il n'était pas présent sur mon serveur :

```bash
sudo mkdir -p /var/log/apps
```
Ensuite, j'ai créé un fichier de log nommé **app.log**. Celui-ci me permettra de simuler les logs d'une application :

```bash
sudo touch /var/log/apps/app.log
```

J'ai ajouté quelques lignes de logs manuellement dans ce fichier pour générer du contenu :

```bash
sudo nano /var/log/apps/app.log
```
```bash
INFO Application démarrée
ERROR Connexion base de données impossible
```

La configuration Alloy mise en place surveille déjà les fichiers de log présents dans le répertoire **/var/log**. Le fichier **app.log** sera donc détecté automatiquement.

```bash
__path__ = "/host/var/log/**/*.log"
```

Sur Grafana, je vois bien que les lignes de logs sont bien visibles :

<img width="1603" height="857" alt="image" src="https://github.com/user-attachments/assets/40794a93-8c0b-4074-ae77-b644ed27a810" />
<img width="1571" height="855" alt="image" src="https://github.com/user-attachments/assets/9a566523-a1ab-481f-b507-03720dfc2180" />

Nous allons à présent tester la rotation des logs. L'objectif est de m'assurer que, lorsque le fichier de log est renommé, modifié ou recréé, Alloy parvienne toujours à lire les logs.

Pour faire cela, je vais renommer le fichier **app.log** en **app.log.1** :

```bash
sudo mv /var/log/apps/app.log /var/log/apps/app.log.1
```

Je vais recréer un fichier avec le même nom, **app.log** :

```bash
sudo touch /var/log/apps/app.log
```

Je vais ajouter un nouveau log dans ce fichier : 

```bash
INFO Nouveau fichier test après rotation
```
Grâce a cette requête sur Grafana je vois bien le log : 

```bash
{environment="development"} |= "Nouveau fichier"
```
<img width="1587" height="851" alt="image" src="https://github.com/user-attachments/assets/da6b7ae3-34e7-40d7-b6de-a4de2ad3c9ed" />

