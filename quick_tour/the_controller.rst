.. index::
   single: Controller
   single: MVC; Controller

Контроллер
==========

Вы все еще с нами после первых двух частей? Вы уже становитесь ярым приверженцем Symfony2! Без лишней суеты, давайте узнаем что контроллеры могут для вас сделать.

.. index::
   single: Formats
   single: Controller; Formats
   single: Routing; Formats
   single: View; Formats

Using Formats
-------------

Nowadays, a web application should be able to deliver more than just HTML
pages. From XML for RSS feeds or Web Services, to JSON for Ajax requests,
there are plenty of different formats to choose from. Supporting those formats
in Symfony2 is straightforward. Edit ``routing.yml`` and add a ``_format`` with
a value of ``xml``:

.. configuration-block::

    .. code-block:: yaml

        # src/Application/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/:name
            defaults: { _controller: HelloBundle:Hello:index, _format: xml }

    .. code-block:: xml

        <!-- src/Application/HelloBundle/Resources/config/routing.xml -->
        <route id="hello" pattern="/hello/:name">
            <default key="_controller">HelloBundle:Hello:index</default>
            <default key="_format">xml</default>
        </route>

    .. code-block:: php

        // src/Application/HelloBundle/Resources/config/routing.php
        $collection->add('hello', new Route('/hello/:name', array(
            '_controller' => 'HelloBundle:Hello:index',
            '_format'     => 'xml',
        )));

Then, add an ``index.xml.php`` template along side ``index.php``:

.. code-block:: xml+php

    # src/Application/HelloBundle/Resources/views/Hello/index.xml.php
    <hello>
        <name><?php echo $name ?></name>
    </hello>

That's all there is to it. No need to change the controller. For standard
formats, Symfony2 will also automatically choose the best ``Content-Type``
header for the response. If you want to support different formats for a single
action, use the ``:_format`` placeholder in the pattern instead:

.. configuration-block::

    .. code-block:: yaml

        # src/Application/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:      /hello/:name.:_format
            defaults:     { _controller: HelloBundle:Hello:index, _format: html }
            requirements: { _format: (html|xml|json) }

    .. code-block:: xml

        <!-- src/Application/HelloBundle/Resources/config/routing.xml -->
        <route id="hello" pattern="/hello/:name.:_format">
            <default key="_controller">HelloBundle:Hello:index</default>
            <default key="_format">html</default>
            <requirement key="_format">(html|xml|json)</requirement>
        </route>

    .. code-block:: php

        // src/Application/HelloBundle/Resources/config/routing.php
        $collection->add('hello', new Route('/hello/:name.:_format', array(
            '_controller' => 'HelloBundle:Hello:index',
            '_format'     => 'html',
        ), array(
            '_format' => '(html|xml|json)',
        )));

The controller will now be called for URLs like ``/hello/Fabien.xml`` or
``/hello/Fabien.json``. As the default value for ``_format`` is ``html``, the
``/hello/Fabien`` and ``/hello/Fabien.html`` will both match for the ``html``
format.

The ``requirements`` entry defines regular expressions that placeholders must
match. In this example, if you try to request the ``/hello/Fabien.js`` resource,
you will get a 404 HTTP error, as it does not match the ``_format`` requirement.

.. index::
   single: Response

The Response Object
-------------------

Теперь, давайте вернемся к контроллеру ``Hello``::

    // src/Application/HelloBundle/Controller/HelloController.php

    public function indexAction($name)
    {
        return $this->render('HelloBundle:Hello:index.php', array('name' => $name));
    }

The ``render()`` method renders a template and returns a ``Response`` object. The
response can be tweaked before it is sent to the browser, for instance to
change the default ``Content-Type``::

    public function indexAction($name)
    {
        $response = $this->render('HelloBundle:Hello:index.php', array('name' => $name));
        $response->headers->set('Content-Type', 'text/plain');

        return $response;
    }

Для простейших шаблонов, вы даже можете создать объект ``Response`` вручную и сэкономить этим несколько миллисекунд::

    public function indexAction($name)
    {
        return $this->createResponse('Hello '.$name);
    }

Это действительно полезно, когда контроллер должен отправить JSON ответ на Ajax запрос.

.. index::
   single: Exceptions

Управление Ошибками
-------------------

When things are not found, you should play well with the HTTP protocol and
return a 404 response. This is easily done by throwing a built-in HTTP
exception::

    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    public function indexAction()
    {
        $product = // retrieve the object from database
        if (!$product) {
            throw new NotFoundHttpException('The product does not exist.');
        }

        return $this->render(...);
    }

The ``NotFoundHttpException`` will return a 404 HTTP response back to the
browser.

.. index::
   single: Controller; Redirect
   single: Controller; Forward

Перемещения и Перенаправления
-----------------------------

Если вы хотите переместить пользователя на другую страницу, используйте метод ``redirect()``::

    $this->redirect($this->generateUrl('hello', array('name' => 'Lucas')));

The ``generateUrl()`` is the same method as the ``generate()`` method we used
on the ``router`` helper before. It takes the route name and an array of
parameters as arguments and returns the associated friendly URL.

You can also easily forward the action to another one with the ``forward()``
method. As for the ``actions`` helper, it makes an internal sub-request, but it
returns the ``Response`` object to allow for further modification if the need
arises::

    $response = $this->forward('HelloBundle:Hello:fancy', array('name' => $name, 'color' => 'green'));

    // do something with the response or return it directly

.. index::
   single: Request

The Request Object
------------------

Besides the values of the routing placeholders, the controller also has access
to the ``Request`` object::

    $request = $this->get('request');

    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(array('en', 'fr'));

    $request->query->get('page'); // get a $_GET parameter

    $request->request->get('page'); // get a $_POST parameter

In a template, you can also access the ``Request`` object via the ``request``
helper:

.. code-block:: html+php

    <?php echo $view['request']->getParameter('page') ?>

Сессия
-----------

Even if the HTTP protocol is stateless, Symfony2 provides a nice session object
that represents the client (be it a real person using a browser, a bot, or a
web service). Between two requests, Symfony2 stores the attributes in a cookie
by using the native PHP sessions.

Storing and retrieving information from the session can be easily achieved
from any controller::

    $session = $this->get('request')->getSession();

    // store an attribute for reuse during a later user request
    $session->set('foo', 'bar');

    // in another controller for another request
    $foo = $session->get('foo');

    // set the user locale
    $session->setLocale('fr');

You can also store small messages that will only be available for the very
next request::

    // store a message for the very next request (in a controller)
    $session->setFlash('notice', 'Congratulations, your action succeeded!');

    // display the message back in the next request (in a template)
    <?php echo $view['session']->getFlash('notice') ?>

Заключительное Слово
--------------------

Вот и все что хотелось рассказать, и я даже не уверен, что мы использовали все отведенные 10 минут. В предыдущей части мы видели, как расширить систему шаблонов при помощи хелперов. Но в Symfony2 все может быть расширено или заменено при помощи бандлов. Это и есть тема следующей части данного руководства.
