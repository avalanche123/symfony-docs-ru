Как создать и разместить Проект на Symfony2 в Subversion
=========================================================

.. tip::

    Эта статья повествует непосредственно о Subversion и основана на принципах,
    описанных в статье :doc:`/cookbook/workflow/new_project_git`.

Как только Вы прочитали :doc:`/book/page_creation` и познакомились с принципами
Symfony, Вы наверняка уже готовы начинать разрабатывать собственный проект.
Управлять проектами Symfony рекомендуется с помощью системы контроля версии
`git`_, но некоторые предпочитают использовать `Subversion`_, и это тоже
хорошо! После прочтения этого рецепта Вы научитесь управлять проектом,
используя `svn`_, по аналогии, как это делается с помощью `git`_.

.. tip::

    Этот метод позволяет отслеживать Ваш проект на Symfony2 в репозитории
    Subversion. Существует несколько способов сделать это и здесь описывается
    один из них.

Репозиторий Subversion
-------------------------

В этой статье предполагается, что Ваш репозиторий соответствует
стандартной широкораспространенный структуре:

.. code-block:: text

    myproject/
        branches/
        tags/
        trunk/

.. tip::

    Большинство хостингов Subversion должны следовать этой общепринятой
    практике. Это рекомендуемая структура в `Version Control with Subversion`_
    и она используется многими бесплатными хостингами (смотрите :ref:`svn-hosting`).

Начальная настройка проекта
---------------------------

Для начала вам нужно скачать Symfony2 и получить базовые настройки Subversion:

1. Скачайте `Symfony2 Standard Edition`_ со сторонними библиотеками (vendors)
   или без них.

2. Разархивируйте дистрибутив. Будет создана папка с именем Symfony. В ней
   будет располагаться структура нового проекта, файлы конфигурации и т.д.
   Переименуйте ее, как Вам больше нравится.

3. Выгрузите репозиторий Subversion, который будет хранить проект. Скажем, он
   будет расположен в `Google code`_ под именем ``myproject``:

    .. code-block:: bash
    
        $ svn checkout http://myproject.googlecode.com/svn/trunk myproject

4. Скопируйте файлы проекта Symfony2 в директорию Subversion:

    .. code-block:: bash

        $ mv Symfony/* myproject/

5. Давайте сейчас установим правила для игнорируемых файлов. Не все *должно*
   храниться в Вашем репозитории. Некоторые файлы (такие как кеш) будут
   генерироваться, другие (например файлы настройки доступа к базе данных)
   должны быть настроены на каждом компьютере по своему. Это делается
   с помощью свойства ``svn:ignore``. Таким образом мы можем указать
   игнорируемые файлы.

    .. code-block:: bash

        $ cd myproject/
        $ svn add --depth=empty app app/cache app/logs app/config web

        $ svn propset svn:ignore "vendor" .
        $ svn propset svn:ignore "bootstrap*" app/
        $ svn propset svn:ignore "parameters.ini" app/config/
        $ svn propset svn:ignore "*" app/cache/
        $ svn propset svn:ignore "*" app/logs/

        $ svn propset svn:ignore "bundles" web

        $ svn ci -m "commit basic symfony ignore list (vendor, app/bootstrap*, app/config/parameters.ini, app/cache/*, app/logs/*, web/bundles)"

6. Остальные файлы могут быть добавлены и отправлены в репозиторий:

    .. code-block:: bash

        $ svn add --force .
        $ svn ci -m "add basic Symfony Standard 2.X.Y"

7. Скопируйте ``app/config/parameters.ini`` в ``app/config/parameters.ini.dist``.
   Subversion игнорирует файл ``parameters.ini`` (смотрите выше), поэтому
   индивидуальные настройки, такие как пароль от базы данных и т.д., не будут
   попадать в репизиторий. Если мы создадим ``app/config/parameters.ini.dist``,
   новые разработчики смогут быстро выгрузить проект, скопировать файл в
   ``parameters.ini``, настроить его и начать работать над проектом.

8. Наконец, скачайте и установите все сторонние библиотеки:

    .. code-block:: bash
    
        $ php bin/vendors install

.. tip::

    Чтобы запустить ``bin/vendors`` должен быть установлен `git`_. Этот
    протокол  используется для выгрузки сторонних библиотек. Это означает,
    что ``git`` используется как инструмент, который помогает скачать
    библиотеки в директорию ``vendor/``.

На этом этапе Вы имеете полностью функционирующий проект на Symfony2,
хранящийся в репозитории Subversion. Можно начать работу над проектом и
сохранять наработки непосредственно в репозитории Subversion.

Вы можете продолжать изучение главы :doc:`/book/page_creation`, чтобы
лучше понимать, как настраивать и разрабатывать Ваше приложение.

.. tip::

    Symfony2 Standard Edition поставляется с дополнительными примерами. Чтобы
    удалить лишний код, следуйте инструкциям, которые описаны в
    `Standard Edition Readme`_.

.. include:: _vendor_deps.rst.inc

.. _svn-hosting:

Решения для Subversion хостинга
-------------------------------

Главное отличие между `git`_ и `svn`_ в том, что Subversion для работы
*необходим* централизованный репозиторий. Поэтому у Вас есть несколько
решений:

- Собственный хостинг: создайте свой репозиторий и организуйте доступ к нему
  через файловую систему или сеть. Более подробно об этом Вы можете почитать в
  `Version Control with Subversion`_.

- Сторонние хостинги: существует множество надежных бесплатных хостинговых
  решений. Например, `GitHub`_, `Google code`_, `SourceForge`_ или `Gna`_.
  Некоторые из них также предоставляют хостинг для git репозиториев.

.. _`git`: http://git-scm.com/
.. _`svn`: http://subversion.apache.org/
.. _`Subversion`: http://subversion.apache.org/
.. _`Symfony2 Standard Edition`: http://symfony.com/download
.. _`Standard Edition Readme`: https://github.com/symfony/symfony-standard/blob/master/README.md
.. _`Version Control with Subversion`: http://svnbook.red-bean.com/
.. _`GitHub`: http://github.com/
.. _`Google code`: http://code.google.com/hosting/
.. _`SourceForge`: http://sourceforge.net/
.. _`Gna`: http://gna.org/

.. toctree::
    :hidden:

    Translation source:     2012-02-10 [76b75713b5]