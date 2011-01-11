Следует переводить наиболее стабильные части документации, в которых немного
аннотаций к коду, т. к. код меняется и аннотации устаревают.


Форматирование
==============

* http://docs.symfony-reloaded.org/contributing/documentation/format.html
* http://sphinx.pocoo.org/rest.html
* http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.html


Публикация патча
================

1. Создайте новый бранч в своём репозитории `git checkout -b patchN`

2. Выберите файлы для перевода

3. Скопируйте их последние версии на английском языке из репозитория
   *symfony-docs*

4. Выполните первичный коммит::

    git add .
    git commit -m "English version"

5. Переведите :)

6. Если переведено не всё::

    git add .
    git commit -m "partial"

   Теперь можно обновить мастер до апстрима, например

7. Когда всё переведено выполните финальный коммит::

    git add .
    git commit -m "Some files done"

8. Добавьте бранч на github::

    git push origin patch

Соглашения по переводу
======================

1. Названия классов стараемся не переводить: ``Response object`` переводим как ``объект Response``, а не ``объект ответа``.

Не переводятся
==============

1. Служебная разметка
~~~~~~~~~~~~~~~~~~~~~

::

    :orphan:

    .. glossary::

    .. note::

    .. tip::

    .. toctree::
        :maxdepth: 2
        :glob:
        :numbered:

2. Служебные слова
~~~~~~~~~~~~~~~~~~

* API
* bundle - бандл?