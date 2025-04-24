Getting Started
===============

IgnisHPC is a modular Docker-based framework for distributed computing. It consists of multiple source code repositories, all of which are open source and available on GitHub: `IgnisHPC <https://github.com/ignishpc>`_.

------------
Requirements
------------

IgnisHPC relies on containerization for execution. Users only need the following installed on their system:

  1. *Docker* or *Singularity*: Used to run all system modules in isolated containers.
  2. *Python 3*: Required to run the IgnisHPC client.
  3. *Pip*: For installing the IgnisHPC client.

------------
Installation
------------

You can install the IgnisHPC client with pip:

.. code-block:: bash

    pip install ignishpc

Once installed, the client provides the ``ignishpc`` command-line tool:

.. code-block:: bash

    ignishpc --help

This will show a list of all available commands and usage options.

-----------------------
Launching the First Job
-----------------------

To run a job, prepare your application code. For example, here is a WordCount application:

.. code-block:: python

    #!/usr/bin/python

    import ignis

    ignis.Ignis.start()

    prop = ignis.IProperties()
    prop["ignis.executor.image"] = "ignishpc/python"
    prop["ignis.executor.instances"] = "1"
    prop["ignis.executor.cores"] = "2"
    prop["ignis.executor.memory"] = "1GB"

    cluster = ignis.ICluster(prop)
    worker = ignis.IWorker(cluster, "python")

    text = worker.textFile("text.txt")
    words = text.flatmap(lambda line: [(word, 1) for word in line.split()])
    count = words.toPair().reduceByKey(lambda a, b: a + b)
    count.saveAsTextFile("wordcount.txt")

    ignis.Ignis.stop()

Save the code as ``driver.py`` and create a ``text.txt`` file in the same directory with sample input.

You can then run the job with:

.. code-block:: bash

    ignishpc run driver.py

Or use the ``job`` subcommand:

.. code-block:: bash

    ignishpc job run driver.py

After the job completes, the output will be available in ``wordcount.txt`` in the current working directory.

To check the execution logs, go to the job directory (e.g., ``driverpy-0sAdh``) and look inside the ``logs`` folder.

-------------------
Next Steps
-------------------

To learn how to list or cancel jobs, or how to edit client configuration, refer to the :doc:`commands` section.