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
    дополнительной информацией обратитесь к статье ":doc:`/cookbook/doctrine/mongodb`"
    рецептов.

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
``app/config/parameters.ini``:

.. code-block:: ini

    ;app/config/parameters.ini
    [parameters]
        database_driver   = pdo_mysql
        database_host     = localhost
        database_name     = test_project
        database_user     = root
        database_password = password

.. note::

    Указание параметров в ``parameters.ini`` всего лишь соглашение. На них
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
    
        php app/console doctrine:generate:entity AcmeStoreBundle:Product "name:string(255) price:float description:text"

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
             * @ORM\Column(type="string", length="100")
             */
            protected $name;

            /**
             * @ORM\Column(type="decimal", scale="2")
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

.. tip::

    Когда используется другая библиотека или программа (например, Doxygen),
    использующая аннотации, необходимо использовать аннотацию
    ``@IgnoreAnnotation``, чтобы указать, какие из них Symfony и Doctrine должны
    игнорировать. Она должна размещаться в блоке-комментарие того класса,
    которому он принадлежит. Её отсутствие может привести к выбросу исключения.
    
    Например, чтобы уберечь ``@fn`` аннотацию от выдачи исключения, добавьте
    следующее::
    
        /**
         * @IgnoreAnnotation("fn")
         * 
         */
        class Product

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

.. note::

    Doctrine не интересует являются ли свойства ``protected`` или ``private``,
    или имеются либо нет функции геттеров или сеттеров для свойства. Геттеры и
    сеттеры создаются здесь только потому что они понадобятся для взаимодействия
    с PHP объектом.

.. tip::

    Также можно создать все известные сущности (например, любой PHP класс с
    информацией для отображения Doctrine) для бандла или целого пространства
    имён:

    .. code-block:: bash

        php app/console doctrine:generate:entities AcmeStoreBundle
        php app/console doctrine:generate:entities Acme

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
    добавления этого нового столбца к существующей таблице ``products``.

    Лучший способ получить преимущества от её функциональности это
    :doc:`миграции</cookbook/doctrine/migrations>`, которые позволяют создавать
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

* **строки 7-10** В этой части, берётся экземпляр объекта ``$product`` и с ним
  проводится работа как с любым другим нормальным PHP объектом;

* **строка 12** Эта строка получает Doctrine-овый объект *entity manager*,
  отвественный за управление процессами сохранения и получения объектов из базы
  данных;

* **строка 13** Метод ``persist()`` сообщает Doctrine команду на "управление"
  объектом ``$product``. Она не вызывает создание запроса к базе данных (пока).

* **строка 14** Когда вызывается метод ``flush()``, Doctrine просматривает все
  объекты, которыми она управляет, чтобы узнать, надо ли сохранить их в базу
  данных. В этом примере объект ``$product`` ещё не был сохранён, поэтому
  entity manager выполнит запрос ``INSERT`` и будет создана строка в таблице
  ``product``.

.. note::

  Фактически, т. к. Doctrine знает обо всех управляемых сущностях, когда
  вызывается метод ``flush()``, она прощитывает общий набор изменений и
  выполняет наиболее эффективный и возможный запрос или запросы. Например, если
  сохраняется 100 объектов ``Product`` и затем вызывается ``persist()``, то
  Doctrine создаст *единственное* подготовленное выражение и повторно использует
  его для каждой вставки. Этот паттерн называется *Unit of Work* и используется
  потомучто быстр и эффективен.

При создании или обновлении объектов рабочий процесс всегда одинаков. В
следующем разделе вы увидите что Doctrine достаточно умна чтобы автоматически
выдать запрос ``UPDATE`` если запись уже существует в базе данных.

.. tip::

    Doctrine предлагает библиотеку, позволяющую программно загружать тестовые
    данные в проект (т. н. "fixture data"). Информацию можно узнать в
    :doc:`/cookbook/doctrine/doctrine_fixtures`.

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
        array('price', 'ASC')
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

Вы уже видели как объект-репозиторий позволяет запускать простые запросы без
како-либо работы::

    $repository->find($id);
    
    $repository->findOneByName('Foo');

Конечно, Doctrine также позволяет писать более сложные запросы, используя
Doctrine Query Language (DQL). DQL похож на SQL за исключением того, что следует
представить что запрашиваются один или несколько объектов из класса-сущности
(например, ``Product``) вместо строк из таблицы (например, ``product``).

Запрашивать из Doctrine можно двумя способами: написанием чистых Doctrine
запросов либо использованием Doctrine-ового Query Builder.

Querying for Objects with DQL
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Imaging that you want to query for products, but only return products that
cost more than ``19.99``, ordered from cheapest to most expensive. From inside
a controller, do the following::

    $em = $this->getDoctrine()->getEntityManager();
    $query = $em->createQuery(
        'SELECT p FROM AcmeStoreBundle:Product p WHERE p.price > :price ORDER BY p.price ASC'
    )->setParameter('price', '19.99');
    
    $products = $query->getResult();

If you're comfortable with SQL, then DQL should feel very natural. The biggest
difference is that you need to think in terms of "objects" instead of rows
in a database. For this reason, you select *from* ``AcmeStoreBundle:Product``
and then alias it as ``p``.

The ``getResult()`` method returns an array of results. If you're querying
for just one object, you can use the ``getSingleResult()`` method instead::

    $product = $query->getSingleResult();

.. caution::

    The ``getSingleResult()`` method throws a ``Doctrine\ORM\NoResultException``
    exception if no results are returned and a ``Doctrine\ORM\NonUniqueResultException``
    if *more* than one result is returned. If you use this method, you may
    need to wrap it in a try-catch block and ensure that only one result is
    returned (if you're querying on something that could feasibly return
    more than one result)::
    
        $query = $em->createQuery('SELECT ....')
            ->setMaxResults(1);
        
        try {
            $product = $query->getSingleResult();
        } catch (\Doctrine\Orm\NoResultException $e) {
            $product = null;
        }
        // ...

The DQL syntax is incredibly powerful, allowing you to easily join between
entities (the topic of :ref:`relations<book-doctrine-relations>` will be
covered later), group, etc. For more information, see the official Doctrine
`Doctrine Query Language`_ documentation.

.. sidebar:: Setting Parameters

    Take note of the ``setParameter()`` method. When working with Doctrine,
    it's always a good idea to set any external values as "placeholders",
    which was done in the above query:
    
    .. code-block:: text

        ... WHERE p.price > :price ...

    You can then set the value of the ``price`` placeholder by calling the
    ``setParameter()`` method::

        ->setParameter('price', '19.99')

    Using parameters instead of placing values directly in the query string
    is done to prevent SQL injection attacks and should *always* be done.
    If you're using multiple parameters, you can set their values at once
    using the ``setParameters()`` method::

        ->setParameters(array(
            'price' => '19.99',
            'name'  => 'Foo',
        ))

Using Doctrine's Query Builder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of writing the queries directly, you can alternatively use Doctrine's
``QueryBuilder`` to do the same job using a nice, object-oriented interface.
If you use an IDE, you can also take advantage of auto-completion as you
type the method names. From inside a controller::

    $repository = $this->getDoctrine()
        ->getRepository('AcmeStoreBundle:Product');

    $query = $repository->createQueryBuilder('p')
        ->where('p.price > :price')
        ->setParameter('price', '19.99')
        ->orderBy('p.price', 'ASC')
        ->getQuery();
    
    $products = $query->getResult();

The ``QueryBuilder`` object contains every method necessary to build your
query. By calling the ``getQuery()`` method, the query builder returns a
normal ``Query`` object, which is the same object you built directly in the
previous section.

For more information on Doctrine's Query Builder, consult Doctrine's
`Query Builder`_ documentation.

Custom Repository Classes
~~~~~~~~~~~~~~~~~~~~~~~~~

In the previous sections, you began constructing and using more complex queries
from inside a controller. In order to isolate, test and reuse these queries,
it's a good idea to create a custom repository class for your entity and
add methods with your query logic there.

To do this, add the name of the repository class to your mapping definition.

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

Doctrine can generate the repository class for you by running the same command
used earlier to generate the missing getter and setter methods:

.. code-block:: bash

    php app/console doctrine:generate:entities Acme

Next, add a new method - ``findAllOrderedByName()`` - to the newly generated
repository class. This method will query for all of the ``Product`` entities,
ordered alphabetically.

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

    The entity manager can be accessed via ``$this->getEntityManager()``
    from inside the repository.

You can use this new method just like the default finder methods of the repository::

    $em = $this->getDoctrine()->getEntityManager();
    $products = $em->getRepository('AcmeStoreBundle:Product')
                ->findAllOrderedByName();

.. note::

    When using a custom repository class, you still have access to the default
    finder methods such as ``find()`` and ``findAll()``.

.. _`book-doctrine-relations`:

Entity Relationships/Associations
---------------------------------

Suppose that the products in your application all belong to exactly one "category".
In this case, you'll need a ``Category`` object and a way to relate a ``Product``
object to a ``Category`` object. Start by creating the ``Category`` entity.
Since you know that you'll eventually need to persist the class through Doctrine,
you can let Doctrine create the class for you:

.. code-block:: bash

    php app/console doctrine:generate:entity AcmeStoreBundle:Category "name:string(255)" --mapping-type=yml

This task generates the ``Category`` entity for you, with an ``id`` field,
a ``name`` field and the associated getter and setter functions.

Relationship Mapping Metadata
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To relate the ``Category`` and ``Product`` entities, start by creating a
``products`` property on the ``Category`` class::

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

First, since a ``Category`` object will relate to many ``Product`` objects,
a ``products`` array property is added to hold those ``Product`` objects.
Again, this isn't done because Doctrine needs it, but instead because it
makes sense in the application for each ``Category`` to hold an array of
``Product`` objects.

.. note::

    The code in the ``__construct()`` method is important because Doctrine
    requires the ``$products`` property to be an ``ArrayCollection`` object.
    This object looks and acts almost *exactly* like an array, but has some
    added flexibility. If this makes you uncomfortable, don't worry. Just
    imagine that it's an ``array`` and you'll be in good shape.

Next, since each ``Product`` class can relate to exactly one ``Category``
object, you'll want to add a ``$category`` property to the ``Product`` class::

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

Finally, now that you've added a new property to both the ``Category`` and
``Product`` classes, tell Doctrine to generate the missing getter and setter
methods for you:

.. code-block:: bash

    php app/console doctrine:generate:entities Acme

Ignore the Doctrine metadata for a moment. You now have two classes - ``Category``
and ``Product`` with a natural one-to-many relationship. The ``Category``
class holds an array of ``Product`` objects and the ``Product`` object can
hold one ``Category`` object. In other words - you've built your classes
in a way that makes sense for your needs. The fact that the data needs to
be persisted to a database is always secondary.

Now, look at the metadata above the ``$category`` property on the ``Product``
class. The information here tells doctrine that the related class is ``Category``
and that it should store the ``id`` of the category record on a ``category_id``
field that lives on the ``product`` table. In other words, the related ``Category``
object will be stored on the ``$category`` property, but behind the scenes,
Doctrine will persist this relationship by storing the category's id value
on a ``category_id`` column of the ``product`` table.

.. image:: /images/book/doctrine_image_2.png
   :align: center

The metadata above the ``$products`` property of the ``Category`` object
is less important, and simply tells Doctrine to look at the ``Product.category``
property to figure out how the relationship is mapped.

Before you continue, be sure to tell Doctrine to add the new ``category``
table, and ``product.category_id`` column, and new foreign key:

.. code-block:: bash

    php app/console doctrine:schema:update --force

.. note::

    This task should only be really used during development. For a more robust
    method of systematically updating your production database, read about
    :doc:`Doctrine migrations</cookbook/doctrine/migrations>`.

Saving Related Entities
~~~~~~~~~~~~~~~~~~~~~~~

Now, let's see the code in action. Imagine you're inside a controller::

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
            // relate this product to the category
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

Now, a single row is added to both the ``category`` and ``product`` tables.
The ``product.category_id`` column for the new product is set to whatever
the ``id`` is of the new category. Doctrine manages the persistence of this
relationship for you.

Fetching Related Objects
~~~~~~~~~~~~~~~~~~~~~~~~

When you need to fetch associated objects, your workflow looks just like it
did before. First, fetch a ``$product`` object and then access its related
``Category``::

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        $categoryName = $product->getCategory()->getName();
        
        // ...
    }

In this example, you first query for a ``Product`` object based on the product's
``id``. This issues a query for *just* the product data and hydrates the
``$product`` object with that data. Later, when you call ``$product->getCategory()->getName()``,
Doctrine silently makes a second query to find the ``Category`` that's related
to this ``Product``. It prepares the ``$category`` object and returns it to
you.

.. image:: /images/book/doctrine_image_3.png
   :align: center

What's important is the fact that you have easy access to the product's related
category, but the category data isn't actually retrieved until you ask for
the category (i.e. it's "lazily loaded").

You can also query in the other direction::

    public function showProductAction($id)
    {
        $category = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Category')
            ->find($id);

        $products = $category->getProducts();
    
        // ...
    }

In this case, the same things occurs: you first query out for a single ``Category``
object, and then Doctrine makes a second query to retrieve the related ``Product``
objects, but only once/if you ask for them (i.e. when you call ``->getProducts()``).
The ``$products`` variable is an array of all ``Product`` objects that relate
to the given ``Category`` object via their ``category_id`` value.

.. sidebar:: Relationships and Proxy Classes

    This "lazy loading" is possible because, when necessary, Doctrine returns
    a "proxy" object in place of the true object. Look again at the above
    example::
    
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        $category = $product->getCategory();

        prints "Proxies\AcmeStoreBundleEntityCategoryProxy"
        echo get_class($category);

    This proxy object extends the true ``Category`` object, and looks and
    acts exactly like it. The difference is that, by using a proxy object,
    Doctrine can delay querying for the real ``Category`` data until you
    actually need that data (e.g. until you call ``$category->getName()``).

    The proxy classes are generated by Doctrine and stored in the cache directory.
    And though you'll probably never even notice that your ``$category``
    object is actually a proxy object, it's important to keep in mind.

    In the next section, when you retrieve the product and category data
    all at once (via a *join*), Doctrine will return the *true* ``Category``
    object, since nothing needs to be lazily loaded.

Joining to Related Records
~~~~~~~~~~~~~~~~~~~~~~~~~~

In the above examples, two queries were made - one for the original object
(e.g. a ``Category``) and one for the related object(s) (e.g. the ``Product``
objects).

.. tip::

    Remember that you can see all of the queries made during a request via
    the web debug toolbar.

Of course, if you know up front that you'll need to access both objects, you
can avoid the second query by issuing a join in the original query. Add the
following method to the ``ProductRepository`` class::

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

Now, you can use this method in your controller to query for a ``Product``
object and its related ``Category`` with just one query::

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->findOneByIdJoinedToCategory($id);

        $category = $product->getCategory();
    
        // ...
    }    

More Information on Associations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section has been an introduction to one common type of entity relationship,
the one-to-many relationship. For more advanced details and examples of how
to use other types of relations (e.g. ``one-to-one``, ``many-to-many``), see
Doctrine's `Association Mapping Documentation`_.

.. note::

    If you're using annotations, you'll need to prepend all annotations with
    ``ORM\`` (e.g. ``ORM\OneToMany``), which is not reflected in Doctrine's
    documentation. You'll also need to include the ``use Doctrine\ORM\Mapping as ORM;``
    statement, which *imports* the ``ORM`` annotations prefix.

Configuration
-------------

Doctrine is highly configurable, though you probably won't ever need to worry
about most of its options. To find out more about configuring Doctrine, see
the Doctrine section of the :doc:`reference manual</reference/configuration/doctrine>`.

Lifecycle Callbacks
-------------------

Sometimes, you need to perform an action right before or after an entity
is inserted, updated, or deleted. These types of actions are known as "lifecycle"
callbacks, as they're callback methods that you need to execute during different
stages of the lifecycle of an entity (e.g. the entity is inserted, updated,
deleted, etc).

If you're using annotations for your metadata, start by enabling the lifecycle
callbacks. This is not necessary if you're using YAML or XML for your mapping:

.. code-block:: php-annotations

    /**
     * @ORM\Entity()
     * @ORM\HasLifecycleCallbacks()
     */
    class Product
    {
        // ...
    }

Now, you can tell Doctrine to execute a method on any of the available lifecycle
events. For example, suppose you want to set a ``created`` date column to
the current date, only when the entity is first persisted (i.e. inserted):

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

    The above example assumes that you've created and mapped a ``created``
    property (not shown here).

Now, right before the entity is first persisted, Doctrine will automatically
call this method and the ``created`` field will be set to the current date.

This can be repeated for any of the other lifecycle events, which include:

* ``preRemove``
* ``postRemove``
* ``prePersist``
* ``postPersist``
* ``preUpdate``
* ``postUpdate``
* ``postLoad``
* ``loadClassMetadata``

For more information on what these lifecycle events mean and lifecycle callbacks
in general, see Doctrine's `Lifecycle Events documentation`_

.. sidebar:: Lifecycle Callbacks and Event Listeners

    Notice that the ``setCreatedValue()`` method receives no arguments. This
    is always the case for lifecylce callbacks and is intentional: lifecycle
    callbacks should be simple methods that are concerned with internally
    transforming data in the entity (e.g. setting a created/updated field,
    generating a slug value).
    
    If you need to do some heavier lifting - like perform logging or send
    an email - you should register an external class as an event listener
    or subscriber and give it access to whatever resources you need. For
    more information, see :doc:`/cookbook/doctrine/event_listeners_subscribers`.

Doctrine Extensions: Timestampable, Sluggable, etc.
---------------------------------------------------

Doctrine is quite flexible, and a number of third-party extensions are available
that allow you to easily perform repeated and common tasks on your entities.
These include thing such as *Sluggable*, *Timestampable*, *Loggable*, *Translatable*,
and *Tree*.

For more information on how to find and use these extensions, see the cookbook
article about :doc:`using common Doctrine extensions</cookbook/doctrine/common_extensions>`.

.. _book-doctrine-field-types:

Doctrine Field Types Reference
------------------------------

Doctrine comes with a large number of field types available. Each of these
maps a PHP data type to a specific column type in whatever database you're
using. The following types are supported in Doctrine:

* **Strings**

  * ``string`` (used for shorter strings)
  * ``text`` (used for larger strings)

* **Numbers**

  * ``integer``
  * ``smallint``
  * ``bigint``
  * ``decimal``
  * ``float``

* **Dates and Times** (use a `DateTime`_ object for these fields in PHP)

  * ``date``
  * ``time``
  * ``datetime``

* **Other Types**

  * ``boolean``
  * ``object`` (serialized and stored in a ``CLOB`` field)
  * ``array`` (serialized and stored in a ``CLOB`` field)

For more information, see Doctrine's `Mapping Types documentation`_.

Field Options
~~~~~~~~~~~~~

Each field can have a set of options applied to it. The available options
include ``type`` (defaults to ``string``), ``name``, ``length``, ``unique``
and ``nullable``. Take a few annotations examples:

.. code-block:: php-annotations

    /**
     * A string field with length 255 that cannot be null
     * (reflecting the default values for the "type", "length" and *nullable* options)
     * 
     * @ORM\Column()
     */
    protected $name;

    /**
     * A string field of length 150 that persists to an "email_address" column
     * and has a unique index.
     *
     * @ORM\Column(name="email_address", unique="true", length="150")
     */
    protected $email;

.. note::

    There are a few more options not listed here. For more details, see
    Doctrine's `Property Mapping documentation`_

.. index::
   single: Doctrine; ORM Console Commands
   single: CLI; Doctrine ORM

Console Commands
----------------

The Doctrine2 ORM integration offers several console commands under the
``doctrine`` namespace. To view the command list you can run the console
without any arguments:

.. code-block:: bash

    php app/console

A list of available command will print out, many of which start with the
``doctrine:`` prefix. You can find out more information about any of these
commands (or any Symfony command) by running the ``help`` command. For example,
to get details about the ``doctrine:database:create`` task, run:

.. code-block:: bash

    php app/console help doctrine:database:create

Some notable or interesting tasks include:

* ``doctrine:ensure-production-settings`` - checks to see if the current
  environment is configured efficiently for production. This should always
  be run in the ``prod`` environment:
  
  .. code-block:: bash
  
    php app/console doctrine:ensure-production-settings --env=prod

* ``doctrine:mapping:import`` - allows Doctrine to introspect an existing
  database and create mapping information. For more information, see
  :doc:`/cookbook/doctrine/reverse_engineering`.

* ``doctrine:mapping:info`` - tells you all of the entities that Doctrine
  is aware of and whether or not there are any basic errors with the mapping.

* ``doctrine:query:dql`` and ``doctrine:query:sql`` - allow you to execute
  DQL or SQL queries directly from the command line.

.. note::

   To be able to load data fixtures to your database, you will need to have the
   ``DoctrineFixturesBundle`` bundle installed. To learn how to do it,
   read the ":doc:`/cookbook/doctrine/doctrine_fixtures`" entry of the Cookbook.

Summary
-------

With Doctrine, you can focus on your objects and how they're useful in your
application and worry about database persistence second. This is because
Doctrine allows you to use any PHP object to hold your data and relies on
mapping metadata information to map an object's data to a particular database
table.

And even though Doctrine revolves around a simple concept, it's incredibly
powerful, allowing you to create complex queries and subscribe to events
that allow you to take different actions as objects go through their persistence
lifecycle.

For more information about Doctrine, see the *Doctrine* section of the
:doc:`cookbook</cookbook/index>`, which includes the following articles:

* :doc:`/cookbook/doctrine/doctrine_fixtures`
* :doc:`/cookbook/doctrine/migrations`
* :doc:`/cookbook/doctrine/mongodb`
* :doc:`/cookbook/doctrine/common_extensions`

.. _`Doctrine`: http://www.doctrine-project.org/
.. _`MongoDB`: http://www.mongodb.org/
.. _`Basic Mapping Documentation`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html
.. _`Query Builder`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/query-builder.html
.. _`Doctrine Query Language`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/dql-doctrine-query-language.html
.. _`Association Mapping Documentation`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/association-mapping.html
.. _`DateTime`: http://php.net/manual/en/class.datetime.php
.. _`Mapping Types Documentation`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#doctrine-mapping-types
.. _`Property Mapping documentation`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/basic-mapping.html#property-mapping
.. _`Lifecycle Events documentation`: http://www.doctrine-project.org/docs/orm/2.0/en/reference/events.html#lifecycle-events
