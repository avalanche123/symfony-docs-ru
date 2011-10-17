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

    <phpunit bootstrap="bootstrap.php.cache">
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

В модульном тесте автозагрузка уже включена через файл ``bootstrap.php.cache``
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
   single: Тесты; Crawler

Crawler
-------

Экземпляр Crawler возвращается каждый раз когда выполняется запрос посредством
клиента. Он позволяет перемещаться по HTML документам, выбирать узлы, искать
ссылки и формы.

Создание экземпляра Crawler
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Экземпляр Crawler автоматически создаётся когда выполняется запрос через клиента.
Также легко можно создать его своими руками::

    use Symfony\Component\DomCrawler\Crawler;

    $crawler = new Crawler($html, $url);

Конструктор принимает два аргумента: из которых второй это URL, используемый для
создания абсолютных URL-ов для ссылок и форм, а первый может принимать следующие
значения:

* HTML документ;
* XML документ;
* экземпляр ``DOMDocument``;
* экземпляр ``DOMNodeList``;
* экземпляр ``DOMNode``;
* либо массив из перечисленных элементов.

После создания, можно добавить ещё узлов:

+-----------------------+----------------------------------+
| Метод                 | Описание                         |
+=======================+==================================+
| ``addHTMLDocument()`` | HTML документ                    |
+-----------------------+----------------------------------+
| ``addXMLDocument()``  | XML документ                     |
+-----------------------+----------------------------------+
| ``addDOMDocument()``  | экземпляр ``DOMDocument``        |
+-----------------------+----------------------------------+
| ``addDOMNodeList()``  | экземпляр ``DOMNodeList``        |
+-----------------------+----------------------------------+
| ``addDOMNode()``      | экземпляр ``DOMNode``            |
+-----------------------+----------------------------------+
| ``addNodes()``        | массив перечисленных элементов   |
+-----------------------+----------------------------------+
| ``add()``             | принимает любые перечисленные    |
|                       | выше элементы                    |
+-----------------------+----------------------------------+

Перемещения
~~~~~~~~~~~

Как и jQuery, Crawler имеет методы для перемещения по DOM документа HTML/XML:

+-----------------------+----------------------------------------------------+
| Метод                 | Описание                                           |
+=======================+====================================================+
| ``filter('h1')``      | Узлы, соотвествующие CSS селектору                 |
+-----------------------+----------------------------------------------------+
| ``filterXpath('h1')`` | Узлы, соотвествующие выражению XPath               |
+-----------------------+----------------------------------------------------+
| ``eq(1)``             | Узел с определённым индексом                       |
+-----------------------+----------------------------------------------------+
| ``first()``           | Первый узел                                        |
+-----------------------+----------------------------------------------------+
| ``last()``            | Последний узел                                     |
+-----------------------+----------------------------------------------------+
| ``siblings()``        | Дочерние узлы                                      |
+-----------------------+----------------------------------------------------+
| ``nextAll()``         | Все последующие дочерние узлы                      |
+-----------------------+----------------------------------------------------+
| ``previousAll()``     | Все предшествующие дочерние узлы                   |
+-----------------------+----------------------------------------------------+
| ``parents()``         | Родительские узлы                                  |
+-----------------------+----------------------------------------------------+
| ``children()``        | Дети                                               |
+-----------------------+----------------------------------------------------+
| ``reduce($lambda)``   | Узлы, для которых вызываемая функция не            |
|                       | возвращает false                                   |
+-----------------------+----------------------------------------------------+

Можно постепенно сузить выборку из узлов, объединяя вызовы методов в цепочки
т. к. методы возвращают экземпляр Crawler для соотвествующих узлов::

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

    Используйте функцию ``count()`` чтобы получить количество узлов, хранящихся
    в Crawler: ``count($crawler)``

Извлечение информации
~~~~~~~~~~~~~~~~~~~~~

Crawler может извлечь информацию из узлов::

    // Возвращает значение атрибута для первого узла
    $crawler->attr('class');

    // Возвращает значение узла для первого узла
    $crawler->text();

    // Извлекает массив атрибутов для всех узлов (_text возвращает значение узла)
    $crawler->extract(array('_text', 'href'));

    // Выполняет lambda для каждого узла и возвращает массив результатов
    $data = $crawler->each(function ($node, $i)
    {
        return $node->getAttribute('href');
    });

Ссылки
~~~~~~

Можно выбирать ссылки с помощью методов обхода, но сокращение ``selectLink()``
часто более удобно::

    $crawler->selectLink('Click here');

Оно выбирает ссылки, содержащие указанный текст, либо изображения, по которым
можно кликать, содержащие этот текст в атрибуте ``alt``.

Клиентский метод ``click()`` принимает экземпляр ``Link``, возвращаемый методом
``link()``::

    $link = $crawler->link();

    $client->click($link);

.. tip::

    Метод ``links()`` возвращает массив объектов ``Link`` для всех узлов.

Формы
~~~~~

Как и ссылки, формы выбирайте методом ``selectButton()``::

    $crawler->selectButton('submit');

Заметьте что выбирается кнопка на форме, а не сама форма, т. к. она может иметь
несколько кнопок; если используются API перемещений, то помните что надо искать
кнопку.

Метод ``selectButton()`` может выбрать теги ``button`` и ``input`` с типом submit;
в нём заложено несколько эвристик для их нахождения по:

* значению атрибута ``value``;

* значению атрибута ``id`` или ``alt`` для изображений;

* значению атрибута ``id`` или ``name`` для тегов ``button``.

Когда имеется узел, описывающий кнопку, вызовите метод ``form()`` чтобы получить
экземпляр ``Form``, формы обёртывающей его::

    $form = $crawler->form();

При вызове метода ``form()`` можно передать массив значений для полей,
перезаписывающих начальные значения::

    $form = $crawler->form(array(
        'name'         => 'Fabien',
        'like_symfony' => true,
    ));

А если надо симулировать определённый HTTP метод для формы, передайте его вторым
аргументом::

    $form = $crawler->form(array(), 'DELETE');

Клиент может отправлять эзкемпляры ``Form``::

    $client->submit($form);

Значения полей могут быть переданы вторым аргументом метода ``submit()``::

    $client->submit($form, array(
        'name'         => 'Fabien',
        'like_symfony' => true,
    ));

В более сложных случаях, используйте экземпляр ``Form`` как массив чтобы задать
значения каждого поля индивидуально::

    // Изменяет значение поля
    $form['name'] = 'Fabien';

Здесь тоже есть красивый API для управления значениями полей в зависимости от
их типов::

    // Выбирает option или radio
    $form['country']->select('France');

    // Ставит галочку в checkbox
    $form['like_symfony']->tick();

    // Загружает файл
    $form['photo']->upload('/path/to/lucas.jpg');

.. tip::

    Можно получить значения, которые будут отправлены, вызвав метод ``getValues()``.
    Загружаемые файлы доступны в отдельном массиве, возвращаемом через
    ``getFiles()``. ``getPhpValues()`` и ``getPhpFiles()`` тоже возвращают
    значения для отправки, но в формате PHP (он преобразует ключи с квадратными
    скобками в PHP массивы).

.. index::
   pair: Тесты; Конфигурация

Тестовая конфигурация
---------------------

.. index::
   pair: PHPUnit; Конфигурация

PHPUnit конфигурация
~~~~~~~~~~~~~~~~~~~~

Каждое приложение имеет свою конфигурацию PHPUnit, хранящуюся в файле
``phpunit.xml.dist``. Можете отредактировать его чтобы изменить начальные
установки или создать файл ``phpunit.xml``, чтобы подстроить конфигурацию под
локальную машину.

.. tip::

    Храните файл ``phpunit.xml.dist`` в своём репозитории кода и игнорируйте
    файл ``phpunit.xml``.

Только тесты, хранящиеся в "стандартных" бандлах, запускаются через
``phpunit`` по умолчанию (стандартными будут тесты из пространства имён
Vendor\\*Bundle\\Tests). Хотя легко можно добавить ещё пространства имён.
Например, следующая конфигурация добавляет тесты из установленных third-party
бандлов:

.. code-block:: xml

    <!-- hello/phpunit.xml.dist -->
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

Чтобы включить другие пространства имён в покрытие кода, подправьте раздел
``<filter>``:

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

Конфигурация клиента
~~~~~~~~~~~~~~~~~~~~

Клиент, используемый в функциональныйх тестах, создаёт Kernel, который
запускается в специальной среде ``test``, т. о. можно настроить его так, как это
будет необходимо:

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

Также можно изменить среду (``test``) и режим отладки (``true``), заданные по
умолчанию, передав их методу ``createClient()`` в виде опций::

    $client = static::createClient(array(
        'environment' => 'my_test_env',
        'debug'       => false,
    ));

Если приложение зависит от каких-либо HTTP заголовков, передайте их вторым
аргументом ``createClient()``::

    $client = static::createClient(array(), array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

Также можно изменять HTTP заголовки для каждого запроса::

    $client->request('GET', '/', array(), array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

.. tip::

    Чтобы указать своего собственного клиента, измените параметр
    ``test.client.class`` или установите службу ``test.client``.

Узнайте больше из Рецептов
--------------------------

* :doc:`/cookbook/testing/http_authentication`
* :doc:`/cookbook/testing/insulating_clients`
* :doc:`/cookbook/testing/profiling`

.. toctree::
    :hidden:

    Translation source: n/a
    Corrected from: 2011-10-16 a591960
