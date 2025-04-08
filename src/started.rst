Getting Started
===============

IgnisHPC is a modular Docker-based framework consisting of multiple source code repositories. The framework is open source, and all repositories can be found on GitHub: `IgnisHPC <https://github.com/ignishpc>`_.

------------
Requirements
------------

As mentioned, IgnisHPC is a dockerized framework, so all system modules run inside Docker containers. External dependencies, such as schedulers or image storage systems, can be installed independently. However, IgnisHPC includes dockerized versions of these dependencies.

Minimum requirements to run IgnisHPC:

  1. *Docker*: Must be installed and accessible. We recommend using the latest version available.
  2. *Python 3*: Included by default in most Linux distributions. Required to execute a deployment script and simplify the installation of IgnisHPC and its dependencies.
  3. *Git*: Required to build IgnisHPC images from the repositories.
  4. *Poetry*: A Python package manager used to install and manage the dependencies of the IgnisHPC project.

------------
Installation
------------

First, clone the IgnisHPC client repository from https://github.com/ignishpc/client \
Then navigate to the **ignishpc** directory:

.. code-block:: bash

  $ git clone https://github.com/ignishpc/client
  $ cd ignishpc

Install the client with the following command:

.. code-block:: bash

  $ poetry install 

Once installed, you can verify the installation by running:

.. code-block:: bash

  $ poetry run python main.py -h

You should see a help message listing the available commands for the Ignis client.

----------------------------
Creating IgnisHPC Containers
----------------------------

IgnisHPC container images are not available for direct download. They must be built in your local environment. This allows for the creation of a custom development setup that is isolated from other installations, even on the same machine.

To build the necessary images, run:

.. code-block:: bash

  $ poetry run python main.py images build --sources \
    https://github.com/ignishpc/core-base \
    https://github.com/ignishpc/core-python \
    https://github.com/ignishpc/core-cpp \
    https://github.com/ignishpc/core-go

The first repository is essential for constructing the base images. This command can be run in multiple stages; it's not necessary to list all repositories in a single execution.

This command allows users to build container images capable of running Python, C++, and Go code.

-----------------------
Launching the First Job
-----------------------

To submit a job, you need code ready to execute. In this example, we use the classic WordCount application:

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
    text = worker.textFile("text.txt")
    words = text.flatmap(lambda line: [(word, 1) for word in line.split()])
    # Task 2 - Reduce pairs with the same word and obtain totals
    count = words.toPair().reduceByKey(lambda a, b: a + b)
    # Print results to file
    count.saveAsTextFile("wordcount.txt")

    # Stop the framework
    ignis.Ignis.stop()

To run it, create a file named ``text.txt`` containing your input text and place it in the working directory.

By default, the submitter sets the working directory to ``/media/dfs``. All relative paths used in the code are resolved based on this directory, so ``text.txt`` refers to ``/media/dfs/text.txt``.

Execute your code with the following command:

.. code-block:: bash

  $ poetry run python main.py run driver.py

Or:

.. code-block:: bash

  $ poetry run python main.py job run driver.py

After the job finishes, you can find the results in ``wordcount.txt`` located in the working directory.

To check the execution logs, go to the job directory mentioned in the submission message—for example, ``driverpy-0sAdh``—and navigate to the ``logs`` folder.
