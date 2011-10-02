.. index::
   single: Формы; Поля; date

Поле date
===============

Это поле позволяет пользователю изменять информацию о дате при помощи
различных HTML-элементов.

Соответствующие данные для этого поля должны быть в виде объекта ``\DateTime``,
строки, таймштампа или массива. Если опция `input`_ указана верно, поле будет
заботиться о деталях самостоятельно.

Это поле может быть отображено как один текстбокс, три текстбокса (месяц,
день, год) или же три селектбокса (см. опцию `widget_`).

+----------------------+-----------------------------------------------------------------------------+
| Тип данных           | ``\DateTime``, string, timestamp, или array (см. опцию ``input``)           |
+----------------------+-----------------------------------------------------------------------------+
| Отображается как     | один text box или три поля select                                           |
+----------------------+-----------------------------------------------------------------------------+
| Опции                | - `widget`_                                                                 |
|                      | - `input`_                                                                  |
|                      | - `empty_value`_                                                            |
|                      | - `years`_                                                                  |
|                      | - `months`_                                                                 |
|                      | - `days`_                                                                   |
|                      | - `format`_                                                                 |
|                      | - `pattern`_                                                                |
|                      | - `data_timezone`_                                                          |
|                      | - `user_timezone`_                                                          |
+----------------------+-----------------------------------------------------------------------------+
| Родительский тип     | ``field`` (если текст), иначе ``form``                                      |
+----------------------+-----------------------------------------------------------------------------+
| Класс                | :class:`Symfony\\Component\\Form\\Extension\\Core\\Type\\DateType`          |
+----------------------+-----------------------------------------------------------------------------+

Примеры использования
---------------------

Это поле имеет много натроект, но оно просто в использовании. Наиболее
важными опциями являются ``input`` и ``widget``.

Полагая, что у вас есть поле ``publishedAt``, которое содержит дату в
виде обхекта ``DateTime``. Следующий код конфигурирует поле ``date``
для этого поля в вите трёх селектов:

.. code-block:: php

    <?php
    $builder->add('publishedAt', 'date', array(
        'input'  => 'datetime',
        'widget' => 'choice',
    ));

Опция ``input`` *должна* всегда соответствовать типу данных, которые используются
для этого поля. Например, если поле ``publishedAt`` использует unix timestamp,
вам нужно присвоить опции ``input`` значение ``timestamp``:

.. code-block:: php

    <?php
    $builder->add('publishedAt', 'date', array(
        'input'  => 'timestamp',
        'widget' => 'choice',
    ));

Это поле также подерживает ``array`` и ``string`` в качестве валидных опций
для ``input``.

Опции поля
----------

.. include:: /reference/forms/types/options/date_widget.rst.inc

.. _form-reference-date-input:

.. include:: /reference/forms/types/options/date_input.rst.inc

empty_value
~~~~~~~~~~~

**type**: ``string`` | ``array``

Если опция widget имеет значение ``choice``, тогда это поле будет представлено
последовательностью селектбоксов. Опция ``empty_value`` может быть использована
для добавления пустой опции в начале списка селектбокса:

.. code-block:: php

    <?php
    $builder->add('dueDate', 'date', array(
        'empty_value' => '',
    ));

Также вы можете указать строку, которая будет значением "пустой" опции в selectbox:

.. code-block:: php

    <?php
    $builder->add('dueDate', 'date', array(
        'empty_value' => array('year' => 'Year', 'month' => 'Month', 'day' => 'Day')
    ));

.. include:: /reference/forms/types/options/years.rst.inc

.. include:: /reference/forms/types/options/months.rst.inc

.. include:: /reference/forms/types/options/days.rst.inc

.. include:: /reference/forms/types/options/date_format.rst.inc

.. include:: /reference/forms/types/options/date_pattern.rst.inc

.. include:: /reference/forms/types/options/data_timezone.rst.inc

.. include:: /reference/forms/types/options/user_timezone.rst.inc

.. toctree::
    :hidden:

    Translation source: 2011-10-02 d150185
    Corrected from:
