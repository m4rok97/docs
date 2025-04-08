IgnisHPC
========

.. image:: _static/img/ignis-hpc.svg
   :align: right
   :target: index.html
   :width: 112
   :alt: IgnisHPC logo

.. only:: html or man

   :Authors:   César Piñeiro, Juan C. Pichel
   :Affiliation: `CiTIUS <https://citius.usc.es/>`_, Universidade de Santiago de Compostela (Spain)
   :Contact:  cesaralfredo.pineiro@usc.es, juancarlos.pichel@usc.es
   :Date:     |today|

.. topic:: About

   One of the most important issues in the path to the convergence of High Performance Computing (HPC) and Big Data is caused by the differences in their software stacks. Despite some research efforts, the interoperability between their programming models and languages is still limited. To deal with this problem we introduce a new computing framework called *IgnisHPC*, whose main objective is to unify the execution of Big Data and HPC workloads in the same framework. *IgnisHPC* has native support for multi-language applications using JVM and non-JVM-based languages (currently Java, Python and C/C++). Since MPI was used as its backbone technology, *IgnisHPC* takes advantage of many communication models and network architectures. Moreover, MPI applications can be directly executed in a efficient way in the framework. The main consequence is that users could combine in the same multi-language code HPC tasks (using MPI) with Big Data tasks (using MapReduce operations). The experimental evaluation demonstrates the benefits of our proposal in terms of performance and productivity with respect to other frameworks such as Spark. *IgnisHPC* is publicly available for the Big Data and HPC research community.

.. toctree::
   :caption: Contents
   :maxdepth: 2

   started
   api
   images
   properties
