New Getting Started
===================

IgnisHPC is a modularized docker framework consisting of multiple source code repositories. The framework is open source, so all repositories can be found in GitHub: `IgnisHPC <https://github.com/ignishpc>`_.

Next we summarize the minimum steps to execute a simple job in IgnisHPC.

------------
Requirements
------------

As we mentioned, IgnisHPC is a dockerized framework, so all the system modules are executed inside docker containers. On the other hand, IgnisHPC external dependencies such as schedulers or image storage can be installed independently, but IgnisHPC includes a dockerized version of them.

Therefore, the minimum requirements to run Ignis are:

  1. *Docker*: It must be installed and accessible. We recommend using the newest version available.
  2. *Python3*: Available by default in most Linux distributions. It is used to execute a deploy script and simplify the installation of IgnisHPC and its dependencies.
  3. *Pip*: The deploy script is available as a pip package, although it can be downloaded from the source code repository.
  4. *Git* (optional): The git binary is required for building IgnisHPC images from repositories.
  5. Poetry: Poetry is a python package manager that is used to manage the dependencies of the IgnisHPC project. It is used for install all needed dependencies for the project. 


------------
Installation
------------

First you need to clone the IgnisHPC client repo from https://github.com/ignishpc/client \
then you need to enter to the directory **ignishpc**

IgnisHPC client can be installed just using the following command::

  $ poetry install 

Once is installed you can try the installation executing the command::

  $ poetry run python main.py -h

And you should see a help prompt with the command you can use in the ignis client.

----------------------------
Creating IgnisHPC Containers
----------------------------

Images of IgnisHPC containers are not available for download. They must be built in your runtime environment. This allows \
the creation of a custom development environment with complete isolation from other framework installations even on the same machines.

Then you can build the images you need executing the command::

 $  poetry run python main.py images build --sources \
    https://github.com/ignishpc/core-base \
    https://github.com/ignishpc/core-python \
    https://github.com/ignishpc/core-cpp \
    https://github.com/ignishpc/core-go

The first repository is essential for the construction of the base images. The command can be executed \
in several phases, it is not necessary to specify all the repositories in the same execution. This command \ 
allows users to create an image that can run Python, C++ and Go codes in the same container.

-----------------------------
Deploying IgnisHPC Containers
-----------------------------

Once all images are created, it is necessary to deploy the containers. IgnisHPC jobs are launched using the submitter module, \
but to use a cluster it requires a resource and scheduler manager such as Nomad or Mesos. 

Alternatively, IgnisHPC can also be launched as a slurm job in an HPC cluster, docker is replaced by singularity, so a different \
submitter must be used.

Next we show examples deploying the containers locally (no manager is needed), and using Nomad, Mesos and Slurm. 

Docker (Only local)
^^^^^^^^^^^^^^^^^^^^
Submitter using an http endpoint::

 $ ignis-deploy submitter start --dfs <working-directory-path> --scheduler docker tcp://myhost:2375

Submitter using a Unix-socket::

 $ ignis-deploy submitter start --dfs <working-directory-path> --scheduler docker /var/run/docker.sock \
   --mount /var/run/docker.sock /var/run/docker.sock


Nomad
^^^^^

Master node::

 $ ignis-deploy nomad start --password 1234 --volumes <working-directory-path\*>

Worker nodes::

 $ ignis-deploy nomad start --password 1234 --join myhost1 --default-registry myhost1:5000

Submitter::

 $ ignis-deploy submitter start --dfs <working-directory-path\*> \
    --scheduler nomad http://myhostX:4646


\* The working directory must be available on all nodes via NFS (Network File System) or a DFS (Distributed File System). (Only required for working with files)

Mesos
^^^^^

Zookeeper is requiered by Mesos::

 $ ignis-deploy zookeeper start --password 1234

Master node::

 $ ignis-deploy mesos start -q 1 --name master -zk  zk://master:2281 \
    --service [marathon | singularity] --port-service 8888

Worker nodes::

 $ ignis-deploy mesos start --name nodoX -zk  zk://master:2281 \
    --port-service 8888 --default-registry master:5000

Submitter::

 $ ignis-deploy submitter start --dfs <working-directory-path*> \
    --scheduler [marathon | singularity] http://master:8888


\* The working directory must be available on all nodes via NFS (Network File System) or a DFS (Distributed File System). (Only required for working with files)

Slurm
^^^^^

The ``ignis-slurm`` submitter can be obtained from ``ignishpc/slurm-submitter`` with::

 $ docker run --rm -v $(pwd):/target ignishpc/slurm-submitter ignis-export /target

This submitter will allow you to launch ignisHPC on a cluster as a non-root user and without docker.

IgnisHPC Docker images can be converted to singulairty image files with::

 $ ignis-deploy images singularity [--host] ignishpc/full ignis_full.sif

The basic syntax of ``ignis-slurm`` is the same as the later shown ``ignis-submit``, but a first parameter with job-time must be passed to be requested to slurm. The time can be specified in any format supported by slurm. 
For example, a 10 minute job should start with::

  $ ignis-slurm 00:10:00 ....

In addition, help text can be displayed using::

  $ ignis-slurm --help

-----------------------
Launching the first job
-----------------------

The first step to launch a job is to connect to the submiter container. The default password is ``ignis``, but we can change it inside the container or choose one when launching the submitter.::

 $  ssh root@myhost -p 2222

The code we will use as an example is the classic Wordcount application, which can be seen below.

.. code-block:: python

    #!/usr/bin/python

    import ignis

    # Initialization of the framework
    ignis.Ignis.start()
    # Resources/Configuration of the cluster
    prop = ignis.IProperties()
    prop["ignis.executor.image"] = "ignishpc/python"
    prop["ignis.executor.instances"] = "1"
    prop["ignis.executor.cores"] = "2"
    prop["ignis.executor.memory"] = "1GB"
    # Construction of the cluster
    cluster = ignis.ICluster(prop)

    # Initialization of a Python Worker in the cluster
    worker = ignis.IWorker(cluster, "python")
    # Task 1 - Tokenize text into pairs ('word', 1)
    text =  worker.textFile("text.txt")
    words = text.flatmap(lambda line: [(word, 1) for word in line.split()])
    # Task 2 - Reduce pairs with same word and obtain totals
    count = words.toPair().reduceByKey(lambda a, b: a + b)
    # Print results to file
    count.saveAsTextFile("wordcount.txt")

    # Stop the framework
    ignis.Ignis.stop()


In order to run it, we need to create a file containing a text sample (``text.txt``) and store it in the working
directory. By default the submitter sets the working directory to ``/media/dfs``. All relative paths used in the
source code are resolved using this working directory, so ``/media/dfs/text.txt`` is an alias of ``text.txt``.

Finally, we can execute our code using the submitter::

 $ ignis-submit ignishpc/python python3 driver.py

or::

  $ ignis-submit ignishpc/python ./driver.py


When the execution has finished, we can see the result of the execution in ``wordcount.txt`` located in the working directory. If we want to check the execution logs, we must navigate to the scheduler web or use ``docker log`` in case of using docker directly.


Launching without Container
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``ignis-submit`` can also be used outside the submiter container, for example where permanent containers are not allowed::

$ docker run --rm -v $(pwd):/target ignishpc/submitter ignis-export /target

This command will create a  ``ignis`` folder in the current directory with everything needed to run the submiter. The ``ignis-deploy`` command configures the submitter container, but when there is no container, we must set the configuration manually.
The submitter needs a dfs and a scheduler, as ``ignis-deploy`` showed, these can be defined as environment variables or in ``ignis/etc/ignis.conf`` property file.

.. code-block:: sh

	# set current directory as job directory (ignis.dfs.id in ignis.conf)
	export IGNIS_DFS_ID=$(pwd)
	# set docker as scheduler (ignis.scheduler.type in ignis.conf)
	export IGNIS_SCHEDULER_TYPE=docker
	# set where docker is available (ignis.scheduler.url in ignis.conf)
	export IGNIS_SCHEDULER_URL=/var/run/docker.sock


The above example could be launched as follows::

$ ./ignis/bin/ignis-submit ignishpc/python ./driver.py