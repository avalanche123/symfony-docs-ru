.. index::
   single: Security; Users

Users
=====

Security only makes sense because your application is accessed by clients you
cannot trust. A client can be a human behind a browser, but also a device, a
web service, or even a bot.

Безопасность имеет сенс только в случае, когда с вашим приложением работают пользователи, которым вы не можете доверять. Пользователем может быть человек, работающий через браузер, некоторое устройство, web сервис, или же это бот.

Определение пользователей
-------------------------

Во время аутентификации, Symfony2 пытается найти соответствующего пользователя сравнивая клиентские учетные данные (чаще всего это имя пользователя и пароль). Та как Symfony2 не строит никаких догадок по поводу PHP представления клиента/пользователя, приложение должно определить класс пользователя и подключить его к Symfony2 через класс поставщика пользователей.

UserProviderInterface
~~~~~~~~~~~~~~~~~~~~~

Провайдер пользователей должен реализовывать интерфейс :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`::

    interface UserProviderInterface
    {
         function loadUserByUsername($username);
    }

Метод ``loadUserByUsername()`` получает имя пользователя и должен возвратить объект пользователя User. Если пользователь не может быть найден, он должен выбросить исключение :class:`Symfony\\Component\\Security\\Exception\\UsernameNotFoundException`.

.. примечание::

    Чаще всего, вам нет надобности определять провайдер пользователя самим, так как в состав Symfony2 входят наиболее общие варианты. Смотрите следующую секцию для более детальной информации.

AccountInterface
~~~~~~~~~~~~~~~~

Провайдер пользователя должен возвращать объекты, реализующие интерфейс :class:`Symfony\\Component\\Security\\User\\AccountInterface`::

    interface AccountInterface
    {
        function __toString();
        function getRoles();
        function getPassword();
        function getSalt();
        function getUsername();
        function eraseCredentials();
    }

* ``__toString()``: Возвращает строковое представление пользователя;
* ``getRoles()``: Возвращает роли, сопоставленные пользователю;
* ``getPassword()``: Возвращает пароль, используемое для аутентификации пользователя;
* ``getSalt()``: Возвращает соль;
* ``getUsername()``: Возвращает имя пользователя, используемое для аутентификации пользователя;
* ``eraseCredentials()``: Удаляет чувствительные данные из объекта пользователя.

Кодирование Паролей
~~~~~~~~~~~~~~~~~~~

Вместо хранения паролей в чистом виде, вы можете закодировать их. Для этого вам следует использовать объект :class:`Symfony\\Component\\Security\\Encoder\\PasswordEncoderInterface`::

    interface PasswordEncoderInterface
    {
        function encodePassword($raw, $salt);
        function isPasswordValid($encoded, $raw, $salt);
    }

.. note::

    Во время аутентификации, Symfony2 будет использовать метод ``isPasswordValid()``
    для проверки пароля пользователя; прочитайте следующую секцию, чтобы узнать, как уведомить ваш провайдер
    аутентификации использовать кодирование.

В большинстве случаев, используйте
:class:`Symfony\\Component\\Security\\Encoder\\MessageDigestPasswordEncoder`::

    $user = new User();

    $encoder = new MessageDigestPasswordEncoder('sha1');
    $password = $encoder->encodePassword('MyPass', $user->getSalt());
    $user->setPassword($password);

Когда кодируете ваши пароли, очень хорошо будет определить уникальную "соль" для каждого пользователя (метод ``getSalt()`` может возвращать первичный ключ если пользователи хранятся, например, в базе данных).

AdvancedAccountInterface
~~~~~~~~~~~~~~~~~~~~~~~~

Перед и после аутентификацией, Symfony2 может проверить различные флаги для пользователя.
Если ваш класс пользователя реализует интерфейс :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface` вместо :class:`Symfony\\Component\\Security\\User\\AccountInterface`, Symfony2 сделает сопутствующие проверки автоматически::

    interface AdvancedAccountInterface extends AccountInterface
    {
        function isAccountNonExpired();
        function isAccountNonLocked();
        function isCredentialsNonExpired();
        function isEnabled();
    }

* ``isAccountNonExpired()``: Возвращает ``true`` если аккаунт пользователя истек;
* ``isAccountNonLocked()``: Возвращает ``true`` когда пользователь заблокирован;
* ``isCredentialsNonExpired()``: Возвращает ``true`` пользовательские учетные данные (пароль) устарели;
* ``isEnabled()``: Возвращает ``true`` когда пользователь включен.

.. note::

    Интерфейс :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface`
    зависит от объекта
    :class:`Symfony\\Component\\Security\\User\\AccountCheckerInterface`
    для того чтобы выполнить пре-аутентификационные и пост-аутентификационные проверки.

Определение Провайдера
----------------------

Как мы видели в предыдущей секции, провайдер реализует интерфейс :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`. В состав Symfony2 входит провайдер для пользователей "в памяти", Doctrine Entity, и базовый класс для любого DAO провайдера, который вы, возможно, захотите создать.

Провайдер "в памяти"
~~~~~~~~~~~~~~~~~~~~

Провайдер "в памяти" это отличный провайдер для защиты серверной части вашего персонального web сайта или прототипа. Это также лучший провайдер когда вы пишите unit тесты:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            providers:
                main:
                    users:
                        foo: { password: foo, roles: ROLE_USER }
                        bar: { password: bar, roles: [ROLE_USER, ROLE_ADMIN] }
                encoded:
                    password_encoder: sha1
                    users:
                        foo: { password: 0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33, roles: ROLE_USER }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="main">
                <user name="foo" password="foo" roles="ROLE_USER" />
                <user name="bar" password="bar" roles="ROLE_USER,ROLE_ADMIN" />
            </provider>

            <provider name="encoded">
                <password-encoder>sha1</password-encoder>
                <user name="foo" password="0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33" roles="ROLE_USER" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'providers' => array(
                'main' => array('users' => array(
                    'foo' => array('password' => 'foo', 'roles' => 'ROLE_USER'),
                    'bar' => array('password' => 'bar', 'roles' => array('ROLE_USER', 'ROLE_ADMIN')),
                )),
                'encoded' => array('password_encoder' => 'sha1', 'users' => array(
                    'foo' => array('password' => '0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33', 'roles' => 'ROLE_USER'),
                )),
            ),
        ));

Конфигурация сверху определяет два провайдера "в памяти". Как вы можете видеть, второй из них использует 'sha1' для кодировки пользовательских паролей.

Провайдер Doctrine Entity
~~~~~~~~~~~~~~~~~~~~~~~~~

В большинстве случаев, пользователи описываются через Doctrine Entity::

    /**
     * @Entity
     */
    class User implements AccountInterface
    {
        // ...
    }

В этом случае, вы можете использовать провайдер Doctrine по умолчанию без его самостоятельного создания:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            providers:
                main:
                    password_encoder: sha1
                    entity: { class: SecurityBundle:User, property: username }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="main">
                <password-encoder>sha1</password-encoder>
                <entity class="SecurityBundle:User" property="username" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'providers' => array(
                'main' => array(
                    'password_encoder' => 'sha1',
                    'entity' => array('class' => 'SecurityBundle:User', 'property' => 'username'),
                ),
            ),
        ));

Вхождение ``entity`` конфигурирует класс Entity для использования для пользователя, а ``property`` - это название колонки PHP, где хранится имя пользователя.

Если получение пользователя сложнее, чем просто вызов ``findOneBy()``,
удалите установку ``property`` и сделайте чтобы класс Entity Repository реализовывал интерфейс :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`::

    /**
     * @Entity(repositoryClass="SecurityBundle:UserRepository")
     */
    class User implements AccountInterface
    {
        // ...
    }

    class UserRepository extends EntityRepository implements UserProviderInterface
    {
        public function loadUserByUsername($username)
        {
            // do whatever you need to retrieve the user from the database
            // code below is the implementation used when using the property setting

            return $this->findOneBy(array('username' => $username));
        }
    }

.. tip::

    If you use the
    :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface`
    interface, don't check the various flags (locked, expired, enabled, ...)
    when retrieving the user from the database as this will be managed by the
    authentication system automatically (and proper exceptions will be thrown
    if needed). If you have special flags, override the default
    :class:`Symfony\\Component\\Security\\User\\AccountCheckerInterface`
    implementation.

.. index::
   single: Security; Doctrine Document Provider
   single: Doctrine; Doctrine Document Provider

Doctrine Document Provider
~~~~~~~~~~~~~~~~~~~~~~~~~~

Most of the time, users are described by a Doctrine Document::

    /**
     * @Document
     */
    class User implements AccountInterface
    {
        // ...
    }

In such a case, you can use the default Doctrine provider without creating one
yourself:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            providers:
                main:
                    password_encoder: sha1
                    document: { class: SecurityBundle:User, property: username }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="main">
                <password-encoder>sha1</password-encoder>
                <document class="SecurityBundle:User" property="username" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'providers' => array(
                'main' => array(
                    'password_encoder' => 'sha1',
                    'document' => array('class' => 'SecurityBundle:User', 'property' => 'username'),
                ),
            ),
        ));

The ``document`` entry configures the Document class to use for the user, and
``property`` the PHP column name where the username is stored.

If retrieving the user is more complex than a simple ``findOneBy()`` call,
remove the ``property`` setting and make your Document Repository class
implement :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`::

    /**
     * @Document(repositoryClass="SecurityBundle:UserRepository")
     */
    class User implements AccountInterface
    {
        // ...
    }

    class UserRepository extends DocumentRepository implements UserProviderInterface
    {
        public function loadUserByUsername($username)
        {
            // do whatever you need to retrieve the user from the database
            // code below is the implementation used when using the property setting

            return $this->findOneBy(array('username' => $username));
        }
    }

.. tip::

    Если вы используете интерфейс 
    :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface`
    не проверяйте различные флаги (закрыт, устарел, включен, ...)
    когда получаете пользователя с базы данных так как проверки будут сделаны системой аутентификации автоматически (и соответствующие исключения будут выброшены при необходимости). Если у вас установлены специальные флаги, переопределите реализацию интерфейса
    :class:`Symfony\\Component\\Security\\User\\AccountCheckerInterface`
    по умолчанию.

Извлечение Пользователя
-----------------------

После аутентификации, пользователь доступен через безопасный контекст::

    $user = $container->get('security.context')->getUser();

You can also check if the user is authenticated with the ``isAuthenticated()``
method::

    $container->get('security.context')->isAuthenticated();

.. tip::

    Be aware that anonymous users are considered authenticated. If you want to
    check if a user is "fully authenticated" (non-anonymous), you need to check
    if the user has the special ``IS_AUTHENTICATED_FULLY`` role (or check that
    the user has not the ``IS_AUTHENTICATED_ANONYMOUSLY`` role).

.. index::
   single: Security; Roles

Roles
-----

Роли
----

У пользователя может быть столько ролей, сколько необходимо. Роли обычно определяются как строки,
но они могут быть любым объектом, реализующим интерфейс :class:`Symfony\\Component\\Security\\Role\\RoleInterface` (роли во внутреннем представлении всегда объекты). Роли, определяемые строкой, должны начинаться префиксом ``ROLE_``, чтобы автоматически обрабатываться Symfony2.

Роли используются менеджером принятия решений по контролю доступа для защиты ресурсов. Прочитайте секцию :doc:`Authorization </guides/security/authorization>`, чтобы узнать больше о контроле доступа, ролях и голосующих.

.. примечание::

    Если вы определили ваши собственные роли при помощи внешнего класса ролей, не используйте префикс ``ROLE_``.

Иерархические Роли
~~~~~~~~~~~~~~~~~~

Вместо того, чтобы ассоциировать пользователям множество ролей, вы можете определить правила наследования ролей путем создания иерархии ролей:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            role_hierarchy:
                ROLE_ADMIN:       ROLE_USER
                ROLE_SUPER_ADMIN: [ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <role-hierarchy>
                <role id="ROLE_ADMIN">ROLE_USER</role>
                <role id="ROLE_SUPER_ADMIN">ROLE_USER,ROLE_ADMIN,ROLE_ALLOWED_TO_SWITCH</role>
            </role-hierarchy>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'role_hierarchy' => array(
                'ROLE_ADMIN'       => 'ROLE_USER',
                'ROLE_SUPER_ADMIN' => array('ROLE_USER,ROLE_ADMIN', 'ROLE_ALLOWED_TO_SWITCH'),
            ),
        ));

В конфигурации сверху, у пользователи с ролью 'ROLE_ADMIN' также будет роль 'ROLE_USER'. Роль 'ROLE_SUPER_ADMIN' обладает множественным наследованием.