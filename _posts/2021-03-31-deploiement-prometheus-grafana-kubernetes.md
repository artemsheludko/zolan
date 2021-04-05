---
layout: post
comments: true
title: Deploiement de la suite prometheus grafana sous Kubernetes
tags: [kubernetes, prometheus, grafana, k3d, helm]
image: /post/2021/03/2021-03-31-deploiement-prometheus-grafana-kubernetes.png
summary: "Superviser le cloud est un enjeu majeur lorsque l on décide d y déployer une application métier, Prometheus, Grafana s imposent comme des outils essentiels pour répondre aux enjeux de monitoring. Voici comment les déployer dans un environnement Kubernetes."
---

Superviser le cloud est un enjeu majeur lorsque l'on décide d'y déployer une application métier. En effet, quel intérêt de bénéficier d'une disponibilité de 5*9 au niveau de l'infrastructure si la couche applicative n'est pas surveillée et n'offre pas le même niveau de continuité de service. À l'heure où les architectures IT sont évolutives, où les principes Dev/Ops de CI/CD se généralise, la DSI a besoin d'outils de supervision agiles et performants.  [Prometheus](https://go2.grafana.com/prometheus-grafana.html), [Grafana](https://go2.grafana.com/prometheus-grafana.html) s imposent comme des outils essentiels pour répondre à ces enjeux pour fournir des services de monitoring et d'alerting. Voici comment les déployer dans un environnement Kubernetes.

## Prometheus / Grafana : Architecture

On parle souvent de Prométheus et Grafana, mais en réalité il s'agit d'une suite d'outillage plus complète comme le montre le diagramme d'architecture ci-dessous:

![Prometheus Grafana architecture](/blog/images/post/2021/03/2021-03-31-deploiement-prometheus-grafana-kubernetes-1.png)

Les éléments constitutifs de la plateforme sont donc:

1. Le prometheus serveur: qui collecte la donnée de supervision et les stocke
2. La partie grafana qui assure la partie visualisation des données
3. La partie PushGateway dont le rôle est d'assurer la collecte de données de supervision qui serait collectée en mode push (en effet le mode de supervision de prometheus est le pull par défaut)
4. la partie Alert manager qui permet de diffuser les événements pour les équipes de supervision.
5. Les nodes exporter, qui constitue la partie collecte des données en mode pull.

On peut noter que pour le déploiement de cette chaine de supervision dans une réel environnement de production il convient souvent d'ajouter les composants suivants:

1. [Blackbox](https://geekflare.com/fr/monitor-website-with-blackbox-prometheus-grafana/): pour la supervision des performances de site web
2. [Thanos](https://thanos.io/): pour la gestion avancée du stockage à long terme

## Objectifs du tutoriel

Dans ce tutoriel nous allons installer la suite prometheus grafana sur un cluster Kubernetes afin de monitorer les performances de celui-ci. Nous allons pour réaliser ces travaux produire un Chart Helm qui nous donnera la possibilité d'ajouter nos propres tableaux de bord Grafana.

## Prérequis

Vous devez avoir accès à un cluster Kubernetes sur votre poste de développement. Dans le cadre de cet article nous allons utiliser K3d, mais vous pouvez de même utiliser [Kind](https://kind.sigs.k8s.io/) ou [Minikube](https://kubernetes.io/fr/docs/setup/learning-environment/minikube/).

Vous devez aussi avoir les connaissances de base autour de la création de [Chart Helm](https://helm.sh).

## Création du cluster k3d

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
k3d cluster create mycluster -p "80:80@loadbalancer" --agents 2
```

Noter le paramètre `-p "80:80@loadbalancer"`  qui permettra d'exposer votre ingress controller

## Création du chart à partir d'un modèle et clean du code

Dans un terminal, lancer la commande suivante:

```bash
helm3 create prometheus
```

Vous devez obtenir l'arborescence suivante, qui correspond à un chart de base:

```bash
📦prometheus
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

Supprimer les fichiers inutiles, car nous n'avons pas besoin de déployment, hpa, ingress, service ou encore service account ou encore des tests. Par contre nous avons besoin de configmaps et d'un dossier dashboard à la racine du chart. Après l'étape de nettoyage, notre chart correspond à ceci:

```bash
📦prometheus
 ┣ 📂charts
 ┣ 📂dashboards
 ┣ 📂templates
 ┃ ┗ 📂configmap
 ┣ 📜.helmignore
 ┣ 📜Chart.yaml
 ┗ 📜values.yaml
```

## Spécialisation du chart

### Modification du fichier Chart.yaml

Notre chart va s'appuyer sur celui fournit par la communauté : [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) qui de façon sous-jacente installe plusieurs opérateurs Kubernetes.

Afin de configurer cette dépendance, il convient donc de modifier le fichier `chart.yaml` comme suit:

```yaml
apiVersion: v2
name: prometheus
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"

dependencies:
  - name: kube-prometheus-stack
    version: 14.4.0
    repository: https://prometheus-community.github.io/helm-charts
    import-values:
      - child: grafana
        parent: grafana
```

### Modication du fichier values.yaml

Le fichier de values doit être modifié comme suit:

```yaml
kube-prometheus-stack:
    ## Provide a name in place of kube-prometheus-stack for `app:` labels
    ##
    nameOverride: ""

    ## Override the deployment namespace
    ##
    namespaceOverride: ""

    ## Provide a k8s version to auto dashboard import script example: kubeTargetVersionOverride: 1.16.6
    ##
    kubeTargetVersionOverride: ""

    ## Provide a name to substitute for the full names of resources
    ##
    fullnameOverride: ""

    ## Component scraping the kube controller manager
    kubeControllerManager:
      enabled: false

    ## Component scraping etcd
    kubeEtcd:
      enabled: false

    ## Component scraping kube proxy
    kubeProxy:
      enabled: false

    ## Component scraping kube scheduler
    kubeScheduler:
      enabled: false

    ## Configuration for alertmanager
    ## ref: https://prometheus.io/docs/alerting/alertmanager/
    ##
    alertmanager:
      alertmanagerSpec:
        image:
          repository: quay.io/prometheus/alertmanager
          tag: latest
          sha: ""  
    
    ingress:
    ## If true, Grafana Ingress will be created
    ##
      enabled: true

    ## Annotations for Grafana Ingress
    ##
      annotations: 
      kubernetes.io/ingress.class: nginx
      # kubernetes.io/tls-acme: "true"

    ## Labels to be added to the Ingress
    ##
      labels: {}

    ## Hostnames.
    ## Must be provided if Ingress is enable.
    ##
    # hosts:
    #   - grafana.domain.com
      hosts: 
      - dev-platform

    ## Path for grafana ingress
      path: /grafana/  
    
    ## Manages Prometheus and Alertmanager components
    ##
    prometheusOperator:
      enabled: true
      admissionWebhooks:
        failurePolicy: Fail
        enabled: true
        patch:
          enabled: true
          image:
            repository: jettech/kube-webhook-certgen
            tag: v1.5.1
            sha: ""
            pullPolicy: IfNotPresent
          resources: {}
      ## Prometheus-operator image
      ##
      image:
        repository: quay.io/prometheus-operator/prometheus-operator
        tag: v0.46.0
        sha: ""
        pullPolicy: IfNotPresent

      prometheusConfigReloaderImage:
        repository: quay.io/prometheus-operator/prometheus-config-reloader
        tag: v0.46.0
        sha: ""

    prometheus:
      enabled: true
      ## Configuration for Prometheus service
      ##
      service:
        type: LoadBalancer
      prometheusSpec:
        image:
          repository: quay.io/prometheus/prometheus
          tag: v2.24.0
          sha: ""
        ## External URL at which Prometheus will be reachable.
        ##
        externalUrl: "http://kind-dev/prometheus"
        ## Prefix used to register routes, overriding externalUrl route.
        ## Useful for proxies that rewrite URLs.
        ##
        routePrefix: /prometheus
    grafana:
      adminPassword: "password"
      grafana.ini:
        server:
          domain: cluster-dev.local
          root_url: "%(protocol)s://%(domain)s:%(http_port)s/grafana"
          serve_from_sub_path: true
      image:
        repository: grafana/grafana

      downloadDashboardsImage:
        repository: curlimages/curl

      initChownData:
        image:
          repository: busybox

      sidecar:
        image:
          repository: kiwigrid/k8s-sidecar
      ingress:
        ## If true, Grafana Ingress will be created
        ##
        enabled: true

        ## Annotations for Grafana Ingress
        ##
        #annotations: {
        #  kubernetes.io/ingress.class: nginx,:qa:pod
        #  kubernetes.io/tls-acme: "false"
        #}

        ## Labels to be added to the Ingress
        ##
        labels: {}

        ## Hostnames.
        ## Must be provided if Ingress is enable.
        ##
        # hosts:
        #   - grafana.domain.com
        hosts: 
          - grafana.cluster-dev.local

        ## Path for grafana ingress
        path: /grafana

        ## TLS configuration for grafana Ingress
        ## Secret must be manually created in the namespace
        ##
        tls: []
        # - secretName: grafana-general-tls
        #   hosts:
        #   - grafana.example.com

    kube-state-metrics:
      image:
        repository: quay.io/coreos/kube-state-metrics

    prometheus-node-exporter:
      image:
        repository: quay.io/prometheus/node-exporter
```

Notez la valeur `grafana.cluster-dev.local` que nous avons positionnée de façon a créer des ingress pour accéder à grafana.

### Modification du fichier de configuration map afin d'ajouter les dashboard grafana custom

Dans le dossier `/template/configmap/` de votre chart, veuillez ajouter un fichier yaml pour créer une configmap de configuration des dashboard custom. Le contenu de ce dernier doit être comme suit:

```yaml
{% raw %}
{{- $files := .Files.Glob "dashboards/*.json" }}
{{- if $files }}
apiVersion: v1
kind: ConfigMapList
items:
{{- range $path, $fileContents := $files }}
{{- $dashboardName := regexReplaceAll "(^.*/)(.*)\\.json$" $path "${2}" }}
- apiVersion: v1
  kind: ConfigMap
  metadata:
    name: {{ printf "%s-%s" (include "prometheus.fullname" $) $dashboardName | trunc 63 | trimSuffix "-" }}
    namespace: {{ $.Release.Namespace }}
    labels:
      {{- if $.Values.grafana.sidecar.dashboards.label }}
      {{ $.Values.grafana.sidecar.dashboards.label }}: "1"
      {{- end }}
      app: {{ template "prometheus.name" $ }}-grafana
{{ include "prometheus.labels" $ | indent 6 }}
  data:
    {{ $dashboardName }}.json: {{ $.Files.Get $path | toJson }}
{{- end }}
{{- end }}
{% endraw %}
```

Comme vous pouvez le constater, cette configuration map est créée en parcourant la liste des fichiers json contenus dans le chart lui-même. C'est pour cela que nous avons mis en place un dossier dashboards dans notre chart qui contiendra les fichiers json de définition des tableaux de bord custom. Vous pouvez trouver une liste d'exemple à l'adresse suivante: [https://grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards)

## Déploiement de notre chart

### installation de notre chart

Pour installer votre chart lancer la commande suivante:

```bash
helm install -n prometheus prometheus ./prometheus/ --create-namespace
```

Lancez votre outil de gestion de Kubernetes préféré pour constater la liste des pods qui ont été créés (ici nous utilisons k9s):

![liste de pods prometheus et grafana](/blog/images/post/2021/03/2021-03-31-deploiement-prometheus-grafana-kubernetes-2.png)

## configuration de votre DNS

Afin d'accédé à votre portail grafana il faut que vous modifier votre fichier `/etc/hosts` afin d'y ajouter une entrée pour le host suivant `grafana.cluster-dev.local`. En effet, le service grafana est diffusé par défaut au travers d'un [ingress controller](https://kubernetes.github.io/ingress-nginx/) et d'un ingress. Il faut donc que votre domaine soit connu sur votre machine hôte.

Ajoutez donc l'entrée suivante dans votre fichier /etc/hosts

```bash
127.0.0.1 grafana.cluster-dev.local
```

Il vous suffit ensuite de vous connecter sur l'URL suivante [http://grafana.cluster-dev.local/grafana](http://grafana.cluster-dev.local/grafana) pour avoir accès à la mire de connexion grafana.

![portail grafana](/blog/images/post/2021/03/2021-03-31-deploiement-prometheus-grafana-kubernetes-3.png)

Entrez le login suivant `admin` et le mot de passe `password` pour vous connecter à grafana. Vous aurez ensuite dans la partie manage accès à la liste de vos tableaux de bord.

![dashboard grafana](/blog/images/post/2021/03/2021-03-31-deploiement-prometheus-grafana-kubernetes-4.png)

Pour avoir accès au code source de cet exemple, c'est par ici [https://github.com/matthieupetite/helm-samples](https://github.com/matthieupetite/helm-samples)