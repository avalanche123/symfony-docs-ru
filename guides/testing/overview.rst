.. index::
   single: Tests

Тестирование
=======

Как только вы пишете новую строку кода, вы также потенциально добавляете новые
ошибки. Автоматические тесты должны защитить вас и это руководство покажет как
писать модульные и функциональные тесты для приложения Symfony2.

Фреймворк для тестирования
-----------------

Тесты Symfony2 полагаются на PHPUnit: его лучшие методики и некоторые соглашения.
Эта часть не описывает сам PHPUnit, но если вы его ещё не знаете, можете прочесть
отличную документацию `documentation`_.

.. note::

    Symfony2 работает с PHPUnit 3.5 или старше.

Изначально PHPUnit настроен искать тесты в подпапках ``Tests/`` внутри бандлов:

.. code-block:: xml

    <!-- app/phpunit.xml.dist -->

    <phpunit ... bootstrap="../src/autoload.php">
        <testsuites>
            <testsuite name="Project Test Suite">
                <directory>../src/Application/*/Tests</directory>
            </testsuite>
        </testsuites>

        ...
    </phpunit>

Выполнить комплект тестов для данного приложения просто:

.. code-block:: bash

    # specify the configuration directory on the command line
    $ phpunit -c app/

    # or run phpunit from within the application directory
    $ cd app/
    $ phpunit

.. tip::

    Покрытие кода может быть получено опцией ``--coverage-html``.

.. index::
   single: Tests; Unit Tests

Модульные тесты
----------

Написание модульных тестов для Symfony2 не отличается от написания стандартных
модульных тестов для PHPUnit. По соглашению рекомендуется повторять структуру
папки бандла в его подпапке ``Tests/``. Таким образом пишите тесты для класса
``Application\HelloBundle\Model\Article`` в файле
``Application/HelloBundle/Tests/Model/ArticleTest.php``.

В модульном тесте автозагрузка уже включена через файл ``src/autoload.php``
(это настроено по умолчанию в файле ``phpunit.xml.dist``).

Выполнить тесты для заданного файла или папки также просто:

.. code-block:: bash

    # run all tests for the Model
    $ phpunit -c app Application/HelloBundle/Tests/Model/

    # run tests for the Article class
    $ phpunit -c app Application/HelloBundle/Tests/Model/ArticleTest.php

.. index::
   single: Tests; Functional Tests

Функциональные тесты
----------------

Функциональные тесты проверяют интеграцию различных слоёв приложения (от
маршрутизации до видов). Они не отличаются от модульных тестов настолько,
насколько PHPUnit позволяет это, но имеют специфичный рабочий процесс:

* Сделать запрос;
* Протестировать ответ;
* Кликнуть по ссылке или отправить форму;
* Протестировать ответ;
* Профильтровать и повторить.

Запросы, клики и передачи выполняются клиентом, который знает как общаться с
приложением. Чтобы воспользоваться таким клиентом, тесты должны наследовать класс
Symfony2 ``WebTestCase``. Песочница предоставляет простой функциональный тест
для ``HelloController``, представляющий собой следующее:

    // src/Application/HelloBundle/Tests/Controller/HelloControllerTest.php
    namespace Application\HelloBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class HelloControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = $this->createClient();
            $crawler = $client->request('GET', '/hello/Fabien');

            $this->assertEquals(1, count($crawler->filter('html:contains("Hello Fabien")')));
        }
    }

Метод ``createClient()`` возвращает клиента, привязанного к текущему приложению:

    $crawler = $client->request('GET', 'hello/Fabien');

Метод ``request()`` возвращает объект ``Crawler``, используемый для выбора
элементов в ответе, чтобы кликать по ссылкам и отправлять формы.

.. tip::

    Crawler может использоваться только в том случае, если содержимое ответа
    это XML или HTML документ.

Выбрав с помощью Crawler ссылку, используя выражение XPath или CSS селектор,
а затем использовав Client чтобы кликнуть по ней, вы выполните клик по ссылке:

    $link = $crawler->filter('a:contains("Greet")')->eq(1)->link();

    $crawler = $client->click($link);

Отправка формы похожа; выберите кнопку на форме, по желанию переопределите
какие-нибудь значения, и отправьте соотвествующую форму:

    $form = $crawler->selectButton('submit');

    // set some values
    $form['name'] = 'Lucas';

    // submit the form
    $crawler = $client->submit($form);

Каждое поле ``Form`` имеет специализированные методы, зависящие от его типа:

    // fill an input field
    $form['name'] = 'Lucas';

    // select an option or a radio
    $form['country']->select('France');

    // tick a checkbox
    $form['like_symfony']->tick();

    // upload a file
    $form['photo']->upload('/path/to/lucas.jpg');

Вместо изменения одного поля за раз, можно также передать массив значений методу
``submit()``::

    $crawler = $client->submit($form, array(
        'name'         => 'Lucas',
        'country'      => 'France',
        'like_symfony' => true,
        'photo'        => '/path/to/lucas.jpg',
    ));

Теперь, когда вы с лёгкостью можете перемещаться по приложению, используйте
утверждения чтобы проверять что они действительно делают то, что вы ожидаете.
Воспользуйтесь Crawler чтобы сделать утверждения для DOM:

    // Assert that the response matches a given CSS selector.
    $this->assertTrue(count($crawler->filter('h1')) > 0);

Или проверьте содержимое ответа напрямую, если вы хотите убедиться что
содержимое включает какой-то текст, а также если ответ не является документом
XML/HTML::

    $this->assertRegExp('/Hello Fabien/', $client->getResponse()->getContent());

.. _documentation: http://www.phpunit.de/manual/3.5/en/
