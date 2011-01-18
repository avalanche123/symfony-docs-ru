.. index::
   single: Controller
   single: MVC; Controller

Контроллер
==========

Всё ещё с нами после первых двух частей? Вы становитесь ярым приверженцем Symfony2!
Давайте, без лишней суеты, узнаем что контроллеры могут сделать для вас.

.. index::
   single: Formats
   single: Controller; Formats
   single: Routing; Formats
   single: View; Formats

Использование форматов
-------------

В наши дни, web приложение должно уметь выдавать не только HTML страницы.
Начиная с XML для RSS каналов и Web служб, заканчивая JSON для Ajax запросов,
существует множество различных форматов. Поддержка этих форматов в Symfony2
проста. Измените ``routing.yml``, добавив ``_format`` со значением ``xml``:

.. configuration-block::

    .. code-block:: yaml

        # src/Application/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/{name}
            defaults: { _controller: HelloBundle:Hello:index, _format: xml }

    .. code-block:: xml

        <!-- src/Application/HelloBundle/Resources/config/routing.xml -->
        <route id="hello" pattern="/hello/{name}">
            <default key="_controller">HelloBundle:Hello:index</default>
            <default key="_format">xml</default>
        </route>

    .. code-block:: php

        // src/Application/HelloBundle/Resources/config/routing.php
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'HelloBundle:Hello:index',
            '_format'     => 'xml',
        )));

Затем, наряду с ``index.php`` добавьте шаблон ``index.xml.php``:

.. code-block:: xml+php

    # src/Application/HelloBundle/Resources/views/Hello/index.xml.php
    <hello>
        <name><?php echo $name ?></name>
    </hello>

Вот и всё что для этого нужно. Нет нужды изменять контроллер. Для стандартных
форматов Symfony2 автоматически подбирает заголовок ``Content-Type`` для ответа.
Если хотите поддержку форматов лишь для одного действия, тогда используйте
заполнитель ``{_format}`` в паттерне:

.. configuration-block::

    .. code-block:: yaml

        # src/Application/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:      /hello/{name}.{_format}
            defaults:     { _controller: HelloBundle:Hello:index, _format: html }
            requirements: { _format: (html|xml|json) }

    .. code-block:: xml

        <!-- src/Application/HelloBundle/Resources/config/routing.xml -->
        <route id="hello" pattern="/hello/{name}.{_format}">
            <default key="_controller">HelloBundle:Hello:index</default>
            <default key="_format">html</default>
            <requirement key="_format">(html|xml|json)</requirement>
        </route>

    .. code-block:: php

        // src/Application/HelloBundle/Resources/config/routing.php
        $collection->add('hello', new Route('/hello/{name}.{_format}', array(
            '_controller' => 'HelloBundle:Hello:index',
            '_format'     => 'html',
        ), array(
            '_format' => '(html|xml|json)',
        )));

Таким образом контроллер будет вызыван для следующих URL:: ``/hello/Fabien.xml``
или ``/hello/Fabien.json``. ``html`` это первоначальное значение для ``_format``,
т. е. ``/hello/Fabien`` и ``/hello/Fabien.html`` оба соотвествуют формату ``html``.

Запись ``requirements`` устанавилвает регулярные выражения, которым должны
соотвествовать заполнители. Если в этом примере запросить ресурс ``/hello/Fabien.js``
вы получите ошибку 404 HTTP, потому что он не удовлетворяет тербованию для ``_format``.

.. index::
   single: Response

Объект Response
-------------------

Теперь, давайте вернёмся к контроллеру ``Hello``::

    // src/Application/HelloBundle/Controller/HelloController.php

    public function indexAction($name)
    {
        return $this->render('HelloBundle:Hello:index.php', array('name' => $name));
    }

Метод ``render()`` заполняет шаблон и возвращает объект ``Response``. Ответ может
быть оптимизирован, перед тем как отправится в браузер, допустим, чтобы изменить
``Content-Type``::

    public function indexAction($name)
    {
        $response = $this->render('HelloBundle:Hello:index.php', array('name' => $name));
        $response->headers->set('Content-Type', 'text/plain');

        return $response;
    }

Для простейших шаблонов, вы даже можете создать объект ``Response`` вручную и
сэкономить этим несколько миллисекунд::

    public function indexAction($name)
    {
        return $this->createResponse('Hello '.$name);
    }

Это действительно полезно, когда контроллер должен отправить JSON ответ на Ajax
запрос.

.. index::
   single: Exceptions

Управление ошибками
-------------------

Когда что-нибудь не найдено, вы должны вести честную игру с протоколом HTTP и
вернуть ответ 404. Это легко сделать выдав встроенное исключение для HTTP::

    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    public function indexAction()
    {
        $product = // retrieve the object from database
        if (!$product) {
            throw new NotFoundHttpException('The product does not exist.');
        }

        return $this->render(...);
    }

``NotFoundHttpException`` вернёт в браузер ответ 404 HTTP.

.. index::
   single: Controller; Redirect
   single: Controller; Forward

Перемещения и перенаправления
-----------------------------

Если вы хотите переместить пользователя на другую страницу, используйте метод
``redirect()``::

    $this->redirect($this->generateUrl('hello', array('name' => 'Lucas')));

``generateUrl()`` такой же метод как и ``generate()``, который мы применяли ранее в
хелпере ``router``. Он получает имя маршрута и массив параметров как аргументы
и возвращает ассоциированный дружественный URL.

Также вы можете легко переместить одно действие на другое с помощью метода
``forward()``. Как и для хелпера ``actions``, он применяет внутренний подзапрос,
но возвращает объект ``Response``, что позволяет в дальнейшем его изменить если
возникнет необходимость::

    $response = $this->forward('HelloBundle:Hello:fancy', array('name' => $name, 'color' => 'green'));

    // do something with the response or return it directly

.. index::
   single: Request

Объект Request
------------------

Помимо значений заполнителей для маршрутизации, контроллер имеет доступ к
объекту ``Request``::

    $request = $this->get('request');

    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(array('en', 'fr'));

    $request->query->get('page'); // get a $_GET parameter

    $request->request->get('page'); // get a $_POST parameter

В шаблоне получить доступ к объекту ``Request`` можно через хелпер ``request``::

.. code-block:: html+php

    <?php echo $view['request']->getParameter('page') ?>

Сессия
-----------

Протокол HTTP не имеет состояний, но Symfony2 предоставляет удобный объект
сиссии, который представляет клиента (будь он человеком, использующим браузер,
ботом или web службой). Между двумя запросами Symfony2 хранит атрибуты в cookie,
используя родные сессии из PHP.

Сохранение и получение информации из сессии легко выполняется из любого
контроллера::

    $session = $this->get('request')->getSession();

    // store an attribute for reuse during a later user request
    $session->set('foo', 'bar');

    // in another controller for another request
    $foo = $session->get('foo');

    // set the user locale
    $session->setLocale('fr');

Также можно хранить небольшие сообщения, которые будут доступны для следующего
запроса::

    // store a message for the very next request (in a controller)
    $session->setFlash('notice', 'Congratulations, your action succeeded!');

    // display the message back in the next request (in a template)
    <?php echo $view['session']->getFlash('notice') ?>

Заключительное слово
--------------------

Вот и всё что хотелось рассказать, и я даже уверен, что мы не использовали все
отведённые 10 минут. В предыдущей части мы рассмотрели как расширить систему
шаблонов при помощи хелперов. Но в Symfony2 всё может быть расширено или
заменено с помощью бандлов. Это и есть тема следующей части руководства.
