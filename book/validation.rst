.. index::
   single: Валидация

Валидация
==========

Валидация - это вполне обычная задача для web-приложения. Данные, вводимые
в формы должны быть валидированы (проверены). В то же время, данные должны быть
валидированы до того, как они будут записаны в базу данных или же будут
переданы далее некоторому web-сервису.

Symfony2 содержит компонент `Validator`_ для того, чтобы упростить эту задачу.
Этот компонент основан на документе `JSR303 Bean Validation specification`_. Что?!
Java спецификация в PHP? Однако же вы всё услышали верно, но всё не так плохо как
вам могло показаться. Давайте посмотрим, как мы можем использовать это в PHP.

.. index:
   single: Валидация; Основы

Основы Валидации
----------------

Самый лучший способ понять валидацию - это увидеть её в действии. Для начала,
предположим, что вы создали обычный PHP-объект и вам нужно использовать его
где-то внутри приложения:

.. code-block:: php

    <?php
    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    class Author
    {
        public $name;
    }

Итак, это обычный класс, который обслуживает некоторый круг задач внутри
вашего приложения. Цель валидации заключается в том, чтобы сообщить вам -
являются ли данные объекта корректными (валидными). Для этого, вам нужно
настроить перечень правил (называемых :ref:`Ограничениями<validation-constraints>`),
которым объект должен удовлетворять, чтобы быть валидным. Эти правила могут
быть указаны во многих форматах (YAML, XML, аннотации, или PHP).

Например, для того, чтобы гарантировать, что свойство ``$name`` не пусто,
добавьте следующий код:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                name:
                    - NotBlank: ~

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\NotBlank()
             */
            public $name;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="name">
                    <constraint name="NotBlank" />
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        <?php
        // src/Acme/BlogBundle/Entity/Author.php

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;

        class Author
        {
            public $name;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('name', new NotBlank());
            }
        }

.. tip::

    Защищённые (protected) и закрытые (private) члены класса могут быть также
    валидированы как и любой ``get``-метод (см. `validator-constraint-targets`_).

.. index::
   single: Валидация; Использование валидатора

Использование сервиса ``validator``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Далее, чтобы проверить объект ``Author``, используйте метод ``validate``
сервиса ``validator`` (class :class:`Symfony\\Component\\Validator\\Validator`).
Обязанности у класса ``validator`` простые: прочитать ограничения (т.е. правила),
определённые для класса, и определить, удовлетворяют ли данные из объекта этим
ограничениям. Если валидация проходит с ошибкой, возвращает массив ошибок.
Давайте рассмотрим этот простой пример контроллера:

.. code-block:: php

    <?php
    use Symfony\Component\HttpFoundation\Response;
    use Acme\BlogBundle\Entity\Author;
    // ...

    public function indexAction()
    {
        $author = new Author();
        // ... выполняются какие-либо действия с объектом $author

        $validator = $this->get('validator');
        $errors = $validator->validate($author);

        if (count($errors) > 0) {
            return new Response(print_r($errors, true));
        } else {
            return new Response('The author is valid! Yes!');
        }
    }

Если свойство ``$name`` пустое, вы увидите следующую ошибку:

.. code-block:: text

    Acme\BlogBundle\Author.name:
        This value should not be blank

Если же вы укажете некоторое непустое значение для ``name``, появится сообщение
об успешной валидации.

.. tip::

    Большую часть времени вы не будете напрямую взаимодействовать с сервисом
    ``validator`` и вам не нужно будет беспокоиться об отображении ошибок.
    Большую часть времени вы будете использовать валидацию косвенно при
    обработке данных из отправленных приложению форм. Подробнее об этом написано
    в секции: :ref:`book-validation-forms`.

Вы также можете передать перечень ошибок в шаблон.

.. code-block:: php

    <?php
    if (count($errors) > 0) {
        return $this->render('AcmeBlogBundle:Author:validate.html.twig', array(
            'errors' => $errors,
        ));
    } else {
        // ...
    }

Внутри шаблона вы можете отобразить список ошибок так, как вам нужно:

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Author/validate.html.twig #}

        <h3>The author has the following errors</h3>
        <ul>
        {% for error in errors %}
            <li>{{ error.message }}</li>
        {% endfor %}
        </ul>

    .. code-block:: html+php

        <!-- src/Acme/BlogBundle/Resources/views/Author/validate.html.php -->

        <h3>The author has the following errors</h3>
        <ul>
        <?php foreach ($errors as $error): ?>
            <li><?php echo $error->getMessage() ?></li>
        <?php endforeach; ?>
        </ul>

.. note::

    Каждая ошибка валидации (называемая "constraint violation"), представлена
    объектом класса :class:`Symfony\\Component\\Validator\\ConstraintViolation`

.. index::
   single: Валидация; Валидация форм

.. _book-validation-forms:

Валидация и Формы
~~~~~~~~~~~~~~~~~~~~

Сервис ``validator`` может быть использован в любое время для проверки объекта.
В жизни же, не смотря такую возможность, вы будете работать с сервисом
``validator`` косвенно при обработке форм. Библиотека форм Symfony использует
сервис валидации для проверки объектов форм после того, как данные были отправлены
пользователем и привязаны к форме. Объекты ошибок валидации ("constraint violations")
будут конвертированы в объекты ``FieldError``, которые могут быть легко отображены
вместе с формами. Типичный процесс отправки формы со стороны контроллера выглядит так:

.. code-block:: php

    <?php
    use Acme\BlogBundle\Entity\Author;
    use Acme\BlogBundle\Form\AuthorType;
    use Symfony\Component\HttpFoundation\Request;
    // ...

    public function updateAction(Request $request)
    {
        $author = new Acme\BlogBundle\Entity\Author();
        $form = $this->createForm(new AuthorType(), $author);

        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // валидация прошла успешно, можно выполнять дальнейшие действия с объектом $author

                $this->redirect($this->generateUrl('...'));
            }
        }

        return $this->render('BlogBundle:Author:form.html.twig', array(
            'form' => $form->createView(),
        ));
    }

.. note::

    Этот пример использует класс формы ``AuthorType``, который в этой главе не описан.

Более подробную информацию о формах вы можете получить в главе :doc:`Формы</book/forms>`.

.. index::
   pair: Валидация; Конфигурирование

.. _book-validation-configuration:

Конфигурирование
----------------

Валидатор Symfony2 доступен по умолчанию, но вы должны тщательно настроить его
при помощи аннотаций (если вы используете метод аннотаций для настройки ограничений):

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            validation: { enable_annotations: true }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <framework:validation enable_annotations="true" />
        </framework:config>

    .. code-block:: php

        <?php
        // app/config/config.php
        $container->loadFromExtension('framework', array('validation' => array(
            'enable_annotations' => true,
        )));

.. index::
   single: Валидация; Ограничения

.. _validation-constraints:

Ограничения
-----------

Валидатор создан для того, чтобы проверять объекты на соответствие *ограничениям*
(т.е. правилам). Для того чтобы валидировать объект, укажите для его класса одно
или более ограничений и передайте его сервису валидации (``validator``).

По сути, ограничение - это PHP объект, который выполняет проверочное выражение.
В жизни ограничение может выглядеть так: "пирог не должен подгореть". В Symfony2
ограничения выглядят похожим образом: это утверждения, что некоторое выражение
истинно. Учитывая значение, ограничение скажет вам, соответствует ли это значение
правилу ограничения.

Поддерживаемые ограничения
~~~~~~~~~~~~~~~~~~~~~~~~~~

Symfony2 содержит большое количество ограничений, необходимых в повседневной работе:

.. include:: /reference/constraints/map.rst.inc

Вы также можете создавать свои ограничения. Этот вопрос освещается в топике
":doc:`/cookbook/validation/custom_constraint`" в книге рецептов.

.. index::
   single: Валидация; Конфигурация ограничений

.. _book-validation-constraint-configuration:

Конфигурация ограничений
~~~~~~~~~~~~~~~~~~~~~~~~

Некоторые ограничения, как например :doc:`NotBlank</reference/constraints/NotBlank>`,
просты, в то время как другие, как например :doc:`Choice</reference/constraints/Choice>`,
имеют много различных опций. Предположим, что класс ``Author`` имеет поле ``gender``,
которое может иметь два значения - "male" или "female":

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                gender:
                    - Choice: { choices: [male, female], message: Choose a valid gender. }

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\Choice(
             *     choices = { "male", "female" },
             *     message = "Choose a valid gender."
             * )
             */
            public $gender;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="gender">
                    <constraint name="Choice">
                        <option name="choices">
                            <value>male</value>
                            <value>female</value>
                        </option>
                        <option name="message">Choose a valid gender.</option>
                    </constraint>
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        <?php
        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;

        class Author
        {
            public $gender;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('gender', new Choice(array(
                    'choices' => array('male', 'female'),
                    'message' => 'Choose a valid gender.',
                )));
            }
        }

.. _validation-default-option:

Опции ограничения всегда представлены в виде массива. Однако, некоторые ограничения
также позволяют вам указать одну опцию - основную, а не массив опций. В случае с
ограничением ``Choice``, можно указать только варианты выбора (``choices``).

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                gender:
                    - Choice: [male, female]

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\Choice({"male", "female"})
             */
            protected $gender;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="gender">
                    <constraint name="Choice">
                        <value>male</value>
                        <value>female</value>
                    </constraint>
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        <?php
        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\Choice;

        class Author
        {
            protected $gender;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('gender', new Choice(array('male', 'female')));
            }
        }

Такая возможность позволяет сделать настройку базовых опций ограничения короче и быстрее.

Если вы не уверены, как нужно указывать опцию, или же сверьтесь с документацией API
для ограничения или же поступайте просто - всегда передавайте массив опций
(как показано выше в первом примере).

.. index::
   single: Валидация; Цели для ограничений

.. _validator-constraint-targets:

Цели для ограничений
--------------------

Ограничение могут быть применены к свойству класса (например, ``name``) или же
к публичному аксессору (или геттеру, например, ``getFullName``). Первый вариант
наиболее простой и чаще всего встречающийся, но второй вариант позволяет вам
создавать более сложные правила валидации.

.. index::
   single: Валидация; Ограничения для полей класса

.. _validation-property-target:

Поля класса
~~~~~~~~~~~

Валидация полей класса - это наиболее простая техника валидации. Symfony2
позволяет вам выполнять валидацию приватных, защищённых и публичных полей.
Следующий листинг показывает как настроить поле ``$firstName`` класса ``Author``,
чтобы оно имело как минимум три символа.

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                firstName:
                    - NotBlank: ~
                    - MinLength: 3

    .. code-block:: php-annotations

        // Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\NotBlank()
             * @Assert\MinLength(3)
             */
            private $firstName;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\Author">
            <property name="firstName">
                <constraint name="NotBlank" />
                <constraint name="MinLength">3</constraint>
            </property>
        </class>

    .. code-block:: php

        <?php
        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\MinLength;

        class Author
        {
            private $firstName;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('firstName', new NotBlank());
                $metadata->addPropertyConstraint('firstName', new MinLength(3));
            }
        }

.. index::
   single: Валидация; Ограничения для методов

Методы класса
~~~~~~~~~~~~~

Ограничения также могут быть применены к значениям, возвращаемым методами.
Symfony2 позволяет вам добавлять ограничения к любому *публичному* методу,
если его имя начинается с "get" или "is". В этом руководстве оба этих типа
методов называются "геттерами" (от ``getters``).

Выгода от этой техники в том, что она позволяет вам валидировать ваш проект
динамически. Например, предположим, что вы хотите быть уверенными, что поле пароля
не соответствует имени пользователя (по соображениям безопасности конечно, а не от
излишнего снобизма)). Вы можете достичь этого, создав метод ``isPasswordLegal`` и
указав, ограничение, что этот метод должен возвращать ``true``:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            getters:
                passwordLegal:
                    - "True": { message: "The password cannot match your first name" }

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\True(message = "The password cannot match your first name")
             */
            public function isPasswordLegal()
            {
                // return true or false
            }
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\Author">
            <getter property="passwordLegal">
                <constraint name="True">
                    <option name="message">The password cannot match your first name</option>
                </constraint>
            </getter>
        </class>

    .. code-block:: php

        <?php
        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\True;

        class Author
        {
            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addGetterConstraint('passwordLegal', new True(array(
                    'message' => 'The password cannot match your first name',
                )));
            }
        }

Теперь создайте метод ``isPasswordLegal()`` и реализуйте его логику::

    public function isPasswordLegal()
    {
        return ($this->firstName != $this->password);
    }

.. note::

    Особо внимательные читатели наверняка отметили, что в примере конфигурации
    опущен префикс геттера ("get" или "is"). Это позволяет вам легко переместить
    ограничение на поле класса с тем же именем в последствии (или же, наоборот, с
    поля на метод класса) без изменения логики валидации.

.. _validation-class-target:

Классы
~~~~~~

Некоторые ограничения применяются к целому классу во время валидации. Например
ограничение :doc:`Callback</reference/constraints/Callback>` - это универсальное
ограничение, которое применяется к классу целиком. Когда этот класс валидируется,
вызываются методы указанные ограничением, что позволяет выполнять более детальную
или же избирательную валидацию.

.. _book-validation-validation-groups:

Валидационные группы
--------------------

До сих пор вы могли добавлять ограничения к классу и узнавать, удовлетворяет ли
класс всем указанным для него ограничениям или же нет. В некоторых случаях, однако,
вам может потребоваться валидировать объект, используя лишь некоторые из
определённых для него ограничений. Для того чтобы получить такую возможность,
вы можете определить каждое ограничение в одну или более "валидационных групп"
и после этого выполнять валидацию лишь для одной из этих групп.

Например, положим у вас есть класс ``User``, который используется при регистрации
пользователя и при обновлении его профайла впоследствии:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\User:
            properties:
                email:
                    - Email: { groups: [registration] }
                password:
                    - NotBlank: { groups: [registration] }
                    - MinLength: { limit: 7, groups: [registration] }
                city:
                    - MinLength: 2

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/User.php
        namespace Acme\BlogBundle\Entity;

        use Symfony\Component\Security\Core\User\UserInterface
        use Symfony\Component\Validator\Constraints as Assert;

        class User implements UserInterface
        {
            /**
            * @Assert\Email(groups={"registration"})
            */
            private $email;

            /**
            * @Assert\NotBlank(groups={"registration"})
            * @Assert\MinLength(limit=7, groups={"registration"})
            */
            private $password;

            /**
            * @Assert\MinLength(2)
            */
            private $city;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\User">
            <property name="email">
                <constraint name="Email">
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
            </property>
            <property name="password">
                <constraint name="NotBlank">
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
                <constraint name="MinLength">
                    <option name="limit">7</option>
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
            </property>
            <property name="city">
                <constraint name="MinLength">7</constraint>
            </property>
        </class>

    .. code-block:: php

        <?php
        // src/Acme/BlogBundle/Entity/User.php
        namespace Acme\BlogBundle\Entity;

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\Email;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\MinLength;

        class User
        {
            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('email', new Email(array(
                    'groups' => array('registration')
                )));

                $metadata->addPropertyConstraint('password', new NotBlank(array(
                    'groups' => array('registration')
                )));
                $metadata->addPropertyConstraint('password', new MinLength(array(
                    'limit'  => 7,
                    'groups' => array('registration')
                )));

                $metadata->addPropertyConstraint('city', new MinLength(3));
            }
        }

При использовании такой конфигурации имеется две валидационные группы:

* ``Default`` - содержит ограничения, не включённые ни в одну из групп;

* ``registration`` - содержит ограничения для полей  ``email`` и ``password``.

Для того, чтобы явно указать валидатору группу, передайте одно или более
наименований групп вторым аргументом в метод ``validate()``::

    $errors = $validator->validate($author, array('registration'));

Конечно же, как правило, вы работаете с валидацией косвенно через библиотеку Форм.
Чтобы узнать как использовать группы в формах, смотрите раздел :ref:`book-forms-validation-groups`.

.. index::
   single: Валидация; Валидация обычных значений

.. _book-validation-raw-values:

Валидация простых значений и массивов
-------------------------------------

Ранее вы увидели как можно валидировать целые объекты. Но иногда, вам всего лишь
нужно валидировать простое значение - например, проверить, является ли строка
валидным email-адресом. Это также легко сделать. Внутри контроллера это будет
выглядеть так:

.. code-block:: php

    <?php
    // add this to the top of your class
    use Symfony\Component\Validator\Constraints\Email;

    public function addEmailAction($email)
    {
        $emailConstraint = new Email();
        // все опции ограничения можно задать таким образом
        $emailConstraint->message = 'Invalid email address';

        // используем валидатор для проверки значения
        $errorList = $this->get('validator')->validateValue($email, $emailConstraint);

        if (count($errorList) == 0) {
            // это ВАЛИДНЫЙ адрес, делаем дело дальше
        } else {
            // это НЕ валидный адрес
            $errorMessage = $errorList[0]->getMessage()

            // делаем что-то с ошибкой
        }

        // ...
    }

Вызывая метод ``validateValue`` в валидаторе, вы можете передать ему значение
в виде параметра и объект ограничения, которое хотите проверить. Полный список
доступных ограничений, а также полные имена классов для каждого ограничения,
можно найти в :doc:`Справочнике по ограничениям</reference/constraints>`.

Метод ``validateValule`` возвращает объект класса
:class:`Symfony\\Component\\Validator\\ConstraintViolationList`, который, по сути, является
массивом ошибок. Каждая ошибка в коллекции - это объект класса
:class:`Symfony\\Component\\Validator\\ConstraintViolation`, который содержит
сообщение об ошибке, которое можно получить, вызвав метод `getMessage`.

Заключение
----------

Валидатор Symfony2 - это мощный инструмент, который используется для получения
гарантий, что данные некоторого объекта "правильные". Сила валидации - в
ограничениях, которые являются правилами, которые применяются к полям классов
или же их "геттерам". И даже если вам приходится большей частью использовать
фреймворк для валидации косвенно - совместно с формами, помните, что он может
быть использован где угодно для валидации любого объекта.

Дополнительно в книге рецептов:
------------------------------

* :doc:`/cookbook/validation/custom_constraint`

.. _Validator: https://github.com/symfony/Validator
.. _JSR303 Bean Validation specification: http://jcp.org/en/jsr/detail?id=303

.. toctree::
    :hidden:

    Translation source: 2011-09-30 243с069
    Corrected from:
