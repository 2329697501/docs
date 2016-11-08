Routage
=======

Le composant routeur vous permet de d�finir des routes qui correspondent � des contr�leurs ou des gestionnaires qui doivent
recevoir la requ�te. Un routeur analyse l'URI pour extraire cette information. Le routeur dispose de deux modes: MVC,
et correspondance seulement (match-only). Le premier mode est id�al pour travailler sur des applications MVC.

D�finir des Routes
------------------
:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` fournit des possibilit� de routage avanc�es. En mode MVC
vous pouvez d�finir des routes et les faire correspondre � des contr�leurs ou des actions dont vous avez besoin. Une route est d�fnie comme suit:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Cr�ation du routeur
    $router = new Router();

    // D�fintion d'une route
    $router->add(
        "/admin/users/my-profile",
        [
            "controller" => "users",
            "action"     => "profile",
        ]
    );

    // Une autre route
    $router->add(
        "/admin/users/change-password",
        [
            "controller" => "users",
            "action"     => "changePassword",
        ]
    );

    $router->handle();

Le premier param�tre de la m�thode :code:`add()` est le motif recherch� et, optionnellement, le second param�tre est un ensemble de chemins.
Dans ce cas, si l'URI est /admin/users/my-profile, alors l'action "profile" du contr�leur "users" sera ex�cut�e.
Il faut se rappeler que le routeur n'ex�cute pas l'action du contr�leur, il r�cup�re uniquement cette information
pour en informer le bon composant (par ex. :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`)
que c'est ce contr�leur ou cette action qui doit �tre ex�cut�e.

D�finir les routes une � une d'une application qui poss�de plusieurs chemins peut �tre une t�che p�nible. Pour ces cas nous pouvons
cr�er des routes plus flexibles:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Cr�ation du routeur
    $router = new Router();

    // D�finition de route
    $router->add(
        "/admin/:controller/a/:action/:params",
        [
            "controller" => 1,
            "action"     => 2,
            "params"     => 3,
        ]
    );

Dans l'exemple pr�c�dent nous utilisons des jokers pour rendre la route valide pour plusieurs URIs. Par exemple, cette URL
(/admin/users/a/delete/dave/301) pourrait produire:

+------------+---------------+
| Contr�leur | users         |
+------------+---------------+
| Action     | delete        |
+------------+---------------+
| Param�tre  | dave          |
+------------+---------------+
| Param�tre  | 301           |
+------------+---------------+

La m�thode :code:`add()` re�oit un motif qui peut optionnellement avoir des marqueurs et des expressions r�guli�res.
Tous les modtifs de routage doivent commencer avec une barre oblique (/). La syntaxe utilis�e pour les expressions r�guli�res
est la m�me que les `PCRE regular expressions`_. Notez qu'il n'est pas n�cessaire d'ajouter les d�limiteurs d'expression r�guli�re.
Tous les motifs de route sont insensibles � la casse.

Le second param�tre d�finit comment les parties reconnues sont reli�es aux contr�leur/action/param�tre. Les parties � reconna�tre
sont des marqueurs ou des sous-motifs d�limit�s par des parenth�ses (round brackets). Dans l'exemple donn� pr�c�demment,
le premier sous-motif correspondant (:code:`:controller`) est partie contr�leur de la route, le deuxi�me est l'action, et ainsi de suite.

Ces marqueurs facilite l'�criture d'expression r�guli�re qui sont plus lisible pour le d�veloppeur et facile � comprendre.
Les marqueurs suivant sont support�s:

+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| Marqueur             | Expression r�guli�re        | Utilisation                                                                                            |
+======================+=============================+========================================================================================================+
| :code:`/:module`     | :code:`/([a-zA-Z0-9\_\-]+)` | Correspond � un module valide contenant seulement des caract�res alphanum�riques                       |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:controller` | :code:`/([a-zA-Z0-9\_\-]+)` | Correspond � un contr�leur valide contenant seulement des caract�res alphanum�riques                   |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:action`     | :code:`/([a-zA-Z0-9\_]+)`   | Correspond � une action valide contenant seulement des caract�res alphanum�riques                      |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:params`     | :code:`(/.*)*`              | Correspond � une liste de mots optionnels s�par�s bar des slashs. A n'utiliser qu'en fin de route !    |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:namespace`  | :code:`/([a-zA-Z0-9\_\-]+)` | Correspond � un espace de nom � un seul niveau                                                         |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+
| :code:`/:int`        | :code:`/([0-9]+)`           | Correspond � un param�tre de type entier                                                               |
+----------------------+-----------------------------+--------------------------------------------------------------------------------------------------------+

Les noms de contr�leur sont "cam�lis�s". Ceci signifie que les caract�res (:code:`-`) et (:code:`_`) sont retir�s et que le caract�re qui suit
est mis en majuscule. Par exemple, un_controleur est convertit en UnControleur.

Depuis que vous pouvez ajouter autant de routes que n�cessaire gr�ce � la m�thode  :code:`add()`, l'ordre d'ajout des routes indique
leur pertinence, les derni�res routes ajout�s �tant plus pertinentes que les premi�res. En interne, toutes les routes
sont parcourues dans l'ordre inverse jusqu'� ce que :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` trouve
celle qui correspond � l'URI fournie et la traite, ignorant alors le reste.

Param�tres avec des Noms
^^^^^^^^^^^^^^^^^^^^^^^^
L'exemple ci-dessous d�montre comment d�finir des noms pour les param�tres d'une route:

.. code-block:: php

    <?php

    $router->add(
        "/news/([0-9]{4})/([0-9]{2})/([0-9]{2})/:params",
        [
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1, // ([0-9]{4})
            "month"      => 2, // ([0-9]{2})
            "day"        => 3, // ([0-9]{2})
            "params"     => 4, // :params
        ]
    );

Dans l'exemple pr�c�dent, la route ne contient aucune partie "contr�ler" ou "action". Ces parties sont remplac�es
par des valeurs constantes ("posts" et "show"). L'utilisateur ignore quel est le contr�leur qui est r�ellement
concern� par la requ�te. Dans le contr�leur, on peut acc�der � ces param�tres nomm�s de la mani�re suivante:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function showAction()
        {
            // Get "year" parameter
            $year = $this->dispatcher->getParam("year");

            // Get "month" parameter
            $month = $this->dispatcher->getParam("month");

            // Get "day" parameter
            $day = $this->dispatcher->getParam("day");

            // ...
        }
    }

Notez que les valeurs des param�tres sont obtenues depuis le r�partiteur. Ceci arrive parce que c'est
le composant qui finalement interagit avec les pilotes de votre application. De plus, il existe une autre
fa�on de cr�er des param�tres nomm�es � l'int�rieur du motif:

.. code-block:: php

    <?php

    $router->add(
        "/documentation/{chapter}/{name}.{type:[a-z]+}",
        [
            "controller" => "documentation",
            "action"     => "show",
        ]
    );

Vous pouvez acc�der aux valeurs de la m�me fa�on que pr�c�demment:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class DocumentationController extends Controller
    {
        public function showAction()
        {
            // Get "name" parameter
            $name = $this->dispatcher->getParam("name");

            // Get "type" parameter
            $type = $this->dispatcher->getParam("type");

            // ...
        }
    }

Syntaxe courte
^^^^^^^^^^^^^^
Si vous n'aimez pas utiliser les tableaux pour d�finir des routes, une autre syntaxe est possible.
L'exemple suivant produit le m�me r�sultat:

.. code-block:: php

    <?php

    // Forme courte
    $router->add(
        "/posts/{year:[0-9]+}/{title:[a-z\-]+}",
        "Posts::show"
    );

    // Forme tableau
    $router->add(
        "/posts/([0-9]+)/([a-z\-]+)",
        [
           "controller" => "posts",
           "action"     => "show",
           "year"       => 1,
           "title"      => 2,
        ]
    );

M�langer les Syntaxes Tableau et Courtes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Les syntaxes tableau et courtes peuvent �tre m�lang�es pour d�finir une route. Dans ce cas, notez que les param�tres nomm�es
sont ajout�s automatiquement aux chemins selon la position dans laquelle ils sont d�finis:

.. code-block:: php

    <?php

    // La premi�re position est ignor�e parce qu'elle est utilis�e
    // pour le param�tre 'country'
    $router->add(
        "/news/{country:[a-z]{2}}/([a-z+])/([a-z\-+])",
        [
            "section" => 2, // Positions start with 2
            "article" => 3,
        ]
    );

Router vers des Modules
^^^^^^^^^^^^^^^^^^^^^^^
Vous pouvez d�finir des routes dont les chemins incluent des modules. Ceci est sp�cialement adapt� aux application multi-modules.
Il est possible de d�finir une route qui inclus un joker pour le module:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    $router = new Router(false);

    $router->add(
        "/:module/:controller/:action/:params",
        [
            "module"     => 1,
            "controller" => 2,
            "action"     => 3,
            "params"     => 4,
        ]
    );

Dans le cas le nom de module sera toujours partie int�grante de l'URL. Par exemple, l'URL: /admin/users/edit/sonny
sera trait�e comme:

+------------+---------------+
| Module     | admin         |
+------------+---------------+
| Contr�leur | users         |
+------------+---------------+
| Action     | edit          |
+------------+---------------+
| Param�tre  | sonny         |
+------------+---------------+

Ou bien vous pouvez rattacher des routes sp�cifiques � des modules sp�cifiques:

.. code-block:: php

    <?php

    $router->add(
        "/login",
        [
            "module"     => "backend",
            "controller" => "login",
            "action"     => "index",
        ]
    );

    $router->add(
        "/products/:action",
        [
            "module"     => "frontend",
            "controller" => "products",
            "action"     => 1,
        ]
    );

Ou les rattacher � des espaces de noms sp�cifiques:

.. code-block:: php

    <?php

    $router->add(
        "/:namespace/login",
        [
            "namespace"  => 1,
            "controller" => "login",
            "action"     => "index",
        ]
    );

Les noms d'espace de nom et de classe doivent �tre transmis s�par�ment:

.. code-block:: php

    <?php

    $router->add(
        "/login",
        [
            "namespace"  => "Backend\\Controllers",
            "controller" => "login",
            "action"     => "index",
        ]
    );

Restriction de la M�thode HTTP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Lorsque vous ajoutez une route en utilisant simplement :code:`add()` la route est d�fnie pour toutes les m�thodes HTTP. De temps en temps, nous pouvons restreindre une route
� une m�thode en particulier. Ceci est sp�cialement utile lors de la cr�ation d'applications RESTful:

.. code-block:: php

    <?php

    // Cette route correspondra seulement si la m�thode HTTP est GET
    $router->addGet(
        "/products/edit/{id}",
        "Products::edit"
    );

    // Cette route correspondra seulement si la m�thode HTTP est POST
    $router->addPost(
        "/products/save",
        "Products::save"
    );

    // Cette route correspondra seulement si la m�thode HTTP est POST ou PUT
    $router->add(
        "/products/update",
        "Products::update"
    )->via(
        [
            "POST",
            "PUT",
        ]
    );

Utilisation de Convertisseurs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Les convertisseurs vous permettent de transformer librement les param�tres d'une route avant de les transmettre au r�partiteur.
Les exemples qui suivent vous montre comment s'en servir:

.. code-block:: php

    <?php

    // Le nom de l'action autorise les tirets. Une action peut �tre: /products/new-ipod-nano-4-generation
    $route = $router->add(
        "/products/{slug:[a-z\-]+}",
        [
            "controller" => "products",
            "action"     => "show",
        ]
    );

    $route->convert(
        "slug",
        function ($slug) {
            // Transforme slug en supprimant les tirets
            return str_replace("-", "", $slug);
        }
    );

Un autre cas d'utilisation des convertisseurs est de relier un mod�le � une route. Ceci permet de transmettre directement le mod�le � l'action:

.. code-block:: php

    <?php

    // Cet exemple fonctionne en supposant que l'ID est transmis en param�tre dans l'url: /products/4
    $route = $router->add(
        "/products/{id}",
        [
            "controller" => "products",
            "action"     => "show",
        ]
    );

    $route->convert(
        "id",
        function ($id) {
            // Fetch the model
            return Product::findFirstById($id);
        }
    );

Groupe de Routes
^^^^^^^^^^^^^^^^
Si un ensemble de route a des chemins communs, ils peuvent �tre regroup�s pour les maintenir ais�ment:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;
    use Phalcon\Mvc\Router\Group as RouterGroup;

    $router = new Router();

    // Cr�ation d'un groupe avec un module et un contr�leur communs
    $blog = new RouterGroup(
        [
            "module"     => "blog",
            "controller" => "index",
        ]
    );

    // Toutes les routes commencent par /blog
    $blog->setPrefix("/blog");

    // Ajout d'une route au groupe
    $blog->add(
        "/save",
        [
            "action" => "save",
        ]
    );

    // Ajout d'une autre route au groupe
    $blog->add(
        "/edit/{id}",
        [
            "action" => "edit",
        ]
    );

    // Cette route est reli�e � un autre contr�leur que celui par d�faut
    $blog->add(
        "/blog",
        [
            "controller" => "blog",
            "action"     => "index",
        ]
    );

    // Ajout du groupe au routeur
    $router->mount($blog);

Vous pouvez placer les groupes de routes dans des fichiers distincts pour am�liorer l'organisation et la r�utilisation de code:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Group as RouterGroup;

    class BlogRoutes extends RouterGroup
    {
        public function initialize()
        {
            // Default paths
            $this->setPaths(
                [
                    "module"    => "blog",
                    "namespace" => "Blog\\Controllers",
                ]
            );

            // Toutes les routes commencent par /blog
            $this->setPrefix("/blog");

            // Ajout d'une route au groupe
            $this->add(
                "/save",
                [
                    "action" => "save",
                ]
            );

            // Ajout d'une autre route au groupe
            $this->add(
                "/edit/{id}",
                [
                    "action" => "edit",
                ]
            );

            // Cette route est reli�e � un autre contr�leur que celui par d�faut
            $this->add(
                "/blog",
                [
                    "controller" => "blog",
                    "action"     => "index",
                ]
            );
        }
    }

On monte le groupe dans le routeur:

.. code-block:: php

    <?php

    // Ajout du groupe au routeur
    $router->mount(
        new BlogRoutes()
    );

Correspondance de Routes
------------------------
Une URI valide doit �tre transmise au routeur pour qu'il puisse la traiter et trouver une route correspondante.
Par d�faurt, l'URI � router est prise dans la variable :code:`$_GET['_url']` qui est cr��e par le module de r��criture.
Un ensemble de r�gles de r��criture qui fonctionne bien avec Phalcon est:

.. code-block:: apacheconf

    RewriteEngine On
    RewriteCond   %{REQUEST_FILENAME} !-d
    RewriteCond   %{REQUEST_FILENAME} !-f
    RewriteRule   ^((?s).*)$ index.php?_url=/$1 [QSA,L]

Avec cette configuration, toutes les requ�tes vers des fichiers ou des dossiers qui n'existent pas sont envoy�s � index.php.

L'exemple suivant montre comment utiliser ce composant dans un mode autonome:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Cr�ation du routeur
    $router = new Router();

    // D�finition de routes s'il y a
    // ...

    // R�cup�re l'URI depuis $_GET["_url"]
    $router->handle();

    // Ou en d�finissant l'URI directement
    $router->handle("/employees/edit/17");

    // R�cup�ration du contr�leur trouv�
    echo $router->getControllerName();

    // R�cup�ration de l'action trouv�e
    echo $router->getActionName();

    // R�cup�ration de la route trouv�e
    $route = $router->getMatchedRoute();

Routes Nomm�es
--------------
Chaque route ajout�e au routeur est stock�e en interne en tant qu'objet de :doc:`Phalcon\\Mvc\\Router\\Route <../api/Phalcon_Mvc_Router_Route>`.
Cette classe encapsule tous les d�tails d'une route. Par exemple, nous pouvons donn�e un nom au chemin afin de l'identifier de mani�re unique dans notre application.
Ceci est particuli�rement utile lorsqu'il faut s'en servir pour cr�er des URLs.

.. code-block:: php

    <?php

    $route = $router->add(
        "/posts/{year}/{title}",
        "Posts::show"
    );

    $route->setName("show-posts");

    // Ou juste

    $router->add("/posts/{year}/{title}", "Posts::show")->setName("show-posts");

Ensuite en utilisant par exemple le composant :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>` nous pouvons contruire des routes � partir de son nom:

.. code-block:: php

    <?php

    // Retourne /posts/2012/phalcon-1-0-released
    echo $url->get(
        [
            "for"   => "show-posts",
            "year"  => "2012",
            "title" => "phalcon-1-0-released",
        ]
    );

Exemple d'utilisation
---------------------
Ce qui suit sont des exemples de routes personnalis�es:

.. code-block:: php

    <?php

    // Trouve "/system/admin/a/edit/7001"
    $router->add(
        "/system/:controller/a/:action/:params",
        [
            "controller" => 1,
            "action"     => 2,
            "params"     => 3,
        ]
    );

    // Trouve "/es/news"
    $router->add(
        "/([a-z]{2})/:controller",
        [
            "controller" => 2,
            "action"     => "index",
            "language"   => 1,
        ]
    );

    // Trouve "/es/news"
    $router->add(
        "/{language:[a-z]{2}}/:controller",
        [
            "controller" => 2,
            "action"     => "index",
        ]
    );

    // Trouve "/admin/posts/edit/100"
    $router->add(
        "/admin/:controller/:action/:int",
        [
            "controller" => 1,
            "action"     => 2,
            "id"         => 3,
        ]
    );

    // Trouve "/posts/2015/02/some-cool-content"
    $router->add(
        "/posts/([0-9]{4})/([0-9]{2})/([a-z\-]+)",
        [
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1,
            "month"      => 2,
            "title"      => 4,
        ]
    );

    // Trouve "/manual/en/translate.adapter.html"
    $router->add(
        "/manual/([a-z]{2})/([a-z\.]+)\.html",
        [
            "controller" => "manual",
            "action"     => "show",
            "language"   => 1,
            "file"       => 2,
        ]
    );

    // Trouve /feed/fr/le-robots-hot-news.atom
    $router->add(
        "/feed/{lang:[a-z]+}/{blog:[a-z\-]+}\.{type:[a-z\-]+}",
        "Feed::get"
    );

    // Trouve /api/v1/users/peter.json
    $router->add(
        "/api/(v1|v2)/{method:[a-z]+}/{param:[a-z]+}\.(json|xml)",
        [
            "controller" => "api",
            "version"    => 1,
            "format"     => 4,
        ]
    );

.. highlights::

    Prenez garde aux caract�res autoris�s dans les expressions r�guli�re pour les contr�leurs et les espaces de noms. Comme ils
    deviennent des noms de classe, ils peuvent permettre � des attaquants d'atteindre le syst�me de fichiers et donc de lire des
    fichiers non autoris�s. Une expression r�guli�re s�re est :code:`/([a-zA-Z0-9\_\-]+)`

Comportement par D�faut
-----------------------
:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` a un comportement par d�faut qui fournit un routage tr�s simple
qui s'attend � ce que l'URI corresponde au motif: /:controller/:action/:params

Par exemple pour une URL du style *http://phalconphp.com/documentation/show/about.html*, le routeur transformera comme suit:

+------------+---------------+
| Contr�leur | documentation |
+------------+---------------+
| Action     | show          |
+------------+---------------+
| Param�tre  | about.html    |
+------------+---------------+

Si vous ne souhaitez pas que le routeur ait ce comportement, vous devez cr�er le routeur en passant :code:`false` en premier param�tre:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Cr�ation du routeur sans route par d�faut
    $router = new Router(false);

D�finir la route par d�faut
---------------------------
Quand votre application est acc�d�e sans aucune route c'est la route '/' qui est utilis�e pour d�terminer quels sont les chemins � utiliser pour
afficher la page initiale de votre site web ou de votre application:

.. code-block:: php

    <?php

    $router->add(
        "/",
        [
            "controller" => "index",
            "action"     => "index",
        ]
    );

Chemins Introuvables
--------------------
Si aucune des routes sp�cifi�es au routeur ne correspond, vous pouvez d�finir un groupe de chemin pour ce type de sc�nario;

.. code-block:: php

    <?php

    // Set 404 paths
    $router->notFound(
        [
            "controller" => "index",
            "action"     => "route404",
        ]
    );

Ceci est typiquement pour une page d'Erreur 404.

Etablir des chemins par d�faut
------------------------------
Il est possible de d�finir des valeurs par d�faut pour le module, le contr�leur ou l'action. Lorqu'il manque une route,
n'importe lequel des ces chemin peut �tre automatiquement compl�t� par le routeur:

.. code-block:: php

    <?php

    // D�finition d'un d�faut sp�cifique
    $router->setDefaultModule("backend");
    $router->setDefaultNamespace("Backend\\Controllers");
    $router->setDefaultController("index");
    $router->setDefaultAction("index");

    // Avec un tableau
    $router->setDefaults(
        [
            "controller" => "index",
            "action"     => "index",
        ]
    );

Traitement des slashs terminaux
-------------------------------
Il arrive qu'une route soit acc�d�e avec des slashs terminaux.
Ces slashs en trop peuvent provoquer un �tat de non-trouv� dans le r�partiteur.
Vous pouvez param�trer le routeur pour qu'il retire automatiquement les slashs qui se trouvent � la fin d'une route:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    $router = new Router();

    // Retrait automatique des slashs terminaux
    $router->removeExtraSlashes(true);

Ou bien, vous pouvez modifier des routes en particulier pour qu'elles acceptent des slashs terminaux:

.. code-block:: php

    <?php

    // The [/]{0,1} autorise cette route de terminer �ventuellement avec un slash
    $router->add(
        "/{language:[a-z]{2}}/:controller[/]{0,1}",
        [
            "controller" => 2,
            "action"     => "index",
        ]
    );

Rappel sur Correspondance
--------------------------
De temps en temps, des routes ne peuvent correspondre que si elle remplissent certaines conditions.
Vous pouvez ajouter des conditions arbitraires aux routes en utilisant la fonction de rappel :code:`beforeMatch()`.
Si la fonction retourne :code:`false`, la route sera consid�r�e comme ne pas correspondre:

.. code-block:: php

    <?php

    $route = $router->add("/login",
        [
            "module"     => "admin",
            "controller" => "session",
        ]
    );

    $route->beforeMatch(
        function ($uri, $route) {
            // V�rifie qu'il s'agit d'une requ�te Ajax
            if (isset($_SERVER["HTTP_X_REQUESTED_WITH"]) && $_SERVER["HTTP_X_REQUESTED_WITH"] === "XMLHttpRequest") {
                return false;
            }

            return true;
        }
    );

Vous pouvez r�utiliser des conditions compl�mentaires dans des classes:

.. code-block:: php

    <?php

    class AjaxFilter
    {
        public function check()
        {
            return $_SERVER["HTTP_X_REQUESTED_WITH"] === "XMLHttpRequest";
        }
    }

Et exploiter cette classe au lieu d'une fonction anonyme:

.. code-block:: php

    <?php

    $route = $router->add(
        "/get/info/{id}",
        [
            "controller" => "products",
            "action"     => "info",
        ]
    );

Depuis Phalcon 2.1.0 beta 1, il existe une autre fa�on de v�rifier:
    $route->beforeMatch(
        [
            new AjaxFilter(),
            "check"
        ]
    );

As of Phalcon 3, there is another way to check this:

.. code-block:: php

    <?php

    $route = $router->add(
        "/login",
        [
            "module"     => "admin",
            "controller" => "session",
        ]
    );

        // V�rifie qu'il s'agit d'une requ�te Ajax
        return $request->isAjax();
    });
    $route->beforeMatch(
        function ($uri, $route) {
            /**
             * @var string $uri
             * @var \Phalcon\Mvc\Router\Route $route
             * @var \Phalcon\DiInterface $this
             * @var \Phalcon\Http\Request $request
             */
            $request = $this->getShared("request");

Contraintes de Nom d'H�te
-------------------------
Le routeur vous permet d'�tablir des contraintes selon le nom de l'h�te, ceci signifie que des routes sp�cifiques ou des groupes de routes
peuvent �tre restreintes seulement si la route satisfait la contrainte du nom d'h�te;

.. code-block:: php

    <?php

    $route = $router->add(
        "/login",
        [
            "module"     => "admin",
            "controller" => "session",
            "action"     => "login",
        ]
    );

Le nom d'h�te peut �galement �tre transmis sous forme d'expression r�guli�re:

.. code-block:: php

    <?php

    $route = $router->add(
        "/login",
        [
            "module"     => "admin",
            "controller" => "session",
            "action"     => "login",
        ]
    );

Vous pouvez faire en sorte qu'une contrainte de nom d'h�te s'applique � toutes les routes d'un groupe de routes:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Group as RouterGroup;

    // Cr�ation d'un groupe avec un module et un contr�leur communs
    $blog = new RouterGroup(
        [
            "module"     => "blog",
            "controller" => "posts",
        ]
    );

    // Restriction sur le nom de l'h�te
    $blog->setHostName("blog.mycompany.com");

    // Toutes les routes commencent par /blog
    $blog->setPrefix("/blog");

    // Route par d�faut
    $blog->add(
        "/",
        [
            "action" => "index",
        ]
    );

    // Ajout d'une route au groupe
    $blog->add(
        "/save",
        [
            "action" => "save",
        ]
    );

    // Ajout d'un autre route au groupe
    $blog->add(
        "/edit/{id}",
        [
            "action" => "edit",
        ]
    );

    // Ajout du groupe au routeur
    $router->mount($blog);

Sources d'URI
-------------
Par d�faut l'URI est extraite de la variable :code:`$_GET['_url']` qui est transmise � Phalcon par le moteur de r��criture.
Vous pouvez �galement utiliser :code:`$_SERVER['REQUEST_URI']` si c'est n�cessaire:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // ...

    // Use $_GET["_url"] (default)
    $router->setUriSource(
        Router::URI_SOURCE_GET_URL
    );

Ou bien vous pouvez transmettre manuellement l'URI � la m�thode :code:`handle()`:
    // Use $_SERVER["REQUEST_URI"]
    $router->setUriSource(
        Router::URI_SOURCE_SERVER_REQUEST_URI
    );


.. code-block:: php

    <?php

    $router->handle("/some/route/to/handle");

Test de vos routes
------------------
Tant que le composant n'a pas de d�pendances, vous pouvez cr�er un fichier comme montr� ci-dessous pour tester vos routes:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    // Ces routes simulent de vrai URIs
    $testRoutes = [
        "/",
        "/index",
        "/index/index",
        "/index/test",
        "/products",
        "/products/index/",
        "/products/show/101",
    ];

    $router = new Router();

    // Ajoutez ici vos propres routes
    // ...

    // Test de chaque route
    foreach ($testRoutes as $testRoute) {
        // Gestion de la route
        $router->handle($testRoute);

        echo "Testing ", $testRoute, "<br>";

        // V�rifie que chaque route corresponde
        if ($router->wasMatched()) {
            echo 'Contr�leur: ', $router->getControllerName(), '<br>';
            echo 'Action: ', $router->getActionName(), '<br>';
        } else {
            echo 'La route n\'a pas de correspondance<br>';
        }

        echo "<br>";
    }

Annotations du Routeur
----------------------
Ce composant fournit une variante du service :doc:`annotations <annotations>`. Avec cette strat�gie vous
pouvez �crire les routes directement dans les contr�leurs plut�t que les ajouter dans le service d'inscription:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Annotations as RouterAnnotations;

    $di["router"] = function () {
        // Utilise les annotations du routeur. Nous passons 'faux' si nous ne voulons pas que le routeur ajoute son motif par d�faut
        $router = new RouterAnnotations(false);

        // Lecture des annotations depuis ProductsController si l'URI commence par /api/products
        $router->addResource("Products", "/api/products");

        return $router;
    };

Les annotations peuvent �tre �crites de la fa�on suivante:

.. code-block:: php

    <?php

    /**
     * @RoutePrefix("/api/products")
     */
    class ProductsController
    {
        /**
         * @Get(
         *     "/"
         * )
         */
        public function indexAction()
        {

        }

        /**
         * @Get(
         *     "/edit/{id:[0-9]+}",
         *     name="edit-robot"
         * )
         */
        public function editAction($id)
        {

        }

        /**
         * @Route(
         *     "/save",
         *     methods={"POST", "PUT"},
         *     name="save-robot"
         * )
         */
        public function saveAction()
        {

        }

        /**
         * @Route(
         *     "/delete/{id:[0-9]+}",
         *     methods="DELETE",
         *     conversors={
         *         id="MyConversors::checkId"
         *     }
         * )
         */
        public function deleteAction($id)
        {

        }

        public function infoAction($id)
        {

        }
    }

Seules les m�thodes marqu�es par une annotation valide sont utilis�es comme routes. Voyez la liste des annotations support�es:

+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Nom          | Description                                                                                                    | Exemple de d�claration                                                     |
+==============+================================================================================================================+============================================================================+
| RoutePrefix  | Un pr�fixe qui sera plac� devant chaque route URI. Cette annotation est � placer dans le docblock de la classe | :code:`@RoutePrefix("/api/products")`                                      |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Route        | Cette annotation associe une m�thode � une route. Cette annotation est � placer dans le docblock d'une m�thode | :code:`@Route("/api/products/show")`                                       |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Get          | Cette annotation associe une m�thode � une route avec une restriction sur la m�thode HTTP GET                  | :code:`@Get("/api/products/search")`                                       |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Post         | Cette annotation associe une m�thode � une route avec une restriction sur la m�thode HTTP POST                 | :code:`@Post("/api/products/save")`                                        |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Put          | Cette annotation associe une m�thode � une route avec une restriction sur la m�thode HTTP PUT                  | :code:`@Put("/api/products/save")`                                         |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Delete       | Cette annotation associe une m�thode � une route avec une restriction sur la m�thode HTTP DELETE               | :code:`@Delete("/api/products/delete/{id}")`                               |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Options      | Cette annotation associe une m�thode � une route avec une restriction sur la m�thode HTTP OPTIONS              | :code:`@Option("/api/products/info")`                                      |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+

Pour les annotations qui ajoutent des routes, les param�tres suivants sont support�s:

+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| Nom          | Description                                                                                                    | Exemple de d�claration                                                     |
+==============+================================================================================================================+============================================================================+
| methods      | D�finit une ou plusieurs m�thodes HTPP que la route doit respecter                                             | :code:`@Route("/api/products", methods={"GET", "POST"})`                   |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| name         | D�finit le nom d'une route                                                                                     | :code:`@Route("/api/products", name="get-products")`                       |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| paths        | Un tableau de chemins identiques � ceux pass�s � :code:`Phalcon\Mvc\Router::add()`                             | :code:`@Route("/posts/{id}/{slug}", paths={module="backend"})`             |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+
| conversors   | Un ensemble de convertisseurs qui s'appliquent aux param�tres                                                  | :code:`@Route("/posts/{id}/{slug}", conversors={id="MyConversor::getId"})` |
+--------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------+

Si vous utilisez des modules dans votre application, il vaut mieux utiliser la m�thode :code:`addModuleResource()`:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router\Annotations as RouterAnnotations;

    $di["router"] = function () {
        // Utilise les annotations de routage
        $router = new RouterAnnotations(false);

        // Lecture des annotations depuis Backend\Controllers\ProductsController si l'URI commence par /api/products
        $router->addModuleResource("backend", "Products", "/api/products");

        return $router;
    };

Inscription d'une Instance de Routeur
-------------------------------------
Vous pouvez inscrire le routeur lors de la proc�dure d'inscription du service dans l'injecteur de d�pdendance de Phalcon pour le rendre disponible aux contr�leurs.

Vous devez ajouter le code suivant dans votre fichier d'amorce (par exemple index.php ou app/config/services.php si vous utilisez `Phalcon Developer Tools <http://phalconphp.com/en/download/tools>`_)

.. code-block:: php

    <?php

    /**
     * Ajout de la capacit� de routage
     */
    $di->set(
        "router",
        function () {
            require __DIR__ . "/../app/config/routes.php";

            return $router;
        }
    );

Vous devrez cr�er app/config/routes.php et d'ajouter du code d'initialisation du routeur, comme par exemple:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Router;

    $router = new Router();

    $router->add(
        "/login",
        [
            "controller" => "login",
            "action"     => "index",
        ]
    );

    $router->add(
        "/products/:action",
        [
            "controller" => "products",
            "action"     => 1,
        ]
    );

    return $router;

Ecriture de votre propre Routeur
--------------------------------
L'interface :doc:`Phalcon\\Mvc\\RouterInterface <../api/Phalcon_Mvc_RouterInterface>` doit �tre impl�ment�e pour cr�er un routeur en remplacement
de celui fournit par Phalcon.

.. _PCRE regular expressions: http://php.net/manual/fr/book.pcre.php
