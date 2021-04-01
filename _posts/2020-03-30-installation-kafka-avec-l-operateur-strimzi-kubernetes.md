---
layout: post
comments: true
title: Installation de Kafka avec l opérateur Strimzi sur Kubernetes
tags: [kubernetes, kafka, strimzi, helm]
image: /post/2020/03/2020-03-30-installation-kafka-strimzi-kubernetes.png
---

Dans cet article nous allons détailler l'installation de l'application Kafka a l'aide de l'[opérateur Strimzi](https://strimzi.io/). En effet, avec l'arrivée de plus en plus régulière d'application Cloud Native, les experts Dev/Ops sont de plus en plus amenés à déployer Kafka afin d'assurer la communication entre les différentes API. Le faire dans un contexte Kubernetes est de plus en plus régulier. Nous allons donc déployer, à l'aide de l'outil [Helm3](https://helm.sh/), un cluster Kafka au sein d'un cluster Kubernetes [K3d](https://k3d.io/).

## Qu'est-ce que Strimzi

Strimzi fournit le moyen d'exécuter un cluster Apache Kafka sur Kubernetes dans diverses configurations de déploiement. Pour le développement, il est facile de configurer un cluster dans K3d par exemple en quelques minutes. Pour la production, vous pouvez personnaliser le cluster en fonction de vos besoins, en utilisant des fonctionnalités telles que le rack awarness pour répartir les brokers Kafka entre différentes zones de disponibilité, et les fonctionnalités de "Taints and Tolerations" pour exécuter Kafka sur des nœuds dédiés de votre cluster Kubernetes. Vous pouvez exposer Kafka en dehors de Kubernetes à l'aide de NodePort, Load Balancer, Ingress et/ou des routes OpenShift, selon de vos besoins, et ceux-ci sont facilement sécurisés à l'aide de TLS.

La gestion native de Kafka ne se limite pas au broker. Vous pouvez également gérer les topics Kafka, les utilisateurs, Kafka MirrorMaker et Kafka Connect à l'aide de ressources personnalisées. Cela signifie que vous pouvez utiliser vos processus et outils Kubernetes familiers pour gérer des applications Kafka complètes.

Utiliser Strimzi c'est aussi bénéficier d'un ensemble de Chart qui vous permette, via Helm de manager vos ressources Kafka.

## Objectif du tutoriel

Dans le cadre de cet article nous allons déployer un cluster Kafka, Kafka bridge et un topic nommé `TopicTest` dans un cluster Kubernetes au sein d'un namespace que nous allons nommer `Kafka`. L'ensemble de ces éléments sera créé à l'aide du déploiement d'un chart Helm qui de façon sous-jacente utilisera l'opérateur strimzi après l'avoir déployé.

## Prérequis

Vous devez avoir accès à un cluster Kubernetes sur votre poste de développement. Dans le cadre de cet article nous allons utiliser K3d, mais vous pouvez de même utiliser [Kind](https://kind.sigs.k8s.io/) ou [Minikube](https://kubernetes.io/fr/docs/setup/learning-environment/minikube/).

Vous devez aussi avoir les connaissances de base autour de la création de Chart Helm.

## Ajout de la registry de chart helm strimzi

Dans un terminal lancer la commande suivante:

```bash
helm repo add strimzi https://strimzi.io/charts/
```

Vous pouvez vérifier que la registry a bien été ajoutée en lançant la commande suivante:

```bash
helm repo list
```

qui doit vous donner le résultat suivant:

![Helm repo list](/blog/images/post/2020/03/2020-03-30-installation-kafka-strimzi-kubernetes-1.png)

## Création du squelette de notre Chart

Dans un terminal lancer la commande suivante afin de créer le squelette de notre chart:

```bash
helm3 create kafka
```

Vous devez obtenir l'arborescence suivante, qui correspond à un chart de base:

```bash
📦kafka
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

Supprimer les fichiers inutiles, car nous n'avons pas besoin de déployment, hpa, ingress, service ou encore service account. Après l'étape de nettoyage, notre chart correspond à ceci:

```bash
📦kafka
 ┣ 📂charts
 ┣ 📂templates
 ┃ ┣ 📂tests
 ┃ ┗ 📜_helpers.tpl
 ┣ 📜.helmignore
 ┣ 📜Chart.yaml
 ┗ 📜values.yaml
```

## Ajout de la dépendance au chart de l'opérateur Strimzi

Comme dit plus haut, le déploiement de l'opérateur Strimzi (en réalité des opérateurs) peut se faire à l'aide d'un chart Helm. Nous allons donc utiliser le mécanisme de dépendance de chart pour forcer l'installation de l'opérateur avant le déploiement de notre chart.

Dans le fichier Chart.yaml, ajouter le contenu suivant:

```yaml
dependencies:
- name: strimzi-kafka-operator
  version: "0.21.1"
  repository: "https://strimzi.io/charts/"
```

## Déclaration du Kafka cluster, du topic et du Kafka bridge

### Édition de votre fichier values.yaml

Remplacer le contenu de votre fichier Values.yaml par le contenu suivant:

```yaml
serviceAccount:
  create: true
  annotations: {}
  name: "kafka-cluster-serviceaccount"

kafka:
  name: 'Kafka-cluster'
  replicas: 3
  config:
    offsets:
      topic:
        replication:
          factor: 3
    transaction:
      state:
        log:
          replication:
            factor: 3
          min:
            isr: 2
  storage:
    volume:
      size: 1Mi

zookeeper:
  replicas: 3
  storage:
    size: 1Mi

topic:
  partitions: 3
  replicas: 3
  retention:
    ms: 7200000
  segment:
    bytes: 1073741824
  topics:
  - name: 'topictest'

kafkaMonitoring:
  enabled: true

prometheus-kafka-exporter:
  image:
    repository: "danielqsj/Kafka-exporter"
```

### Création du Kafka cluster

Créé un fichier kafkacluster.yaml dans le dossier template de votre chart avec le contenu suivant:

```yaml
{% raw %}
---
apiVersion: kafka.strimzi.io/v1beta1
kind: Kafka
metadata:
  name: {{ .Values.kafka.name }}
spec:
  kafka:
    version: 2.6.0
    replicas: {{ .Values.kafka.replicas }}
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: {{ .Values.kafka.config.offsets.topic.replication.factor }}
      transaction.state.log.replication.factor: {{ .Values.kafka.config.transaction.state.log.replication.factor }}
      transaction.state.log.min.isr: {{ .Values.kafka.config.transaction.state.log.min.isr }}
      log.message.format.version: "2.6"
      inter.broker.protocol.version: "2.6"
    storage:
      type: jbod
      volumes:
      - id: 0
        type: persistent-claim
        size: {{ .Values.kafka.storage.volume.size }}
        deleteClaim: false
  zookeeper:
    replicas: {{ .Values.zookeeper.replicas }}
    storage:
      type: persistent-claim
      size: {{ .Values.zookeeper.storage.size }}
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
{% endraw %}
```

### Création du topic

Créer un fichier kafkatopic.yaml dans le dossier template de votre chart avec le contenu suivan

```yaml
{% raw %}
{{- $root := . -}}
{{- range $.Values.topic.topics }}
---
apiVersion: kafka.strimzi.io/v1beta1
kind: KafkaTopic
metadata:
  name: {{ .name | lower | quote }}
  labels:
    strimzi.io/cluster: {{ $root.Values.kafka.name }}
spec:
  partitions: {{ $root.Values.topic.partitions }}
  replicas: {{ $root.Values.topic.replicas }}
  config:
    retention.ms:  {{ $root.Values.topic.retention.ms }}
    segment.bytes:  {{ $root.Values.topic.segment.bytes }}
{{- end}}
{% endraw %}
```

Ce fichier va looper sur la variable du topic.topics du fichier de value pour demander la création de notre topic.

### Création du Kafka bridge

Dans le dossier template de votre chart créé un fichier kafkabridge.yaml et positionner le contenu suivant:

```yaml
{% raw %}
---
apiVersion: kafka.strimzi.io/v1alpha1
kind: KafkaBridge
metadata:
  name:  {{ .Values.kafka.name }}-bridge
spec:
  replicas: 1
  bootstrapServers:  {{ .Values.kafka.name }}-kafka-bootstrap:9092
  http:
    port: 8080
{% endraw %}
```

### Mise à jour des dépendances de chart

Avant de déployer votre chart il convient de mettre à jour les dependances de ce dernier. Pour cela lancer la commande suivante dans un terminal:

```bash
helm dependency update ./Kafka/
```

Vous devez obtenir le résultat suivant:

![helm repository update](/blog/images/post/2020/03/2020-03-30-installation-kafka-strimzi-kubernetes-2.png)

Cette commande a pour effet de télécharger le chart le dossier /charts de votre Chart.

## installation de votre chart et résultat

Pour installer votre chart lancer la commande suivante:

```bash
helm install -n Kafka Kafka ./Kafka/ --create-namespace
```

Après quelques minutes vous constaterez dans votre cluster que l'ensemble des Pods Kafka sont là:

1. l'opérateur strimzi
2. le cluster zookeeper dédié à l'infrastructure kakfa
3. le cluster Kafka lui-même
4. le pod pour le bridge Kafka

![resultat installation kafka](/blog/images/post/2020/03/2020-03-30-installation-kafka-strimzi-kubernetes-3.png)

et que les customs ressources de l'opérateur strimzi sont présentes:

![custom ressource strimzi](/blog/images/post/2020/03/2020-03-30-installation-kafka-strimzi-kubernetes-4.png)

Pour avoir accès au code source de cet exemple, c'est par ici [https://github.com/matthieupetite/helm-samples](https://github.com/matthieupetite/helm-samples)