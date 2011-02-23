Переводы
========
Компонент :namespace:`Symfony\\Component\\Translation`занимается интернационализацией всех сообщений вашего приложения.
*message* (сообщение) может быть любой строкой, которая нуждается в интернационализации. Сообщения делятся на локальные и доменные.
*domain* (домен) позволяет лучше организовать сообщения в данной локализации (может быть любой строкой; по умолчанию все сообщения хранятся в домене ``messages``).
*locale* (локализация) может быть любой строкой, но рекомендуется использовать языковой код ISO639-1 с подчеркиванием (``_``), а затем код страны ISO3166 (например, ``fr_FR`` для French/France).

Настройки
---------
Прежде, чем начать использовать особенности переводчика, включите его в вашей конфигурации:
.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        app.config:
            translator: { fallback: en }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <app:config>
            <app:translator fallback="en" />
        </app:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('app', 'config', array(
            'translator' => array('fallback' => 'en'),
        ));
Свойство ``fallback`` определяет резервную локализацию на случай, если не существует перевода в пользовательской локализации.

.. tip::
    Если нет перевода для некоторой локализации, переводчик пытается найти перевод для языка (``fr``, если локализация - ``fr_FR``, например); если и это не удастся, он будет искать перевод для резервной локализации.

Используемая в переводах локализация хранится в сессии пользователя.

Переводы
--------

Переводы - часть сервиса ``translator`` - (:class:`Symfony\\Component\\Translation\\Translator`). Используйте метод :method:`Symfony\\Component\\Translation\\Translator::trans` для перевода сообщения::

    $t = $this->get('translator')->trans('Symfony2 is great');

Если у вас в строках есть заменяемые значения, пропустите их значения в качестве второго аргумента:

    $t = $this->get('translator')->trans('Symfony2 is {{ what }}!', array('{{ what }}' => 'great'));

.. note::

    Заменяемые значения могут принимать любые формы, но применение нотации ``{{ var }}`` позволит использовать сообщение в шаблонах Twig.

По умолчанию переводчик будет искать сообщения в домене по умолчанию ``messages``. Переопределить это можно с помощью третьего аргумента:

    $t = $this->get('translator')->trans('Symfony2 is great', array(), 'applications');

Каталоги
--------

Переводы хранятся в файловой системе и могут быть найдены Symfony2 благодаря некоторым положениям.

Храните переводы для сообщений, найденных в узлах, в папке ``Resources/translations/`` и переопределяйте их в папке ``app/translations/``.

Каждое сообщение должно именоваться в соответствии со следующим шаблоном:
``domain.locale.loader`` (доменное имя, затем точка(``.``), затем локальное имя, снова точка (``.``), затем имя загрузчика).

Загрузчик может иметь имя любого зарегистрированного загрузчика. По умолчанию Symfony2 предоставляет следующие загрузчики:

* ``php``:   PHP file;
* ``xliff``: XLIFF file;
* ``yaml``:  YAML file.

Каждый файл состоит из пар строк id/перевод для заданного домена и локализации. Id может быть сообщением в основной локализации вашего приложения с уникальным идентификатором:

.. configuration-block::

    .. code-block:: xml

        <?xml version="1.0"?>
        <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
            <file source-language="en" datatype="plaintext" original="file.ext">
                <body>
                    <trans-unit id="1">
                        <source>Symfony2 is great</source>
                        <target>J'aime Symfony2</target>
                    </trans-unit>
                    <trans-unit id="2">
                        <source>symfony2.great</source>
                        <target>J'aime Symfony2</target>
                    </trans-unit>
                </body>
            </file>
        </xliff>

    .. code-block:: php

        return array(
            'Symfony2 is great' => 'J\'aime Symfony2',
            'symfony2.great'    => 'J\'aime Symfony2',
        );

    .. code-block:: yaml

        Symfony2 is great: J'aime Symfony2
        symfony2.great:    J'aime Symfony2

.. sidebar:: Лучше организовывайте ваши переводы

Кроме того, файловые форматы ``php`` и ``yaml`` поддерживают вложенные id для предотвращения повторов самих себя, если для ваших id используются ключевые слова вместо обычного текста:

    .. configuration-block::

        .. code-block:: yaml

            symfony2:
                is:
                    great: Symfony2 is great
                    amazing: Symfony2 is amazing
                has:
                    bundles: Symfony2 has bundles
            user:
                login: Login

        .. code-block:: php

            return array(
                'symfony2' => array(
                    'is' => array(
                        'great' => 'Symfony2 is great',
                        'amazing' => 'Symfony2 is amazing',
                    ),
                    'has' => array(
                        'bundles' => 'Symfony2 has bundles',
                    ),
                ),
                'user' => array(
                    'login' => 'Login',
                ),
            );

Множественные уровни соединены в одиночные пары id/перевод путем добавления точки (.) между каждым уровнем, поэтому рассмотренные выше примеры эквивалентны следующим:

    .. configuration-block::

        .. code-block:: yaml

            symfony2.is.great: Symfony2 is great
            symfony2.is.amazing: Symfony2 is amazing
            symfony2.has.bundles: Symfony2 has bundles
            user.login: Login

        .. code-block:: php

            return array(
                'symfony2.is.great' => 'Symfony2 is great',
                'symfony2.is.amazing' => 'Symfony2 is amazing',
                'symfony2.has.bundles' => 'Symfony2 has bundles',
                'user.login' => 'Login',
            );

.. note::

    Вы также можете хранить переводы в базе данных или любом другом хранилище путем создания специального класса, имплементирующего интерфейс :class:`Symfony\\Component\\Translation\\Loader\\LoaderInterface`. 
    Читайте дальше, чтобы узнать, как регистрировать специальные загрузчики.

Множественность
------------

Сообщение множественности - сложная тема, поскольку правила могут быть весьма общими. Например, вот математическое представление русских правил множественности::

    (($number % 10 == 1) && ($number % 100 != 11)) ? 0 : ((($number % 10 >= 2) && ($number % 10 <= 4) && (($number % 100 < 10) || ($number % 100 >= 20))) ? 1 : 2);

Как видите, можно использовать три различные множественные формы, основанные на этом алгоритме. Для каждой формы множественное число различно, различен и перевод. В таком случае можно хранить все формы множественного числа в строках, разделяя их вертикальной чертой (``|``)::

 'There is one apple|There are {{ count }} apples'

Основываясь на заданном числе, переводчик выбирает правильную множественную форму. Если ``count`` равняется 1, переводчик будет использовать первую строку (``There is one apple``) как перевод, если нет, он будет применять ``There are {{ count }} apples``.

Вот французский перевод::

'Il y a {{ count }} pomme|Il y a {{ count }} pommes'

Даже если строка выглядит похоже (состоит из двух подстрок, разделенных вертикаьной чертой), французские правила будут работать по-другому: первая форма (не множественное число) используется, если ``count`` равняется ``0`` или ``1``. Таким образом, переводчик будет автоматически использовать первую строку (`Il y a {{ count }} pomme``), если ``count`` - ``0`` или ``1``. 

Правила весьма просты для английского и французского, но для русского вам лучше ввести подсказку, чтобы знать, какое правило совпадает с какой строкой. В помощь переводчику можно "заключать в тэг" каждую строку, примерно так::

 'one: There is one apple|some: There are {{ count }} apples'

    'none_or_one: Il y a {{ count }} pomme|some: Il y a {{ count }} pommes'

Тэги в действительности - всего лишь подсказки для переводчика, чтобы помочь ему лучше понять контекст (заметьте, что тэги не обязательно должны быть одинаковыми в оригинальном сообщении и в переведённом).

..tip::
Поскольку тэги необязательны, переводчик не использует их (переводчик лишь получит строку, основанную на позиции в строке ).

Иногда требуется различный перевод для различных случаев (для ``0``, или когда число слишком велико, или когда число отрицательное, ...). Для таких случаев можно использовать математические интервалы::

  '{0} There is no apples|{1} There is one apple|]1,19] There are {{ count }} apples|[20,Inf] There are many apples'

Вы также можете смешивать чистые математические правила и стандартные. Позиция для стандартных правил определяется после удаления чистых правил::

   '{0} There is no apples|[20,Inf] There are many apples|There is one apple|a_few: There are {{ count }} apples'

Класс:class:`Symfony\\Component\\Translation\\Interval` может представлять конечное множество чисел::

    {1,2,3,4}

Или числа между двумя другими числами::

    [1, +Inf[
    ]-1,2[

Левым разделителем может быть ``[`` (inclusive) или ``]`` (exclusive). Правым разделителем может быть ``[`` (exclusive) или ``]`` (inclusive). Кроме чисел можно использовать ``-Inf`` и ``+Inf`` для бесконечности.

.. note::

    Symfony2 использует `ISO 31-11`_ для обозначения интервалов. 

Метод переводчика :method:`Symfony\\Component\\Translation\\Translator::transChoice` знает, как обращаться со множественными числами::

    $t = $this->get('translator')->transChoice(
        '{0} There is no apples|{1} There is one apple|]1,Inf[ There are {{ count }} apples',
        10,
        array('{{ count }}' => 10)
    );

Заметьте, что второй аргумент - число, применяемое для определения того, какую множественную строку использовать.

Переводы в шаблонах
-------------------

Преимущественно перевод происходит в шаблонах. Symfony2 обеспечивает встроенную поддержку как PHP шаблонов, так и Twig.

PHP шаблоны
~~~~~~~~~~~

Услуга переводчика доступна в PHP шаблонах с применением помощника ``translator``:

.. code-block:: html+php

    <?php echo $view['translator']->trans('Symfony2 is great') ?>

    <?php echo $view['translator']->transChoice(
        '{0} There is no apples|{1} There is one apple|]1,Inf[ There are {{ count }} apples',
        10,
        array('{{ count }}' => 10)
    ) ?>

Шаблоны Twig
~~~~~~~~~~~~~~

В Symfony2 есть специальные тэги Twig (``trans`` и ``transChoice``) для помощи в переводе сообщений:

.. code-block:: jinja

    {% trans "Symfony2 is great" %}

    {% trans %}
        Foo {{ name }}
    {% endtrans %}

    {% transchoice count %}
        {0} There is no apples|{1} There is one apple|]1,Inf] There are {{ count }} apples
    {% endtranschoice %}

Тэг ``transChoice`` автоматически получает переменные из текущего контекста и пропускает их через переводчик. Этот механизм работает только, если вы используете заменяемые значения и шаблон ``{{ var }}``.

Еще можно настроить сообщение домена:

.. code-block:: jinja

    {% trans "Foo {{ name }}" from "app" %}

    {% trans from "app" %}
        Foo {{ name }}
    {% endtrans %}

    {% transchoice count from "app" %}
        {0} There is no apples|{1} There is one apple|]1,Inf] There are {{ count }} apples
    {% endtranschoice %}

.. _translation_loader_tag:

Включение специальных загрузчиков
---------------------------------

Для включения специального загрузчика добавьте его в качестве постоянного сервиса в одну из ваших конфигураций с тэгом ``translation.loader`` и определите атрибут ``alias`` (для загрузчиков, работающих с файловой системой, alias - расширение файла, которое нужно использовать для ссылки на загрузчик):

.. configuration-block::

    .. code-block:: yaml

        services:
            translation.loader.your_helper_name:
                class: Fully\Qualified\Loader\Class\Name
                tags:
                    - { name: translation.loader, alias: alias_name }

    .. code-block:: xml

        <service id="translation.loader.your_helper_name" class="Fully\Qualified\Loader\Class\Name">
            <tag name="translation.loader" alias="alias_name" />
        </service>

    .. code-block:: php

        $container
            ->register('translation.loader.your_helper_name', 'Fully\Qualified\Loader\Class\Name')
            ->addTag('translation.loader', array('alias' => 'alias_name'))
        ;

.. _ISO 31-11: http://en.wikipedia.org/wiki/Interval_%28mathematics%29#The_ISO_notation
