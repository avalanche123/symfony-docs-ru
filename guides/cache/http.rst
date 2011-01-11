.. index::
   single: Кэш

HTTP Кэш
========

Лучшим способом улучшить производительность приложения будет, вероятно, 
кэшировать его вывод и обойти его полностью. Конечно же, это невозможно для 
сильно динамичных web сайтов, или все-таки можно? Этот раздел покажет вам, 
как работает система кэширования Symfony2 и почему мы думаем что это 
наилучший возможный подход.

Система кэширования Symfony2 полагается на простоту и мощь HTTP кэша, как 
это определено в спецификации HTTP. В основном, если вам уже знакомы модели 
кэширования HTTP валидации и устаревания, вы готовы к использованию 
большей части системы кэширования Symfony2.

.. index::
   single: Кэш; Типы
   single: Кэш; Прокси
   single: Кэш; Обратный Прокси
   single: Кэш; Шлюз

Разновидности Кэша
------------------

Заголовки HTTP кэша обрабатываются и интерпретируются тремя различными видами 
кэша:

* *Кэширование браузером*: Каждый браузер включает в себя собственный локальный кэш, 
  который наиболее полезен, когда вы нажимаете кнопку "back" или когда изображения 
  используются на web сайте неоднократно;

* *Proxy кэширование*: Прокси это *распределенный* кэш, так как много людей могут 
  использовать один и тот же кэш. Он всегда устанавливается большими корпорациями 
  и ISP для уменьшения латентности и сетевого трафика.

* *Gateway кэширование*: Как и прокси, тоже является распределенным кэшем, но на 
  серверной стороне. Установленный сетевыми администраторами, он делает web сайты 
  более масштабируемыми, надежными и более продуктивными (CDN-ы такие как Akamaï 
  являются gateway кэшами).

.. note::

    Gateway кэш иногда упоминается как обратный прокси кэш,
    кэш-заместитель, или даже HTTP акселератор.

Протокол HTTP 1.1 по умолчанию позволяет кэшировать все, за исключением  
явно указанного в заголовке ``Cache-Control``. На практике, большинство кэшей 
ничего не делают если в запросе установлены cookies, заголовок авторизации, или 
передача ведется по защищенному методу и когда у ответа есть статус кода редиректа.

Symfony2 автоматически устанавливает рациональный и умеренный заголовок 
``Cache-Control`` когда он не указан разработчиком, следуя таким правилам:

* Если не указан заголовок кэша (``Cache-Control``, ``ETag``,
  ``Last-Modified``, и ``Expires``), ``Cache-Control`` устанавливается как 
  ``no-cache``;

* Если ``Cache-Control`` пустой, его значение устанавливается в ``private, 
  max-age=0, must-revalidate``;

* Но когда установлена хотя бы одна директива ``Cache-Control``, и явно не 
  добавлены директивы 'public' или ``private``, Symfony2 добавляет директиву 
  ``private`` автоматически (кроме случая, когда установлено ``s-maxage``).

.. tip::

    Большинство gateway кэшей могут удалять cookies перед перенаправлением 
    запроса к серверному приложению, и добавлять их обратно при отправке 
    ответа браузеру (это полезно для cookies, которые не изменяют 
    представление ресурса, такие как отслеживающие cookies).

Модификация Заголовков Ответа
-----------------------------

Перед тем как мы начнем наш тур по различным HTTP заголовкам, которые вы 
можете использовать для включения кэширования в вашем приложении, первым 
делом вам нужно знать как изменять их в Symfony2 приложении.

Класс :class:`Symfony\\Component\\HttpFoundation\\Response` предоставляет красивый 
и простой API для упрощения манипуляций с HTTP заголовками::

    // передавайте массив заголовков третьим аргументом в конструктор Response
    $response = new Response($content, $status, $headers);

    // устанавливайте значение заголовка
    $response->headers->set('Content-Type', 'text/plain');

    // добавляйте значения заголовка к существующим значениям
    $response->headers->set('Vary', 'Accept', false);

    // устанавливайте заголовок с многими значениями
    $response->headers->set('Vary', array('Accept', 'Accept-Encoding'));

    // удаляйте заголовок
    $response->headers->delete('Content-Type');

Кроме этих встроенных путей установки заголовков, класс Response также предоставляет 
много специализированных методов, упрощающих манипуляции с заголовками HTTP кэша.
По пути вы узнаете о них больше.

.. tip::

    Имена HTTP заголовков регистро-независимы. Так как Symfony2 внутренне 
    конвертирует их в нормализированную форму, регистр ввода значения не имеет
    (``Content-Type`` рассматривается идентично с ``content-type``). Вы также 
    можете использовать нижние подчеркивания (``_``) вместо дефисов (``-``), если захотите.

Если вы используете укороченный метод класса Controller ``render`` для формирования шаблона и 
создания объекта Response, вы также можете легко манипулировать заголовками Response::

    // Сперва создайте объект Response и установите заголовки...
    $response = new Response();
    $response->headers->set('Content-Type', 'text/plain');

    // ...и потом установите их как третий аргумент в метод render
    return $this->render($name, $vars, $response);

    // Или, вызовите render сначала...
    $response = $this->render($name, $vars);

    // ...и манипулируйте заголовками Response потом
    $response->headers->set('Content-Type', 'text/plain');

    return $response;

.. index::
   single: Кэш; HTTP

Понимание HTTP Кэша
-------------------

HTTP спецификация (aka `RFC 2616`_) определяет две модели кэширования:

* *Истечение*: Вы указываете как долго ответ считается "свежим" путем установки
  ``Cache-Control`` и/или ``Expires`` заголовков. При кэшировании помните, 
  что истечение не будет делать одинаковый запрос пока кэшируемая версия 
  не достигнет своего времени истечения срока и станет "старой".

* *Валидация*: Когда некоторые страницы действительно динамичны (в смысле, что их 
  содержимое часто изменяется), модель валидации использует уникальный идентификатор
  (заголовок ``Etag``) и/или метку времени (заголовок ``Last-Modified``) для проверки, 
  изменилась ли страница с последнего раза.

Целью обоих моделей является никогда не генерировать один и тот же Response дважды.

.. tip::

    Принимаются усилия (`HTTP Bis`_) переписать RFC 2616. Он не описывает
    новую версию HTTP, но преимущественно освещает первоначальную спецификацию HTTP.
    Организация также намного лучше, так как спецификация разделена на несколько 
    частей; все что касается HTTP кэширования может быть найдено в двух разделенных
    частях (`P4 - Conditional Requests`_ and `P6 - Caching: Browser and intermediary caches`_).

.. tip::

    Заголовки HTTP кэша работают только с "безопасными" HTTP методами (такими как 
    GET и HEAD). Быть безопасным означает, что вы никогда не должны изменять состояние 
    приложения на сервере когда отрабатываются такие запросы (но вы, конечно же, можете 
    логировать информацию, кэшировать данные, ...)

.. index::
   single: Кэш; HTTP Истечение

Истечение
~~~~~~~~~

По возможности, нужно использовать модель кэширования с истечением, если ваше 
приложение будет вызвано при первом запросе и оно не будет вызываться снова пока 
не устареет (это экономит CPU сервера и улучшает масштабируемость).

.. index::
   single: Кэш; заголовок Истечения
   single: HTTP заголовки; Истечение

Истечение с заголовком ``Expires``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В соответствии с RFC 2616, "the ``Expires`` header field gives the date/time after
which the response is considered stale." Заголовок ``Expires`` может быть установлен 
при помощи ``setExpires()`` метод класса Response. Он принимает экземпляр ``DateTime`` 
в качестве аргумента::

    $date = new DateTime();
    $date->modify('+600 seconds');

    $response->setExpires($date);

.. note::

    Метод ``setExpires()`` автоматически конвертирует дату в формат GMT, 
    чего требует спецификация (дата должна быть в формате RFC1123).

Заголовок ``Expires`` обладает двумя недостатками. Во-первых, часы Web сервера 
и кэша (aka браузера) должны быть синхронизированы. Then, the
specification states that "HTTP/1.1 servers should not send ``Expires`` dates
more than one year in the future."

.. index::
   single: Кэш; Cache-Control заголовок
   single: HTTP заголовки; Cache-Control

Истечение с заголовком ``Cache-Control``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Так как у заголовка ``Expires`` есть ограничения, чаще всего, вам следует 
использовать вместо него заголовок ``Cache-Control``. Так как ``Cache-Control`` 
это заголовок общего назначения, используемый для установки различных директив, 
Symfony2 предоставляет методы, которые абстрагируют манипуляции ими. Для истечения, 
есть две директивы, ``max-age`` и ``s-maxage``. Первая используется всеми видами 
кэшей, тогда как вторая берется во внимание только shared кэшами::

    // Устанавливает количество секунд по истечению которых
    // ответ уже не будет считаться свежим
    $response->setMaxAge(600);

    // Тоже что и сверху, но только для shared кэшей
    $response->setSharedMaxAge(600);

.. index::
   single: Кэш; Валидация

Валидация
~~~~~~~~~

Когда ресурс должен быть обновлен как только были изменены данные, модель 
истечения терпит крах. Модель валидации решает эту задачу. В этой модели, 
вы преимущественно сохраняете пропускную способность, так как представление не 
отсылается дважды одному и тому же клиенту (вместо этого отсылается ответ 304). 
Но если вы внимательно проектируете дизайн вашего приложения, у вас должна быть 
возможность получить минимальный объем данных, необходимый для отправки ответа 
304, и сохранить также CPU; и если необходимо, выполнить более трудоемкие 
задачи (смотрите ниже практический пример).

.. index::
   single: Кэш; Etag заголовок
   single: HTTP заголовки; Etag

Валидация с использованием заголовка ``ETag``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В соответствии с RFC, "Поле ответа-заголовка ``ETag`` обеспечивает текущее 
значение entity-tag для одного представления рассматриваемого ресурса. 
Entity-tag предполагается использовать в качестве локального для ресурса 
идентификатора для дифференциации между представлениями одного и того же 
ресурса, изменяющегося во времени или через согласование содержания.". 
"Entity-tag ДОЛЖЕН быть уникальным во всех версиях всех представлений, 
ассоциированных с конкретным ресурсом."

Возможным значением для "entity-tag" может быть, например, хэш содержимого 
ответа::

    $response->setETag(md5($response->getContent()));

Этот алгоритм достаточно прост и очень универсален, но вам нужно создать 
Response полностью, перед тем как вы сможете рассчитать ETag, что не совсем 
оптимально. Эта стратегия часто используется как алгоритм по умолчанию во 
многих фреймворках, но вам следует использовать какой-нибудь алгоритм, 
который лучше учитывает путь создания ресурсов (смотрите секцию ниже про 
оптимизацию валидации).

.. tip::

    Symfony2 также поддерживает слабые ETags путем передачи ``true`` в качестве 
    второго аргумента в метод 
    :method:`Symfony\\Component\\HttpFoundation\\Response::setETag`.

.. index::
   single: Кэш; Last-Modified заголовок
   single: HTTP заголовки; Last-Modified

Валидация с использованием заголовка ``Last-Modified``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В соответствии с RFC, "Поле заголовка ``Last-Modified`` отображает 
дату и время, при которой, по мнению главного сервера, отображение было 
последний раз изменено."

Например, в качестве даты последнего изменения для всех объектов, требующих 
расчета времени отображения, значение заголовка ``Last-Modified``::

    $articleDate = new \DateTime($article->getUpdatedAt());
    $authorDate = new \DateTime($author->getUpdatedAt());

    $date = $authorDate > $articleDate ? $authorDate : $articleDate;

    $response->setLastModified($date);

.. index::
   single: Cache; Conditional Get
   single: HTTP; 304

Оптимизация вашего Кода при помощи Валидации
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Главной целью любой стратегии кэширования является облегчение загрузки 
приложения; следуя другим путем, минимум что можно сделать в вашем 
приложении - вернуть ответ 304, что еще лучще. Метод Symfony2 
``Response::isNotModified()`` делает именно это через использование 
простого и производительного паттерна::

    // Получаем минимум информации для вычисления
    // ETag или значение Last-Modified
    // (базируясь на Request, данные получены, например, из
    // базы банных или хранилища ключ-значение)
    $article = Article::get(...);

    // создадим объект Response с заголовком ETag и/или a Last-Modified
    $response = new Response();
    $response->setETag($article->computeETag());
    $response->setLastModified($article->getPublishedAt());

    // Проверяем что Response не изменился для заданного Request
    if ($response->isNotModified($request)) {
        // сразу же отсылаем 304 Response
        $response->send();
    } else {
        // делаем здесь что-нибудь трудоемкое
        // такое как работа с БД
        // или рендеринг шаблона
    }

Когда Response не был изменен, ``isNotModified()`` автоматически 
устанавливает статус кода ответа ``304``, удалите содержимое, и удалите 
некоторые заголовки которые не должны присутствовать в ответах ``304`` 
(смотрите :method:`Symfony\\Component\\HttpFoundation\\Response::setNotModified`).

.. index::
   single: Кэш; Vary
   single: HTTP заголовки; Vary

Варьирование Response
~~~~~~~~~~~~~~~~~~~~~

Иногда, представление ресурса зависит не только от его URI, но также и от
значений некоторых заголовков. Например, если вы сжимаете страницы, когда это 
поддерживает клиент, каждый выдаваемый URI имеет два представления: одно когда 
клиент поддерживает компрессию, и еще одно когда нет. Для таких случаев, вы 
должны использовать заголовок ``Vary``, чтобы помочь кэшу определить, когда 
хранимый ответ может быть использован в соответствии в полученным запросом::

    $response->setVary('Accept-Encoding');

    $response->setVary(array('Accept-Encoding', 'Accept'));

Метод ``setVary()`` получает имя заголовка или массив имен заголовков от 
которых изменяется ответ.

Истечение и Валидация
~~~~~~~~~~~~~~~~~~~~~

Конечно вы можете использовать одновременно и валидацию и истечение в 
одинаковых Response. Так как истечение выигрывает у валидации, вы можете 
легко взять лучшее из обоих миров. Это дает вам множество путей настройки 
и корректировки вашей стратегии кэширования.

.. index::
    pair: Кэш; Конфигурация

Больше Методов Response
~~~~~~~~~~~~~~~~~~~~~~~

Класс Response предоставляет еще множество методов связанных с кэшем. Вот самые 
полезные из них::

    // Пометим Response как приватный
    $response->setPrivate();

    // Пометим Response как публичный
    $response->setPublic();

    // Пометим Response как устаревший
    $response->expire();

Последнее но совсем не последнее по значимости, HTTP заголовки, 
наиболее связанные с кэшем, могут быть установлены вызовом одного 
метода ``setCache()``::

    // Установим настройки кэша в один вызов
    $response->setCache(array(
        'etag'          => $etag,
        'last_modified' => $date,
        'max_age'       => 10,
        'public'        => true,
    ));

Конфигурирование Кэша
---------------------

Как вы уже догадались, наилучшей конфигурацией для ускорения вашего приложения 
будет добавление gateway cache на входе вашего приложения. И так как Symfony2 
использует только стандартные HTTP заголовки для управления его кэшем, здесь 
нет необходимости в внутреннем слое кэша. Вместо этого, вы можете использовать 
любой обратимый прокси какой захотите, такой как Apache mod_cache, Squid, или 
Varnish. Если вы не хотите устанавливать дополнительное программное обеспечение, 
вы можете использовать обратимый прокси встроенный в Symfony2, который написан на 
PHP и делает ту же работу что и любой другой обратимый прокси.

Public vs Private Responses
~~~~~~~~~~~~~~~~~~~~~~~~~~~

As explained at the beginning of this document, Symfony2 is very conservative
and makes all Responses private by default (the exact rules are described
there).

If you want to use a shared cache, you must remember to explicitly add the
``public`` directive to ``Cache-Control``::

    // The Response is private by default
    $response->setEtag($etag);
    $response->setLastModified($date);
    $response->setMaxAge(10);

    // Change the Response to be public
    $response->setPublic();

    // Set cache settings in one call
    $response->setCache(array(
        'etag'          => $etag,
        'last_modified' => $date,
        'max_age'       => 10,
        'public'        => true,
    ));

Symfony2 Reverse Proxy
~~~~~~~~~~~~~~~~~~~~~~

Symfony2 comes with a reverse proxy written in PHP. Enable it and it will
start to cache your application resources right away. Installing it is as easy
as it can get. Each new Symfony2 application comes with a pre-configured
caching Kernel (``AppCache``) that wraps the default one (``AppKernel``).
Modify the code of a front controller so that it reads as follows to enable
caching::

    // web/app.php

    require_once __DIR__.'/../app/AppCache.php';

    use Symfony\Component\HttpFoundation\Request;

    // wrap the default AppKernel with the AppCache one
    $kernel = new AppCache(new AppKernel('prod', false));
    $kernel->handle(new Request())->send();

.. tip::

    The cache kernel has a special ``getLog()`` method that returns a string
    representation of what happened in the cache layer. In the development
    environment, use it to debug and validate your cache strategy::

        error_log($kernel->getLog());

The ``AppCache`` object has a sensible default configuration, but it can be
finely tuned via a set of options you can set by overriding the
``getOptions()`` method::

    // app/AppCache.php
    class AppCache extends Cache
    {
        protected function getOptions()
        {
            return array(
                'debug'                  => false,
                'default_ttl'            => 0,
                'private_headers'        => array('Authorization', 'Cookie'),
                'allow_reload'           => false,
                'allow_revalidate'       => false,
                'stale_while_revalidate' => 2,
                'stale_if_error'         => 60,
            );
        }
    }

Here is a list of the main options:

* ``default_ttl``: The number of seconds that a cache entry should be
  considered fresh when no explicit freshness information is provided in a
  response. Explicit ``Cache-Control`` or ``Expires`` headers override this
  value (default: ``0``);

* ``private_headers``: Set of request headers that trigger "private"
  ``Cache-Control`` behavior on responses that don't explicitly state whether
  the response is ``public`` or ``private`` via a ``Cache-Control`` directive.
  (default: ``Authorization`` and ``Cookie``);

* ``allow_reload``: Specifies whether the client can force a cache reload by
  including a ``Cache-Control`` "no-cache" directive in the request. Set it to
  ``true`` for compliance with RFC 2616 (default: ``false``);

* ``allow_revalidate``: Specifies whether the client can force a cache
  revalidate by including a ``Cache-Control`` "max-age=0" directive in the
  request. Set it to ``true`` for compliance with RFC 2616 (default: false);

* ``stale_while_revalidate``: Specifies the default number of seconds (the
  granularity is the second as the Response TTL precision is a second) during
  which the cache can immediately return a stale response while it revalidates
  it in the background (default: ``2``); this setting is overridden by the
  ``stale-while-revalidate`` HTTP ``Cache-Control`` extension (see RFC 5861);

* ``stale_if_error``: Specifies the default number of seconds (the granularity
  is the second) during which the cache can serve a stale response when an
  error is encountered (default: ``60``). This setting is overridden by the
  ``stale-if-error`` HTTP ``Cache-Control`` extension (see RFC 5861).

If ``debug`` is ``true``, Symfony2 automatically adds a ``X-Symfony-Cache``
header to the Response containing useful information about cache hits and
misses.

The Symfony2 reverse proxy is a great tool to use when developing your website
on your local network or when you deploy your website on a shared host where
you cannot install anything beyond PHP code. But being written in PHP, it
cannot be as fast as a proxy written in C. That's why we highly recommend you
to use Squid or Varnish on your production servers if possible. The good news
is that the switch from one proxy server to another is easy and transparent as
no code modification is needed in your application; start easy with the
Symfony2 reverse proxy and upgrade later to Varnish when your traffic raises.

.. note::

    The performance of the Symfony2 reverse proxy is independent of the
    complexity of the application; that's because the application kernel is
    only booted when the request needs to be forwarded to it.

Apache mod_cache
~~~~~~~~~~~~~~~~

If you use Apache, it can act as a simple gateway cache when the mod_cache
extension is enabled.

Squid
~~~~~

Squid is a "regular" proxy server that can also be used as a reverse proxy
server. If you already use Squid in your architecture, you can probably
leverage its power for your Symfony2 applications. If not, we highly recommend
you to use Varnish as it has many advantages over Squid and because it
supports features needed for advanced Symfony2 caching strategies (like ESI
support).

Varnish
~~~~~~~

Varnish is our preferred choice for three main reasons:

* It has been designed as a reverse proxy from day one so its configuration is
  really straightforward;

* Its modern architecture means that it is insanely fast;

* It supports ESI, a technology used by Symfony2 to allow different elements
  of a page to have their own caching strategy (read the next section for more
  information).

.. index::
  single: Cache; ESI
  single: ESI

Using Edge Side Includes
------------------------

Gateway caches are a great way to make your website performs better. But they
have one limitation: they can only cache whole pages. So, if you cannot cache
whole pages or if a page has "more" dynamic parts, you are out of luck.
Fortunately, Symfony2 provides a solution for these cases, based on a
technology called `ESI`_, or Edge Side Includes. Akamaï wrote this
specification almost 10 years ago, and it allows specific parts of a page to
have a different caching strategy that the main page.

The ESI specification describes tags you can embed in your pages to
communicate with the gateway cache. Only one tag is implemented in Symfony2,
``include``, as this is the only useful one outside of Akamaï context:

.. code-block:: html

    <html>
        <body>
            Some content

            <!-- Embed the content of another page here -->
            <esi:include src="http://..." />

            More content
        </body>
    </html>

When a request comes in, the gateway cache gets the page from its cache or
calls the backend application. If the response contains one or more ESI tags,
the proxy behaves like for the main request. It gets the included page content
from its cache or calls the backend application again. Then it merges all the
included content in the main page and sends it back to the client.

.. index::
    single: Helper; actions

As the embedded content comes from another page (or controller for that
matter), Symfony2 uses the standard ``render`` helper to configure ESI tags:

.. configuration-block::

    .. code-block:: php

        <?php echo $view['actions']->render('...:list', array(), array('standalone' => true)) ?>

    .. code-block:: jinja

        {% render '...:list' with [], ['standalone': true] %}

By setting ``standalone`` to ``true``, you tell Symfony2 that the action
should be rendered as an ESI tag. You might be wondering why you would want to
use a helper instead of just writing the ESI tag yourself. That's because
using a helper makes your application works even if there is no gateway cache
installed. Let's see how it works.

When standalone is ``false`` (the default), Symfony2 merges the included page
content within the main one before sending the response to the client. But
when standalone is ``true`` and if Symfony 2 detects that it talks to a
gateway cache that supports ESI, it generates an ESI include tag. But if there
is no gateway cache or if it does not support ESI, Symfony2 will just merge
the included page content within the main one as it would have done when
standalone is ``false``.

.. note::

    Symfony2 detects if a gateway cache supports ESI via another Akamaï
    specification that is supported out of the box by the Symfony2 reverse
    proxy (a working configuration for Varnish is also provided below).

For the ESI include tag to work properly, you must define the ``_internal``
route:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        _internal:
            resource: FrameworkBundle/Resources/config/routing/internal.xml
            prefix:   /_internal

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://www.symfony-project.org/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.symfony-project.org/schema/routing http://www.symfony-project.org/schema/routing/routing-1.0.xsd">

            <import resource="FrameworkBundle/Resources/config/routing/internal.xml" prefix="/_internal" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection->addCollection($loader->import('FrameworkBundle/Resources/config/routing/internal.xml', '/_internal'));

        return $collection;

.. tip::

    You might want to protect this route by either choosing a non easily
    guessable prefix, or by protecting them using the Symfony2 firewall
    feature (by allowing access to your reverse proxies IP range).

One great advantage of this caching strategy is that you can make your
application as dynamic as needed and at the same time, hit the application as
less as possible.

.. note::

    Once you start using ESI, remember to always use the ``s-maxage``
    directive instead of ``max-age``. As the browser only ever receives the
    aggregated resource, it is not aware of the sub-components, and so it will
    obey the ``max-age`` directive and cache the entire page. And you don't
    want that.

.. tip::

    The ``render`` helper supports two other useful options, ``alt`` and
    ``ignore_errors``. They are automatically converted to ``alt`` and
    ``onerror`` attributes when an ESI include tag is generated.

.. index::
    single: Cache; Varnish

Varnish Configuration
~~~~~~~~~~~~~~~~~~~~~

As seen previously, Symfony2 is smart enough to detect whether it talks to a
reverse proxy that understands ESI or not. It works out of the box when you
use the Symfony2 reverse proxy, but you need a special configuration to make
it work with Varnish. Thankfully, Symfony2 relies on yet another standard
written by Akamaï (`Edge Architecture`_), so the configuration tips in this
chapter can be useful even if you don't use Symfony2.

.. note::

    Varnish only supports the ``src`` attribute for ESI tags (``onerror`` and
    ``alt`` attributes are ignored).

First, configure Varnish so that it advertises its ESI support by adding a
``Surrogate-Capability`` header to requests forwarded to the backend
application:

.. code-block:: text

    sub vcl_recv {
        set req.http.Surrogate-Capability = "abc=ESI/1.0";
    }

Then, optimize Varnish so that it only parses the Response contents when there
is at least one ESI tag by checking the ``Surrogate-Control`` header that
Symfony2 adds automatically:

.. code-block:: text

    sub vcl_fetch {
        if (beresp.http.Surrogate-Control ~ "ESI/1.0") {
            unset beresp.http.Surrogate-Control;
            esi;
        }
    }

.. caution::

    Don't use compression with ESI as Varnish won't be able to parse the
    response content. If you want to use compression, put a web server in
    front of Varnish to do the job.

.. index::
    single: Cache; Invalidation

Invalidation
------------

"There are only two hard things in Computer Science: cache invalidation and
naming things." --Phil Karlton

You never need to invalidate cached data because invalidation is already taken
into account natively in the HTTP cache models. If you use validation, you
never need to invalidate anything by definition; and if you use expiration and
need to invalidate a resource, it means that you set the expires date too far
away in the future.

.. note::

    It's also because there is no invalidation mechanism that you can use any
    reverse proxy without changing anything in your application code.

Actually, all reverse proxies provide ways to purge cached data, but you
should avoid them as much as possible. The most standard way is to purge the
cache for a given URL by requesting it with the special ``PURGE`` HTTP method.

.. index::
    single: Cache; Invalidation with Varnish

Here is how you can configure the Symfony2 reverse proxy to support the
``PURGE`` HTTP method::

    // app/AppCache.php
    class AppCache extends Cache
    {
        protected function invalidate(Request $request)
        {
            if ('PURGE' !== $request->getMethod()) {
                return parent::invalidate($request);
            }

            $response = new Response();
            if (!$this->store->purge($request->getUri())) {
                $response->setStatusCode(404, 'Not purged');
            } else {
                $response->setStatusCode(200, 'Purged');
            }

            return $response;
        }
    }

And the same can be done with Varnish too:

.. code-block:: text

    sub vcl_hit {
        if (req.request == "PURGE") {
            set obj.ttl = 0s;
            error 200 "Purged";
        }
    }

    sub vcl_miss {
        if (req.request == "PURGE") {
            error 404 "Not purged";
        }
    }

.. caution::

    You must protect the ``PURGE`` HTTP method somehow to avoid random people
    purging your cached data.

.. _`RFC 2616`: http://www.ietf.org/rfc/rfc2616.txt
.. _`HTTP Bis`: http://tools.ietf.org/wg/httpbis/
.. _`P4 - Conditional Requests`: http://tools.ietf.org/id/draft-ietf-httpbis-p4-conditional-12.txt
.. _`P6 - Caching: Browser and intermediary caches`: http://tools.ietf.org/id/draft-ietf-httpbis-p6-cache-12.txt
.. _`ESI`: http://www.w3.org/TR/esi-lang
.. _`Edge Architecture`: http://www.w3.org/TR/edge-arch
