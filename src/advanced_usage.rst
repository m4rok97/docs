Advanced Usage
==============

This section describes advanced workflows for working with IgnisHPC images, covering Docker vs. Singularity, scheduler options, and registry considerations.

-------------------------
1. Docker vs. Singularity
-------------------------

IgnisHPC supports both Docker and Singularity container formats. Although both provide isolated environments, there are important differences:

- **Docker**

  - Widely used for local development and single-node workloads.  
  - Relies on a running Docker daemon with root privileges (unless configured otherwise).  
  - Ideal for building and testing images before deploying to production.  

- **Singularity**  

  - Designed for HPC environments where users typically do not have root access.  
  - Runs containers as the calling user by default, integrating seamlessly with shared filesystems.  
  - Supports converting Docker images into a single SIF (Singularity Image Format) file.  
  - Preferred for Slurm-based multi-node clusters where Docker may not be permitted.

Use Docker for local, single-node experimentation and image building; use Singularity when you need user-space execution on HPC clusters or want portable, immutable SIF files.

------------------------
2. Scheduler Options
------------------------

In the current IgnisHPC release, three scheduler backends are supported:

- **Docker Scheduler**

  - Runs jobs in Docker containers on the current node.  
  - Suitable for development or single-node deployments.  
  - Job submission command:

    ``ignishpc run driver.py --img <docker-image>``

- **Singularity Scheduler**

  - Runs jobs inside Singularity containers on the local node.  
  - Used when Docker is unavailable or not permitted (e.g. shared HPC login nodes).
  - It's the default scheduler if no other is specified.
  - To select Singularity as the scheduler, pass the scheduler name as a job property:

    ``-p "ignis.scheduler.name=singularity"``  
  
  - Example job submission (Singularity):

    ``ignishpc run driver.py --img <singularity.sif> -p "ignis.scheduler.name=singularity``

- **Slurm Scheduler**

  - Submits jobs via Slurm to a multi-node cluster.  
  - To select Slurm as the scheduler, pass the scheduler name as a job property:

    ``-p "ignis.scheduler.name=slurm"``

  - Example job submission (Slurm):

    ``ignishpc run driver.py --image ./tmp.sif -p "ignis.scheduler.name=slurm"`` 

  - Under the hood, IgnisHPC will use Slurm to allocate resources and run the job across multiple nodes.


**Single-node vs. Multi-node**

- Docker and Singularity backends are limited to a single host: all executor containers run on the same machine where you issue ``ignishpc run``.  
- Slurm enables multi-node parallelism: IgnisHPC distributes tasks across multiple compute nodes managed by the cluster's Slurm controller.

Choose the scheduler that matches your environment:

- For local testing, Docker or Singularity is fastest to set up.
- For HPC-scale jobs, prefer Slurm to leverage multiple nodes and shared filesystems.

-----------------------------
3. Building Custom Images
-----------------------------

You can build IgnisHPC images from any combination of core repositories. Example:

.. code-block:: bash

   ignishpc images build \
     -s https://github.com/ignishpc/core-base \
     -s https://github.com/ignishpc/backend \
     -s https://github.com/ignishpc/core-python \
     -s https://github.com/ignishpc/core-go \
     -s https://github.com/ignishpc/core-cpp \
     --registry localhost:5000 \
     --namespace ignishpc \
     --tag v1.0

- ``-s`` / ``--source``: URL or path for each repo  
- ``--registry``: your registry host (here, localhost:5000)  
- ``--namespace``: image namespace (default “ignishpc”)  
- ``--tag``: image tag (e.g. “v1.0”)

Note: You can use local repositories by specifying the path to the repo instead of a URL.

For multi-architecture builds (requires Docker Buildx plugin):

.. code-block:: bash

   ignishpc images build \
     --buildx \
     --arch linux/amd64,linux/arm64 \
     -s https://github.com/ignishpc/core-base \
     -s https://github.com/ignishpc/core-python

---------------------------
3. Pushing & Pulling Images
---------------------------

After building, push images to your local registry:

.. code-block:: bash

   ignishpc images push \
     --pattern ignishpc/core-* \
     --yes

To pull and optionally convert to Singularity:

.. code-block:: bash

   ignishpc images pull \
     --local \
     --singularity core-base.sif \
     ignishpc/core-base:v1.0

----------------------------------
4. Running Jobs with Custom Images
----------------------------------

Reference your locally-built image when submitting a job:

.. code-block:: bash

   ignishpc run \
     --img localhost:5000/ignishpc/core-base:v1.0 \
     driver.py

Or, with your Singularity SIF:

.. code-block:: bash

   ignishpc run \
     --img ./core-base.sif \
     driver.py

------------------------
5. Cleanup & Maintenance
------------------------

Remove outdated or untagged images:

.. code-block:: bash

   ignishpc images rm \
     --pattern core-base \
     --untagged \
     --yes

If you need a fresh registry:

.. code-block:: bash

   ignishpc services registry destroy
   ignishpc deploy registry start --default

------------------------
6. Local Docker Registry
------------------------

IgnisHPC doesn't yet provide a built-in registry subcommand. Instead, you can spin up a local Docker registry with the Docker CLI:

.. code-block:: bash

   # Pull the official registry image
   docker pull registry:2

   # Run a registry container on localhost:5000
   docker run -d \
     --name ignis-registry \
     --restart=always \
     -p 5000:5000 \
     registry:2

This starts a registry listening on **localhost:5000**.

If you encounter an “insecure registry” warning, add this to ``/etc/docker/daemon.json`` and restart Docker:

.. code-block:: json

   {
     "insecure-registries": ["localhost:5000"]
   }

Verify the registry is running:

.. code-block:: bash

   docker ps --filter name=ignis-registry
   docker pull hello-world

Your local registry is now ready for IgnisHPC image pushes and pulls.