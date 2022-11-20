=================
Update Graph Info
=================

Updates the attribute ``_graph_info_cache`` with the ``graph`` passed as argument.
Here ``_graph_info_cache`` is the namedtuple ``GraphInfo`` declared inside ``class Context``.
It only updates it if ``_graph_info_cache`` is ``None`` or if the graph properties like number of
edges or number of nodes changes. A boolean flag ``reset`` is also returned, indicating if 
``_graph_info_cache`` was updated by this function.

.. function:: _update_graph_info(self, graph)

    :param graph: A DGL graph object
    :rtype: GraphInfo, bool

    .. code-block:: python
        :linenos:
        :caption: source code for _update_graph_info()

        def _update_graph_info(self, graph):
            reset = False

            if not (self._graph_info_cache != None
                    and self._graph_info_cache.number_of_nodes == graph.number_of_nodes()
                    and self._graph_info_cache.number_of_edges == graph.number_of_edges()):

                in_csr = graph.get_in_csr()
                out_csr = graph.get_out_csr()

                self._graph_info_cache = Context.GraphInfo(
                    graph.number_of_nodes(),
                    graph.number_of_edges(),
                    in_csr(0).copy_to_gpu(0),
                    in_csr(1).copy_to_gpu(0),
                    in_csr(2).copy_to_gpu(0),
                    out_csr(0).copy_to_gpu(0),
                    out_csr(1).copy_to_gpu(0),
                    out_csr(2).copy_to_gpu(0),
                    graph.nbits())

                reset = True

            return self._graph_info_cache, reset

Function Overview
-----------------

    .. code-block:: python
        :lineno-start: 2

        reset = False

    If ``reset`` is set to false, then the _graph_info_cache is not updated. Else it will be updated.

    .. code-block:: python
        :lineno-start: 2

        if not (self._graph_info_cache != None
                    and self._graph_info_cache.number_of_nodes == graph.number_of_nodes()
                    and self._graph_info_cache.number_of_edges == graph.number_of_edges()):

    The if-block is executed if **atleast one** of the following conditions are True:

    1. _graph_info_cache is None
    2. _graph_info_cache.number_of_nodes != graph.number_of_nodes()
    3. _graph_info_cache.number_of_edges != graph.number_of_edges()

    .. code-block:: python
        :lineno-start: 8
        :caption: todo explanation

        in_csr = graph.get_in_csr()
        out_csr = graph.get_out_csr()

    .. code-block:: python
        :lineno-start: 11

        self._graph_info_cache = Context.GraphInfo(
                graph.number_of_nodes(),
                graph.number_of_edges(),
                in_csr(0).copy_to_gpu(0),
                in_csr(1).copy_to_gpu(0),
                in_csr(2).copy_to_gpu(0),
                out_csr(0).copy_to_gpu(0),
                out_csr(1).copy_to_gpu(0),
                out_csr(2).copy_to_gpu(0),
                graph.nbits())

            reset = True


    ``nbits`` is the number of integer bits used in the storage (32 or 64). _graph_info_cache is now the namedtuple ``GraphInfo``.
    ``reset`` is now set to True, because _graph_info_cache was changed.

            