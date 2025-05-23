 .. Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

 ..   http://www.apache.org/licenses/LICENSE-2.0

 .. Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

Manually building and testing release candidate distributions
=============================================================

Breeze can be used to test new release candidates of distributions - both Airflow and providers. You can easily
configure the CI image of Breeze to install and start Airflow for both Airflow and providers, whether they
are built from sources or downloaded from PyPI as release candidates.

**The outline for this document in GitHub is available at top-right corner button (with 3-dots and 3 lines).**

Prerequisites
-------------

The way to test it is rather straightforward:

1) Make sure that the distributions - both ``airflow`` and ``providers`` are placed in the ``dist`` folder
   of your Airflow source tree. You can either build them there or download from PyPI (see the next chapter).

2) You can run ``breeze shell`` or ``breeze start-airflow`` commands with adding the following flags -
   ``--mount-sources remove``, ``--use-distributions-from-dist``, and ``--use-airflow-version wheel/sdist``. The first one
   removes the ``airflow`` source tree from the container when starting it, the second one installs ``airflow`` and
   ``providers`` distributions from the ``dist`` folder when entering breeze, and the third one specifies the distribution's
   format (either ``wheel`` or ``sdist``). Omitting the latter will result in skipping the installation of the
   distribution(s), and a consequent error when later importing them.

Testing pre-release distributions
---------------------------------

There are two ways how you can get Airflow distributions in ``dist`` folder - by building them from sources or
downloading them from PyPI.

.. note ::

    Make sure you run ``rm dist/*`` before you start building distributions or downloading them from PyPI because
    the distributions built there already are not removed manually.

In order to build apache-airflow from sources, you need to run the following command:

.. code-block:: bash

    breeze release-management prepare-airflow-distributions

In order to build providers from sources, you need to run the following command:

.. code-block:: bash

    breeze release-management prepare-provider-distributions <PROVIDER_1> <PROVIDER_2> ... <PROVIDER_N>

The distributions are built in ``dist`` folder and the command will summarise what distributions are available in the
``dist`` folder after it finishes.

If you want to download the distributions from PyPI, you need to run the following command:

.. code-block:: bash

    pip download apache-airflow-providers-<PROVIDER_NAME>==X.Y.Zrc1 --dest dist --no-deps

You can use it for both release and pre-release distributions.

Examples of testing pre-release distributions
---------------------------------------------

Few examples below explain how you can test pre-release distributions, and combine them with locally build
and released distributions.

The following example downloads ``apache-airflow`` and ``celery`` and ``kubernetes`` providers from PyPI and
eventually starts Airflow with the Celery Executor. It also loads example dags and default connections:

.. code:: bash

    rm dist/*
    pip download apache-airflow==2.9.0rc1 --dest dist --no-deps
    pip download apache-airflow-providers-celery==3.6.2rc1 --dest dist --no-deps
    pip download apache-airflow-providers-cncf-kubernetes==8.1.0rc1 --dest dist --no-deps
    breeze start-airflow --mount-sources remove --use-distributions-from-dist --use-airflow-version sdist --executor CeleryExecutor --backend postgres --load-default-connections --load-example-dags


The following example downloads ``celery`` and ``kubernetes`` providers from PyPI, builds
``apache-airflow`` distribution from the main sources and eventually starts Airflow with the Celery Executor.
It also loads example dags and default connections:

.. code:: bash

    rm dist/*
    breeze release-management prepare-airflow-distributions
    pip download apache-airflow-providers-celery==3.6.2rc1 --dest dist --no-deps
    pip download apache-airflow-providers-cncf-kubernetes==8.1.0rc1 --dest dist --no-deps
    breeze start-airflow --mount-sources remove --use-distributions-from-dist --use-airflow-version sdist --executor CeleryExecutor --backend postgres --load-default-connections --load-example-dags

The following example builds ``celery``, ``kubernetes`` providers from the main sources, downloads 2.9.0 version
of ``apache-airflow`` distribution from PyPI and eventually starts Airflow using default executor
for the backend chosen (no example dags, no default connections):

.. code:: bash

    rm dist/*
    breeze release-management prepare-provider-distributions celery cncf.kubernetes
    pip download apache-airflow==2.9.0 --dest dist --no-deps
    breeze start-airflow --mount-sources remove --use-distributions-from-dist --use-airflow-version sdist

You can mix and match distributions from PyPI (final or pre-release candidates) with locally build distributions. You
can also choose which providers to install this way since the ``--mount-sources remove`` flag makes sure that Airflow
installed does not contain all the providers - only those that you explicitly downloaded or built in the
``dist`` folder. This way you can test all the combinations of Airflow and Providers you might need.

-----

For other kinds of tests look at `Testing document <../09_testing.rst>`__
