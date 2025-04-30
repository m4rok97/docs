Advanced Usage
==============

This section dives into power-user workflows for building and managing your own IgnisHPC images, running them via a local registry, and converting to Singularity.

------------------------
1. Local Docker Registry
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

-------------------------
2. Building Custom Images
-------------------------

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
5. Singularity Workflow
------------------------

If you prefer Singularity end-to-end:

.. code-block:: bash

   # Build Docker image for feature branch
   ignishpc images build \
     -s "https://github.com/ignishpc/core-base.git" \
     -s "https://github.com/ignishpc/core-python.git" \
     --registry localhost:5000 \
     --tag singularity-test

   # Pull image locally and convert to SIF
   ignishpc images pull \
     --local \
     --singularity tmp.sif \
     ignishpc/core-base:singularity-test

   # Run with SIF
   ignishpc run --img ./tmp.sif ls

------------------------
6. Cleanup & Maintenance
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