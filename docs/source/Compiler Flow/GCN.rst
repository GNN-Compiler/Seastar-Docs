===
GCN
===

Please refer to the modulewise decoupled version of `GCN Seastar <https://github.com/GNN-Compiler/Seastar-modulewise-code>`_.

.. note::
    
    We have used the toy graph example `Koorah <https://github.com/GNN-Compiler/Seastar-modulewise-code/tree/main/dataset/koorah>`_.
    We might switch to some other standard dataset later when learning, but Koorah is simple to understandand and view whats
    going on in each step of the way.

Here is the decoupled version of gcn.py

.. code-block:: python
    :linenos:

    class GCNModel():
        def __init__(self,g):
            self.g = g
            self.cm = CtxManager(None)
        
        def forward(self,h):

            @self.cm.zoomIn(nspace=[self, torch])
            def nb_compute(v):
                h = sum([nb.h*nb.norm for nb in v.innbs])
                h = h * v.norm
                return h

            nb_compute( g = self.g, 
                        n_feats = {'norm': self.g.ndata['norm'], 'h' : h})
            

    if __name__ == '__main__':

        out_file_path = "../output/cuda_code_output.txt"
        f = open(out_file_path, "a")
        f.write("GCN CUDA OUTPUT\n" + "*"*15 + "\n")
        f.close()

        dataset = 'koorah'
        path = '../dataset/' + str(dataset) + '/'

        edges = np.load(path + 'edges.npy')
        features = np.load(path + 'features.npy')
        labels = np.load(path + 'labels.npy')

        print('dataset {}'.format(dataset))
        print('# of edges : {}'.format(edges.shape[0]))
        print('# of nodes : {}'.format(features.shape[0]))
        print('# of features : {}'.format(features.shape[1]))
        print('# of classes : {}'.format(int(max(labels) - min(labels) + 1)))
        
        features = torch.FloatTensor(features)
        labels = torch.LongTensor(labels)

        source_nodes = edges[:,0]  
        destination_nodes = edges[:,1]
        koorah_graph = dgl.graph((source_nodes,destination_nodes),
                                  num_nodes=features.shape[0])

        in_degree_list = koorah_graph.in_degrees().float()

        norm = torch.pow(in_degree_list, -0.5)
        norm[torch.isinf(norm)] = 0
        koorah_graph.ndata['norm'] = norm.unsqueeze(1)

        model = GCNModel(koorah_graph)
        model.forward(features)


Defining the GCN Model
======================

.. code-block:: python
    :lineno-start: 52

    model = GCNModel(koorah_graph)

This creates an instance of the class ``GCNModel``. A DGLGraph object representing the Koorah dataset is passed to the constructor and is set to the ``g`` attribute.
``model`` also has a context manager attribute ``cm`` which is a ``CtxManager`` object.

.. code-block:: python
    :lineno-start: 2

    def __init__(self,g):
        self.g = g
        self.cm = CtxManager(None)

>>> (Pdb) p model.__dict__
{'g': Graph(num_nodes=6, num_edges=10,
      ndata_schemes={'norm': Scheme(shape=(1,), dtype=torch.float32)}
      edata_schemes={}), 'cm': <seastar.zoom_v2.CtxManager object at 0x7f6b7fe6b640>}

``cm`` has ``_ctx_map`` and ``_run_cb`` as class attributes. ``_ctx_map`` is initialised to an empty dictionary
and ``_run_cb_`` is set to None.

.. code-block:: python
    :linenos:

    class CtxManager():
        def __init__(self, run_cb):
            self._ctx_map = {}
            self._run_cb = run_cb

>>> (Pdb) p model.cm.__dict__
{'_ctx_map': {}, '_run_cb': None}

GCN Forward Call
================

.. code-block:: python
    :lineno-start: 53

    model.forward(features)

We call the ``model.forward()`` method. 

.. code-block:: python
    :lineno-start: 6

    def forward(self,h):

        @self.cm.zoomIn(nspace=[self, torch])
        def nb_compute(v):
            h = sum([nb.h*nb.norm for nb in v.innbs])
            h = h * v.norm
            return h

        nb_compute( g = self.g, 
                    n_feats = {'norm': self.g.ndata['norm'], 'h' : h})   

Decorating nb_compute
---------------------

We notice that the nb_compute() function is decorated by ``cm.zoomIn()`` inside forward().

.. code-block:: python
    :linenos:
    :caption: CtxManager.zoomIn() in zoom_v2.py

    def zoomIn(self, nspace, hetero_graph=False):
        def wrapper(func):
            if not func.__name__ in self._ctx_map:
                if not hetero_graph:
                    print('create context:', self._ctx_map)
                    self._ctx_map[func.__name__] = Context(func, nspace, self._run_cb)
                else:
                    raise NotImplementedError('Heterogeneous graph is not supported yet')
            return self._ctx_map[func.__name__]
        return wrapper

nspace = [self, torch] is passed as argument to zoomIn() and wrapper() takes func = ``nb_compute()`` as argument.
wrapper() creates a ``Context`` object is stored in ``_ctx_map`` dictionary with nb_compute as key.
This Context object is then returned from wrapper() and the decorated nb_compute() function now points to this Context object.

Calling nb_compute
------------------

Since nb_compute() is decorated and points to a Context object, when we call nb_compute(), Context.__call__() is called.

.. code-block:: python
    :linenos:
    :caption: Context.__call__() in zoom_v2.py

    def __call__(self, **kwargs):

        self._setup_executor(**kwargs)
        return None

kwargs is a dictionary containing the arguments passed. Since the following call to nb_compute() is made

.. code-block:: python
    :lineno-start: 14

    nb_compute( g = self.g, 
                n_feats = {'norm': self.g.ndata['norm'], 'h' : h})

kwargs will contain the following arguments passed to nb_compute().

>>> (Pdb) p kwargs
{'g': Graph(num_nodes=6, num_edges=10,
      ndata_schemes={'norm': Scheme(shape=(1,), dtype=torch.float32)}
      edata_schemes={}), 'n_feats': {'norm': tensor([[0.0000],
        [1.0000],
        [1.0000],
        [0.5000],
        [0.7071],
        [0.7071]]), 'h': tensor([[1., 2., 4.],
        [5., 1., 3.],
        [3., 2., 2.],
        [1., 2., 3.],
        [1., 2., 4.],
        [4., 1., 3.]])}}

