.. index::
   single: Forms

Работа с формами
==================

Symfony2 выходит со встроенным компонентом форм. Он занимается отображением, рендерингом и передачей HTML форм.

Хотя можно обрабатывать формы одним только классом Symfony2 :class:`Symfony\\Component\\HttpFoundation\\Request`, компонент форм занимается также рядом общих, связанных с формами, задач:
1. Отображение HTML формы с автоматически созданными полями формы
2. Преобразование представленных данных в типы данных PHP
3. Чтение из POPOs и запись в них же (POPOs - простые старые PHP объекты)
4. Подтверждение представленной в формах информации при помощи ``Validator`` Symfony2.
5. Защита форм представлений от атак CSRF

Обзор
--------
Компонент имеет дело со следующими понятиями: 
*Field* 
Класс, преображающий представленную информацию в нормализованные значения.

*Form*
Набор полей, знающих, как проверить самих себя.

*Template*
Файл, который рендерит форму или поле в HTML.

*Domain objects*
Объект, используемый формой для заполнения значений по умолчанию, где записываются представленные данные. 

Объект формы в своей работе опирается лишь на компоненты HttpFoundation и Validator. Если вы планируете использовать особенности интернационализации, также потребуются международные PHP расширения.

Объекты форм
------------
Объект формы инкапсулирует набор полей, которые преобразовывают представленные данные в формат, используемый в приложении. Классы форм создаются как подклассы :class:`Symfony\\Component\\Form\\Form`. Вам следует применить метод ``configure()`` для инициализации формы со множественными полями.

.. code-block:: php

    // src/Sensio/HelloBundle/Contact/ContactForm.php
    namespace Sensio\HelloBundle\Contact;

    use Symfony\Component\Form\Form;
    use Symfony\Component\Form\TextField;
    use Symfony\Component\Form\TextareaField;
    use Symfony\Component\Form\CheckboxField;
    
    class ContactForm extends Form
    {
        protected function configure()
        {
            $this->add(new TextField('subject', array(
                'max_length' => 100,
            )));
            $this->add(new TextareaField('message'));
            $this->add(new TextField('sender'));
            $this->add(new CheckboxField('ccmyself', array(
                'required' => false,
            )));
        }
    }

Форма состоит из объектов ``Field``. В этом случае наша форма обладает полями ``subject``, ``message``, ``sender`` и ``ccmyself``. ``TextField``,
``TextareaField`` и ``CheckboxField`` - только три из возможных полей формы; полный список можно найти в :doc:`Form fields
<fields>`.

Использование формы в контроллере
---------------------------------
Стандартный шаблон для использования формы в контроллере выглядит примерно так: 

.. code-block:: php

    // src/Sensio/HelloBundle/Controller/HelloController.php
    public function contactAction()
    {
        $contactRequest = new ContactRequest($this->get('mailer'));
        $form = ContactForm::create($this->get('form.context'), 'contact');
        
        // If a POST request, write the submitted data into $contactRequest
        // and validate the object
        $form->bind($this->get('request'), $contactRequest);
        
        // If the form has been submitted and is valid...
        if ($form->isValid()) {
            $contactRequest->send();
        }

        // Display the form with the values in $contactRequest
        return $this->render('HelloBundle:Hello:contact.html.twig', array(
            'form' => $form
        ));
    }

Здесь существует два пути кода:
1. Если форма не была отправлена или не валидна, она просто пропускается в шаблон.
2. Если форма была отправлена и является валидной, выполняется запрос отправки.

Мы создали форму со статическим методом ``create()``. Этот метод ожидает, что в содержании формы будут присутствовать все сервисы по умолчанию (например, ``Validator``) и настройки, которые необходимы форме для работы.

Примечание.
Если вы не используете Symfony2 или ее контейнер сервиса, не волнуйтесь. Вы легко можете создать ``FormContext`` и ``Request`` вручную:

 .. code-block:: php
    
        use Symfony\Component\Form\FormContext;
        use Symfony\Component\HttpFoundation\Request;
        
        $context = FormContext::buildDefault();
        $request = Request::createFromGlobals();

Формы и доменные объекты
------------------------
В предыдущем примере ``ContactRequest`` был привязан к форме. Значения свойств этого объекта используются для заполнения полей формы. После привязывания представленные значения снова записываются в объект. Класс ``ContactRequest`` может выглядеть примерно так: 

.. code-block:: php

    // src/Sensio/HelloBundle/Contact/ContactRequest.php
    namespace Sensio\HelloBundle\Contact;

    class ContactRequest
    {
        protected $subject = 'Subject...';
        
        protected $message;
        
        protected $sender;
        
        protected $ccmyself = false;
        
        protected $mailer;
        
        public function __construct(\Swift_Mailer $mailer)
        {
            $this->mailer = $mailer;
        }
        
        public function setSubject($subject)
        {
            $this->subject = $subject;
        }
        
        public function getSubject()
        {
            return $this->subject;
        }
        
        // Setters and getters for the other properties
        // ...
        
        public function send()
        {
            // Send the contact mail
            $message = \Swift_Message::newInstance()
                ->setSubject($this->subject)
                ->setFrom($this->sender)
                ->setTo('me@example.com')
                ->setBody($this->message);
                
            $this->mailer->send($message);
        }
    }

Примечание.

Смотрите :doc:`Emails </guides/emails>` для получения информации об отправке писем.

Для каждого поля в вашей форме класс доменного объекта должен иметь:
1. Публичное свойство с именем поля или
2. Публичные setter и getter с префиксом "set"/"get" с последующим именем поля, наичнающимся с заглавной буквы.
   
Проверка отправленных данных
-----------------------------------

Форма использует компонент ``Validator`` для проверки отправленных значений формы. Все ограничения доменного объекта в форме и в ее полях будут проверены при вызове ``bind()``. Затем мы добавим несколько ограничений в ``ContactRequest``, чтобы убедиться, что никто не сможет отправить формы с некорректными данными. 

.. code-block:: php

    // src/Sensio/HelloBundle/Contact/ContactRequest.php
    namespace Sensio\HelloBundle\Contact;

    class ContactRequest
    {
        /**
         * @validation:MaxLength(100)
         * @validation:NotBlank
         */
        protected $subject = 'Subject...';
        
        /**
         * @validation:NotBlank
         */
        protected $message;
        
        /**
         * @validation:Email
         * @validation:NotBlank
         */
        protected $sender;
        
        /**
         * @validation:AssertType("boolean")
         */
        protected $ccmyself = false;
        
        // Other code...
    }

Если какое-либо ограничение срабатывает, рядом с соответствующим полем формы выводится сообщение об ошибке. Вы можете узнать больше об ограничениях в :doc:`Validation 
Constraints </guides/validator/constraints>`.

Автоматическое создание полей формы
-----------------------------------

Если вы используете ``Validator`` Doctrine2 или Symfony2, Symfony уже знает довольно много о ваших доменных классах. Она знает, какой тип данных используется для сохранения свойств в базе данных, какими проверками ограничений обладает свойство и т.д. Компонет формы может использовать эту информацию для "угадывания", какой из типов поля должен быть создан с теми же свойствами.

Для использования этой особенности форме необходимо знать класс связанного доменного объекта. Вы можете выставить этот класс с помощью метода формы ``configure()``, используя ``setDataClass()`` и пропуская полное имя класса как строку. Вызов ``add()`` только с именем свойства автоматически создаст лучшее соответствующее поле.

.. code-block:: php

    // src/Sensio/HelloBundle/Contact/ContactForm.php
    class ContactForm extends Form
    {
        protected function configure()
        {
            $this->setDataClass('Sensio\\HelloBundle\\Contact\\ContactRequest');
            $this->add('subject');  // TextField with max_length=100 because
                                    // of the @MaxLength constraint
            $this->add('message');  // TextField
            $this->add('sender');   // EmailField because of the @Email constraint
            $this->add('ccmyself'); // CheckboxField because of @AssertType("boolean")
        }
    }

Эти предугадывания полей не всегда оказываются верными. Для свойства ``message``Symfony создаст ``TextField``, поскольку не сможет узнать из ограничений, что вы хотели создать ``TextareaField``. Так что вам придется создать это поле вручную. Вы также можете настроить эти опции генерируемых полей путем передачи их в качестве второго параметра. Мы добавим опцию ``max_length`` в поле ``sender`` для ограничения его длины. 

.. code-block:: php

    // src/Sensio/HelloBundle/Contact/ContactForm.php
    class ContactForm extends Form
    {
        protected function configure()
        {
            $this->setDataClass('Sensio\\HelloBundle\\Contact\\ContactRequest');
            $this->add('subject'); 
            $this->add(new TextareaField('message'));
            $this->add('sender', array('max_length' => 50));
            $this->add('ccmyself');
        }
    }

Автоматическое создание полей формы поможет вам увеличить скорость разработки и сократить дублирование кода. Вы можете сохранить информацию о свойствах класса лишь однажды и позволить Symfony2 делать остальную работу за вас.

Рендеринг форм как HTML
-----------------------

В контроллере мы пропускаем форму в шаблон в переменной ``form``. В шаблоне мы можем использовать помощник ``form_field`` для вывода сырого прототипа формы.

.. code-block:: html+jinja

    # src/Sensio/HelloBundle/Resources/views/Hello/contact.html.twig
    {% extends 'HelloBundle::layout.html.twig' %}

    {% block content %}
    <form action="#" method="post">
        {{ form_field(form) }}
        
        <input type="submit" value="Send!" />
    </form>
    {% endblock %}

Настройка вывода в формате HTML
-------------------------------

В большинстве приложений вам нужно будет настроить HTML формы. Это можно сделать используя остальные встроенные помощники рендеринга форм.

.. code-block:: html+jinja

    # src/Sensio/HelloBundle/Resources/views/Hello/contact.html.twig
    {% extends 'HelloBundle::layout.html.twig' %}

    {% block content %}
    <form action="#" method="post" {{ form_enctype(form) }}>
        {{ form_errors(form) }}
        
        {% for field in form %}
            {% if not field.ishidden %}
            <div>
                {{ form_errors(field) }}
                {{ form_label(field) }}
                {{ form_field(field) }}
            </div>
            {% endif %}
        {% endfor %}

        {{ form_hidden(form) }}
        <input type="submit" />
    </form>
    {% endblock %}

В составе Symfony2 есть следующие помощники:

*``form_enctype``*
 Выводит атрибут ``enctype`` тэга формы. Требуется для загрузки файлов.

*``form_errors``*
  Выводит тэг ``<ul>`` с ошибками поля или формы.

*``form_label``*
  Выводит тэг поля ``<label>``.

*``form_field``*
  Выводит HTML поля или формы.

*``form_hidden``*
  Выводит все скрытые поля формы.

Рендеринг форм в деталях описан в :doc:`Forms in Templates <view>`. 

Поздравляем! Вы только что создали вашу первую полностью фунциональную форму с помощью Symfony2.

