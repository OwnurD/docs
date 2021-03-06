Url
###

.. php:namespace:: Cake\View\UrlHelper

.. php:class:: UrlHelper(View $view, array $config = [])

The UrlHelper makes it easy for you to generate URL's from your other helpers.
It also gives you a single place to customize how URLs are generated by
overriding the core helper with an application one. See the
:ref:`aliasing-helpers` section for how to do this.

Generating URLs
===============

.. php:method:: build(mixed $url = NULL, boolean $full = false)

Returns a URL pointing to a combination of controller and action.
If ``$url`` is empty, it returns the ``REQUEST\_URI``, otherwise it
generates the URL for the controller and action combo. If ``full`` is
``true``, the full base URL will be prepended to the result::

    echo $this->Url->build([
        "controller" => "posts",
        "action" => "view",
        "bar"
    ]);

    // Output
    /posts/view/bar

Here are a few more usage examples:

URL with named parameters::

    echo $this->Url->build([
        "controller" => "posts",
        "action" => "view",
        "foo" => "bar"
    ]);

    // Output
    /posts/view/foo:bar

URL with extension::

    echo $this->Url->build([
        "controller" => "posts",
        "action" => "list",
        "_ext" => "rss"
    ]);

    // Output
    /posts/list.rss

URL (starting with '/') with the full base URL prepended::

    echo $this->Url->build('/posts', true);

    // Output
    http://somedomain.com/posts

URL with GET params and named anchor::

    echo $this->Url->build([
        "controller" => "posts",
        "action" => "search",
        "?" => ["foo" => "bar"],
        "#" => "first"
    ]);

    // Output
    /posts/search?foo=bar#first

URL for named route::

    echo $this->Url->build(['_name' => 'product-page', 'slug' => 'i-m-slug']);

    // Assuming route is setup like:
    // $router->connect(
    //     '/products/:slug',
    //     [
    //         'controller' => 'products',
    //         'action' => 'view'
    //     ],
    //     [
    //         '_name' => 'product-page'
    //     ]
    // );
    /products/i-m-slug

For further information check
`Router::url <http://api.cakephp.org/3.0/class-Cake.Routing.Router.html#_url>`_
in the API.

.. meta::
    :title lang=en: UrlHelper
    :description lang=en: The role of the UrlHelper in CakePHP is to help build urls.
    :keywords lang=en: url helper,url
