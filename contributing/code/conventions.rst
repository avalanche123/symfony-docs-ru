Соглашения
===========

Документ :doc:`стандарты <standards>` описывает стандарты кодирования для проекта
Symfony2, его внутренних и сторонних бандлов. Этот документ описывает стандарты
кодирования и соглашения, используемые в ядре фреймворка чтобы сделать его
более последовательным и предсказуемым. Можете следовать им в своём коде, но в
этом нет особой нужды.

Имена методов
------------

Когда объект имеет "главную" множественную связь со связанными "предметами"
(объекты, параметры и т. д.), имена методов стандартизуются:

  * ``get()``
  * ``set()``
  * ``has()``
  * ``all()``
  * ``replace()``
  * ``remove()``
  * ``clear()``
  * ``isEmpty()``
  * ``add()``
  * ``register()``
  * ``count()``
  * ``keys()``

Использование этих методов применимо только когда ясно что существует основная
связь:

* ``CookieJar`` имеет множество ``Cookie``;

* Служба ``Container`` имеет множество служб и множество параметров (т. к.
  службы это главная связь, то мы используем соглашения для имён для этой связи);

* Консоль ``Input`` имеет множество аргументов и множество опций. Здесь нет
  "основной" связи, поэтому соглашения для имён не применяются.

Для множественных связей, к которым не применяется соглашение, должны
использоваться следующие методы (где ``XXX`` это имя соотвествующего предмета):

============== =================
Главная связь  Другие связи
============== =================
``get()``      ``getXXX()``
``set()``      ``setXXX()``
``has()``      ``hasXXX()``
``all()``      ``getXXXs()``
``replace()``  ``setXXXs()``
``remove()``   ``removeXXX()``
``clear()``    ``clearXXX()``
``isEmpty()``  ``isEmptyXXX()``
``add()``      ``addXXX()``
``register()`` ``registerXXX()``
``count()``    ``countXXX()``
``keys()``     не доступно
============== =================
