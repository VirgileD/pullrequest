---
layout: post
title: XWiki au banc de test
author: feugy
tags: [xwiki, groovy, velocity, wiki, CMS, smartdata]
published: false
---

Il existe plétorde de produits pour faire un Wiki, tous plus complets les uns que les autres.
Alors en quoi XWiki se démarque-t-il ? 

> XWiki, c'est bien plus qu'un wiki. (~~vous voila convaincus, non ?)

C'est en effet ce qui est cranement annoncé sur le site [xwiki.org](http://www.xwiki.org/xwiki/bin/view/Main/WebHome) sous l'appellation ["Wiki de seconde génération"](http://platform.xwiki.org/xwiki/bin/view/Main/SecondGenerationWiki).
Alors, qu'il y a t'il derrière ce discours commercial ? 
La promesse d'un véritable couteau-suisse du Web ? 
Ou un n-ième et décevant CMS ?


## Cas d'étude

Pour les besoin d'une démonstration, j'ai du mettre en place un portail Web minimaliste incluant les fonctionnalités suivantes :

* une page d'accueil
* un espace de partage de données publiques (nous allons y revenir)
* un blog sans workflow de validation (brouillon > publié)
* un Forum simpliste (pas de message privés)

Le tout en 2 semaines, mise en prod inclue.

L'occasion révée de tester XWiki, régulièrement promu par Vincent Massol dans son podcast [les cast codeurs](http://lescastcodeurs.com/), 
et qui propose 3 des 2 fonctionnalités (le forum est un "plugin" nommé [Bulletin board](http://extensions.xwiki.org/xwiki/bin/view/Extension/Bulletin+Board+Application))

XWiki est écrit en Java, et utilise Spring, Hibernate/JPA, Struts, et Velocity. Il est personnalisable avec Groovy, Python, Ruby ou encore Php.


## Qu'est ce qu'un Wiki 2ème génération ?

Alors si j'ai bien tout compris (à vous de vérifier donc :)), le Wiki de 2ème génération est un [CMS](http://fr.wikipedia.org/wiki/Syst%C3%A8me_de_gestion_de_contenu).

XWiki (en version 3.2) propose effet tout ce qui caractérise un CMS : 

* un portail Web qui présente du contenu éditorial, produit par plusieurs utilisateurs
* une séparation entre les données (contenues dans Classes et de Objets) et leur présentation (utilisation de Velocity comme moteur de templating, et de CSS)
* une structuration du contenu : hiérarchisation, organisation en espaces plus ou moins spécialisé (espace forum, espace blog...)
* une véritable gestion des comptes utilisateur et de leurs permissions sur l'ensemble des objets
* un versionning du contenu

En revanche, pas de workflow de publication, bien qu'il soit tout a fait possible d'en créer un.

Car ce qui différentie XWiki des autres produits de Wiki et CMS, c'est son extrème flexibilité.
Il est important de distinguer plusieurs rôles dans la population des utilisateur : l'utilisateur final (l'internaute), l'utilisateur privilégié (le webmaster), le développeur (qui cré des composants/plugins).
Du point de vue des utilisateurs privilégiés, XWiki n'est pas particulièrement facile à prendre en main. Il est en revanche [très bien documenté](http://enterprise.xwiki.org/xwiki/bin/view/UserGuide/WebHome) 
Pour les développeurs c'est l'inverse : très mal documenté, et extèmement puissant.


## Le modèle de donnée XWiki

Dans XWiki, tout est "document". Héritage naturel du Wiki. 
Un document dispose à minima d'un nom unique (utilisé comme url), et d'un certain nombre de propriétés (clé-valeur typée String, nombre, date...). 
Ces propriétés sont définis dans une <tt>Classe</tt>, et les documents sont donc des <tt>Objets</tt>, instances de ces classe.

En gros, si je veux définir un billet de blog, je crée une classe Blog avec 3 champs (auteur, contenu, date de publication), et pour chaque billet le système crée un <tt>Objet</tt> ayant son url propre (<tt>/xwiki/bin/view/Blog/Mon+nom+de+billet</tt>).

Sur chaque document, XWiki propose les actions suivantes :

* <tt>edit</tt>: création/mise à jour du document
* <tt>view</tt>: visualisation
* <tt>delete</tt>: suppression

Au niveau rendu, il est possible d'attacher à la classe un "Class Template" : c'est le formulaire qui permet de remplir les propriétés d'un document lors de l'action <tt>edit</tt>.
On peux aussi attacher une "Class Sheet" : template Velocity pour le rendu (action <tt>view</tt>) des <tt>Objets</tt>.


## Premier pas en temps que développeur.

Très facile à installer (un war à déposer dans un conteneur Servlet), XWiki propose ensuite des fonctionnalités d'import/export, et de personnalisation en ligne.

Ainsi, avec le compte administrateur, et directement dans l'application, on crée et modifie <tt>Classes</tt> et <tt>Objets</tt>.
Le moteur de template [Velocity](http://velocity.apache.org/) est très facile à prendre en main, et j'ai choisis Groovy pour la "logique applicative" à l'intérieur des pages. [Groovy](http://groovy.codehaus.org/) apporte toute la puissance d'un language dynamique à la plateforme Java, tirant partie des librairies existantes, 

Tout ce passe donc à chaud, sans redémarrage. 
Très pratique. Par contre, on édite du code dans un textarea : aucune fonctionnalité d'IDE. 
Il existe cependant un plugin Eclipse pour combler ce manque.

Coté versionning, chaque sauvegarde provoque une nouvelle version de l'<tt>Object</tt>, la <tt>Classe</tt>, le <tt>Class Template</tt> ou la <tt>Class Sheet</tt>. Donc possible de revenir en arrière lorsqu'on à cassé quelque chose.

La fonction d'export permet de faire un zip avec les modifications et configuration qu'on a apporté à son instance. Et naturellement, la fonction d'import permet de "déployer" son zip sur une autre instance.


## Le développement spécifique

XWiki promet une grande facilité d'extension, permettant au développeur d'enrichir les fonctionnalités de base pour "spécialiser" le produit pour un besoin fonctionnel donné.

Dans mon cas, je devais proposer à l'intérieur du portail les fonctionnalités de collecte et partage de données publiques. Le service sous-jacent est Smart Data, accessible via une API Http.
Pour résumer :
* Une page pour uploader un fichier de données, et donner nom/description de la source de données
* Une page pour afficher et réaliser une recherche sur les sources de données existantes.
* Une page pour visualiser le contenu d'une source de données (et télécharger le fichier original)
* Une page pour supprimer les sources de données.

A chaque fonctionnalité correspond une API du service Web de Smart Data.

Première solution : créer une Classe "DataSource", et avoir un Objet "DataSource" pour chaque source de données. Toute la logique de médiation est embarquée dans les page.

Deuxième solution : créer un "composant" XWiki : des classes Java pour enrichir le fonctionnel XWiki, et encapsuler la logique de médiation un peu mieux que dans les pages. Les fonctionnalités sont ensuite accessible dans les pages sous forme de "macro".

J'ai pris la seconde solution, car elle me semblait plus "propre": meilleur découpage/découplage de ma logique propre à Smart Data, meilleur testabilité.

Ex: l'Object XWiki ViewDataSource (template Velocity)
{% highlight html %}
{{velocity}}
{{html wiki="true"}}
## Appel au composant maison : invoque la méthode get() pour récupérer le datasource sur Smart Data.
#set($dataSource = $services.distantDataSourceService.get($request.getParameter('id')))

<div class="sd-datasource">
  <div class="sd-title">
    ## Utilisation de l'objet renvoyé :
    <span class="sd-title">${dataSource.name}</span>
  </div>   
  <div class="sd-content sd-tags">
    #foreach($tag in $dataSource.tags)
      <span class="sd-value sd-tag">$tag</span>
    #end
  </div>
  <div class="sd-content sd-description">
    <span class="sd-label">$msg.get('sd.labels.desc')</span>
    <span class="sd-value">$!{dataSource.desc}</span>
  </div>
</div>

{{/html}}
{{/velocity}}
{% endhighlight %}

Ex: l'interface DataSourceService (Groovy)
{% highlight java %}
import org.xwiki.component.annotation.ComponentRole;

@ComponentRole
interface DataSourceService {
	def get(id)
}
{% endhighlight %}

Ex: l'implémentation DistantDataSourceService (Groovy)
{% highlight java %}
import javax.inject.Named
import javax.inject.Singleton
import org.xwiki.component.annotation.Component
import org.xwiki.component.phase.Initializable

@Singleton
@Component
@Named('distantDataSourceService')
class DistantDataSourceService implements DataSourceService, ScriptService {

    def get(def id) {
		def retrieved = null
		// Retrieve the DataSource
		http.request(GET, JSON) { req ->
			uri.path = "api/datasources/$id"
			response.success = { resp, json ->
				retrieved = json
			}
			response.'500' = {
				retrieved = null
			}
			response.failure = { resp, json ->
				throw new Exception("Unable to retrieved DataSource (id $id) ${json.errors}");
			}
		}
		return retrieved;
	}
{% endhighlight %}

Le système est assez simple : le composant déclare un certain nombre de services (avec les annotations de la JSR-330), ainsi que les implémentations correspondantes. Ces service sont utilisable directement depuis les Object XWiki (injection de dépendance Spring). 


## Conclusion : bien ou quoi ?

XWiki se veux comme un outil très flexible et très puissant, une plateforme pouvant supporter le développement de n'importe quel portail Web, quelque soit ses fonctionnalités. C'est pour le moins ambitieux.

Après avoir travaillé deux semaines avec, je suis assez tenté de dire que le pari est réussi. 

Pour un développeur, la courbe d'apprentissage est minimale lorsqu'on connait Spring/Hibernate. Le modèle de donnée est simple à prendre en main. Velocity et Groovy également. Pour moi, l'aggrégation de ces technologies est naturelle et à propos.

Maintenant, si j'ai pu atteindre mon objectif, ce ne fut pas sans grincements de dents. Et c'est principalement dû à la flexibilité de l'outil : il y a toujours 2 ou 3 manières de réaliser la même fonctionnalité, et parfois, on s'y perd un peu.

La documentation développeur est un gros point faible à mon sens. 
Enfin, les performances ne sont vraiment pas au niveau. Le temps de chargement d'une page est rédhibitoire pour un site à très fort traffic.

En conclusion, je suis assez conquis par XWiki, mais pas dans une optique "grosse production de gros site à gros traffic". Etre productif en solo est facile, mais je ne suis pas aussi confiant dans le cadre d'une équipe d'une dizaine de personnes.
Si je peux donc emettre un conseil : essayez le !