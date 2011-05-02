.. index::
   single: Тесты

Как тестировать взаимодействие с несколькими клиентами
======================================================

Если требуется моделирование ситуации в которой происходит
взаимодействие между разными Клиентами (представьте, например, чат),
то выходом в данной ситуации будет создание нескольких Клиентов::

    $harry = $this->createClient();
    $sally = $this->createClient();

    $harry->request('POST', '/say/sally/Hello');
    $sally->request('GET', '/messages');

    $this->assertEquals(201, $harry->getResponse()->getStatusCode());
    $this->assertRegExp('/Hello/', $sally->getResponse()->getContent());

Данный подход будет работать, только в тех случаях когда ваш код не хранит
глобального состояния или не зависит от библиотек третьих лиц, которые его используют. 
В подобных случаях, можно изолировать клиентов::

    $harry = $this->createClient();
    $sally = $this->createClient();

    $harry->insulate();
    $sally->insulate();

    $harry->request('POST', '/say/sally/Hello');
    $sally->request('GET', '/messages');

    $this->assertEquals(201, $harry->getResponse()->getStatusCode());
    $this->assertRegExp('/Hello/', $sally->getResponse()->getContent());

Изолированные клиенты, выполняют свои запросы в отдельных PHP процессах, 
что исключает любые побочные эффекты.

.. tip::
    Так как изолированный клиент работает медленнее, то можно одного клиента 
    оставить выполняться в главном процессе, и изолировать остальных.    
