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
