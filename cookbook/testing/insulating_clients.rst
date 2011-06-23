.. index::
   single: Тесты

Как тестировать взаимодействие с несколькими клиентами
======================================================

Если требуется смоделировать взаимодействие между разными Клиентами
(представьте, например, чат), то создайте несколько Клиентов::

    $harry = static::createClient();
    $sally = static::createClient();

    $harry->request('POST', '/say/sally/Hello');
    $sally->request('GET', '/messages');

    $this->assertEquals(201, $harry->getResponse()->getStatusCode());
    $this->assertRegExp('/Hello/', $sally->getResponse()->getContent());

Этот подход работает, за исключением тех случаев, когда ваш код обрабатывает
глобальное состояния или зависит от библиотек третьих лиц, которые также его
используют. Для подобных случаев можно изолировать клиентов::

    $harry = static::createClient();
    $sally = static::createClient();

    $harry->insulate();
    $sally->insulate();

    $harry->request('POST', '/say/sally/Hello');
    $sally->request('GET', '/messages');

    $this->assertEquals(201, $harry->getResponse()->getStatusCode());
    $this->assertRegExp('/Hello/', $sally->getResponse()->getContent());

Изолированные клиенты выполняют свои запросы в отдельных чистых PHP процессах, 
что исключает любые побочные эффекты.

.. tip::

    Так как изолированный клиент работает медленнее, то можно одного клиента 
    оставить выполняться в главном процессе, а остальных изолировать.    
