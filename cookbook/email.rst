.. index::
   single: Emails

Как отправлять электронную почту
================================
Рассылка электронной почты, является классической задачей для любого веб-приложения,
и одной из тех задач, в которой имеются определенные сложности и потенциальные
проблемы. Вместо изобретения колеса, одним из решений по рассылке электронной почты, 
является использование пакета ``SwiftmailerBundle``, который использует возможности 
библиотеки `Swiftmailer`_ .

.. note::

    Не забудьте подключить пакет в ядре, до начала его использования::    

        public function registerBundles()
        {
            $bundles = array(
                // ...
                new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            );

            // ...
        }

.. _swift-mailer-configuration:

Настройка
-------------

До момента использования компонента Swiftmailer, его необходимо настроить.
Обязательным в настройке компонента является параметр ``transport``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        swiftmailer:
            transport:  smtp
            encryption: ssl
            auth_mode:  login
            host:       smtp.gmail.com
            username:   ваш_логин
            password:   ваш_пароль

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <!--
        xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
        http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
        -->

        <swiftmailer:config
            transport="smtp"
            encryption="ssl"
            auth-mode="login"
            host="smtp.gmail.com"
            username="ваш_логин"
            password="ваш_пароль" />

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('swiftmailer', array(
            'transport'  => "smtp",
            'encryption' => "ssl",
            'auth_mode'  => "login",
            'host'       => "smtp.gmail.com",
            'username'   => "ваш_логин",
            'password'   => "ваш_пароль",
        ));

Большая часть настроек Swiftmailer отвечает за то каким образом сообщения
должны быть доставлены.

Возможно использовать следующие параметры:

* ``transport``         (``smtp``, ``mail``, ``sendmail``, или ``gmail``)
* ``username``
* ``password``
* ``host``
* ``port``
* ``encryption``        (``tls``, или ``ssl``)
* ``auth_mode``         (``plain``, ``login``, или ``cram-md5``)
* ``spool``

  * ``type`` (каким образом организовывать очередь сообщений, на данный момент поддерживается только способ ``file``)
  * ``path`` (где хранить сообщения)
* ``delivery_address``  (адрес на который отправляют ВСЕ письма)
* ``disable_delivery``  (установка значения в true отключает доставку писем)

Рассылка электронных сообщений
------------------------------

Библиотека Swiftmailer работает с объектами ``Swift_Message``, и занимается их созданием, 
конфигурированием и рассылкой. "Рассылатель" (или mailer) отвечает за доставку сообщений
и доступен через сервис ``mailer``. В целом отправка письма достаточно проста::

    public function indexAction($name)
    {
        // получаем 'mailer' (обязателен для инициализации Swift Mailer)
        $mailer = $this->get('mailer');

        $message = \Swift_Message::newInstance()
            ->setSubject('Hello Email')
            ->setFrom('send@example.com')
            ->setTo('recipient@example.com')
            ->setBody($this->renderView('HelloBundle:Hello:email', array('name' => $name)))
        ;
        $mailer->send($message);

        return $this->render(...);
    }

Отметим, что "тело" письма, хранится в шаблоне и отображается с помощью метода
``renderView()``.

Объект ``$message`` содержит множество других опций, таких как вложения, содержимое в формате HTML,
и т.д. В документации к библиотеке Swiftmailer хорошо освещена глава `Создание сообщений`_ в которой
можно найти информацию о том как создавать сообщения и опциях которые при этом доступны.

.. tip::

    Рекомендуем прочитать документ ":doc:`gmail`" в котором рассказано как использовать почту
    gmail в качестве транспорта на стадии разработки.

.. _`Swiftmailer`: http://www.swiftmailer.org/
.. _`Создание сообщений`: http://swiftmailer.org/docs/messages