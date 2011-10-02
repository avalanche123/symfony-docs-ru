Как предложить свой вариант перевода?
=====================================

1. Для начала, вам `нужно сделать Fork <http://help.github.com/forking/>`_
   текущего `репозитория с переводом <https://github.com/avalanche123/symfony-docs-ru>`_

2. В своем репозитории перевода `создайте branch <http://www.kernel.org/pub/software/scm/git/docs/git-branch.html>`_
   и внесите законченный набор изменений в перевод. Желательно, чтобы изменения
   касались не более чем 10 файлов на один ``Pull Request``

3. Актуальную версию английской документации берите из `репозитория
   symfony-docs <https://github.com/symfony/symfony-docs>`_

4. Когда вы закончили правки в переводе, из нужного ``branch``
   `сделайте Pull Request <http://help.github.com/pull-requests/>`_

Соглашения по переводу
======================

1. Некоторые частные случаи:

+----------------------+------------------------+--------------------------+
| **Исходный вариант** | **Правильный перевод** | **Неправильный перевод** |
+----------------------+------------------------+--------------------------+
| **Response object**  | объект Response        | объект ответа            |
+----------------------+------------------------+--------------------------+

Не переводятся
==============

1. Служебная разметка::

    - :orphan:
    - .. glossary::
    - .. note::
    - .. tip::
    - .. toctree::
         :maxdepth: 2
         :glob:
         :numbered:

2. Служебные слова: "``API``", "``HTTP``", "``Symfony2``", "``Controller``", "``Response``", etc.

Полезные ссылки
===============

* Правила форматирования документации: `Symfony2 Documentation Format
<http://symfony.com/doc/current/contributing/documentation/format.html>`_ и
`reStructuredText Markup Specification <http://docutils.sourceforge.net/docs/ref/rst/restructuredtext.html>`_
