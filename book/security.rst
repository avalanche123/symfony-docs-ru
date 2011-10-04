Безопасность
============

Обеспечение безопасности (Security) - это двухшаговый процесс, целью которого
является предотвращение доступа пользователя к ресурсам, отобразить которые
он не имеет права.

В первую очередь, система безопасности идентифицирует пользователя, запрашивая
у него ту или иную информацию. Этот процесс называется **аутентификацией** и
означает, что система пытается понять кто вы есть такой.

После того как система идентифицирует вас, следующим шагом требуется определить,
имеете ли вы права на доступ к озатребованному ресурсу. Эта часть процесса
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
* Любой URL, соответствующий шаблону ``/admin/*``, будет защищён и лишь пользователь ``admin`` сможет попасть туда;
* Любой URL, *НЕ* соответствующий шаблону ``/admin/*``, будет доступен любому пользователю без ввода логина/пароля.

Давайте взглянем на то, как работает безопасность и как каждая часть конфигурации
вступает в игру.

Как работает безопасность: Аутентификация и Авторизация
-------------------------------------------------------

Система безопасности Symfony работает определяя "личность" пользователя
(аутентификация) и? затем, проверяя имеет ли этот пользователь доступ к
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
брандмауэр не требует немедленной аутентификации. И, посколько доступ к ``/foo``
не требует никакой особой роли (``role``) (это указано в секции ``access_control``),
запрос будет выполнен без аутентификации пользователя.

Если вы удалите ключ ``anonymous``, брандмауэр будет *всегда* запрашивать у
пользователя немедленной аутентификации.

Контроль доступа (Авторизация)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если пользователь запрашивает ``/admin/foo``, процесс ведёт себя иным образом.
Это обусловлено тем, что в секции ``access_control`` указано, что любой URL,
соответствующий шаблону ``^/admin`` (т.е. ``/admin`` или всё прочее, что
соответствует ``/admin/*``) требует наличия у пользователя роли `ROLE_ADMIN``.
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
брандмауэре, но, так как ``ryan`` не меет роли ``ROLE_ADMIN``, он по прежнему
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
    HTTP аутентификации пользователь отправляет его данные не уходя с
    запрошенного URL (например, ``/admin/foo``) и после этого страница
    отправляется пользователю без перенаправлений.

    Эти особенности не должны вам причинять проблем, но лучше о них знать заранее.

.. tip::

    В дальнейшем вы также узнаете как можно защитить *любой* объект в Symfony2,
    включая отдельные контроллеры, объекты и даже PHP-методы.

.. _book-security-form-login:

Используем традиционную форму логина
------------------------------------

До сих пор вы узнали, как защитить ваше приложение при помощи брандмауэра
и, затем, ограничить доступ к отдельным разделам при помощи ролей. Используя
HTTP аутентификацию, вы можете, не прилагая усилий, воспользоваться
нативным окошком для аутентификации при помощи логина и проля. Помимо этого,
Symfony поддерживает много механизмов аутентификации "из коробки". Подробнее
с ними вы можете ознакомиться в разделе справочной информации
:doc:`Настройка параметров безопасности</reference/configuration/security>`.

В этой секции вы улучшите процесс аутентификации, дав пользователю возмозность
воспользоваться традиционной формой логина.

Во-первых, активируйте форму в бранмауэре:

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
Если ющер отправил неверные имя и пароль, этот контроллер получает ошибки
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

    **3. Удостоверьтесь, что ``/login_check`` находится под защитой брандмаура**

    Далее, удостоверьтесь, что ``check_path`` (например, ``/login_check``)
    находится под защитой брандмаура, который используется для создания вашей
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

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

Authorization
-------------

The first step in security is always authentication: the process of verifying
who the user is. With Symfony, authentication can be done in any way - via
a form login, basic HTTP Authentication, or even via Facebook.

Once the user has been authenticated, authorization begins. Authorization
provides a standard and powerful way to decide if a user can access any resource
(a URL, a model object, a method call, ...). This works by assigning specific
roles to each user, and then requiring different roles for different resources.

The process of authorization has two different sides:

#. The user has a specific set of roles;
#. A resource requires a specific role in order to be accessed.

In this section, you'll focus on how to secure different resources (e.g. URLs,
method calls, etc) with different roles. Later, you'll learn more about how
roles are created and assigned to users.

Securing Specific URL Patterns
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most basic way to secure part of your application is to secure an entire
URL pattern. You've seen this already in the first example of this chapter,
where anything matching the regular expression pattern ``^/admin`` requires
the ``ROLE_ADMIN`` role.

You can define as many URL patterns as you need - each is a regular expression.

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

        // app/config/config.php
        $container->loadFromExtension('security', array(
            // ...
            'access_control' => array(
                array('path' => '^/admin/users', 'role' => 'ROLE_SUPER_ADMIN'),
                array('path' => '^/admin', 'role' => 'ROLE_ADMIN'),
            ),
        ));

.. tip::

    Prepending the path with ``^`` ensures that only URLs *beginning* with
    the pattern are matched. For example, a path of simply ``/admin`` (without
    the ``^``) would correctly match ``/admin/foo`` but would also match URLs
    like ``/foo/admin``.

For each incoming request, Symfony2 tries to find a matching access control
rule (the first one wins). If the user isn't authenticated yet, the authentication
process is initiated (i.e. the user is given a chance to login). However,
if the user *is* authenticated but doesn't have the required role, an
:class:`Symfony\\Component\\Security\\Core\\Exception\\AccessDeniedException`
exception is thrown, which you can handle and turn into a nice "access denied"
error page for the user. See :doc:`/cookbook/controller/error_pages` for
more information.

Since Symfony uses the first access control rule it matches, a URL like ``/admin/users/new``
will match the first rule and require only the ``ROLE_SUPER_ADMIN`` role.
Any URL like ``/admin/blog`` will match the second rule and require ``ROLE_ADMIN``.

.. _book-security-securing-ip:

Securing by IP
~~~~~~~~~~~~~~

Certain situations may arise when you may need to restrict access to a given
route based on IP. This is particularly relevant in the case of :ref:`Edge Side Includes<edge-side-includes>`
(ESI), for example, which utilize a route named "_internal". When
ESI is used, the _internal route is required by the gateway cache to enable
different caching options for subsections within a given page. This route
comes with the ^/_internal prefix by default in the standard edition (assuming
you've uncommented those lines from the routing file).

Here is an example of how you might secure this route from outside access:

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

Securing by Channel
~~~~~~~~~~~~~~~~~~~

Much like securing based on IP, requiring the use of SSL is as simple as
adding a new access_control entry:

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

Securing a Controller
~~~~~~~~~~~~~~~~~~~~~

Protecting your application based on URL patterns is easy, but may not be
fine-grained enough in certain cases. When necessary, you can easily force
authorization from inside a controller:

.. code-block:: php

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

You can also choose to install and use the optional ``JMSSecurityExtraBundle``,
which can secure your controller using annotations:

.. code-block:: php

    use JMS\SecurityExtraBundle\Annotation\Secure;

    /**
     * @Secure(roles="ROLE_ADMIN")
     */
    public function helloAction($name)
    {
        // ...
    }

For more information, see the `JMSSecurityExtraBundle`_ documentation. If you're
using Symfony's Standard Distribution, this bundle is available by default.
If not, you can easily download and install it.

Securing other Services
~~~~~~~~~~~~~~~~~~~~~~~

In fact, anything in Symfony can be protected using a strategy similar to
the one seen in the previous section. For example, suppose you have a service
(i.e. a PHP class) whose job is to send emails from one user to another.
You can restrict use of this class - no matter where it's being used from -
to users that have a specific role.

For more information on how you can use the security component to secure
different services and methods in your application, see :doc:`/cookbook/security/securing_services`.

Access Control Lists (ACLs): Securing Individual Database Objects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Imagine you are designing a blog system where your users can comment on your
posts. Now, you want a user to be able to edit his own comments, but not
those of other users. Also, as the admin user, you yourself want to be able
to edit *all* comments.

The security component comes with an optional access control list (ACL) system
that you can use when you need to control access to individual instances
of an object in your system. *Without* ACL, you can secure your system so that
only certain users can edit blog comments in general. But *with* ACL, you
can restrict or allow access on a comment-by-comment basis.

For more information, see the cookbook article: :doc:`/cookbook/security/acl`.

Users
-----

In the previous sections, you learned how you can protect different resources
by requiring a set of *roles* for a resource. In this section we'll explore
the other side of authorization: users.

Where do Users come from? (*User Providers*)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During authentication, the user submits a set of credentials (usually a username
and password). The job of the authentication system is to match those credentials
against some pool of users. So where does this list of users come from?

In Symfony2, users can come from anywhere - a configuration file, a database
table, a web service, or anything else you can dream up. Anything that provides
one or more users to the authentication system is known as a "user provider".
Symfony2 comes standard with the two most common user providers: one that
loads users from a configuration file and one that loads users from a database
table.

Specifying Users in a Configuration File
........................................

The easiest way to specify your users is directly in a configuration file.
In fact, you've seen this already in the example in this chapter.

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

This user provider is called the "in-memory" user provider, since the users
aren't stored anywhere in a database. The actual user object is provided
by Symfony (:class:`Symfony\\Component\\Security\\Core\\User\\User`).

.. tip::
    Any user provider can load users directly from configuration by specifying
    the ``users`` configuration parameter and listing the users beneath it.

.. caution::

    If your username is completely numeric (e.g. ``77``) or contains a dash
    (e.g. ``user-name``), you should use that alternative syntax when specifying
    users in YAML:

    .. code-block:: yaml

        users:
            - { name: 77, password: pass, roles: 'ROLE_USER' }
            - { name: user-name, password: pass, roles: 'ROLE_USER' }

For smaller sites, this method is quick and easy to setup. For more complex
systems, you'll want to load your users from the database.

.. _book-security-user-entity:

Loading Users from the Database
...............................

If you'd like to load your users via the Doctrine ORM, you can easily do
this by creating a ``User`` class and configuring the ``entity`` provider.

.. tip:

    A high-quality open source bundle is available that allows your users
    to be stored via the Doctrine ORM or ODM. Read more about the `FOSUserBundle`_
    on GitHub.

With this approach, you'll first create your own ``User`` class, which will
be stored in the database.

.. code-block:: php

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

As far as the security system is concerned, the only requirement for your
custom user class is that it implements the :class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`
interface. This means that your concept of a "user" can be anything, as long
as it implements this interface.

.. note::

    The user object will be serialized and saved in the session during requests,
    therefore it is recommended that you `implement the \Serializable interface`_
    in your user object. This is especially important if your ``User`` class
    has a parent class with private properties.

Next, configure an ``entity`` user provider, and point it to your ``User``
class:

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

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'providers' => array(
                'main' => array(
                    'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'username'),
                ),
            ),
        ));

With the introduction of this new provider, the authentication system will
attempt to load a ``User`` object from the database by using the ``username``
field of that class.

.. note::
    This example is just meant to show you the basic idea behind the ``entity``
    provider. For a full working example, see :doc:`/cookbook/security/entity_provider`.

For more information on creating your own custom provider (e.g. if you needed
to load users via a web service), see :doc:`/cookbook/security/custom_provider`.

Encoding the User's Password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So far, for simplicity, all the examples have stored the users' passwords
in plain text (whether those users are stored in a configuration file or in
a database somewhere). Of course, in a real application, you'll want to encode
your users' passwords for security reasons. This is easily accomplished by
mapping your User class to one of several built-in "encoders". For example,
to store your users in memory, but obscure their passwords via ``sha1``,
do the following:

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

By setting the ``iterations`` to ``1`` and the ``encode_as_base64`` to false,
the password is simply run through the ``sha1`` algorithm one time and without
any extra encoding. You can now calculate the hashed password either programmatically
(e.g. ``hash('sha1', 'ryanpass')``) or via some online tool like `functions-online.com`_

If you're creating your users dynamically (and storing them in a database),
you can use even tougher hashing algorithms and then rely on an actual password
encoder object to help you encode passwords. For example, suppose your User
object is ``Acme\UserBundle\Entity\User`` (like in the above example). First,
configure the encoder for that user:

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

        // app/config/config.php
        $container->loadFromExtension('security', array(
            // ...

            'encoders' => array(
                'Acme\UserBundle\Entity\User' => 'sha512',
            ),
        ));

In this case, you're using the stronger ``sha512`` algorithm. Also, since
you've simply specified the algorithm (``sha512``) as a string, the system
will default to hashing your password 5000 times in a row and then encoding
it as base64. In other words, the password has been greatly obfuscated so
that the hashed password can't be decoded (i.e. you can't determine the password
from the hashed password).

If you have some sort of registration form for users, you'll need to be able
to determine the hashed password so that you can set it on your user. No
matter what algorithm you configure for your user object, the hashed password
can always be determined in the following way from a controller:

.. code-block:: php

    $factory = $this->get('security.encoder_factory');
    $user = new Acme\UserBundle\Entity\User();

    $encoder = $factory->getEncoder($user);
    $password = $encoder->encodePassword('ryanpass', $user->getSalt());
    $user->setPassword($password);

Retrieving the User Object
~~~~~~~~~~~~~~~~~~~~~~~~~~

After authentication, the ``User`` object of the current user can be accessed
via the ``security.context`` service. From inside a controller, this will
look like:

.. code-block:: php

    public function indexAction()
    {
        $user = $this->get('security.context')->getToken()->getUser();
    }

.. note::

    Anonymous users are technically authenticated, meaning that the ``isAuthenticated()``
    method of an anonymous user object will return true. To check if your
    user is actually authenticated, check for the ``IS_AUTHENTICATED_FULLY``
    role.

Using Multiple User Providers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each authentication mechanism (e.g. HTTP Authentication, form login, etc)
uses exactly one user provider, and will use the first declared user provider
by default. But what if you want to specify a few users via configuration
and the rest of your users in the database? This is possible by creating
a new provider that chains the two together:

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

Now, all authentication mechanisms will use the ``chain_provider``, since
it's the first specified. The ``chain_provider`` will, in turn, try to load
the user from both the ``in_memory`` and ``user_db`` providers.

.. tip::

    If you have no reasons to separate your ``in_memory`` users from your
    ``user_db`` users, you can accomplish this even more easily by combining
    the two sources into a single provider:

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

You can also configure the firewall or individual authentication mechanisms
to use a specific provider. Again, unless a provider is specified explicitly,
the first provider is always used:

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

In this example, if a user tries to login via HTTP authentication, the authentication
system will use the ``in_memory`` user provider. But if the user tries to
login via the form login, the ``user_db`` provider will be used (since it's
the default for the firewall as a whole).

For more information about user provider and firewall configuration, see
the :doc:`/reference/configuration/security`.

Roles
-----

The idea of a "role" is key to the authorization process. Each user is assigned
a set of roles and then each resource requires one or more roles. If the user
has the required roles, access is granted. Otherwise access is denied.

Roles are pretty simple, and are basically strings that you can invent and
use as needed (though roles are objects internally). For example, if you
need to start limiting access to the blog admin section of your website,
you could protect that section using a ``ROLE_BLOG_ADMIN`` role. This role
doesn't need to be defined anywhere - you can just start using it.

.. note::

    All roles **must** begin with the ``ROLE_`` prefix to be managed by
    Symfony2. If you define your own roles with a dedicated ``Role`` class
    (more advanced), don't use the ``ROLE_`` prefix.

Hierarchical Roles
~~~~~~~~~~~~~~~~~~

Instead of associating many roles to users, you can define role inheritance
rules by creating a role hierarchy:

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

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'role_hierarchy' => array(
                'ROLE_ADMIN'       => 'ROLE_USER',
                'ROLE_SUPER_ADMIN' => array('ROLE_ADMIN', 'ROLE_ALLOWED_TO_SWITCH'),
            ),
        ));

In the above configuration, users with ``ROLE_ADMIN`` role will also have the
``ROLE_USER`` role. The ``ROLE_SUPER_ADMIN`` role has ``ROLE_ADMIN``, ``ROLE_ALLOWED_TO_SWITCH``
and ``ROLE_USER`` (inherited from ``ROLE_ADMIN``).

Logging Out
-----------

Usually, you'll also want your users to be able to log out. Fortunately,
the firewall can handle this automatically for you when you activate the
``logout`` config parameter:

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

Once this is configured under your firewall, sending a user to ``/logout``
(or whatever you configure the ``path`` to be), will un-authenticate the
current user. The user will then be sent to the homepage (the value defined
by the ``target`` parameter). Both the ``path`` and ``target`` config parameters
default to what's specified here. In other words, unless you need to customize
them, you can omit them entirely and shorten your configuration:

.. configuration-block::

    .. code-block:: yaml

        logout: ~

    .. code-block:: xml

        <logout />

    .. code-block:: php

        'logout' => array(),

Note that you will *not* need to implement a controller for the ``/logout``
URL as the firewall takes care of everything. You may, however, want to create
a route so that you can use it to generate the URL:

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

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('logout', new Route('/logout', array()));

        return $collection;

Once the user has been logged out, he will be redirected to whatever path
is defined by the ``target`` parameter above (e.g. the ``homepage``). For
more information on configuring the logout, see the
:doc:`Security Configuration Reference</reference/configuration/security>`.

Access Control in Templates
---------------------------

If you want to check if the current user has a role inside a template, use
the built-in helper function:

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

    If you use this function and are *not* at a URL where there is a firewall
    active, an exception will be thrown. Again, it's almost always a good
    idea to have a main firewall that covers all URLs (as has been shown
    in this chapter).

Access Control in Controllers
-----------------------------

If you want to check if the current user has a role in your controller, use
the ``isGranted`` method of the security context:

.. code-block:: php

    public function indexAction()
    {
        // show different content to admin users
        if($this->get('security.context')->isGranted('ADMIN')) {
            // Load admin content here
        }
        // load other regular content here
    }

.. note::

    A firewall must be active or an exception will be thrown when the ``isGranted``
    method is called. See the note above about templates for more details.

Impersonating a User
--------------------

Sometimes, it's useful to be able to switch from one user to another without
having to logout and login again (for instance when you are debugging or trying
to understand a bug a user sees that you can't reproduce). This can be easily
done by activating the ``switch_user`` firewall listener:

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

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => true
                ),
            ),
        ));

To switch to another user, just add a query string with the ``_switch_user``
parameter and the username as the value to the current URL:

    http://example.com/somewhere?_switch_user=thomas

To switch back to the original user, use the special ``_exit`` username:

    http://example.com/somewhere?_switch_user=_exit

Of course, this feature needs to be made available to a small group of users.
By default, access is restricted to users having the ``ROLE_ALLOWED_TO_SWITCH``
role. The name of this role can be modified via the ``role`` setting. For
extra security, you can also change the query parameter name via the ``parameter``
setting:

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

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => array('role' => 'ROLE_ADMIN', 'parameter' => '_want_to_be_this_user'),
                ),
            ),
        ));

Stateless Authentication
------------------------

By default, Symfony2 relies on a cookie (the Session) to persist the security
context of the user. But if you use certificates or HTTP authentication for
instance, persistence is not needed as credentials are available for each
request. In that case, and if you don't need to store anything else between
requests, you can activate the stateless authentication (which means that no
cookie will be ever created by Symfony2):

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

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('http_basic' => array(), 'stateless' => true),
            ),
        ));

.. note::

    If you use a form login, Symfony2 will create a cookie even if you set
    ``stateless`` to ``true``.

Final Words
-----------

Security can be a deep and complex issue to solve correctly in your application.
Fortunately, Symfony's security component follows a well-proven security
model based around *authentication* and *authorization*. Authentication,
which always happens first, is handled by a firewall whose job is to determine
the identity of the user through several different methods (e.g. HTTP authentication,
login form, etc). In the cookbook, you'll find examples of other methods
for handling authentication, including how to implement a "remember me" cookie
functionality.

Once a user is authenticated, the authorization layer can determine whether
or not the user should have access to a specific resource. Most commonly,
*roles* are applied to URLs, classes or methods and if the current user
doesn't have that role, access is denied. The authorization layer, however,
is much deeper, and follows a system of "voting" so that multiple parties
can determine if the current user should have access to a given resource.
Find out more about this and other topics in the cookbook.

Learn more from the Cookbook
----------------------------

* :doc:`Forcing HTTP/HTTPS </cookbook/security/force_https>`
* :doc:`Blacklist users by IP address with a custom voter </cookbook/security/voters>`
* :doc:`Access Control Lists (ACLs) </cookbook/security/acl>`
* :doc:`/cookbook/security/remember_me`

.. _`security component`: https://github.com/symfony/Security
.. _`JMSSecurityExtraBundle`: https://github.com/schmittjoh/JMSSecurityExtraBundle
.. _`FOSUserBundle`: https://github.com/FriendsOfSymfony/FOSUserBundle
.. _`implement the \Serializable interface`: http://php.net/manual/en/class.serializable.php
.. _`functions-online.com`: http://www.functions-online.com/sha1.html

.. toctree::
    :hidden:

    Translation source: 2011-10-04 e6bc8ed
    Corrected from:
