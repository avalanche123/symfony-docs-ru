.. index::
   single: Тесты

Тестирование
============

Как только вы пишете новую строку кода, вы также потенциально добавляете новые
ошибки. Автоматические тесты должны защитить вас и это руководство покажет как
писать модульные и функциональные тесты для приложения Symfony2.

Фреймворк для тестирования
--------------------------

Тесты Symfony2 полагаются на PHPUnit, на его лучшие методики и некоторые соглашения.
Здесь не описывается сам PHPUnit, но если он вам не знаком, можете прочесть
отличную `документацию`_.

.. note::

    Symfony2 работает с PHPUnit 3.5.11 или старше.

Изначально PHPUnit настроен чтобы искать тесты в подпапках ``Tests/`` внутри
бандлов:

.. code-block:: xml

    <!-- app/phpunit.xml.dist -->

    <phpunit bootstrap="../src/autoload.php">
        <testsuites>
            <testsuite name="Project Test Suite">
                <directory>../src/*/*Bundle/Tests</directory>
            </testsuite>
        </testsuites>

        ...
    </phpunit>

Выполнить комплект тестов для данного приложения просто:

.. code-block:: bash

    # укажите папку с конфигами в командной строке
    $ phpunit -c app/

    # или запустите phpunit из папки приложения
    $ cd app/
    $ phpunit

.. tip::

    Покрытие кода может быть получено с помощью опции ``--coverage-html``.

.. index::
   single: Тесты; Модульные тесты

Модульные тесты
---------------

Написание модульных тестов для Symfony2 не отличается от написания стандартных
модульных тестов для PHPUnit. По соглашению рекомендуется повторять структуру
папки бандла в его подпапке ``Tests/``. Таким образом пишите тесты для класса
``Acme\HelloBundle\Model\Article`` в файле
``Acme/HelloBundle/Tests/Model/ArticleTest.php``.

В модульном тесте автозагрузка уже включена через файл ``src/autoload.php``
(это настроено по умолчанию в файле ``phpunit.xml.dist``).

Выполнить тесты для заданного файла или папки также просто:

.. code-block:: bash

    # запустить все тесты для Controller
    $ phpunit -c app src/Acme/HelloBundle/Tests/Controller/

    # запустить все тесты для Model
    $ phpunit -c app src/Acme/HelloBundle/Tests/Model/

    # запустить тесты для класса Article
    $ phpunit -c app src/Acme/HelloBundle/Tests/Model/ArticleTest.php

    # запустить все тесты для целого Bundle
    $ phpunit -c app src/Acme/HelloBundle/

.. index::
   single: Тесты; Функциональные тесты

Функциональные тесты
--------------------

Функциональные тесты проверяют объединения различных слоёв приложения (от
маршрутизации до видов). Они не отличаются от модульных тестов настолько,
насколько PHPUnit позволяет это, но имеют конкретный рабочий процесс:

* Сделать запрос;
* Протестировать ответ;
* Кликнуть по ссылке или отправить форму;
* Протестировать ответ;
* Профильтровать и повторить.

Запросы, клики и отправки выполняются клиентом, который знает как общаться с
приложением. Чтобы воспользоваться таким клиентом, тесты должны наследовать класс
Symfony2 ``WebTestCase``. Стандартное издание поставляется с простым
функциональным тестом для ``DemoController``, представляющим собой следующее::

    // src/Acme/DemoBundle/Tests/Controller/DemoControllerTest.php
    namespace Acme\DemoBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class DemoControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/demo/hello/Fabien');

            $this->assertTrue($crawler->filter('html:contains("Hello Fabien")')->count() > 0);
        }
    }

Метод ``createClient()`` возвращает клиента, привязанного к текущему приложению::

    $crawler = $client->request('GET', 'hello/Fabien');

Метод ``request()`` возвращает объект ``Crawler``, используемый для выбора
элементов в Response, для кликов по ссылкам и отправке форм.

.. tip::

    Crawler может использоваться только в том случае, если содержимое Response
    это XML или HTML документ. Для других типов нужно получать содержимое Response
    через ``$client->getResponse()->getContent()``.

Чтобы кликнуть по ссылке, сначала выберите её с помощью Crawler, используя
выражение XPath или CSS селектор, затем кликните по ней с помощью Client::

    $link = $crawler->filter('a:contains("Greet")')->eq(1)->link();

    $crawler = $client->click($link);

Отправка формы происходит схожим образом: выберите кнопку на форме, по желанию
переопределите какие-нибудь значения формы, и отправьте её::

    $form = $crawler->selectButton('submit')->form();

    // устанавливает какие-нибудь значения
    $form['name'] = 'Lucas';

    // отправляет форму
    $crawler = $client->submit($form);

Каждое поле ``Form`` имеет определённые методы, зависящие от его типа::

    // заполняет поле input
    $form['name'] = 'Lucas';

    // выбирает option или radio
    $form['country']->select('France');

    // ставит галочку в checkbox
    $form['like_symfony']->tick();

    // загружает файл
    $form['photo']->upload('/path/to/lucas.jpg');

Вместо изменения одного поля за раз, можно передать массив значений методу
``submit()``::

    $crawler = $client->submit($form, array(
        'name'         => 'Lucas',
        'country'      => 'France',
        'like_symfony' => true,
        'photo'        => '/path/to/lucas.jpg',
    ));

Теперь, когда вы с лёгкостью можете перемещаться по приложению, воспользуйтесь
утверждениями чтобы проверить ожидаемые действия. Воспользуйтесь Crawler чтобы
сделать утверждения для DOM::

    // Утверждает что ответ соотвествует заданному CSS селектору.
    $this->assertTrue($crawler->filter('h1')->count() > 0);

Или проверьте содержимое Response напрямую, если хотите убедиться что его
содержимое включает какой-то текст, или что Response не является документом
XML/HTML::

    $this->assertRegExp('/Hello Fabien/', $client->getResponse()->getContent());

.. index::
   single: Tests; Assertions

Useful Assertions
~~~~~~~~~~~~~~~~~

After some time, you will notice that you always write the same kind of
assertions. To get you started faster, here is a list of the most common and
useful assertions::

    // Assert that the response matches a given CSS selector.
    $this->assertTrue($crawler->filter($selector)->count() > 0);

    // Assert that the response matches a given CSS selector n times.
    $this->assertEquals($count, $crawler->filter($selector)->count());

    // Assert the a response header has the given value.
    $this->assertTrue($client->getResponse()->headers->contains($key, $value));

    // Assert that the response content matches a regexp.
    $this->assertRegExp($regexp, $client->getResponse()->getContent());

    // Assert the response status code.
    $this->assertTrue($client->getResponse()->isSuccessful());
    $this->assertTrue($client->getResponse()->isNotFound());
    $this->assertEquals(200, $client->getResponse()->getStatusCode());

    // Assert that the response status code is a redirect.
    $this->assertTrue($client->getResponse()->isRedirected('google.com'));

.. _документацию: http://www.phpunit.de/manual/3.5/en/

.. index::
   single: Tests; Client

The Test Client
---------------

The test Client simulates an HTTP client like a browser.

.. note::

    The test Client is based on the ``BrowserKit`` and the ``Crawler``
    components.

Making Requests
~~~~~~~~~~~~~~~

The client knows how to make requests to a Symfony2 application::

    $crawler = $client->request('GET', '/hello/Fabien');

The ``request()`` method takes the HTTP method and a URL as arguments and
returns a ``Crawler`` instance.

Use the Crawler to find DOM elements in the Response. These elements can then
be used to click on links and submit forms::

    $link = $crawler->selectLink('Go elsewhere...')->link();
    $crawler = $client->click($link);

    $form = $crawler->selectButton('validate')->form();
    $crawler = $client->submit($form, array('name' => 'Fabien'));

The ``click()`` and ``submit()`` methods both return a ``Crawler`` object.
These methods are the best way to browse an application as it hides a lot of
details. For instance, when you submit a form, it automatically detects the
HTTP method and the form URL, it gives you a nice API to upload files, and it
merges the submitted values with the form default ones, and more.

.. tip::

    You will learn more about the ``Link`` and ``Form`` objects in the Crawler
    section below.

But you can also simulate form submissions and complex requests with the
additional arguments of the ``request()`` method::

    // Form submission
    $client->request('POST', '/submit', array('name' => 'Fabien'));

    // Form submission with a file upload
    $client->request('POST', '/submit', array('name' => 'Fabien'), array('photo' => '/path/to/photo'));

    // Specify HTTP headers
    $client->request('DELETE', '/post/12', array(), array(), array('PHP_AUTH_USER' => 'username', 'PHP_AUTH_PW' => 'pa$$word'));

When a request returns a redirect response, the client automatically follows
it. This behavior can be changed with the ``followRedirects()`` method::

    $client->followRedirects(false);

When the client does not follow redirects, you can force the redirection with
the ``followRedirect()`` method::

    $crawler = $client->followRedirect();

Last but not least, you can force each request to be executed in its own PHP
process to avoid any side-effects when working with several clients in the same
script::

    $client->insulate();

Browsing
~~~~~~~~

The Client supports many operations that can be done in a real browser::

    $client->back();
    $client->forward();
    $client->reload();

    // Clears all cookies and the history
    $client->restart();

Accessing Internal Objects
~~~~~~~~~~~~~~~~~~~~~~~~~~

If you use the client to test your application, you might want to access the
client's internal objects::

    $history   = $client->getHistory();
    $cookieJar = $client->getCookieJar();

You can also get the objects related to the latest request::

    $request  = $client->getRequest();
    $response = $client->getResponse();
    $crawler  = $client->getCrawler();

If your requests are not insulated, you can also access the ``Container`` and
the ``Kernel``::

    $container = $client->getContainer();
    $kernel    = $client->getKernel();

Accessing the Container
~~~~~~~~~~~~~~~~~~~~~~~

It's highly recommended that a functional test only tests the Response. But
under certain very rare circumstances, you might want to access some internal
objects to write assertions. In such cases, you can access the dependency
injection container::

    $container = $client->getContainer();

Be warned that this does not work if you insulate the client or if you use an
HTTP layer.

.. tip::

    If the information you need to check are available from the profiler, use
    them instead.

Accessing the Profiler Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To assert data collected by the profiler, you can get the profile for the
current request like this::

    $profile = $client->getProfile();

Redirecting
~~~~~~~~~~~

By default, the Client doesn't follow HTTP redirects, so that you can get
and examine the Response before redirecting. Once you do want the client
to redirect, call the ``followRedirect()`` method::

    // do something that would cause a redirect to be issued (e.g. fill out a form)

    // follow the redirect
    $crawler = $client->followRedirect();

If you want the Client to always automatically redirect, you can call the
``followRedirects()`` method::

    $client->followRedirects();

    $crawler = $client->request('GET', '/');

    // all redirects are followed

    // set Client back to manual redirection
    $client->followRedirects(false);

.. index::
   single: Tests; Crawler

The Crawler
-----------

A Crawler instance is returned each time you make a request with the Client.
It allows you to traverse HTML documents, select nodes, find links and forms.

Creating a Crawler Instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~

A Crawler instance is automatically created for you when you make a request
with a Client. But you can create your own easily::

    use Symfony\Component\DomCrawler\Crawler;

    $crawler = new Crawler($html, $url);

The constructor takes two arguments: the second one is the URL that is used to
generate absolute URLs for links and forms; the first one can be any of the
following:

* An HTML document;
* An XML document;
* A ``DOMDocument`` instance;
* A ``DOMNodeList`` instance;
* A ``DOMNode`` instance;
* An array of the above elements.

After creation, you can add more nodes:

+-----------------------+----------------------------------+
| Method                | Description                      |
+=======================+==================================+
| ``addHTMLDocument()`` | An HTML document                 |
+-----------------------+----------------------------------+
| ``addXMLDocument()``  | An XML document                  |
+-----------------------+----------------------------------+
| ``addDOMDocument()``  | A ``DOMDocument`` instance       |
+-----------------------+----------------------------------+
| ``addDOMNodeList()``  | A ``DOMNodeList`` instance       |
+-----------------------+----------------------------------+
| ``addDOMNode()``      | A ``DOMNode`` instance           |
+-----------------------+----------------------------------+
| ``addNodes()``        | An array of the above elements   |
+-----------------------+----------------------------------+
| ``add()``             | Accept any of the above elements |
+-----------------------+----------------------------------+

Traversing
~~~~~~~~~~

Like jQuery, the Crawler has methods to traverse the DOM of an HTML/XML
document:

+-----------------------+----------------------------------------------------+
| Method                | Description                                        |
+=======================+====================================================+
| ``filter('h1')``      | Nodes that match the CSS selector                  |
+-----------------------+----------------------------------------------------+
| ``filterXpath('h1')`` | Nodes that match the XPath expression              |
+-----------------------+----------------------------------------------------+
| ``eq(1)``             | Node for the specified index                       |
+-----------------------+----------------------------------------------------+
| ``first()``           | First node                                         |
+-----------------------+----------------------------------------------------+
| ``last()``            | Last node                                          |
+-----------------------+----------------------------------------------------+
| ``siblings()``        | Siblings                                           |
+-----------------------+----------------------------------------------------+
| ``nextAll()``         | All following siblings                             |
+-----------------------+----------------------------------------------------+
| ``previousAll()``     | All preceding siblings                             |
+-----------------------+----------------------------------------------------+
| ``parents()``         | Parent nodes                                       |
+-----------------------+----------------------------------------------------+
| ``children()``        | Children                                           |
+-----------------------+----------------------------------------------------+
| ``reduce($lambda)``   | Nodes for which the callable does not return false |
+-----------------------+----------------------------------------------------+

You can iteratively narrow your node selection by chaining method calls as
each method returns a new Crawler instance for the matching nodes::

    $crawler
        ->filter('h1')
        ->reduce(function ($node, $i)
        {
            if (!$node->getAttribute('class')) {
                return false;
            }
        })
        ->first();

.. tip::

    Use the ``count()`` function to get the number of nodes stored in a Crawler:
    ``count($crawler)``

Extracting Information
~~~~~~~~~~~~~~~~~~~~~~

The Crawler can extract information from the nodes::

    // Returns the attribute value for the first node
    $crawler->attr('class');

    // Returns the node value for the first node
    $crawler->text();

    // Extracts an array of attributes for all nodes (_text returns the node value)
    $crawler->extract(array('_text', 'href'));

    // Executes a lambda for each node and return an array of results
    $data = $crawler->each(function ($node, $i)
    {
        return $node->getAttribute('href');
    });

Links
~~~~~

You can select links with the traversing methods, but the ``selectLink()``
shortcut is often more convenient::

    $crawler->selectLink('Click here');

It selects links that contain the given text, or clickable images for which
the ``alt`` attribute contains the given text.

The Client ``click()`` method takes a ``Link`` instance as returned by the
``link()`` method::

    $link = $crawler->link();

    $client->click($link);

.. tip::

    The ``links()`` method returns an array of ``Link`` objects for all nodes.

Forms
~~~~~

As for links, you select forms with the ``selectButton()`` method::

    $crawler->selectButton('submit');

Notice that we select form buttons and not forms as a form can have several
buttons; if you use the traversing API, keep in mind that you must look for a
button.

The ``selectButton()`` method can select ``button`` tags and submit ``input``
tags; it has several heuristics to find them:

* The ``value`` attribute value;

* The ``id`` or ``alt`` attribute value for images;

* The ``id`` or ``name`` attribute value for ``button`` tags.

When you have a node representing a button, call the ``form()`` method to get a
``Form`` instance for the form wrapping the button node::

    $form = $crawler->form();

When calling the ``form()`` method, you can also pass an array of field values
that overrides the default ones::

    $form = $crawler->form(array(
        'name'         => 'Fabien',
        'like_symfony' => true,
    ));

And if you want to simulate a specific HTTP method for the form, pass it as a
second argument::

    $form = $crawler->form(array(), 'DELETE');

The Client can submit ``Form`` instances::

    $client->submit($form);

The field values can also be passed as a second argument of the ``submit()``
method::

    $client->submit($form, array(
        'name'         => 'Fabien',
        'like_symfony' => true,
    ));

For more complex situations, use the ``Form`` instance as an array to set the
value of each field individually::

    // Change the value of a field
    $form['name'] = 'Fabien';

There is also a nice API to manipulate the values of the fields according to
their type::

    // Select an option or a radio
    $form['country']->select('France');

    // Tick a checkbox
    $form['like_symfony']->tick();

    // Upload a file
    $form['photo']->upload('/path/to/lucas.jpg');

.. tip::

    You can get the values that will be submitted by calling the ``getValues()``
    method. The uploaded files are available in a separate array returned by
    ``getFiles()``. The ``getPhpValues()`` and ``getPhpFiles()`` also return
    the submitted values, but in the PHP format (it converts the keys with
    square brackets notation to PHP arrays).

.. index::
   pair: Tests; Configuration

Testing Configuration
---------------------

.. index::
   pair: PHPUnit; Configuration

PHPUnit Configuration
~~~~~~~~~~~~~~~~~~~~~

Each application has its own PHPUnit configuration, stored in the
``phpunit.xml.dist`` file. You can edit this file to change the defaults or
create a ``phpunit.xml`` file to tweak the configuration for your local machine.

.. tip::

    Store the ``phpunit.xml.dist`` file in your code repository, and ignore the
    ``phpunit.xml`` file.

By default, only the tests stored in "standard" bundles are run by the
``phpunit`` command (standard being tests under Vendor\\*Bundle\\Tests
namespaces). But you can easily add more namespaces. For instance, the
following configuration adds the tests from the installed third-party bundles:

.. code-block:: xml

    <!-- hello/phpunit.xml.dist -->
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

To include other namespaces in the code coverage, also edit the ``<filter>``
section:

.. code-block:: xml

    <filter>
        <whitelist>
            <directory>../src</directory>
            <exclude>
                <directory>../src/*/*Bundle/Resources</directory>
                <directory>../src/*/*Bundle/Tests</directory>
                <directory>../src/Acme/Bundle/*Bundle/Resources</directory>
                <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
            </exclude>
        </whitelist>
    </filter>

Client Configuration
~~~~~~~~~~~~~~~~~~~~

The Client used by functional tests creates a Kernel that runs in a special
``test`` environment, so you can tweak it as much as you want:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_test.yml
        imports:
            - { resource: config_dev.yml }

        framework:
            error_handler: false
            test: ~

        web_profiler:
            toolbar: false
            intercept_redirects: false

        monolog:
            handlers:
                main:
                    type:  stream
                    path:  %kernel.logs_dir%/%kernel.environment%.log
                    level: debug

    .. code-block:: xml

        <!-- app/config/config_test.xml -->
        <container>
            <imports>
                <import resource="config_dev.xml" />
            </imports>

            <webprofiler:config
                toolbar="false"
                intercept-redirects="false"
            />

            <framework:config error_handler="false">
                <framework:test />
            </framework:config>

            <monolog:config>
                <monolog:main
                    type="stream"
                    path="%kernel.logs_dir%/%kernel.environment%.log"
                    level="debug"
                 />               
            </monolog:config>
        </container>

    .. code-block:: php

        // app/config/config_test.php
        $loader->import('config_dev.php');

        $container->loadFromExtension('framework', array(
            'error_handler' => false,
            'test'          => true,
        ));

        $container->loadFromExtension('web_profiler', array(
            'toolbar' => false,
            'intercept-redirects' => false,
        ));

        $container->loadFromExtension('monolog', array(
            'handlers' => array(
                'main' => array('type' => 'stream',
                                'path' => '%kernel.logs_dir%/%kernel.environment%.log'
                                'level' => 'debug')
           
        )));

You can also change the default environment (``test``) and override the
default debug mode (``true``) by passing them as options to the
``createClient()`` method::

    $client = static::createClient(array(
        'environment' => 'my_test_env',
        'debug'       => false,
    ));

If your application behaves according to some HTTP headers, pass them as the
second argument of ``createClient()``::

    $client = static::createClient(array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

You can also override HTTP headers on a per request basis::

    $client->request('GET', '/', array(), array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

.. tip::

    To provide your own Client, override the ``test.client.class`` parameter,
    or define a ``test.client`` service.

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/testing/http_authentication`
* :doc:`/cookbook/testing/insulating_clients`
* :doc:`/cookbook/testing/profiling`
