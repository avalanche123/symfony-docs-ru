.. index::
   pair: Forms; View

Формы в шаблонах
================

:doc:`Формы </guides/forms/overview>` в Symfony2 состоят из полей. Поля описывают семантику форм, а не отображение для пользователей; это означает, что форма не обязательно будет привязана к HTML. Вместо этого, на веб-дизайнера ложится ответственность за отображение каждого поля формы. Таким образом, отображение формы в Symfony2 может легко быть сделано вручную. Но Symfony2 облегчает интеграцию и настройку форм, используя помощников для PHP и Twig шаблонов.


Отображение формы вручную
-------------------------
Прежде, чем погружаться в изучение обёрток Symfony2 и того, как они помогут вам с отображением форм - легко, быстро и безопасно, вам следует знать, что внутри системы ничего особенного не происходит. Для отображения форм Symfony2 можно использовать даже HTML:

.. code-block:: html

    <form action="#" method="post">
        <input type="text" name="name" />

        <input type="submit" />
    </form>

Если происходит ошибка валидации, следует отображать её и заполнить поля с представленными значениями для облегчения быстрого исправления проблемы. Просто используйте методы для форм:

.. configuration-block::

    .. code-block:: html+jinja

        <form action="#" method="post">
            <ul>
                {% for error in form.name.errors %}
                    <li>{{ error.0 }}</li>
                {% endfor %}
            </ul>
            <input type="text" name="name" value="{{ form.name.data }}" />

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <form action="#" method="post">
            <ul>
                <?php foreach ($form['name']->getErrors() as $error): ?>
                    <li><?php echo $error[0] ?></li>
                <?php endforeach; ?>
            </ul>
            <input type="text" name="name" value="<?php $form['name']->getData() ?>" />

            <input type="submit" />
        </form>

Использование помощников Symfony2 сделает ваш шаблон коротким, а слой формы - легко настраиваемым, поддерживающим многие локализации, защиту CSRF, загрузку файлов и многое другое сразу "из коробки". Следующие подразделы расскажут вам всё об этом.

Отображение формы
-----------------

Поскольку глобальная структура формы (тэг формы, кнопка "отправить") не определяется самой формой, можно использовать любой HTML код. Простой шаблон формы выглядит следующим образом: 

.. code-block:: html

    <form action="#" method="post">
        <!-- Display the form fields -->

        <input type="submit" />
    </form>

Кроме глобальной структуры форм, вам потребуется способ отображения глобальных ошибок и скрытых файлов. С выполнением этих функций в Symfony2 прекрасно справляются помощники. В шаблонах Twig эти помощники реализованы как глобальные функции, которые могут быть применены к формам и полям форм. В шаблонах PHP помощник "form" реализует тот же функционал, используя публичные методы, которые принимают форму или поле формы за параметр. 

.. configuration-block::

    .. code-block:: html+jinja

        <form action="#" method="post">
            {{ form_errors(form) }}

            <!-- Display the form fields -->

            {{ form_hidden(form) }}
            <input type="submit" />
        </form>

    .. code-block:: html+php

        <form action="#" method="post">
            <?php echo $view['form']->errors($form) ?>

            <!-- Display the form fields -->

            <?php echo $view['form']->hidden($form) ?>

            <input type="submit" />
        </form>

.. Примечание::

Как видите, функции Twig реализуются с префиксом "form\_". В отличие от методов помощника "form", это - глобальные функции, использование которых может привести к конфликтам имён.

.. Совет::
По умолчанию помощник ``error`` создает список ``<ul>``, но это можно легко перенастроить; как - увидите чуть позже.

Последнее, но не по важности: форма, содержащая файл ввода, должна также включать в себя атрибут ``enctype``; используйте помощник ``enctype`` для начала рендеринга:

.. configuration-block::

    .. code-block:: html+jinja

        <form action="#" {{ form_enctype(form) }} method="post">

    .. code-block:: html+php

        <form action="#" <?php echo $view['form']->enctype($form) ?> method="post">

Отображение полей
-----------------
Можно легко получить доступ к полям формы, ибо формы Symfony2 работают как массив:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form.title }}

        {# access a field (first_name) nested in a group (user) #}
        {{ form.user.first_name }}

    .. code-block:: html+php

        <?php $form['title'] ?>

        <!-- access a field (first_name) nested in a group (user) -->
        <?php $form['user']['first_name'] ?>

Поскольку каждое поле является экземпляром Field, оно не может отображаться как показано выше; вместо этого используйте один из существующих помощников.

Помощник ``render`` рендерит HTML-представление поля:

.. configuration-block::

    .. code-block:: jinja

        {{ form_field(form.title) }}

    .. code-block:: html+php

        <?php echo $view['form']->render($form['title']) ?>

.. Примечание::
Шаблон поля выбирается, основываясь на имени класса поля; как - узнаете позже.

Помощник ``label`` рендерит тэг <label>, ассоциированный с полем:

.. configuration-block::

    .. code-block:: jinja

        {{ form_label(form.title) }}

    .. code-block:: html+php

        <?php echo $view['form']->label($form['title']) ?>

По умолчанию Symfony2 "очеловечивает" имя поля, но вы можете дать свое название:

.. configuration-block::

    .. code-block:: jinja

        {{ form_label(form.title, 'Give me a title') }}

    .. code-block:: html+php

        <?php echo $view['form']->label($form['title'], 'Give me a title') ?>

.. note::
    Symfony2 автоматически интернационализирует все названия полей и сообщения об ошибках.

Помощник ``errors`` рендерит все ошибки в полях:

.. configuration-block::

    .. code-block:: jinja

        {{ form_errors(form.title) }}

    .. code-block:: html+php

        <?php echo $view['form']->errors($form['title']) ?>


Определение HTML-отображения
----------------------------

Помощники основываются на шаблонах при рендеринге HTML. По умолчанию Symfony2 выходит с комплектом шаблонов для всех встроенных полей.

В шаблонах Twig каждый помощник ассоциируется с одним блоком шаблона. К примеру, функцие ``form_errors``требуется блок ``errors``. Встроенный блок работает так:

.. code-block:: html+jinja

    {# TwigBundle::form.html.twig #}

    {% block errors %}
        {% if errors %}
        <ul>
            {% for error in errors %}
                <li>{% trans error.0 with error.1 from validators %}</li>
            {% endfor %}
        </ul>
        {% endif %}
    {% endblock errors %}

В PHP-шаблонах, с другой стороны, каждый помощник ассоциируется с одним PHP шаблоном. Помощник ``errors()`` ищет шаблон ``errors.php``, который читается так: 

.. code-block:: html+php

    {# FrameworkBundle:Form:errors.php #}

    <?php if ($errors): ?>
        <ul>
            <?php foreach ($errors as $error): ?>
                <li><?php echo $view['translator']->trans($error[0], $error[1], 'validators') ?></li>
            <?php endforeach; ?>
        </ul>
    <?php endif; ?>

Вот полный список помощников и ассоциирующихся с ними блоков/шаблонов:

========== ================== ==================
Помощник     Блок Twig         Имя шаблона в PHP 
========== ================== ==================
``errors`` ``errors``         ``FrameworkBundle:Form:errors.php``
``hidden`` ``hidden``         ``FrameworkBundle:Form:hidden.php``
``label``  ``label``          ``FrameworkBundle:Form:label.php``
``render`` see below          see below
========== ================== ==================

Помощник ``render`` - слегка другой, потому что он выбирает шаблон для рендеринга, основываясь на версии имени класса поля с подчеркиванием. Например, он будет искать блок ``textarea_field`` или шаблон ``textarea_field.php`` в процессе рендеринга ``TextareaField``:

.. configuration-block::

    .. code-block:: html+jinja

        {# TwigBundle::form.html.twig #}

        {% block textarea_field %}
            <textarea {% display field_attributes %}>{{ field.displayedData }}</textarea>
        {% endblock textarea_field %}

    .. code-block:: html+php

        <!-- FrameworkBundle:Form:textarea_field.php -->
        <textarea id="<?php echo $field->getId() ?>" name="<?php echo $field->getName() ?>" <?php if ($field->isDisabled()): ?>disabled="disabled"<?php endif ?>>
            <?php echo $view->escape($field->getDisplayedData()) ?>
        </textarea>
Если блок или шаблон не существуют, метод будет искать в родительских классах поля. Поэтому нет блока ``collection_field`` по умолчанию - его представление точно такое же, как в его родительском классе (``field_group``).

Настройка представления полей
------------------------------
Самый легкий способ настроить поле - путем передачи пользовательских HTML-атрибутов в качестве аргументов для помощника ``render``:

.. configuration-block::

    .. code-block:: jinja

        {{ form_field(form.title, { 'class': 'important' }) }}

    .. code-block:: html+php

        <?php echo $view['form']->render($form['title'], array(
            'class' => 'important'
        )) ?>

Некоторые поля, например, ``ChoiceField``, принимают параметры для настройки представления поля. Их можно передать в следующем аргументе.

.. configuration-block::

    .. code-block:: jinja

        {{ form_field(form.country, {}, { 'separator': ' -- Other countries -- ' }) }}

    .. code-block:: html+php

        <?php echo $view['form']->render($form['country'], array(), array(
            'separator' => ' -- Other countries -- '
        )) ?>

Все помощники принимают имя шаблона в последнем аргументе, что позволяет вам полностью изменить вывод HTML помощника:

.. configuration-block::

    .. code-block:: jinja

        {{ form_field(form.title, {}, {}, 'HelloBundle::form.html.twig') }}

    .. code-block:: html+php

        <?php echo $view['form']->render($form['title'], array(), array(), 
            'HelloBundle:Form:text_field.php'
        ) ?>

Темы оформления форм (только в Twig)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В последнем примере ``HelloBundle::form.html.twig`` - регулярный шаблон Twig, содержащий блоки, определяющие HTML представление для полей, которые вы хотели бы переопределить:
.. code-block:: html+jinja

    {# HelloBundle/Resources/views/form.html.twig #}

    {% block textarea_field %}
        <div class="textarea_field">
            <textarea {% display field_attributes %}>{{ field.displayedData }}</textarea>
        </div>
    {% endblock textarea_field %}

В этом примере переопределяется блок ``textarea_field``. Вместо того, чтобы заменить представление по умолчанию, вы также можете расширить его, используя нативную особенность наследования Twig:

.. code-block:: html+jinja

    {# HelloBundle/Resources/views/form.html.twig #}

    {% extends 'TwigBundle::form.html.twig' %}

    {% block date_field %}
        <div class="important_date_field">
            {{ parent() }}
        </div>
    {% endblock date_field %}

Если вы хотите настроить все поля данной формы, используйте тэг ``form_theme``:

.. code-block:: jinja

    {% form_theme form 'HelloBundle::form.html.twig' %}

Когда бы вы не вызвали функцию ``form`` - ``form_field``, Symfony2 будет искать представление в вашем шаблоне прежде, чем откатываться до того, что установлен по умолчанию.

Если блоки поля определены в нескольких шаблонах, добавьте их как упорядоченный массив: 

.. code-block:: jinja

    {% form_theme form ['HelloBundle::form.html.twig', 'HelloBundle::form.html.twig', 'HelloBundle::hello_form.html.twig'] %}

Тема может быть присоединена к форме целиком (как выше) или только к группе полей: 

.. code-block:: jinja

    {% form_theme form.user 'HelloBundle::form.html.twig' %}

Наконец, настроить представление всех форм приложения можно с помощью конфигурации:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        twig.config:
            form:
                resources: [BlogBundle::form.html.twig, TwigBundle::form.html.twig]

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <twig:config>
            <twig:form>
                <twig:resource>BlogBundle::form.html.twig</twig:resource>
                <twig:resource>TwigBundle::form.html.twig</twig:resource>
            </twig:form>
        </twig:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('twig', 'config', array('form' => array(
            'resources' => array('BlogBundle::form.html.twig', 'TwigBundle::form.html.twig),
        )));

.. tip::
    Когда бы функция формы или тэг не принимали имя шаблона как аргумент, можно вместо них использовать ``_self`` и определять настройки непосредственно в текущем шаблоне:

    .. code-block:: jinja

        {% form_theme form _self %}

        {% block textarea_field %}
            ...
        {% endblock %}

        {{ form_field(form.description, {}, {}, _self) }}

Прототипы
---------
Когда вы используете прототип формы, можно применять помощник render в форме вместо того, чтобы рендерить вручную все поля:

.. configuration-block::

    .. code-block:: html+jinja

        <form action="#" {{ form_enctype(form) }} method="post">
            {{ form_field(form) }}
            <input type="submit" />
        </form>

    .. code-block:: html+php

        <form action="#" <?php echo $view['form']->enctype($form) ?> method="post">
            <?php echo $view['form']->render($form) ?>

            <input type="submit" />
        </form>

Поскольку нет блока или шаблона, определенного для класса ``Form``, вместо них используется один из родительских классов - ``FieldGroup``:

.. configuration-block::

    .. code-block:: html+jinja

        {# TwigBundle::form.html.twig #}

        {% block field_group %}
            {{ form_errors(field) }}
            {% for child in field %}
                {% if not child.ishidden %}
                    <div>
                        {{ form_label(child) }}
                        {{ form_errors(child) }}
                        {{ form_field(child) }}
                    </div>
                {% endif %}
            {% endfor %}
            {{ form_hidden(field) }}
        {% endblock field_group %}

    .. code-block:: html+php

        <!-- FrameworkBundle:Form:group/table/field_group.php -->

        <?php echo $view['form']->errors($field) ?>

        <div>
            <?php foreach ($field->getVisibleFields() as $child): ?>
                <div>
                    <?php echo $view['form']->label($child) ?>
                    <?php echo $view['form']->errors($child) ?>
                    <?php echo $view['form']->render($child) ?>
                </div>
            <?php endforeach; ?>
        </div>

        <?php echo $view['form']->hidden($field) ?>

.. caution::

    Метод ``render`` - не очень гибкий, и должен использоваться лишь для постройки прототипов.
