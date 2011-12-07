.. index::
   single: Templating; Global variables

Внедрение переменных во все шаблоны (т.н. Глобальные переменные)
==============================================================

Иногда вам может потребоваться, чтобы некоторая переменная была доступна
во всех ваших шаблонах. Этого можно достичь при помощи вашего файла ``app/config/config.yml``:

.. code-block:: yaml

    # app/config/config.yml
    twig:
        # ...
        globals:
            ga_tracking: UA-xxxxx-x

Теперь, переменная ``ga_tracking`` будет доступна во всех шаблонах Twig:

.. code-block:: html+jinja

    <p>Our google tracking code is: {{ ga_tracking }} </p>

Так просто! Вы также можете получить доступ к системным параметрам
":ref:`book-service-container-parameters`", которые позволят вам изолировать
или повторно использовать значение:

.. code-block:: ini

    ; app/config/parameters.yml
    [parameters]
        ga_tracking: UA-xxxxx-x

.. code-block:: yaml

    # app/config/config.yml
    twig:
        globals:
            ga_tracking: %ga_tracking%

The same variable is available exactly as before.

Более сложные глобальные переменные
-----------------------------------

Если глобальная переменная, которую вам надо использовать, более сложная -
к примеру, объект - тогда вы не сможете воспользоваться приведённым выше методом.
Вместо этого вам нужно создать :ref:`Расширение Twig <reference-dic-tags-twig-extension>`
и возвращать глобальную переменную среди значений метода ``getGlobals``.

.. toctree::
    :hidden:

    Translation source: 2011-12-06 5d5cdf2
