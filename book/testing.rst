.. index::
   single: Тесты

Тестирование
============

Как только вы пишете новую строку кода, вы также потенциально добавляете новые
ошибки. Для того чтобы создавать надёжные приложения, вы должны использовать
как функциональные, так и модульные (unit) тесты.

Тестовый фреймворк PHPUnit
--------------------------

В Symfony2 интегрирована поддержка независимой библиотеки - называемой PHPUnit - чтобы
предоставить вам отличный тестовый фреймворк. Эта глава не покрывает все нюансы PHPUnit,
так как вы всегда можете почитать его подробную `документацию`_.

.. note::

    Symfony2 работает с PHPUnit 3.5.11 или старше.

Каждый тест - вне зависимости от того функциональный он или модульный - это
PHP класс, который расположен в поддиректории `Tests/` вашиг пакетов. Если вы
будете следовать этому правилу, то вы сможете запускать все тесты вашего приложения
при помощи команды:

.. code-block:: bash

    # укажите папку с конфигами в командной строке
    $ phpunit -c app/

Опция ``-c`` указывает PHPUnit искать конфигурационный файл в директории ``app/``.
Если вы интересуетесь опциями PHPUnit, обратите внимание на файл ``app/phpunit.xml.dist``.

.. tip::

    Покрытие кода может быть получено с помощью опции ``--coverage-html``.

.. index::
   single: Тесты; Модульные тесты

Модульные тесты
---------------

Модульный тест - это как правило тест одного отдельного PHP класса. Если вы
хотите тестировать поведение вашего приложения целиком, обратитесь к секции
`Функциональные тесты`_.

Написание модульных тестов в Symfony2 не отличается от написания стандартных
модульных тестов PHPUnit. Например, предположим, у вас есть *очень* простой
класс ``Calculator`` в директории ``Utility/`` вашего пакета:

.. code-block:: php

    <?php
    // src/Acme/DemoBundle/Utility/Calculator.php
    namespace Acme\DemoBundle\Utility;

    class Calculator
    {
        public function add($a, $b)
        {
            return $a + $b;
        }
    }

Для того, чтобы его протестировать, создайте файл ``CalculatorTest`` в директории
``Tests/Utility`` вашего пакета:

.. code-block:: php

    <?php
    // src/Acme/DemoBundle/Tests/Utility/CalculatorTest.php
    namespace Acme\DemoBundle\Tests\Utility;

    use Acme\DemoBundle\Utility\Calculator;

    class CalculatorTest extends \PHPUnit_Framework_TestCase
    {
        public function testAdd()
        {
            $calc = new Calculator();
            $result = $calc->add(30, 12);

            // assert that our calculator added the numbers correctly!
            $this->assertEquals(42, $result);
        }
    }

.. note::

    По соглашению, под-директория ``Tests/`` должна повторять структуру директорий
    вашего пакета. Т.о. если вы тестируете класс вашего пакета из директории
    ``Utility/``, поместите тест в директорию ``Tests/Utility/``.

Как и в вашем приложении, автозагрузка включается автоматически при помощи
файла ``bootstrap.php.cache`` (это по умолчанию настроено в файле ``phpunit.xml.dist``).

Выполнить тесты для заданного файла или папки также просто:

.. code-block:: bash

    # run all tests in the Utility directory
    $ phpunit -c app src/Acme/DemoBundle/Tests/Utility/

    # run tests for the Calculator class
    $ phpunit -c app src/Acme/DemoBundle/Tests/Utility/CalculatorTest.php

    # запустить все тесты для целого Bundle
    $ phpunit -c app src/Acme/DemoBundle/

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

Ваш первый функциональный тест
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Функциональные тесты - это простые PHP классы, которые, как правило, располагаются
в директории пакета ``Tests/Controller``. Если вы хотите протестировать страницы,
которые содержит ваш класс ``DemoController``, создайте новый класс, который
расширяет специальный класс ``WebTestCase``.

Например, Symfony2 Standard Edition предоставляет простой функциональный тест для
его ``DemoController`` (`DemoControllerTest`_), который выглядит так:

.. code-block:: php

    <?php
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

.. tip::

    Для запуска ваших функциональных тестов, класс ``WebTestCase``
    загружает ядро вашего приложения. В большинстве случаев, это происходит
    автоматически. Тем не менее, если ваше ядро находится в нестандартной
    директории, вам нужно модифицировать файл ``phpunit.xml.dist`` и
    установить переменную среды ``KERNEL_DIR`` на директорию вашего ядра::

        <phpunit
            <!-- ... -->
            <php>
                <server name="KERNEL_DIR" value="/path/to/your/app/" />
            </php>
            <!-- ... -->
        </phpunit>

Метод ``createClient()`` возвращает клиент, который напоминает браузер, который
вы используете для просмотра вашего сайта::

    $crawler = $client->request('GET', '/demo/hello/Fabien');

Метод ``request()`` (см. :ref:`подробнее о методе request <book-testing-request-method-sidebar>`)
возвращает объект :class:`Symfony\\Component\\DomCrawler\\Crawler`, который может быть
использован для выбора элементов в Response, кликов по ссылкам и отправки форм.

.. tip::

    Crawler может использоваться только в том случае, если содержимое Response
    это XML или HTML документ. Для других типов нужно получать содержимое Response
    через ``$client->getResponse()->getContent()``.

Давайте кликнем по ссылке, выбрав её при помощи Crawler используя XPath
или CSS селектор, затем используем Client для собственно клика. Например,
следующий код находит все ссылки с текстом ``Greet``, затем выбирает
вторую из них и кликает на неё::

    $link = $crawler->filter('a:contains("Greet")')->eq(1)->link();

    $crawler = $client->click($link);

Отправка формы происходит схожим образом: выберите кнопку на форме, по желанию
переопределите какие-нибудь значения формы, и отправьте её:

.. code-block:: php

    <?php
    $form = $crawler->selectButton('submit')->form();

    // устанавливает какие-нибудь значения
    $form['name'] = 'Lucas';
    $form['form_name[subject]'] = 'Hey there!';

    // отправляет форму
    $crawler = $client->submit($form);

.. tip::

    Форма также поддерживет загрузку файлов и содержит методы для заполнения различных типов
    полей (например, ``select()`` и ``tick()``). Подробнее читайте в секции `Формы`_
    ниже.

Теперь, когда вы с лёгкостью можете перемещаться по приложению, воспользуйтесь
утверждениями чтобы проверить ожидаемые действия. Воспользуйтесь Crawler чтобы
сделать утверждения для DOM::

    // Утверждает что ответ соотвествует заданному CSS селектору.
    $this->assertTrue($crawler->filter('h1')->count() > 0);

Или проверьте содержимое Response напрямую, если хотите убедиться что его
содержимое включает какой-то текст, или что Response не является документом
XML/HTML::

    $this->assertRegExp('/Hello Fabien/', $client->getResponse()->getContent());

.. _book-testing-request-method-sidebar:

.. sidebar:: Подробнее о методе ``request()``:

    Полная сигнатура метода ``request()``:

        .. code-block:: php

            <?php
            request(
                $method,
                $uri,
                array $parameters = array(),
                array $files = array(),
                array $server = array(),
                $content = null,
                $changeHistory = true
            )

    Массив ``server`` - это значения, которые вы как правило ожидаете найти в
    суперглобальном массиве `$_SERVER`_. Например, для того, чтобы установить
    HTTP заголовки `Content-Type` и `Referer` вы должны передать следующее:

        .. code-block:: php

            <?php
            $client->request(
                'GET',
                '/demo/hello/Fabien',
                array(),
                array(),
                array(
                    'CONTENT_TYPE' => 'application/json',
                    'HTTP_REFERER' => '/foo/bar',
                )
            );

.. index::
   single: Tests; Assertions

.. sidebar: Полезные утверждения
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    Для быстрого старта обратите внимание на список наиболе типовых
    и полезных утверждений:

    .. code-block:: php

        <?php
        // Утверждает что имеется единственный таг h2 с классом "subtitle"
        $this->assertTrue($crawler->filter('h2.subtitle')->count() > 0);

        // Утверждает что на странице имеется 4 тага h2
        $this->assertEquals(4, $crawler->filter('h2')->count());

        // Утверждает что заголовок "Content-Type" - "application/json"
        $this->assertTrue($client->getResponse()->headers->contains('Content-Type', 'application/json'));

        // Утверждает что тело ответа соответствует регулярному выражению
        $this->assertRegExp('/foo/', $client->getResponse()->getContent());

        // Утверждает что статус-код ответа 2xx
        $this->assertTrue($client->getResponse()->isSuccessful());
        // Утверждает что статус-код ответа 404
        $this->assertTrue($client->getResponse()->isNotFound());
        // Утверждает что статус-код ответа точно 200
        $this->assertEquals(200, $client->getResponse()->getStatusCode());

        // Утверждает что ответ - это перенаправление на /demo/contact
        $this->assertTrue($client->getResponse()->isRedirect('/demo/contact'));
        // или просто проверяет, что ответ - это перенаправление на любой URL
        $this->assertTrue($client->getResponse()->isRedirect());

.. index::
   single: Тесты; Клиент

Работаем с Тестовым клиентом
----------------------------

Тестовый клиент симулирует HTTP клиент (как правило это браузер) и выполняет запросы к
вашему Symfony2 приложению::

    $crawler = $client->request('GET', '/hello/Fabien');

Метод ``request()`` принимает в качестве аргументов HTTP метод и URL и возвращает экземпляр
``Crawler``.

Используйет Crawler для нахождения DOM-элементов в теле Response. После эти элементы могут
быть использованы для кликов по ссылкам и отправки форм:

.. code-block:: php

    <?php
    $link = $crawler->selectLink('Go elsewhere...')->link();
    $crawler = $client->click($link);

    $form = $crawler->selectButton('validate')->form();
    $crawler = $client->submit($form, array('name' => 'Fabien'));

Методы ``click()`` и ``submit()`` возвращают объект ``Crawler``.
Эти методы - лучший способ просматривать ваше приложение, так как
они заботятся о многих вещах, например определении HTTP метода формы,
и предоставляют вам удобный API для загрузки файлов.

.. tip::

    Больше узнать об объектах ``Link`` и ``Form`` можно в разделе
    :ref:`Crawler <book-testing-crawler>`.

Метод ``request`` может также быть использован для симуляции отправки форм
или для выполнения более сложных запросов:

.. code-block:: php

    <?php
    // Прямая отправка формы (можно и так, но легче использовать Crawler!)
    $client->request('POST', '/submit', array('name' => 'Fabien'));

    // Отправка формы с загрузкой файла
    use Symfony\Component\HttpFoundation\File\UploadedFile;

    $photo = new UploadedFile(
        '/path/to/photo.jpg',
        'photo.jpg',
        'image/jpeg',
        123
    );
    // или
    $photo = array(
        'tmp_name' => '/path/to/photo.jpg',
        'name' => 'photo.jpg',
        'type' => 'image/jpeg',
        'size' => 123,
        'error' => UPLOAD_ERR_OK
    );
    $client->request(
        'POST',
        '/submit',
        array('name' => 'Fabien'),
        array('photo' => $photo)
    );

    // Выполнение DELETE запросов, и отправка HTTP заголовков
    $client->request(
        'DELETE',
        '/post/12',
        array(),
        array(),
        array('PHP_AUTH_USER' => 'username', 'PHP_AUTH_PW' => 'pa$$word')
    );

И последнее, но не менее важное, можно заставить каждый запрос выполняться в
собственном процессе PHP чтобы избежать любых побочных эффектов когда несколько
клиентов работают в одном скрипте::

    $client->insulate();

Браузинг
~~~~~~~~

Клиент поддерживает многие операции, свойственные настоящему браузеру:

.. code-block:: php

    <?php
    $client->back();
    $client->forward();
    $client->reload();

    // Очищает все куки и историю.
    $client->restart();

Получение внутренних объектов
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Когда клиент используется для тестирования приложения, возникает необходимость
получить доступ к его внутренним объектам:

.. code-block:: php

    <?php
    $history   = $client->getHistory();
    $cookieJar = $client->getCookieJar();

Также можно получить объекты, относящиеся к последнему запросу:

.. code-block:: php

    <?php
    $request  = $client->getRequest();
    $response = $client->getResponse();
    $crawler  = $client->getCrawler();

Если запросы не были изолированы, то можно получить доступ к ``Container`` и
``Kernel``:

.. code-block:: php

    <?php
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
HTTP слой. Для получения списка служб, доступных в вашем приложении, используйте
консольную команду ``container:debug``.

.. tip::

    Если необходимая для проверки информация доступна из профилировщика, тогда
    используйте его.

Получение данных профилировщика
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для каждого запроса профайлер Symfony собирает и сохраняет множество данных
о том как обрабатывается этот запрос. Например, профайлер может быть использован
для верификации, что данная страница выполняет SQL запросов меньше, чем некоторое
пороговое значение.

Для получения профайлера для последнего запроса выполните следующий код::

    $profile = $client->getProfile();

Подробнее про использование профайлера в тестах читайте в книге рецептов:
:doc:`/cookbook/testing/profiling`.

Перенаправление
~~~~~~~~~~~~~~~

Когда запрос возвращает ответ с перенаправлением, клиент автоматически следует
ему. Если вы хотите проверить ответ перед перенаправлением, вы можете указать клиенту
не следовать перенаправлению припомощи метода ``followRedirects()``::

    $client->followRedirects(false);

Если клиент не следует перенаправлениям, вы можете форсировать перенаправление при помощи
метода ``followRedirect()``::

    $crawler = $client->followRedirect();

.. _book-testing-crawler:

.. index::
   single: Тесты; Crawler

Crawler
-------

Экземпляр Crawler возвращается каждый раз когда выполняется запрос посредством
клиента. Он позволяет перемещаться по HTML документам, выбирать узлы, искать
ссылки и формы.

Перемещения
~~~~~~~~~~~

Как и jQuery, Crawler имеет методы для перемещения по DOM документа HTML/XML. Например,
следующий код находит все элементы ``input[type=submit]``, выбирает последний на странице и
выбирает ближайший родительский элемент:

.. code-block:: php

    <?php
    $newCrawler = $crawler->filter('input[type=submit]')
        ->last()
        ->parents()
        ->first()
    ;

Также доступны следующие методы:

+------------------------+----------------------------------------------------+
| Метод                  | Описание                                           |
+========================+====================================================+
| ``filter('h1.title')`` | Ноды, соответствующие CSS селектору                |
+------------------------+----------------------------------------------------+
| ``filterXpath('h1')``  | Ноды, соответствующие выражению XPath              |
+------------------------+----------------------------------------------------+
| ``eq(1)``              | Ноды с определённым индексом                       |
+------------------------+----------------------------------------------------+
| ``first()``            | Первый нод                                         |
+------------------------+----------------------------------------------------+
| ``last()``             | Последний нод                                      |
+------------------------+----------------------------------------------------+
| ``siblings()``         | Элементы одного уровня (сёстры)                    |
+------------------------+----------------------------------------------------+
| ``nextAll()``          | Все последующие сёстры                             |
+------------------------+----------------------------------------------------+
| ``previousAll()``      | Все предыдущие сёстры                              |
+------------------------+----------------------------------------------------+
| ``parents()``          | Родительские ноды                                  |
+------------------------+----------------------------------------------------+
| ``children()``         | Потомки                                            |
+------------------------+----------------------------------------------------+
| ``reduce($lambda)``    | Ноды, для которых функция не возвращает false      |
+------------------------+----------------------------------------------------+

Так как каждый метод возвращает новый экземпляр ``Crawler``, вы можете
упростить ваш код путём выстраивания вызовов в цепочку:

.. code-block:: php

    <?php
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

Crawler может извлечь информацию из узлов:

.. code-block:: php

    <?php
    // Возвращает значение атрибута для первого узла
    $crawler->attr('class');

    // Возвращает значение узла для первого узла
    $crawler->text();

    // Возвращает массив для каждого элемента с его значением и ссылкой
    $info = $crawler->extract(array('_text', 'href'));

    // Выполняет lambda для каждого узла и возвращает массив результатов
    $data = $crawler->each(function ($node, $i)
    {
        return $node->attr('href');
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

    $form = $buttonCrawlerNode->form();

При вызове метода ``form()`` можно передать массив значений для полей,
перезаписывающих начальные значения::

    $form = $buttonCrawlerNode->form(array(
        'name'              => 'Fabien',
        'my_form[subject]'  => 'Symfony rocks!',
    ));

А если надо симулировать определённый HTTP метод для формы, передайте его вторым
аргументом::

    $form = $crawler->form(array(), 'DELETE');

Клиент может отправлять эзкемпляры ``Form``::

    $client->submit($form);

Значения полей могут быть переданы вторым аргументом метода ``submit()``::

    $client->submit($form, array(
        'name'              => 'Fabien',
        'my_form[subject]'  => 'Symfony rocks!',
    ));

В более сложных случаях, используйте экземпляр ``Form`` как массив чтобы задать
значения каждого поля индивидуально::

    // Изменяет значение поля
    $form['name'] = 'Fabien';
    $form['my_form[subject]'] = 'Symfony rocks!';

Здесь тоже есть красивый API для управления значениями полей в зависимости от
их типов::

    // Выбирает option или radio
    $form['country']->select('France');

    // Ставит галочку в checkbox
    $form['like_symfony']->tick();

    // Загружает файл
    $form['photo']->upload('/path/to/lucas.jpg');

.. tip::

    Можно получить значения, которые будут отправлены, вызвав метод ``getValues()`` объекта ``Form``.
    Загружаемые файлы доступны в отдельном массиве, возвращаемом через
    ``getFiles()``. ``getPhpValues()`` и ``getPhpFiles()`` тоже возвращают
    значения для отправки, но в формате PHP (он преобразует ключи с квадратными
    скобками - например, ``my_form[subject]`` - в PHP массивы).

.. index::
   pair: Тесты; Конфигурация

Тестовая конфигурация
---------------------

.. index::
   pair: PHPUnit; Конфигурация

PHPUnit конфигурация
~~~~~~~~~~~~~~~~~~~~

Каждое приложение имеет свою собственную конфигурацию PHPUnit, которая хранится
в файле ``phpunit.xml.dist``. Вы можете редактировать этот файл и менять значения
по умолчанию или же вы можете создать файл ``phpunit.xml`` для подгонки
конфигурации на вашей локальной машине.

.. tip::

    Сохраниете файл ``phpunit.xml.dist`` в вашем репозитории и игнорьте файл ``phpunit.xml``.

По умолчанию, по команде ``phpunit`` запускаются только тесты из "стандартных" пакетов
(стандартными считаются тесты в директориях ``src/*/Bundle/Tests`` или ``src/*/Bundle/*Bundle/Tests``),
но вы можете запросто добавить больше директорий. Например, следующая конфигурация
добавляет тесты сторонних пакетов:

.. code-block:: xml

    <!-- hello/phpunit.xml.dist -->
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

Для того, чтобы включить прочие директории в отчёт по покрытию кода, необходимо
отредактировать секцию ``<filter>``:

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

Узнайте больше из Рецептов
--------------------------

* :doc:`/cookbook/testing/http_authentication`
* :doc:`/cookbook/testing/insulating_clients`
* :doc:`/cookbook/testing/profiling`

.. _`DemoControllerTest`: https://github.com/symfony/symfony-standard/blob/master/src/Acme/DemoBundle/Tests/Controller/DemoControllerTest.php
.. _`$_SERVER`: http://php.net/manual/en/reserved.variables.server.php
.. _документацию: http://www.phpunit.de/manual/3.5/en/

.. toctree::
    :hidden:

    Translation source: n/a
    Corrected from: 2011-10-16 a591960
    Corrected from: 2011-11-29 b620a57
