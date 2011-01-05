.. index::
   single: Security; Users

Пользователи
=====

Безопасность имеет смысл только в случае когда ваше приложение доступно клиентам,
которым вы не можете доверять. Клиентом может быть как человек, работающий через
браузер, так и некоторое устройство, web служба или даже бот.

Определение пользователей
-------------------------

Во время идентификации, Symfony2 пытается найти пользователя, соотвествующего
клиентским полномочиям (чаще всего это имя пользователя и пароль). Так как
Symfony2 не делает предположений по поводу PHP представления клиента/пользователя,
то приложение должно определить класс пользователя и подключить его к Symfony2
через класс провайдера пользователя.

.. index::
   single: Security; UserProviderInterface

UserProviderInterface
~~~~~~~~~~~~~~~~~~~~~

Провайдер пользователя должен реализовывать интерфейс :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`::

    interface UserProviderInterface
    {
         function loadUserByUsername($username);
    }

Метод ``loadUserByUsername()`` получает имя пользователя и должен возвратить
объект User. Если пользователь не может быть найден, он выбрасывает исключение
:class:`Symfony\\Component\\Security\\Exception\\UsernameNotFoundException`.

.. tip::

    Чаще всего вам нет надобности определять провайдер пользователя самим,
    так как Symfony2 поставляется с наиболее общими их вариантами. Обратитесь к
    следующему разделу за дополнительной информацией.

.. index::
   single: Security; AccountInterface

AccountInterface
~~~~~~~~~~~~~~~~

Провайдер пользователя должен возвращать объекты, реализующие интерфейс
:class:`Symfony\\Component\\Security\\User\\AccountInterface`::

    interface AccountInterface
    {
        function __toString();
        function getRoles();
        function getPassword();
        function getSalt();
        function getUsername();
        function eraseCredentials();
    }

* ``__toString()``: Возвращает строковое представление для пользователя;
* ``getRoles()``: Возвращает роли, предоставленные пользователю;
* ``getPassword()``: Возвращает пароль, используемый для идентификации пользователя;
* ``getSalt()``: Возвращает соль;
* ``getUsername()``: Возвращает имя пользователя, используемое для идентификации пользователя;
* ``eraseCredentials()``: Удаляет деликатные данные пользователя.

.. index::
   single: Security; Password encoding

Шифрование паролей
~~~~~~~~~~~~~~~~~~~

Вместо хранения паролей в чистом виде, вы можете зашифровать их. Если сделаете
это, то вам придётся использовать объект :class:`Symfony\\Component\\Security\\Encoder\\PasswordEncoderInterface`::

    interface PasswordEncoderInterface
    {
        function encodePassword($raw, $salt);
        function isPasswordValid($encoded, $raw, $salt);
    }

.. note::

    Во время идентификации Symfony2 будет использовать метод ``isPasswordValid()``
    для проверки пароля пользователя; прочтите следующий раздел чтобы узнать
    как уведомить ваш провайдер идентификации об использовании шифрования.

В большинстве случаев, используйте
:class:`Symfony\\Component\\Security\\Encoder\\MessageDigestPasswordEncoder`::

    $user = new User();

    $encoder = new MessageDigestPasswordEncoder('sha1');
    $password = $encoder->encodePassword('MyPass', $user->getSalt());
    $user->setPassword($password);

Когда шифруете ваши пароли, также лучше будет определить уникальную соль для
каждого пользователя (например, метод ``getSalt()`` может возвращать первичный
ключ если пользователи хранятся в базе данных).

.. index::
   single: Security; AdvancedAccountInterface

AdvancedAccountInterface
~~~~~~~~~~~~~~~~~~~~~~~~

Перед и после идентификации Symfony2 может проверить различные отметки у
пользователя. Если ваш класс пользователя реализует :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface`
вместо :class:`Symfony\\Component\\Security\\User\\AccountInterface`, то Symfony2
сделает сопутствующие проверки автоматически:

    interface AdvancedAccountInterface extends AccountInterface
    {
        function isAccountNonExpired();
        function isAccountNonLocked();
        function isCredentialsNonExpired();
        function isEnabled();
    }

* ``isAccountNonExpired()``: Возвращает ``true`` если аккаунт пользователя устарел;
* ``isAccountNonLocked()``: Возвращает ``true`` если пользователь заблокирован;
* ``isCredentialsNonExpired()``: Возвращает ``true`` если полномочия пользователя (пароль) устарели;
* ``isEnabled()``: Возвращает ``true`` если пользователь включён.

.. note::

    :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface`
    полагается на объект
    :class:`Symfony\\Component\\Security\\User\\AccountCheckerInterface`
    для выполнения пред- и после-идентификационных проверок.

.. index::
   single: Security; User Providers

Определение провайдера
----------------------

Как мы видели в предыдущем разделе, провайдер реализует :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`.
Symfony2 поставляется с провайдером для пользователей "в памяти",
Doctrine Entities, Doctrine Documents и определяет базовый класс для любого DAO
провайдера, который вы, возможно, захотите создать.

.. index::
   single: Security; In-memory user provider

Провайдер "в памяти"
~~~~~~~~~~~~~~~~~~~~

Провайдер "в памяти" это отличный провайдер для защиты бэкэнда вашего
персонального web сайта или прототипа, а также лучший провайдер когда пишутся
модульные тесты:

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

Предыдущая конфигурация определяет два провайдера "в памяти". Как вы можете
видеть второй из них использует 'sha1' для шифрования пользовательского пароля.

.. index::
   single: Security; Doctrine Entity Provider
   single: Doctrine; Doctrine Entity Provider

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

В этом случае можете использовать провайдер Doctrine по умолчанию, не создавая
его самостоятельного:

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

Запись ``entity`` конфигурирует класс Entity для использования на пользователе,
а ``property`` - это название колонки PHP, где хранится имя пользователя.

Если получение пользователя сложнее чем просто вызов ``findOneBy()``, то
удалите установку ``property`` и сделайте так чтобы класс Entity Repository
реализовывал :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`::

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

    Если вы используете интерфейс
    :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface`,
    не проверяйте различные установки (locked, expired, enabled и т. д.)
    когда получаете пользователя из базы данных т. к. это будет сделано системой
    идентификации автоматически (и правильные исключения будут выброшены
    если это необходимо). Если имеются специальные установки, переопределите
    первоначальную реализацию :class:`Symfony\\Component\\Security\\User\\AccountCheckerInterface`.

.. index::
   single: Security; Doctrine Document Provider
   single: Doctrine; Doctrine Document Provider

Провайдер Doctrine Document
~~~~~~~~~~~~~~~~~~~~~~~~~~

В большинстве случаев, пользователи описываются через Doctrine Document::

    /**
     * @Document
     */
    class User implements AccountInterface
    {
        // ...
    }

В этом случае можете использовать провайдер Doctrine по умолчанию, не создавая
его самостоятельного:

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

Запись ``document`` конфигурирует класс Document для использования на пользователе,
а ``property`` - это название колонки PHP, где хранится имя пользователя.

Если получение пользователя сложнее чем просто вызов ``findOneBy()``, то
удалите установку ``property`` и сделайте так чтобы класс Document Repository
реализовывал :class:`Symfony\\Component\\Security\\User\\UserProviderInterface`::

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
    :class:`Symfony\\Component\\Security\\User\\AdvancedAccountInterface`,
    не проверяйте различные установки (locked, expired, enabled и т. д.)
    когда получаете пользователя из базы данных т. к. это будет сделано системой
    идентификации автоматически (и правильные исключения будут выброшены
    если это необходимо). Если имеются специальные установки, переопределите
    первоначальную реализацию :class:`Symfony\\Component\\Security\\User\\AccountCheckerInterface`.

Получение пользователя
-------------------

После идентификации, пользователь доступен через безопасный контекст::

    $user = $container->get('security.context')->getUser();

Также можно проверить идентифицирован ли пользователь через метод
``isAuthenticated()``::

    $container->get('security.context')->isAuthenticated();

.. tip::

    Не забудьте что анонимные пользователи считаются идентифицированными. Если
    хотите узнать является ли пользователь "полностью идентифицированным"
    (не анонимным), необходимо проверить имеет ли он роль ``IS_AUTHENTICATED_FULLY``
    (или удостовериться что у него нет роли ``IS_AUTHENTICATED_ANONYMOUSLY``).

.. index::
   single: Security; Roles

Роли
-----

Пользователь может иметь столько ролей, сколько необходимо. Роли обычно
определяются как строки, но они могут быть любым объектом, реализующим
:class:`Symfony\\Component\\Security\\Role\\RoleInterface`
(во внутреннем представлении роли всегда объекты). Роли, определяемые строками,
должны начинаться с префикса ``ROLE_``, чтобы автоматически обрабатываться
Symfony2.

Роли используются менеджером разрешения доступа для защиты ресурсов. Прочитайте
документ :doc:`Авторизация </guides/security/authorization>`, чтобы узнать больше
о контроле доступа, ролях и голосующих.

.. tip::

    Если вы определили собственные роли при помощи внешнего класса Role,
    не используйте префикс ``ROLE_``.

.. index::
   single: Security; Roles (Hierarchical)

Иерархические роли
~~~~~~~~~~~~~~~~~~

Вместо того чтобы ассоциировать множество ролей с пользователями, вы можете
определить правила наследования ролей, путём создания их иерархии:

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
                'ROLE_SUPER_ADMIN' => array('ROLE_USER', 'ROLE_ADMIN', 'ROLE_ALLOWED_TO_SWITCH'),
            ),
        ));

В этой конфигурации пользователи с ролью 'ROLE_ADMIN' также будут иметь роль
'ROLE_USER', а роль 'ROLE_SUPER_ADMIN' обладает множественным наследованием.