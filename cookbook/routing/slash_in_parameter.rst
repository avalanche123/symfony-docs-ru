.. index::
   single: Routing; Allow / in route parameter

Как разрешить символ "/" в параметре маршрута
=================================================

Иногда у Вас возникает необходимость иметь в параметрах URL символ слеш
``/``. Например, рассмотрим классический маршрут ``/hello/{name}``.
По умолчанию, ``/hello/Fabien`` будет соответствовать этому маршруту, но не
``/hello/Fabien/Kris``. Так получается из-за того, что Symfony использует
этот символ в качестве разделителя между частями маршрута.

В этом руководстве описывается, как Вы можете настроить маршрут так, чтобы
``/hello/Fabien/Kris`` соответствовал шаблону ``/hello/{name}``, и где
``{name}`` был бы равен ``Fabien/Kris``.

Настройка Маршрута
-------------------

По умолчанию система маршрутизации Symfony принимает только те параметры,
которые соответствуют регулярному выражению: ``[^/]+``. Оно соответствует
всем символам, за исключением символа слеш ``/``.

Вам необходимо явно разрешить слеш ``/`` в параметрах. Для этого укажите
более "либеральное" регулярное выражение.

.. configuration-block::

    .. code-block:: yaml

        _hello:
            pattern: /hello/{name}
            defaults: { _controller: AcmeDemoBundle:Demo:hello }
            requirements:
                name: ".+"

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="_hello" pattern="/hello/{name}">
                <default key="_controller">AcmeDemoBundle:Demo:hello</default>
                <requirement key="name">.+</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('_hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeDemoBundle:Demo:hello',
        ), array(
            'name' => '.+',
        )));

        return $collection;

    .. code-block:: php-annotations

        use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

        class DemoController
        {
            /**
             * @Route("/hello/{name}", name="_hello", requirements={"name" = ".+"})
             */
            public function helloAction($name)
            {
                // ...
            }
        }

Вот и все! Сейчас параметр ``{name}`` может содержать символ ``/``.

.. toctree::
    :hidden:

    Translation source: 2012-02-10 [d4cee24]