CollectionField
===============

``CollectionField`` - особая группа полей для управления массивами или объектами, которые реализуют интерфейс ``Traversable``. Чтобы продемонстрировать это, мы расширим класс ``Customer`` до хранения трех e-mail адресов::

    class Customer
    {
        // other properties ...

        public $emails = array('', '', '');
    }

А теперь добавим ``CollectionField`` для управления этими адресами: 

use Symfony\Component\Form\CollectionField;

    $form->add(new CollectionField('emails', array(
        'prototype' => new EmailField(),
    )));

Если установить опцию "modifiable" в ``true``, вы сможете даже добавлять или удалять строки в набор с помощью Javascript! ``CollectionField`` заметит это и соответственно изменит размер основного массива.


