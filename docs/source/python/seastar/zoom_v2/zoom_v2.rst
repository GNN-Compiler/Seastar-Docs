=======
zoom_v2
=======

Todo: Give a brief overview of what zoom_v2 does

Dependencies
============

Imported Libraries
------------------

``functools``
    for functools.update_wrapper() in __init__ of class ``Context``

``collections``
    for defaultdict, Iterable in ``_trace()`` and namedtuple in class ``Context``

Modules
-------

``node`` 
``val``
``op``
``program``
``passes``
``schema``
``autodiff``
``code_gen``
``executor``
``utils``

Used In
-------

``__init__``

Classes
=======

.. toctree::
    :maxdepth: 1

    Context/Context
    CtxManager/CtxManager