.. index::
   single: Поисковик (Finder)

Как искать файлы
================ 

С помощью компонента :namespace:`Symfony\\Component\\Finder` можно 
легко и быстро находить необходимые файлы и каталоги.

Использование
-------------

Класс :class:`Symfony\\Component\\Finder\\Finder` производит поиск 
файлов и/или каталогов::

    use Symfony\Component\Finder\Finder;

    $finder = new Finder();
    $finder->files()->in(__DIR__);

    foreach ($finder as $file) {
        print $file->getRealpath()."\n";
    }

Объект ``$file`` является экземпляром класса :phpclass:`SplFileInfo`.
Код выше печатает имена всех файлов в текущем каталоге рекурсивно. Класс Finder
использует свободный интерфейс, так что все методы возвращают тип данных Finder.

.. tip::
    Экземпляр класса Finder является `Iterator`_ (итератором) PHP. Так, что вместо
    прохода над Finder'ом с помощью ``foreach``, можно также конвертировать его
    в массив с помощью метода :phpfunction:`iterator_to_array`, или получить 
    количество элементов, с помощью :phpfunction:`iterator_count`.
   
Критерии поиска
---------------
Расположение
~~~~~~~~~~~~

Расположение является единственным обязательным параметром. Данный параметр
указывает поисковику какую директорию использовать для поиска::

    $finder->in(__DIR__);

Поиск в нескольких местах реализуется с помощью последовательных вызовов
метода :method:`Symfony\\Component\\Finder\\Finder::in`::

    $finder->files()->in(__DIR__)->in('/elsewhere');

Исключение каталогов из поиска осуществляется методом
:method:`Symfony\\Component\\Finder\\Finder::exclude` ::

    $finder->in(__DIR__)->exclude('ruby');

Т.к. Finder использует PHP итераторы, ему можно передать любой 
URL с поддерживаемым протоколом `protocol`_::

    $finder->in('ftp://example.com/pub/');

Также он работает с пользовательскими потоками::

    use Symfony\Component\Finder\Finder;

    $s3 = new \Zend_Service_Amazon_S3($key, $secret);
    $s3->registerStreamWrapper("s3");

    $finder = new Finder();
    $finder->name('photos*')->size('< 100K')->date('since 1 hour ago');
    foreach ($finder->in('s3://bucket-name') as $file) {
        // do something

        print $file->getFilename()."\n";
    }

.. note::
    В документации `Streams`_ можно узнать как создавать свои собственные потоки.

Файлы или каталоги
~~~~~~~~~~~~~~~~~~

По-умолчанию, Finder возвращает файлы или каталоги; но
методами  :method:`Symfony\\Component\\Finder\\Finder::files`
и :method:`Symfony\\Component\\Finder\\Finder::directories`
можно управлять его поведением::

    $finder->files();

    $finder->directories();

Если хотите следовать по ссылкам, используйте метод ``followLinks()``::

    $finder->files()->followLinks();

По-умолчанию, итератор игнорирует популярные файлы VCS. Данное поведение может быть изменено
с помощью метода ``ignoreVCS()``:: 

    $finder->ignoreVCS(false);

Сортировка
~~~~~~~~~~

Сортировка результатов по имени или типу (каталоги первыми, файлы последними)::

    $finder->sortByName();

    $finder->sortByType();

.. note::
    Обратите внимание, что методам ``sort*`` требуется получить все подходящие 
    под обработку объекты. Данная процедура при больших объемах весьма медленна.

Также можно определить свои собственные алгоритмы сортировки с помощью метода ``sort()``::

    $sort = function (\SplFileInfo $a, \SplFileInfo $b)
    {
        return strcmp($a->getRealpath(), $b->getRealpath());
    };

    $finder->sort($sort);

Имена файлов
~~~~~~~~~~~~

Наложить ограничения по имени файлов можно с помощью метода
:method:`Symfony\\Component\\Finder\\Finder::name`::

    $finder->files()->name('*.php');

Метод ``name()`` принимает строки, регулярные выражения или шаблоны::

    $finder->files()->name('/\.php$/');

Метод ``notNames()`` исключает файлы по шаблону::

    $finder->files()->notName('*.rb');

Размер файла
~~~~~~~~~~~~

Ограничить размер файлов можно с помощью метода
Restrict files by size with the :method:`Symfony\\Component\\Finder\\Finder::size`::

:method:`Symfony\\Component\\Finder\\Finder::size` method::

    $finder->files()->size('< 1.5K');
Ограничить размер в рамках можно с помощью связанных вызовов::

Оператор сравнения может быть любым из следующих: ``>``, ``>=``, ``<``, '<=',
'=='.

Целевое значение может использовать приставки (``k``, ``ki``) килобайты, 
(``m``, ``mi``) мегабайты, или (``g``, ``gi``) гигабайты. Те которые используют 
суффиксы ``i`` (киби/миби и т.д.) в названии являются версиями ``2**n`` согласно 
стандарту`IEC standard`_.

Дата файла
~~~~~~~~~~

С помощью метода :method:`Symfony\\Component\\Finder\\Finder::date`
можно наложить ограничения на файлы по дате последнего изменения::

    $finder->date('since yesterday');
Оператор сравнения может быть любым из следующих:``>``, ``>=``, ``<``, '<=',
'=='. Также можно использовать псевдонимы ``since`` или ``after`` для оператора
``>``, и  ``until`` или ``before`` в качестве ``<``.

Целевое значение может быть любой датой поддерживаемой функцией `strtotime`_


Глубина каталогов
~~~~~~~~~~~~~~~~~

По-умолчанию Finder просматривает каталоги рекурсивно. Ограничить глубину 
поиска можно с помощью метода :method:`Symfony\\Component\\Finder\\Finder::depth`::

    $finder->depth('== 0');
    $finder->depth('< 3');

Фильтрация
~~~~~~~~~~

Ограничить результаты поиска согласно собственным параметрам,
можно используя метод :method:`Symfony\\Component\\Finder\\Finder::filter`::

    $filter = function (\SplFileInfo $file)
    {
        if (strlen($file) > 10) {
            return false;
        }
    };

    $finder->files()->filter($filter);

Метод ``filter()`` получает замыкание в качестве аргумента. Для каждого подходящего
файла, замыкание вызывается с аргументом в виде объекта который является экземпляром 
класса :phpclass:`SplFileInfo`.
Файл исключается из множества результатов если замыкание возвращает ``false``.

.. _strtotime:   http://www.php.net/manual/en/datetime.formats.php
.. _Iterator:     http://www.php.net/manual/en/spl.iterators.php
.. _protocol:     http://www.php.net/manual/en/wrappers.php
.. _Streams:      http://www.php.net/streams
.. _IEC standard: http://physics.nist.gov/cuu/Units/binary.html
