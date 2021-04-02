---
title: Ingenieur Cloud
tags: [iptv, reseau, déploiement, numericable, adsl, supervision]
image: swatchgroup-1.png
compagny: Orange Application For Buiness - Swatchgroup
start: Decembre 2007
end: Juin 2007
order: 8
logo: swatch.png
---

Dans le cadre d'un développement de produit pour le monde de l'horlogerie, je suis en charge de construire la solution d'application lifecycle management qui couvrent d'une part le déploiement applicatif (Solution à base de microservice), le déploiement de l'infrastructure Kubernetes (~30 clusters worldwide) et enfin de l'infrastructure de supervision fonctionnelle (Cluster ELK) et technique (Déploiement Prométhéus / Elk / Grafana / Thanos).

La forge livrée contient donc l'ensemble des builds applicatives et des builds liées à l'IaC, ainsi que les release pipeline pour déployer, dans le monde entier, les clusters K8s derrière un Azure Traffic Manager.

La forge est construite de telle sorte que l’ensemble des plateformes (dev, integration, UAT,…) soient construites automatiquement, en injectant par paramétrage, les différents besoins en termes de topologie.

La prestation a couvert en amont la définition de l'infrastructure Azure, mais aussi la sélection du mode de gestion des clusters K8s au sein de la forge. J'ai donc opté pour l'usage de Chart Helm afin de faciliter les déploiements.

Enfin, j’ai mis en place la solution de tests de charge afin de valider les processus d’horizontal auto scaling défini au sein des charts Helm.

D’un point de vue opérationnel, je pilote une équipe de 4 personnes en charge de la construction des éléments d’infrastructure et de supervision

Environnements techniques et méthodologiques:

1. Docker, Helm, Azure ACR,
2. Azure Dev/Ops, Azure (IaaS), Azure (PaaS), ARM, Az Cli, Powershell
3. Java ,
4. Linux,
5. Azure VPN, Application Gateway, Traffic Manager,
6. Microsoft Load Testing,
7. Grafana, Prometheus, Thanos, CouchDB, …
