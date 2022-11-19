==============
Setup Executor
==============

.. function:: _setup_executor(self, **kwargs)

    TODO: Add functions description

    :param kwargs: A dictionary containing the following keys - ``g`` (graph info), ``n_feats`` (node features)
                    and ``e_feats`` (edge features)


    .. code-block:: python
        :linenos:
        :caption: Source code for _setup_executor()

        def _setup_executor(self, **kwargs):
            graph = kwargs.get('g', None)
            node_feats = kwargs.get('n_feats', {})
            edge_feats = kwargs.get('e_feats', {})

            if not graph:
                raise NameError('Need to provide the graph as one of keyward arguments')

            graph_info, need_reset = self._update_graph_info(graph)

            if self._entry_count == 0:
                fprog = Program()
                ret = self._trace(node_feats, edge_feats, self._input_cache, fprog)
                print('TracedProgram' + str(fprog), 'Ret value:', ret)
                self._executor_cache = self._diff_then_compile(ret, fprog, graph_info)

            for k, v in node_feats.items():
                self._input_cache[var_prefix + k + cen_attr_postfix] = v
                self._input_cache[var_prefix + k + inb_attr_postfix] = v

            for k, v in edge_feats.items():
                self._input_cache[var_prefix+k] = v

            self._executor_cache.restart(self._input_cache, graph_info if need_reset else None)
            self._entry_count += 1

            return self._executor_cache

Function Overview
-----------------

    .. code-block:: python
        :lineno-start: 2

        graph = kwargs.get('g', None)
        node_feats = kwargs.get('n_feats', {})
        edge_feats = kwargs.get('e_feats', {})

    Here ``graph`` is a DGL Graph object. ``node_feats`` and ``edge_feats`` are dictionaries with the following
    key-value pairs 

    .. code-block:: python

        node_feats = {  'norm': tensor, 'h': tensor }
        edge_feats = {}

        # For cora dataset, num_nodes = 2708 and num_features = 1433
        node_feats['norm'].size() = torch.Size([2708, 1])
        node_feats['h'].size() = torch.Size([2708, 1433])

    .. code-block:: python
        :lineno-start: 9
        :caption: todo explanation

        graph_info, need_reset = self._update_graph_info(graph)

    .. code-block:: python
        :lineno-start: 11
        :caption: todo explanation

        if self._entry_count == 0:
            fprog = Program()
            ret = self._trace(node_feats, edge_feats, self._input_cache, fprog)
            print('TracedProgram' + str(fprog), 'Ret value:', ret)
            self._executor_cache = self._diff_then_compile(ret, fprog, graph_info)

    .. code-block:: python
        :lineno-start: 17
        :caption: todo explanation

        for k, v in node_feats.items():
            self._input_cache[var_prefix + k + cen_attr_postfix] = v
            self._input_cache[var_prefix + k + inb_attr_postfix] = v

    .. code-block:: python
        :lineno-start: 21
        :caption: todo explanation

        for k, v in edge_feats.items():
            self._input_cache[var_prefix+k] = v

    .. code-block:: python
        :lineno-start: 24
        :caption: todo explanation

        self._executor_cache.restart(self._input_cache, graph_info if need_reset else None)
        self._entry_count += 1

        return self._executor_cache