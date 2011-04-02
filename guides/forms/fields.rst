Поля формы
===========

Форма состоит из одного или более полей. Каждое поле является объектом, класс которого реализует :class:`Symfony\\Component\\Form\\FormFieldInterface`. Поля преобразовывают данные из нормализованного представления в человеческое.

Посмотрим, к примеру, на ``DateField``. Хотя ваше приложение хранит даты как строки или объекты ``DateField``, пользователи предпочитают выбирать дату из выпадающего списка. ``DateField`` занимается за вас рендерингом и преобразованием типов.

Параметры полей ядра
--------------------

Все встроенные поля принимают массив свойств в свой конструктор. Для удобства эти поля ядра являются подклассами :class:`Symfony\\Component\\Form\\Field`, который предопределяет несколько свойств. 

``data``
~~~~~~~~
При создании формы каждое поле изначально отображает значение соответствующего объекта доменного объекта формы. Если вы хотите переопределить это первоначальное значение, можете выставить его в свойстве ``data``.

.. code-block:: php

    use Symfony\Component\Form\HiddenField
    
    $field = new HiddenField('token', array(
        'data' => 'abcdef',
    ));
    
    assert('abcdef' === $field->getData());

..Примечание::
    При выставлении опций ``data`` поле не будет писать доменный объект, потому что опция ``property_path`` неявно будет нулевой. Больше информации в :ref:`form-field-property_path`.

``required``
~~~~~~~~~~~~
По умолчанию каждое ``Field`` предполагает, что его значение является обязательным, поэтому должны использоваться не пустые значения. Этот параметр влияет на поведение и рендеринг некоторых полей. Например, ``ChoiceField`` включает в себя пустой выбор, если не является обязательным.

.. code-block:: php

    use Symfony\Component\Form\ChoiceField
    
    $field = new ChoiceField('status', array(
        'choices' => array('tbd' => 'To be done', 'rdy' => 'Ready'),
        'required' => false,
    ));

``disabled``
~~~~~~~~~~~~
Если вы хотите, чтобы пользователь мог изменять значение поля, можете установить опцию ``disabled`` в ``true``. Любые отправленные значения будут игнорироваться.

.. code-block:: php

    use Symfony\Component\Form\TextField
    
    $field = new TextField('status', array(
        'data' => 'Old data',
        'disabled' => true,
    ));
    $field->submit('New data');
    
    assert('Old data' === $field->getData());

``trim``
~~~~~~~~
Многие пользователи случайно вводят начальный или конечный пробел в поля ввода данных. Формы фреймворка автоматически удаляют такие пробелы. Если хотите сохранять их, установите опцию ``trim`` в ``false``.
.. code-block:: php

    use Symfony\Component\Form\TextField
    
    $field = new TextField('status', array(
        'trim' => false,
    ));
    $field->submit('   Data   ');
    
    assert('   Data   ' === $field->getData());

.. _form-field-property_path:


``property_path``
~~~~~~~~~~~~~~~~~
Поля по умолчанию отображают свойства значений доменного объекта формы. При отправке формы, отправляемое значение записывается обратно в объект. 

Если вы хотите переопределить свойство, из которого поле читает и куда впоследствии записывает, можно установить опцию ``property_path``. Ее значение по умолчанию - имя поля.
.. code-block:: php

    use Symfony\Component\Form\Form
    use Symfony\Component\Form\TextField
    
    $author = new Author();
    $author->setFirstName('Your name...');
    
    $form = new Form('author');
    $form->add(new TextField('name', array(
        'property_path' => 'firstName',
    )));
    $form->bind($request, $author);
    
    assert('Your name...' === $form['name']->getData());

Для свойства пути классу доменного объекта необходимы:
1. Совпадающее публичное свойство или
2. Публичные setter и getter с префиксами "set"/"get", следующие за путем свойства.

Пути свойства также могут ссылаться на вложенные объекты при помощи точек. 

.. code-block:: php

    use Symfony\Component\Form\Form
    use Symfony\Component\Form\TextField
    
    $author = new Author();
    $author->getEmail()->setAddress('me@example.com');
    
    $form = new Form('author');
    $form->add(new EmailField('email', array(
        'property_path' => 'email.address',
    )));
    $form->bind($request, $author);
    
    assert('me@example.com' === $form['email']->getData());

Вы можете обращаться к записям вложенных массивов или объектов, реализующих  ``\Traversable``, с помощью квадратных скобок.

.. code-block:: php

    use Symfony\Component\Form\Form
    use Symfony\Component\Form\TextField
    
    $author = new Author();
    $author->setEmails(array(0 => 'me@example.com'));
    
    $form = new Form('author');
    $form->add(new EmailField('email', array(
        'property_path' => 'emails[0]',
    )));
    $form->bind($request, $author);
    
    assert('me@example.com' === $form['email']->getData());

Если свойство пути - ``null``, поле не будет ни читать из доменного объекта, ни писать в него. Это удобно, если вы хотите иметь поля с фиксированными значениями. 

.. code-block:: php

    use Symfony\Component\Form\HiddenField
    
    $field = new HiddenField('token', array(
        'data' => 'abcdef',
        'property_path' => null,
    ));

Поскольку это весьма распространенный сценарий, ``property_path`` всегда ``null``, если вы устанавливаете опцию ``data``. Так что предыдущий пример кода может быть упрощен до:

.. code-block:: php

    use Symfony\Component\Form\HiddenField
    
    $field = new HiddenField('token', array(
        'data' => 'abcdef',
    ));

.. Примечание::
Если вы хотите установить специальное значение, но тем не менее писать в доменный объект, вам придется пропустить ``property_path`` вручную.

    .. code-block:: php

        use Symfony\Component\Form\TextField
        
        $field = new TextField('name', array(
            'data' => 'Custom default...',
            'property_path' => 'token',
        ));

Обычно в этом нет необходимости, поскольку лучше использовать значение по умолчанию свойства ``token`` в вашем доменном объекте.

Встроенные поля
---------------
Symfony2 использует следующие поля: 
.. toctree::
    :hidden:

    fields/index

.. include:: fields/map.rst.inc

