.. index::
   single: Routing; Scheme requirement

Как заставить маршрутизатор всегда использовать HTTPS или HTTP
===============================================

Иногда Вам необходимо установить защищенное соединение для ресурса и Вы хотите
быть уверенными, что доступ к этому ресурсу будет всегда осуществляться через
протокол HTTPS. Компонент маршрутизации позволяет Вам настроить принудительное
использование схемы URI с помощью параметра ``_scheme``:

.. configuration-block::

    .. code-block:: yaml

        secure:
            pattern:  /secure
            defaults: { _controller: AcmeDemoBundle:Main:secure }
            requirements:
                _scheme:  https

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="secure" pattern="/secure">
                <default key="_controller">AcmeDemoBundle:Main:secure</default>
                <requirement key="_scheme">https</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('secure', new Route('/secure', array(
            '_controller' => 'AcmeDemoBundle:Main:secure',
        ), array(
            '_scheme' => 'https',
        )));

        return $collection;

Приведенная выше конфигурация маршрута ``secure`` всегда будет использовать
протокол HTTPS.

Когда генерируется URL ``secure``, в случае, если текущая схема HTTP, то
Symfony автоматически сгенерирует абсолютный URL со схемой HTTPS.

.. code-block:: text

    # Если текущая схема - HTTPS
    {{ path('secure') }}
    # сгенерирует /secure

    # Если текущая схема - HTTP
    {{ path('secure') }}
    # сгенерирует https://example.com/secure

Правило также применяется и для входящих запросов. Если Вы попробуете
получить доступ к ресурсу ``/secure`` через HTTP, Symfony автоматически
перенаправит Вас на тот же URL, но с использованием схемы HTTPS.

Приведенные выше примеры используют протокол ``https`` для ``_scheme``, но
также Вы можете ограничить маршрут на использование только ``http`` протокола.

.. note::

    Компонент Безопасности предлагает другой способ ограничить использование
    только HTTP или HTTPs протокола посредством параметра ``requires_channel``.
    Этот альтернативный метод больше подходит для защиты "области" Вашего web-сайта
    (все URL в ``/admin``) или когда Вы хотите защитить все URL, объявленные в
    стороннем пакете.

.. toctree::
    :hidden:

    Translation source: 2011-10-30 [a4dbf4c91]
