Безопасность
============

Обеспечение безопасности (Security) - это двух шаговый процесс, целью которого
является предотвращение доступа пользователя к ресурсам, получить которые
он не имеет права.

В первую очередь, система безопасности идентифицирует пользователя, запрашивая
у него ту или иную информацию. Этот процесс называется **аутентификацией** и
означает, что система пытается понять, кто вы есть такой.

После того как система идентифицирует вас, следующим шагом требуется определить,
имеете ли вы права на доступ к затребованному ресурсу. Эта часть процесса
называется **авторизацией** и подразумевает проверку ваших привилегий на
выполнение того или иного действия.

.. image:: /images/book/security_authentication_authorization.png
   :align: center

Поскольку лучший путь изучить что-либо - увидеть на примере, давайте
углубимся в вопросы безопасности web-приложений.

.. note::

    `security component`_ Symfony доступен как самостоятельная PHP-библиотека
    и может быть использован в любом PHP-проекте.

Простой пример: базовая HTTP аутентификация
-------------------------------------------

Компонент безопасности может быть настроен при помощи конфигурации
приложения. На самом деле, наиболее стандартные сценарии безопасности
можно настроить непосредственно через конфигурацию. Следующая конфигурация
подскажет Symfony, что нужно защитить любой URL, соответствующий шаблону
``/admin/*``, и запрашивать пользовательские данные при помощи базовой
HTTP-аутентификации (т.е. суровый олдскульный бокс username/password):

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                secured_area:
                    pattern:    ^/
                    anonymous: ~
                    http_basic:
                        realm: "Secured Demo Area"

            access_control:
                - { path: ^/admin, roles: ROLE_ADMIN }

            providers:
                in_memory:
                    users:
                        ryan:  { password: ryanpass, roles: 'ROLE_USER' }
                        admin: { password: kitten, roles: 'ROLE_ADMIN' }

            encoders:
                Symfony\Component\Security\Core\User\User: plaintext

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <firewall name="secured_area" pattern="^/">
                    <anonymous />
                    <http-basic realm="Secured Demo Area" />
                </firewall>

                <access-control>
                    <rule path="^/admin" role="ROLE_ADMIN" />
                </access-control>

                <provider name="in_memory">
                    <user name="ryan" password="ryanpass" roles="ROLE_USER" />
                    <user name="admin" password="kitten" roles="ROLE_ADMIN" />
                </provider>

                <encoder class="Symfony\Component\Security\Core\User\User" algorithm="plaintext" />
            </config>
        </srv:container>

    .. code-block:: php

        <?php
        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    'pattern' => '^/',
                    'anonymous' => array(),
                    'http_basic' => array(
                        'realm' => 'Secured Demo Area',
                    ),
                ),
            ),
            'access_control' => array(
                array('path' => '^/admin', 'role' => 'ROLE_ADMIN'),
            ),
            'providers' => array(
                'in_memory' => array(
                    'users' => array(
                        'ryan' => array('password' => 'ryanpass', 'roles' => 'ROLE_USER'),
                        'admin' => array('password' => 'kitten', 'roles' => 'ROLE_ADMIN'),
                    ),
                ),
            ),
            'encoders' => array(
                'Symfony\Component\Security\Core\User\User' => 'plaintext',
            ),
        ));

.. tip::

    Стандартный дистрибутив Symfony выделяет настройку безопасности в отдельный
    файл (по умолчанию ``app/config/security.yml``). Если вам не нужен отдельный
    конфигурационный файл для настройки безопасности, вы можете переместить
    его контент непосредственно в основной конфигурационный файл
    (по умолчанию ``app/config/config.yml``).

В результате использования такой конфигурации вы получите полнофункциональную
систему безопасности, которая выглядит следующим образом:

* Есть два пользователя системы (``ryan`` and ``admin``);
* Пользователи аутентифицируются при помощи базовой HTTP-аутентификации;
* Любой URL, соответствующий шаблону ``/admin/*``, будет защищен, и лишь пользователь ``admin`` сможет попасть туда;
* Любой URL, *НЕ* соответствующий шаблону ``/admin/*``, будет доступен любому пользователю без ввода логина/пароля.

Давайте взглянем на то, как работает безопасность и как каждая часть конфигурации
вступает в игру.

Как работает безопасность: Аутентификация и Авторизация
-------------------------------------------------------

Система безопасности Symfony работает, определяя "личность" пользователя
(аутентификация) и? затем, проверяя, имеет ли этот пользователь доступ к
конкретному ресурсу или URL.

Брандмауэры (Аутентификация)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Когда пользователь выполняет запрос к URL, защищённому брандмауэром,
активируется система безопасности. Работа брандмауэра заключается в
определении того, требуется ли аутентификация пользователя и, если
требуется, отправить обратно ответ, инициирующий процесс аутентификации.

Брандмауэр активируется, когда URL входящего запроса соответствует
регулярному выражению ``pattern``, которое было указано в конфигурации. В данном
примере шаблон ``pattern`` (``^/``) будет соответствовать *любому* входящему
запросу. То, что брандмауэр активируется, *не означает*, что HTTP аутентификация
(бокс с логином/паролем) будет требоваться для каждого URL. К примеру,
пользователь может получить доступ к ``/foo`` без запроса аутентификации:

.. image:: /images/book/security_anonymous_user_access.png
   :align: center

Это работает, так как брандмауэр позволяет доступ *анонимному пользователю* на
основании параметра ``anonymous`` в настройках безопасности. Другими словами,
брандмауэр не требует немедленной аутентификации. И, поскольку доступ к ``/foo``
не требует никакой особой роли (``role``) (это указано в секции ``access_control``),
запрос будет выполнен без аутентификации пользователя.

Если вы удалите ключ ``anonymous``, брандмауэр будет *всегда* запрашивать у
пользователя немедленной аутентификации.

Контроль доступа (Авторизация)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если пользователь запрашивает ``/admin/foo``, процесс ведёт себя иным образом.
Это обусловлено тем, что в секции ``access_control`` указано, что любой URL,
соответствующий шаблону ``^/admin`` (т.е. ``/admin`` или всё прочее, что
соответствует ``/admin/*``) требует наличия у пользователя роли ``ROLE_ADMIN``.
Роли являются основой авторизации: пользователь может получить доступ к
``/admin/foo`` лишь тогда, когда у него есть роль ``ROLE_ADMIN``.

.. image:: /images/book/security_anonymous_user_denied_authorization.png
   :align: center

Как и ранее, когда пользователь выполняет запрос, брандмауэр не требует
идентификации пользователя. Тем не менее, как только контроль доступа
отказывает пользователю в действии (так как анонимный пользователь не
имеет роли ``ROLE_ADMIN``), брандмауэр вступает в игру и инициирует
процесс аутентификации. Процесс аутентификации зависит от механизма аутентификации,
который вы используете. Например, если вы используете аутентификацию с
использованием формы логина, пользователь будет перенаправлен на страницу
логина. Если используется HTTP-аутентификация, пользователю будет направлен
ответ с HTTP статус кодом 401 и в браузере будет отображён бокс с полями
username и password.

Пользователь теперь имеет возможность отправить свои данные обратно приложению.
Если эти данные будут валидными, оригинальный запрос пользователя будет обработан.

.. image:: /images/book/security_ryan_no_role_admin_access.png
   :align: center

В этом примере, пользователь ``ryan`` успешно проходит аутентификацию в
брандмауэре, но, так как ``ryan`` не имеет роли ``ROLE_ADMIN``, он по-прежнему
не имеет доступа к ``/admin/foo``. В конечном итоге, это означает, что пользователь
увидит некое сообщение, о том, что ему отказано в доступе.

.. tip::

    Когда Symfony запрещает доступ пользователю, ему отображается страница
    с ошибкой и возвращается HTTP статус код 403 (``Forbidden``). Вы можете
    изменить дизайн страницы с ошибкой 403, следуя руководству из книги рецептов
    :ref:`Error Pages<cookbook-error-pages-by-status-code>`.

Наконец, если пользователь ``admin`` запрашивает ``/admin/foo``, имеет
место схожий процесс, за тем исключением, что система контроля доступа
разрешит прохождение этого запроса:

.. image:: /images/book/security_admin_role_access.png
   :align: center

Путь запроса, когда пользователь запрашивает защищённый ресурс, прямолинеен,
но вместе с тем гибок. Как вы увидите позднее, аутентификация может быть выполнена
различными способами, включая форму логина, сертификат X.509 или же посредством
Twitter. Вне зависимости от метода аутентификации, запрос проходит следующий путь:

#. Пользователь запрашивает защищённый ресурс;
#. Приложение перенаправляет пользователя на форму логина (или её аналог);
#. Пользователь отправляет свои данные (например username/password);
#. Брандмауэр производит аутентификацию пользователя;
#. Аутентифицированный пользователь получает оригинальный запрос.

.. note::

    Процесс аутентификации *целиком* зависит от типа аутентификации, который
    вы используете. Например, при использовании формы логина, пользователь
    отправляет свои данные по URL-адресу, который обрабатывает форму
    (например, ``/login_check``) и после этого он перенаправляется на
    изначально запрошенный URL (например, ``/admin/foo``). Но при использовании
    HTTP аутентификации пользователь отправляет свои данные не уходя с
    запрошенного URL (например, ``/admin/foo``) и после этого страница
    отправляется пользователю без перенаправлений.

    Эти особенности не должны вам причинять проблем, но лучше о них знать заранее.

.. tip::

    В дальнейшем вы также узнаете, как можно защитить *любой* объект в Symfony2,
    включая отдельные контроллеры, объекты и даже PHP-методы.

.. _book-security-form-login:

Используем традиционную форму логина
------------------------------------

До сих пор вы узнали, как защитить ваше приложение при помощи брандмауэра
и, затем, ограничить доступ к отдельным разделам при помощи ролей. Используя
HTTP аутентификацию, вы можете, не прилагая усилий, воспользоваться
нативным окошком для аутентификации при помощи логина и пароля. Помимо этого,
Symfony поддерживает много механизмов аутентификации "из коробки". Подробнее
с ними вы можете ознакомиться в разделе справочной информации
:doc:`Настройка параметров безопасности</reference/configuration/security>`.

В этой секции вы улучшите процесс аутентификации, дав пользователю возможность
воспользоваться традиционной формой логина.

Во-первых, активируйте форму в брандмауэре:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                secured_area:
                    pattern:    ^/
                    anonymous: ~
                    form_login:
                        login_path:  /login
                        check_path:  /login_check

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <config>
                <firewall name="secured_area" pattern="^/">
                    <anonymous />
                    <form-login login_path="/login" check_path="/login_check" />
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        <?php
        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    'pattern' => '^/',
                    'anonymous' => array(),
                    'form_login' => array(
                        'login_path' => '/login',
                        'check_path' => '/login_check',
                    ),
                ),
            ),
        ));

.. tip::

    Если вы не хотите изменять значения ``login_path`` или ``check_path``
    используемые по умолчанию, вы можете упростить конфигурацию:

    .. configuration-block::

        .. code-block:: yaml

            form_login: ~

        .. code-block:: xml

            <form-login />

        .. code-block:: php

            'form_login' => array(),

Теперь, когда система безопасности инициирует процесс аутентификации, она
перенаправляет пользователя на форму логина (``/login`` по умолчанию). Как
эта форма будет выглядеть - это ваша забота. Сначала создайте два маршрута:
один для отображения формы (т.е. ``/login``) другой будет обрабатывать
отправку формы логина (т.е. ``/login_check``):

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        login:
            pattern:   /login
            defaults:  { _controller: AcmeSecurityBundle:Security:login }
        login_check:
            pattern:   /login_check

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="login" pattern="/login">
                <default key="_controller">AcmeSecurityBundle:Security:login</default>
            </route>
            <route id="login_check" pattern="/login_check" />

        </routes>

    ..  code-block:: php

        <?php
        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('login', new Route('/login', array(
            '_controller' => 'AcmeDemoBundle:Security:login',
        )));
        $collection->add('login_check', new Route('/login_check', array()));

        return $collection;

.. note::

    Вам *не требуется* реализовывать контроллер для URL ``/login_check``,
    так как брандмауэр будет автоматически перехватывать и обрабатывать
    формы, отправленные на этот URL. Не обязательно, но полезно, будет создание
    маршрута, который вы будете использовать для генерации URL отправки
    формы в шаблоне логина ниже.

Обратите внимание, что наименование маршрута ``login`` не обязательно. Действительно
же важным является URL этого маршрута (``/login``), соответствующий значению
параметра ``login_path``, так как на него система безопасности будет перенаправлять
пользователя, которому нужно залогиниться.

Затем, создайте контроллер, который будет отображать форму логина:

.. code-block:: php

    <?php
    // src/Acme/SecurityBundle/Controller/Main;
    namespace Acme\SecurityBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\Security\Core\SecurityContext;

    class SecurityController extends Controller
    {
        public function loginAction()
        {
            $request = $this->getRequest();
            $session = $request->getSession();

            // получить ошибки логина, если таковые имеются
            if ($request->attributes->has(SecurityContext::AUTHENTICATION_ERROR)) {
                $error = $request->attributes->get(SecurityContext::AUTHENTICATION_ERROR);
            } else {
                $error = $session->get(SecurityContext::AUTHENTICATION_ERROR);
            }

            return $this->render('AcmeSecurityBundle:Security:login.html.twig', array(
                // имя, введённое пользователем в последний раз
                'last_username' => $session->get(SecurityContext::LAST_USERNAME),
                'error'         => $error,
            ));
        }
    }

Не дайте этому коду запутать вас. Как вы увидите, когда пользователь
отправляет форму, система безопасности автоматически обрабатывает её.
Если юзер отправил неверные имя и пароль, этот контроллер получает ошибки
от системы безопасности, чтобы вы могли их отобразить пользователю.

Другими словами, ваша работа заключается в отображении формы логина и
ошибок, которые могут возникнуть, в то время как система безопасности
берёт на себя заботу по проверке введённых имени и пароля и аутентификации
пользователя.

Наконец, создадим шаблон формы:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/SecurityBundle/Resources/views/Security/login.html.twig #}
        {% if error %}
            <div>{{ error.message }}</div>
        {% endif %}

        <form action="{{ path('login_check') }}" method="post">
            <label for="username">Username:</label>
            <input type="text" id="username" name="_username" value="{{ last_username }}" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            {#
                Если вы хотите контролировать URL, на который перенаправить пользователя:
                <input type="hidden" name="_target_path" value="/account" />
            #}

            <input type="submit" name="login" />
        </form>

    .. code-block:: html+php

        <?php // src/Acme/SecurityBundle/Resources/views/Security/login.html.php ?>
        <?php if ($error): ?>
            <div><?php echo $error->getMessage() ?></div>
        <?php endif; ?>

        <form action="<?php echo $view['router']->generate('login_check') ?>" method="post">
            <label for="username">Username:</label>
            <input type="text" id="username" name="_username" value="<?php echo $last_username ?>" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            <!--
                Если вы хотите контролировать URL, на который перенаправить пользователя:
                <input type="hidden" name="_target_path" value="/account" />
            -->

            <input type="submit" name="login" />
        </form>

.. tip::

    Переменная ``error``, передаваемая в шаблон, это экземпляр класса
    :class:`Symfony\\Component\\Security\\Core\\Exception\\AuthenticationException`.
    Этот объект может содержать дополнительную информацию - даже секретную -
    об ошибке аутентификации, так что используйте его с умом!

Форма имеет немного требований. Во-первых, отправляя форму на ``/login_check``
(маршрут ``login_check``), система безопасности перехватит отправленную форму
и обработает её автоматически. Во вторых, система безопасности ожидает, что
отправленные поля будут называться ``_username`` и ``_password``
(эти наименования также можно :ref:`настроить<reference-security-firewall-form-login>`).

Вот и всё! Когда вы отправляете форму, система безопасности автоматически
проверит данные пользователя и, либо выполнить его аутентификацию, либо
отправить пользователя обратно на форму логина, где он увидит возникшие ошибки.

Давайте ещё раз взглянем на процесс целиком:

#. Пользователь пытается получить доступ к защищённому ресурсу;
#. Брандмауэр инициирует процесс аутентификации, перенаправляя пользователя на форму логина (``/login``);
#. Страница ``/login`` отображает форму логина при помощи маршрута и контроллера, созданных в этом примере;
#. Пользователь отправляет форму логина на URL ``/login_check``;
#. Система безопасности перехватывает запрос, проверяет данные, отправленные пользователем,
   аутентифицирует пользователя, если данные верны или же возвращает
   пользователю страницу логина, если данные не верны.

По умолчанию, если данные пользователя верны, пользователь будет
перенаправлен на ту же страницу, которую и запрашивал (например, ``/admin/foo``).
Если пользователь сразу открыл страницу логина, то он будет перенаправлен на
главную страницу (``homepage``). Это поведение можно настроить, к примеру
разрешить перенаправление пользователя на фиксированный URL.

Дополнительную информацию о том, как настраивается форма логина, смотрите
статью в книге рецептов :doc:`/cookbook/security/form_login`.

.. _book-security-common-pitfalls:

.. sidebar:: Типичные ошибки

    При настройке вашей формы логина, избегайте следующих типичных ошибок.

    **1. Создавайте корректные маршруты**

    Во-первых, удостоверьтесь, что вы корректно определили маршруты
    ``/login`` и ``/login_check`` и что они соответствуют конфигурационным
    параметрам ``login_path`` и ``check_path``. Ошибки здесь будут вызывать
    перенаправление на страницу 404 вместо страницы логина или же
    отправленная форма не будет обрабатываться (вы будете видеть форму логина
    снова и снова).

    **2. Удостоверьтесь, что страница логина НЕ защищена**

    Также, удостоверьтесь, что страница логина *НЕ* требует никаких ролей
    для доступа к ней. Например, следующая конфигурация - которая требует
    роли ``ROLE_ADMIN`` для всех URL (включая URL ``/login``), будет
    вызывать зацикливание перенаправлений:

    .. configuration-block::

        .. code-block:: yaml

            access_control:
                - { path: ^/, roles: ROLE_ADMIN }

        .. code-block:: xml

            <access-control>
                <rule path="^/" role="ROLE_ADMIN" />
            </access-control>

        .. code-block:: php

            'access_control' => array(
                array('path' => '^/', 'role' => 'ROLE_ADMIN'),
            ),

    Удаление контроля доступа для URL ``/login`` исправляет проблему:

    .. configuration-block::

        .. code-block:: yaml

            access_control:
                - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
                - { path: ^/, roles: ROLE_ADMIN }

        .. code-block:: xml

            <access-control>
                <rule path="^/login" role="IS_AUTHENTICATED_ANONYMOUSLY" />
                <rule path="^/" role="ROLE_ADMIN" />
            </access-control>

        .. code-block:: php

            'access_control' => array(
                array('path' => '^/login', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY'),
                array('path' => '^/', 'role' => 'ROLE_ADMIN'),
            ),

    Также, если ваш брандмауэр *не* разрешает доступ анонимным пользователям,
    вам необходимо создать особый брандмауэр, который позволяет анонимному
    пользователю получить доступ к странице логина:

    .. configuration-block::

        .. code-block:: yaml

            firewalls:
                login_firewall:
                    pattern:    ^/login$
                    anonymous:  ~
                secured_area:
                    pattern:    ^/
                    form_login: ~

        .. code-block:: xml

            <firewall name="login_firewall" pattern="^/login$">
                <anonymous />
            </firewall>
            <firewall name="secured_area" pattern="^/">
                <form_login />
            </firewall>

        .. code-block:: php

            'firewalls' => array(
                'login_firewall' => array(
                    'pattern' => '^/login$',
                    'anonymous' => array(),
                ),
                'secured_area' => array(
                    'pattern' => '^/',
                    'form_login' => array(),
                ),
            ),

    **3. Удостоверьтесь, что /login_check находится под защитой брандмауэра**

    Далее, удостоверьтесь, что ``check_path`` (например, ``/login_check``)
    находится под защитой брандмауэра, который используется для создания вашей
    формы логина (в этом примере используется один брандмауэр, соответствующий
    *всем* URL, включая ``/login_check``). Если ``/login_check`` не соответствует
    никакому брандмауэру, вы получите исключение ``Unable to find the controller
    for path "/login_check"``.

    **4. Несколько брандмауэров не разделяют контекст безопасности**

    Если вы используете много брандмауэров и аутентификацию производите
    через один из них, вы *не* будете аутентифицированы всеми оставшимися
    брандмауэрами автоматически. Различные брандмауэры можно рассматривать как
    различные системы безопасности. Вот почему большинству приложений достаточно
    одного основного брандмауэра.

Авторизация
-----------

Первым шагом в обеспечении безопасности всегда является аутентификация: процесс
идентификации пользователя. В Symfony аутентификацию можно выполнять различными
способами, начиная с базовой HTTP аутентификации и формы логина и заканчивая
Facebook.

После того как пользователь аутентифицирован, начинается процесс авторизации.
Авторизация является стандартным путём определения имеет ли пользователь
право доступа к какому-либо ресурсу (URL, объект модели, вызов метода...).
В основе этого процесса лежит присвоение некоторых ролей каждому пользователю
и после этого для различных ресурсов можно требовать наличия различных ролей.

Авторизация имеет две различные грани:

#. Пользователю назначен некоторый набор ролей;
#. Ресурс требует наличия некоторых ролей для получения доступа к нему.

В этой секции вы узнаете о том, как защитить различные ресурсы (например, URL,
вызов метода и т.д.) при помощи различных ролей. Затем, вы узнаете о том, как
создаются роли и как их можно присвоить пользователю.

Защищаем URL по шаблону
~~~~~~~~~~~~~~~~~~~~~~~

Наиболее простой и понятный способ защиты вашего приложения - защита некоторого
набора URL по шаблону. Вы уже видели ранее, в первом примере этой главы, где
все URL, что соответствовали регулярному выражению ``^/admin``, требовали
роли ``ROLE_ADMIN``.

Вы можете определить столько URL, сколько вам нужно - каждый при помощи шаблона
для регулярного выражения:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...
            access_control:
                - { path: ^/admin/users, roles: ROLE_SUPER_ADMIN }
                - { path: ^/admin, roles: ROLE_ADMIN }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->
            <rule path="^/admin/users" role="ROLE_SUPER_ADMIN" />
            <rule path="^/admin" role="ROLE_ADMIN" />
        </config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('security', array(
            // ...
            'access_control' => array(
                array('path' => '^/admin/users', 'role' => 'ROLE_SUPER_ADMIN'),
                array('path' => '^/admin', 'role' => 'ROLE_ADMIN'),
            ),
        ));

.. tip::

    Добавление в начало пути символа ``^`` гарантирует, что этому шаблону
    будут соответствовать лишь URL, которые *начинаются* c него. Например,
    путь ``/admin`` (без ``^`` в начале) будет соответствовать как URL ``/admin/foo``,
    так и URL ``/foo/admin``.

Для каждого входящего запроса, Symfony2 пытается найти соответствующее правило
контроля доступа (используется первое найденное правило). Если пользователь
ещё не прошёл аутентификацию, инициируется процесс аутентификации (т.е.
пользователю предоставляется возможность залогиниться в систему). Если же
пользователь уже прошёл аутентификацию, но не имеет требуемой роли, будет
брошено исключение :class:`Symfony\\Component\\Security\\Core\\Exception\\AccessDeniedException`,
которое вы можете обработать и показать пользователю красивую страничку
"access denied". Подробнее читайте в книге рецептов: :doc:`/cookbook/controller/error_pages`

Так как Symfony использует первое найденное правило, URL вида ``/admin/users/new``
будет соответствовать первому правилу и требовать наличия роли ``ROLE_SUPER_ADMIN``.
Любой URL вида ``/admin/blog`` будет соответствовать второму правилу и требовать
наличия роли ``ROLE_ADMIN``.

.. _book-security-securing-ip:

Защита по IP
~~~~~~~~~~~~

В жизни могут возникать различные ситуации, в которых вам будет необходимо
ограничить доступ для некоего маршрута по IP. Это особенно важно в случае
использования :ref:`Edge Side Includes<edge-side-includes>` (ESI), которые,
например, используют маршрут под названием "_internal". Когда используются
ESI, маршрут _internal необходим кэширующему шлюзу для подключения различных
опций кэширования субсекций внутри указанной страницы. Этот маршрут по умолчанию
использует префикс ^/_internal в Symfony Standard Edition (предполагается
также что вы раскомментировали эти строки в файле маршрутов).

Ниже приводится пример того, как вы можете защитить этот маршрут от
доступа извне:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            access_control:
                - { path: ^/_internal, roles: IS_AUTHENTICATED_ANONYMOUSLY, ip: 127.0.0.1 }

    .. code-block:: xml

            <access-control>
                <rule path="^/_internal" role="IS_AUTHENTICATED_ANONYMOUSLY" ip="127.0.0.1" />
            </access-control>

    .. code-block:: php

            'access_control' => array(
                array('path' => '^/_internal', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY', 'ip' => '127.0.0.1'),
            ),

.. _book-security-securing-channel:

Использование защищённого канала
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Как и защита на основании IP, требование использования SSL добавляется очень
просто:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            access_control:
                - { path: ^/cart/checkout, roles: IS_AUTHENTICATED_ANONYMOUSLY, requires_channel: https }

    .. code-block:: xml

            <access-control>
                <rule path="^/cart/checkout" role="IS_AUTHENTICATED_ANONYMOUSLY" requires_channel: https />
            </access-control>

    .. code-block:: php

            'access_control' => array(
                array('path' => '^/cart/checkout', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY', 'requires_channel' => 'https'),
            ),

.. _book-security-securing-controller:

Защита Контроллера
~~~~~~~~~~~~~~~~~~

Защищать ваше приложение на основании шаблонов URL легко, но в некоторых случаях
может быть не слишком удобным. При необходимости, вы также можете легко
форсировать авторизацию внутри контроллера:

.. code-block:: php

    <?php
    use Symfony\Component\Security\Core\Exception\AccessDeniedException
    // ...

    public function helloAction($name)
    {
        if (false === $this->get('security.context')->isGranted('ROLE_ADMIN')) {
            throw new AccessDeniedException();
        }

        // ...
    }

.. _book-security-securing-controller-annotations:

Вы также можете использовать опциональный пакет ``JMSSecurityExtraBundle``,
который поможет вам защитить контроллер с использованием аннотаций:

.. code-block:: php

    <?php
    use JMS\SecurityExtraBundle\Annotation\Secure;

    /**
     * @Secure(roles="ROLE_ADMIN")
     */
    public function helloAction($name)
    {
        // ...
    }

Дополнительную информацию об этом пакете вы можете получить из документации
`JMSSecurityExtraBundle`_. Если вы используете дистрибутив Symfony Standard Edition,
этот пакет уже доступен вам по умолчанию. В противном случае вам необходимо
загрузить и установить его.

Защита прочих сервисов
~~~~~~~~~~~~~~~~~~~~~~

Фактически, всё что угодно в Symfony может быть защищено при помощи стратегии,
описанной в предыдущей секции. Например, предположим у вас есть сервис (т.е.
PHP класс), который отсылает email-сообщения от одного пользователя другому.
Вы можете ограничить использование этого класса - неважно где он будет использован -
для пользователей с определённой ролью.

Подробнее о том как вы можете использовать компонент безопасности для
защиты различных сервисов и методов в вашем приложении, смотрите статью
в книге рецептов: :doc:`/cookbook/security/securing_services`.

Списки контроля доступа (ACL): Защита отдельных объектов базы данных
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Представьте, что вы проектируете блог, где пользователи могут создавать
комментарии к вашим постам. Теперь вы хотите, чтобы пользователь имел возможность
редактировать его собственный комментарий, но не мог редактировать комментарии
других пользователей. Также, в качестве администратора, вы хотите иметь возможность
редактировать комментарии *всех* пользователей.

Компонент безопасности содержит опциональную систему "списков контроля доступа"
(ACL), которую вы можете использовать при необходимости контроля доступа к
отдельным экземплярам объектов в вашей системе. *Без* использования ACL,
вы можете защитить свою систему таким образом, что лишь некоторые пользователи
смогут иметь возможность редактирования комментариев. Но с помощью ACL,
вы можете ограничить ли разрешить доступ к каждому конкретному комментарию.

Подробнее читайте в книге рецептов: :doc:`/cookbook/security/acl`.

Пользователи
------------

В предыдущей секции вы узнали как можно защитить различные ресурсы, требуя
для них наличия одной или нескольких *ролей*. В этой секции вы узнаете о другой
грани авторизации: пользователях.

Откуда берутся пользователи? (*Провайдеры Пользователей*)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Во время аутентификации, пользователь отправляет некоторые данные (как правило
имя и пароль). Работа системы аутентификации заключается в том, чтобы проверить
эти данные на некотором наборе пользователей. Откуда же берутся эти пользователи?

В Symfony2 пользователи могут появляться отовсюду - из файла конфигурации,
базы данных, веб сервиса или откуда вашей душе угодно будет. Всё, что предоставляет
одного или более пользователей системе аутентификации называется "провайдером пользователя"
(user provider). Symfony2 поставляется с двумя, наиболее простыми провайдерами:
один из них загружает пользователей из конфигурационного файла, другой загружает
пользователей из таблицы в базе данных.

Определение пользователей в файле конфигурации
..............................................

Наиболее простой способ создать пользователей - определить их прямо
в файле конфигурации. Фактически вы уже видели этот способ ранее в
одном из примеров этой главы.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...
            providers:
                default_provider:
                    users:
                        ryan:  { password: ryanpass, roles: 'ROLE_USER' }
                        admin: { password: kitten, roles: 'ROLE_ADMIN' }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->
            <provider name="default_provider">
                <user name="ryan" password="ryanpass" roles="ROLE_USER" />
                <user name="admin" password="kitten" roles="ROLE_ADMIN" />
            </provider>
        </config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('security', array(
            // ...
            'providers' => array(
                'default_provider' => array(
                    'users' => array(
                        'ryan' => array('password' => 'ryanpass', 'roles' => 'ROLE_USER'),
                        'admin' => array('password' => 'kitten', 'roles' => 'ROLE_ADMIN'),
                    ),
                ),
            ),
        ));

Такой провайдер называется провайдером "в памяти" (in-memory), так как
пользователи не сохранены где-либо в базе данных. В итоге предоставляется
объект класса :class:`Symfony\\Component\\Security\\Core\\User\\User`.

.. tip::

    Любой провайдер пользователей может загружать пользователей непосредственно
    из конфигурации, если для него указан параметр ``users`` и определены
    пользователи.

.. caution::

    Если имя вашего пользователя полностью цифровое (например, ``77``) или
    содержит тире (например, ``user-name``), вы должны использовать
    альтернативный синтаксис при создании пользователей в YAML файле:

    .. code-block:: yaml

        users:
            - { name: 77, password: pass, roles: 'ROLE_USER' }
            - { name: user-name, password: pass, roles: 'ROLE_USER' }

Для небольших сайтов этот метод быстр и прост в настройке. Для более сложных
систем вы вероятно захотите загружать пользователей из базы данных.

.. _book-security-user-entity:

Загрузка пользователей из базы данных
.....................................

Если вы хотите загружать пользователей из базы данных при помощи Doctrine ORM,
вы можете этого легко достичь, создав класс ``User`` и настроив провайдер ``entity``.

.. tip:

    Существует очень хороший и удобный сторонний пакет, который позволяет
    хранить пользователей в базе данных при помощи Doctrine ORM или ODM.
    Подробнее о нём можно узнать на GitHub: `FOSUserBundle`_.

При таком подходе вы сначала создаёте свой собственный класс ``User``,
который будет сохраняться в базе данных:

.. code-block:: php

    <?php
    // src/Acme/UserBundle/Entity/User.php
    namespace Acme\UserBundle\Entity;

    use Symfony\Component\Security\Core\User\UserInterface;
    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity
     */
    class User implements UserInterface
    {
        /**
         * @ORM\Column(type="string", length="255")
         */
        protected $username;

        // ...
    }

Что же качается системы безопасности, единственным её требованием к вашему
классу пользователя является имплементация им интерфейса
:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`. Это означает,
что ваша концепция пользователя может быть какой угодно, коль скоро класс
имплементирует этот интерфейс.

.. note::

    Объект пользователя будет сериализован и сохранён в сессии во время
    обработки запроса, поэтому рекомендуется также имплементировать интерфейс
    `\Serializable`_ для вашего пользователя. Это особенно важно, если
    ваш класс ``User`` имеет родителя с приватными свойствами.

Далее, настроим провайдер ``entity`` и укажем для него класс ``User``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                main:
                    entity: { class: Acme\UserBundle\Entity\User, property: username }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="main">
                <entity class="Acme\UserBundle\Entity\User" property="username" />
            </provider>
        </config>

    .. code-block:: php

        <?php
        // app/config/security.php
        $container->loadFromExtension('security', array(
            'providers' => array(
                'main' => array(
                    'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'username'),
                ),
            ),
        ));

Добавив этот новый провайдер, система аутентификации будет пытаться
загрузить объект ``User`` из базы данных, используя его поле ``username``.

.. note::

    Этот пример предназначен чтобы показать вам основную идею провайдера
    ``entity``. Полный рабочий пример приводится в книге рецептов:
    :doc:`/cookbook/security/entity_provider`.

Больше информации о создании вашего собственного провайдера (например, если вам
нужно загружать пользователей из веб-сервиса), смотрите статью
:doc:`/cookbook/security/custom_provider`.

Шифрование пароля пользователя
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ранее, для упрощения, все примеры хранили пароли пользователей в виде
текста (вне зависимости от того где эти пользователи были определены
- в файле настроек или в базе данных). Конечно, в настоящем приложении
вы захотите шифровать пароли пользователей из соображений безопасности.
Этого легко достичь, связав ваш класс User с одним из нескольких встроенных
"процедур шифрования". Например, при хранении ваших пользователей в памяти, чтобы
скрывать их пароли при помощи функции ``sha1``, выполните следующие настройки:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...
            providers:
                in_memory:
                    users:
                        ryan:  { password: bb87a29949f3a1ee0559f8a57357487151281386, roles: 'ROLE_USER' }
                        admin: { password: 74913f5cd5f61ec0bcfdb775414c2fb3d161b620, roles: 'ROLE_ADMIN' }

            encoders:
                Symfony\Component\Security\Core\User\User:
                    algorithm:   sha1
                    iterations: 1
                    encode_as_base64: false

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->
            <provider name="in_memory">
                <user name="ryan" password="bb87a29949f3a1ee0559f8a57357487151281386" roles="ROLE_USER" />
                <user name="admin" password="74913f5cd5f61ec0bcfdb775414c2fb3d161b620" roles="ROLE_ADMIN" />
            </provider>

            <encoder class="Symfony\Component\Security\Core\User\User" algorithm="sha1" iterations="1" encode_as_base64="false" />
        </config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('security', array(
            // ...
            'providers' => array(
                'in_memory' => array(
                    'users' => array(
                        'ryan' => array('password' => 'bb87a29949f3a1ee0559f8a57357487151281386', 'roles' => 'ROLE_USER'),
                        'admin' => array('password' => '74913f5cd5f61ec0bcfdb775414c2fb3d161b620', 'roles' => 'ROLE_ADMIN'),
                    ),
                ),
            ),
            'encoders' => array(
                'Symfony\Component\Security\Core\User\User' => array(
                    'algorithm'         => 'sha1',
                    'iterations'        => 1,
                    'encode_as_base64'  => false,
                ),
            ),
        ));

Присвоив параметру ``iterations`` значение 1 и параметру ``encode_as_base64`` - false,
пароль будет просто прогоняться один раз через алгоритм шифрования ``sha1``
без дополнительного шифрования. Теперь вы можете вычислить хэш пароля програмно
(``hash('sha1', 'ryanpass')``) или же при помощи онлайн-инструментов типа `functions-online.com`_.

Если вы создаёте ваших пользователей динамически (и храните их в базе данных),
вы можете использовать более сложные алгоритмы шифрования, а затем
передавать оригинал пароля объекту-шифровальщику для хеширования паролей. Например,
предположим что ваш объект User - это экземпляр класса ``Acme\UserBundle\Entity\User``
(как в примере выше). Сначала настройте шифрование для этого класса пользователя:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            # ...

            encoders:
                Acme\UserBundle\Entity\User: sha512

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <!-- ... -->

            <encoder class="Acme\UserBundle\Entity\User" algorithm="sha512" />
        </config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('security', array(
            // ...

            'encoders' => array(
                'Acme\UserBundle\Entity\User' => 'sha512',
            ),
        ));

В этом случае вы используете более стойкий алгоритм ``sha512``. Также, поскольку
вы просто указали алгоритм шифрования в виде строки (``sha512``), система
будет по умолчанию хэшировать ваш пароль 5000 раз подряд и затем шифровать его
в base64. Другими словами, пароль будет многократно зашифрован и пароль не сможет
быть декодирован (т.е. будет невозможно определить оригинал пароля по его хэшу).

Если у вас предусмотрена некая регистрация для пользователей, вам потребуется
определить хэш пароля, чтобы присвоить его пользователю. Вне зависимости от алгоритма
шифрования, который вы настроили для объекта пользователя, в контроллере получить хэш
пароля можно следующим образом:

.. code-block:: php

    <?php
    // ...

    $factory = $this->get('security.encoder_factory');
    $user = new Acme\UserBundle\Entity\User();

    $encoder = $factory->getEncoder($user);
    $password = $encoder->encodePassword('ryanpass', $user->getSalt());
    $user->setPassword($password);

Получение объекта пользователя
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

После аутентификации, объект ``User`` для текущего юзера можно получить
через сервис ``security.context``. В контроллере это будет выглядеть следующим
образом:

.. code-block:: php

    <?php
    public function indexAction()
    {
        $user = $this->get('security.context')->getToken()->getUser();
    }

.. note::

    Анонимные пользователи технически считаются также аутентифицированными, т.е. метод
    ``isAuthenticated()`` анонимного пользователя будет возвращать ``true``. Для того,
    чтобы действительно убедиться, что ваш пользователь прошёл аутентификацию, необходимо
    проверить наличие роли ``IS_AUTHENTICATED_FULLY``.

Использование нескольких провайдеров пользователей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Любой механизм аутентификации (HTTP аутентификация, форма логина и т.п.)
использует только один провайдер и будет по умолчанию использовать первый
указанный. Но что, если вы хотите указать несколько пользователей при помощи
конфигурации и остальных пользователей сохранять в базу данных? Можно
создать новый chain-провайдер, который позволит добиться этого:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                chain_provider:
                    providers: [in_memory, user_db]
                in_memory:
                    users:
                        foo: { password: test }
                user_db:
                    entity: { class: Acme\UserBundle\Entity\User, property: username }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <provider name="chain_provider">
                <provider>in_memory</provider>
                <provider>user_db</provider>
            </provider>
            <provider name="in_memory">
                <user name="foo" password="test" />
            </provider>
            <provider name="user_db">
                <entity class="Acme\UserBundle\Entity\User" property="username" />
            </provider>
        </config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('security', array(
            'providers' => array(
                'chain_provider' => array(
                    'providers' => array('in_memory', 'user_db'),
                ),
                'in_memory' => array(
                    'users' => array(
                        'foo' => array('password' => 'test'),
                    ),
                ),
                'user_db' => array(
                    'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'username'),
                ),
            ),
        ));

Теперь, любой механизм аутентификации будет использовать ``chain_provider``,
так как он указан первым. В свою очередь, ``chain_provider`` будет пытаться
получить пользователя как из провайдера ``in_memory``, так и из ``user_db``.

.. tip::

    Если вам не требуется разделять пользователей ``in_memory`` от пользователей
    ``user_db``, вы можете достигнуть того же эффекта ещё быстрее, скомбинировав
    эти два ресурса в один провайдер:

    .. configuration-block::

        .. code-block:: yaml

            # app/config/security.yml
            security:
                providers:
                    main_provider:
                        users:
                            foo: { password: test }
                        entity: { class: Acme\UserBundle\Entity\User, property: username }

        .. code-block:: xml

            <!-- app/config/config.xml -->
            <config>
                <provider name=="main_provider">
                    <user name="foo" password="test" />
                    <entity class="Acme\UserBundle\Entity\User" property="username" />
                </provider>
            </config>

        .. code-block:: php

            <?php
            // app/config/config.php
            $container->loadFromExtension('security', array(
                'providers' => array(
                    'main_provider' => array(
                        'users' => array(
                            'foo' => array('password' => 'test'),
                        ),
                        'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'username'),
                    ),
                ),
            ));

Вы также можете настроить брандмауэр или индивидуальный механизм аутентификации
на использование конкретного провайдера. Напоминаем, если провайдер явно не
указан, будет использоваться первый по списку:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            firewalls:
                secured_area:
                    # ...
                    provider: user_db
                    http_basic:
                        realm: "Secured Demo Area"
                        provider: in_memory
                    form_login: ~

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <firewall name="secured_area" pattern="^/" provider="user_db">
                <!-- ... -->
                <http-basic realm="Secured Demo Area" provider="in_memory" />
                <form-login />
            </firewall>
        </config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    // ...
                    'provider' => 'user_db',
                    'http_basic' => array(
                        // ...
                        'provider' => 'in_memory',
                    ),
                    'form_login' => array(),
                ),
            ),
        ));

В этом примере, если пользователь пытается залогиниться при помощи HTTP
аутентификации - будет использоваться провайдер ``in_memory``, но если
пользователь попытается залогиниться при помощи формы логина, будет
использован провайдер ``user_db`` (так как этот провайдер является
провайдером по умолчанию для всего брандмауэра).

Подробную информацию о провайдерах пользователей и настройках брандмауэра
вы можете прочитать в справочнике: :doc:`/reference/configuration/security`.

Роли
-----

Роль имеет ключевое значение в процессе авторизации. Каждый пользователь
получает набор ролей и каждый ресурс требует наличие одной или нескольких ролей.
Если пользователь имеет необходимую роль - доступ будет разрешён. В противном
случае - доступ будет запрещён.

Роли, по сути, очень просты, это строки, которые вы можете создавать и использовать
по мере надобности (тем не менее, внутри системы роли это всё-таки объекты).
Например, если вам нужно ограничить доступ к админке блога на вашем сайте,
вы можете защитить эту секцию, используя роль ``ROLE_BLOG_ADMIN``.
Эта роль не должна быть нигде определена, вы просто начинаете её использовать
и всё.

.. note::

    Все роли в Symfony2 **должны** начинаться с префикса ``ROLE_``. Если
    вы определяете ваши роли в отдельном классе ``Role`` (продвинутый вариант),
    использовать префикс ``ROLE_`` не нужно.

Иерархические роли
~~~~~~~~~~~~~~~~~~

Вместо того, чтобы присваивать пользователю много ролей, вы можете определить
правила наследования ролей, создав их иерархию:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            role_hierarchy:
                ROLE_ADMIN:       ROLE_USER
                ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <role id="ROLE_ADMIN">ROLE_USER</role>
            <role id="ROLE_SUPER_ADMIN">ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH</role>
        </config>

    .. code-block:: php

        <?php
        // app/config/security.php
        $container->loadFromExtension('security', array(
            'role_hierarchy' => array(
                'ROLE_ADMIN'       => 'ROLE_USER',
                'ROLE_SUPER_ADMIN' => array('ROLE_ADMIN', 'ROLE_ALLOWED_TO_SWITCH'),
            ),
        ));

В примере выше, пользователь с ролью ``ROLE_ADMIN`` будет также иметь роль ``ROLE_USER``.
Роль ``ROLE_SUPER_ADMIN`` включает в себя ``ROLE_ADMIN``, ``ROLE_ALLOWED_TO_SWITCH``,
и ``ROLE_USER`` (унаследовав её от ``ROLE_ADMIN``).

Выход из системы
----------------

Как правило, вы также захотите дать вашим пользователям возможность
выйти из системы. К счастью, брандмауэр позволяет обрабатывать выход
автоматически, если вы активируете параметр ``logout`` в конфигурации:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        security:
            firewalls:
                secured_area:
                    # ...
                    logout:
                        path:   /logout
                        target: /
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <config>
            <firewall name="secured_area" pattern="^/">
                <!-- ... -->
                <logout path="/logout" target="/" />
            </firewall>
            <!-- ... -->
        </config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    // ...
                    'logout' => array('path' => 'logout', 'target' => '/'),
                ),
            ),
            // ...
        ));

Будучи настроенной для вашего брандмауэра, эта конфигурация при направлении
пользователя на ``/logout`` (или любой другой путь, который вы укажете в
параметре ``path``) будет де-аутентифицировать его. Этот пользователь будет
перенаправлен на главную страницу сайта (также может быть настроено при
помощи параметра ``target``). Оба эти параметра - ``path`` и ``target``
имеют параметры по умолчанию, такие же, как указаны в примере выше. Другими
словами, вы можете их не указывать, что упростит настройку:

.. configuration-block::

    .. code-block:: yaml

        logout: ~

    .. code-block:: xml

        <logout />

    .. code-block:: php

        'logout' => array(),

Отметим также, что вам *не* нужно создавать контроллер для URL ``/logout``,
так как брандмауэр сам позаботится обо всём. Возможно, вы также захотите создать
маршрут и использовать его для генерации URL:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        logout:
            pattern:   /logout

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="logout" pattern="/logout" />

        </routes>

    ..  code-block:: php

        <?php
        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('logout', new Route('/logout', array()));

        return $collection;

После того как пользователь выходит из системы, он будет перенаправлен по
пути, указанному в параметре ``target`` (например ``homepage``). Подробнее
о конфигурации logout читайте в
:doc:`Справочнике по настройке системы безопасности</reference/configuration/security>`.

Контроль доступа в шаблонах
---------------------------

Если вы хотите проверить, имеет ли пользователь некоторую роль внутри шаблона,
воспользуйтесь встроенным хелпером:

.. configuration-block::

    .. code-block:: html+jinja

        {% if is_granted('ROLE_ADMIN') %}
            <a href="...">Delete</a>
        {% endif %}

    .. code-block:: html+php

        <?php if ($view['security']->isGranted('ROLE_ADMIN')): ?>
            <a href="...">Delete</a>
        <?php endif; ?>

.. note::

    Если вы используете эту функцию на странице, URL которой *не* обрабатывается
    брандмауэром, будет брошено исключение. Напомним ещё раз, что в большинстве
    случаев хорошей практикой является наличие главного брандмауэра, который
    контролирует все URL (как было показано в этой главе).

Контроль доступа в контроллерах
-------------------------------

Если вы хотите проверить, имеет ли текущий пользователь ту или иную роль
в вашем контроллере, используйте метод ``isGranted`` контекста безопасности:

.. code-block:: php

    <?php
    public function indexAction()
    {
        // show different content to admin users
        if($this->get('security.context')->isGranted('ADMIN')) {
            // Загружаем админ-контент
        }
        // Загружаем прочий контент
    }

.. note::

    Брандмауэр должен быть активен, или же будет брошено исключение при вызове
    метода ``isGranted``. Посмотрите также замечание для шаблонов чуть выше.

Подмена пользователя
--------------------

Иногда необходимо иметь возможность переключения с одного пользователя
на другого без выполнения выхода/входа (например, при отладке или при попытке
воспроизвести баг, который пользователь видит, а вы нет). Это можно выполнить
при помощи листенера ``switch_user`` в брандмауэре:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    # ...
                    switch_user: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <!-- ... -->
                <switch-user />
            </firewall>
        </config>

    .. code-block:: php

        <?php
        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => true
                ),
            ),
        ));

Для переключения на другого пользователя просто добавьте в строку запроса
текущего URL параметр ``_switch_user`` и имя пользователя:

    http://example.com/somewhere?_switch_user=thomas

Для того, чтобы переключиться обратно, используйте специальное имя ``_exit``:

    http://example.com/somewhere?_switch_user=_exit

Естественно, такая возможность должна быть доступна небольшой группе пользователей.
По умолчанию, эта функция доступна пользователям с ролью ``ROLE_ALLOWED_TO_SWITCH``.
Наименование этой роли можно изменить при помощи опции ``role``. Для
большей безопасности вы также можете изменить наименования параметра для
строки запроса при помощи опции ``parameter``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    // ...
                    switch_user: { role: ROLE_ADMIN, parameter: _want_to_be_this_user }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <!-- ... -->
                <switch-user role="ROLE_ADMIN" parameter="_want_to_be_this_user" />
            </firewall>
        </config>

    .. code-block:: php

        <?php
        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => array('role' => 'ROLE_ADMIN', 'parameter' => '_want_to_be_this_user'),
                ),
            ),
        ));

Аутентификация без сохранения состояния (stateless)
---------------------------------------------------

По умолчанию, Symfony2 использует куки (сессию) для хранения контекста
безопасности пользователя. Но, если вы используете, к примеру, сертификаты или HTTP
аутентификацию, сохранение не требуется, так как авторизационные данные доступны
для каждого запроса. В этом случае, и если вы не хотите сохранять что-либо
между запросами, вы можете активировать *stateless аутентификацию* (без сохранения
состояния, т.е. Symfony2 не будет создавать куки):

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    http_basic: ~
                    stateless:  true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall stateless="true">
                <http-basic />
            </firewall>
        </config>

    .. code-block:: php

        <?php
        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('http_basic' => array(), 'stateless' => true),
            ),
        ));

.. note::

    Если вы используете форму логина, Symfony2 будет создавать куки всегда,
    даже если ``stateless`` имеет значение ``true``.

Заключение
----------

Безопасность может быть весьма сложным вопросом для решения его в вашем
приложении. К счастью, компонент безопасности Symfony следует хорошо
зарекомендовавшей себя модели, основанной на *аутентификации* и
*авторизации*. Аутентификация, которая всегда идёт первой, обрабатывается
брандмауэром, чья работа заключается установить "личность" пользователя
при помощи любого из доступных методов (HTTP аутентификация, форма логина и т.д.).
В книге рецептов вы также найдёте примеры других методов аутентификации,
включая то, как реализовать функцию "запомнить меня" при помощи куки.

После того как пользователь аутентифицирован, авторизация может определить
имеет ли он доступ к тому или иному ресурсу. Как правило, к URL, классам или
методам ставятся в соответствие некоторые *роли* и если пользователь не
имеет требуемой роли, доступ ему будет запрещён. Тем не менее, авторизация
это более сложный механизм, следующий системе "голосования", благодаря
которой множество разных участников могут определить имеет ли пользователь
права доступа к ресурсу или же нет.

Читайте также в книге рецептов
------------------------------

* :doc:`Форсирование HTTP/HTTPS </cookbook/security/force_https>`
* :doc:`Блэклистинг пользователя по IP при помощи custom voter </cookbook/security/voters>`
* :doc:`Списки контроля доступа (ACLs) </cookbook/security/acl>`
* :doc:`/cookbook/security/remember_me`

.. _`security component`: https://github.com/symfony/Security
.. _`JMSSecurityExtraBundle`: https://github.com/schmittjoh/JMSSecurityExtraBundle
.. _`FOSUserBundle`: https://github.com/FriendsOfSymfony/FOSUserBundle
.. _`\Serializable`: http://php.net/manual/en/class.serializable.php
.. _`functions-online.com`: http://www.functions-online.com/sha1.html

.. toctree::
    :hidden:

    Translation source: 2011-10-04 e6bc8ed
    Corrected from:
