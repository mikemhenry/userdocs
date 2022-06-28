# Singularity

## Description

Put your scientific workflows, software and libraries in a Singularity container and run it on UBELIX

## Examples

### Work interactively

Submit an interactive lsf job and then use the shell command to spawn an interactive shell within the Singularity container:

```Bash
srun --time=01:00:00 --mem-per-cpu=2G --pty bash
singularity shell <image>
```

### Execute the containers "runscript"

```Bash
#!/bin/bash
#SBATCH --partition=all
#SBATCH --mem-per-cpu=2G

singularity run <image>   #or ./<image>
```

### Run a command within your container image

```Bash
singularity exec <image> <command>

e.g:
singularity exec container.img cat /etc/os-release
```

### Bind directories

Per default the started application (e.g. `cat` in the last example) runs withing the container. The container works like a seperate machine with own operation system etc. Thus, per default you have no access to files and directories outside the container. This can be changed using binding paths. 

If files are needed outside the container, e.g. in your HOME you can add the a path to `SINGULARITY_BINDPATH="src1[:dest1],src2[:dest2]`. All subdirectories and files will be accessible. Thus you could bind your HOME directory as:

```Bash
export SINGULARITY_BINDPATH="$HOME/:$HOME/"   
# or simply 
export SINGULARITY_BINDPATH="$HOME"
```

## Further Information

Official Singularity Documentation can be found at [https://sylabs.io/docs/](https://sylabs.io/docs/)
