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
   single: Тесты; Утверждения

Полезные утверждения
~~~~~~~~~~~~~~~~~~~~

Несколько позже вы заметите что всегда пишите типичные утверждения. Вот список
наиболее общих и полезных утверждений, чтобы вы смогли начать быстрее::

    // Утверждает что ответ соотвествует заданному CSS селектору.
    $this->assertTrue($crawler->filter($selector)->count() > 0);

    // Утверждает что ответ соотвествует заданному CSS селектору n раз.
    $this->assertEquals($count, $crawler->filter($selector)->count());

    // Утверждает что заголовок ответа имеет указанное значение.
    $this->assertTrue($client->getResponse()->headers->contains($key, $value));

    // Утверждает что содержимое ответа соотвествует заданному regexp.
    $this->assertRegExp($regexp, $client->getResponse()->getContent());

    // Проверяет статус код у ответа.
    $this->assertTrue($client->getResponse()->isSuccessful());
    $this->assertTrue($client->getResponse()->isNotFound());
    $this->assertEquals(200, $client->getResponse()->getStatusCode());

    // Утверждает что статус код ответа является редиректом.
    $this->assertTrue($client->getResponse()->isRedirected('google.com'));

.. _документацию: http://www.phpunit.de/manual/3.5/en/

.. index::
   single: Тесты; Клиент

Тестовый клиент
---------------

Тестовый клиент симулирует HTTP клиента, такого как браузер.

.. note::

    Тестовый клиент основан на компонентах ``BrowserKit`` и ``Crawler``.

Создание запросов
~~~~~~~~~~~~~~~~~

Клиент знает как делать запросы к приложению Symfony2::

    $crawler = $client->request('GET', '/hello/Fabien');

Метод ``request()`` принимает в качестве аргументов HTTP метод и URL и возвращает
экземпляр ``Crawler``.

Воспользуйтесь Crawler чтобы найти DOM элементы в Response. Затем их можно
использовать для кликания по ссылкам и отправки форм::

    $link = $crawler->selectLink('Go elsewhere...')->link();
    $crawler = $client->click($link);

    $form = $crawler->selectButton('validate')->form();
    $crawler = $client->submit($form, array('name' => 'Fabien'));

Методы ``click()`` и ``submit()`` возвращают объект ``Crawler``.
Они являются лучшим способом для осмотра приложения так как скрывают многие детали.
Например, метод submit, когда вы посылаете форму он автоматически определяет
HTTP метод и URL, даёт красивый API для загрузки файлов и объединяет присланные
значения со значениями по умолчанию и т. д.

.. tip::

    Больше узнать об объектах ``Link`` и ``Form`` можно в разделе Crawler.

Но вы также можете симулировать отправку форм и сложные запросы с помощью
дополнительных аргументов метода ``request()``::

    // Отправляет форму
    $client->request('POST', '/submit', array('name' => 'Fabien'));

    // Отправляет форму с загрузкой файла
    $client->request('POST', '/submit', array('name' => 'Fabien'), array('photo' => '/path/to/photo'));

    // Указывает заголовки HTTP
    $client->request('DELETE', '/post/12', array(), array(), array('PHP_AUTH_USER' => 'username', 'PHP_AUTH_PW' => 'pa$$word'));

Когда запрос возвращает ответ с перенаправлением, клиент автоматически проследует
туда. Это поведение можно изменить с помощью метода ``followRedirects()``::

    $client->followRedirects(false);

Если же клиент не следует по перенаправлениям, можно заставить его с помощью
метода ``followRedirect()``::

    $crawler = $client->followRedirect();

И последнее, но не менее важное, можно заставить каждый запрос выполняться в
собственном процессе PHP чтобы избежать любых побочных эффектов когда несколько
клиентов работают в одном скрипте::

    $client->insulate();

Браузинг
~~~~~~~~

Клиент поддерживает многие операции, свойственные настоящему браузеру::

    $client->back();
    $client->forward();
    $client->reload();

    // Очищает все куки и историю.
    $client->restart();

Получение внутренних объектов
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Когда клиент используется для тестирования приложения, возникает необходимость
получить доступ к его внутренним объектам::

    $history   = $client->getHistory();
    $cookieJar = $client->getCookieJar();

Также можно получить объекты, относящиеся к последнему запросу::

    $request  = $client->getRequest();
    $response = $client->getResponse();
    $crawler  = $client->getCrawler();

Если запросы не были изолированы, то можно получить доступ к ``Container`` и
``Kernel``::

    $container = $client->getContainer();
    $kernel    = $client->getKernel();

Получение Container
~~~~~~~~~~~~~~~~~~~

Настоятельно рекомендуется использовать функциональные тесты только для проверки
Response. Но в некоторых редких случаях необходимо получить доступ к каким-либо
внутренним объектам для написания утверждений. Для этого можно использовать
контейнер внедрения зависимости::

    $container = $client->getContainer();

Имейте в виду что это не сработает если вы изолировали клиента или использовали
HTTP слой.

.. tip::

    Если необходимая для проверки информация доступна из профилировщика, тогда
    используйте его.

Получение данных профилировщика
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы проверить данные, собранные профилировщиком, можно взять профиль текущего
запроса::

    $profile = $client->getProfile();

Перенаправление
~~~~~~~~~~~~~~~

По умолчанию клиент не следует по HTTP перенаправлениям, чтобы можно было получить
и проверить Response до перенаправления. Когда же будет необходимо перенаправить
клиента, вызовите метод ``followRedirect()``::

    // Делает что-нибудь, что вызывает перенаправление (например, заполняет форму)

    // проходит по перенаправлению
    $crawler = $client->followRedirect();

Если необходимо всегда перенаправлять клиента автоматически, можно вызвать
метод ``followRedirects()``::

    $client->followRedirects();

    $crawler = $client->request('GET', '/');

    // проходит по всем перенаправлениям

    // возвращает ручное перенаправление клиента
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
