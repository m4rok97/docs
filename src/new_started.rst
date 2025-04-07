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

-----------------------
Launching the first job
-----------------------



To submit a job, you need to have code ready to run. In this example, we will use the classic WordCount application, shown below.

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

 $ poetry run python main.py run driver.py

or::

  $ poetry run python main.py job run driver.py 


When the execution is finished, you can see the result in wordcount.txt, located in the working directory. \
To check the execution logs, go to the job directory identified in the submission message, \
for example ``driverpy-0sAdh`` and enter to the folder ``logs``.