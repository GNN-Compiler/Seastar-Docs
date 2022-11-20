=====
Trace
=====

**Add function desciption**

.. function:: _trace(self, nfeats, efeats, input_cache, fprog)

    :param nfeats: A dictionary containing node features *norm* and *h*
    :param efeats: A dictionary containing edge features
    :param input_cache: A dictionary
    :param fprog: *None = head(None)*
    :rtype: 

    .. code-block:: python
        :linenos:
        :caption: source code for _trace()

        def _trace(self, nfeats, efeats, input_cache, fprog):
            backend = self.find_backend(self._nspace)
            central_node = self._init_central_node(nfeats, efeats, fprog, backend)
            old_libs = defaultdict(dict)

            self._monkey_patch_namespace(old_libs, input_cache, fprog, backend)
            ret = self._f(central_node)
            self._remove_patch(old_libs, backend)

            if ret == None:
                raise NameError('Ret is none. Execution is aborted')

            return [ret.var] if not isinstance(ret, Iterable) else ret.var

Function Overview
-----------------

    .. code-block:: python
        :lineno-start: 2

        backend = self.find_backend(self._nspace)
        central_node = self._init_central_node(nfeats, efeats, fprog, backend)
        old_libs = defaultdict(dict)

    ``self.find_backend`` returns a tuple containing information about the backend being used. The
    tuple is of the form *(backend-name, backend-module-instance)*. 

    >>> (pdb) p backend
    ('torch', <module 'torch' from '/home/nithin/anaconda3/envs/seastar-new/lib/python3.10/site-packages/torch/__init__.py'>)

    ``central_node`` is a ``CentralNode`` object returned by ``_init_central_node()``.

    >>> (pdb) p central_node
    <seastar.node.CentralNode object at 0x7fa5cb667340>

    ``old_libs`` is a defaultdict with the default_factory argument as a dictionary. 
    To learn more about defaultdict, refer `this <https://www.geeksforgeeks.org/defaultdict-in-python/>`_

    >>> (pdb) p old_libs
    defaultdict(<class 'dict'>, {})

    .. code-block:: python
        :lineno-start: 6
        :caption: todo explanation for line 6 and 8

        self._monkey_patch_namespace(old_libs, input_cache, fprog, backend)
        ret = self._f(central_node)
        self._remove_patch(old_libs, backend)


    ``self._f`` makes a call to the function ``nb_compute()`` with central_node passed as an argument.
    The returned value is stored in ``ret``

    >>> (pdb) p ret
    V2(ValType.D,[1433],grad:None)

    .. code-block:: python
        :lineno-start: 10

        if ret == None:
            raise NameError('Ret is none. Execution is aborted')

        return [ret.var] if not isinstance(ret, Iterable) else ret.var

    Incase ``ret`` has the value ``None``, an error is raised and execution is aborted.
    The return statement just makes sure that the return value is an iterable.

    >>> Return value from Context._trace: [V2(ValType.D,[1433],grad:None)]