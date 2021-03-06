# ps_pytorch
implement [parameter server](https://www.cs.cmu.edu/~muli/file/parameter_server_osdi14.pdf) with PyTorch and OpenMPI

# motivations:
1. PyTorch provides easy-to-use APIs with dynamic computational graph
2. PyTorch dose not offer full distributed packages (there is some comm lib but not support operations with same flexiblity as OpenMPI)
3. [mpi4py](https://github.com/mpi4py/mpi4py) is a good Python bindings for any distributions of MPI (e.g. OpenMPI, MPICH, and etc)

# system design:
1. parameter server node: This node serves both as master and parameter server in our system, i.e. it synchronize all workers to enter next iteration by broadcast global step to workers and also store the global model, which are keeping fetched by worker nodes at beginning of one iteration. For a user defined frequency, parameter server node will save the current model as checkpoint to shared file system (NFS in our system) for model evaluation.
2. workers mainly aim at sample data points (or mini-batch) in from local dataset (we don't pass data among nodes to maintain data locality), computing gradients, and send them back to parameter server.
3. evaluator read the checkpoints from the shared directory, and do model evaluation. Note that: there is only testset data saved on evaluator nodes.
