---
layout: post
comments: true
title: Helm, Kubernetes, et les systèmes de fichier multiniveau
description: Creer un système de fichier complexe via Helm pour vos pods
tags: [Kubernetes, Docker]
image: /post/2020/03/2020-03-18-helm-filesystem-multin-niveau.png
---

Dans cet article nous allons traiter de la mise en place d'un système de fichier multiniveau au sein d'une image docker dans un contexte Kubernetes, avec un déploiement kind.

# Problématique

Bien souvent, il convient lors du déploiement d'une application sous Kubernetes de s'assurer que les fichiers de configuration applicatifs soient présents sur l'image. Certes, il n'est pas préconisé d'injecter de la configuration par système de fichier (on préfère souvent les variables d'environnement), mais quand on porte une application sous k8s c'est hélas souvent une problématique à laquelle on est confronté. D'autant que les dit fichiers doivent évolués en fonction des environnements de déploiement.

Une approche simple consisterait à mettre en place une config map par fichier, mais, le travail peut être long et fastidieux quand on est dans le cas de plusieurs dizaines de fichiers.

Je vous présente donc ici la solution que j'ai mise en oeuvre dans de multiples projets pour me simplifier la tâche.

# Prérequis

Voici la liste de prérequis pour le ce tutoriel:

1. disposer d'un environnement Kubernetes, nous utiliserons [k3d](https://k3d.io/) ici
2. avoir des connaissances sur la solution de déploiement HELM
3. maitriser les concepts Kubernetes de configuration map / déploiement / ...

# Solution

Nous allons mettre en place un mécanisme de construction dynamique de configuration map que nous allons ensuite monter dans une image. Les fichiers présentés dans la configuration map seront des fichiers texte dans lequel il conviendra de changer le contenu en fonction de valeur définie dans le fichier de values.yaml du chart.

# Définition de l'attendu

Pour illustrer notre configuration map multiniveau nous allons construire dans un container le filesystem suivant:

```bash
App
 ┣ subfolder
 ┃ ┗ file3.cfg
 ┣ file1.cfg
 ┗ file2.cfg
```

Dans le file1.cfg nous allons faire en sorte d'injecter des données de configuration en provenance du fichier de configuration du chart : le fichier values.yaml

# Étape du tutoriel

## Installation de k3d

Après avoir installé l'outil k3d en suivant la [procédure d'installation](https://k3d.io), vérifiez la bonne installation de l'outil en lançant la commande suivante:

```bash
k3d --version
```

Vous devez obtenir le résultat suivant:

```bash
k3d version v3.2.0
k3s version v1.18.9-k3s1 (default)
```

Créer votre premier cluster en lançant la commande

```bash
k3d cluster create mycluter
```

qui vous donne le résultat suivant:

![Création cluster k3d](/blog/images/post/2020/03/2020-03-18-helm-filesystem-multin-niveau-1.png)

vous devez pouvoir voir la liste des pods de votre tout nouveau cluster en lançant la commande suivante:

```bash
kubectl get pods --all-namespaces
```

qui vous donne ce résultat:

![Liste des pods du cluster](/blog/images/post/2020/03/2020-03-18-helm-filesystem-multin-niveau-2.png)

## Installation de HELM3

La procédure d'installation de l'outil est présente ici [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/).

Dans mon cas je suis sur une machine Ubuntu je lance donc simplement le script suivant comme indiqué dans la documentation:

```bash
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

vous pouvez vérifier que helm3 est correctement installé en lançant la commande suivante:

```bash
helm version
```

Qui doit vous retourner l'information suivante:

```bash
version.BuildInfo{Version:"v3.5.1", GitCommit:"32c22239423b3b4ba6706d450bd044baffdcf9e6", GitTreeState:"clean", GoVersion:"go1.15.7"}
```

## Création du chart à partir du modèle

Passons maintenant à la mise en place du chart helm à proprement dit. Créez un dossier de projet, dans lequel nous allons travailler.

```bash
mkdir ~/home/multilevel && cd ~/home/multilevel
```

et lancer la commande de création du chart Helm:

```bash
helm create multilevelcm
```

Cette commande crée le squelette du chart dans le dossier `~/home/multilevel/`.

vous obtenez la structure de fichier suivante:

```bash
📦multilevelcm
 ┣ 📂charts
 ┣ 📂templates
 ┃ ┣ 📂tests
 ┃ ┃ ┗ 📜test-connection.yaml
 ┃ ┣ 📜NOTES.txt
 ┃ ┣ 📜_helpers.tpl
 ┃ ┣ 📜deployment.yaml
 ┃ ┣ 📜hpa.yaml
 ┃ ┣ 📜ingress.yaml
 ┃ ┣ 📜service.yaml
 ┃ ┗ 📜serviceaccount.yaml
 ┣ 📜.helmignore
 ┣ 📜Chart.yaml
 ┗ 📜values.yaml
```

il s'agit d'un chart de base qui crée déploie un serveur nginx.

Pour plus d'information sur l'anatomie des chart la documentation est ici [https://helm.sh/docs/helm/helm_create/](https://helm.sh/docs/helm/helm_create/).

## Création de la configuration map dynamique

### création de la structure de fichier dans le chart

Dans le dossier de chart, créer un dossier files qui contient la structure de notre dossier. Vous devez obtenir la structure de fichier suivant:

```bash
📦multilevelcm
 ┣ 📂charts
 ┣ 📂files
 ┃ ┣ 📂subfolder
 ┃ ┃ ┗ 📜file3.cfg
 ┃ ┣ 📜file1.cfg
 ┃ ┗ 📜file2.cfg
 ┣ 📂templates
 ┃ ┣ 📂tests
 ┃ ┃ ┗ 📜test-connection.yaml
 ┃ ┣ 📜NOTES.txt
 ┃ ┣ 📜_helpers.tpl
 ┃ ┣ 📜deployment.yaml
 ┃ ┣ 📜hpa.yaml
 ┃ ┣ 📜ingress.yaml
 ┃ ┣ 📜service.yaml
 ┃ ┗ 📜serviceaccount.yaml
 ┣ 📜.helmignore
 ┣ 📜Chart.yaml
 ┗ 📜values.yaml
```

Dans les fichiers `file2.cfg` et `file3.cfg` mettez le contenu que vous souhaitez. Dans le `file1.cfg`, positionnez le contenu suivant:

```bash
{% raw %}
{{ .Values.configuration.file1 }}
{% endraw %}
```

Ceci permettra au moteur de template d'aller chercher la valeur dans le fichier de chart. Pour cela à la fin du fichier values.yaml, ajoutez le contenu suivant:

```yaml
configuration:
  file1: |-
    contenu du fichier 1
```

### création de la configmap

Dans le dossier du Chart, créer un dossier configmap pour loger le fichier de définition de notre objet.

dans ce fichier, positionner le contenu suivant:

```yaml
{% raw %}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "multilevelcm.fullname" . }}-config
  labels:
    {{- include "multilevelcm.labels" . | nindent 4 }}
data:
  {{ $root := . }}
  {{- range $path, $bytes := $root.Files.Glob (printf "files/**") -}}
  {{ $name := base $path }}
  {{- sha256sum (printf "%s" (index (regexSplit "files/" ($path) -1 ) 1))}}{{ print ": |- " }}
    {{- tpl ($root.Files.Get ($path)) $ | nindent 4 }}
  {{end }}
{% endraw %}
```

l'usage de la fonction sha256sum permet de s'affranchir des caractères spéciaux éventuels dans le nom du fichier. La fonction tpl permet de considérer les fichiers comme des templates helm et donc d'y injecter des données en provenance du helm chart.

## montage de la configuration map dans le pod

Il reste a monter la configuration map en tant que volume dans le container. Pour ce faire, éditer le fichier de déploiement pour y ajouter les éléments suivants:

Dans la partie du container, ajouter la déclaration du module:

```yaml
containers:
  - name: {{ .Chart.Name }}
      securityContext:
      {{- toYaml .Values.securityContext | nindent 12 }}
      image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart  AppVersion }}"
      imagePullPolicy: {{ .Values.image.pullPolicy }}
      ports:
      - name: http
        containerPort: 80
        protocol: TCP
      livenessProbe:
      httpGet:
        path: /
        port: http
      readinessProbe:
      httpGet:
        path: /
        port: http
      resources:
      {{- toYaml .Values.resources | nindent 12 }}
      volumeMounts:
      - name: config
        mountPath: "/app"
```

puis à la fin du fichier ajouter la déclaration du volume:

```yaml
{% raw %}
volumes:
- name: config
  configMap:
    name: {{ include "multilevelcm.fullname" . }}-config
    items:
    {{- range $path, $bytes := .Files.Glob (printf "files/**") -}}
    {{ $name := base $path }}
        - key: {{ sha256sum (printf "%s" (index (regexSplit "files/" ($path) -1) 1)) }}
        path: {{ printf "%s" (index (regexSplit "files/" ($path) -1) 1) }}
    {{- end}}
{% endraw %}
```

## Résultat

Notre chart est pret, pour le déployer sur le cluster k3d. positionnez vous dans votre dossier ~/multilevel et lancer la commande suivante:

```bash
helm install -name myrelease -n myrelease --create-namespace ./multilevelcm/
```

Avec l'outil [lens](https://k8slens.dev/) connectez vous sur votre cluster, puis entrez dans votre pod. En navigant dans le dossier /app, vous constatez que tout vos fichiers sont la et que le contenu de votre fichier file1.cfg corresponds à ce que vous avez mis dans le fichier de values.

![resultat](/blog/images/post/2020/03/2020-03-18-helm-filesystem-multin-niveau-3.png)

le code du chart est disponible sur mon [github](https://github.com/matthieupetite/kubernetes-multilevel-configurationmap)
