Рецепты шаблонов
================

.. _twig_extension_tag:

Включение пользовательских расширений Twig
-------------------------------

Чтобы включить расширение Twig добавьте его как постоянную службу в одну из
ваших конфигураций и привяжите через тег ``twig.extension``:

.. configuration-block::

    .. code-block:: yaml

        services:
            twig.extension.your_extension_name:
                class: Fully\Qualified\Extension\Class\Name
                tags:
                    - { name: twig.extension }

    .. code-block:: xml

        <service id="twig.extension.your_extension_name" class="Fully\Qualified\Extension\Class\Name">
            <tag name="twig.extension" />
        </service>

    .. code-block:: php

        $container
            ->register('twig.extension.your_extension_name', 'Fully\Qualified\Extension\Class\Name')
            ->addTag('twig.extension')
        ;

.. _templating_renderer_tag:

Включение пользовательских визуализаторов шаблона
----------------------------------

Чтобы включить пользовательский визуализатор шаблона добавьте его как постоянную
службу в одну из ваших конфигураций и привяжите через тег ``templating.renderer``
и установите атрибут ``alias`` (визуализатор будет известен через алиас в имени
шаблона):

.. configuration-block::

    .. code-block:: yaml

        services:
            templating.renderer.your_renderer_name:
                class: Fully\Qualified\Renderer\Class\Name
                tags:
                    - { name: templating.renderer, alias: alias_name }

    .. code-block:: xml

        <service id="templating.renderer.your_renderer_name" class="Fully\Qualified\Renderer\Class\Name">
            <tag name="templating.renderer" alias="alias_name" />
        </service>

    .. code-block:: php

        $container
            ->register('templating.renderer.your_renderer_name', 'Fully\Qualified\Renderer\Class\Name')
            ->addTag('templating.renderer', array('alias' => 'alias_name'))
        ;

.. _templating_helper_tag:

Включение пользовательских хелперов для шаблонов PHP
------------------------------------

ЧТобы включить пользовательский хелпер для шаблона добавьте его как постоянную
службу в одну из ваших конфигураций и привяжите через тег ``templating.helper``
и установите атрибут ``alias`` (хелпер будет доступен через него в шаблонах):

.. configuration-block::

    .. code-block:: yaml

        services:
            templating.helper.your_helper_name:
                class: Fully\Qualified\Helper\Class\Name
                tags:
                    - { name: templating.helper, alias: alias_name }

    .. code-block:: xml

        <service id="templating.helper.your_helper_name" class="Fully\Qualified\Helper\Class\Name">
            <tag name="templating.helper" alias="alias_name" />
        </service>

    .. code-block:: php

        $container
            ->register('templating.helper.your_helper_name', 'Fully\Qualified\Helper\Class\Name')
            ->addTag('templating.helper', array('alias' => 'alias_name'))
        ;
