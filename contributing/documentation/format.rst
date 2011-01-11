Формат документации
====================

Документация Symfony2 использует `reStructuredText`_ как язык разметки и
`Sphinx`_ для создания вывода (HTML, PDF и т. д.).

reStructuredText
----------------

reStructuredText это "легкочитаемый, что видишь то и получишь, синтаксис
разметки открытым текстом и система анализа".

Узнайте больше о его синтаксисе, прочитав Symfony2 `documents`_
или `reStructuredText Primer`_ на web сайте Sphinx.

Если вы знакомы с Markdown, будьте осторожны, т. к. некоторые вещи очень знакомы,
но отличаются:

* Списки начинаются с начала строки (необходимость отступа отсутствует);

* Встроенные блоки кода используют двойные кавычки (````как здесь````).

Sphinx
------

Sphinx - это система сборки, добавляющая полезные инструменты для создания
документации из документов reStructuredText. Она добавляет указания и
роли интерпретированного текста к стандартной reST `markup`_.

Подсветка синтаксиса
~~~~~~~~~~~~~~~~~~~

Все примеры кода подсвечиваются по умолчанию как язык PHP. Вы можете изменить
их через директиву ``code-block``:

.. code-block:: rst

    .. code-block:: yaml

        { foo: bar, bar: { foo: bar, bar: baz } }

Если ваш PHP код начинается с ``<?php``, тогда используйте ``html+php`` как
подсвечиваемый псевдо-язык:

.. code-block:: rst

    .. code-block:: html+php

        <?php echo $this->foobar(); ?>

.. note::

    Список поддерживаемых языков доступен на `Pygments website`_.

Блоки конфигураций
~~~~~~~~~~~~~~~~~~~~

Всякий раз как вы показываете конфигурацию, используйте директиву
``configuration-block`` чтобы отразить конфигурацию во всех поддерживаемых
форматах (``PHP``, ``YAML`` и ``XML``):

.. code-block:: rst

    .. configuration-block::

        .. code-block:: yaml

            # Configuration in YAML

        .. code-block:: xml

            <!-- Configuration in XML //-->

        .. code-block:: php

            // Configuration in PHP

Предыдущая reST разметка отобразится следующим образом:

.. configuration-block::

    .. code-block:: yaml

        # Configuration in YAML

    .. code-block:: xml

        <!-- Configuration in XML //-->

    .. code-block:: php

        // Configuration in PHP

Текущий список поддерживаемых форматов:

=============== ===========
Формат разметки Отображается
=============== ===========
html            HTML
xml             XML
php             PHP
yaml            YAML
jinja           Twig
html+jinja      Twig
jinja+html      Twig
php+html        PHP
html+php        PHP
ini             INI
php-annotations Аннотации
=============== ===========

.. _reStructuredText:        http://docutils.sf.net/rst.html
.. _Sphinx:                  http://sphinx.pocoo.org/
.. _documents:               http://github.com/symfony/symfony-docs
.. _reStructuredText Primer: http://sphinx.pocoo.org/rest.html
.. _markup:                  http://sphinx.pocoo.org/markup/
.. _Pygments website:        http://pygments.org/languages/
