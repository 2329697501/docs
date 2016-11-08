Utilisation de Contr�leurs
==========================

Les contr�leurs fournissent un certain nombre de m�thodes qui sont appel�es actions. Les actions sont des m�thodes du contr�leur qui traitent les requ�tes.
Par d�faut toutes les m�thodes publiques d'un contr�leur sont reli�es � des actions et sont accessibles par URL. Les actions sont responsables
de l'interpr�tation de la requ�te et de la cr�ation de la r�ponse. Habituellement les r�ponses sont sous la forme de vues mais il existe d'autres fa�ons de cr�er des r�ponses adapt�es.

Par exemple, en acc�dant � une URL comme ceci: http://localhost/blog/posts/show/2015/the-post-title Phalcon par d�faut d�composera chaque 
partie comme ceci:

+------------------------+----------------+
| **R�pertoire Phalcon** | blog           |
+------------------------+----------------+
| **Contr�leur**         | posts          |
+------------------------+----------------+
| **Action**             | show           |
+------------------------+----------------+
| **Param�tre**          | 2015           |
+------------------------+----------------+
| **Param�tre**          | the-post-title |
+------------------------+----------------+

Dans ce cas, le contr�leur "PostsController" g�rera cette requ�te. Il n'y a pas d'emplacement particulier pour placer les contr�leurs dans une application,
ils seraient charg�s gr��e � un :doc:`chargeur automatique <loader>`, et donc vous restez libre d'organiser vos contr�leurs selon vos besoins.

Les contr�leurs doivent poss�der le suffixe "Controller", et les actions le suffixe "Action". Un exemple de contr�leur se trouve ci-apr�s:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function showAction($year, $postTitle)
        {

        }
    }

Les param�tres suppl�mentaires de l'URI sont d�finis comme des param�tres de l'action, ainsi ils restent facilement accessibles en utilisant des variables locales.
Un contr�leur peut �ventuellement �tendre :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>`. En faisant ceci, le contr�leur acc�de
facilement aux services de l'application.

Les param�tres sans valeur par d�faut sont correctement g�r�s. La d�finition de param�tres optionels se fait comme d'habitude en PHP:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function showAction($year = 2015, $postTitle = "some default title")
        {

        }
    }

Les param�tres sont assign�s dans le m�me ordre qu'ils sont transmis dans la route. Vous pouvez r�cup�rer arbitrairement des param�tres de la fa�on suivante:

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
            $year      = $this->dispatcher->getParam("year");
            $postTitle = $this->dispatcher->getParam("postTitle");
        }
    }

Boucle de r�partition (dispatch)
--------------------------------
La boucle de r�partition sera ex�cut�e dans le "Dispatcher" tant qu'il reste des actions � ex�cuter. Dans l'exemple pr�c�dent, seule une
action �tait ex�cut�e. Voyons maintenant comment la m�thode :code:`forward()` peut fournir un flot plus complexe d'op�ration � la boucle de r�partition en faisant
suivre le fil d'ex�cution � un autre contr�leur ou une autre action.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function showAction($year, $postTitle)
        {
            $this->flash->error(
				"Vous n'avez pas la permission d'acc�der � cette zone"
			);

            // Redirection du flux � une autre action
            $this->dispatcher->forward(
                [
                    "controller" => "users",
                    "action"     => "signin",
                ]
            );
        }
    }

Si les utilisateurs n'ont pas la permission d'acc�der � une certaine action, ils seront redirig�s vers le contr�leur "Users", action "signin".

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UsersController extends Controller
    {
        public function indexAction()
        {

        }

        public function signinAction()
        {

        }
    }

Votre application n'est pas limit�e en nombre de "forward", tant qu'il n'y a pas de r�f�rences circulaires, sinon votre application sera arr�t�e.
S'il ne reste plus d'action � r�partir dans la boucle, le r�partiteur invoquera automatiquement la couche vue du MVC qui est g�r�e par
:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`.

Initialisation des contr�leurs
------------------------------
:doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>` propose une m�thode d'initialisation qui est ex�cut�e en premier avant n'importe quelle
action du cont�leur. L'utilisation de la m�thode "__construct" est � proscrire.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public $settings;

        public function initialize()
        {
            $this->settings = [
                "mySetting" => "value",
            ];
        }

        public function saveAction()
        {
            if ($this->settings["mySetting"] === "value") {
                // ...
            }
        }
    }

.. highlights::

    La m�thode "initialize" n'est appel�e que si l'�v�nement "beforeExecuteRoute" est ex�cut� avec succ�s.
    Ceci �vite l'�xecution de l'initialiseur sans autorisation.

Si vous souhaitez proc�der � une initialisation juste apr�s la construction du contr�leur, vous pouvez d�finir
la m�thode "onConstruct":

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function onConstruct()
        {
            // ...
        }
    }

.. highlights::

    Soyez attentifs au fait que la m�thode :code:`onConstruct()` sera ex�cut�e m�me si aucune action n'existe
    dans le contr�leur, ou bien que l'utilisateur n'y ait pas acc�s (selon le contr�le d'acc�s fournit
    par le d�veloppeur).

Injection de services
---------------------
Si un contr�leur �tend :doc:`Phalcon\\Mvc\\Controller <../api/Phalcon_Mvc_Controller>` il a alors facilement acc�s au conteneur
de service de l'application. Si vous avez par exemple inscrit un service comme celui-ci:

.. code-block:: php

    <?php

    use Phalcon\Di;

    $di = new Di();

    $di->set(
        "storage",
        function () {
            return new Storage(
                "/un/repertoire"
            );
        },
        true
    );

Nous pouvons alors acc�der � ce service de plusieurs fa�ons:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class FilesController extends Controller
    {
        public function saveAction()
        {
            // Injection de service en acc�dant seulement � la propri�t� du m�me nom
            $this->storage->save('/un/fichier');

            // Acc�s au service depuis le DI
            $this->di->get('storage')->save('/un/fichier');

            // Une autre fa�on avec l'accesseur magique
            $this->di->getStorage()->save('/un/fichier');

            // Une autre fa�on avec l'accesseur magique
            $this->getDi()->getStorage()->save('/un/fichier');

            // Avec l'�criture tableau
            $this->di['storage']->save('/un/fichier');
        }
    }

Si vous utilisez Phalcon comme un framework full-stack, vous pouvez consulter les services fournis :doc:`par d�faut <di>` par le framework.

Requ�te et r�ponse
------------------
En supposant que le framework fournisse un ensemble de services pr�-inscrit, nous allons expliquer comment interagir avec l'environnement HTTP.
Le service "request" contient un instance de :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>` et le service "response" est une instance de
:doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>` qui repr�sente ce qui est renvoy� au client.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function saveAction()
        {
            // V�rifie que la requ�te utilise la m�thode POST
            if ($this->request->isPost()) {
                // Access POST data
                $customerName = $this->request->getPost("name");
                $customerBorn = $this->request->getPost("born");
            }
        }
    }

Normalement, l'objet r�ponse n'est pas utilis� directement mais il est construit avant l'ex�cution de l'action. Parfois, comme
dans l'�v�nement "afterDispatch" il peut �tre utile d'acc�der � la r�ponse directement:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function indexAction()
        {

        }

        public function notFoundAction()
        {
            // Envoi d'une ent�te de r�ponse HTTP 404
            $this->response->setStatusCode(404, "Not Found");
        }
    }

Vous apprendrez plus sur l'environnement HTTP dans les articles d�di�s � :doc:`request <request>` et :doc:`response <response>`.

Donn�es de Session
------------------
Les sessions nous aident � maintenir la persistence des donn�es entre les requ�tes. Vous pouvez acc�der � :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>`
� partir de n'importe quel contr�leur pour encapsuler les donn�es devant �tre persistantes.

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class UserController extends Controller
    {
        public function indexAction()
        {
            $this->persistent->name = "Michael";
        }

        public function welcomeAction()
        {
            echo "Welcome, ", $this->persistent->name;
        }
    }

Utilisation des Services en tant que Contr�leurs
------------------------------------------------
Les services peuvent agir comme des contr�leurs. Les classes contr�leur sont toujours int�rrog�es depuis le conteneur de services.
En cons�quence, n'importe quelle autre classe inscrite avec son nom peut ais�ment remplacer un contr�leur:

.. code-block:: php

    <?php

    // Inscription d'un contr�leur en tant que service
    $di->set(
        "IndexController",
        function () {
            $component = new Component();

            return $component;
        }
    );

    // Inscription d'un contr�leur avec espace de nom en tant que service
        $component = new Component();
    $di->set(
        "Backend\\Controllers\\IndexController",
        function () {
            $component = new Component();

            return $component;

        }
    );

Les Ev�nements dans les Contr�leurs
-----------------------------------
Les contr�leurs agissent automatiquement comme des �couteurs pour les �v�nements du :doc:`r�partiteur <dispatching>`. La r�alisation de m�thodes
avec ces noms d'�v�nements vous permet de cr�er des points d'interception avant que les actions ne soient ex�cut�es:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Controller;

    class PostsController extends Controller
    {
        public function beforeExecuteRoute($dispatcher)
        {
            // Ceci est ex�cut� avant chaque action trouv�e
            if ($dispatcher->getActionName() === "save") {
                $this->flash->error(
                    "You don't have permission to save posts"
                );

                $this->flash->error("Vous n'avez pas l'autorisation d'enregistrer des annonces");
                $this->dispatcher->forward(
                    [
                        "controller" => "home",
                        "action"     => "index",
                    ]
                );

                return false;
            }
        }

        public function afterExecuteRoute($dispatcher)
        {
            // Ex�cut�e apr�s chaque action trouv�e
        }
    }

.. _DRY: https://fr.wikipedia.org/wiki/Ne_vous_r%C3%A9p%C3%A9tez_pas
