.. index::
   single: Контроллер
   single: MVC; Контроллер

Контроллер
==========

Вы все еще с нами после первых двух частей? Вы уже становитесь ярым приверженцем Symfony2! Без лишней суеты, давайте узнаем что контроллеры могут для вас сделать.

.. index::
   single: Форматы
   single: Контроллер; Форматы
   single: Маршрутизация; Форматы
   single: Вид; Форматы

Форматы
-------

В наши дни, web приложение должно иметь возможность доставлять пользователю больше чем просто HTML страницы. Начиная от XML для RSS потоков или Web Сервисов, до JSON для Ajax запросов, существует множество форматов на выбор. Symfony имеет встроенную поддержку этих форматов. Отредактируйте ``routing.yml`` и добавьте ``_format`` со значением ``xml``:

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
        $collection->addRoute('hello', new Route('/hello/:name', array(
            '_controller' => 'HelloBundle:Hello:index',
            '_format'     => 'xml',
        )));

Затем, добавьте шаблон ``index.xml.php`` рядом с ``index.php``:

.. code-block:: xml+php

    # src/Application/HelloBundle/Resources/views/Hello/index.xml.php
    <hello>
        <name><?php echo $name ?></name>
    </hello>

Это все что нужно сделать. Не требуется даже менять контроллер. Для стандартных форматов, Symfony автоматически будет выбирать лучший ``Content-Type`` заголовок для ответа. Если вам необходима поддержка различных форматов в одном действии, используйте в выражении pattern модификатор ``:_format``:

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
        $collection->addRoute('hello', new Route('/hello/:name.:_format', array(
            '_controller' => 'HelloBundle:Hello:index',
            '_format'     => 'html',
        ), array(
            '_format' => '(html|xml|json)',
        )));

Контроллер теперь будет вызван для URL-адресов вида ``/hello/Fabien.xml`` или
``/hello/Fabien.json``. В качестве значения по умолчанию для ``_format`` используется ``html``, ``/hello/Fabien`` и ``/hello/Fabien.html`` оба используют ``html`` формат.

Ключ ``requirements`` определяет регулярное выражение, под которое должна подпадать переменная подстановки. В этом примере, если вы захотите вызвать ``/hello/Fabien.js``, вы получите ошибку HTTP 404, так как он не соответствует требованиям для ``_format``.

.. index::
   single: Ответ

Объект Ответа
-------------------

Теперь, давайте вернемся к контроллеру ``Hello``::

    // src/Application/HelloBundle/Controller/HelloController.php

    public function indexAction($name)
    {
        return $this->render('HelloBundle:Hello:index', array('name' => $name));
    }

Метод ``render()`` осущевляет отображение шаблона и возвращает объект ``Response``. Ответ может быть донастроен перед отправкой браузеру, например, для изменения значения ``Content-Type``::

    public function indexAction($name)
    {
        $response = $this->render('HelloBundle:Hello:index', array('name' => $name));
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
   single: Исключения

Управление Ошибками
-------------------

Когда что-либо не найдено, вам следует хорошо поиграться с HTTP протоколом и вернуть ответ 404. Это легко сделать путем вызова встроенного HTTP исключения::

    use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

    public function indexAction()
    {
        $product = // retrieve the object from database
        if (!$product) {
            throw new NotFoundHttpException('The product does not exist.');
        }

        return $this->render(...);
    }

``NotFoundHttpException`` вернет браузеру ответ HTTP 404. Похожим образом, ``ForbiddenHttpException`` вернет ошибку 403 и ``UnauthorizedHttpException`` вернет ответ 401. Для любых других кодов ошибок HTTP, вы можете использовать базовый класс ``HttpException`` и передавать код HTTP ошибки в качестве кода исключения::

    throw new HttpException('Unauthorized access.', 401);

.. index::
   single: Controller; Перемещение
   single: Controller; Перенаправление

Перемещения и Перенаправления
-----------------------------

Если вы хотите переместить пользователя на другую страницу, используйте метод ``redirect()``::

    $this->redirect($this->generateUrl('hello', array('name' => 'Lucas')));

Метод ``generateUrl()`` здесь - такой же метод как и ``generate()``, который мы использовали в хелпере маршрутизации ранее. Он принимает имя пути и массив параметров в качестве аргументов и возвращает соответствующий дружественный URL.

Вы также легко можете перенаправить одно действие на другое при помощи метода ``forward()``. Как и для хелпера ``$view['actions']``, он выполняет внутренний подзапрос, но он возвращает объект ``Response`` для возможности дальнейшей модификации при необходимости::

    $response = $this->forward('HelloBundle:Hello:fancy', array('name' => $name, 'color' => 'green'));

    // do something with the response or return it directly

.. index::
   single: Запрос

Объект Запроса
------------------

Кроме значений переменных подстановки в маршрутах, у контроллера также есть доступ к объекту запроса ``Request``::

    $request = $this['request'];

    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(array('en', 'fr'));

    $request->query->get('page'); // get a $_GET parameter

    $request->request->get('page'); // get a $_POST parameter

В шаблоне вы также можете получить доступ к объекту запроса через хелпер ``request``:

.. code-block:: html+php

    <?php echo $view['request']->getParameter('page') ?>

Сессия
-----------

Даже если у HTTP протокола нет поддержки состояний, Symfony снабжен изящным объектом сессии,
которая представляет собой клиента (будь то реальный человек, использующий браузер, бот, или web сервис). Между двумя запросами, Symfony сохраняет атрибуты в cookie путем встроенного в PHP механизма сессий.

Сохранение и извлечение информации из сессии может быть легко произведено из любого контроллера::

    // store an attribute for reuse during a later user request
    $this['request']->getSession()->set('foo', 'bar');

    // in another controller for another request
    $foo = $this['request']->getSession()->get('foo');

    // get/set the user culture
    $this['request']->getSession()->setLocale('fr');

Вы также можете хранить небольшие сообщения, которые будут доступны только в ближайших запросах::

    // store a message for the very next request (in a controller)
    $this['session']->setFlash('notice', 'Congratulations, your action succeeded!');

    // display the message back in the next request (in a template)
    <?php echo $view['session']->getFlash('notice') ?>

Заключительное Слово
--------------------

Вот и все что хотелось рассказать, и я даже не уверен, что мы использовали все отведенные 10 минут. В предыдущей части мы видели, как расширить систему шаблонов при помощи хелперов. Но в Symfony2 все может быть расширено или заменено при помощи бандлов. Это и есть тема следующей части данного руководства.
