Commands
========

This section documents the full command-line interface provided by the ``ignishpc`` tool.

The CLI is organized into functional groups such as ``job``, ``images``, ``config``, etc. Some commands are top-level (like ```run```), while others are nested within groups.

Run ``ignishpc --help`` or ``ignishpc <command> --help`` for in-terminal help.

----------------------
Global CLI Parameters
----------------------

.. code-block:: bash

  ignishpc [-d] [-c path] <command> [options]

**Options:**

- ``-d, --debug``: Display debugging information.
- ``-c, --config path``: Specify a configuration file.

------------------
Top-Level Commands
------------------

The following commands are available directly under the ``ignishpc`` CLI:


Run a Job
^^^^^^^^^

.. code-block:: bash

   ignishpc run [options] <command> [args...]

**Description**  
Submit and execute a job script or command in IgnisHPC.

**Positional Arguments**

- ``command``  The executable or script to run (e.g., ``myapp.py``)  
- ``args``     Additional arguments passed to the command  

**Options**

- ``-n, --name str``                 Specify a human-friendly name for the job  
- ``-j, --img str``                  Specify a container image for all job tasks  
- ``-p, --property key=value``       Set an arbitrary job property  
- ``-i, --interactive``              Attach to STDIN/STDOUT/STDERR (job ends when you exit)  
- ``-e, --env key=value``            Set an environment variable inside the job  
- ``-b, --bind key[=value]``         Bind-mount a host path into the job’s workspace  
- ``-t, --time [[dd-]hh:]mm:ss``      Limit total job runtime (e.g., ``1:30:00`` for 1h30m)  
- ``-s, --static path|int``          Force static resource allocation; pass ``int`` for homogeneous cluster  
- ``-v, --verbose``                  Enable detailed execution logs  

**Resource Aliases**  
These shorthand flags map to common executor/driver properties:

- ``--cores n``               Executor cores (ignis.executor.cores)  
- ``--instances n``           Executor instances (ignis.executor.instances)  
- ``--mem n``                 Executor memory (ignis.executor.memory)  
- ``--gpu str``               Executor GPU type (ignis.executor.gpu)  
- ``--driver-cores n`` / ``--dcores n``   Driver cores (ignis.driver.cores)  
- ``--driver-mem n`` / ``--dmem n``       Driver memory (ignis.driver.memory)  
- ``--driver-img str`` / ``--dimg str``   Driver image (ignis.driver.image)  

**Examples**  

.. code-block:: bash

   # Run a simple script
   ignishpc run myapp.py

   # Override resources and pass script arguments
   ignishpc run --cores 4 --instances 2 --mem 10GB myapp.py --input data.txt

   # Use a custom image and static allocation
   ignishpc run --img ./myimg.sif --cores 4 --static - myapp.py arg1 arg2


Show Version
^^^^^^^^^^^^

.. code-block:: bash

   ignishpc version

**Description**  
Display the current version of the IgnisHPC client.



-----------------
Job Command Group
-----------------

.. code-block:: bash

  ignishpc job <action> [options] [arguments]

Manage jobs in the IgnisHPC system.

Available Actions:

- ``run``     Submit a new job using a script  
- ``list``    List active and recent jobs  
- ``info``    Get detailed information about a job  
- ``cancel``  Cancel a running job  


Run a Job
^^^^^^^^^


.. code-block:: bash

  ignishpc job run [options] <command> [args...]

**Description**  
Submit a new job to IgnisHPC by specifying the command or script to run.

**Positional Arguments** 

- ``command`` The executable or script (e.g. ``driver.py``)  
- ``args``    Additional arguments passed to the command  

**Options**  

- ``-n, --name str``              Specify a name for the job  
- ``-j, --img str``               Specify a container image for all tasks  
- ``-p, --property key=value``    Set an arbitrary job property  
- ``-i, --interactive``           Attach to STDIN/STDOUT/STDERR; job stops when you exit  
- ``-e, --env key=value``         Set an environment variable inside the job  
- ``-b, --bind key[=value]``      Bind-mount a host path into the job  
- ``-t, --time [[dd-]hh:]mm:ss``  Set a runtime limit for the job  
- ``-s, --static path|int``       Force static cluster allocation (pass int for homogeneous cluster)  
- ``-v, --verbose``               Enable verbose job logs  

**Resource Aliases**  
These shorthand flags set common executor/driver properties:

- ``--cores n``: Executor cores (ignis.executor.cores)  
- ``--instances n``: Executor instances (ignis.executor.instances)  
- ``--mem n``: Executor memory (ignis.executor.memory)  
- ``--gpu str``: Executor GPU type (ignis.executor.gpu)  
- ``--driver-cores n`` / ``--dcores n``: Driver cores (ignis.driver.cores)  
- ``--driver-mem n`` / ``--dmem n``: Driver memory (ignis.driver.memory)  
- ``--driver-img str`` / ``--dimg str``: Driver image (ignis.driver.image)  

**Examples**  

.. code-block:: bash

  # Basic run
  ignishpc job run myapp.py

  # With resource overrides
  ignishpc job run --cores 4 --instances 2 --mem 10GB myapp.py --input data.txt


List Jobs
^^^^^^^^^


.. code-block:: bash

  ignishpc job list

**Description**  
Displays all active and recently completed jobs, along with their IDs and statuses.


Job Info
^^^^^^^^

.. code-block:: bash

  ignishpc job info [options] <job-id>

**Description**  
Shows detailed metadata and status for a specific job.

**Positional Arguments**
  
- ``job-id``          The ID of the job to inspect  

**Options**  
- ``-f, --field str`` Only display the value of a single field (e.g. “status”, “startTime”)


Cancel Job
^^^^^^^^^^

.. code-block:: bash

  ignishpc job cancel <job-id>

**Description**  
Stops a running job and frees its resources.

**Positional Arguments**  
- ``job-id`` The ID of the job to cancel  


------------------------
Images Command Group
------------------------

.. code-block:: bash

   ignishpc images <action> [options] [arguments]

Manage Docker container images in IgnisHPC (Docker v23.0+ required).

Available Actions:

- ``build`` Build images  
- ``list`` Display images  
- ``rm`` Remove images  
- ``push`` Push images to the registry  
- ``pull`` Pull an image


Build Images
^^^^^^^^^^^^

.. code-block:: bash

   ignishpc images build [options]

**Description**  
Build IgnisHPC container images from one or more source repositories.

**Options**

- ``-s, --source path/url``  
  Repository URL or local path. URL may include ``[tag]``.  
- ``--name str``  
  Name of the final image. Default: ``ignishpc``. Use ``-`` to disable naming.  
- ``-g, --get-core name``  
  Only build core images matching wildcard `name`.  
- ``--core-images``  
  Build isolated core images only.  
- ``-r, --registry str``  
  Image registry to push to (default: Docker Hub).  
- ``-n, --namespace str``  
  Image namespace (default: ``ignishpc``).  
- ``-t, --tag str``  
  Image tag (default: ``latest``).  
- ``--log``  
  Record build logs for each image.  
- ``--arch ARCH``  
  Target architecture(s), e.g. ``linux/amd64,linux/arm64``.  
- ``-a, --all``  
  Build all optional images.  
- ``-j, --jobs n``  
  Parallel build jobs; default: automatic.  
- ``--ignore folder [folder ...]``  
  Skip building images matching wildcard patterns.  
- ``--enable folder [folder ...]``  
  Only build optional images matching wildcard patterns.  
- ``--dry-run``  
  Simulate the build without creating images.  
- ``--buildx``  
  Use Docker Buildx for multi-arch builds (requires Buildx plugin).

**Examples**

.. code-block:: bash

   # Build from a remote URL
   ignishpc images build -s https://github.com/ignishpc/core-base

   # Multi-architecture build with Buildx
   ignishpc images build --buildx --arch linux/amd64,linux/arm64,linux/ppc64le

   # Build only specific core images without naming
   ignishpc images build -g coreA -g coreB --core-images --name -

   # Build all optional images from a URL
   ignishpc images build -a -s https://github.com/ignishpc/dockerfiles.git


List Images
^^^^^^^^^^^

.. code-block:: bash

   ignishpc images list [options]

**Description**  
Display available container images in your local registry.

**Options**

- ``-p, --pattern str``  
  Filter images using a wildcard pattern.  
- ``-u, --untagged``  
  Include images that have no tags.


Remove Images
^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc images rm [options]

**Description**  
Remove container images from the local registry.

**Options**

- ``-p, --pattern str``  
  Filter images by wildcard pattern.  
- ``-u, --untagged``  
  Remove images without tags.  
- ``-f, --force``  
  Force removal (ignore errors).  
- ``-y, --yes``  
  Skip confirmation prompts.


Push Images
^^^^^^^^^^^

.. code-block:: bash

   ignishpc images push [options]

**Description**  
Push images from the local registry to the configured remote registry.

**Options**

- ``-p, --pattern str``  
  Filter which images to push by wildcard pattern.  
- ``-y, --yes``  
  Skip confirmation prompts.


Pull Image
^^^^^^^^^^

.. code-block:: bash

   ignishpc images pull [options] <image>

**Description**  
Pull an image from a registry. Optionally convert it to a Singularity SIF.

**Positional Arguments**

- ``image``  
  Name of the image to pull (e.g., ``ignishpc/python:latest``).

**Options**

- ``-s path, --singularity path``  
  Convert pulled image to a Singularity file at ``path``.  
- ``-l, --local``  
  Use a local registry mirror.  
- ``--arch ARCH``  
  Pull a specific architecture variant (e.g., ``arm64``).

**Examples**

.. code-block:: bash

   # Pull and convert to Singularity
   ignishpc images pull ignishpc/python:latest -s ignishpc-python.sif

   # Pull from a local mirror
   ignishpc images pull myapp:1.0 --local


---------------------------
Configuration Command Group
---------------------------

.. code-block:: bash

   ignishpc config <action> [options] [arguments]

Manage IgnisHPC client configuration settings.

Available Actions:

- ``info``  – Show the entire configuration  
- ``list``  – List all property keys  
- ``set``   – Define or update property values  
- ``get``   – Retrieve property values  
- ``rm``    – Remove property entries  

**Examples**

.. code-block:: bash

   $ ignishpc config info
   $ ignishpc config list
   $ ignishpc config set ignis.container.docker.registry=mynode:5000 ignis.wdir=~
   $ ignishpc config get -s ignis.container.provider
   $ ignishpc config rm -u ignis.container.image


Show Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc config info

**Description**  
Display the full merged configuration from both user and system files.


List Properties
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc config list [options]

**Description**  
List all property keys currently defined.

**Options**

- ``-s, --split``  Show keys split by configuration file (user vs system)


Set Properties
^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc config set [options] key=value [key=value ...]

**Description**  
Create or update one or more configuration properties.

**Positional Arguments**  

- ``key=value``  One or more property assignments

**Options**  

- ``-s, --system``  Write changes to the system-wide config file instead of the user file


Get Properties
^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc config get [options] key [key ...]

**Description**  
Retrieve the value(s) of specified property key(s).

**Positional Arguments**

- ``key``  One or more property names

**Options**  

- ``-u, --user``      Only read from user config file  
- ``-s, --system``    Only read from system config file  
- ``-f, --fail``      Exit with error if any key is not found  
- ``-v, --only-value`` Print only the property value (omit key)  
- ``-p, --plain-value`` Require that the property value is a plain string (no list/map)


Remove Properties
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc config rm [options] key [key ...]

**Description**  
Delete one or more property entries.

**Positional Arguments**  

- ``key``  One or more property names to remove

**Options**

- ``-u, --user``      Remove only from the user config file  
- ``-s, --system``    Remove only from the system config file  
- ``-a, --all``       Remove from both files if defined in both



--------------------------
Completion Command Group
--------------------------

.. code-block:: bash

   ignishpc completion [options]

Generate shell autocompletion scripts for the ``ignishpc`` CLI.

**Options**

- ``-h, --help``  
  Show this help message and exit.

**Usage**

To load completions into your current shell session:

.. code-block:: bash

   source <(ignishpc completion)

To enable completions in all future sessions (e.g., for Bash):

.. code-block:: bash

   ignishpc completion > ~/.bash_completion.d/ignishpc

You can adapt the output path to your shell's completion directory (e.g., ``~/.zsh/completions/``).  


----------------------
Services Command Group
----------------------

.. code-block:: bash

   ignishpc services <service> <action> [options]

Docker-Based Service Management

Available Services:

- ``registry``     Service for managing Docker image registry  
- ``registry-ui``  Web interface for the Docker registry service  
- ``etcd``         Service for managing etcd for container discovery  

Examples:

.. code-block:: bash

   # Start registry with self-signed HTTPS
   ignishpc services registry start --https-self

   # Destroy the registry service
   ignishpc services registry destroy

Note:  
  The path ``/etc/ignis/<service>`` is mounted for certificates and config (e.g. domain.crt, domain.key, secret).


Registry Service
^^^^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc services registry <action> [options]

Service for managing Docker image registry.

Actions:

- ``start``   Start the registry service  
- ``stop``    Stop the registry service  
- ``resume``  Resume the registry service  
- ``destroy`` Destroy the registry service  
- ``status``  Show registry service status  

**Examples**

.. code-block:: bash

   ignishpc services registry start --https-self
   ignishpc services registry destroy

Start Service

.. code-block:: bash

   ignishpc services registry start [options]

**Options**

- ``-b, --bind address``  
  Address for internal communications (default: all interfaces)  
- ``-p, --port int``  
  Server port (default: 5000)  
- ``-e, --env key=value``  
  Set a registry environment variable  
- ``--path str``  
  Path to store registry data (default: ``/var/lib/ignis/registry``)  
- ``--https``  
  Enable HTTPS (default port: 443)  
- ``-f, --force``  
  Destroy existing instance before starting  

Stop Service

.. code-block:: bash

   ignishpc services registry stop

Stop the registry service.

Resume Service

.. code-block:: bash

   ignishpc services registry resume

Resume the registry service.

Destroy Service

.. code-block:: bash

   ignishpc services registry destroy

Destroy the registry service.

Status Service

.. code-block:: bash

   ignishpc services registry status

Show the current status of the registry service.


Registry-UI Service
^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   ignishpc services registry-ui <action> [options]

Web interface for the Docker registry service.

Actions:

- ``start``   Start the registry-UI service  
- ``stop``    Stop the registry-UI service  
- ``resume``  Resume the registry-UI service  
- ``destroy`` Destroy the registry-UI service  
- ``status``  Show registry-UI service status  

Start Service

.. code-block:: bash

   ignishpc services registry-ui start [options]

**Options**

- ``-p, --port int``  
  Server port (default: 3000)  
- ``-u, --url str``  
  URL of your Docker registry (defaults to port 5000)  
- ``-e, --env key=value``  
  Set an environment variable  
- ``-v, --volume path``  
  Mount a host path into the UI container  
- ``-f, --force``  
  Destroy existing instance before starting  

Stop, Resume, Destroy, Status

.. code-block:: bash

   ignishpc services registry-ui <stop|resume|destroy|status>

Manage the registry-UI service without additional options.


etcd Service
^^^^^^^^^^^^

.. code-block:: bash

   ignishpc services etcd <action> [options]

Service for managing etcd for container discovery.

Actions:

- ``start``   Start the etcd service  
- ``stop``    Stop the etcd service  
- ``resume``  Resume the etcd service  
- ``destroy`` Destroy the etcd service  
- ``status``  Show etcd service status  

**Examples**

.. code-block:: bash

   ignishpc services etcd start --secure
   ignishpc services etcd destroy

Start Service

.. code-block:: bash

   ignishpc services etcd start [options]

**Options**

- ``-b, --bind address``  
  Address for internal communications (default: first network interface)  
- ``-p, --port int``  
  Client port (default: 2379)  
- ``--extra-port EXTRA_PORT``  
  Additional ports to expose  
- ``--path str``  
  Path to store etcd data (default: ``/var/lib/ignis/etcd``)  
- ``-s, --secure``  
  Enable self-signed transport security  
- ``--extra-args 'args'``  
  Pass additional binary arguments  
- ``-f, --force``  
  Destroy existing instance before starting  

Stop, Resume, Destroy, Status

.. code-block:: bash

   ignishpc services etcd <stop|resume|destroy|status>

Manage the etcd service without additional options.