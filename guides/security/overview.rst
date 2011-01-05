.. index::
   single: Security

Безопасность
========

Symfony2 поставляется со встроенным слоем безопасности. Он защищает ваше
приложение, обеспечивая идентификацию и авторизацию.

*Идентификация* удостоверяет что пользователь действительно тот, на кого он
претендует. *Авторизация* связана с принятием решения, может или нет пользователь
выполнить действие (авторизация идёт после идентификации).

Этот документ является кратким обзором этих концепций, но настоящая мощь
содержится в следующих трёх частях: :doc:`Пользователи </guides/security/users>`,
:doc:`Идентификация </guides/security/authentication>` и
:doc:`Авторизация </guides/security/authorization>`.

.. index::
   pair: Security; Configuration

Конфигурация
-------------

В большинстве случаев безопасность в Symfony2 может быть легко настроена через
ваш главный конфигурационный файл; вот типичная конфигурация:

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

Предпочтительнее вынести всю конфигурацию, касающуюся безопасности, во внешний
файл. Если вы используете XML, внешний файл может использовать пространство имён
безопасности как значение по умолчанию, чтобы сделать его более читабельным:

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

    Все примеры документации предполагают, что вы используете внешний файл с
    пространством имён безопасности по умолчанию, как сказано выше.

Как вы можете видеть, конфигурация состоит из трех разделов:

* *provider*: Провайдер знает как создавать пользователей;

* *firewall*: Фаервол определяет механизмы идентификации для приложения в целом
  или хотя бы его части;

* *access-control*: Правила контроля доступа защищают части вашего приложения
  с помощью ролей.

Подводя итоги рабочего процесса, фаервол идентифицирует клиента на основе
представленных полномочий, пользователь создаётся провайдером, а контроль
доступа уполномачивает на доступ к ресурсам.

Идентификация
--------------

В Symfony2 есть поддержка различных механизмов идентификации прямо из корбки,
а также многие могут быть легко добавлены при необходимости; главными из них
являются:

* HTTP Basic;
* HTTP Digest;
* идентификация через форму;
* сертификаты X.509.

Здесь показано как вы можете защитить приложение при помощи базовой HTTP
идентификации:

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

Можно определить несколько фаерволов если вам необходимо использование различных
механизмов идентификаци в различных частях приложения:

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

    Проще всего использовать базовую HTTP идентификацию, но прочтите документ
    :doc:`Идентификация </guides/security/authentication>` чтобы научиться
    настраивать другие механизмы идентификации, настраивать идентификацию без
    состояний, имитировать другого пользователя, навязать https и многое другое.

Пользователи
-----

Во время идентификации, Symfony2 опрашивает провайдера пользователя для создания
объекта пользователя, отвечающего клиентскому запросу (через такие полномочия
как имя пользователя и пароль). Чтобы быстрее начать, вы можете определить
провайдера "в памяти" прямо в конфигурации:

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

Эта конфигурация устанавливает пользователя 'foo' с паролем 'foo'. После
идентификации, вы можете получить доступ к идентифицированному пользователю
через безопасный контекст (пользователь является экземпляром класса :class:`Symfony\\Component\\Security\\User\\User`)::

    $user = $container->get('security.context')->getUser();

.. tip::

    Использование провайдера "в памяти" это отличный способ легко защитить
    бэкэнд вашего персонального сайта, создать прототип или приспособления для
    тестов. Прочитайте документ :doc:`Пользователи </guides/security/users>` чтобы
    изучить как избежать паролей, которые будут в чистом виде, как использовать
    Doctrine Entity в качестве провайдера пользователя, как определить
    нескольких поставщиков и многое другое.

Авторизация
-------------

Авторизация необязательна, но предоставляет мощный способ ограничивать доступ к
ресурсам вашего приложения на основе пользовательских ролей:

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

Эта конфигурация устанавливает пользователя 'foo' с ролями 'ROLE_USER' и
'ROLE_ADMIN' и ограничивает доступ ко всему приложению для пользователей,
имеющих роль 'ROLE_USER'.

.. tip::

    Прочитайте документ :doc:`Авторизация </guides/security/authorization>` чтобы
    узнать как устанавливать иерархию роли, как настроить шаблон, основываясь на
    ролях, как определить правила контроля доступа, основываясь на атрибутах
    запроса и многое другое.
