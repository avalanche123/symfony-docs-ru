.. index::
   single: Doctrine

Базы данных и Doctrine ("Модель")
=================================

Давайте посмотрим правде в глаза, одни из самых распространённых и сложных
задач для любого приложения включают хранение и чтение информации из базы
данных. К счастью, Symfony поставляется совмещённым с `Doctrine`_ - библиотекой,
главная цель которой дать мощный инструмент, позволяющий делать это просто.
В этой главе вы постигнете основу философии Doctrine и увидите насколько простой
может быть работа с базой данных.

.. note::

    Doctrine полностью отделёна от Symfony и её использование необязательно. Эта
    глава о Doctrine ORM, цель которой позволить представить объекты в
    реляционных базах данных (таких как *MySQL*, *PostgreSQL* или *Microsoft SQL*).
    Если вы предпочитаете пользоваться необработанными запросами, то это просто
    и раскрыто в статье ":doc:`/cookbook/doctrine/dbal`" среди рецептов.

    Также можно хранить данные в `MongoDB`_ используя библиотеку Doctrine ODM. За
    дополнительной информацией обратитесь к статье ":doc:`/bundles/DoctrineMongoDBBundle/index`"
    из документации.

Простой пример: Product
-----------------------

Простейший путь для понимания Doctrine - это увидеть её в действии. В этом
разделе вы настроите базу данных, создадите объект ``Product`` (``Продукт``),
поместите его туда и получите обратно.

.. sidebar:: Код вместе с примером

    Если хотите придерживаться примера из этой главы создайте ``AcmeStoreBundle``:

    .. code-block:: bash

        php app/console generate:bundle --namespace=Acme/StoreBundle

Конфигурация базы данных
~~~~~~~~~~~~~~~~~~~~~~~~

Перед тем как действительно начать, необходимо настроить соединение с базой
данных. По соглашению эта информация обычно указывается в файле
``app/config/parameters.yml``:

.. code-block:: yaml

    #app/config/parameters.yml
    parameters:
        database_driver:   pdo_mysql
        database_host:     localhost
        database_name:     test_project
        database_user:     root
        database_password: password

.. note::

    Указание параметров в ``parameters.yml`` всего лишь соглашение. На них
    ссылается основной файл конфигурации, когда настраивается Doctrine:

    .. code-block:: yaml

        doctrine:
            dbal:
                driver:   %database_driver%
                host:     %database_host%
                dbname:   %database_name%
                user:     %database_user%
                password: %database_password%

    Разделяя информацию о базе данных по отдельным файлам, можно легко хранить
    различные версии этих файлов на каждом сервере. Также легко можно хранить
    конфигурацию базы данных (или любую важную информацию) вне проекта, например
    внутри конфигурации Apache. Дополнительная информация здесь
    :doc:`/cookbook/configuration/external_parameters`.

Теперь, когда Doctrine знает о базе данных, вы хотите чтобы она создала базу
данных для вас:

.. code-block:: bash

    php app/console doctrine:database:create

Создание сущностного класса
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Предположим, создаётся приложение, в котором необходимо показывать продукты.
Даже не задумываясь о Doctrine или базах данных, понятно что необходим объект
``Product`` чтобы представить эти продукты. Создайте его внутри папки ``Entity``
(``Сущность``) в ``AcmeStoreBundle``::

    // src/Acme/StoreBundle/Entity/Product.php
    namespace Acme\StoreBundle\Entity;

    class Product
    {
        protected $name;

        protected $price;

        protected $description;
    }

Этот класс - часто называемый "сущность", что значит *базовый класс, содержащий
данные* - простой и помогает выполнять бизнес требования к необходимым продуктам
в приложении. Он пока не может хранится в базе данных - он всего лишь простой
PHP класс.

.. tip::

    Однажды, когда вы изучите Doctrine, то сможете поручить ей создать этот
    класс-сущность:

    .. code-block:: bash

        php app/console doctrine:generate:entity --entity="AcmeStoreBundle:Product" --fields="name:string(255) price:float description:text"

.. index::
    single: Doctrine; Adding mapping metadata

.. _book-doctrine-adding-mapping:

Добавление информации об отображении
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine позволяет работать с базами данных гораздо более интересным способом
чем простое получение строк в массив из таблицы, основанной на колонках. Вместо
него, Doctrine хранить *объекты* целиком в базе данных и получать целые объекты
из неё. Это возможно благодаря отображению PHP класса в таблицу для базы данных
и свойств этого PHP класса в колонки этой таблицы:

.. image:: /images/book/doctrine_image_1.png
   :align: center

Чтобы Doctrine могла сделать это, надо просто создать "метаданные" или
конфигурацию, которые в точности расскажут ей как класс ``Product`` и его
свойства должны быть *отображены* в базу данных. Эти метаданные могут быть
указаны в большом количестве форматов, включая YAML, XML или прямо внутри класса
``Product`` через аннотации:

.. note::

    Bundle может принимать только один формат определения метаданных. Например,
    нельзя смешивать YAML определения метаданных и определения через аннотациии
    в классе-сущности PHP.

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Entity/Product.php
        namespace Acme\StoreBundle\Entity;

        use Doctrine\ORM\Mapping as ORM;

        /**
         * @ORM\Entity
         * @ORM\Table(name="product")
         */
        class Product
        {
            /**
             * @ORM\Id
             * @ORM\Column(type="integer")
             * @ORM\GeneratedValue(strategy="AUTO")
             */
            protected $id;

            /**
             * @ORM\Column(type="string", length=100)
             */
            protected $name;

            /**
             * @ORM\Column(type="decimal", scale=2)
             */
            protected $price;

            /**
             * @ORM\Column(type="text")
             */
            protected $description;
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.yml
        Acme\StoreBundle\Entity\Product:
            type: entity
            table: product
            id:
                id:
                    type: integer
                    generator: { strategy: AUTO }
            fields:
                name:
                    type: string
                    length: 100
                price:
                    type: decimal
                    scale: 2
                description:
                    type: text

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.xml -->
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                            http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

            <entity name="Acme\StoreBundle\Entity\Product" table="product">
                <id name="id" type="integer" column="id">
                    <generator strategy="AUTO" />
                </id>
                <field name="name" column="name" type="string" length="100" />
                <field name="price" column="price" type="decimal" scale="2" />
                <field name="description" column="description" type="text" />
            </entity>
        </doctrine-mapping>

.. tip::

    Имя таблицы необязательно и если опущено, то оно будет определено
    автоматически, исходя из названия класса-сущности.

Doctrine позволяет выбирать из широкого разнообразия различных типов полей,
каждый из которых со своими настройками. За информацией о доступных типах
обращайтесь к разделу :ref:`book-doctrine-field-types`.

.. seealso::

    Также можно обратиться к Doctrine-овой `Basic Mapping Documentation`_ за
    детальной информацией об отображении. Если будете использовать аннотации,
    необходимо предварять их, используя ``ORM\`` (например, ``ORM\Column(..)``),
    об этом не говорится в документации Doctrine. Также надо будет включать
    ``use Doctrine\ORM\Mapping as ORM;`` утверждение, которое *импортирует*
    ``ORM`` префикс для аннотаций.

.. caution::

    Будьте осторожны имена классов и свойств не отображаются в защищённые
    ключевые слова SQL (такие как ``group`` или ``user``). Например, если имя
    сущностного класса ``Group``, тогда, по умолчанию, таблица будет названа
    ``group``, что вызовет ошибку SQL в некоторых движках. Обратитесь к
    `документации по зарезервированным ключевым словам SQL`_ чтобы узнать как
    лучше экранировать такие имена.

.. note::

    Когда используется другая библиотека или программа (например, Doxygen),
    использующая аннотации, необходимо поместить в класс аннотацию
    ``@IgnoreAnnotation``, чтобы указать какие из них Symfony должен
    игнорировать.

    Например, чтобы уберечь ``@fn`` аннотацию от выдачи исключения, добавьте
    следующее::

        /**
         * @IgnoreAnnotation("fn")
         *
         */
        class Product

Создание геттеров и сеттеров
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Теперь, когда Doctrine знает как сохранить объект ``Product`` в базу данных,
сам класс пока ещё бесполезен. Так как ``Product`` всего лишь обычный PHP класс,
необходимо создать геттер и сеттер методы (например, ``getName()``,
``setName()``) чтобы получить доступ к его свойствам (т. к. свойства являются
``protected``). К счастью, Doctrine может сделать это по команде:

.. code-block:: bash

    php app/console doctrine:generate:entities Acme/StoreBundle/Entity/Product

Эта команда удостоверяется что все геттеры и сеттеры созданы для класса ``Product``.
Она безопасна - можно запускать её снова и снова: команда лишь создаёт геттеры и
сеттеры, которых ещё нет (т. о. она не изменит существующие методы).

.. caution::

    Команда ``doctrine:generate:entities`` сохраняет резервную копию исходного
    файла ``Product.php`` в ``Product.php~``. В некоторых случаях, присутствие
    этого файла может вызвать ошибку "Cannot redeclare class". Он может быть
    безопасно удалён.

Также можно создать все известные сущности (например, любой PHP класс с
информацией для отображения Doctrine) для бандла или целого пространства имён:

.. code-block:: bash

    php app/console doctrine:generate:entities AcmeStoreBundle
    php app/console doctrine:generate:entities Acme

.. note::

    Doctrine не интересует являются ли свойства ``protected`` или ``private``,
    или имеются либо нет функции геттеров или сеттеров для свойства. Геттеры и
    сеттеры создаются здесь только потому что они понадобятся для взаимодействия
    с PHP объектом.

Создание таблиц/схемы для базы данных
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Теперь есть удобный класс ``Product`` с информацией для отображения, который
Doctrine точно знает как сохранить. Конечно, пока нет соотвествующей таблицы
``product`` в базе данных. К счастью, Doctrine может автоматически создать все
таблицы базы данных, необходимые для всех известных сущностей приложения. Чтобы
создать их, выполните:

.. code-block:: bash

    php app/console doctrine:schema:update --force

.. tip::

    Эта команда необычайно мощная. Она сравнивает как *должна* выглядеть база
    данных (основываясь на информации об отображении для сущностей) с тем, как
    она выглядит *на самом деле*, и создаёт SQL выражения, необходимые для
    *обновления* базы данных до того вида, какой она должна быть. Другими
    словами, добавив новое свойство с метаданными отображения в ``Product`` и
    запустив её снова, она создаст выражение "alter table", необходимое для
    добавления этого нового столбца к существующей таблице ``product``.

    Лучший способ получить преимущества от её функциональности это
    :doc:`миграции</bundles/DoctrineMigrationsBundle/index>`, которые позволяют создавать
    эти SQL выражения и хранить их в миграционных классах, которые могут
    систематически запускаться на продакшн сервере чтобы соотвествовать схеме
    базы данных и изменять её безопасно и надёжно.

Теперь база данных имеет полноценную таблицу ``product`` со столбцами,
соотвествующими указанным метаданным.

Сохранение объектов в базе данных
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Теперь, когда есть отображённая сущность ``Product`` и соотвествующая таблица
``product``, всё готово к сохранению данных в базу. Внутри контроллера это
очень просто. Добавьте следующий метод в ``DefaultController`` бандла:

.. code-block:: php
    :linenos:

    // src/Acme/StoreBundle/Controller/DefaultController.php
    use Acme\StoreBundle\Entity\Product;
    use Symfony\Component\HttpFoundation\Response;
    // ...

    public function createAction()
    {
        $product = new Product();
        $product->setName('A Foo Bar');
        $product->setPrice('19.99');
        $product->setDescription('Lorem ipsum dolor');

        $em = $this->getDoctrine()->getEntityManager();
        $em->persist($product);
        $em->flush();

        return new Response('Created product id '.$product->getId());
    }

.. note::

    Если вы следуете этому примеру, необходимо создать маршрут, указывающий на
    это действие, чтобы увидеть его в работе.

Пройдёмся по примеру:

* **строки 8-11** В этой части, берётся экземпляр объекта ``$product`` и с ним
  проводится работа как с любым другим нормальным PHP объектом;

* **строка 13** Эта строка получает Doctrine-овый объект *entity manager*,
  отвественный за управление процессами сохранения и получения объектов из базы
  данных;

* **строка 14** Метод ``persist()`` сообщает Doctrine команду на "управление"
  объектом ``$product``. Она не вызывает создание запроса к базе данных (пока).

* **строка 15** Когда вызывается метод ``flush()``, Doctrine просматривает все
  объекты, которыми она управляет, чтобы узнать, надо ли сохранить их в базу
  данных. В этом примере объект ``$product`` ещё не был сохранён, поэтому
  entity manager выполнит запрос ``INSERT`` и будет создана строка в таблице
  ``product``.

.. note::

  Фактически, т. к. Doctrine знает обо всех управляемых сущностях, когда
  вызывается метод ``flush()``, она прощитывает общий набор изменений и
  выполняет наиболее эффективный и возможный запрос или запросы. Например, если
  сохраняется 100 объектов ``Product`` и впоследствии вызывается ``flush()``, то
  Doctrine создаст *единственное* подготовленное выражение и повторно использует
  его для каждой вставки. Этот паттерн называется *Unit of Work* и используется
  потомучто быстр и эффективен.

При создании или обновлении объектов рабочий процесс всегда одинаков. В
следующем разделе вы увидите что Doctrine достаточно умна чтобы автоматически
выдать запрос ``UPDATE`` если запись уже существует в базе данных.

.. tip::

    Doctrine предлагает библиотеку, позволяющую программно загружать тестовые
    данные в проект (т. н. "fixture data"). Информацию можно узнать в
    :doc:`/bundles/DoctrineFixturesBundle/index`.

Получение объектов из базы данных
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Получение объекта назад из базы данных ещё проще. Например, представим что
настроен маршрут, отображающий определённый ``Product``, основываясь на его
значении ``id``::

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        if (!$product) {
            throw $this->createNotFoundException('No product found for id '.$id);
        }

        // делает что-нибудь, например передаёт объект $product в шаблон
    }

Когда запрашивается объект определённого типа, всегда используется так
называемый "репозиторий". Можно представить репозиторий как PHP класс, чья
работа состоит в предоставлении помощи в получении сущностей определённого
класса. Можно получить доступ к объекту-репозиторию для класса-сущности через::

    $repository = $this->getDoctrine()
        ->getRepository('AcmeStoreBundle:Product');

.. note::

    Строка ``AcmeStoreBundle:Product`` - это сокращение, которое можно
    использовать в Doctrine вместо полного имени класса для сущности (например,
    ``Acme\StoreBundle\Entity\Product``). Оно будет работать пока сущность
    находится в простанстве имён ``Entity`` вашего бандла.

Когда имеется репозиторий, у вас есть доступ ко всем видам полезных методов::

    // запрос по первичному ключу (обычно "id")
    $product = $repository->find($id);

    // динамические имена методов, использующиеся для поиска по значению столбцов
    $product = $repository->findOneById($id);
    $product = $repository->findOneByName('foo');

    // ищет *все* продукты
    $products = $repository->findAll();

    // ищет группу продуктов, основываясь на произвольном значении столбца
    $products = $repository->findByPrice(19.99);

.. note::

    Конечно, также можно задавать сложные запросы, о которых вы узнаете больше
    в разделе :ref:`book-doctrine-queries`.

Также можно использовать преимущества полезных методов ``findBy`` и ``findOneBy``
для лёгкого извлечения объектов, основываясь на многочисленных условиях::

    // запрос одного продукта, подходящего по заданным имени и цене
    $product = $repository->findOneBy(array('name' => 'foo', 'price' => 19.99));

    // запрос всех продуктов, подходящих по имени и отсортированных по цене
    $product = $repository->findBy(
        array('name' => 'foo'),
        array('price' => 'ASC')
    );

.. tip::

    Когда выдаётся любая страница, можно увидеть сколько запросов было сделано в
    нижнем правом углу на панели инструментов web debug.

    .. image:: /images/book/doctrine_web_debug_toolbar.png
       :align: center
       :scale: 50
       :width: 350

    Если кликнуть на иконке, откроется профилировщик, показывающий точные
    запросы, которые были сделаны.

Обновление объекта
~~~~~~~~~~~~~~~~~~

Когда вы получили объект из Doctrine, обновить его также просто. Предположим,
есть маршрут, связывающий id продукта с действием обновления в контроллере::

    public function updateAction($id)
    {
        $em = $this->getDoctrine()->getEntityManager();
        $product = $em->getRepository('AcmeStoreBundle:Product')->find($id);

        if (!$product) {
            throw $this->createNotFoundException('No product found for id '.$id);
        }

        $product->setName('New product name!');
        $em->flush();

        return $this->redirect($this->generateUrl('homepage'));
    }

Обновление объекта включает три шага:

1. получение объкта из Doctrine;
2. изменение объекта;
3. вызов ``flush()`` из entity manager

Заметьте, что в вызове ``$em->persist($product)`` нет необходимости. Вспомните,
что этот метод лишь сообщает Doctrine что нужно управлять или "наблюдать" за
объектом ``$product``. В данной же ситуации, т. к. объект ``$product`` получен
из Doctrine, он уже является управляемым.

Удаление объекта
~~~~~~~~~~~~~~~~

Удаление объекта очень похоже, но требует вызова метода ``remove()`` из entity
manager::

    $em->remove($product);
    $em->flush();

Как и ожидалось, метод ``remove()`` уведомляет Doctrine о том, что вам хочется
удалить указанную сущность из базы данных. Тем не менее, фактический запрос
``DELETE`` не вызывается до тех пор, пока метод ``flush()`` не запущен.

.. _`book-doctrine-queries`:

Запрашивание объектов
---------------------

Вы уже видели как объект-репозиторий позволяет выполнять простые запросы без
какой-либо работы::

    $repository->find($id);

    $repository->findOneByName('Foo');

Конечно, Doctrine также позволяет писать более сложные запросы, используя
Doctrine Query Language (DQL). DQL похож на SQL за исключением того, что следует
представить что запрашиваются один или несколько объектов из класса-сущности
(например, ``Product``) вместо строк из таблицы (например, ``product``).

Запрашивать из Doctrine можно двумя способами: написанием чистых Doctrine
запросов либо использованием Doctrine-ового Query Builder.

Запрашивание объектов через DQL
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Представьте что нужно запросить продукты, но вернуть только те, чья цена больше
чем ``19.99`` и по порядку от дешёвого до самого дорогого. Внутри контроллера
сделайте следующее::

    $em = $this->getDoctrine()->getEntityManager();
    $query = $em->createQuery(
        'SELECT p FROM AcmeStoreBundle:Product p WHERE p.price > :price ORDER BY p.price ASC'
    )->setParameter('price', '19.99');

    $products = $query->getResult();

Если вам удобно с SQL, то DQL должен быть также понятен. Наибольшее различие
в том, что надо думать терминами "объектов", а не строк в базе данных. По этой
причине, вы выбираете *из* ``AcmeStoreBundle:Product`` и присваиваете ему
псевдоним ``p``.

Метод ``getResult()`` возвращает массив результатов. Если же нужен лишь один
объект можно воспользоваться методом ``getSingleResult()``::

    $product = $query->getSingleResult();

.. caution::

    Метод ``getSingleResult()`` выбрасывает исключение
    ``Doctrine\ORM\NoResultException`` если нет результатов и
    ``Doctrine\ORM\NonUniqueResultException`` если возвращается *больше* одного
    результата. Если используется этот метод, возможно придётся обернуть его
    в try-catch блок и убедиться в том, что возвращается только один результат
    (если запрашивается что-то, что может вероятно вернуть более одного
    результата)::

        $query = $em->createQuery('SELECT ....')
            ->setMaxResults(1);

        try {
            $product = $query->getSingleResult();
        } catch (\Doctrine\Orm\NoResultException $e) {
            $product = null;
        }
        // ...

Синтаксис DQL невероятно мощный, позволяет легко устанавливать объединения между
сущностями (тема :ref:`отношений<book-doctrine-relations>` будет раскрыта
позже), группами и т. д. Дополнительная информация в документации Doctrine
`Doctrine Query Language`_.

.. sidebar:: Настройка параметров

    Заметка о методе ``setParameter()``. Работая с Doctrine, хорошим тоном
    является указание любых внешних значений через "placeholders",
    что и было сделанов приведённом выше примере:

    .. code-block:: text

        ... WHERE p.price > :price ...

    Позже можно указать значение ``price`` placeholder через метод
    ``setParameter()``::

        ->setParameter('price', '19.99')

    Использование параметров вместо установки значений непосредственно в строку
    запроса предотвращает атаки через SQL инъекции и должно использоваться
    *всегда*. При использовании нескольких параметров, можно указать их за один
    раз воспользовавшись методом ``setParameters()``::

        ->setParameters(array(
            'price' => '19.99',
            'name'  => 'Foo',
        ))

Использование Doctrine's Query Builder (Конструктор запросов Doctrine)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Вместо непосредственного написания запросов, можно также использовать Doctrine
``QueryBuilder`` чтобы сделать ту же работу используя симпатичный,
объект-ориентированный интерфейс. Если используется IDE, то можно также получить
преимущество от авто-подстановки когда будут вводиться имена методов. Внутри
контроллера::

    $repository = $this->getDoctrine()
        ->getRepository('AcmeStoreBundle:Product');

    $query = $repository->createQueryBuilder('p')
        ->where('p.price > :price')
        ->setParameter('price', '19.99')
        ->orderBy('p.price', 'ASC')
        ->getQuery();

    $products = $query->getResult();

Объект ``QueryBuilder`` содержит все необходимые методы для создания запроса.
Вызвав метод ``getQuery()``, конструктор запросов вернёт нормальный объект
``Query``, являющийся таким же объектом, какой создавался в предыдущем разделе.

За дополнительной информацией о Doctrine's Query Builder, обращайтесь к
документации `Query Builder`_.

Custom Repository Classes
~~~~~~~~~~~~~~~~~~~~~~~~~

В предыдущих разделах вы начали создавать и использовать более сложные запросы
изнутри контроллера. Чтобы изолировать, тестировать и повторно использовать
их, хорошим тоном будет создать custom repository class для сущности и добавить
в него методы с запросами.

Чтобы сделать это добавьте имя репозиторного класса в отбражение.

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Entity/Product.php
        namespace Acme\StoreBundle\Entity;

        use Doctrine\ORM\Mapping as ORM;

        /**
         * @ORM\Entity(repositoryClass="Acme\StoreBundle\Repository\ProductRepository")
         */
        class Product
        {
            //...
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.yml
        Acme\StoreBundle\Entity\Product:
            type: entity
            repositoryClass: Acme\StoreBundle\Repository\ProductRepository
            # ...

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.xml -->
        <!-- ... -->
        <doctrine-mapping>

            <entity name="Acme\StoreBundle\Entity\Product"
                    repository-class="Acme\StoreBundle\Repository\ProductRepository">
                    <!-- ... -->
            </entity>
        </doctrine-mapping>

Doctrine может создать репозиторный класс с помощью команды, использованной
ранее для создания пропущенных getter и setter методов:

.. code-block:: bash

    php app/console doctrine:generate:entities Acme

Затем добавьте новый метод - ``findAllOrderedByName()`` - к только что
созданному репозитороному классу. Он будет запрашивать все сущности ``Product``,
сортированные в алфавитном порядке.

.. code-block:: php

    // src/Acme/StoreBundle/Repository/ProductRepository.php
    namespace Acme\StoreBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    class ProductRepository extends EntityRepository
    {
        public function findAllOrderedByName()
        {
            return $this->getEntityManager()
                ->createQuery('SELECT p FROM AcmeStoreBundle:Product p ORDER BY p.name ASC')
                ->getResult();
        }
    }

.. tip::

    Менеджер сущностей доступен через ``$this->getEntityManager()`` внутри
    репозитория.

Можете использовать этот новый метод как и ранее доступные по умолчанию
поисковые методы репозитория::

    $em = $this->getDoctrine()->getEntityManager();
    $products = $em->getRepository('AcmeStoreBundle:Product')
                ->findAllOrderedByName();

.. note::

    Когда используется custom repository class, всё ещё есть доступ к таким
    поисковым методам как ``find()`` и ``findAll()``.

.. _`book-doctrine-relations`:

Связи/объединения сущностей
---------------------------

Предположим что все продукты в приложении принадлежат единственной "категории".
В этом случае, необходим объект ``Category`` и способ связывания его с объектом
``Product``. Начнём с соаздания сущности ``Category``. Так как известно что в
конечном счёте понадобится сохранить класс с помощью Doctrine, то можно
позволить Doctrine создать его для вас.

.. code-block:: bash

    php app/console doctrine:generate:entity --entity="AcmeStoreBundle:Category" --fields="name:string(255)"

Это задание создаст сущность ``Category`` с полями ``id``, ``name`` и связанными
getter и setter функциями.

Метаданные отображения связей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы связать сущности ``Category`` и ``Product``, начните с создания свойства
``products`` в классе ``Category``::

    // src/Acme/StoreBundle/Entity/Category.php
    // ...
    use Doctrine\Common\Collections\ArrayCollection;

    class Category
    {
        // ...

        /**
         * @ORM\OneToMany(targetEntity="Product", mappedBy="category")
         */
        protected $products;

        public function __construct()
        {
            $this->products = new ArrayCollection();
        }
    }

Во-первых, т. к. объект ``Category`` связан со множеством объектов ``Product``,
то добавленное свойство ``products`` будет массивом для хранения объектов
``Product``. Далее, this isn't done because Doctrine needs it, but instead because it
makes sense in the application for each ``Category`` to hold an array of
``Product`` objects.

.. note::

    Код в методе ``__construct()`` важен, потому что Doctrine необходимо
    чтобы свойство ``$products`` было объектом ``ArrayCollection``. Этот объект
    выглядит и работает почти *также* как массив, но имеет расширенную гибкость.
    Если это заставляет вас чувствовать неудобство, то не переживайте.
    Представьте что это просто ``массив`` и вы будете снова в хорошей форме.

Далее, т. к. каждый класс ``Product`` может связываться только с одним объектом
``Category``, необходимо добавить свойство ``$category`` к классу ``Product``::

    // src/Acme/StoreBundle/Entity/Product.php
    // ...

    class Product
    {
        // ...

        /**
         * @ORM\ManyToOne(targetEntity="Category", inversedBy="products")
         * @ORM\JoinColumn(name="category_id", referencedColumnName="id")
         */
        protected $category;
    }

Наконец, когда добавлены новые свойства к обоим классам ``Category`` и
``Product``, сообщите Doctrine что надо создать отсутствующие методы getter и
setter:

.. code-block:: bash

    php app/console doctrine:generate:entities Acme

Забудьте о метаданных Doctrine на секунду. Имеется два класса - ``Category``
и ``Product`` with a natural one-to-many relationship. Класс ``Category``
holds массив объектов ``Product`` и объект ``Product`` может hold один объект
``Category``. Другими словами - классы построены таким способом, который имеет
смысл для вашей задачи. А тот факт, что данные должны быть сохранены в базу
данных, всегда второстепенен.

Теперь взгляните на метаданные над свойством ``$category`` в классе ``Product``.
Эта информация сообщает doctrine что связанным классом является ``Category`` и
что он должен хранить ``id`` от записи категории в поле ``category_id``,
находящемся в таблице ``product``. Другими словами, связанный объект ``Category``
будет хранится в свойстве ``$category``, но, за кулисами, Doctrine будет хранить
эту связь, записывая значение id категории в столбец ``category_id`` таблицы
``product``.

.. image:: /images/book/doctrine_image_2.png
   :align: center

Метаданные над свойством ``$products`` объекта ``Category`` менее важны и
попросту сообщают Doctrine что нужно посмотреть свойство ``Product.category``
чтобы вычислить как отображается связь.

Перед тем как продолжить, убедитесь что сообщили Doctrine добавить новые таблицу
``category`` и столбец ``product.category_id``, а также новый внешний ключ:

.. code-block:: bash

    php app/console doctrine:schema:update --force

.. note::

    Эта задача должна выполняться только во время разработки. Более надёжный
    способ систематических обновлений производственной базы данных описан в
    :doc:`Миграциях Doctrine</bundles/DoctrineFixturesBundle/index>`.

Сохранение связанных сущностей
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Теперь давайте посмотрим код в действии. Представьте, что вы внутри контроллера::

    // ...
    use Acme\StoreBundle\Entity\Category;
    use Acme\StoreBundle\Entity\Product;
    use Symfony\Component\HttpFoundation\Response;
    // ...

    class DefaultController extends Controller
    {
        public function createProductAction()
        {
            $category = new Category();
            $category->setName('Main Products');

            $product = new Product();
            $product->setName('Foo');
            $product->setPrice(19.99);
            // Связывает этот продукт с категорией
            $product->setCategory($category);

            $em = $this->getDoctrine()->getEntityManager();
            $em->persist($category);
            $em->persist($product);
            $em->flush();

            return new Response(
                'Created product id: '.$product->getId().' and category id: '.$category->getId()
            );
        }
    }

Итак, одна строка добавлена в таблицы ``category`` и ``product``.
В столбец ``product.category_id`` для нового продукта установлен тот ``id``,
который соотвествует новой категории. Doctrine осуществляет сохранение этой
связи для вас.

Получение связанных объектов
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Когда необходимо получить объединённые объекты, рабочий процесс выглядит также
как и раньше. Сначала получаете объект ``$product``, а затем доступ к связанной
``Category``::

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        $categoryName = $product->getCategory()->getName();

        // ...
    }

В этом примере, сначала запрашивается объект ``Product`` по ``id`` продукта.
Этот запрос выдаёт ответ *только* для данных о продукте и гидратирует (hydrate)
объект ``$product`` с этими данными. Затем, когда вызовется
``$product->getCategory()->getName()``, Doctrine без лишнего шума сделает второй
запрос, чтобы найти ``Category``, которая связана с этим ``Product``. Она
подготовит объект ``$category`` и возвратит его вам.

.. image:: /images/book/doctrine_image_3.png
   :align: center

Важен тот факт, что у вас есть простой доступ к категории, связанной с
продуктом, но её данные не извлекаются, пока она вам не понадобится (т. е. это
"ленивая загрузка").

Также можно запросить в другом направлении::

    public function showProductAction($id)
    {
        $category = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Category')
            ->find($id);

        $products = $category->getProducts();

        // ...
    }

В этом случае происходят похожие дела: сначала запрашиваете один объект
``Category``, затем Doctrine делает второй запрос для получения связанных
объектов ``Product``, но только однажды - когда они вам понадобятся (т. е. когда
вызывается ``->getProducts()``). Переменная ``$products`` является массивом всех
объектов ``Product``, связанных с данным объектом ``Category`` через значение их
``category_id``.

.. sidebar:: Связи и proxy классы

    Эта "ленивая загрузка" возможна, когда необходима, потому, что Doctrine
    возвращает "proxy" объект вместо настоящего объекта. Взгляните снова на
    пример, приведённый ранее::

        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        $category = $product->getCategory();

        // prints "Proxies\AcmeStoreBundleEntityCategoryProxy"
        echo get_class($category);

    Этот proxy объект расширяет настоящий объект ``Category``, и выглядит и
    действует так же как и он. Отличие лишь в том, что используя proxy объект,
    Doctrine может отложить запрос действительных данных о ``Category`` пока
    они вам не понадобятся (т. е. пока не вызовете ``$category->getName()``).

    Proxy классы создаются Doctrine и хранятся в папке cache. И хотя вам,
    вероятно, никогда не придётся принимать во внимание что объект ``$category``
    на самом деле является proxy объектом, но важно знать об этом.

    В следующем разделе будем получать данные о продукте и категории за один
    заход (через *join*), а Doctrine будет возвращать *настоящий* объект
    ``Category``, т. к. не будет нужды в ленивой загрузке.

Объединение со связанными записями
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

В предыдущих примерах выполнялось по два запроса - один для исходного объекта
(например, ``Category``) и один для связанного (например, объекты ``Product``).

.. tip::

    Вспомните, что можно увидеть все запросы к базе данных, сделанные во время
    веб-запроса, через панель инструментов web debug.

Конечно, если заранее известно что будет необходим доступ к обоим объектам, то
можно избежать второго запроса, используя join в исходном запросе. Добавьте
следующий метод к классу ``ProductRepository``::

    // src/Acme/StoreBundle/Repository/ProductRepository.php

    public function findOneByIdJoinedToCategory($id)
    {
        $query = $this->getEntityManager()
            ->createQuery('
                SELECT p, c FROM AcmeStoreBundle:Product p
                JOIN p.category c
                WHERE p.id = :id'
            )->setParameter('id', $id);

        try {
            return $query->getSingleResult();
        } catch (\Doctrine\ORM\NoResultException $e) {
            return null;
        }
    }

Теперь можете использовать этот метод в контроллере чтобы получать объект
``Product`` и связанную ``Category`` за один запрос::

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->findOneByIdJoinedToCategory($id);

        $category = $product->getCategory();

        // ...
    }

Подробнее об объединениях
~~~~~~~~~~~~~~~~~~~~~~~~~

Этот раздел является введением к одному общему типу связи сущностей - связи
один-ко-многим. За более продвинутыми подробностями и примерами использования
других типов связей (напр., ``один-к-одному``, ``многие-ко-многим``), обращайтесь
к `Отображениям объединений`_ для Doctrine.

.. note::

    Если использовать аннотации, необходимо предварять их упоминаниями об
    ``ORM\`` (напр., ``ORM\OneToMany``), про это не говорится в документации
    Doctrine. Также необходимо включить выражение
    ``use Doctrine\ORM\Mapping as ORM;``, которое *внедряет* префикс аннотации
    ``ORM``.

Конфигурация
------------

Doctrine очень гибка, хотя вам, вероятно, никогда не придёться беспокоиться о
большей части её опций. Чтобы узнать больше о настройке Doctrine, see
the Doctrine section of the :doc:`reference manual</reference/configuration/doctrine>`.

Lifecycle Callbacks
-------------------

Иногда требуется выполнить действия сразу же перед или после того как сущность
будет вставлена, обновлена или же удалена. Такие типы действий известны как
"lifecycle" callbacks, т. к. они вызывают методы, которые необходимо выполнить
во время различных стадий жизненного цикла сущности (напр., сущность вставлена,
обновлена, удалена и т. д.).

Если для метаданных вы используете аннотации, то начните с включения lifecycle
callbacks. В этом нет необходимости если для отображений используются YAML или
XML:

.. code-block:: php-annotations

    /**
     * @ORM\Entity()
     * @ORM\HasLifecycleCallbacks()
     */
    class Product
    {
        // ...
    }

Теперь можно дать задание Doctrine выполнить метод для любого доступного события
жизненного цикла. Например, надо установить текущую дату в колонку ``created``
только во время первого сохранения сущности (т. е. во время вставки):

.. configuration-block::

    .. code-block:: php-annotations

        /**
         * @ORM\prePersist
         */
        public function setCreatedValue()
        {
            $this->created = new \DateTime();
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.yml
        Acme\StoreBundle\Entity\Product:
            type: entity
            # ...
            lifecycleCallbacks:
                prePersist: [ setCreatedValue ]

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.xml -->
        <!-- ... -->
        <doctrine-mapping>

            <entity name="Acme\StoreBundle\Entity\Product">
                    <!-- ... -->
                    <lifecycle-callbacks>
                        <lifecycle-callback type="prePersist" method="setCreatedValue" />
                    </lifecycle-callbacks>
            </entity>
        </doctrine-mapping>

.. note::

    Предыдущие примеры предполагают что свойство ``created`` уже создано и
    отображено (здесь это не было показано).

Сразу же перед первым сохранением сущности, Doctrine автоматически вызовет этот
метод и в поле ``created`` будет установлена текущая дата.

То же самое можно проделать для любого другого события жизненного цикла, среди
которых:

* ``preRemove``
* ``postRemove``
* ``prePersist``
* ``postPersist``
* ``preUpdate``
* ``postUpdate``
* ``postLoad``
* ``loadClassMetadata``

Дополнительная информация о том, что из себя представляют эти события и вызовы
внутри жизненного цикла в общем виде, находится в `Документации по Lifecycle Events`_

.. sidebar:: Lifecycle Callbacks и Event Listeners

    Обратите внимание что метод ``setCreatedValue()`` не получает аргументов.
    Это необходимость для lifecylce callbacks и это сделано преднамеренно:
    lifecycle callbacks должны быть простыми методами, занимающимися внутренними
    изменениями данных для сущности (напр., установка значений для полей
    created/updated, создание slug).

    Если планируется делать более тяжёлую работу - запись логов или отправка
    email - необходимо зарегистрировать внешний класс как event listener
    или subscriber и дать ему доступ к необходимым ресурсам. Дополнительную
    информацию найдёте в :doc:`/cookbook/doctrine/event_listeners_subscribers`.

Расширения для Doctrine: Timestampable, Sluggable и другие
----------------------------------------------------------

Doctrine расширяема, поэтому доступно множество сторонних решений, позволяющих с
лёгкостью выполнять повторяющиеся и общие задачи над сущностями.
Среди них есть следующие: *Sluggable*, *Timestampable*, *Loggable*,
*Translatable* и *Tree*.

Подробнее о том где найти и как использвать эти расширения расказывает статья
:doc:`Использование общих расширений Doctrine</cookbook/doctrine/common_extensions>`.

.. _book-doctrine-field-types:

Справка по типам полей в Doctrine
---------------------------------

Doctrine представляет огромное количество типов полей. Каждый из которых
отображает тип данных из PHP в установленный тип колонки для любой используемой
базы данных. В Doctrine поддерживаются следующие типы:

* **Строки**

  * ``string`` (используется для коротких строк)
  * ``text`` (используется для длинных строк)

* **Числа**

  * ``integer``
  * ``smallint``
  * ``bigint``
  * ``decimal``
  * ``float``

* **Дата и время** (используйте объект `DateTime`_ в PHP для этих полей)

  * ``date``
  * ``time``
  * ``datetime``

* **Другие типы**

  * ``boolean``
  * ``object`` (сериализуется и хранится в поле ``CLOB``)
  * ``array`` (сериализуется и хранится в поле ``CLOB``)

Дополнительная информация содержится в `Отображении типов`_.

Опции полей
~~~~~~~~~~~

Каждое поле может иметь набор опций, применимых к нему. Доступные опции включают:
``type`` (стандартный для ``string``), ``name``, ``length``, ``unique`` и
``nullable``. Несколько примеров таких аннотаций:

.. code-block:: php-annotations

    /**
     * Строковое поле длиной 255, которое не должно быть null
     * (это стандартные значения для опций "type", "length" и *nullable*)
     *
     * @ORM\Column()
     */
    protected $name;

    /**
     * Строковое поле длиной 150, хранящееся в колонке "email_address"
     * и имеющее уникальный индекс.
     *
     * @ORM\Column(name="email_address", unique="true", length="150")
     */
    protected $email;

.. note::

    Существуют ещё опции, о которых здесь не упоминается. За дополнительной
    информацией обращайтесь к документации Doctrine's `Property Mapping documentation`_

.. index::
   single: Doctrine; ORM Console Commands
   single: CLI; Doctrine ORM

Консольные команды
------------------

Интеграция Doctrine2 ORM предлагает несколько консольных команд внутри
пространства имён ``doctrine``. Чтобы вывести список команд запустите консоль
без аргументов:

.. code-block:: bash

    php app/console

В выведенном списке доступных команд многие из них начинаются с префикса
``doctrine:``. Подробнее о них (или любых других командах для Symfony) можно
узнать запустив команду ``help``. Например, чтобы получить подробности о
процессе ``doctrine:database:create``, запустите:

.. code-block:: bash

    php app/console help doctrine:database:create

Некоторые интересные или примечательные команды включают:

* ``doctrine:ensure-production-settings`` - проверяет текущее окружение,
  настроено ли оно эффективно для производственных нужд. Она всегда должна
  запускаться в окружении ``prod``:

  .. code-block:: bash

    php app/console doctrine:ensure-production-settings --env=prod

* ``doctrine:mapping:import`` - разрешает Doctrine проанализировать существующую
  базу данных и создать информацию для её отображения. За дополнительной
  информацией обращайтесь к :doc:`/cookbook/doctrine/reverse_engineering`.

* ``doctrine:mapping:info`` - расскажет обо всех сущностях, которые знает
  Doctrine, а также есть ли в отображениях какие-нибудь простые ошибки.

* ``doctrine:query:dql`` и ``doctrine:query:sql`` - позволяет выполнять DQL или
  SQL запросы прямо из командной строки.

.. note::

   Чтобы иметь возможность загружать fixtures с данными в базу данных,
   необходимо установить бандл ``DoctrineFixturesBundle``. Чтобы узнать как это
   сделать, прочтите статью ":doc:`/bundles/DoctrineFixturesBundle/index`" в
   документации.

Выводы
------

Применяя Doctrine, можно сфокусироваться на объектах и их использовании в
приложении и только потом заботиться об их сохранении в базу данных. Благодаря
тому, что Doctrine позволяет использовать любой объект PHP для хранения данных и
применяет информацию метаданных для отображения чтобы отобразить эти данные об
объекте в определённую таблицу базы данных.

Хотя в основе Doctrine простая идея, она необычайно мощна, позволяет
создавать сложные запросы и подписываться на события, которые дают возможность
совершать различные действия когда объекты проходят по своим жизненным циклам во
время сохранения.

За дополнительной информацией о Doctrine обращайтесь к разделу *Doctrine* из
:doc:`Книги рецептов</cookbook/index>`, который включает следующие статьи:

* :doc:`/bundles/DoctrineFixturesBundle/index`
* :doc:`/cookbook/doctrine/common_extensions`

.. _`Doctrine`: http://www.doctrine-project.org/
.. _`MongoDB`: http://www.mongodb.org/
.. _`Basic Mapping Documentation`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html
.. _`Query Builder`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/query-builder.html
.. _`Doctrine Query Language`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/dql-doctrine-query-language.html
.. _`Отображениям объединений`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/association-mapping.html
.. _`DateTime`: http://php.net/manual/en/class.datetime.php
.. _`Отображении типов`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#doctrine-mapping-types
.. _`Property Mapping documentation`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#property-mapping
.. _`Документации по Lifecycle Events`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html#lifecycle-events
.. _`документации по зарезервированным ключевым словам SQL`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#quoting-reserved-words

.. toctree::
    :hidden:

    Translation source: N/A
    Corrected from: 2011-09-30 d739c57
