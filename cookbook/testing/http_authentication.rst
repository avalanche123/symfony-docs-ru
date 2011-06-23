.. index::
   single: Тесты; HTTP Аутентификация

Как смоделировать HTTP аутентификацию в Функциональном тесте
============================================================

Если вашему приложению необходима HTTP аутентификация, передайте имя
пользователя и пароль в качестве переменных сервера в метод ``createClient()``::

    $client = $this->createClient(array(), array(
        'PHP_AUTH_USER' => 'username',
        'PHP_AUTH_PW'   => 'pa$$word',
    ));

Также можно делать переопределения в каждом запросе::

    $client->request('DELETE', '/post/12', array(), array(
        'PHP_AUTH_USER' => 'username',
        'PHP_AUTH_PW'   => 'pa$$word',
    ));

