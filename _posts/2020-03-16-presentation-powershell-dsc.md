---
layout: post
comments: true
title: Présentation de Powershell DSC
<<<<<<< Updated upstream
tags: [Powershell, DevOps]
=======
description: Article de presentation de powershell DSC, l'outils MS pour les devops
tags: [powershell, devOps]
>>>>>>> Stashed changes
image:  /post/2019/01/presentation-powershell-dsc.png
---

Et voilà, je suis **tombé en amour**, comme le disent nos amis francophones d'outre-Atlantique... Et quand on "tombe en amour" et bien on a tout simplement envie d'en parler.

Mon amour du jour s'appelle **DSC**, comme **Desired State Configuration**.

Je vais vous présenter dans cet article, comment DSC va vous sauver la vie et je l'espère vous donner envie de vous jeter à l'eau en montrant un exemple pratique de mise en oeuvre.

Cet article est le premier d'une série qui je l'espère qui vous présentera :

- Comment réaliser votre première configuration DSC
- Le fonctionnement des modules et des ressources DSC
- La création de vos premiers modules custom, parce que oui, tout n'a pas encore été fait sur GitHub
- Le test de vos configurations
- Comment distribuer vos configurations sur votre infrastructure
- Superviser la bonne application de vos configurations

# Desired State Configuration, rapide présentation

On observe depuis quelques années une accélération dans notre petit monde de l'IT.
Il faut délivrer plus vite, mettre rapidement des applications en ligne et donc des plateformes dans les mains des utilisateurs pour rapidement avoir leurs retours.

Cette accélération est un vrai défi pour les développeurs qui n'ont plus le temps de tester, de valider, et même de spécifier.
**L'agilité**, la pratique du **test**, et l'automatisation ont été la réponse face à ces nouvelles contraintes.

Si dans le domaine du développement d'application, la question est ancienne, les réponses techniques relativement matures, et l'outillage présent avec des outils de type [Application Lifecycle Management](https://azure.microsoft.com/fr-fr/services/devops/), dans le domaine de l'infrastructure, les réponses ont un peu plus tardé à venir.

C'est avec l'émergence de plateformes cloud telles que [Azure](https://azure.microsoft.com) ou [Amazon](https://aws.amazon.com) que les choses sont devenues possibles.
Et avec **Desired State Configuration**, Microsoft nous donne l'outil qui permet de répondre à ces enjeux au niveau de l'OS.

Déployer de l'infrastructure de façon testable, automatisée, prédictible le tout en proposant enfin une solution face au cauchemar des OPS : la "dérive de configuration", c'est la promesse de DSC face aux enjeux du **Dev/OPS**.

L'objectif de **DSC**, c'est donc de vous permettre de décrire vos infrastructures à l'aide d'une **syntaxe de script** que vous pourrez appliquer sur vos serveurs de façon périodique afin de vous assurer que ce qui est effectivement déployé correspond à ce que vous avez inscrit au référentiel.

Et comme la description d'une configuration c'est du code vous avez la possibilité de:

- **Gérer** votre infrastructure dans un référentiel de code source et ainsi faire le lien avec les applications,
- **Tester** votre infrastructure,
- **Automatiser** les déploiements des changements de configuration
- **Auditer** la bonne application des configurations

# Linux, Windows: DSC est disponible

Au passage, l'utilisation de DSC n'est pas réduite à la seule plateforme Windows.
En effet, il est possible d'utiliser cette technologie sur un large panel de [distribution Linux](https://docs.microsoft.com/fr-fr/powershell/dsc/getting-started/lnxgettingstarted).

# Fonctionnement général

Comme évoqué plus haut, une configuration DSC c'est avant tout un script powershell (extension .ps1) dans lequel on retrouve le mot clé "Configuration".
Ce fichier s'appuie sur des ressources que nous pouvons importer afin de manipuler l'état du serveur cible.

Une configuration peut décrire l'état désiré d'un ensemble de serveurs. Cette approche permet de mutualiser les configurations ce qui est très utile dans le cas de déploiement de clusters ou pour "**masteriser**" un large volume de serveurs.

Le fichier ```ps1``` de configuration n'est pas directement utilisé pas l'agent DSC déployé sur le serveur qui applique les configurations DSC. Il convient de "**compiler**" ces dernières afin de produire un fichier Mof par serveur.

Le schéma de production d'une configuration DSC est donc le suivant:

![presentation-powershell-dsc-03](/blog/images/post/2019/01/presentation-powershell-dsc-03.png)

Une fois la configuration compilée, c'est le **local configuration manager** du serveur qui va pouvoir appliquer le fichier ```MOF``` résultat de la compilation de notre configuration.

Tout serveur Windows depuis la version 2012 exécute l'agent LCM, et c'est là toute la force de la technologie DSC puisqu'aucun ajout n'est nécessaire pour interpréter nos configurations.
Cette fonctionnalité est native ce qui nous affranchit de toute configuration supplémentaire.

# Présentation de la structure d'une configuration

Rentrons maintenant un peu plus dans le détail de ce qu'est notre configuration DSC.
Comme évoqué plus haut, une configuration est un simple fichier powershell.

```powershell
Configuration MaPremiereConfiguration {

  Import-Module -ModuleName xTimeZone

  Node MonPremierServeur {
    xTimeZone 'FrenchTimeZone' {
      IsSingleInstance = 'Yes'
      TimeZone = 'Romance Standard Time'
    }
  }

}
```

Voyons en détail, sur cet exemple anodin, les objets qui sont manipulés.

| Éléments de configuration                                   | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Configuration                                               | Comme vous le constatez, une configuration DSC commence toujours par le mot clé du langage Powershell **Configuration**. Ce mot clé est de façon sous-jacente au langage une fonction powershell que l'on pourra appeler pour produire les fichiers Mof.                                                                                                                                                                                                                                                                                            |
| MaPremiereConfiguration                                     | Cet élément est simplement le nom de la configuration DSC                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| Import-Module -ModuleName xTimeZone                         | Cette directive powershell permet de charger les ressources DSC dont dépend notre configuration. Une ressource DSC est un module powershell qui doit être installé sur le serveur sur lequel on souhaite appliquer notre configuration. C'est dans ces modules que l'on retrouvera toute la logique d'installation et de configuration du serveur que l'on souhaite appliquer. La communauté met à disposition un [large panel de modules open source](https://github.com/PowerShell/DscResources) qui vous permettrons de configurer vos serveurs. |
| Node MonPremierServeur                                      | Nous déclarons ici le serveur pour lequel la configuration doit être appliquée. Il faut lire cette directive comme suit: si le serveur possède un hostname égal à "MaPremiereConfiguration" alors la configuration sera appliquée. Dans le cas inverse, elle sera simplement ignorée. Cette approche vous permet donc de créer une seule configuration pour un ensemble de serveurs et d'appliquer ou non des ressources DSC en fonction de vos besoins.                                                                                            |
| xTimeZone "FrenchTimeZone"                                  | Cette ligne de configuration précise que dans notre configuration nous faisons appel à la ressource **xTimeZone** qui comme son nom l'indique nous permet de configurer la localisation temporelle de notre serveur afin de gérer l'heure convenablement. Si vous souhaitez voir le code de cette ressource, il vous suffit de vous reporter au contenu du module que nous avons déclaré plus haut.                                                                                                                                                 |
| IsSingleInstance = "Yes" TimeZone = "Romance Standard Time" | Enfin, ces deux derniers éléments constituent les paramètres nécessaires au module DSC xTimeZone                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

# Mode d'application d'une configuration DSC

La cinématique d'appel d'une configuration DSC est la suivante. Pour chaque ressource présente dans la configuration, le local configuration manager va appeler en séquence trois méthodes powershell:

1. Get-TargetResource
2. Test-TargetResource
3. Set-TargetResource

Comme vous l'aurez compris, toutes les ressources DSC doivent implémenter ces fonctions afin qu'elles puissent être appliquées par le local configuration manager.

## Get-TargetResource

Cette première méthode permet de récupérer le composant que nous volons paramétrer.
Ainsi le module xTimeZone récupère la configuration courante de la TimeZone du serveur sur lequel nous voulons appliquer notre configuration.

## Test-TargetResource

Cette méthode teste l'état du serveur courant vis-à-vis de la configuration attendue.
Deux cas peuvent se présenter alors:

1. Si l'état courant du serveur est conforme à celui demandé dans la configuration, la fonction Test-TargetResource retourne alors la valeur ```$true```
2. Si l'état courant n'est pas dans l'état désiré, alors elle retourne la valeur ```$false```.

En fonction du retour de cette méthode, le Local Configuration Manager exécute ou non la fonction ```Set-TargetResource```.

## Set-TargetResource

Cette dernière méthode réalise effectivement la configuration du serveur. Dans notre cas, elle configure la TimeZone du serveur en "**Romance Standard Time**".

En fin de cycle, le serveur est donc configuré tel que décrit dans le référentiel.

# Et maintenant, parlons déploiement des configurations

Maintenant que vous maîtrisez la logique globale d'une configuration DSC, passons à la pratique.

## La compilation

En premier lieu, il convient de créer et **compiler** la configuration DSC. Pour ce faire, nous allons partir de l'exemple suivant:

```powershell

  Configuration ConfigurationTest {
    node Localhost {
      File DossierExemple {
        Ensure = 'Present'
        Type = 'Directory'
        DestinationPath = 'C:\temp\test'
      }
    }
  }

```

Copier le contenu de ce fichier dans votre éditeur Powershell préféré et sauvegardez-le dans le dossier ```c:\temp``` sous le nom ```ConfigurationTest.ps1```

![presentation-powershell-dsc-04](/blog/images/post/2019/01/presentation-powershell-dsc-04.png)

Ensuite, il faut que vous "**dotsourciez**" le fichier afin de charger la configuration DSC que vous venez de créer dans votre session powershell.

![presentation-powershell-dsc-05](/blog/images/post/2019/01/presentation-powershell-dsc-05.png)

Vous avez maintenant dans votre Shell la possibilité d'appeler la fonction ```ConfigurationTest``` pour compiler votre configuration.

![presentation-powershell-dsc-06](/blog/images/post/2019/01/presentation-powershell-dsc-06.png)

Et voilà, la configuration est maintenant prête à être appliquée par le local configuration manager.
Comme vous pouvez le constater, l'action de configuration a créé un sous-dossier dans le répertoire ```c:\temp``` nommé ConfigurationTest et contenant le fichier Mof, résultat de la configuration.

![presentation-powershell-dsc-07](/blog/images/post/2019/01/presentation-powershell-dsc-07.png)

## Le déploiement

Maintenant, passons au déploiement effectif de notre configuration sur un serveur.
Il existe deux modes d'application des configurations:

1. Le mode Push
2. Le mode pull

### Le mode Push

Dans le cadre du déploiement en mode **Push**, c'est l'administrateur du système qui localement applique la configuration sur le serveur.
C'est l'approche la plus simple.
À chaque modification de la configuration, l'administrateur du système doit réappliquer manuellement la configuration.

La commande utilisée pour réaliser cette opération est la suivante:

```powershell
$spats = @{
  ComputerName = '<nom du serveur cible>'
  Path         = '<Chemin du dossier contenant le fichier Mof'
  Wait         = $true
  Force        = $true
  Verbose      = $true
}
Start-DscConfiguration @spats
```

L'application de la "**ConfigurationTest**" que nous avons créé plus haut est donc déclenchée comme suit:

![presentation-powershell-dsc-08](//dev-portfolio-blogassets/img/post/2019/01/presentation-powershell-dsc-08.png)

Comme le montre l'image ci-dessus, qui présente les traces des actions du local configuration manager, nous pouvons observer que ce dernier à d'abord appelé la méthode **Test** de la ressource **File**.

Cette dernière a vérifié la présence sur le disque d'un dossier nommé ```c:\temp\test```.
Ce dernier n'existait pas, en conséquence la directive Set a été appelée afin de créer le dossier.

Lors du deuxième appel de la configuration DSC, nous pouvons constater que la méthode Set a été "**Skip**"

![presentation-powershell-dsc-09](/blog/images/post/2019/01/presentation-powershell-dsc-09.png)

### Le mode pull

Si le mode Push offre un premier niveau de réponse, lors de l'usage de DSC sur une infrastructure comprenant un grand nombre de serveurs vous serez vite confronté à certaines **limites**:

- L'application des configurations en mode Push impose une **action de l'administrateur** pour appliquer les nouvelles versions de configuration
- Vous devez **déployer à la main** les modules de ressource DSC dont a besoin votre configuration sur l'ensemble des serveurs ciblés
- Vous n'avez pas accès à du reporting sur le statut de l'application de vos configurations de façon centralisée.

Le mode Pull est donc pour vous.
En effet dans ce mode de configuration, le local configuration manager de vos serveurs pointe vers un serveur de configuration central nommée **pull serveur**.
Sur une base de temps configurable, il interroge ce dernier pour récupérer la dernière version de configuration DSC qui lui est assignée et éventuellement télécharger les modules nécessaires avant l'application de la configuration.

![presentation-powershell-dsc-10](/blog/images/post/2019/01/presentation-powershell-dsc-10.png)

Nous verrons dans un prochain article comment mettre en oeuvre un pull serveur au sein de votre infrastructure.