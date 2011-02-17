RepeatedField
=============

``RepeatedField`` - обширная группа полей, которая позволяет выводить поле дважды. Повторяемое поле будет проверяться только если пользователь введет одно и то же значение в оба поля::

    use Symfony\Component\Form\RepeatedField;

    $form->add(new RepeatedField(new TextField('email')));

Это весьма удобное поле для проверки e-mail адресов или паролей!
