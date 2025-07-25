---
title: Scaling HPC applications - Azure Virtual Machines | Microsoft Docs
description: Learn how to scale HPC applications on Azure VMs.
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.custom:
ms.topic: how-to
ms.date: 08/06/2024
ms.reviewer: cynthn, mattmcinnes
ms.author: jushiman
author: ju-shim
# Customer intent: "As a cloud architect, I want to optimize the deployment of HPC applications on Azure VMs, so that I can enhance scalability and performance for large-scale computational tasks."
---

# Scaling HPC applications

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Optimal scale-up and scale-out performance of HPC applications on Azure requires performance tuning and optimization experiments for the specific workload. This section and the VM series-specific pages offer general guidance for scaling your applications.

## Application setup

The [azurehpc repo](https://github.com/Azure/azurehpc) contains many examples of:

- Setting up and running [applications](https://github.com/Azure/azurehpc/tree/master/apps) optimally.
- Configuration of [file systems, and clusters](https://github.com/Azure/azurehpc/tree/master/examples).
- [Tutorials](https://github.com/Azure/azurehpc/tree/master/tutorials) on how to get started easily with some common application workflows.

## Optimally scaling MPI

The following suggestions apply for optimal application scaling efficiency, performance, and consistency:

- For smaller scale jobs (< 256K connections) use: `UCX_TLS=rc,sm`

- For larger scale jobs (> 256K connections) use: `UCX_TLS=dc,sm`

- To calculate the number of connections for your MPI job, use: `Max Connections = (processes per node) x (number of nodes per job) x (number of nodes per job)`

## Adaptive Routing

Adaptive Routing (AR) allows Azure Virtual Machines (VMs) running EDR and HDR InfiniBand to automatically detect and avoid network congestion by dynamically selecting optimal network paths. As a result, AR offers improved latency and bandwidth on the InfiniBand network, which in turn drives higher performance and scaling efficiency. For more information, see [TechCommunity article](https://techcommunity.microsoft.com/t5/azure-compute/adaptive-routing-on-azure-hpc/ba-p/1205217).

## Process pinning

- Pin processes to cores using a sequential pinning approach (as opposed to an autobalance approach).
- Binding by Numa/Core/HwThread is better than default binding.
- For hybrid parallel applications (OpenMP+MPI), use four threads and one MPI rank per [CCX]([HB-series virtual machines overview including info on CCXs](/azure/virtual-machines/hb-series-overview)) on HB and HBv2 VM sizes.
- For pure MPI applications, experiment with between one to four MPI ranks per CCX for optimal performance on HB and HBv2 VM sizes.
- Some applications with extreme sensitivity to memory bandwidth may benefit from using a reduced number of cores per CCX. For these applications, using three or two cores per CCX may reduce memory bandwidth contention and yield higher real-world performance or more consistent scalability. In particular, MPI 'Allreduce' may benefit from this approach.
- For larger scale runs, it's recommended to use UD or hybrid RC+UD transports. Many MPI libraries/runtime libraries use these transports internally (such as UCX or MVAPICH2). Check your transport configurations for large-scale runs.

## Compiling applications
<br>
<details>
<summary>Click to expand</summary>

Though not necessary, compiling applications with appropriate optimization flags provides the best scale-up performance on HB and HC-series VMs.

### AMD Optimizing C/C++ Compiler

The AMD Optimizing C/C++ Compiler (AOCC) compiler system offers a high level of advanced optimizations, multi-threading, and processor support that includes global optimization, vectorization, inter-procedural analyses, loop transformations, and code generation. AOCC compiler binaries are suitable for Linux systems having GNU C Library (glibc) version 2.17 and above. The compiler suite consists of a C/C++ compiler (clang), a Fortran compiler (FLANG), and a Fortran front end to Clang (Dragon Egg).

### Clang

Clang is a C, C++, and Objective-C compiler handling preprocessing, parsing, optimization, code generation, assembly, and linking.
Clang supports the  `-march=znver1` flag to enable best code generation and tuning for AMD’s Zen based x86 architecture.

### FLANG

The FLANG compiler is a recent addition to the AOCC suite (added April 2018) and is currently in prerelease for developers to download and test. Based on Fortran 2008, AMD extends the GitHub version of [FLANG](https://github.com/flang-compiler/flang). The FLANG compiler supports all Clang compiler options and other number of FLANG-specific compiler options.

### DragonEgg

DragonEgg is a gcc plugin that replaces GCC’s optimizers and code generators from the LLVM project. DragonEgg that comes with AOCC works with gcc-4.8.x, has been tested for x86-32/x86-64 targets, and has been successfully used on various Linux platforms.

GFortran is the actual frontend for Fortran programs responsible for preprocessing, parsing, and semantic analysis generating the GCC GIMPLE intermediate representation (IR). DragonEgg is a GNU plugin, plugging into GFortran compilation flow. It implements the GNU plugin API. With the plugin architecture, DragonEgg becomes the compiler driver, driving the different phases of compilation.  After following the download and installation instructions, Dragon Egg can be invoked using:

```bash
gfortran [gFortran flags]
   -fplugin=/path/AOCC-1.2-Compiler/AOCC-1.2-
   FortranPlugin/dragonegg.so [plugin optimization flags]
   -c xyz.f90 $ clang -O3 -lgfortran -o xyz xyz.o $./xyz
```

### PGI Compiler

PGI Community Edition 17 is confirmed to work with AMD EPYC. A PGI-compiled version of STREAM does deliver full memory bandwidth of the platform. The newer Community Edition 18.10 (Nov 2018) should likewise work well. Use this CLI command to compile with the Intel Compiler:

```bash
pgcc $(OPTIMIZATIONS_PGI) $(STACK) -DSTREAM_ARRAY_SIZE=800000000 stream.c -o stream.pgi
```

### Intel Compiler

Intel Compiler 18 is confirmed to work with AMD EPYC. Use this CLI command to compile with the Intel Compiler.

```bash
icc -o stream.intel stream.c -DSTATIC -DSTREAM_ARRAY_SIZE=800000000 -mcmodel=large -shared-intel -Ofast –qopenmp
```

### GCC Compiler

For HPC workloads, AMD recommends GCC compiler 7.3 or newer. Older versions, such as 4.8.5 included with RHEL 7.4, aren't recommended. GCC 7.3, and newer, delivers higher performance on HPL, HPCG, and DGEMM tests.

```bash
gcc $(OPTIMIZATIONS) $(OMP) $(STACK) $(STREAM_PARAMETERS) stream.c -o stream.gcc
```

</details>

## Next steps

- Test your knowledge with a [learning module on optimizing HPC applications on Azure](/training/modules/optimize-tightly-coupled-hpc-apps/).
- Review the [HBv3-series overview](hbv3-series-overview.md) and [HC-series overview](hc-series-overview.md).
- Read about the latest announcements, HPC workload examples, and performance results at the [Azure Compute Tech Community Blogs](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- Learn more about [HPC](/azure/architecture/topics/high-performance-computing/) on Azure.
