# SLURM partition and QOS

## Description
UBELIX provides different CPU and GPU architectures. Furthermore, we provide job different queues for different job priorities and limitations. 

!!! note "restructuring 04.05.2021"
    With the maintainace 04.05.2021 we restructured the Slurm partition to:

    - more effcient resource usage: All users get access to all resources. Investors have priviledeged access
    - general access to GPUs without preempting
    - resource sharing (fair share) based on research groups using Workspaces, instead of institute based sharing 
    - simpler partition design

There are three main control mechanisms to specify queues and resources: 

- [**Partitions**](#partitions) and 
- [Quality of Service (**QoS**)](#qos)
- `--gres` to select the targeted GPU architecture, see [GPUs](gpus.md)

## Partitions
There are the current 3 partitions, epyc2 is default:

| Partition | job type | CPU / GPU | node / GPU memory | local Scratch |
| --------- | -------- | ---------- | ---------------- | ------------- |
|**epyc2** | single and multi-core |AMD Epyc2 2x64 cores | 1TB | 1TB |
| bdw | full nodes only (x*20cores) | Intel Broadwell 2x10 cores | 156GB | 1TB |
| gpu | GPU <br> (8 GPUs per node, <br> varying CPUs) | Nvidia GTX 1080 Ti <br> Nvidia RTX 2080 Ti <br> Nvidia RTX 3090 <br> Nvidia Tesla P100  | 11GB <br> 11GB <br> 12GB <br> 24GB | 800GB <br> 2x960GB <br> 1.92TB <br> 800GB  |

The **current usage** can be listed on the [UBELIX status page](https://www.ubelix.unibe.ch/)

## QoS
Within these partitions, **QoS** are used to distinguish different job limits. In each partition there is a default QoS (**bold** below). Each QoS has specific limits:

| Partition | QoS | time limit | cores/node/GPU limit | max jobs |
| --------- | --- | ---------- | ---------------- | ------- |
| **epyc2** | **job_epyc2** | 4 days | 512 cores | array jobs up to 10000 tasks |
| | job_epyc2_debug | 20 min | 20 cores | 1 |
| | job_epyc2_long | 15 days | 64 cores | 50 |
| | job_epyc2_short | 6h | 10 nodes | 50 | 
| bdw | **job_bdw** | 24 h | 40 nodes | 300 |
| | job_bdw_debug | 20 min | 2 nodes | 1 |
| | job_bdw_short | 6 h | 2 nodes | 10 |
| gpu | **job_gpu** | 24 h | 6x GTX 1080 Ti <br> 2x RTX 2080 Ti <br> 1x RTX 3090 <br> 1x Tesla P100 | 10 [^jobLim] <br> 4 CPU cores per requested GPU |
| | job_gpu_debug | 20 min | 1 GPU | 1 <br> 4 CPU cores per requested GPU |
| | job_gpu_preempt | 24 h | 12x GTX 1080 Ti <br> 4x RTX 2080 Ti <br> 4x RTX 3090 <br> 4x Tesla P100 | 24 [^jobLim] <br> 4 CPU cores per requested GPU |
 
[^jobLim]: The gpu job limits are determined by the maximum of single GPU jobs on all architecture. But keep in mind, that jobs should be submitted to the best suited architecture.

The QoS job_epyc2_short and job_gpu_preempt have access extended resources. In case of job_gpu_preempt, these jobs will be pre-empted if not enough resources for the investors. 

Thus a job can be submitted to the gpu partition using a RTX3090 and allow pre-emption:

```Bash
sbatch --partition=gpu --qos=job_gpu_preempt --gres=gpu:rtx3090 myjob.sh
```

Please see for more details: [GPUs](gpus.md)

## Default and Investor Partition
All resources are tried to privide in a most accessible way, preventing idle time. Thus the resouces meant for investors can be used by non-investing users too. 
Therefore, from the whole resources a certain amount of CPUs/GPUs are "reserved" in the investor partition. But if not used, jobs with `job_gpu_preempt` or `job_{bdw|epyc2}_short` can run on these resources. These are floating resources and do not bind to specific hardware, only certain amount per hardware type.

## Investor QoS
Investors get elvated priviledges on certain resources. 
The membership to these investor priviledges will/are managed on basis of HPC Workspaces, see [Workspace Management](../hpc-workspaces/management.md#investor-qos).

To utilize the invested resources, members need to specify one of the following investor partitions:

- `epyc2-invest`, 
- `bdw-invest`, 
- `gpu-invest`

As an example, the members of an GPU investor, submit jobs with:
```Bash
module load Workspace         # use the Workspace account
sbatch --partition=gpu-invest job.sh
```

!!! note ""
    previous `empi` investors now use `bdw-invest` partition and GPU investors use `gpu-invest`

### Technical Details:
Within the investor partition, *investor QoS* are specified. These have the format `job_<partitionBase>_<investorID>`, e.g. "job_bdw_aschauer". 
These QoS will be default for the members in the investor partition. And therewith do not need to be specified. 
You can check your investor QoS and the used partitions using:
```Bash
sacctmgr show assoc where user=$USER format=user%20,account%20,partition%16,qos%40,defaultqos%20
```




