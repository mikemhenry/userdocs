# GPUs

## Description

This page contains all information you need to submit GPU-jobs successfully on Ubelix.

## Important Information on GPU Usage

!!! types note ""
    Code that runs on the CPU will **not** magically make use of GPUs by simply submitting a job to the 'gpu' partition! You have to explicitly adapt your code to run on the GPU, e.g. an CUDA or OpenACC implementation. Also, code that runs on a GPU will not necessarily run faster than it runs on the CPU. For example, GPUs require a huge amount of highly parallelizable tasks. In other words, you must understand the characteristics of your job, and make sure that you only submit jobs to the 'gpu' partition that can actually benefit from GPUs.

When submitting to the GPU partition the GPU type specification is **required**. 

## GPU Type

Ubelix currently features four types of GPUs. You have to choose an architecture and use the following `--gres` option to select it.

| Type | lsf gres option |
| ---- | ----------------- | 
| Nvidia Geforce GTX 1080 Ti | `--gres=gpu:gtx1080ti:<number_of_gpus>` |
| Nvidia Geforce RTX 2080 Ti | `--gres=gpu:rtx2080ti:<number_of_gpus>` |
| Nvidia Geforce RTX 3090 | `--gres=gpu:rtx3090:<number_of_gpus>` |
| Nvidia Tesla P100 | `--gres=gpu:teslap100:<number_of_gpus>` |


## Job Submission

Use the following options to submit a job to the gpu partition using the default job QoS:

```Bash
#SBATCH --partition=gpu
#SBATCH --gres=gpu:<type>:<number_of_gpus>
```


## job_gpu_preempt
For investors we provides investor partitions with specific QoS for each investor, defining the purchased resources. In case of GPU we want/need to provide instant access to purchased GPU resources. Nevertheless, to efficiently use all resources, the `job_gpu_preemt` exists in the `gpu` partition. Jobs, submitted with this QoS, may interrupted if resources are required for investors. Short jobs, and jobs with checkpointing benefit from these additional resources. 

For example requesting 4 RTX2080Ti
```Bash
#SBATCH --partition=gpu
#SBATCH --qos=job_gpu_preempt
#SBATCH --gres=gpu:rtx2080ti:4
```


Use the following option to ensure that the job, if preempted, won't be requeued but canceled instead:

```Bash
#SBATCH --no-requeue
```

## Application adaptation

Applications do only ran on GPUs if they are build specifically for GPUs. There are multiple ways to implement algorithms for GPU usage. The most common ones are low level languages like CUDA or pragma oriented implementations like OpenACC. 

### CUDA
To build and run CUDA applications, its compiler and libraries are provided managed via modules. Run _module avail_ to see which versions are available, for example:

```Bash
module avail CUDA
---- /software.el7/modulefiles/all ----
   CUDA/8.0.61                           cuDNN/7.1.4-CUDA-9.2.88
   CUDA/9.0.176                          cuDNN/7.6.0.64-gcccuda-2019a (D)
   CUDA/9.1.85                           fosscuda/2019a
   CUDA/9.2.88                           fosscuda/2019b               (D)
   CUDA/10.1.105-GCC-8.2.0-2.31.1        gcccuda/2019a
   CUDA/10.1.243                  (D)    gcccuda/2019b                (D)
   cuDNN/6.0-CUDA-8.0.61                 OpenMPI/3.1.3-gcccuda-2019a
   cuDNN/7.0.5-CUDA-9.0.176              OpenMPI/3.1.4-gcccuda-2019b
   cuDNN/7.0.5-CUDA-9.1.85
```

Run `module load <module>` to load a specific version of CUDA:

```Bash
module load cuDNN/7.1.4-CUDA-9.2.88
```

!!! types note ""
    If you need cuDNN you must load the cuDNN module. The appropriate CUDA version is then loaded automatically as a dependency.

## GPU usage monitoring

To verify the **usage** of one or multiple GPUs the `nvidia-smi` tool can be utilized. The tool need to be launched on the related nodes. After the job started running, a new job step can be created using `srun` and call `nvidia-smi` to display the resource utilization. Here we attach the process to an job with the jobID `123456`. You need to replace the jobId with your gathered jobID, presented in the sbatch output. 

```Bash
$ sbatch job.sh
Submitted batch job 123456
$ squeue --me
# verify that job gets started
$ srun --ntasks-per-node=1 --jobid 123456 nvidia-smi
Fri Nov 11 11:11:11 2021
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 495.29.05    Driver Version: 495.29.05    CUDA Version: 11.5     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:04:00.0 Off |                  N/A |
| 23%   25C    P8     8W / 250W |      1MiB / 11178MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  NVIDIA GeForce ...  On   | 00000000:08:00.0 Off |                  N/A |
| 23%   24C    P8     8W / 250W |      1MiB / 11178MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
```
Therewith the GPU core utilization and memory usage can be displayed for all GPU cards belonging to that job. 

Note that this is a one off presentation of the usage and the called `nvidia-smi` command runs within your allocation. The required resources for this job step should be minimal and should not noticably influence your job performance. 

## Further Information

CUDA: [https://developer.nvidia.com/cuda-zone](https://developer.nvidia.com/cuda-zone)  
CUDA C/C++ Basics: [http://www.nvidia.com/docs/IO/116711/sc11-cuda-c-basics.pdf](http://www.nvidia.com/docs/IO/116711/sc11-cuda-c-basics.pdf)  
Nvidia Geforce GTX 1080 Ti: [https://www.nvidia.com/en-us/geforce/products/10series/geforce-gtx-1080-ti](https://www.nvidia.com/en-us/geforce/products/10series/geforce-gtx-1080-ti/)  
Nvidia Geforce RTX 2080 Ti: [https://www.nvidia.com/de-de/geforce/graphics-cards/rtx-2080-ti/](https://www.nvidia.com/de-de/geforce/graphics-cards/rtx-2080-ti/)
Nvidia Geforce RTX 3090: [https://www.nvidia.com/de-de/geforce/graphics-cards/30-series/rtx-3090/](https://www.nvidia.com/de-de/geforce/graphics-cards/30-series/rtx-3090/)
Nvidia Tesla P100: [http://www.nvidia.com/object/tesla-p100.html](http://www.nvidia.com/object/tesla-p100.html)

