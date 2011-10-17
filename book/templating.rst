.. index::
   single: Шаблонизатор

Создание и использование Шаблонов
=================================

Как вам известно, :doc:`контроллер </book/controller>` отвечает за
обработку запросов, получаемых приложением Symfony2. Фактически же,
контроллер делегирует большую часть тяжёлой работы другим частям
Фреймворка, чтобы код можно было тестировать и использовать повторно.
Когда контроллеру требуется сгенерировать HTML, CSS или любой другой
контент, он поручает эту работу шаблонизатору. В этой главе вы узнаете
как создавать мощные и функциональные шаблоны, которые могут быть
использованы для того, чтобы вернуть контент пользователю, создавать
тело email-сообщений и многое другое. Вы узнаете, как наследовать шаблоны
и как повторно использовать код шаблонов.

.. index::
   single: Шаблонизатор; Что такое шаблон?

Шаблоны
---------

Шаблон - это просто текстовый файл, который может генерировать любой текстовый
формат (HTML, XML, CSV, LaTeX и т.д.). Наиболее простой тип шаблона - это *PHP*
шаблон - текстовый файл, обрабатываемый PHP, который содержит как собственно текст,
так и PHP-код:

.. code-block:: html+php

    <!DOCTYPE html>
    <html>
        <head>
            <title>Welcome to Symfony!</title>
        </head>
        <body>
            <h1><?php echo $page_title ?></h1>

            <ul id="navigation">
                <?php foreach ($navigation as $item): ?>
                    <li>
                        <a href="<?php echo $item->getHref() ?>">
                            <?php echo $item->getCaption() ?>
                        </a>
                    </li>
                <?php endforeach; ?>
            </ul>
        </body>
    </html>

.. index:: Twig; Введение

В состав Symfony2 входит мощный язык шаблонов, называемый `Twig`_. Twig
позволяет создавать лаконичные и читабельные шаблоны, которые более удобны
для веб-дизайнеров и, во многом, более функциональны, нежели PHP шаблоны:

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>Welcome to Symfony!</title>
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

Twig определяет два типа специальных синтаксических конструкций:

* ``{{ ... }}``: "Напечатать что-либо": отображает переменную или результат некоторого
  выражения;

* ``{% ... %}``: "Выполнить что-либо": **таг**, который контролирует логику шаблона;
  он используется для выполнения выражений, таких как циклы ``for``.

.. note::

   Есть также третий тип синтаксической конструкции для создания комментариев:
   ``{# это комментарий #}``. Этот синтаксис может быть использован в качестве
   многострочного комментария как PHP-аналог ``/* comment */``.

Twig также содержит **фильтры**, которые модифицируют контент перед его
отображением. Следующий пример выведет переменную ``title`` в верхнем
регистре:

.. code-block:: jinja

    {{ title | upper }}

Twig по умолчанию содержит много тагов (`tags`_) и фильтров (`filters`_).
Кроме этого вы можете также `создавать свои собственные расширения`_ Twig,
если это потребуется.

.. tip::

    Зарегистрировать расширение Twig просто: нужно создать сервис и указать ему
    :ref:`таг<book-service-container-tags>` ``twig.extension``.

Как вы увидите далее, Twig также поддерживает функции, и вы сможете легко
добавлять новые функции. Например, следующий код использует стандартный
таг цикла ``for`` и функцию ``cycle`` для того, чтобы вывести десять тагов
``div`` с css-классами ``odd`` и ``even``:

.. code-block:: html+jinja

    {% for i in 0..10 %}
      <div class="{{ cycle(['odd', 'even'], i) }}">
        <!-- some HTML here -->
      </div>
    {% endfor %}

На протяжении всей главы, примеры шаблонов будут отображаться как в Twig-формате,
так и PHP.

.. sidebar:: Почему Twig?

    Twig шаблоны выглядят проще и не обрабатывают PHP код. Это сделано намеренно:
    система шаблонов Twig задумана для быстрого создания представления, а не
    для программной логики. Чем больше вы используете Twig, тем более вы будете
    ценить его и тем более будете получать пользы от такого разделения. И вас
    будут уважать все веб-дизайнеры в мире.

    Twig также умеет делать то, что не умеет PHP, например полноценное
    наследование шаблонов (Twig-шаблоны компилируются в PHP-классы, которые
    наследуются как любые другие классы), контроль пробельных символов,
    песочница (для контроля выполнения "подозрительного" кода в шаблонах),
    расширение пользовательскими фильтрами и функциями. Twig имеет много
    небольших особенностей, которые делают написание шаблонов проще и
    лаконичнее. Взгляните на следующий пример, который комбинирует цикл
    с логическим выражением ``if``:

    .. code-block:: html+jinja

        <ul>
            {% for user in users %}
                <li>{{ user.username }}</li>
            {% else %}
                <li>No users found</li>
            {% endfor %}
        </ul>

.. index::
   pair: Twig; Кэш

Кэширование Шаблонов в Twig
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Twig быстр. Каждый Twig-шаблон компилируется в обычный PHP-класс, который
и отображается во время выполнения. Компилированные классы расположены в
директории ``app/cache/{environment}/twig`` (здесь ``{environment}`` - это
название окружения, например ``dev`` или ``prod``) и в некоторых случаях
может быть полезен при отладке. Об окружениях подробнее написано в разделе
:ref:`environments-summary`.

Когда активен режим ``debug`` (как правило, в ``dev`` окружении), шаблон
Twig будет автоматически перекомпилироваться каждый раз, когда в нём
были произведены изменения. Это означает, что в процессе разработки вы
спокойно можете выполнять изменения в шаблонах и видеть эти изменения
без необходимости постоянно чистить кэш.

Когда ``debug`` отключен (как правило, в ``prod`` окружении), вы должны
очищать кэш в директории Twig для того чтобы шаблоны перекомпилировались.
Помните об этом при выкладке вашего приложения на сервер.

.. index::
   single: Шаблонизатор; Наследование

Наследование шаблонов и Layout
------------------------------

Обычно в проекте шаблоны используют некоторое количество общих элементов, таких
как "шапка" (header), "подвал" (footer), боковые панели и т.п. В Symfony2 мы
решаем эту проблему по другому: шаблон может быть декорирован другим шаблоном.
Это работает точно также как с классами PHP: наследование шаблонов позволяет
вам создавать базовый шаблон - т.н. **layout**, который содержит все базовые
элементы вашего сайта, называемые **блоками** (аналогично "PHP-классу с
базовыми методами"). Дочерний шаблон может расширять базовый шаблон и
переопределять любой его блок (аналогично "дочерний PHP-класс может
переопределять некоторые методы родительского класса").

Сначала создайте файл базового шаблона (layout):

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title>{% block title %}Test Application{% endblock %}</title>
            </head>
            <body>
                <div id="sidebar">
                    {% block sidebar %}
                    <ul>
                        <li><a href="/">Home</a></li>
                        <li><a href="/blog">Blog</a></li>
                    </ul>
                    {% endblock %}
                </div>

                <div id="content">
                    {% block body %}{% endblock %}
                </div>
            </body>
        </html>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title><?php $view['slots']->output('title', 'Test Application') ?></title>
            </head>
            <body>
                <div id="sidebar">
                    <?php if ($view['slots']->has('sidebar'): ?>
                        <?php $view['slots']->output('sidebar') ?>
                    <?php else: ?>
                        <ul>
                            <li><a href="/">Home</a></li>
                            <li><a href="/blog">Blog</a></li>
                        </ul>
                    <?php endif; ?>
                </div>

                <div id="content">
                    <?php $view['slots']->output('body') ?>
                </div>
            </body>
        </html>

.. note::

    Хотя обсуждение наследования шаблонов будет вестись в терминах Twig,
    в PHP шаблонах используется та же философия.

Этот шаблон определяет базовый скелетон HTML-документа для простой страницы
с двумя колонками. В этом примере определено три области ``{% block %}`` (
``title``, ``sidebar`` и ``body``). Каждый блок может быть переопределён
дочерним шаблоном, иначе будет сохранена изначальная реализация этих блоков.
Это шаблон может также быть отображен самостоятельно. В этом случае блоки ``title``,
``sidebar`` и ``body`` будут содержать свои значения по умолчанию.

Дочерний шаблон может выглядеть следующим образом:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Blog/index.html.twig #}
        {% extends '::base.html.twig' %}

        {% block title %}My cool blog posts{% endblock %}

        {% block body %}
            {% for entry in blog_entries %}
                <h2>{{ entry.title }}</h2>
                <p>{{ entry.body }}</p>
            {% endfor %}
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/BlogBundle/Resources/views/Blog/index.html.php -->
        <?php $view->extend('::base.html.php') ?>

        <?php $view['slots']->set('title', 'My cool blog posts') ?>

        <?php $view['slots']->start('body') ?>
            <?php foreach ($blog_entries as $entry): ?>
                <h2><?php echo $entry->getTitle() ?></h2>
                <p><?php echo $entry->getBody() ?></p>
            <?php endforeach; ?>
        <?php $view['slots']->stop() ?>

.. note::

   Родительский шаблон определяется благодаря специальному синтаксису (
   ``::base.html.twig``), который указывает, что шаблон расположен в
   директории проекта ``app/Resources/views``. Этот синтаксис будет рассматриваться
   в секции :ref:`template-naming-locations`.

Ключом к наследованию шаблонов является таг ``{% extends %}``. Он сообщает
движку шаблонизатора, что необходимо сначала выполнить базовый шаблон, который
настроит общую разметку и определит некоторое количество блоков. После этого
будет отображаться дочерний шаблон, который указывает, что блоки родителя
``title`` и ``body`` будут замещаться аналогичными блоками потомка. В
зависимости от значение переменной ``blog_entries``, результат может быть
таким:

.. code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            <title>My cool blog posts</title>
        </head>
        <body>
            <div id="sidebar">
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            </div>

            <div id="content">
                <h2>My first post</h2>
                <p>The body of the first post.</p>

                <h2>Another post</h2>
                <p>The body of the second post.</p>
            </div>
        </body>
    </html>

Обратите внимание, что дочерний шаблон не определяет блок ``sidebar``, поэтому
используется значение из родительского шаблона. Контент тага ``{% block %}``
внутри родительского шаблона всегда будет использоваться по умолчанию.

Вы можете использовать столько уровней наследования, сколько вам требуется. В
следующей секции будет разобрано типичное трёхуровневое наследование, а
также будет рассказано о том, как шаблоны располагаются внутри Symfony2 проекта.

При работе с наследованием шаблонов есть несколько правил, о которых нужно помнить:

* Если вы используете таг ``{% extends %}`` в шаблоне, он должен быть первым
  тагом в этом шаблоне.

* Чем больше тагов ``{% block %}`` у вас в базовом шаблоне, тем лучше. Запомните,
  дочерний шаблон не обязан реализовывать все блоки родителя, поэтому создавайте
  столько блоков в базовом шаблоне, сколько хотите и указывайте для них разумный
  контент по умолчанию. Чем больше блоков имеет ваш базовый шаблон, тем более
  гибким будет ваш layout.

* Если вы обнаружите повторяющийся контент в нескольких шаблонах, вероятно
  это означает, что лучше бы переместить этот контент в ``{% block %}``
  родительского шаблона. В некоторых случаях, более удачным решением будет
  создание нового шаблона и его подключение (см. :ref:`including-templates`).

* Если вам нужен контент блока из родительского шаблона, вы можете использовать
  функцию ``{{ parent() }}``. Это удобно, в случае если вы хотите добавить
  к контенту родительского блока что-либо, вместо того, чтобы полностью
  заменять его.

    .. code-block:: html+jinja

        {% block sidebar %}
            <h3>Table of Contents</h3>
            ...
            {{ parent() }}
        {% endblock %}

.. index::
   single: Шаблонизатор; Правила именования
   single: Шаблонизатор; Расположение файлов

.. _template-naming-locations:

Правила именования и расположения Шаблонов
------------------------------------------

По умолчанию, шаблону могут располагаться в двух различных местах:

* ``app/Resources/views/``: Директория ``views`` может содержать
  шаблоны, общие для всего приложения (например layout приложения), а также
  шаблоны, которые переопределяют шаблоны пакетов (см. :ref:`overriding-bundle-templates`);

* ``путь/к/пакету/Resources/views/``: Каждый пакет содержит свои собственные шаблоны
  в директории ``Resources/views`` (и её поддиректориях). Большинство шаблонов будет
  располагаться внутри пакета.

Symfony2 использует синтаксис **bundle**:**controller**:**template** для шаблонов.
Это позволяет определять место расположения для различных типов шаблонов,
каждый из которых располагается в определённом месте:

* ``AcmeBlogBundle:Blog:index.html.twig``: Эта форма записи используется для
  шаблона определённой страницы. Эти три строки, разделённые двоеточием (``:``)
  означает следующее:

    * ``AcmeBlogBundle``: (*пакет*), шаблон расположен внутри пакета
      ``AcmeBlogBundle`` (например ``src/Acme/BlogBundle``);

    * ``Blog``: (*контроллер*), указывает, что шаблон расположен внутри субдиректории ``Blog``
      директории ``Resources/views``;

    * ``index.html.twig``: (*шаблон*), собственно имя файла - ``index.html.twig``.

  При условии что ``AcmeBlogBundle`` расположен в директории ``src/Acme/BlogBundle``,
  полный путь к файлу шаблона будет следующий:
  ``src/Acme/BlogBundle/Resources/views/Blog/index.html.twig``.

* ``AcmeBlogBundle::layout.html.twig``: Эта форма записи сообщает, что это базовый
  шаблон для пакета ``AcmeBlogBundle``. Так как наименование контроллера не указано,
  шаблон располагается в директории ``Resources/views/layout.html.twig`` пакета
  ``AcmeBlogBundle``.

* ``::base.html.twig``:  Эта форма записи ссылается на шаблон или мастер-шаблон
  (layout) уровня всего приложения. Обратите внимание, что эта строка начинается
  с двух двоеточий (``::``), что означает следующее: шаблон не принадлежит никакому
  пакету и расположен он в директории ``app/Resources/views/``.

В секции :ref:`overriding-bundle-templates` вы узнаете как любой шаблон, находящийся,
например, в пакете ``AcmeBlogBundle``, может быть переопределён путём размещения шаблона
с тем же именем в директории ``app/Resources/AcmeBlogBundle/views/``. Это даёт
возможность переопределять любые шаблоны любого стороннего пакета.

.. tip::

    Надеемся синтаксис именования шаблонов показался вам знакомым - такой же
    формат используется и для контроллеров (см. :ref:`controller-string-syntax`).

Суффиксы Шаблонов
~~~~~~~~~~~~~~~~~

Формат **bundle**:**controller**:**template** имени шаблона указывает *где*
шаблон находится. Каждое имя шаблона также имеет два расширения, которые
определяют *формат* и *тип шаблонизатора* для этого шаблона.

* **AcmeBlogBundle:Blog:index.html.twig** - HTML формат, шаблонизатор - Twig;

* **AcmeBlogBundle:Blog:index.html.php** - HTML формат, шаблонизатор - PHP;

* **AcmeBlogBundle:Blog:index.css.twig** - CSS формат, шаблонизатор - Twig.

По умолчанию, любой шаблон в Symfony2 может быть написан либо на Twig либо
на PHP и последняя часть расширения (``.twig`` или ``.php``) указывает,
какой из этих двух шаблонизаторов будет использован. Первая часть расширения
(``.html``, ``.css`` и т.д.) - это конечный формат, который шаблон будет
генерировать. В отличие от типа шаблонизатора, который определяет как Symfony2
будет анализировать шаблон, указание формата всего лишь способ организации
шаблонов, в случае если один ресурс может быть отображен и как HTML
(``index.html.twig``), и как XML (``index.xml.twig``) и в любом другом
формате, который может потребоваться. Дополнительную информацию ищите
в секции :ref:`template-formats`.

.. note::

   Доступные "движки" шаблонизаторов можно настроить и даже добавить новые.
   Дополнительную информацию ищите в секции о
   :ref:`Настройке шаблонизатора<template-configuration>`

.. index::
   single: Шаблонизатор; Таги и Хелперы
   single: Шаблонизатор; Хелперы

Таги и Хелперы
----------------

.. note::

    Примечание переводчика: здесь и далее функция-"помощник" (``helper``) будет
    обозначена как **хелпер**.

Вы уже узнали основы создания шаблонов, как они именуются и как работает
наследование шаблонов. Самое тяжелое уже позади. В этой секции вы узнаете
о множестве инструментов, помогающих выполнять типичные для шаблонов задачи,
такие как подключение других шаблонов, создание ссылок на страницы и
вставку изображений.

Symfony2 содержит много специализированных тагов и функций Twig, которые упрощают
работу дизайнера шаблонов. PHP шаблонизатор предоставляет расширяемую систему
*хелперов*, которая предоставляет полезные функции в рамках шаблона.

Мы уже видели несколько встроенных в Twig тагов (``{% block %}`` & ``{% extends %}``),
а также пример PHP-хелпера (``$view['slots']``). Давайте же узнаем и о других.

.. index::
   single: Шаблонизатор; Подключение других шаблонов

.. _including-templates:

Подключение других шаблонов
~~~~~~~~~~~~~~~~~~~~~~~~~

На практике у вас часто будет возникать потребность подключить один и тот
же шаблон или же фрагмент кода для многих страниц. Например, в приложении с
некоторыми статьями, код шаблона, отображающий одну статью, может быть
использован на странице статьи, на странице, отображающей наиболее популярные
статьи или же на странице со списком последних статей.

Когда вам необходимо использовать некоторый блок PHP-кода, вы выносите этот
код в класс или в функцию. Тоже верно и для шаблонов. Переместив код шаблона,
используемый в нескольких местах, в отдельный файл, впоследствии он может быть
подключен к любому другому шаблону. Сначала создайте шаблон, который хотите
использовать в нескольких местах:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/articleDetails.html.twig #}
        <h1>{{ article.title }}</h1>
        <h3 class="byline">by {{ article.authorName }}</h3>

        <p>
          {{ article.body }}
        </p>

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/articleDetails.html.php -->
        <h2><?php echo $article->getTitle() ?></h2>
        <h3 class="byline">by <?php echo $article->getAuthorName() ?></h3>

        <p>
          <?php echo $article->getBody() ?>
        </p>

Подключить этот шаблон к любому другому несложно:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/Article/list.html.twig #}
        {% extends 'AcmeArticleBundle::layout.html.twig' %}

        {% block body %}
            <h1>Recent Articles<h1>

            {% for article in articles %}
                {% include 'AcmeArticleBundle:Article:articleDetails.html.twig' with {'article': article} %}
            {% endfor %}
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/Article/list.html.php -->
        <?php $view->extend('AcmeArticleBundle::layout.html.php') ?>

        <?php $view['slots']->start('body') ?>
            <h1>Recent Articles</h1>

            <?php foreach ($articles as $article): ?>
                <?php echo $view->render('AcmeArticleBundle:Article:articleDetails.html.php', array('article' => $article)) ?>
            <?php endforeach; ?>
        <?php $view['slots']->stop() ?>

Шаблон подключается при помощи тага ``{% include %}``. Обратите внимание, что
имя шаблона следует типовым конвенциям об именовании. Шаблон ``articleDetails.html.twig``
использует переменную ``article``. Она передаётся в него из шаблона ``list.html.twig``
при помощи команды ``with``.

.. tip::

    Выражение ``{'article': article}`` - это стандартный синтаксис для
    хэшей (ассоциативных массивов) в Twig. Если вам нужно передать много
    элементов - массив будет выглядеть следующим образом: ``{'foo': foo, 'bar': bar}``.

.. index::
   single: Шаблонизатор; Внедрение контроллеров

.. _templating-embedding-controller:

Внедрение контроллеров
~~~~~~~~~~~~~~~~~~~~~

В некоторых случаях, вам может потребоваться нечто большее, нежели просто
подключение шаблонов. Положим, у вас есть боковая панель в шаблоне, которая содержит
три самых последних статьи. Получение этих трёх статей может включать запросы к
базе данных или же выполнение некоторых других операций, которые нельзя
выполнить непосредственно из шаблона.

Решением в данном случае является встраивание в ваш шаблон результата контроллера
целиком. Во-первых, создайте контроллер, который будет отображать некое число последних
статей:

.. code-block:: php

    <?php
    // src/Acme/ArticleBundle/Controller/ArticleController.php

    class ArticleController extends Controller
    {
        public function recentArticlesAction($max = 3)
        {
            // make a database call or other logic to get the "$max" most recent articles
            $articles = ...;

            return $this->render('AcmeArticleBundle:Article:recentList.html.twig', array('articles' => $articles));
        }
    }

Шаблон ``recentList`` очень прост:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/recentList.html.twig #}
        {% for article in articles %}
          <a href="/article/{{ article.slug }}">
              {{ article.title }}
          </a>
        {% endfor %}

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/recentList.html.php -->
        <?php foreach ($articles in $article): ?>
            <a href="/article/<?php echo $article->getSlug() ?>">
                <?php echo $article->getTitle() ?>
            </a>
        <?php endforeach; ?>

.. note::

    Обратите внимание, что мы слукавили и захардкодили URL статьи в этом примере
    (``/article/*slug*``). Это очень плохая практика. В следующей секции вы
    узнаете, как правильно создавать ссылки на страницы приложения.

Для того чтобы подключить контроллер, вам необходимо сослаться на него,
используя стандартный синтаксис логического имени котроллера
(**bundle**:**controller**:**action**):

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        ...

        <div id="sidebar">
            {% render "AcmeArticleBundle:Article:recentArticles" with {'max': 3} %}
        </div>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        ...

        <div id="sidebar">
            <?php echo $view['actions']->render('AcmeArticleBundle:Article:recentArticles', array('max' => 3)) ?>
        </div>

Всякий раз, когда вы понимаете, что вам нужна переменная или данные, к
которым вы не можете получить доступ из шаблона, обязательно рассмотрите
вариант с встраиванием контроллера. Контроллеры быстро выполняются и
способствуют хорошей организации кода и его повторному
использованию.

.. index::
   single: Маршрутизация; Ссылки на страницы

Создание ссылок на страницы
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Создание ссылок на другие страницы вашего приложения - это одна из типичных
операций в шаблоне. Вместо того, чтобы хардкодить URL в шаблоне, используйте
Twig-функцию ``path`` (в PHP - хелпер ``router``) для создания URL, основанных
на конфигурации маршрутизатора. Потом, если вы захотите изменить URL некоторой
страницы, вам всего лишь потребуется изменить конфигурацию маршрутизатора. Шаблоны
автоматически сгенерируют новый URL.

Сначала создадим ссылку на страницу "_welcome", которая определяется следующей конфигураций
маршрутизатора:

.. configuration-block::

    .. code-block:: yaml

        _welcome:
            pattern:  /
            defaults: { _controller: AcmeDemoBundle:Welcome:index }

    .. code-block:: xml

        <route id="_welcome" pattern="/">
            <default key="_controller">AcmeDemoBundle:Welcome:index</default>
        </route>

    .. code-block:: php

        <?php
        $collection = new RouteCollection();
        $collection->add('_welcome', new Route('/', array(
            '_controller' => 'AcmeDemoBundle:Welcome:index',
        )));

        return $collection;

Для создания ссылки на страницу используйте функцию ``path`` и маршрут:

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('_welcome') }}">Home</a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('_welcome') ?>">Home</a>

Как и ожидалось, она сгенерирует URL ``/``. Давайте теперь посмотрим, как это
работает для более сложных маршрутов:

.. configuration-block::

    .. code-block:: yaml

        article_show:
            pattern:  /article/{slug}
            defaults: { _controller: AcmeArticleBundle:Article:show }

    .. code-block:: xml

        <route id="article_show" pattern="/article/{slug}">
            <default key="_controller">AcmeArticleBundle:Article:show</default>
        </route>

    .. code-block:: php

        <?php
        $collection = new RouteCollection();
        $collection->add('article_show', new Route('/article/{slug}', array(
            '_controller' => 'AcmeArticleBundle:Article:show',
        )));

        return $collection;

В этом случае, вам нужно указать и имя маршрута (``article_show``) и значение
параметра ``{slug}``. Используя этот маршрут, давайте вернёмся к шаблону
``recentList`` из предыдущей секции и создадим ссылку на статью правильно:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/recentList.html.twig #}
        {% for article in articles %}
          <a href="{{ path('article_show', { 'slug': article.slug }) }}">
              {{ article.title }}
          </a>
        {% endfor %}

    .. code-block:: php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/recentList.html.php -->
        <?php foreach ($articles in $article): ?>
            <a href="<?php echo $view['router']->generate('article_show', array('slug' => $article->getSlug()) ?>">
                <?php echo $article->getTitle() ?>
            </a>
        <?php endforeach; ?>

.. tip::

    Для генерации абсолютных URL необходимо использовать Twig-функцию ``url``:

    .. code-block:: html+jinja

        <a href="{{ url('_welcome') }}">Home</a>

    В PHP-шаблонах для этого нужно передать третий аргумент в метод ``generate()``:

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('_welcome', array(), true) ?>">Home</a>

.. index::
   single: Шаблонизатор; Ссылки на ресурсы

Ссылки на ресурсы (assets)
~~~~~~~~~~~~~~~~~~~~~~~~~~

Шаблоны также часто ссылаются на картинки, скрипты, страницы стилей и прочие
ресурсы (здесь и далее вместо asset будет использован термин *ресурс*). Конечно,
вы можете хардкодить пути к ресурсам (например так ``/images/logo.png``), но
Symfony2 предлагает использовать более гибкую Twig-функцию ``asset``:

.. configuration-block::

    .. code-block:: html+jinja

        <img src="{{ asset('images/logo.png') }}" alt="Symfony!" />

        <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    .. code-block:: php

        <img src="<?php echo $view['assets']->getUrl('images/logo.png') ?>" alt="Symfony!" />

        <link href="<?php echo $view['assets']->getUrl('css/blog.css') ?>" rel="stylesheet" type="text/css" />

Главная задача функции ``asset`` - сделать ваше приложение по возможности более
переносимым. Если приложение находится в корне домена (http://example.com),
тогда будет отображен путь ``/images/logo.png``. Но, если ваше приложение
находится в поддиректории (http://example.com/my_app), путь к ресурсам будет
учитывать эту поддиректорию (``/my_app/images/logo.png``). Функция ``asset``
берёт на себя заботу по определению того, как именно ваше приложение используется
и генерирует соответствующие пути.

В дополнение к этому, если использовать функцию ``asset``, Symfony сможет
автоматически добавлять строку запроса к ресурсам, чтобы исключить их кеширование
при выгрузке на сервер. Например, ``/images/logo.png`` может выглядеть так:
``/images/logo.png?v2``. Подробнее об этом смотрите тут: :ref:`ref-framework-assets-version`.

.. index::
   single: Шаблонизатор; Подключение CSS и Javascript файлов
   single: Стили; Подключение CSS файлов
   single: Скрипты Javascript; Подключение Javascript файлов

Подключение CSS и Javascript файлов в Twig
------------------------------------------

Ни один современный сайт не может обойтись без подключения CSS и Javascript
файлов. В Symfony, подключение этих ресурсов элегантно обрабатывается, опираясь
на возможности наследования шаблонов.

.. tip::

    В этой секции вы узнаете философию подключения CSS и Javascript файлов
    в Symfony. Symfony также содержит библиотеку Assetic, которая следует
    той же философии, но позволяет вам также выполнять много интересных
    операций над этими ресурсами. Дополнительную информацию можно получить
    в книге рецептов: :doc:`/cookbook/assetic/asset_management`.

Давайте начнём с добавления двух блоков к базовому шаблону, который будет
подключать ваши ресурсы: один назовём ``stylesheets``, располагаться он
будет внутри HTML-тага ``head``, другой назовём ``javascripts`` и размещаться
он будет перед закрывающим HTML-тагом ``body``. Эти блоки будут содержать
все стили и скрипты, которые вам требуются для сайта:

.. code-block:: html+jinja

    {# 'app/Resources/views/base.html.twig' #}
    <html>
        <head>
            {# ... #}

            {% block stylesheets %}
                <link href="{{ asset('/css/main.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
        </head>
        <body>
            {# ... #}

            {% block javascripts %}
                <script src="{{ asset('/js/main.js') }}" type="text/javascript"></script>
            {% endblock %}
        </body>
    </html>

Проще некуда! Но что, если вам потребуется включить дополнительный файл
стилей или скрипт в дочернем шаблоне? Например, положим у вас есть
страница контактов и вам нужно подключить файл стилей ``contact.css``
*лишь на одной этой странице*. Внутри шаблона страницы contact необходимо
выполнить следующее:

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Contact/contact.html.twig #}
    {# extends '::base.html.twig' #}

    {% block stylesheets %}
        {{ parent() }}

        <link href="{{ asset('/css/contact.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}

    {# ... #}

В дочернем шаблоне вы просто переопределяете блок ``stylesheets`` и
размещаете новый стиль внутри этого блока. Конечно же, поскольку вам
нужно добавить стиль, а не изменить его, вы должны использовать функцию
``parent()`` для того, чтобы получить все стили из родительского шаблона.

Конечным результатом является страница, которая включает как ``main.css``,
так и ``contact.css``

.. index::
   single: Шаблонизатор; Сервис шаблонизатора

Настройка и использование сервиса ``шаблонизатора``
--------------------------------------------------

Сердцем системы шаблонов Symfony2 является её "движок" (``Engine``).
Это специализированный объект, который отвечает за отображение шаблонов
и возврат их контента. Например, когда вы отображаете шаблон из контроллера,
вы используете сервис шаблонизатора:

.. code-block:: php

    <?php
    return $this->render('AcmeArticleBundle:Article:index.html.twig');

Этот код эквивалентен следующему:

.. code-block:: php

    <?php
    $engine = $this->container->get('templating');
    $content = $engine->render('AcmeArticleBundle:Article:index.html.twig');

    return $response = new Response($content);

.. _template-configuration:

Сервис шаблонизатора предварительно настроен для автоматической работы внутри
Symfony2. Естественно он может быть настроен через файл с настройками
приложения:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            templating: { engines: ['twig'] }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:templating>
            <framework:engine id="twig" />
        </framework:templating>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'templating'      => array(
                'engines' => array('twig'),
            ),
        ));

Для настройки доступно много разных опций, которые описаны в
:doc:`Приложении о Конфигурации</reference/configuration/framework>`.

.. note::

   ``Twig`` необходим для использования веб-профайлера (а также многих
   пакетов от сторонних разработчиков).

.. index::
    single; Шаблонизатор; Переопределение шаблонов

.. _overriding-bundle-templates:

Переопределение шаблонов пакета
-------------------------------

Сообщество Symfony2 гордится собой за то, что его энтузиастами создано и
поддерживается много различных качественных пакетов (см. `Symfony2Bundles.org`_)
на любой случай жизни. Если вы используете сторонние пакеты, вам может
потребоваться изменять их шаблоны.

Предположим, вы подключили воображаемый пакет с открытым исходным кодом
``AcmeBlogBundle``. Вы, в общем-то, всем довольны, но вам хотелось бы заменить
страницу "list" для блога, чтобы настроить её отображение под ваше приложение.
Если вы залезете в контроллер ``Blog`` пакета ``AcmeBlogBundle``, вы найдёте
следующий код:

.. code-block:: php

    <?php
    public function indexAction()
    {
        $blogs = // получение записей в блоге

        $this->render('AcmeBlogBundle:Blog:index.html.twig', array('blogs' => $blogs));
    }

Когда отображается шаблон ``AcmeBlogBundle:Blog:index.html.twig`` Symfony2 на самом деле
ищет его в двух местах:

#. ``app/Resources/AcmeBlogBundle/views/Blog/index.html.twig``
#. ``src/Acme/BlogBundle/Resources/views/Blog/index.html.twig``

Для того чтобы переопределить шаблон из пакета, просто скопируйте его
в директорию ``app/Resources/AcmeBlogBundle/views/Blog/index.html.twig``
(директорию ``app/Resources/AcmeBlogBundle`` нужно создать, так как по умолчанию
её там не будет). Теперь вы можете настраивать шаблон по вашему усмотрению.

Эта логика также применима к базовому шаблону пакета. Положим что каждый
шаблон в пакете ``AcmeBlogBundle`` наследуется от базового шаблона
``AcmeBlogBundle::layout.html.twig``. Как и ранее, Symfony2 будет искать
 этот шаблон в двух местах:

#. ``app/Resources/AcmeBlogBundle/views/layout.html.twig``
#. ``src/Acme/BlogBundle/Resources/views/layout.html.twig``

Как и ранее, для переопределения шаблона, просто скопируйте его из
пакета ``app/Resources/AcmeBlogBundle/views/layout.html.twig``.
После этого вы вольны править его по своему усмотрению.

Если вы сделаете шаг назад, вы увидите, что Symfony2 всегда начинает искать
файл шаблона в директории ``app/Resources/{BUNDLE_NAME}/views/``. Если
там его нет, поиск продолжается в директории ``Resources/views`` пакета.
Это значит, что все шаблоны любого пакета могут быть переопределены в директории
пакета внутри ``app/Resources``.

.. _templating-overriding-core-templates:

.. index::
    single; Шаблонизатор; Переопределение шаблонов для исключений

Переопределение шаблонов ядра
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Так как Symfony2 это тоже пакет, его шаблоны также можно переопределить
тем же образом. Например, ``TwigBundle`` содержит несколько шаблонов для
различных исключительных ситуаций и ошибок, которые могут быть переопределены,
если их скопировать из директории ``Resources/views/Exception`` пакета
``TwigBundle`` в директорию ``app/Resources/TwigBundle/views/Exception``.

.. index::
   single: Шаблонизатор; Трёхуровневое наследование

Трёхуровневое наследование
--------------------------

Один из способов использовать наследование - трёхуровневый подход. Этот
метод замечательно работает с тремя различными типами шаблонов, которые
мы уже рассмотрели:

* Создайте файл ``app/Resources/views/base.html.twig``, который содержит
  базовую разметку приложения (как в предыдущем примере). Внутри приложения
  такой шаблон называется ``::base.html.twig``;

* Создайте файл для каждой секции сайта. Например ``AcmeBlogBundle`` будет содержать
  шаблон ``AcmeBlogBundle::layout.html.twig``, который включает только элементы
  специфичные для блога;

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/layout.html.twig #}
        {% extends '::base.html.twig' %}

        {% block body %}
            <h1>Blog Application</h1>

            {% block content %}{% endblock %}
        {% endblock %}

* Создайте шаблоны для каждой страницы и унаследуйте из от шаблона соответствующей
  секции (пакета). Например, страница "index" будет вызывать что-то типа
  ``AcmeBlogBundle:Blog:index.html.twig`` и отображать записи блога:

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Blog/index.html.twig #}
        {% extends 'AcmeBlogBundle::layout.html.twig' %}

        {% block content %}
            {% for entry in blog_entries %}
                <h2>{{ entry.title }}</h2>
                <p>{{ entry.body }}</p>
            {% endfor %}
        {% endblock %}

Обратите внимание, что этот шаблон наследуется от шаблона секции
``AcmeBlogBundle::layout.html.twig``, который, в свою очередь, наследуется
от базового шаблона приложения (``::base.html.twig``). Это и есть типичное
трёхуровневое наследование.

При создании приложения вы можете выбрать - будете ли вы следовать этому
методу иди же каждый шаблон будет наследоваться напрямую от базового шаблона
приложения (``{% extends '::base.html.twig' %}``).  Трёхуровневая модель
является проверенным и хорошо зарекомендовавшим себя методом в сторонних
пакетах, так как базовый шаблон пакета может быть легко переопределён для
того чтобы использовать шаблон вашего приложения.

.. index::
   single: Шаблонизатор; Экранирование

Экранирование
-------------

При создании HTML из шаблона, всегда есть риск, что переменная шаблона
будет содержать HTML-код или опасный клиентский скрипт. В результате этот
контент может сломать HTML разметку страницы или же позволить злоумышленнику
выполнить `Cross Site Scripting`_ (XSS) атаку. Вот классический пример этого:

.. configuration-block::

    .. code-block:: jinja

        Hello {{ name }}

    .. code-block:: php

        Hello <?php echo $name ?>

Представьте, что пользователь ввёл следующий код в качестве имени::

    <script>alert('hello!')</script>

Без экранирования, шаблон отобразит::

    Hello <script>alert('hello!')</script>

а клиент (браузер) выполнит JavaScript код и отобразит окошко alert.

Этот пример выглядит безобидным, но этот же пользователь может также
написать скрипт, который выполнит вредоносные действия в защищённой
зоне приложения как будто бы у него были права на это.

Ответом на данную проблему является экранирование (output escaping). При
наличии экранирования, тот же код будет отображен совершенно безобидно::

    Hello &lt;script&gt;alert(&#39;helloe&#39;)&lt;/script&gt;

Twig и PHP шаблонизаторы решают эту проблему различным образом. Если
вы используете Twig, экранирование включено по умолчанию и ваши шаблоны
защищены. В PHP экранирование не автоматическое и подразумевает что вы
вручную будете экранировать данные при необходимости.

Экранирование в Twig
~~~~~~~~~~~~~~~~~~~~

Если вы используете шаблоны Twig, экранирование включено по умолчанию. Это
означает, что ваш код защищён от неожиданных действий пользователей "из коробки".
По умолчанию, экранирование подразумевает, что контент будет экранирован для HTML.

В некоторых случаях вам может потребоваться отключить экранирование, когда вы
отображаете переменную, которой доверяете и которая содержит HTML-разметку,
которую не нужно экранировать. Положим, что администратор имеет возможность писать
статьи, которые содержат HTML-код. По умолчанию Twig будет экранировать тело
статьи. Для того чтобы отобразить его обычным образом необходимо добавить фильтр
``raw``: ``{{ article.body | raw }}``.

Вы также можете отключить экранирование внутри блока или же для шаблона
целиком. Подробнее это описано в документации Twig: `Output Escaping`_.

Экранирование в PHP
~~~~~~~~~~~~~~~~~~~

Экранирование в PHP шаблонах не автоматическое. Это означает, что если
вы не экранировали переменную - вы не защищены. Для экранирования необходимо
использовать специальный метод ``escape()``::

    Hello <?php echo $view->escape($name) ?>

По умолчанию, метод ``escape()`` полагает, что переменная отображается в HTML
контексте (и соответственно переменная экранируется чтобы быть безопасной в HTML).
Второй аргумент позволяет вам изменить контекст. Например, чтобы вывести что-либо
в JavaScript используйте контекст ``js``:

.. code-block:: js

    var myMsg = 'Hello <?php echo $view->escape($name, 'js') ?>';

.. index::
   single: Шаблонизатор; Форматы

.. _template-formats:

Форматы шаблонов
----------------

Шаблоны - это основной способ для отображения контента в *любом* формате.
В большинстве случаев вы будете отображать HTML контент, но шаблон также
может быть использован для генерации JavaScript, CSS, XML или же любого
другого формата на ваш выбор.

Например, один и тот же ресурс может отображаться в разных форматах. Для отображения
индексной страницы статей в XML формате, просто добавьте формат в имя шаблона:

* *XML template name*: ``AcmeArticleBundle:Article:index.xml.twig``
* *XML template filename*: ``index.xml.twig``

По сути это всего лишь соглашение по именованию шаблонов - шаблоны разных форматов
не будут отображаться разными способами.

Во многих случаях вам может потребоваться один и тот же контроллер отобразить
в нескольких форматах в зависимости от формата запроса. Вы можете выполнить это
следующим образом:

.. code-block:: php

    <?php
    public function indexAction()
    {
        $format = $this->getRequest()->getRequestFormat();

        return $this->render('AcmeBlogBundle:Blog:index.'.$format.'.twig');
    }

Метод ``getRequestFormat`` объекта ``Request`` по умолчанию возвращает ``html``,
но может также возвращать любой формат запрошенный пользователем.
Формат запроса часто управляется маршрутизатором, где маршрут может быть
настроен таким образом чтобы URL ``/contact`` возвращал HTML а
``/contact.xml`` устанавливал бы формат запроса XML и контроллер
будет возвращать XML. Более подробно этот вопрос рассматривается в главе
о Маршрутизации - :ref:`Продвинутая маршрутизация <advanced-routing-example>`.

Для создания ссылок, которые используют параметр формата - добавьте ключ
``_format`` к хешу параметров:

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('article_show', {'id': 123, '_format': 'pdf'}) }}">
            PDF Version
        </a>

    .. code-block:: html+php

        <a href="<?php echo $view['router']->generate('article_show', array('id' => 123, '_format' => 'pdf')) ?>">
            PDF Version
        </a>

Заключение
----------

Шаблонизатор Symfony - это мощный инструмент, который может быть
использован каждый раз, когда вам необходимо сгенерировать контент в
HTML, XML или любом другом формате. И, не смотря на то, что шаблоны - это
обычный способ генерации контента в контроллере, он не обязателен. Объект
``Response``, возвращаемый контроллером может быть создан как с
использованием шаблонизатора, так и без него:

.. code-block:: php

    <?php
    // creates a Response object whose content is the rendered template
    $response = $this->render('AcmeArticleBundle:Article:index.html.twig');

    // creates a Response object whose content is simple text
    $response = new Response('response content');

Шаблонизатор Symfony - гибок и позволяет по умолчанию использовать
два различных "визуализатора": традиционные *PHP* шаблоны и мощные *Twig*
шаблоны. Оба этих типа шаблонов поддерживают иерархию и укомплектованы
богатым набором функций-помощников, предназначенных для выполнения
повседневных задач.

В целом, шаблоны следует рассматривать как мощный инструмент в вашем распоряжении.
В некоторых случаях вам может и не потребоваться отображать шаблон - и в Symfony2
это тоже совершенно нормальная ситуация.

Читайте в книге рецептов
----------------------------

* :doc:`/cookbook/templating/PHP`
* :doc:`/cookbook/controller/error_pages`

.. _`Twig`: http://twig.sensiolabs.org
.. _`Symfony2Bundles.org`: http://symfony2bundles.org
.. _`Cross Site Scripting`: http://en.wikipedia.org/wiki/Cross-site_scripting
.. _`Output Escaping`: http://twig.sensiolabs.org
.. _`tags`: http://twig.sensiolabs.org/doc/tags/index.html
.. _`filters`: http://twig.sensiolabs.org/doc/templates.html#filters
.. _`создавать свои собственные расширения`: http://twig.sensiolabs.org/doc/advanced.html

.. toctree::
    :hidden:

    Translation source: 2011-09-29 9367dc0
    Corrected from: 2011-10-16 a9e1ed7
