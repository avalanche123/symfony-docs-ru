.. index::
   single: Security; Authorization

Авторизация
=============

Если пользователь идентифицирован, то вы можете ограничить доступ к ресурсам
приложения через правила контроля доступа. Авторизация в Symfony2 предусматривает
эту потребность, а также предоставляет стандартный и мощный способ рассуждений
может ли пользователь получать любой ресурс (URL, объектную модель, вызов
метода и т. д.), благодаря гибкому менеджеру разрешения доступа.

.. index::
   single: Security; Access Control

Определение правил контроля доступа для  ресурсов HTTP
------------------------------------------------

Авторизация выполняется для каждого запроса, основываясь на правилах контроля
доступа, указанных в конфигурации:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            access_control:
                - { path: /admin/.*, role: ROLE_ADMIN }
                - { path: /.*, role: IS_AUTHENTICATED_ANONYMOUSLY }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <access-control>
                <rule path="/admin/.*" role="ROLE_ADMIN" />
                <rule path="/.*" role="IS_AUTHENTICATED_ANONYMOUSLY" />
            </access-control>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'access_control' => array(
                array('path' => '/admin/.*', 'role' => 'ROLE_ADMIN'),
                array('path' => '/.*', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY'),
            ),
        ));

Для каждого входящего запроса Symfony2 ищет совпадающее правило контроля доступа
(выбирается первое совпадение) и выбрасывает
:class:`Symfony\\Component\Security\\Exception\\AccessDeniedException` если
пользователь не имеет необходимых ролей или
:class:`Symfony\\Component\Security\\Exception\\AuthenticationCredentialsNotFoundException`
если он ещё не авторизован.

В примере выше мы подбирали запросы, основываясь на их пути, но есть и другие
способы, о которых вы узнаете в следующем разделе.

.. tip::

    Symfony2 automatically adds a special role based on the anonymous flag:
    ``IS_AUTHENTICATED_ANONYMOUSLY`` for anonymous users and
    ``IS_AUTHENTICATED_FULLY`` for all others.

Соотвествие запросу
------------------

Правила контроля доступа могут соотвествовать запросу различными способами:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            access_control:
                # match the path info
                - { path: /admin/.*, role: ROLE_ADMIN }

                # match the controller class name
                - { controller: .*\\.*Bundle\\Admin\\.*, role: ROLE_ADMIN }

                # match any request attribute
                -
                    attributes:
                        - { key: _controller, pattern: .*\\.*Bundle\\Admin\\.* }
                    role: ROLE_ADMIN

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <access-control>
                <!-- match the path info -->
                <rule path="/admin/.*" role="ROLE_ADMIN" />

                <!-- match the controller class name -->
                <rule controller=".*\\.*Bundle\\Admin\\.*" role="ROLE_ADMIN" />

                <!-- match any request attribute -->
                <rule role="ROLE_ADMIN">
                    <attribute key="_controller" pattern=".*\\.*Bundle\\Admin\\.*" />
                </rule>
            </access-control>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'access_control' => array(
                // match the path info
                array('path' => '/admin/.*', 'role' => 'ROLE_ADMIN'),

                // match the controller class name
                array('controller' => '.*\\.*Bundle\\Admin\\.*', 'role' => 'ROLE_ADMIN'),

                // match any request attribute
                array(
                    'attributes' => array(
                        array('key' => '_controller', 'pattern' => '.*\\.*Bundle\\Admin\\.*'),
                    ),
                    'role' => 'ROLE_ADMIN',
                ),
            ),
        ));

.. index::
   single: Security; HTTPS

Принудительный HTTP или HTTPS
-----------------------

Помимо ролей также можно принудить части вашего web сайта использовать HTTP или
HTTPS:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            access_control:
                - { path: /admin/.*, role: ROLE_ADMIN, requires_channel: https }
                - { path: /.*, requires_channel: http }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <access-control>
                <rule path="/admin/.*" role="ROLE_ADMIN" requires-channel="https" />
                <rule path="/.*" requires-channel="http" />
            </access-control>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'access_control' => array(
                array('path' => '/admin/.*', 'role' => 'ROLE_ADMIN', 'requires_channel' => 'https'),
                array('path' => '/.*', 'requires_channel' => 'http'),
            ),
        ));

Если ``requires-channel`` не указан, то Symfony2 разрешает оба HTTP и HTTPS.
Но если настройка установлена на HTTP or HTTPS, то Symfony2 переадресует
пользователей при необходимости.

Контроль доступа в шаблонах
---------------------------

Если хотите проверить роль пользователя внутри шаблона, то используйте синтаксис:

.. configuration-block::

    .. code-block:: php

        <?php if ($view['security']->vote('ROLE_ADMIN')): ?>
            <a href="...">Delete</a>
        <?php endif ?>

    .. code-block:: jinja

        {% ifrole "ROLE_ADMIN" %}
            <a href="...">Delete</a>
        {% endifrole %}

.. note::

    Если вам нужен доступ к пользователю из шаблона, то вам необходимо явно
    передавать его.
