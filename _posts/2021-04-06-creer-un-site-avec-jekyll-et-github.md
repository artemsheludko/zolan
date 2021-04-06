---
layout: post
comments: true
title: créer un site avec Jekyll et GitHub
tags: [devops, cicd, website, github, github action]
image: /post/2021/04/2021-04-06-creer-un-site-avec-jekyll-et-github.png
summary: "Gérer votre documentation de repository sur Github est souvent un enjeu pour vous et pour la communauté. Jekyll et les Github Pages sont là pour ça."
---

La documentation n'est pas notre point fort il faut bien l'admettre dans notre petit monde l'IT. Mais qui n'a jamais rencontré de vraies difficultés quand on revient sur un code un an après l'avoir produit. Documentation incomplète, mal organisée, et pas franchement séduisante lors de la lecture sont trop souvent notre lot, surtout quand on développe des sides projects. La solution Github + Jekyll est la pour ça. Je viens récemment de remettre en place un blog et j'ai été enthousiasmé de ce combo. Mes attentes étaient:

1. Avoir une solution ***gratuite***,
2. Ne surtout pas écrire de javascript ou de HTML pour produire mes articles
3. Avoir une solution centrée sur Git et la gestion de code source
4. Ne pas gérer une infrastructure lourde type CMS avec des bases de données et un hébergement complexe
5. Disposer d'une approche CI/CD pour la publication

## Qu'est-ce que Github page

[Github](https://github.com/) offre depuis quelque temps déjà la possibilité d'héberger des sites Web sur l'ensemble de ses dépôts de code, et ce de façon gratuite. Évidemment le site en question doit suivre quelques guidelines et vous devez être averti de certaines contraintes:

1. Le site produit sera un site public, et ce même si votre dépôt de code est un dépôt privé
2. La branche par défaut qui contient le code source de votre site est gh-pages (ce paramètre peut être modifié)
3. Le site que vous allez pouvoir créer doit être constitué de pages statiques
4. L'ensemble des pages sont servies en HTTPS depuis le domaine github.io. 
5. L'utilisation des pages GitHub est sous mise au respect des règles suivantes : [GitHub Terms of Service](https://docs.github.com/en/articles/github-terms-of-service)

Je ne vais pas vous faire l'affront de vous présenter comment on crée un dépôt de code sur cette célèbre plateforme. Une fois votre dépôt de code créé, rendez-vous sur la page de settings.

![settings github](/blog/images/post/2021/04/2021-04-06-creer-un-site-avec-jekyll-et-github-1.png)

Ensuite vous pourrez accéder aux paramètres de gestion des pages comme le montre l'écran si dessous:

![settings github page](/blog/images/post/2021/04/2021-04-06-creer-un-site-avec-jekyll-et-github-2.png)

## Produire le site statique : Jekyll est là

En novembre 2008, Tom Preston-Werner, l'un des quatre fondateurs de la plateforme GitHub publiait le générateur de sites Web Jekyll sous licence de logiciel libre MIT. Écrit en Ruby, ce générateur fonctionne sur la base d’un référentiel de templates qui contient une série de fichiers texte structurés et statiques (markdowns) de différents formats. Ils déterminent non seulement la mise en page, mais aussi le contenu de chaque projet Web. Ils peuvent donc être adaptés aux besoins individuels. Le générateur ne fournit pas l’éditeur WYSIWYG, mais nécessite l'écriture de code classique. À cette fin, il est recommandé d’utiliser l’éditeur vscode et ses plugins, qui simplifie l'édition de Markdown, et qui est aussi optimisé pour Jekyll.

Avant que les modifications apportées au code ne soient incorporées dans la version live de l'application Web, elles peuvent être inspectées grâce au serveur de développement de Jekyll. Le moteur de rendu implémenté s’assure que les fichiers se transforment en un site Web statique (qui peut ensuite être livré avec n'importe quel serveur Web commun). Ceci génère automatiquement le code HTML lorsque les modifications sont apportées aux fichiers texte.

Pour utiliser Jekyll, vous pouvez le télécharger et l’héberger localement sur votre propre ordinateur, mais aussi l'intégré à votre pipeline Github Action.

Voici quelques liens utiles qui m'ont permis de construire mon site:


1. [Documentation Jekyll](https://jekyllrb.com/)
2. [Theme Jekyll](https://jekyllrb.com/docs/themes/):
   1. [GitHub.com #jekyll-theme repos](https://jekyllrb.com/docs/themes/)
   2. [jamstackthemes.dev](jamstackthemes.dev)
   3. [jekyllthemes.org](jekyllthemes.org)
   4. [jekyllthemes.io](jekyllthemes.io)
   5. [jekyll-themes.com](jekyll-themes.com)
3. [Moteur Liquid](https://shopify.github.io/liquid/filters/divided_by/)

## Et la CI/CD ?

Jekyll nous offre donc la possibilité de transformer vos articles écris en markdown et page HTML classique, Github nous permet d'héberger le site Web au sein d'une branche de notre dépôt de code, que nous manque-t-il ? De la fluidité.

Grâce au GitHub action, vous allez pouvoir automatiser le processus de production de votre site Web. À chaque commit sur votre branche main (ou master, ou de documentation), nous allons déclencher un pipeline qui:

1. checkout le code de votre branche
2. Install Jekyll
3. Lance la commande ```Jekyll build``` permettant la production de votre site statique dans un dossier _site de l'agent de build
4. Commit le contenu du dossier _site dans votre dépôt mettant ainsi à jour votre site

Pour cela, dans la branche master de votre site créez le dossier .github et positionner le contenu comme suit:

```yaml
📦.github
 ┗ 📂workflows
 ┃ ┗ 📜github-page.yml
 ```

 Éditez le fichier ```github-page.yml``` pour y ajouter le contenu suivant:

 ```yaml
name: Testing the GitHub Pages publication

on:
  push:
    branches:
      - master

jobs:
  jekyll:
    runs-on: ubuntu-16.04
    steps:
    - uses: actions/checkout@v2
    - name: 💎 setup ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: 2.7 # can change this to 2.7 or whatever version you prefer
        bundler-cache: true

    - name: 🔨 install dependencies & build site
      uses: limjh16/jekyll-action-ts@v2
      with:
        enable_cache: true
          ### Enables caching. Similar to https://github.com/actions/cache.
          #
          # format_output: true
          ### Uses prettier https://prettier.io to format jekyll output HTML.
          #
          # prettier_opts: '{ "useTabs": true }'
          ### Sets prettier options (in JSON) to format output HTML. For example, output tabs over spaces.
          ### Possible options are outlined in https://prettier.io/docs/en/options.html
          #
          # prettier_ignore: 'about/*'
          ### Ignore paths for prettier to not format those html files.
          ### Useful if the file is exceptionally large, so formatting it takes a while.
          ### Also useful if HTML compression is enabled for that file / formatting messes it up.
          #
          # jekyll_src: sample_site
          ### If the jekyll website source is not in root, specify the directory. (in this case, sample_site)
          ### By default, this is not required as the action searches for a _config.yml automatically.
          #
          # gem_src: sample_site
          ### By default, this is not required as the action searches for a _config.yml automatically.
          ### However, if there are multiple Gemfiles, the action may not be able to determine which to use.
          ### In that case, specify the directory. (in this case, sample_site)
          ###
          ### If jekyll_src is set, the action would automatically choose the Gemfile in jekyll_src.
          ### In that case this input may not be needed as well.
          #
          # key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
          # restore-keys: ${{ runner.os }}-gems-
          ### In cases where you want to specify the cache key, enable the above 2 inputs
          ### Follows the format here https://github.com/actions/cache
          #
          # custom_opts: '--drafts --future'
          ### If you need to specify any Jekyll build options, enable the above input
          ### Flags accepted can be found here https://jekyllrb.com/docs/configuration/options/#build-command-options

    - name: 🚀 deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./_site
          # if the repo you are deploying to is <username>.github.io, uncomment the line below.
          # if you are including the line below, make sure your source files are NOT in the master branch:
        publish_branch: gh-pages
 ```

Comme vous pouvez le constatez ce fichier décrit le pipeline github action qui est déclenchée lors de chaque push sur la branche master. Pour plus d'information sur les modes de déclenchement et l'écriture d'action plus complexe je vous recommande le site suivant : [https://github.com/features/actions](https://github.com/features/actions). Attention, la branche gh-pages doit être créée, sans contenu, avant le premier lancement de la pipeline
## Et voilà ...

Une fois mis en place votre site, écrit votre premier article, et pousser tout ça dans votre branche master, vous pourrez constater que le site Web et automatiquement builder. Voici ce à quoi je suis arrivé pour faire mon blog et présenter mon expérience avec cette suite d'outil:

![résultat](/blog/images/post/2021/04/2021-04-06-creer-un-site-avec-jekyll-et-github-3.png)

Vous pouvez accéder au code source de mon blog ici : [https://github.com/matthieupetite/blog/](https://github.com/matthieupetite/blog/)