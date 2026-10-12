# TP Infrastructure as Code avec Terraform et Docker

## Réponses aux questions du TP

### Différence entre un bind mount et une copie dans une image Docker

Un bind mount permet de relier directement un dossier de mon ordinateur à un dossier du conteneur. Dans ce TP, quand je modifie le fichier HTML sur mon Mac, la modification est directement visible dans le conteneur sans avoir besoin de reconstruire l'image.

Avec une copie dans une image Docker, le fichier est intégré dans l'image au moment de sa construction. Si je modifie ensuite ce fichier, je dois généralement reconstruire l'image pour que le changement soit pris en compte.

### Pourquoi le conteneur est connecté à un réseau Docker dédié ?

Le conteneur est connecté à ce réseau pour avoir un réseau Docker dédié au projet. Cela permet de mieux organiser les communications et aussi de faciliter la communication avec d'autres conteneurs si on en ajoute plus tard. C'est plus clair que de tout laisser uniquement sur le réseau Docker par défaut.

### Quel est l'intérêt de déclarer le réseau dans Terraform ?

Déclarer le réseau directement dans Terraform permet de le créer automatiquement en même temps que le reste de l'infrastructure. Cela évite de devoir le créer manuellement avec Docker et rend le projet plus facile à reproduire sur une autre machine.

Le réseau fait aussi partie de l'infrastructure gérée par Terraform, donc Terraform peut suivre son état et le supprimer proprement quand on fait un `terraform destroy`.

### Comment vérifier que le réseau est supprimé lors de la destruction ?

Après avoir exécuté `terraform destroy`, j'ai vérifié les réseaux Docker avec la commande :

```bash
docker network ls
```

Le réseau créé pour le TP n'apparaissait plus dans la liste. Cela confirme que Terraform l'a bien supprimé avec le reste de l'infrastructure.

### Quelle est la différence entre `127.0.0.1` et une écoute sur toutes les interfaces ?

Quand le service écoute sur `127.0.0.1`, il est accessible seulement depuis ma propre machine. Dans ce TP, cela veut dire que le serveur Nginx est accessible depuis mon Mac, mais pas directement depuis une autre machine du réseau.

Si le service écoute sur toutes les interfaces réseau, il peut être accessible depuis l'extérieur de la machine, par exemple depuis un autre ordinateur du même réseau, selon la configuration du pare-feu et du réseau.

Utiliser `127.0.0.1` limite donc l'exposition du service et est plus adapté ici puisque le serveur est seulement utilisé pour les tests en local.

### Quelles ressources apparaissent dans `terraform state list` ?

La commande `terraform state list` affiche les ressources que Terraform gère dans l'état du projet.

Dans mon cas, j'avais :

```text
docker_container.web
docker_image.nginx
docker_network.app_network
```

Cela correspond au conteneur Nginx, à l'image Docker utilisée et au réseau Docker créé pour le projet.

### Quelles informations affiche `terraform state show` ?

La commande `terraform state show` permet d'afficher les informations détaillées d'une ressource gérée par Terraform.

Dans ce TP, avec :

```bash
terraform state show docker_container.web
```

j'ai pu voir par exemple le nom du conteneur, l'image utilisée, le réseau auquel il est connecté, son adresse IP, le port exposé ainsi que le dossier monté dans le conteneur.
Cette commande permet donc de voir comment Terraform connaît et décrit réellement une ressource dans son état.


### Pourquoi l'état Terraform doit-il être protégé ?

Le fichier d'état Terraform contient des informations importantes sur les ressources réellement gérées par le projet. Il permet à Terraform de faire le lien entre ce qui est écrit dans les fichiers `.tf` et les ressources qui existent réellement.

Il peut aussi contenir des informations sensibles ou techniques sur l'infrastructure. Il vaut donc mieux éviter de le modifier manuellement ou de le partager publiquement.

### Pourquoi faut-il éviter de publier l'état Terraform dans un dépôt public ?

Il vaut mieux éviter de publier le fichier `terraform.tfstate` dans un dépôt public car il peut contenir beaucoup d'informations sur l'infrastructure, comme les noms des ressources, les adresses, les ports ou d'autres valeurs qui ne doivent pas forcément être visibles par tout le monde.

Selon le projet, il peut aussi contenir des informations sensibles. C'est pour cela que dans ce TP, les fichiers d'état sont ajoutés dans le `.gitignore` afin qu'ils ne soient pas envoyés sur GitHub.

### Comment l'historique Git aide pour la revue, la collaboration et le retour en arrière ?

L'historique Git permet de garder une trace des différentes modifications faites dans le projet. Chaque commit correspond à une étape précise, ce qui permet de voir ce qui a changé et à quel moment.

Pour la collaboration, cela permet à plusieurs personnes de travailler sur le même projet tout en gardant un historique clair des modifications de chacun.

Si une modification pose un problème, Git permet aussi de revenir à une version précédente ou de comparer deux versions pour comprendre d'où vient l'erreur.

## Questions de compréhension

### 1. Quelle est la différence entre `terraform init`, `terraform plan` et `terraform apply` ?

`terraform init` sert à initialiser le projet Terraform. Il prépare le dossier de travail et télécharge les providers nécessaires au projet.

`terraform plan` permet de voir les changements que Terraform prévoit de faire sur l'infrastructure, sans les appliquer. Cela permet de vérifier ce qui va être créé, modifié ou supprimé avant de continuer.

`terraform apply` applique réellement les changements prévus et modifie l'infrastructure.

### 2. Quel est le rôle d'un provider Terraform ?

Un provider permet à Terraform de communiquer avec un service ou un outil externe.

Dans ce TP, on utilise le provider Docker. Il permet à Terraform de créer et gérer des ressources Docker comme l'image Nginx, le réseau et le conteneur.

Terraform décrit ce qu'on veut dans les fichiers `.tf`, puis le provider traduit cette configuration en actions réalisées avec Docker.

### 3. À quoi sert le fichier `terraform.tfstate` ?

Le fichier `terraform.tfstate` permet à Terraform de garder une trace de l'infrastructure qu'il gère réellement.

Il fait le lien entre les ressources déclarées dans les fichiers Terraform et les ressources qui existent vraiment. Grâce à cet état, Terraform peut savoir ce qui existe déjà et comparer la configuration actuelle avec l'infrastructure réelle.

C'est ce qui lui permet ensuite de déterminer ce qu'il faut créer, modifier ou supprimer.

### 4. Pourquoi utilise-t-on des variables et des outputs ?

Les variables permettent de modifier certaines valeurs du projet sans avoir à réécrire directement les ressources Terraform. Par exemple, dans ce TP, on a utilisé des variables pour changer le nom du conteneur, le port et le nom du réseau.

Les outputs servent à afficher facilement des informations utiles après le déploiement, comme le nom du conteneur, le nom du réseau ou l'URL du serveur.

Cela rend la configuration plus simple à modifier et plus facile à réutiliser.

### 5. Qu'est-ce qu'une dérive de configuration ?

Une dérive de configuration correspond à une différence entre l'état attendu par Terraform et l'état réel de l'infrastructure.

Par exemple, dans ce TP, j'ai arrêté manuellement le conteneur avec Docker. Quand j'ai lancé `terraform plan`, Terraform a détecté que le conteneur n'était plus dans l'état prévu et a proposé de le recréer.

La dérive montre donc qu'une modification a été faite en dehors de Terraform et que l'infrastructure réelle ne correspond plus à la configuration définie.

### 6. Quelle différence existe-t-il entre Infrastructure as Code et configuration manuelle ?

Avec l'Infrastructure as Code, l'infrastructure est décrite dans des fichiers. On peut donc la recréer, la modifier et la versionner plus facilement.

Avec une configuration manuelle, il faut créer ou modifier les ressources directement à la main, par exemple avec des commandes Docker. Cela fonctionne, mais c'est plus difficile à reproduire exactement et il y a plus de risques d'oublier une étape.

L'Infrastructure as Code rend donc le déploiement plus reproductible et plus simple à suivre.

### 7. Quel est l'intérêt de Git dans un projet IaC ?

Git permet de garder un historique des modifications faites dans les fichiers Terraform.

Cela permet de savoir ce qui a changé, de revenir à une version précédente si une modification pose problème et de travailler plus facilement à plusieurs sur le même projet.

Dans un projet IaC, c'est utile car l'infrastructure est décrite dans du code. On peut donc versionner les changements de l'infrastructure comme on versionnerait du code classique.

### 8. Quelle est la différence entre un contrôle CI et un déploiement CD ?

La CI sert surtout à vérifier automatiquement que le code est correct après une modification. Dans ce TP, GitHub Actions vérifie par exemple le formatage avec `terraform fmt` et la validité de la configuration avec `terraform validate`.

Le CD va plus loin puisqu'il sert à déployer automatiquement l'application ou l'infrastructure après les vérifications.

Dans notre cas, le workflow fait seulement de la CI car il contrôle les fichiers Terraform mais ne déploie pas le conteneur Docker.

### 9. Pourquoi faut-il examiner le plan avant d'appliquer une modification ?

Le plan permet de voir à l'avance ce que Terraform compte créer, modifier ou supprimer.

C'est important de le vérifier avant de faire `terraform apply`, car cela permet de repérer une erreur ou un changement qu'on n'avait pas prévu.

Dans ce TP, on a vu par exemple que le changement d'un nom ou d'un port pouvait obliger Terraform à remplacer certaines ressources. Regarder le plan permet donc d'éviter d'appliquer une modification sans savoir exactement ce qu'elle va faire.

### 10. Quelles précautions faut-il prendre avec les fichiers d'état et les secrets ?

Les fichiers d'état Terraform doivent être protégés car ils peuvent contenir des informations importantes sur l'infrastructure et parfois des données sensibles.

Il faut éviter de les publier dans un dépôt public et les ajouter dans le `.gitignore`. Dans ce TP, les fichiers `*.tfstate` et `*.tfvars` sont ignorés par Git.

Pour les secrets, il ne faut pas les écrire directement dans les fichiers versionnés. Il vaut mieux utiliser des variables, des secrets GitHub ou un autre système prévu pour stocker les informations sensibles de manière sécurisée.