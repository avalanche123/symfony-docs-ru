.. index::
   single: Security

Security
========

В состав Symfony2 входит встроенный слой безопасности. Он защищает ваше приложение путем обеспечения механизмов аутентификации и авторизации.

*Аутентификация* обеспечивает, что пользователь действительно тот, за кого он себя выдает. *Авторизация* связана с процессом решения, может ли пользователь выполнить действие или нет (авторизация проходит после аутентификации).

Этот документ является быстрым обзором этих концепций, но настоящая мощь содержится в следующих трех частях: :doc:`Users </guides/security/users>`,
:doc:`Authentication </guides/security/authentication>`, и
:doc:`Authorization </guides/security/authorization>`.

.. index::
   pair: Security; Configuration

Configuration
-------------

Для большинства случаев, безопасность в Symfony2 может быть легко сконфигурирована в вашем главном конфигурационном файле; вот типичная конфигурация:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security.config:
            providers:
                main:
                    password_encoder: sha1
                    users:
                        foo: { password: 0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33, roles: ROLE_USER }

            firewalls:
                main:
                    pattern:    /.*
                    http-basic: true
                    logout:     true

            access_control:
                - { path: /.*, role: ROLE_USER }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <security:config>
            <security:provider>
                <security:password-encoder hash="sha1" />
                <security:user name="foo" password="0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33" roles="ROLE_USER" />
            </security:provider>

            <security:firewall pattern="/.*">
                <security:http-basic />
                <security:logout />
            </security:firewall>

            <security:access-control>
                <security:rule path="/.*" role="ROLE_USER" />
            </security:access-control>
        </security:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('security', 'config', array(
            'provider' => array(
                'main' => array('password_encoder' => 'sha1', 'users' => array(
                    'foo' => array('password' => '0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33', 'roles' => 'ROLE_USER'),
                )),
            ),
            'firewalls' => array(
                'main' => array('pattern' => '/.*', 'http-basic' => true, 'logout' => true),
            ),
            'access_control' => array(
                array('path' => '/.*', 'role' => 'ROLE_USER'),
            ),
        ));

Часто, предпочтительнее вынести всю конфигурацию касающуюся безопасности во внешний файл. Если вы используете XML, внешний файл может использовать пространство имен безопасности как значение по умолчанию, чтобы сделать его более читабельным:

.. code-block:: xml

        <srv:container xmlns="http://www.symfony-project.org/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://www.symfony-project.org/schema/dic/services"
            xsi:schemaLocation="http://www.symfony-project.org/schema/dic/services http://www.symfony-project.org/schema/dic/services/services-1.0.xsd">

            <config>
                <provider>
                    <password-encoder hash="sha1" />
                    <user name="foo" password="0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33" roles="ROLE_USER" />
                </provider>

                <firewall pattern="/.*">
                    <http-basic />
                    <logout />
                </firewall>

                <access-control>
                    <rule path="/.*" role="ROLE_USER" />
                </access-control>
            </config>
        </srv:container>

.. note::

    Во всех примерах документации предполагается, что вы используете внешний файл со значением пространства имен безопасности по умолчанию как сказано выше.

Как вы можете видеть, конфигурация состоит из трех секций:

* *provider*: Поставщик знает как создавать пользователей;

* *firewall*: Брандмауэр определяет механизмы аутентификации для приложения в целом или его части;

* *access-control*: Правила контроля доступа для защищенных частей вашего приложения вместе с ролями.

Подводя итоги рабочего процесса, брандмауэр проводит аутентификацию пользователя на основе установленных правил, пользователи создаются посредством поставщика, а контроль доступа контролирует доступ к ресурсам.

Аутентификация
--------------

В Symfony2 есть поддержка различных внешних механизмов аутентификации, которые могут быть легко добавлены при надобности; главными из них являются:

* HTTP Basic;
* HTTP Digest;
* аутентификация, базирующаяся на форме;
* сертификаты X.509.

Здесь показано, как вы можете защитить ваше приложение при помощи базовой HTTP аутентификации:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http-basic: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <http-basic />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main' => array('http-basic' => true),
            ),
        ));

Можно определить несколько брандмауэров если вам необходимо использование различных механизмов аутентификаци в различных частях приложения:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                backend:
                    pattern: /admin/.*
                    http-basic: true
                public:
                    pattern:  /.*
                    security: false

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall pattern="/admin/.*">
                <http-basic />
            </firewall>

            <firewall pattern="/.*" security="false" />
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'backend' => array('pattern' => '/admin/.*', 'http-basic' => true),
                'public'  => array('pattern' => '/.*', 'security' => false),
            ),
        ));

.. tip::

    Проще всего использовать базовую HTTP аутентификацию, но прочитайте часть :doc:`Authentication
    </guides/security/authentication>` для того чтобы узнать, как настраивать другие механизмы аутентификации, как настраивать аутентификацию без состояний, как вы можете имитировать другого пользователя, как включить https, и многое другое.

Users
-----

Во время аутентификации, Symfony2 опрашивает поставщика пользователей для создания объекта пользователя, отвечающего клиентскому запросу (с помощью учетных данных, как имя пользователя и пароль).
Для быстрого старта, вы можете определить поставщика "в памяти" прямо в конфигурации:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            providers:
                main:
                    users:
                        foo: { password: foo }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider>
                <user name="foo" password="foo" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'provider' => array(
                'main' => array('users' => array(
                    'foo' => array('password' => 'foo'),
                )),
            ),
        ));

Конфигурация сверху определяет пользователя 'foo' с паролем 'foo'. После аутентификации, вы можете получить доступ к аутентифицированному пользователю через безопасный контекст (пользователь является экземпляром класса :class:`Symfony\\Component\\Security\\User\\User`)::

    $user = $container->get('security.context')->getUser();

.. tip::

    Использование поставщика "в памяти" - это отличный вариант легко защитить серверную часть вашего персонального сайта, создать прототип, или создать макет для тестов. Прочитайте часть :doc:`Users </guides/security/users>` для того чтобы изучить, как избежать ненадежных паролей, как использовать Doctrine Entity в качестве пользовательского поставщика, как определить несколько поставщиков, и многое другое.

Authorization
-------------

Авторизация является необязательной, но позволяет мощно ограничивать доступ к ресурсам вашего приложения на основе пользовательских ролей:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            providers:
                main:
                    users:
                        foo: { password: foo, roles: ['ROLE_USER', 'ROLE_ADMIN'] }
            access_control:
                - { path: /.*, role: ROLE_USER }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider>
                <user name="foo" password="foo" roles="ROLE_USER,ROLE_ADMIN" />
            </provider>

            <access-control>
                <rule path="/.*" role="ROLE_USER" />
            </access-control>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'provider' => array(
                'main' => array('users' => array(
                    'foo' => array('password' => 'foo', 'roles' => array('ROLE_USER', 'ROLE_ADMIN')),
                )),
            ),

            'access_control' => array(
                array('path' => '/.*', 'role' => 'ROLE_USER'),
            ),
        ));

Конфигурация сверху определяет пользователя 'foo' с ролями 'ROLE_USER' и 'ROLE_ADMIN' и она ограничивает доступ к приложению в целом для пользователей с ролью 'ROLE_USER'.

.. tip::

    Прочитайте часть :doc:`Authorization </guides/security/authorization>` для того, чтобы узнать, как определять иерархию ролей, как настроить ваш шаблон базируясь на ролях, как определить правила контроля доступа базируясь на атрибутах запроса, и многое другое.
