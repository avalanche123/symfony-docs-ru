.. index::
   pair: Doctrine; DBAL

DBAL
====

Слой абстракции базы данных (DBAL) `Doctrine`_ это слой абстракции,
располагающийся поверх `PDO`_ и предоставляющий инуитивный и гибкий API для
взаимодействия с большинством популярных реляционных баз данных, которые
существуют сегодня!

.. tip::

    Узнать больше о Doctrine DBAL можно в официальной `documentation`_
    на web сайте.

Чтобы начать работу нужно лишь включить и настроить DBAL:

.. code-block:: yaml

    # app/config/config.yml

    doctrine.dbal:
        driver:   PDOMySql
        dbname:   Symfony2
        user:     root
        password: null

Затем вы можете получить Doctrine DBAL соединение, через службу
``database_connection``::

    class UserController extends Controller
    {
        public function indexAction()
        {
            $conn = $this->get('database_connection');

            $users = $conn->fetchAll('SELECT * FROM users');
        }
    }

.. _PDO:           http://www.php.net/pdo
.. _documentation: http://www.doctrine-project.org/projects/dbal/2.0/docs/en
.. _Doctrine:      http://www.doctrine-project.org
