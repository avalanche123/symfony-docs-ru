.. index::
   single: Security; Authentication

Идентификация
==============

Идентификация в Symfony2 регулируется системой Firewall. Она основана на
обработчиках, которые следят за безопасностью и перенаправляют пользователя
если его полномочия не доступны, не достаточны или попросту неправильны.

.. note::

    Firewall реализован через событие ``core.security``, которое вызывается
    сразу после события ``core.request``. Вся функциональность, о которой пойдёт
    речь в этой части, реализована через обработчики этого события.

.. index::
   single: Security; Firewall
  pair: Security; Configuration

Карта Firewall
----------------

Firewall может быть настроен для защиты всего приложения, также может использовать
различные стратегии идентификации для различных частей приложения.

Часто, web сайт открывает публичную часть для всех, при этом защищая панель
управления с помощью формы идентификации, и защищает публичные API/Web Службы
через базовую HTTP идентификацию:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                backend:
                    pattern:    /admin/.*
                    form-login: true
                    logout:     true
                api:
                    pattern:    /api/.*
                    http_basic: true
                    stateless:  true
                public:
                    pattern:    /.*
                    security:   false

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall pattern="/admin/.*">
                <form-login />
                <logout />
            </firewall>
            <firewall pattern="/api/.*" stateless="true">
                <http-basic />
            </firewall>
            <firewall pattern="/.*" security="false" />
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'backend' => array('pattern' => '/admin/.*', 'http_basic' => true, 'logout' => true),
                'api'     => array('pattern' => '/api/.*', 'http_basic' => true, 'stateless' => true),
                'public'  => array('pattern' => '/.*', 'security' => false),
            ),
        ));

Каждая конфигурация firewall-а активируется когда входящий запрос совпадёт с
регулярным выражением, определённым в настройке ``pattern``. Этот шаблон должен
совпадать с запрошенным путём
(``preg_match('#^'.PATTERN_VALUE.'$#', $request->getPathInfo())``.)

.. tip::

    Важен порядок определения правил для фаервола, т. к. Symfony2
    использует первую настройку, для которой паттерн соответствует запросу
    (т. о. сначала определите более конкретные конфигурации).

.. index::
   pair: Security; Configuration

Механизмы идентификации
-------------------------

Symfony2 поставляется с поддержкой следующих механизмов идентификации:

* HTTP Basic;
* HTTP Digest;
* Идентификация через форму;
* X.509 сертификаты;
* Анонимная идентификация.

Каждый механизм состоит из двух классов, выполняющих его работу: слушатель
и точка входа. *Слушатель* пытается идентифицировать запросы. Если пользователь
не идентифицирован или если слушатель обнаружит неправильные полномочия,
*точка входа* создаст ответ чтобы выдать ответную реакцию пользователю и
предоставить ему возможность войти в свои полномочия.

Вы можете настроить фаервол для использования более одного механизма идентификации:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                backend:
                    pattern:    /admin/.*
                    x509:       true
                    http_basic: true
                    form_login: true
                    logout:     true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall pattern="/admin/.*">
                <x509 />
                <http-basic />
                <form-login />
                <logout />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'backend' => array(
                    'pattern'    => '/admin/.*',
                    'x509'       => true,
                    'http_basic' => true,
                    'form_login' => true,
                    'logout'     => true,
                ),
            ),
        ));

Пользователь, проникающий в ресурсы ``/admin/``, должен предоставить либо
сертификат X.509, либо заголовок Authorization HTTP, а также может использовать
форму входа.

.. note::

    Когда пользователь не идентифицирован и установлено более одного механизма
    идентификации, Symfony2 автоматически определяет точку входа по умолчанию
    (в предыдущем примере, ею будет форма входа; но если пользователь отправит
    заголовок Authorization HTTP с неправильными полномочиями, Symfony2 будет
    использовать точку входа HTTP Basic).

.. note::

    Идентификация HTTP Basic применяется повсюду, но не безопасна. HTTP Digest
    более защищена, не очень везде применима на практике.

.. index::
   single: Security; HTTP Basic

HTTP Basic
~~~~~~~~~~

Настроить идентификацию HTTP basic очень просто:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http_basic: true

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
                'main' => array('http_basic' => true),
            ),
        ));

.. index::
   single: Security; HTTP Digest

HTTP Digest
~~~~~~~~~~~

Настроить идентификацию HTTP digest очень просто:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http_digest: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <http-digest />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main' => array('http_digest' => true),
            ),
        ));

.. caution::

    Используя HTTP Digest, храните пароли пользователей в открытом виде.

.. index::
   single: Security; Form based

Идентификация через форму
~~~~~~~~~~~~~~~~~~~~~~~~~

Идентификация через форму - наиболее используемый механизм в Web на сегодня:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    form_login: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main' => array('form_login' => true),
            ),
        ));

Если пользователь не идентифицирован, он перенаправляется на URL ``login_path``
(``/login`` по умолчанию).

Этот прослушиватель полагается на форму для взаимодействия с пользователем. Он
обрабатывает передачу формы автоматически, но не её отображение; т. о. вы должны
выполнить эту задачу самостоятельно:

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\Security\SecurityContext;

    class SecurityController extends Controller
    {
        public function loginAction()
        {
            // get the error if any (works with forward and redirect -- see below)
            if ($this->get('request')->attributes->has(SecurityContext::AUTHENTICATION_ERROR)) {
                $error = $this->get('request')->attributes->get(SecurityContext::AUTHENTICATION_ERROR);
            } else {
                $error = $this->get('request')->getSession()->get(SecurityContext::AUTHENTICATION_ERROR);
            }

            return $this->render('SecurityBundle:Security:login.php', array(
                // last username entered by the user
                'last_username' => $this->get('request')->getSession()->get(SecurityContext::LAST_USERNAME),
                'error'         => $error,
            ));
        }
    }

И соотвествующий шаблон:

.. configuration-block::

    .. code-block:: html+php

        <?php if ($error): ?>
            <div><?php echo $error ?></div>
        <?php endif; ?>

        <form action="<?php echo $view['router']->generate('_security_check') ?>" method="post">
            <label for="username">Username:</label>
            <input type="text" id="username" name="_username" value="<?php echo $last_username ?>" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            <input type="submit" name="login" />
        </form>

    .. code-block:: jinja

        {% if error %}
            <div>{{ error }}</div>
        {% endif %}

        <form action="{% path "_security_check" %}" method="post">
            <label for="username">Username:</label>
            <input type="text" id="username" name="_username" value="{{ last_username }}" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            <input type="submit" name="login" />
        </form>

Шаблон должен иметь поля ``_username`` и ``_password``, в форме URL для передачи
должен быть значением настройки ``check_path`` (``/login_check`` по умолчанию).

Наконец, добавьте URL маршруты для ``/login`` (значение ``login_path``) и
``/login_check`` (значение ``login_check``):

.. code-block:: xml

    <route id="_security_login" pattern="/login">
        <default key="_controller">SecurityBundle:Security:login</default>
    </route>

    <route id="_security_check" pattern="/login_check" />

После неудачной идентификации пользователь переадресовывается на страницу входа.
Вместо этого можно использовать перенаправление установив ``failure_forward`` в
значение ``true``. Также можно переадресовать или перенаправить на другую
страницу, если указать``failure_path``.

После удачной идентификации пользователь переадресовывается согласно следующему
алгоритму:

* Если ``always_use_default_target_path`` равно ``true`` (``false`` по умолчанию),
  переадресовывает пользователя на ``default_target_path`` (``/`` по умолчанию);

* Если запрос содержит параметр, названный ``_target_path`` (настраивается через
  ``target_path_parameter``), то переадресовывает пользователя на адрес, указанный
  в нём;

* Если целевой URL хранится в сессии (это делается автоматически когда
  пользователь переадресовывается на страницу входа), то переадресовывает
  пользователя на этот URL;

* Если ``use_referer`` равно ``true`` (``false`` по умолчанию), то переадресовывает
  пользователя на Referrer URL;

* Переадресовывает пользователя на ``default_target_path`` URL (``/`` по умолчанию).

.. note::

    Все URL-ы должны быть значениями path info или абсолютными URL-ами.

Первоначальные значения для всех настроек являются наиболее точными, но вот
пример конфигурации, показывающий как их можно переопределить:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    form_login:
                        check_path:                     /login_check
                        login_path:                     /login
                        failure_path:                   null
                        always_use_default_target_path: false
                        default_target_path:            /
                        target_path_parameter:          _target_path
                        use_referer:                    false

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <form-login
                    check_path="/login_check"
                    login_path="/login"
                    failure_path="null"
                    always_use_default_target_path="false"
                    default_target_path="/"
                    target_path_parameter="_target_path"
                    use_referer="false"
                />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main' => array('form_login' => array(
                    'check_path'                     => '/login_check',
                    'login_path'                     => '/login',
                    'failure_path'                   => null,
                    'always_use_default_target_path' => false,
                    'default_target_path'            => '/',
                    'target_path_parameter'          => _target_path,
                    'use_referer'                    => false,
                )),
            ),
        ));

.. index::
   single: Security; X.509 certificates

X.509 сертификаты
~~~~~~~~~~~~~~~~~~

X.509 сертификаты отличный способ идентификации пользователей если все они вам
известны:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    x509: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <x509 />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main' => array('x509' => true),
            ),
        ));

Так как Symfony2 не проверяет сертификаты самостоятельно, потому что он не
сможет усилить пароль, вам придётся правильно настроить web сервер перед
включением это механизма идентификации. Вот простой, но рабочий пример
конфигурации для Apache:

.. code-block:: xml

    <VirtualHost *:443>
        ServerName intranet.example.com:443

        DocumentRoot "/some/path"
        DirectoryIndex app.php
        <Directory "/some/path">
            Allow from all
            Order allow,deny
            SSLOptions +StdEnvVars
        </Directory>

        SSLEngine on
        SSLCertificateFile "/path/to/server.crt"
        SSLCertificateKeyFile "/path/to/server.key"
        SSLCertificateChainFile "/path/to/ca.crt"
        SSLCACertificateFile "/path/to/ca.crt"
        SSLVerifyClient require
        SSLVerifyDepth 1
    </VirtualHost>

Первоначально username это email, указанный в сертификате (значение
переменной окружения ``SSL_CLIENT_S_DN_Email``).

.. tip::

    Идентификация через сертификат работает только когда пользователь обращается
    к приложению через HTTPS.

.. index::
   single: Security; Anonymous Users

Анонимные пользователи
~~~~~~~~~~~~~~~

Когда отключается безопасность, то больше ни один из пользователей не
прикрепляется к запросу. Но если вы всё же хотите этого, активизируйте анонимных
пользователей. Анонимный это идентифицированный пользователь, но имеющий только
одну роль ``IS_AUTHENTICATED_ANONYMOUSLY``. "Настоящая" идентификация происходит
только тогда, когда пользователь допускается к ресурсам, ограниченным более
жёсткими правилами контроля:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    anonymous: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <anonymous />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main' => array('anonymous' => true),
            ),
        ));

Так как анонимные пользователи идентифицированы, то метод ``isAuthenticated()``
возвращает ``true``. Чтобы проверить анонимный ли пользователь, проверяйте роль
``IS_AUTHENTICATED_ANONYMOUSLY`` (помните, все не анонимные пользователи
имеют роль ``IS_AUTHENTICATED_FULLY``).

.. index::
   single: Security; Stateless Authentication

Stateless идентификация
------------------------

Первоначально Symfony2 полагается на cookie (Session) чтобы сохранять контекст
безопасности пользователя. Но если вы используете сертификаты или идентификацию
через HTTP, то в сохранении нет необходимости т. к. полномочия доступны каждому
запросу. В этом случае, а также если вам не надо хранить что-либо между
запросами, можете активировать stateless идентификацию (это значит что ни один
cookie не будет создан Symfony2):

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http_basic: true
                    stateless:  true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall stateless="true">
                <http-basic />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main' => array('http_basic' => true, 'stateless' => true),
            ),
        ));

.. note::

    Если используется вход через форму, Symfony2 создаст cookie даже если
    ``stateless`` равно ``true``.

.. index::
   single: Security; Impersonating

Обезличивание пользователя
--------------------

Иногда полезно переключаться с одного пользователя на другого без выхода из
системы и повторного входа (например, когда вы занимаетесь отладкой или пытаетесь
понять ошибку, которую видит пользователь, но не можете её воспроизвести). Это
делается через простую активацию прослушивателя ``switch-user``::

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http_basic:  true
                    switch_user: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <http-basic />
                <switch-user />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main'=> array('http_basic' => true, 'switch_user' => true),
            ),
        ));

Переключение на другого пользователя осуществляется через параметр ``_switch_user``
строки запроса для данного URL со значением равным username:

    http://example.com/somewhere?_switch_user=thomas

Чтобы обратно переключиться используйте специальный username равный ``_exit``:

    http://example.com/somewhere?_switch_user=_exit

Эта возможность должна быть доступна лишь небольшой группе пользователей.
Первоначально доступ ограничен пользователями, имеющими роль
'ROLE_ALLOWED_TO_SWITCH'. Замените первоначальную роль через настройку ``role``
для дополнительной безопасности, также измените имя параметра через настройку
``parameter``::

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http_basic:  true
                    switch_user: { role: ROLE_ADMIN, parameter: _want_to_be_this_user }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <http-basic />
                <switch-user role="ROLE_ADMIN" parameter="_want_to_be_this_user" />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main'=> array(
                    'http_basic'  => true,
                    'switch_user' => array('role' => 'ROLE_ADMIN', 'parameter' => '_want_to_be_this_user'),
                ),
            ),
        ));

.. index::
   single: Security; Logout

Выход пользователей из системы
------------

Если хотите предоставить пользователям возможность выйти, то активируйте
прослушиватель logout:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http_basic: true
                    logout:     true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <http-basic />
                <logout />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main'=> array('http_basic' => true, 'logout' => true),
            ),
        ));

По умолчанию пользователи выходят из системы когда запрашивают путь ``/logout``
и они переадресовываются на ``/``. Это легко изменяется через настройки ``path``
и ``target``::

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            firewalls:
                main:
                    http_basic: true
                    logout:     { path: /signout, target: /signin }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <http-basic />
                <logout path="/signout" target="/signin" />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'firewalls' => array(
                'main'=> array(
                    'http_basic' => true,
                    'logout' => array('path' => '/signout', 'target' => '/signin')),
            ),
        ));

Идентификация и провайдеры пользователя
---------------------------------

Изначально фаервол использует первый объявленный ползовательский провайдер для
идентификации. Но если вы хотите использовать различные провайдеры для разных
частей web сайта, то можете явно указать их для фаервола или хотя бы для
механизма идентификации:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security.config:
            providers:
                default:
                    password_encoder: sha1
                    entity: { class: SecurityBundle:User, property: username }
                certificate:
                    users:
                        fabien@example.com: { roles: ROLE_USER }

            firewalls:
                backend:
                    pattern:    /admin/.*
                    x509:       { provider: certificate }
                    form-login: { provider: default }
                    logout:     true
                api:
                    provider:   default
                    pattern:    /api/.*
                    http_basic: true
                    stateless:  true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="default">
                <password-encoder>sha1</password-encoder>
                <entity class="SecurityBundle:User" property="username" />
            </provider>

            <provider name="certificate">
                <user name="fabien@example.com" roles="ROLE_USER" />
            </provider>

            <firewall pattern="/admin/.*">
                <x509 provider="certificate" />
                <form-login provider="default" />
                <logout />
            </firewall>
            <firewall pattern="/api/.*" stateless="true" provider="default">
                <http-basic />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', 'config', array(
            'providers' => array(
                'default' => array(
                    'password_encoder' => 'sha1',
                    'entity' => array('class' => 'SecurityBundle:User', 'property' => 'username'),
                ),
                'certificate' => array('users' => array(
                    'fabien@example.com' => array('roles' => 'ROLE_USER'),
                ),
            ),

            'firewalls' => array(
                'backend' => array(
                    'pattern' => '/admin/.*',
                    'x509' => array('provider' => 'certificate'),
                    'form-login' => array('provider' => 'default')
                    'logout' => true,
                ),
                'api' => array(
                    'provider' => 'default',
                    'pattern' => '/api/.*',
                    'http_basic' => true,
                    'stateless' => true,
                ),
            ),
        ));

В этом примере ``/admin/.*`` URL-ы принимают пользователей от провайдера
``certificate``, когда используется идентификация X.509, и от провайдера ``default``,
когда пользователь входит через форму. ``/api/.*`` URL-ы используют ``default``
провайдер для всех механизмов идентификации.

.. note::

    Прослушиватели не используют провайдеры пользователей напрямую, но
    идентифицируют через провайдеров. Они делают текущую идентификацию, такую
    как проверка пароля, и могут использовать для этого провайдер пользователя
    (провайдер анонимной идентификации это не тот случай).
