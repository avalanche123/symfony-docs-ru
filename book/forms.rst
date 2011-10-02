.. index::
   single: Формы

Формы
=====

Работа с формами - одна из наиболее типичных и проблемных задач для web-разработчика.
Symfony2 включает компонент для работы с Формами, который облегчает работу с ними.
В этой главе вы узнаете, как создавать сложные формы с нуля, познакомитесь с
наиболее важными особенностями библиотеки форм.

.. note::

   Компонент для работы с формами - это независимая библиотека, которая может
   быть использована вне проектов Symfony2. Подробности ищите по ссылке
   `Symfony2 Form Component`_ на ГитХабе.

.. index::
   single: Формы; Создание простой формы

Создание простой формы
----------------------

Предположим вы работаете над простым приложением - списком ToDo, которое
будет отображать некоторые "задачи". Поскольку вашим пользователям будет
необходимо создавать и редактировать задачи, вам потебуется создать форму.
Но, прежде чем начать, давайте создадим базовый класс ``Task``, который
представляет и хранит данные для одной задачи:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Entity/Task.php
    namespace Acme\TaskBundle\Entity;

    class Task
    {
        protected $task;

        protected $dueDate;

        public function getTask()
        {
            return $this->task;
        }
        public function setTask($task)
        {
            $this->task = $task;
        }

        public function getDueDate()
        {
            return $this->dueDate;
        }
        public function setDueDate(\DateTime $dueDate = null)
        {
            $this->dueDate = $dueDate;
        }
    }

.. note::

   Если вы будете брать код примеров один в один, вам прежде всего
   необходимо создать пакет ``AcmeTaskBundle``, выполнив следующую команду
   (и принимая все опции интерактивного генератора по умолчанию):

   .. code-block:: bash

        php app/console generate:bundle --namespace=Acme/TaskBundle

Этот клас представляет собой обычный PHP-объект и не имеет ничего общего с
Symfony или какой-либо другой библиотекой. Это PHP-объект, который выполняет
задачу непосредственно внутри *вашего* приложение (т.е. является представлением
задачи в вашем приложении). Конечно же, к концу этой главы вы будете иметь
возможность возможность отправлять данные для экземпляра ``Task`` (посредством
HTML-формы), валидировать её данные и сохранять в базу данных.

.. index::
   single: Формы; Создание формы в контроллере

Создание формы
~~~~~~~~~~~~~~~~~

Теперь, когда вы создали класс ``Task``, следующим шагом будет создание и
отображение HTML-формы. В Symfony2 это выполняется посредством создания
объекта формы и отображения его в шаблоне. Теперь, выполним необходимые
действия в контроллере:

.. code-block:: php

    <?php
    // src/Acme/TaskBundle/Controller/DefaultController.php
    namespace Acme\TaskBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Acme\TaskBundle\Entity\Task;
    use Symfony\Component\HttpFoundation\Request;

    class DefaultController extends Controller
    {
        public function newAction(Request $request)
        {
            // создаём задачу и приваиваем ей некоторые начальные данные для примера
            $task = new Task();
            $task->setTask('Write a blog post');
            $task->setDueDate(new \DateTime('tomorrow'));

            $form = $this->createFormBuilder($task)
                ->add('task', 'text')
                ->add('dueDate', 'date')
                ->getForm();

            return $this->render('AcmeTaskBundle:Default:new.html.twig', array(
                'form' => $form->createView(),
            ));
        }
    }

.. tip::

   Этот пример показывает, как создать вашу форму непосредственно в коде
   вашего контроллера. Позднее, в секции :ref:`book-form-creating-form-classes`,
   вы также узнаете как создавать формы в отдельных классах, что является
   более предпочтительным вариантом и сделает ваши формы доступными для
   повторного использования.

Создание формы требует совсем немного кода, так как объекты форм в Symfony2
создаются при помощи конструктора форм - "form builder". Цель конструктора
форм - облегчить насколько это возможно создание форм, выполняя всю тяжелую
работу.

В этом примере вы добавили два поля в вашу форму - ``task`` и ``dueDate``,
соответствующие полям ``task`` и ``dueDate`` класса ``Task``. Вы также
указали каждому полю их типы (например ``text``, ``date``), которые в числе
прочих параметров, определяют - какой HTML таг будет отображен для этого
поля в форме.

Symfony2 включает много встроенных типов, которые будут обсуждаться
совсем скоро (см. :ref:`book-forms-type-reference`).

.. index::
  single: Формы; Отображение формы в шаблоне

Отображение формы
~~~~~~~~~~~~~~~~~~

Теперь, когда форма создана, следующим шагом будет её отображение. Отобразить
форму можно, передав специальный объект "form view" в ваш шаблон (обратите
внимание на конструкцию ``$form->createView()`` в контроллере выше) и
использовать ряд функций-помощников в шаблоне:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        <form action="{{ path('task_new') }}" method="post" {{ form_enctype(form) }}>
            {{ form_widget(form) }}

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/new.html.php -->

        <form action="<?php echo $view['router']->generate('task_new') ?>" method="post" <?php echo $view['form']->enctype($form) ?> >
            <?php echo $view['form']->widget($form) ?>

            <input type="submit" />
        </form>

.. image:: /images/book/form-simple.png
    :align: center

.. note::

    В этом примере предполагается, что вы создали маршрут ``task_new``,
    который указывает на контроллер ``AcmeTaskBundle:Default:new``,
    который был создан ранее.

Вот и всё! Напечатав ``form_widget(form)``, каждое поле формы будет отображено,
так же как метки полей и ошибки (если они есть). Это очень просто, но не
очень гибко (пока что). На практике вам скорее всего захочется отобразить
каждое поле формы отдельно, чтобы иметь полный контроль над тем как форма
выглядит. Вы узнаете как сделать это в секции ":ref:`form-rendering-template`".

Прежде чем двигаться дальше, обратите внимание на то как было отображено поле
``task``, содержащее значение поля ``task`` объекта ``$task` ("Write a blog post").
Это - первая задача форм: получить данные от объекта и перевести их в формат,
подходящий для их последующего отображения в HTML форме.

.. tip::

   Система форм достаточно умна, чтобы получить доступ к значению защищённого
   (protected) поля ``task`` через методы ``getTask()`` и ``setTask()`` класса
   ``Task``. Так как поле не публичное (public), оно должно иметь "геттер" и
   "сеттер" методы для того, чтобы компонент форм мог получить данные из
   этого поля и изменить их. Для булевых полей вы также можете использовать
   "is*" метод (например ``isPublished()``) вместо ``getPublished()``.

.. index::
  single: Формы; Обработка отправки форм

Обработка отправки форм
~~~~~~~~~~~~~~~~~~~~~~~

Второй обязанностью форм является перевод данных, отправленных пользователем,
в свойства объекта. Для того чтобы это произошло, отправленные данные должны
быть привязаны к форме. Добавьте в контроллер следующие строки:

.. code-block:: php

    <?php
    // ...

    public function newAction(Request $request)
    {
        // создаём новый объект $task (без данных по умолчанию)
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->getForm();

        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // выполняем прочие действие, например сохраняем задачу в базе данных

                return $this->redirect($this->generateUrl('task_success'));
            }
        }

        // ...
    }

Теперь, при отправке формы контроллер привязывает отправленные данные к
форме, которая присваивает эти данные полям ``task`` и ``dueDate`` объекта
``$task``. Эта задача выполняется методом ``bindRequest()``.

.. note::

    Как только вызывается метод ``bindRequest()``, отправленные данные
    тут же присваиваются соответствующему объекту формы. Это происходит вне
    зависимости от того, валидны ли эти данные или нет.

Этот контроллер следует типичному сценарию по обработке форм и имеет три возможных
пути:

#. При первичной загрузке страницы в браузер метод запроса будет ``GET``,
   форма лишь создаётся и отображается;

#. Когда пользователь отправляет форму (т.е. метод будет уже ``POST``) с
   неверными данными (вопросы валидации будут рассмотрены ниже, а пока просто
   предположим что данные не валидны), форма будет привязана к данным и
   отображена вместе со всеми ошибками валидации;

#. Когда пользователь отправляет форму с валидными данными, форма будет
   привязана к данным и у вас есть возможность для выполнения некоторых
   действий, используя объект ``$task`` (например сохранить его в базе
   данных) перед тем как перенаправить пользователя на другую страницу
   (например "thank you" или "success").

.. note::

   Перенаправление пользователя после успешной отправки формы предотвращает
   повторную отправку этих же данных, если пользователь обновит страницу.

.. index::
   single: Формы; Валидация

Валидация форм
--------------

В предыдущей секции вы узнали что форма может быть отправлена с валидными
или не валидными даными. В Symfony2 валидация применяется к объекту, лежащему
в основе формы (например, ``Task``). Другими словами, вопрос не в том, валидна
ли форма, а валиден ли объект ``$task``, после того как форма передала
ему отправленные данные. Выполнив метод ``$form->isValid()``, можно узнать
валидны ли данные объекта ``$task`` или же нет.

Валидация выполняется посредством добавление набора правил (называемых ограничениями)
к классу. Для того, чтобы увидеть валидацию в действии, добавьте ограничения для
валидации того, что поле ``task`` не может быть пусто, а поле ``dueDate`` не может
быть пусто и должно содержать объект ``\DateTime``.

.. configuration-block::

    .. code-block:: yaml

        # Acme/TaskBundle/Resources/config/validation.yml
        Acme\TaskBundle\Entity\Task:
            properties:
                task:
                    - NotBlank: ~
                dueDate:
                    - NotBlank: ~
                    - Type: \DateTime

    .. code-block:: php-annotations

        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Task
        {
            /**
             * @Assert\NotBlank()
             */
            public $task;

            /**
             * @Assert\NotBlank()
             * @Assert\Type("\DateTime")
             */
            protected $dueDate;
        }

    .. code-block:: xml

        <!-- Acme/TaskBundle/Resources/config/validation.xml -->
        <class name="Acme\TaskBundle\Entity\Task">
            <property name="task">
                <constraint name="NotBlank" />
            </property>
            <property name="dueDate">
                <constraint name="NotBlank" />
                <constraint name="Type">
                    <value>\DateTime</value>
                </constraint>
            </property>
        </class>

    .. code-block:: php

        <?php
        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\Type;

        class Task
        {
            // ...

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('task', new NotBlank());

                $metadata->addPropertyConstraint('dueDate', new NotBlank());
                $metadata->addPropertyConstraint('dueDate', new Type('\DateTime'));
            }
        }

Это всё! Если вы отправите форму с ошибочными значениями - вы увидите
что соответствующие ошибки будут отображены в форме.

.. _book-forms-html5-validation-disable:

.. sidebar:: HTML5 Валидация

   Начиная с HTML5, многие браузеры могут выполнять некоторые валидационные
   ограничения на строне клиента, без отправки формы. Типичным ограничением
   является указание атрибута ``required`` для полей, которые будут
   обязательными. В браузерах, которые поддерживают HTML5, этот атрибут будет
   позволять отображать браузерное сообщение об ошибке, если пользователь
   попробует отправить форму с пустым соответствующим полем.

   Генерированные формы полностью поддерживают эту фозможность, добавляя
   соответствующие HTML-атрибуты, которые активируют HTML5 клиентскую
   валидацию. Тем не менее, валидация на стороне клиента может
   быть отключена путём добавления атрибута ``novalidate`` к тагу
   ``form`` или ``formnovalidate`` к тагу ``submit``. Это бывает необходимо,
   когда вам нужно протестировать ваши серверные ограничения, но,
   к примеру, браузер не даёт отправить форму с пустыми полями.

Валидация - это важная функция в составе Symfony2, она описывается
в :doc:`отдельной главе</book/validation>`.

.. index::
   single: Формы; Валидационные группы

.. _book-forms-validation-groups:

Валидационные группы
~~~~~~~~~~~~~~~~~~~~

.. tip::

    Если вы не испольуете :ref:`валидационные группы <book-validation-validation-groups>`,
    вы можете пропустить эту секцию.

Если ваш объект использует возможности :ref:`валидационных групп <book-validation-validation-groups>`,
вам нужно указать, какие группы вы хотите использовать:

.. code-block:: php

    <?php
    // ...

    $form = $this->createFormBuilder($users, array(
        'validation_groups' => array('registration'),
    ))->add(...)
    ;

Если вы создаёте :ref:`классы форм<book-form-creating-form-classes>`
(хорошая практика), тогда вам нужно указать следуюущий код в метод
``getDefaultOptions()``:

.. code-block:: php

    <?php
    // ...

    public function getDefaultOptions(array $options)
    {
        return array(
            'validation_groups' => array('registration')
        );
    }

В обоих этих примерах, для валидации объекта, для котрого создана форма, будет
использована *лишь* группа ``registration``.

.. index::
   single: Формы; Встроенные типы полей

.. _book-forms-type-reference:

Встроенные типы полей
---------------------

В состав Symfony входит большое число типов полей, которые покрывают все
типичные поля и типы данных, с которыми вы столкнётесь:

.. include:: /reference/forms/types/map.rst.inc

Вы также можете создавать свои собственные типы полей. Эта возможность
подробно описывается в статье книги рецептов ":doc:`/cookbook/form/create_custom_field_type`".

.. index::
   single: Формы; Опции полей форм

Опции полей форм
~~~~~~~~~~~~~~~~

Каждый тип поля имеет некоторое число опций, которые можно использовать для
их настройки. Например, поле ``dueDate`` сейчас отображает 3 селектбокса.
Тем не менее, :doc:`поле date</reference/forms/types/date>` можно натроить
таким образом, чтобы отображался один текстбокс (где пользователь сможет
ввести дату в виде строки):

.. code-block:: php

    <?php
    // ...
    ->add('dueDate', 'date', array('widget' => 'single_text'))

.. image:: /images/book/form-simple2.png
    :align: center

Все типы полей имеют много различных опций, которые могут быть для них
указаны. Многие из этих опций - специфичны для конкретного типа поля и
более подробную информацию о них можно получить из справочника.

.. sidebar:: Опция ``required``

    Наиболее типичной опцией является опция ``required``, которая может быть
    указана для любого поля. По умолчанию, опция ``required`` установлена в
    ``true``, что даёт возможность браузерам с поддержкой HTML5 использовать
    втроенную в них клиентскую валидацию, если поле остаётся пустым. Если вам
    этого не требуется, или же установите опцию ``required`` в ``false`` или
    же :ref:`отключите валидацию HTML5<book-forms-html5-validation-disable>`.

    Отметим также, что устновка опции ``required`` в ``true`` **не влияет**
    на серверную валидацию. Другими словами, если пользователь отправляет
    пустое значение для этого поля (при помощи старого браузера или веб-сервиса)
    оно будет считаться валидным, если вы не используете ограничения ``NotBlank``
    или ``NotNull``.

    Таким образом, опция ``required`` - хороша, но серверную валидацию
    использовать *необходимо всегда*.

.. index::
   single: Формы; Автоматическое определение типов полей

.. _book-forms-field-guessing:

Автоматическое определение типов полей
--------------------------------------

Теперь, когда вы добавили данные для валидации в класс ``Task``, Symfony
теперь много знает о ваших полях. Если вы позволите, Symfony может
определять ("угадывать") тип вашего поля и устанавливать его автоматически.
В этом примере, Symfony может определить по правилам валидации, что
``task`` является полем типа ``text`` и ``dueDate`` - полем типа ``date``:

.. code-block:: php

    <?php
    public function newAction()
    {
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->getForm();
    }

Автоматическое определение активируется, когда вы опускаете второй аргумент
в методе ``add()`` (или если вы указываете null для него). Если вы передаёте
массив опций в качестве третьего аргумента (как в случае ``dueDate`` выше),
эти опции применяются к "угаданному" полю.

.. caution::

    Если ваша форма использует особую группу валидации, определитель
    типов полей будет учитывать *все* ограничения при определении
    типов полей (включая ограничения, которые не являются частью
    используемых валидационных групп).

.. index::
   single: Формы; Автоматическое определение типов полей

Автоматическое определение опций для полей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В дополнение к определению "типа" поля, Symfony также может попытаться
определить значения опций для поля.

.. tip::

    Когда эти опции будут установлены, поле будет отображено с
    использованием особых HTML атрибутов, которые позволяют выполнять
    HTML5 валидацию на стороне клиета (например ``Assert\MaxLength``).
    И, поскольку вам нужно будет вручную добавлять правила валидации
    на стороне сервера, эти опции могут быть угаданы исходя из ограничений,
    которые вы будете использовать для неё.

* ``required``: Опция ``required`` может быть определена исходя из правил
  валидации (т.е. если поле ``NotBlank`` или ``NotNull``) или же на основании
  метаданных Doctrine (т.е. если поле ``nullable``). Это может быть очень удобно,
  так как плавила клиентской валидации автоматически соответствуют правилам
  серверной валидации.

* ``min_length``: Если поле является одним из видов текстовых полей,
  опция ``min_length`` может быть угадана исходя из правил валидации (
  если используются ограничения ``MinLength`` или ``Min``) или же из
  метаданых Doctrine (основываясь на длинне поля).

* ``max_length``: Аналогично ``min_length`` с той лишь разницей, что определяет
  максимальное значение длины поля.

.. note::

  Эти опции могут быть определены автоматически, *только* если вы используете
  автоопределение полей (не указываете или передаёте ``null`` в качестве второго
  аргумента в метод ``add()``).

Если вы хотите изменить значения, определённые автоматически, вы можете
перезаписать их, передавая требуемые опции в массив опций::

    ->add('task', null, array('min_length' => 4))


.. index::
   single: Формы; Отображение в шаблоне

.. _form-rendering-template:

Отображение формы в шаблоне
------------------------------

Ранее вы увидели как форму целиком можно отобразить при помощи лишь одной
строки кода. Конечно же, на практике вам потребуется большая гибкость при
отображении:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        <form action="{{ path('task_new') }}" method="post" {{ form_enctype(form) }}>
            {{ form_errors(form) }}

            {{ form_row(form.task) }}
            {{ form_row(form.dueDate) }}

            {{ form_rest(form) }}

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <!-- // src/Acme/TaskBundle/Resources/views/Default/newAction.html.php -->

        <form action="<?php echo $view['router']->generate('task_new') ?>" method="post" <?php echo $view['form']->enctype($form) ?>>
            <?php echo $view['form']->errors($form) ?>

            <?php echo $view['form']->row($form['task']) ?>
            <?php echo $view['form']->row($form['dueDate']) ?>

            <?php echo $view['form']->rest($form) ?>

            <input type="submit" />
        </form>

Давайте более подробно рассмотрим каждую часть:

* ``form_enctype(form)`` - если хоть одно поле формы является полем для загрузки
  файла, эта функция отобразит необходимый атрибут ``enctype="multipart/form-data"``;

* ``form_errors(form)`` - Отображает глобальные по отношению к форме целиком ошибки
  валидации (ошибки для полей отображаются после них);

* ``form_row(form.dueDate)`` - Отображает текстовую метку, ошибки и HTML-виджет
  для заданного поля (например для ``dueDate``) внутри ``div`` элемента
  (по умолчанию);

* ``form_rest(form)`` - Отображает все остальные поля, которые ещё не были
  отображены. Как правило хорошая идея расположить вызов этого хелпера внизу
  каждой формы (на случай если вы забыли вывести какое-либо поле или же
  не хотите вручную отображать скрытые поля). Этот хелпер также удобен для
  активации автоматической защиты от :ref:`CSRF атак<forms-csrf>`.

Основная часть работы сделана при помощи хелпера ``form_row``, который
отображает метку, ошибки и виджет для каждого поля внутри тага ``div``.
В секции :ref:`form-theming` вы узнаете как можно настроить вывод ``form_row``
на различных уровнях.

.. tip::

    Вы можете получить доступ к данным вашей формы при помощи ``form.vars.value``:

    .. configuration-block::

        .. code-block:: jinja

            {{ form.vars.value.task }}

        .. code-block:: html+php

            <?php echo $view['form']->get('value')->getTask() ?>

.. index::
   single: Формы; Отображение каждого поля вручную

Отображение каждого поля вручную
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Хелпер ``form_row`` очень удобен, так как вы можете быстро отобразить
каждое поле вашей формы (и разметка, используемая для каждой строки
может быть настроена). Но, так как жизнь как правило сложнее чем нам
хотелось бы, вы можете также отобразить каждое поле вручную. В конечном
итоге вы получите тоже самое что и с хелпером ``form_row``:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_errors(form) }}

        <div>
            {{ form_label(form.task) }}
            {{ form_errors(form.task) }}
            {{ form_widget(form.task) }}
        </div>

        <div>
            {{ form_label(form.dueDate) }}
            {{ form_errors(form.dueDate) }}
            {{ form_widget(form.dueDate) }}
        </div>

        {{ form_rest(form) }}

    .. code-block:: html+php

        <?php echo $view['form']->errors($form) ?>

        <div>
            <?php echo $view['form']->label($form['task']) ?>
            <?php echo $view['form']->errors($form['task']) ?>
            <?php echo $view['form']->widget($form['task']) ?>
        </div>

        <div>
            <?php echo $view['form']->label($form['dueDate']) ?>
            <?php echo $view['form']->errors($form['dueDate']) ?>
            <?php echo $view['form']->widget($form['dueDate']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>

Если автоматически созданная метка для поля вам не нравится, вы можете указать её
явно:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_label(form.task, 'Task Description') }}

    .. code-block:: html+php

        <?php echo $view['form']->label($form['task'], 'Task Description') ?>

Наконец, некоторые типы полей имеют дополнительные опции отображения,
которые можно указывать виджету. Эти опции документированы для каждого
такого типа, но общей для всех опцией является ``attr``, которая позволяет
вам модифицировать атрибуты элемента формы. Следущий пример добавит текстовому
полю CSS класс ``task_field``:

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_widget(form.task, { 'attr': {'class': 'task_field'} }) }}

    .. code-block:: html+php

        <?php echo $view['form']->widget($form['task'], array(
            'attr' => array('class' => 'task_field'),
        )) ?>

Справочник по функциям Twig
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если вы используете Twig, полная справочная информация о функциях, используемых
для отображения форм, доступна в :doc:`справочнике</reference/forms/twig_reference>`.
Ознакомьтесь с этой информацией, для того чтобы узнать больше о доступных хелперах
и опциях, которые для них доступны.

.. index::
   single: Forms; Creating form classes

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. _book-form-creating-form-classes:

Creating Form Classes
---------------------

As you've seen, a form can be created and used directly in a controller.
However, a better practice is to build the form in a separate, standalone PHP
class, which can then be reused anywhere in your application. Create a new class
that will house the logic for building the task form:

.. code-block:: php

    // src/Acme/TaskBundle/Form/Type/TaskType.php

    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class TaskType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('task');
            $builder->add('dueDate', null, array('widget' => 'single_text'));
        }

        public function getName()
        {
            return 'task';
        }
    }

This new class contains all the directions needed to create the task form
(note that the ``getName()`` method should return a unique identifier for this
form "type"). It can be used to quickly build a form object in the controller:

.. code-block:: php

    // src/Acme/TaskBundle/Controller/DefaultController.php

    // add this new use statement at the top of the class
    use Acme\TaskBundle\Form\Type\TaskType;

    public function newAction()
    {
        $task = // ...
        $form = $this->createForm(new TaskType(), $task);

        // ...
    }

Placing the form logic into its own class means that the form can be easily
reused elsewhere in your project. This is the best way to create forms, but
the choice is ultimately up to you.

.. _book-forms-data-class:

.. sidebar:: Setting the ``data_class``

    Every form needs to know the name of the class that holds the underlying
    data (e.g. ``Acme\TaskBundle\Entity\Task``). Usually, this is just guessed
    based off of the object passed to the second argument to ``createForm``
    (i.e. ``$task``). Later, when you begin embedding forms, this will no
    longer be sufficient. So, while not always necessary, it's generally a
    good idea to explicitly specify the ``data_class`` option by add the
    following to your form type class::

        public function getDefaultOptions(array $options)
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Task',
            );
        }

.. index::
   pair: Forms; Doctrine

Forms and Doctrine
------------------

The goal of a form is to translate data from an object (e.g. ``Task``) to an
HTML form and then translate user-submitted data back to the original object. As
such, the topic of persisting the ``Task`` object to the database is entirely
unrelated to the topic of forms. But, if you've configured the ``Task`` class
to be persisted via Doctrine (i.e. you've added
:ref:`mapping metadata<book-doctrine-adding-mapping>` for it), then persisting
it after a form submission can be done when the form is valid::

    if ($form->isValid()) {
        $em = $this->getDoctrine()->getEntityManager();
        $em->persist($task);
        $em->flush();

        return $this->redirect($this->generateUrl('task_success'));
    }

If, for some reason, you don't have access to your original ``$task`` object,
you can fetch it from the form::

    $task = $form->getData();

For more information, see the :doc:`Doctrine ORM chapter</book/doctrine>`.

The key thing to understand is that when the form is bound, the submitted
data is transferred to the underlying object immediately. If you want to
persist that data, you simply need to persist the object itself (which already
contains the submitted data).

.. index::
   single: Forms; Embedded forms

Embedded Forms
--------------

Often, you'll want to build a form that will include fields from many different
objects. For example, a registration form may contain data belonging to
a ``User`` object as well as many ``Address`` objects. Fortunately, this
is easy and natural with the form component.

Embedding a Single Object
~~~~~~~~~~~~~~~~~~~~~~~~~

Suppose that each ``Task`` belongs to a simple ``Category`` object. Start,
of course, by creating the ``Category`` object::

    // src/Acme/TaskBundle/Entity/Category.php
    namespace Acme\TaskBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Category
    {
        /**
         * @Assert\NotBlank()
         */
        public $name;
    }

Next, add a new ``category`` property to the ``Task`` class::

    // ...

    class Task
    {
        // ...

        /**
         * @Assert\Type(type="Acme\TaskBundle\Entity\Category")
         */
        protected $category;

        // ...

        public function getCategory()
        {
            return $this->category;
        }

        public function setCategory(Category $category = null)
        {
            $this->category = $category;
        }
    }

Now that your application has been updated to reflect the new requirements,
create a form class so that a ``Category`` object can be modified by the user::

    // src/Acme/TaskBundle/Form/Type/CategoryType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class CategoryType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('name');
        }

        public function getDefaultOptions(array $options)
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Category',
            );
        }

        public function getName()
        {
            return 'category';
        }
    }

The end goal is to allow the ``Category`` of a ``Task`` to be modified right
inside the task form itself. To accomplish this, add a ``category`` field
to the ``TaskType`` object whose type is an instance of the new ``CategoryType``
class:

.. code-block:: php

    public function buildForm(FormBuilder $builder, array $options)
    {
        // ...

        $builder->add('category', new CategoryType());
    }

The fields from ``CategoryType`` can now be rendered alongside those from
the ``TaskType`` class. Render the ``Category`` fields in the same way
as the original ``Task`` fields:

.. configuration-block::

    .. code-block:: html+jinja

        {# ... #}

        <h3>Category</h3>
        <div class="category">
            {{ form_row(form.category.name) }}
        </div>

        {{ form_rest(form) }}
        {# ... #}

    .. code-block:: html+php

        <!-- ... -->

        <h3>Category</h3>
        <div class="category">
            <?php echo $view['form']->row($form['category']['name']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>
        <!-- ... -->

When the user submits the form, the submitted data for the ``Category`` fields
are used to construct an instance of ``Category``, which is then set on the
``category`` field of the ``Task`` instance.

The ``Category`` instance is accessible naturally via ``$task->getCategory()``
and can be persisted to the database or used however you need.

Embedding a Collection of Forms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also embed a collection of forms into one form. This is done by
using the ``collection`` field type. For more information, see the
:doc:`collection field type reference</reference/forms/types/collection>`.

.. index::
   single: Forms; Theming
   single: Forms; Customizing fields

.. _form-theming:

Form Theming
------------

Every part of how a form is rendered can be customized. You're free to change
how each form "row" renders, change the markup used to render errors, or
even customize how a ``textarea`` tag should be rendered. Nothing is off-limits,
and different customizations can be used in different places.

Symfony uses templates to render each and every part of a form, such as
``label`` tags, ``input`` tags, error messages and everything else.

In Twig, each form "fragment" is represented by a Twig block. To customize
any part of how a form renders, you just need to override the appropriate block.

In PHP, each form "fragment" is rendered via an individual template file.
To customize any part of how a form renders, you just need to override the
existing template by creating a new one.

To understand how this works, let's customize the ``form_row`` fragment and
add a class attribute to the ``div`` element that surrounds each row. To
do this, create a new template file that will store the new markup:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Form/fields.html.twig #}

        {% block field_row %}
        {% spaceless %}
            <div class="form_row">
                {{ form_label(form) }}
                {{ form_errors(form) }}
                {{ form_widget(form) }}
            </div>
        {% endspaceless %}
        {% endblock field_row %}

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Form/field_row.html.php -->

        <div class="form_row">
            <?php echo $view['form']->label($form, $label) ?>
            <?php echo $view['form']->errors($form) ?>
            <?php echo $view['form']->widget($form, $parameters) ?>
        </div>

The ``field_row`` form fragment is used when rendering most fields via the
``form_row`` function. To tell the form component to use your new ``field_row``
fragment defined above, add the following to the top of the template that
renders the form:

.. configuration-block:: php

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        {% form_theme form 'AcmeTaskBundle:Form:fields.html.twig' %}

        <form ...>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/new.html.php -->

        <?php $view['form']->setTheme($form, array('AcmeTaskBundle:Form')) ?>

        <form ...>

The ``form_theme`` tag (in Twig) "imports" the fragments defined in the given
template and uses them when rendering the form. `In other words, when the
``form_row`` function is called later in this template, it will use the ``field_row``
block from your custom theme (instead of the default ``field_row`` block
that ships with Symfony).

To customize any portion of a form, you just need to override the appropriate
fragment. Knowing exactly which block or file to override is the subject of
the next section.

For a more extensive discussion, see :doc:`/cookbook/form/form_customization`.

.. index::
   single: Forms; Template fragment naming

.. _form-template-blocks:

Form Fragment Naming
~~~~~~~~~~~~~~~~~~~~

In Symfony, every part a form that is rendered - HTML form elements, errors,
labels, etc - is defined in a base theme, which is a collection of blocks
in Twig and a collection of template files in PHP.

In Twig, every block needed is defined in a single template file (`form_div_layout.html.twig`_)
that lives inside the `Twig Bridge`_. Inside this file, you can see every block
needed to render a form and every default field type.

In PHP, the fragments are individual template files. By default they are located in
the `Resources/views/Form` directory of the framework bundle (`view on GitHub`_).

Each fragment name follows the same basic pattern and is broken up into two pieces,
separated by a single underscore character (``_``). A few examples are:

* ``field_row`` - used by ``form_row`` to render most fields;
* ``textarea_widget`` - used by ``form_widget`` to render a ``textarea`` field
  type;
* ``field_errors`` - used by ``form_errors`` to render errors for a field;

Each fragment follows the same basic pattern: ``type_part``. The ``type`` portion
corresponds to the field *type* being rendered (e.g. ``textarea``, ``checkbox``,
``date``, etc) whereas the ``part`` portion corresponds to *what* is being
rendered (e.g. ``label``, ``widget``, ``errors``, etc). By default, there
are 4 possible *parts* of a form that can be rendered:

+-------------+--------------------------+---------------------------------------------------------+
| ``label``   | (e.g. ``field_label``)   | renders the field's label                               |
+-------------+--------------------------+---------------------------------------------------------+
| ``widget``  | (e.g. ``field_widget``)  | renders the field's HTML representation                 |
+-------------+--------------------------+---------------------------------------------------------+
| ``errors``  | (e.g. ``field_errors``)  | renders the field's errors                              |
+-------------+--------------------------+---------------------------------------------------------+
| ``row``     | (e.g. ``field_row``)     | renders the field's entire row (label, widget & errors) |
+-------------+--------------------------+---------------------------------------------------------+

.. note::

    There are actually 3 other *parts*  - ``rows``, ``rest``, and ``enctype`` -
    but you should rarely if ever need to worry about overriding them.

By knowing the field type (e.g. ``textarea``) and which part you want to
customize (e.g. ``widget``), you can construct the fragment name that needs
to be overridden (e.g. ``textarea_widget``).

.. index::
   single: Forms; Template Fragment Inheritance

Template Fragment Inheritance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In some cases, the fragment you want to customize will appear to be missing.
For example, there is no ``textarea_errors`` fragment in the default themes
provided with Symfony. So how are the errors for a textarea field rendered?

The answer is: via the ``field_errors`` fragment. When Symfony renders the errors
for a textarea type, it looks first for a ``textarea_errors`` fragment before
falling back to the ``field_errors`` fragment. Each field type has a *parent*
type (the parent type of ``textarea`` is ``field``), and Symfony uses the
fragment for the parent type if the base fragment doesn't exist.

So, to override the errors for *only* ``textarea`` fields, copy the
``field_errors`` fragment, rename it to ``textarea_errors`` and customize it. To
override the default error rendering for *all* fields, copy and customize the
``field_errors`` fragment directly.

.. tip::

    The "parent" type of each field type is available in the
    :doc:`form type reference</reference/forms/types>` for each field type.

.. index::
   single: Forms; Global Theming

Global Form Theming
~~~~~~~~~~~~~~~~~~~

In the above example, you used the ``form_theme`` helper (in Twig) to "import"
the custom form fragments into *just* that form. You can also tell Symfony
to import form customizations across your entire project.

Twig
....

To automatically include the customized blocks from the ``fields.html.twig``
template created earlier in *all* templates, modify your application configuration
file:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        twig:
            form:
                resources:
                    - 'AcmeTaskBundle:Form:fields.html.twig'
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <twig:config ...>
                <twig:form>
                    <resource>AcmeTaskBundle:Form:fields.html.twig</resource>
                </twig:form>
                <!-- ... -->
        </twig:config>

    .. code-block:: php

        // app/config/config.php

        $container->loadFromExtension('twig', array(
            'form' => array('resources' => array(
                'AcmeTaskBundle:Form:fields.html.twig',
             ))
            // ...
        ));

Any blocks inside the ``fields.html.twig`` template are now used globally
to define form output.

.. sidebar::  Customizing Form Output all in a Single File with Twig

    In Twig, you can also customize a form block right inside the template
    where that customization is needed:

    .. code-block:: html+jinja

        {% extends '::base.html.twig' %}

        {# import "_self" as the form theme #}
        {% form_theme form _self %}

        {# make the form fragment customization #}
        {% block field_row %}
            {# custom field row output #}
        {% endblock field_row %}

        {% block content %}
            {# ... #}

            {{ form_row(form.task) }}
        {% endblock %}

    The ``{% form_theme form _self %}`` tag allows form blocks to be customized
    directly inside the template that will use those customizations. Use
    this method to quickly make form output customizations that will only
    ever be needed in a single template.

PHP
...

To automatically include the customized templates from the ``Acme/TaskBundle/Resources/views/Form``
directory created earlier in *all* templates, modify your application configuration
file:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        framework:
            templating:
                form:
                    resources:
                        - 'AcmeTaskBundle:Form'
        # ...


    .. code-block:: xml

        <!-- app/config/config.xml -->

        <framework:config ...>
            <framework:templating>
                <framework:form>
                    <resource>AcmeTaskBundle:Form</resource>
                </framework:form>
            </framework:templating>
            <!-- ... -->
        </framework:config>

    .. code-block:: php

        // app/config/config.php

        $container->loadFromExtension('framework', array(
            'templating' => array('form' =>
                array('resources' => array(
                    'AcmeTaskBundle:Form',
             )))
            // ...
        ));

Any fragments inside the ``Acme/TaskBundle/Resources/views/Form`` directory
are now used globally to define form output.

.. index::
   single: Forms; CSRF Protection

.. _forms-csrf:

CSRF Protection
---------------

CSRF - or `Cross-site request forgery`_ - is a method by which a malicious
user attempts to make your legitimate users unknowingly submit data that
they don't intend to submit. Fortunately, CSRF attacks can be prevented by
using a CSRF token inside your forms.

The good news is that, by default, Symfony embeds and validates CSRF tokens
automatically for you. This means that you can take advantage of the CSRF
protection without doing anything. In fact, every form in this chapter has
taken advantage of the CSRF protection!

CSRF protection works by adding a hidden field to your form - called ``_token``
by default - that contains a value that only you and your user knows. This
ensures that the user - not some other entity - is submitting the given data.
Symfony automatically validates the presence and accuracy of this token.

The ``_token`` field is a hidden field and will be automatically rendered
if you include the ``form_rest()`` function in your template, which ensures
that all un-rendered fields are output.

The CSRF token can be customized on a form-by-form basis. For example::

    class TaskType extends AbstractType
    {
        // ...

        public function getDefaultOptions(array $options)
        {
            return array(
                'data_class'      => 'Acme\TaskBundle\Entity\Task',
                'csrf_protection' => true,
                'csrf_field_name' => '_token',
                // a unique key to help generate the secret token
                'intention'       => 'task_item',
            );
        }

        // ...
    }

To disable CSRF protection, set the ``csrf_protection`` option to false.
Customizations can also be made globally in your project. For more information,
see the :ref:`form configuration reference </reference-frameworkbundle-forms>`
section.

.. note::

    The ``intention`` option is optional but greatly enhances the security of
    the generated token by making it different for each form.

.. index:
   single: Forms; With no class

Using a Form without a Class
----------------------------

In most cases, a form is tied to an object, and the fields of the form get
and store their data on the properties of that object. This is exactly what
you've seen so far in this chapter with the `Task` class.

But sometimes, you may just want to use a form without a class, and get back
an array of the submitted data. This is actually really easy::

    // make sure you've imported the Request namespace above the class
    use Symfony\Component\HttpFoundation\Request
    // ...

    public function contactAction(Request $request)
    {
        $defaultData = array('message' => 'Type your message here');
        $form = $this->createFormBuilder($defaultData)
            ->add('name', 'text')
            ->add('email', 'email')
            ->add('message', 'textarea')
            ->getForm();

            if ($request->getMethod() == 'POST') {
                $form->bindRequest($request);

                // data is an array with "name", "email", and "message" keys
                $data = $form->getData();
            }

        // ... render the form
    }

By default, a form actually assumes that you want to work with arrays of
data, instead of an object. There are exactly two ways that you can change
this behavior and tie the form to an object instead:

1. Pass an object when creating the form (as the first argument to ``createFormBuilder``
   or the second argument to ``createForm``);

2. Declare the ``data_class`` option on your form.

If you *don't* do either of these, then the form will return the data as
an array. In this example, since ``$defaultData`` is not an object (and
no ``data_class`` option is set), ``$form->getData()`` ultimately returns
an array.

Adding Validation
~~~~~~~~~~~~~~~~~

The only missing piece is validation. Usually, when you call ``$form->isValid()``,
the object is validated by reading the constraints that you applied to that
class. But without a class, how can you add constraints to the data of your
form?

The answer is to setup the constraints yourself, and pass them into your
form. The overall approach is covered a bit more in the :ref:`validation chapter<book-validation-raw-values>`,
but here's a short example::

    // import the namespaces above your controller class
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\Collection;

    $collectionConstraint = new Collection(array(
        'name' => new MinLength(5)
        'email' => new Email(array('message' => 'Invalid email address')),
    ));

    // create a form, no default values, pass in the constraint option
    $form = $this->createFormBuilder(null, , array(
        'validation_constraint' => $collectionConstraint,
    ))->add('email', 'email')
        // ...
    ;

Now, when you call `$form->isValid()`, the constraints setup here are run
against your form's data. If you're using a form class, override the ``getDefaultOptions``
method to specify the option::

    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\Collection;

    class ContactType extends AbstractType
    {
        // ...

        public function getDefaultOptions(array $options)
        {
            $collectionConstraint = new Collection(array(
                'name' => new MinLength(5)
                'email' => new Email(array('message' => 'Invalid email address')),
            ));

            $options['validation_constraint'] = $collectionConstraint;
        }
    }

Now, you have the flexibility to create forms - with validation - that return
an array of data, instead of an object. In most cases, it's better - and
certainly more robust - to bind your form to an object. But for simple forms,
this is a great approach.

Final Thoughts
--------------

You now know all of the building blocks necessary to build complex and
functional forms for your application. When building forms, keep in mind that
the first goal of a form is to translate data from an object (``Task``) to an
HTML form so that the user can modify that data. The second goal of a form is to
take the data submitted by the user and to re-apply it to the object.

There's still much more to learn about the powerful world of forms, such as
how to handle :doc:`file uploads with Doctrine
</cookbook/doctrine/file_uploads>` or how to create a form where a dynamic
number of sub-forms can be added (e.g. a todo list where you can keep adding
more fields via Javascript before submitting). See the cookbook for these
topics. Also, be sure to lean on the
:doc:`field type reference documentation</reference/forms/types>`, which
includes examples of how to use each field type and its options.

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/doctrine/file_uploads`
* :doc:`File Field Reference </reference/forms/types/file>`
* :doc:`Creating Custom Field Types </cookbook/form/create_custom_field_type>`
* :doc:`/cookbook/form/form_customization`

.. _`Symfony2 Form Component`: https://github.com/symfony/Form
.. _`DateTime`: http://php.net/manual/en/class.datetime.php
.. _`Twig Bridge`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bridge/Twig
.. _`form_div_layout.html.twig`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig
.. _`Cross-site request forgery`: http://en.wikipedia.org/wiki/Cross-site_request_forgery
.. _`view on GitHub`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bundle/FrameworkBundle/Resources/views/Form

.. toctree::
    :hidden:

    Translation source: 2011-10-02 8892b24
    Corrected from:
