Docker Images
=============

.. image:: _static/img/docker-images.svg
   :align: center
   :alt: IgnisHPC images

Background
----------

IgnisHPC is fully containerized, these can be extended to create custom runtime environments and isolate incompatible modules. 
For those unfamiliar with containers, containers are running instances of an image, which is an immutable environment that serves as the 
starting point when the container is started.

IgnisHPC images are organized according to a hierarchy, shown in the figure above, which allows us to extend and create new modules easily and simply. 
The images have an associated namespace that groups all the hierarchy of images, by default it is ``ignishpc`` but a user can create their own. 
Although all images have an important role to play, in most cases users should only familiarize themselves with the core images. 
A core container stores the IgnisHPC implementation for a given language as well as its dependencies. 
Images without an associated IgnisHPC module are stored in the ``Dockerfiles/`` repository.

Note that only images belonging to the IgnisHPC architecture are covered; images of external dependencies are optional and outside the scope of this document.

Build Process
-------------

When you run::

   $ ignishpc images build --sources <repo1> <repo2> [--all] [--buildx] [--arch <arch>]

the ignis client performs the following steps:

1. **Discover** every ``Dockerfiles/`` directory in the specified repositories and collect each subfolder that contains a Dockerfile.  
2. **Build** one Docker image per folder (e.g. ``template``, ``base-builder``, ``builder``, ``coreA-builder``, ``coreA-lib``, …), exactly as declared by that folder's Dockerfile through its ``FROM`` line.  
3. **Assemble runtime images**:
   - **Aggregated image (default)**  
    
    Unless another image name is passed by the param ``--name``, create a single image named ``ignishpc`` that bundles all modules.  
   
   - **Core images (optional)**  
    
    If you include ``--core-images``, also generate individual runtime images for each core and library folder. For each folder:
    
     - Copy its build artifacts (the contents of ``$IGNIS_HOME``) into a fresh ``template`` image.  
     - Run the install script placed under ``$IGNIS_HOME/bin`` (e.g. ``ignis-coreA-builder-install.sh`` or ``ignis-coreA-lib-install.sh``).  
     - Tag the resulting image exactly with the folder name (e.g. ``coreA``, ``coreA-lib``).  
4. **Filter** out any folder whose Dockerfile is labeled ``LABEL ignis.build="optional"`` unless you explicitly include it with ``--all``, or override via ``--ignore``/``--enable``.

This folder-driven workflow guarantees a one-to-one mapping between your repository layout and the IgnisHPC image hierarchy. Just add or remove 
``Dockerfiles/foo-builder/`` or ``Dockerfiles/foo-lib/`` folders to extend or simplify your image set.

Details
-------

**template**  
Defined by the ``template`` folder under ``core-base/Dockerfiles/``. It extends the base operating system image (e.g. Ubuntu) and establishes 
an empty IgnisHPC runtime layout under ``$IGNIS_HOME`` (``bin/``, ``core/``, ``lib/``, ``include/``, ``etc/``, ``env.d/``).

**base-builder**  
Built from ``core-base/Dockerfiles/base-builder/``, extending ``template`` to install the fundamental build environment 
(compilers, common libraries) used by all IgnisHPC modules.

**builder**  
From ``core-base/Dockerfiles/builder/``, extending ``base-builder`` and adding IgnisHPC’s shared build scripts and configuration.

**core-builder**  
Each of these folders (e.g. ``coreA-builder``) extends ``builder``. They compile that language core’s source code into ``$IGNIS_HOME`` 
and provide an install script (``ignis-coreA-builder-install.sh`` or ``ignis-coreB-builder-install.sh``) under ``$IGNIS_HOME/bin/``.

**core-lib**  
If present, these optional folders extend their matching builder (``coreA-lib`` ← ``coreA-builder``). They build additional libraries or headers, 
install them into ``$IGNIS_HOME/lib`` and ``$IGNIS_HOME/include``, and leave an install script (``ignis-coreA-lib-install.sh`` or ``ignis-coreB-lib-install.sh``).

**Aggregated image: ignishpc**  
Built by default (unless you disable it with ``--name``). Extends ``template`` and bundles every core and lib module into one image.

**Core images: core, core-lib**  
Generated only when you specify ``--core-images``. Each extends ``template`` and depends on its corresponding builder or lib image—created by copying that image's ``$IGNIS_HOME`` artifacts into ``template`` and executing its install script.  
