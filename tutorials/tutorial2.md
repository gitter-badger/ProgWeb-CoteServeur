---
title: TD2 &ndash; La persistance des données en PHP
subtitle: BDD, PDO
layout: tutorial
---


<!-- Attention, j'ai enlevé le constructeur par défaut __construct(); -->


Dans le TD1 vous avez appris à créer des classes et à instancier des objets de
ces classes.  Mais, comme vous l'avez constaté, la durée de vie des objets ainsi
créés ne dépassait pas la durée de l'exécution du programme (soit quelques *ms*).

Dans ce TD, nous allons apprendre à rendre les objets persistants, en les
sauvegardant dans une base de données. Ainsi il sera possible de retrouver les
objets d'une visite à l'autre du site web.

**Rappel :** PHP est un langage interprété, les erreurs ne sont pas détectées
avant exécution comme lors d'une compilation.  Il est donc conseillé de tester 
régulièrement chaque nouvelle partie de code.


## Connexion à la base de données

1. Connectez vous à votre base de données MySQL, à l'aide de l'interface
PhpMyAdmin
[http://infolimon.iutmontp.univ-montp2.fr/my](http://infolimon.iutmontp.univ-montp2.fr/my)
Le login est votre login IUT et votre mot de passe initial votre numéro  INE.  
(Si vous êtes sur votre machine, allez sur votre phpmyadmin à l'adresse
[http://localhost/phpmyadmin](http://localhost/phpmyadmin)).

2. Changez votre mot de passe en quelque chose de simple et pas secret. En
   effet, vous devrez bientôt écrire ce mot de passe en clair dans un fichier.
   Si vous n'arrivez pas à vous logger après avoir changé le mot de passe,
   essayer avec un autre navigateur ou bien videz le cache du navigateur.

2. Créez une table `voiture` possédant 3 champs :

   * `immatriculation` de type `VARCHAR` et de longueur maximale 8, défini comme la
     clé primaire
   * `marque` de type `VARCHAR` est de longueur maximale 25.
   * `couleur` de type `VARCHAR` est de longueur maximale 12.
   
   **Attention** : Les noms des champs sont comme des noms de variables, ils ne
   doivent pas contenir d'accents. Par ailleurs, et contrairement à Oracle,
   MySQL est sensible à la casse (minuscules/majuscules).
   
3. Insérez des données en utilisant l'onglet `insérer` de PhpMyAdmin.

4. Dans la suite du TD, pensez à systématiquement tester vos requêtes SQL dans
   PhpMyAdmin avant de les inclure dans vos pages PHP.

### Configuration

Pour avoir un code portable, il est préférable de ne pas utiliser les
informations du serveur directement dans le code.

1. Commencez par créer un fichier `Conf.php`. Ce fichier contiendra une classe
   `Conf` possédant un  attribut statique `$databases` comme suit.
   
   <!-- Sont-ils à l'aise avec les attributs statiques ? -->

   **Notes :**

   * Où doit-on enregistrer un page Web ? (Souvenez-vous du TD précédent)
   * À moins que vous n'ayez déjà un éditeur de pages Web (qui gère au moins
   l'indentation), utilisez NetBeans. Vous trouverez les indications de bases
   sur NetBeans [dans cette note.]({{site.baseurl}}/assets/TutoNetBeans.html)
   * Qu'est-ce qu'un attribut ou une méthode **statique** ? (Cours de Programmation
   Orientée Objet de l'an dernier)

   ```php
   <?php
   class Conf {
   
     static private $databases = array(
       // Le nom d'hote est infolimon a l'IUT
       // ou localhost sur votre machine
       'hostname' => 'a_remplir',
       // A l'IUT, vous avez une BDD nommee comme votre login
       // Sur votre machine, vous devrez creer une BDD
       'database' => 'a_remplir',
       // A l'IUT, c'est votre login
       // Sur votre machine, vous avez surement un compte 'root'
       'login' => 'a_remplir',
       // A l'IUT, c'est votre mdp (INE par defaut)
       // Sur votre machine personelle, vous avez creez ce mdp a l'installation
       'password' => 'a_remplir'
     );
   
     static public function getLogin() {
       //en PHP l'indice d'un tableau n'est pas forcement un chiffre.
       return self::$databases['login'];
     }
   
   }
   ?>
   ```

2. Pour tester notre classe `Conf`, créons un fichier `testConf.php` que l'on
ouvrira dans le navigateur.

   **Souvenez-vous le TD dernier :** Quelle est la bonne et la mauvaise URL
   pour ouvrir une page PHP ? 
   
   ```php
   <?php
     require 'Conf.php';     //equivalent du import en Java

     // On affiche le login de la base de donnees
     echo Conf::getLogin();
   ?>
   ```

3. Complétez `Conf.php` avec des méthodes statiques `getHostname()`,
   `getDatabase()` et `getPassword()`. Testez ces méthodes dans `testConf.php`.
     

**Remarque :** Notez qu'en PHP, on appelle une méthode statique à partir du nom de
la classe comme en Java, mais en utilisant `::` au lieu du `.` en
Java. Souvenez-vous que les méthodes classiques (c'est-à-dire pas `static`)
s'appellent avec `->` en PHP.


### L'objet **PDO**

En PHP pour se connecter à une base de données on utilise une classe fournie
avec PHP qui s'appelle **PDO**
([Php Data Object](http://php.net/manual/fr/book.pdo.php)). Cette classe va nous
fournir de nombreuses méthodes très utiles pour manipuler n'importe quelle base
de donnée.

1. Commençons par établir une connexion à la BDD. Créez un fichier `Model.php`
   contenant une classe `Model`. Ce modèle possédera 
   * un attribut `public static $pdo` <!-- protected -->
   * une fonction `public static function Init()`.

   <!--**Souvenez-vous :** Qu'est-ce qu'un attribut `protected` ? (Cours de
   Programmation Orientée Objet de l'an dernier)-->

2. Dans la fonction `Init`, nous allons initialiser l'attribut `$pdo` en lui
   assignant un objet **PDO**. Procédons en 3 étapes :
   
   1. Mettez dans les variables `$host`, `$dbname`, `$login` et `$pass` les chaînes
   de caractères correspondant à l'hôte, au nom, au login et au mot de passe de
   notre BDD. Récupérez ces informations à l'aide des fonctions de la classe `Conf`.

   2. Pour créer la connexion à notre base de donnée, il faut utiliser le
   constructeur de **PDO** de la façon suivante
   
      ```php?start_inline=1
      new PDO("mysql:host=$host;dbname=$dbname",$login,$pass);
      ```
   
      Stockez ce nouvel objet **PDO** dans la variable statique `self::$pdo`.  
      **Note :** Comme la variable est statique, elle s'accède par une syntaxe
   `Type::$nom_var` comme indiqué précédemment. Le type de l'objet courant
   s'obtient avec le mot clé `self`.

   4. Comme notre classe `Model` dépend de `Conf.php`, ajoutez un `require_once
   'Conf.php'` au début du fichier.  
   Enfin, on souhaite que `Model` soit initialisée juste après sa
   déclaration. Appelez donc l'initialisation statique `Model::Init()` à la fin
   du fichier.

2. Lorsqu'une erreur se produit, PDO n'affiche pas de message d'erreur. À la
place, il lève une exception qu'il faut donc récupérer et traiter. Placez donc
votre `new PDO(...)` au sein d'un try - catch :

   ```php?start_inline=1
   try{
     ... 
   } catch(PDOException $e) {
     echo $e->getMessage(); // affiche un message d'erreur
     die();
   }
   ```
   
   Vous remarquerez que la syntaxe des exceptions en PHP est très semblable à celle
   de Java.

3. Testons dès à présent notre nouvelle classe. Créez le fichier `testModel.php`
   suivant. Vérifiez que vous obtenez bien le message lorsque vous ouvrez ce
   fichier dans le navigateur.

   ```php
   <?php
   require_once "Model.php";
   echo "Connexion réussie !" ;
   ?>
   ```

4. Pour avoir plus de messages d'erreur de PDO et qu'il gère mieux l'UTF-8,
  **mettez à jour** la connexion dans `Model` avec

   ```php?start_inline=1
   // Connexion à la base de données            
   // Le dernier argument sert à ce que toutes les chaines de caractères 
   // en entrée et sortie de MySql soit dans le codage UTF-8
   self::$pdo = new PDO("mysql:host=$host;dbname=$dbname", $login, $pass,
                        array(PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8"));
   
   // On active le mode d'affichage des erreurs, et le lancement d'exception en cas d'erreur
   self::$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
   ```

**Remarque :** Dans cet exemple, la gestion est très brutale: En effet,
l'instruction `die();` équivaut à un système `System.exit(-1);` en Java.  
Dans un vrai site web "en production" il faudrait indiquer à l'utilisateur qu'il
a fait une erreur de saisie ou que le site est actuellement indisponible, ceci
en fonction du détail de l'exception qui est levée.  
Il est important que tout bout de codes utilisant PDO soit dans un `try` -
`catch` afin de capturer les exceptions.

**Question :** Avez-vous compris pourquoi on souhaite que la connexion à la BDD
  (stockée dans `Model::$pdo`) soit un attribut statique ?

## Opérations sur la base de données

Voyons maintenant comment les objets **PDO** servent à effectuer des requêtes SQL.

### Consulter la base de données

Commençons par la requête SQL la plus simple, celle qui lit tous les éléments
d'une table (`voiture` dans notre exemple) :

```sql
SELECT * FROM voiture
```


1. Créez un fichier `lireVoiture.php` avec les éléments suivants

   i. Récupérons la connexion à la BDD de Model.php. Il faut donc faire un
   `require_once "Model.php"`.
   
   ii. La [fonction `query`](http://php.net/manual/fr/pdo.query.php) de l'objet
   **PDO** `Model::$pdo` prend en entrée un chaîne de caractères représentant
   une requête SQL et renvoie la réponse de la requête (sous la forme
   [d'un objet PDOStatement](http://php.net/manual/fr/class.pdostatement.php)).   
   Appelez cette fonction et stockez sa réponse dans une variable `$rep`.
   
   iii. Pour lire les réponses à des requêtes SQL, vous pouvez utiliser

   ```php?start_inline=1
   $tab_obj = $rep->fetchAll(PDO::FETCH_OBJ)
   ```

   qui renvoie un tableau d'objets `tab_obj` ayant pour attributs les champs de
   la BDD.  Chacun des objets `$obj` de `$tab_obj` contient donc trois attributs
   `immatriculation`, `couleur` et `marque` (les champs de la BDD) qui sont
   accessibles classiquement par `$obj->immatriculation`, .... Utilisez une
   boucle [`foreach`](http://php.net/manual/fr/control-structures.foreach.php)
   comme au TD précédent pour itérer sur le tableau `$tab_obj`.

Utilisez cette fonction pour écrire une boucle qui affiche tous les champs de
   toutes les entrées de la table voiture.

2. Ce code fonctionne mais ne crée pas d'objets de la classe `Voiture` sur
lesquelles l'on pourrait appeler des méthodes (par exemple `afficher`).  Mettez
à jour le code de `lireVoiture.php` pour utiliser le code précédent et faire
l'affichage à l'aide de la fonction `afficher()` de `Voiture` en lisant bien les
trois points suivants :



   1. Voici comment récupérer directement un objet de la classe `Voiture`.

      ```php?start_inline=1
      $rep->setFetchMode(PDO::FETCH_CLASS, 'Voiture');
      $tab_voit = $rep->fetchAll();
      ```

      **Note :** La variable `$tab_voit` contient un tableau de `Voiture`. Pour afficher les
     voitures, il faudra itérer sur le tableau avec une boucle
     [`foreach`](http://php.net/manual/fr/control-structures.foreach.php).

   2. Pensez à importer la classe `Voiture` à l'aide de l'appel `require_once
      'Voiture.php';`

   3. Avec l'option <code id="majconst">PDO::FETCH_CLASS</code>, PDO va créer une instance de la
   classe `Voiture`, écrire les attributs correspondants au champs de la BDD
   **puis** appeler le constructeur sans arguments.  
   **Adaptons donc l'ancien constructeur de `Voiture` pour qu'il accepte aucun
     argument et trois arguments.**

      ```php?start_inline=1
      public function __construct($m = NULL, $c = NULL, $i = NULL) {
        if (!is_null($m) && !is_null($c) && !is_null($i)) {
          $this->marque = $m;
          $this->couleur = $c;
          $this->immatriculation = $i;
        }
        $this->options = array();
      }
      ```



3. Nous allons réorganiser ce code. Créez une fonction statique
   `getAllVoitures()` dans la classe `Voiture` qui ne prend pas d'arguments et
   renvoie le tableau des voitures de la BDD. Mettez à jour `lireVoiture.php`.

### Gestion des erreurs

2. Dans un site en production, pour des raisons de sécurité et de confort
d'utilisation, il est déconseillé d'afficher directement un message d'erreur. Pour
cela on va créer une variable pour activer ou désactiver l'affichage des
messages d'erreurs.

   Dans la classe `Conf`, ajouter un attribut statique `debug` et son getter
   publique.

   ```php
   <?php
     class Conf{
      ...
      
       // la variable debug est un boolean
       static private $debug = True; 
       
       static public function getDebug() {
       	return self::$debug;
       }
   }
   ?>
   ```
   
   Ainsi on peut modifier les messages d'erreurs dans les `catch`.
   
   ```php?start_inline=1
   try {
     ...
   } catch (PDOException $e) {
     if (Conf::getDebug()) {
       echo $e->getMessage(); // affiche un message d'erreur
     } else {
       echo 'Une erreur est survenue <a href=""> retour a la page d\'accueil </a>';
     }
     die();
   }
   ```

### Site de covoiturage

Reprendre les classes du TP précédent sur le covoiturage et y ajouter
la gestion de la persistance.



<!-- Si qq a fini en avance alors il peut coder insert avec query et une
injection SQL-->
