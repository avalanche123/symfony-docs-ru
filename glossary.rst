:orphan:

Словарь терминов
================

.. glossary::

    Проект
        *Проект* - это директория, состоящая из Приложения, набора Пакетов,
        сторонних библиотек, автозагрузчика и веб скриптов фронт-контроллера.

    Приложение
        *Приложение* - это директория, содержащая *конфигурацию* для данного
        набора Пакетов.

    Пакет
        *Пакет* - это структурированный набор файлов (PHP файлов, таблиц стилей,
        JavaScript-ов, картинок и т.д.), *предоставляющий* определенную функциональность
        (блог, форум и т.д.) и который легко может быть использован другими разработчиками.

    Front Controller
        A *Front Controller* is a short PHP that lives in the web directory
        of your project. Typically, *all* requests are handled by executing
        the same front controller, whose job is to bootstrap the Symfony
        application.