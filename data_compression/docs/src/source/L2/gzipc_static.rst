.. Copyright © 2019–2024 Advanced Micro Devices, Inc

.. `Terms and Conditions <https://www.amd.com/en/corporate/copyright>`_.

=========================================
AMD GZip Static Streaming Compression
=========================================

The GZip example resides in the ``L2/tests/gzipc_static`` directory. 

Follow the build instructions to generate the host executable and binary.

The binary host file generated is named "**xil_gzip**", and it is present in the ``./build`` directory.

Executable Usage
----------------

1. To execute single file for compression: ``./build_dir.<TARGET mode>.<xsa_name>/xil_gzip -xbin ./build_dir.<TARGET mode>.<xsa_name>/compress.xclbin -c <file_name>``
2. To execute multiple files for compression: ``./build_dir.<TARGET mode>.<xsa_name>/xil_gzip -xbin ./build_dir.<TARGET mode>.<xsa_name>/compress.xclbin -cfl <files.list>``

	- ``<files.list>``: Contains various file names with the current path.

The usage of the generated executable is as follows:

.. code-block:: bash
 
   Usage: application.exe -[-h-c-cfl-xbin-B]
          --help,                -h        Print Help Options
          --compress,            -c        Compress
          --compress_list,       -cfl      Compress List of Input Files
          --max_cr,              -mcr      Maximum CR                                      Default: [10]
          --xclbin,              -xbin     XCLBIN
          --device_id,           -id       Device ID                                       Default: [0]
          --zlib,                -zlib     [0:GZip, 1:Zlib]                                Default: [0]
 
Results
-------

Resource Utilization 
~~~~~~~~~~~~~~~~~~~~~

The following table presents the resource utilization of GZip static streaming compression kernels. The final Fmax achieved is 300 MHz.

========== ===== ====== ===== ===== ===== 
Flow       LUT   LUTMem REG   BRAM  URAM 
========== ===== ====== ===== ===== ===== 
Compress   35K   2.3K   36.6K  45    64    
========== ===== ====== ===== ===== ===== 

Performance Data
~~~~~~~~~~~~~~~~

The following table presents the kernel throughput achieved for a single compute unit. 

============================= =========================
Topic                         Results
============================= =========================
Compression Throughput        2 Gb/s
Average Compression Ratio     2.31x (Silesia Benchmark)
============================= =========================

Standard GZip Support
---------------------

This application is compatible with the standard Gzip/Zlib application (compress/decompress).  
