.. index::
   single: Emails; Gmail

Как использовать Gmail для отправки электронных писем
=====================================================

Во время разработки, отправка писем с помощью сервиса Gmail может оказаться 
более легким и практичным решением, нежели использование SMTP сервера.

.. tip::
    Вместо того, чтобы использовать свою учетную запись Gmail, лучшим решением будет
    создать новый аккаунт, применительно для этих целей.    

В конфигурационном файле для среды разработки, измените настройку ``transport`` на
``gmail`` и задайте настройки ``username`` и ``password`` согласно значениям из Gmail:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        swiftmailer:
            transport: gmail
            username:  ваш_gmail_логин
            password:  ваш_gmail_пароль

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->

        <!--
        xmlns:swiftmailer="http://symfony.com/schema/dic/swiftmailer"
        http://symfony.com/schema/dic/swiftmailer http://symfony.com/schema/dic/swiftmailer/swiftmailer-1.0.xsd
        -->

        <swiftmailer:config
            transport="gmail"
            username="ваш_gmail_логин"
            password="ваш_gmail_пароль" />

    .. code-block:: php

        // app/config/config_dev.php
        $container->loadFromExtension('swiftmailer', array(
            'transport' => "gmail",
            'username'  => "ваш_gmail_логин",
            'password'  => "ваш_gmail_пароль",
        ));

На этом настройка Gmail закончена!

.. note::

    Транспорт ``gmail`` является всего лишь готовым шаблоном, который использует транспорт ``smtp`` 
    и устанавливает поля ``encryption``, ``auth_mode`` и ``host`` для работы с почтой Gmail.
