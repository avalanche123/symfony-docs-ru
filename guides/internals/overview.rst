.. index::
   single: Internals

Обзор внутренностей
==================

Похоже что вы хотите понять как работает Symfony2 и как его расширить.
Это меня радует! Этот раздел подробно объясняет внутренности Symfony2.

.. note::

    Чтение этого раздела необходимо только если вы хотите понять как работает
    Symfony2 за кулисами или если хотите расширять Symfony2.

Код Symfony2 сделан из нескольких независимых слоёв. Каждый слой возведён на
вершине предыдущего.

.. tip::

    Автозагрузка не управляется фреймворком напрямую; она выполняется
    независимо с помощью класса
    :class:`Symfony\\Component\\HttpFoundation\\UniversalClassLoader`
    и файла ``src/autoload.php``. Прочтите :doc:`посвящённую этому главу
    </guides/tools/autoloader>` для дополнительной информации.

``HttpFoundation`` компонент
----------------------------

Глубочайший уровень это компонент :namespace:`Symfony\\Component\\HttpFoundation`.
HttpFoundation предоставляет главные объекты, необходимые для работы с HTTP.
Это объектно-ориентированная абстракция некоторых встроенных PHP функций и
переменных:

* Класс :class:`Symfony\\Component\\HttpFoundation\\Request` абстрагирует
  основные глобальные переменные в PHP, такие как ``$_GET``, ``$_POST``, ``$_COOKIE``,
  ``$_FILES`` и ``$_SERVER``;

* Класс :class:`Symfony\\Component\\HttpFoundation\\Response` абстрагирует
  некоторые PHP функции типа ``header()``, ``setcookie()`` и ``echo``;

* Класс :class:`Symfony\\Component\\HttpFoundation\\Session` и интерфейс
  :class:`Symfony\\Component\\HttpFoundation\\SessionStorage\\SessionStorageInterface`
  абстрагируют функции для работы с сессией ``session_*()``.

.. seealso::

    Узнайте больше о компоненте :doc:`HttpFoundation <http_foundation>`.

``HttpKernel`` компонент
------------------------

Выше HttpFoundation расположен компонент :namespace:`Symfony\\Component\\HttpKernel`.
HttpKernel управляет динамической частью HTTP; это тонкая обёртка для классов
Request и Response чтобы привести способы отбработки запросов к стандарту.
Компонент также предоставляет точки для расширений и инструменты, делающие
его идеальной стартовой площадкой для создания Web фреймворка без лишних проблем.

Также, по желанию, он добавляет настраиваемость и расширяемость благодаря
компоненту Dependency Injection и могучей системе плагинов (бандлов).

.. seealso::

    Узнайте больше о компоненте :doc:`HttpKernel <kernel>`, о
    :doc:`Dependency Injection </guides/dependency_injection/index>` и о
    :doc:`Бандлах </guides/bundles/index>`.

Бандл ``FrameworkBundle``
--------------------------

:namespace:`Symfony\\Bundle\\FrameworkBundle` - это бандл, связывающий
главные компоненты и библиотеки вместе и делающий лёгкий и быстрый MVC фреймворк.
Он поставляется с рациональной первоначальной конфигурацией и соглашениями для
лучшего обучения.
