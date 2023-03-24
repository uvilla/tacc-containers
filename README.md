# FEniCS TACC Docker image

Here we build a Docker Image with a working FEniCS installation based upon the TACC `tacc-ubuntu18-mvapich2.3-ib` image.

Supported systems: `Frontera`, `Maverick2`.

Image is available on [DockerHub](https://hub.docker.com/r/uvilla/fenics-2019.1-tacc-mvapich2.3-ib).

For more information regarding containers at TACC see [here](https://containers-at-tacc.readthedocs.io/en/latest/containers/00.overview.html).

## Build the image (locally)

To build the image:
```
cd tacc-mvapich2.3-ib
docker build -t fenics-2019.1-tacc-mvapich2.3-ib .
```

To upload the image on DockerHub

```
docker login
docker tag fenics-2019.1-tacc-mvapich2.3-ib uvilla/fenics-2019.1-tacc-mvapich2.3-ib:latest
docker push uvilla/fenics-2019.1-tacc-mvapich2.3-ib:latest
```



## Run the image on your laptop using Docker

To run the container locally using docker, use the command
```
docker run -e MV2_SMP_USE_CMA=0 -e MV2_ENABLE_AFFINITY=0 -ti -v $(pwd):/home1/ uvilla/fenics-2019.1-tacc-mvapich2.3-ib:latest /bin/bash 
```
Your current directory will be mapped to `/home1` on the container. The enviromental variables 
`MV2_SMP_USE_CMA=0` and `MV2_ENABLE_AFFINITY=0` are needed when running local to avoid some nasty MPI errors.


## Run the image on TACC resources via Apptainer

Download the Docker image from DockerHub and save it in $SCRATCH
```
idev -N 1
cd $SCRATCH
module load mvapich2-x/2.3
module load tacc-apptainer/1.1.3
apptainer pull docker://uvilla/fenics-2019.1-tacc-mvapich2.3-ib:latest
```

You can test the container with
```
ibrun apptainer run $SCRATCH/fenics-2019.1-tacc-mvapich2.3-ib_latest.sif hellow
```

First time run of `hIPPYlib`. Before using hIPPYlib in parallel, the FEniCS jit compiler need to create new folders in your `$HOME/.cache` directory. To do so, you simply need to run `python -c "import hippylib"` in serial.

```
git clone https://github.com/hippylib/hippylib.git
cd hippylib
ibrun -n 1 apptainer run $SCRATCH/fenics-2019.1-tacc-mvapich2.3-ib_latest.sif python3 -c "import hippylib"
```

Now you can run the subsurface Poisson example in parallel
```
cd applications/poisson
ibrun apptainer run $SCRATCH/fenics-2019.1-tacc-mvapich2.3-ib_latest.sif python3 model_subsurf.py
```

For an example of (non-interactive) job submission see [hippylib-ex.slurm](hippylib-ex.slurm).
The script assumes that the apptainer contanaier is in `$SCRATCH/fenics-2019.1-tacc-mvapich2.3-ib_latest.sif` and `hIPPYlib` in `$SCRATCH/hippylib`.

```
sbatch hippylib-ex.slurm
```

