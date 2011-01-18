Вид
========

Прочитав первую часть, вы решили что Symfony2 заслуживает ещё 10 минут. Хорошо.
Во второй части вы узнаете больше о движке шаблонов `Twig`_ в Symfony2. Twig
это гибкий, быстрый и безопасный шаблонизатор для PHP. Он делает шаблоны
удобочитаемыми и выразительными, а также более дружественными для web дизайнеров.

.. note::

    Вместо Twig, можете использовать для шаблонов :doc:`PHP </guides/templating/PHP>`.
    Оба шаблонных движка поддерживаются Symfony2 и имееют одинаковую степень
    поддержки.

.. index::
   single: Twig
   single: View; Twig

Twig, краткий обзор
----------------------

.. tip::

    Если хотите изучить Twig, мы настоятельно рекомендуем прочесть официальную
    `documentation`_. Этот раздел лишь кратко описывает основные концепты
    чтобы вы смогли начать работу.

Шаблон Twig это простой текстовый файл, который может генерировать любой формат,
основанный на тексте (HTML, XML, CSV, LaTeX, ...). Twig устанавливает два вида
разделителей:

* ``{{ ... }}``: Печатает переменную или результат выражения в шаблон;

* ``{% ... %}``: Тег, управляющий логикой шаблона; например, используется для
  выполнения операторов for-loops.

Ниже приведён минимальный шаблон, иллюстрирующий несколько основных правил:

.. code-block:: jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>My Webpage</title>
        </head>
        <body>
            <h1>{{ page_title }}</h1>

            <ul id="navigation">
                {% for item in navigation %}
                    <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
                {% endfor %}
            </ul>
        </body>
    </html>

Переменные, переданные в шаблон, могут быть строками, массивами или даже
объектами. Twig абстрагирует разницу между ними и даёт вам доступ к "атрибутам"
переменной, обозначенным через точку (``.``):

.. code-block:: jinja

    {# array('name' => 'Fabien') #}
    {{ name }}

    {# array('user' => array('name' => 'Fabien')) #}
    {{ user.name }}

    {# force array lookup #}
    {{ user['name'] }}

    {# array('user' => new User('Fabien')) #}
    {{ user.name }}
    {{ user.getName }}

    {# force method name lookup #}
    {{ user.name() }}
    {{ user.getName() }}

    {# pass arguments to a method #}
    {{ user.date('Y-m-d') }}

.. note::

    Важно знать что фигурные скобки это не часть переменной, а оператор печати.
    Если вы используете переменные внутри тегов, не ставьте скобки вокруг них.

Декорирование шаблонов
--------------------

Часто шаблоны в проекте разделяют общие элементы, такие как всем известные
header и footer. В Symfony2, мы смотрим на эту проблему иначе: один шаблон
может быть декорирован другим. Это похоже на классы в PHP: наследование шаблона
позволяет создать его базовый "макет", содержащий общие элементы вашего сайта и
устанавливающий "блоки", которые могут быть переопределены дочерними шаблонами.

Шаблон ``index.twig.html`` наследуется от ``layout.twig.html``, спасибо тегу ``extends``:

.. code-block:: jinja

    {# src/Application/HelloBundle/Resources/views/Hello/index.twig.html #}
    {% extends "HelloBundle::layout.twig.html" %}

    {% block content %}
        Hello {{ name }}!
    {% endblock %}

Обозначение ``HelloBundle::layout.twig.html`` выглядит знакомо, не так ли?
Обозначается так же как ссылка на шаблон. Эта часть ``::`` всего лишь обозначает
что контроллер не указан, т. о. соотвествующий файл хранится прямо в ``views/``.

Рассмотрим файл ``layout.twig.html``:

.. code-block:: jinja

    {% extends "::layout.twig.html" %}

    {% block body %}
        <h1>Hello Application</h1>

        {% block content %}{% endblock %}
    {% endblock %}

Тег ``{% block %}`` устанавливает два блока (``body`` и ``content``), которые
дочерние шаблоны смогут заполнить. Всё что делает этот тег, это сообщает движку
шаблонов, что дочерний шаблон может переопределить эти участки. Шаблон ``index.twig.html``
переопределяет блок ``content``, который указан в базовом макете, как если бы наш
макет сам по себе был декорирован оным.

Twig поддерживает много уровней декорирования: макет может быть декорирован
другим. Когда бандл в имени шаблона не указан (``::layout.twig.html``), то виды
ищутся в папке ``app/views/``.
Эта папка хранит глобальные виды для всего проекта:

.. code-block:: jinja

    {# app/views/layout.twig.html #}
    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            <title>{% block title %}Hello Application{% endblock %}</title>
        </head>
        <body>
            {% block body '' %}
        </body>
    </html>

Специальные теги и фильтры
-------------------------

Одна из лучших особенностей Twig его расширяемость через новые теги и фильтры;
Symfony2 поставляется со множеством специльных тегов и фильтров, облегчающими
работу web дизайнера.

Включения других шаблонов
~~~~~~~~~~~~~~~~~~~~~~~~~

Лучший способ распределить фрагмент кода между несколькими различными шаблонами
это определить шаблон, подключаемый в другие.

Создайте шаблон ``hello.twig.html``:

.. code-block:: jinja

    {# src/Application/HelloBundle/Resources/views/Hello/hello.twig.html #}
    Hello {{ name }}

Измените шаблон ``index.twig.html`` таким образом, чтобы подключить его:

.. code-block:: jinja

    {# src/Application/HelloBundle/Resources/views/Hello/index.twig.html #}
    {% extends "HelloBundle::layout.twig.html" %}

    {# override the body block from index.twig.html #}
    {% block body %}
        {% include "HelloBundle:Hello:hello.twig.html" %}
    {% endblock %}

Вложение других контроллеров
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Что если вы захотите вложить результат другого контроллера в шаблон? Это очень
удобно когда работаешь с Ajax или когда встроенному шаблону необходимы
переменные, которые не доступны в главном шаблоне.

Если вы создали действие ``fancy`` и хотите включить его в шаблон ``index``,
используйте тег ``render``:

.. code-block:: jinja

    {# src/Application/HelloBundle/Resources/views/Hello/index.twig.html #}
    {% render "HelloBundle:Hello:fancy" with { 'name': name, 'color': 'green' } %}

Имеем строку ``HelloBundle:Hello:fancy``, обращающуюся к действию ``fancy``
контроллера ``Hello`` и аргумент, используемый для имитирования запроса для
заданного пути::

    // src/Application/HelloBundle/Controller/HelloController.php

    class HelloController extends Controller
    {
        public function fancyAction($name, $color)
        {
            // create some object, based on the $color variable
            $object = ...;

            return $this->render('HelloBundle:Hello:fancy.twig.html', array('name' => $name, 'object' => $object));
        }

        // ...
    }

Создание ссылок между страницами
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Говоря о web приложениях, нельзя не упомянуть о ссылках. Вместо жёстких URL-ов
в шаблонах, функция ``path`` поможет сделать URL-ы, основанные на конфигурации
маршрутизатора. Таким образом URL-ы могут быть легко обновлены, если изменить
конфигурацию:

.. code-block:: jinja

    <a href="{{ path('hello', { 'name': 'Thomas' }) }}">Greet Thomas!</a>

Функция ``path`` использует имя маршрута и массив параметров как аргументы.
Имя маршрута это основа, в соотвествии с которой выбираются маршруты, а
параметры это значения заполнителей, объявленных в паттерне маршрута:

.. code-block:: yaml

    # src/Application/HelloBundle/Resources/config/routing.yml
    hello: # The route name
        pattern:  /hello/{name}
        defaults: { _controller: HelloBundle:Hello:index }

.. tip::

    Можно создавать абсолютные URL-ы с помощью функции ``url``:
    ``{{ url('hello', { 'name': 'Thomas' }) }}``.

Применение активов: изображений, JavaScript-ов и таблиц стилей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Как выглядел бы интернет без изображений, JavaScript-ов и таблиц стилей?
Symfony2 предлагает функцию ``asset`` для работы с ними:

.. code-block:: jinja

    <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    <img src="{{ asset('images/logo.png') }}" />

Основная цель функции ``asset`` сделать приложение более переносимым. Благодаря
ей, можно переместить корневую папку приложения куда угодно внутри вашей
корневой web директории без изменения шаблона.

Экранирование вывода
---------------

Изначально Twig настроен экранировать весь вывод. Прочтите Twig
`documentation`_ чтобы узнать больше об экранировании и расширении Escaper.

Заключительное слово
--------------

Twig простой и мощный. Благодаря макетам, блокам, шаблонам и внедрениям действий,
становится действительно просто организовать ваши шаблоны логически и сделать их
расширяемыми.

Проработав с Symfony2 около 20 минут, вы уже можете делать удивительные вещи.
В этом сила Symfony2. Изучать основы легко, вскоре вы узнаете что эта простота
скрыта в очень гибкой архитектуре.

Я немного поспешил. Во-первых, вы должны узнать больше о контроллере, именно он
станет темой следующей части учебника. Готовы к следующим 10 минутам с Symfony2?

.. _Twig:          http://www.twig-project.org/
.. _documentation: http://www.twig-project.org/documentation
