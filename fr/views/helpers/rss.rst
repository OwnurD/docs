RSSHelper
#########

.. php:namespace:: Cake\View\Helper

.. php:class:: RssHelper(View $view, array $config = [])

Le Helper RSS permet de générer facilement le XML pour les
`flux RSS <https://en.wikipedia.org/wiki/RSS>`_.

Créer un flux RSS avec RssHelper
================================

Cet exemple suppose que vous ayez un Controller Posts et un Model Post
déjà créés et que vous vouliez créer une vue alternative pour les flux RSS.

Créer une version xml/rss de posts/index est vraiment simple avec
CakePHP. Après quelques étapes faciles, vous pouvez tout simplement ajouter
l'extension .rss demandée à ``posts/index`` pour en faire votre URL
``posts/index.rss``. Avant d'aller plus loin en essayant d'initialiser et
de lancer notre service Web, nous avons besoin de faire un certain nombre
de choses. Premièrement, le parsing d'extensions doit être activé dans
``config/routes.php``::

    Router::extensions('rss');

Dans l'appel ci-dessus, nous avons activé l'extension .rss. Quand vous
utilisez :php:meth:`Cake\\Routing\\Router::extensions()`, vous pouvez passer autant
d'arguments ou d'extensions que vous le souhaitez. Cela activera le
content-type de chaque extension utilisée dans votre application. Maintenant,
quand l'adresse ``posts/index.rss`` est demandée, vous obtiendrez une version
XML de votre ``posts/index``. Cependant, nous avons d'abord besoin d'éditer
le controller pour y ajouter le code "rss-spécifique".

Code du Controller
------------------

C'est une bonne idée d'ajouter RequestHandler dans la méthode ``initialize()``
de votre controller Posts. Cela permettra beaucoup d'automagie::

    public function initialize()
    {
        parent::initialize();
        $this->loadComponent('RequestHandler');
    }

Notre vue utilise aussi :php:class:`TextHelper` pour le formatage, ains'il
doit aussi être ajouté au controller::

    public $helpers = ['Text'];

Avant que nous puissions faire une version RSS de notre posts/index, nous
avons besoin de mettre certaines choses en ordre. Il pourrait être tentant
de mettre le canal des métadonnées dans l'action du controller et de le
passer à votre vue en utilisant la méthode :php:meth:`Controller::set()`,
mais ceci est inapproprié. Cette information pourrait également aller dans
la vue. Cela arrivera sans doute plus tard, mais pour l'instant, si vous
avez un ensemble de logique différent entre les données utilisées pour créer
le flux RSS et les données pour la page HTML, vous pouvez utiliser la méthode
:php:meth:`RequestHandler::isRss()`, sinon votre controller pourrait rester
le même::

    // Modifie l'action du Controller Posts correspondant à
    // l'action qui délivre le flux rss, laquelle est
    // l'action index dans notre exemple

    public function index()
    {
        if ($this->RequestHandler->isRss() ) {
            $posts = $this->Post->find(
                'all',
                ['limit' => 20, 'order' => 'Post.created DESC']
            );
            return $this->set(compact('posts'));
        }

        // ceci n'est pas une requête RSS
        // donc on retourne les données utilisées par l'interface du site web
        $this->paginate['Post'] = [
            'order' => 'Post.created DESC',
            'limit' => 10
        ];

        $posts = $this->paginate();
        $this->set(compact('posts'));
    }

Maintenant que toutes ces variables de Vue sont définies, nous avons besoin de
créer un layout rss.

Layout
------

Un layout Rss est très simple, mettez les contenus suivants dans
``src/Template/Layout/rss/default.ctp``::

    if (!isset($documentData)) {
        $documentData = [];
    }
    if (!isset($channelData)) {
        $channelData = [];
    }
    if (!isset($channelData['title'])) {
        $channelData['title'] = $this->fetch('title');
    }
    $channel = $this->Rss->channel([], $channelData, $this->fetch('content'));
    echo $this->Rss->document($documentData, $channel);

Il ne ressemble pas à plus mais grâce à la puissance du ``RssHelper``
il fait beaucoup pour améliorer le visuel pour nous. Nous n'avons pas défini
``$documentData`` ou ``$channelData`` dans le controller, cependant dans
CakePHP vos vues peuvent retourner les variables au layout. Ce qui est
l'endroit où notre tableau ``$channelData`` va venir définir toutes les
données meta pour notre flux.

Ensuite il y a le fichier de vue pour mes posts/index. Un peu comme le fichier
de layout que nous avons crée, nous avons besoin de créer un répertoire
``src/Template/Posts/rss/`` et un nouveau ``index.ctp`` à l'intérieur de ce répertoire.
Les contenus du fichier sont ci-dessous.

View
----

Notre vue, localisée dans ``src/Template/Posts/rss/index.ctp``, commence par
définir les variables ``$documentData`` et ``$channelData`` pour le layout,
celles-ci contiennent toutes les metadonnées pour notre flux RSS. C'est fait
en utilisant la méthode :php:meth:`View::set()`` qui est analogue à la
méthode Controller::set(). Ici nous passons les canaux de données en retour au
layout::

    $this->set('channelData', [
        'title' => __("Most Recent Posts"),
        'link' => $this->Html->url('/', true),
        'description' => __("Most recent posts."),
        'language' => 'en-us']);

La seconde partie de la vue génére les éléments pour les enregistrements
actuels du flux. Ceci est accompli en bouclant sur les données qui ont
été passées à la vue ($items) et en utilisant la méthode
:php:meth:`RssHelper::item()`. L'autre méthode que vous pouvez utiliser
:php:meth:`RssHelper::items()` qui prend un callback et un tableau des items
pour le flux. (La méthode que j'ai vu utilisée pour le callback a toujours
été appelée ``transformRss()``. Il y a un problème pour cette méthode, qui est
qu'elle n'utilise aucune des classes de helper pour préparer vos données à
l'intérieur de la méthode de callback parce que la portée à l'intérieur de la
méthode n'inclut pas tout ce qui n'est pas passé à l'intérieur, ainsi ne
donne pas accès au TimeHelper ou à tout autre helper dont vous auriez besoin.
:php:meth:`RssHelper::item()` transforme le tableau associatif en un élément
pour chaque pair de valeur de clé.

.. note::

    Vous devrez modifier la variable $postLink comme il se doit pour
    votre application.

::

    foreach ($posts as $post) {
        $postTime = strtotime($post['Post']['created']);

        $postLink = [
            'controller' => 'Posts',
            'action' => 'view',
            'year' => date('Y', $postTime),
            'month' => date('m', $postTime),
            'day' => date('d', $postTime),
            $post['Post']['slug']
        ];

        // Retire & échappe tout HTML pour être sûr que le contenu va être validé.
        $bodyText = h(strip_tags($post['Post']['body']));
        $bodyText = $this->Text->truncate($bodyText, 400, [
            'ending' => '...',
            'exact'  => true,
            'html'   => true,
        ]);

        echo  $this->Rss->item([], [
            'title' => $post['Post']['title'],
            'link' => $postLink,
            'guid' => ['url' => $postLink, 'isPermaLink' => 'true'],
            'description' => $bodyText,
            'pubDate' => $post['Post']['created']
        ]);
    }

Vous pouvez voir ci-dessus que nous pouvons utiliser la boucle pour préparer
les données devant être transformées en elements XML. Il est important de
filtrer tout texte de caractères non brute en-dehors de la description,
spécialement si vous utilisez un éditeur de texte riche pour le corps de votre
blog. Dans le code ci-dessus nous utilisons ``strip_tags()`` et
:php:func:`h()` pour retirer/échapper tout caractère spécial XML du contenu,
puisqu'ils peuvent entraîner des erreurs de validation. Une fois que nous avons
défini les données pour le feed, nous pouvons ensuite utiliser la méthode
:php:meth:`RssHelper::item()` pour créer le XML dans le format RSS. Une fois
que vous avez toutes ces configurations, vous pouvez tester votre feed RSS
en allant à votre ``/posts/index.rss`` et que vous verrez votre nouveau feed.
Il est toujours important que vous validiez votre feed RSS avant de le mettre
en live. Ceci peut être fait en visitant les sites qui valident le XML comme
Le Validateur de Feed ou le site de w3c à http://validator.w3.org/feed/.

.. note::

    Vous aurez besoin de définir la valeur de 'debug' dans votre configuration
    du cœur à ``false`` pour obtenir un flux valide, à cause des différentes
    informations de debug ajoutées automatiquement sous des paramètres de
    debug plus haut qui cassent la syntaxe XML ou les règles de validation du
    flux.

.. meta::
    :title lang=fr: RssHelper
    :description lang=fr: RSSHelper permet de générer facilement les XML pour les flux RSS.
    :keywords lang=fr: rss helper,rss feed,isrss,rss item,channel data,document data,parse extensions,request handler
