.. _tutorials_mpmd:

AMReX-MPMD
==========

AMReX-MPMD utilizes the Multiple Program Multiple Data (MPMD) feature of MPI to provide cross-functionality for AMReX-based codes.
The framework enables data transfer across two different applications through **MPMD::Copier** class, which typically takes **BoxArray** of its application as an argument.
**Copier** instances created in both the applications together identify the overlapping cells for which the data transfer must occur.
**Copier::send** & **Copier::recv** functions, which take a **MultiFab** as an argument, are used to transfer the desired data of overlapping regions.

Case-1
------

The current case demonstrates the MPMD capability across two C++ applications.

Contents
^^^^^^^^

**Source_1** subfolder contains ``main_1.cpp`` which will be treated as the first application.
Similarly, **Source_2** subfolder contains ``main_2.cpp`` that will be treated as the second application.

Overview
^^^^^^^^

The domain in ``main_1.cpp`` is set to ``lo = {0, 0, 0}`` and ``hi = {31, 31, 31}``, while the domain in ``main_2.cpp`` is set to ``lo = {16, 16, 16}`` and ``hi = {31, 31, 31}``.
Hence, the data transfer will occur for the region ``lo = {16, 16, 16}`` and ``hi = {31, 31, 31}``.
Furthermore, the domain in ``main_1.cpp`` is split into boxes using ``max_grid_size=16``, while the domain in ``main_2.cpp`` is split using ``max_grid_size=8``.
Therefore, the **BoxArray**, and moreover, the number of boxes for the overlapping region are different across the two applications.
The data transfer demonstration is performed using a two component *MultiFab*.
The first component is populated in ``main_1.cpp`` before it is transferred to ``main_2.cpp``.
The second component is populated in ``main_2.cpp`` based on the received first component.
Finally, the second component is transferred from ``main_2.cpp`` to ``main_1.cpp``.
It can be seen from the plotfile generated by ``main_1.cpp`` that the second component is non-zero only for the overlapping region.

Compile
^^^^^^^

The compile process here assumes that the current working directory is ``ExampleCodes/MPMD/Case-1/``.

.. code-block:: bash

   # cd into Source_1 to compile the first application
   cd Source_1/
   # Include USE_CUDA=TRUE for CUDA GPUs
   make USE_MPI=TRUE

   # cd into Source_2 to compile the second application
   cd ../Source_2/
   # Include USE_CUDA=TRUE for CUDA GPUs
   make USE_MPI=TRUE

Run
^^^

Here, the current case is being run using a total of 12 MPI ranks with 8 allocated to ``main_1.cpp`` and the rest for ``main_2.cpp``.
Please note that MPI ranks attributed to each application/code need to be continuous, i.e., MPI ranks 0-7 are for ``main_1.cpp`` and 8-11 are for ``main_2.cpp``.
This may be default behaviour on several systems.
Furthermore, the run process here assumes that the current working directory is ``ExampleCodes/MPMD/Case-1/``.

.. code-block:: bash

   # Running the MPMD process with 12 ranks
   mpirun -np 8 Source_1/main3d.gnu.DEBUG.MPI.ex : -np 4 Source_2/main3d.gnu.DEBUG.MPI.ex

Running on Perlmutter (NERSC)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section presents information regarding sample scripts that can be used to run this case on `Perlmutter (NERSC) <https://docs.nersc.gov/systems/perlmutter/>`_.
The scripts ``mpmd_cpu.sh`` and ``mpmd_cpu.conf`` can be used to run the CPU version.
Similarly, ``mpmd_gpu.sh`` and ``mpmd_gpu.conf`` can be used to run the GPU version.
Please note that ``perlmutter_gpu.profile`` must be leveraged to compile the GPU version of the applications.

The content presented here is based on the following references:

   * `NERSC documentation <https://docs.nersc.gov/jobs/examples/#mpmd-multiple-program-multiple-data-jobs>`_
   * `WarpX documentation <https://warpx.readthedocs.io/en/latest/install/hpc/perlmutter.html>`_

Case-2
------

The current case demonstrates the MPMD capability across C++ and python applications.
This language interoperability is achieved through the python bindings of AMReX, `pyAMReX <https://github.com/AMReX-Codes/pyamrex>`_.

Contents
^^^^^^^^

``main.cpp`` will be the C++ application and ``main.py`` will be the python application.

Overview
^^^^^^^^

In the previous case (Case-1) of MPMD each application has its own domain, and therefore, different **BoxArray**.
However, there exist scenarios where both applications deal with the same **BoxArray**.
The current case presents such a scenario where the **BoxArray** is defined only in the ``main.cpp`` application, but this information is relayed to ``main.py`` application through the **MPMD::Copier**.

**Please ensure that the same AMReX source code is used to compile both the applications.**

Compiling and Running on a local system
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The compile process for pyAMReX is only briefly described here.
Please refer to the `pyAMReX documentation <https://pyamrex.readthedocs.io/en/latest/install/cmake.html#>`_ for more details.
**It must be mentioned that mpi4py is an important dependency that should exist in the utilized environment**.
For pyAMReX conda environment, it can be installed using ``conda install -c conda-forge mpi4py``, `resource <https://mpi4py.readthedocs.io/en/latest/install.html#using-conda>`_.

.. code-block:: bash

   # find dependencies & configure
   # Include -DAMReX_GPU_BACKEND=CUDA for gpu version
   cmake -S . -B build -DAMReX_SPACEDIM="1;2;3" -DAMReX_MPI=ON -DpyAMReX_amrex_src=/path/to/amrex

   # compile & install, here we use four threads
   cmake --build build -j 4 --target pip_install

main.cpp compile
^^^^^^^^^^^^^^^^

The compile process here assumes that the current working directory is ``ExampleCodes/MPMD/Case-2/``.

.. code-block:: bash

   # Include USE_CUDA=TRUE for CUDA GPUs
   make USE_MPI=TRUE

Run
^^^

Here, the current case is being run using a total of 12 MPI ranks with 8 allocated to ``main.cpp`` and the rest for ``main.py``.
As mentioned earlier, the MPI ranks attributed to each application/code need to be continuous, i.e., MPI ranks 0-7 are for ``main.cpp`` and 8-11 are for ``main.py``.
This may be default behaviour on several systems.
Furthermore, the run process here assumes that the current working directory is ``ExampleCodes/MPMD/Case-2/``.

.. code-block:: bash

   # Running the MPMD process with 12 ranks
   mpirun -np 8 ./main3d.gnu.DEBUG.MPI.ex : -np 4 python main.py

Compiling and Running on Perlmutter (NERSC)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Running this case on perlmutter involves creating a python virtual environment.
pyAMReX must be compiled and installed into this virtual environment after its creation.
Similar to the previous case, this case also has supporting scripts to run on CPUs and GPUs.

The process detailed below assumes that the current working directory is ``ExampleCodes/MPMD/Case-2/``.

Creating a virtual environment
""""""""""""""""""""""""""""""

.. code-block:: bash

   # Setup the required environment variables
   source perlmutter_gpu.profile

   # BEFORE PERFORMING THE FOLLOWING COMMANDS
   # MOVE TO A DIRECTORY WHERE THE PYTHON VIRTUAL ENVIRONMENT MUST EXIST

   python3 -m pip install --upgrade pip
   python3 -m pip install --upgrade virtualenv
   python3 -m pip cache purge
   python3 -m venv pyamrex-gpu
   source pyamrex-gpu/bin/activate
   python3 -m pip install --upgrade pip
   python3 -m pip install --upgrade build
   python3 -m pip install --upgrade packaging
   python3 -m pip install --upgrade wheel
   python3 -m pip install --upgrade setuptools
   python3 -m pip install --upgrade cython
   python3 -m pip install --upgrade numpy
   python3 -m pip install --upgrade pandas
   python3 -m pip install --upgrade scipy
   MPICC="cc -target-accel=nvidia80 -shared" python3 -m pip install --upgrade mpi4py --no-cache-dir --no-build-isolation --no-binary mpi4py
   python3 -m pip install --upgrade openpmd-api
   python3 -m pip install --upgrade matplotlib
   python3 -m pip install --upgrade yt
   python3 -m pip install --upgrade cupy-cuda12x  # CUDA 12 compatible wheel

The content presented here is based on the following reference:

   * `WarpX documentation <https://warpx.readthedocs.io/en/latest/install/hpc/perlmutter.html>`_
