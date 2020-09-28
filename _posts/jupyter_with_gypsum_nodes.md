---
title: 'Running Jupyter notebooks on Gypsum'
date: 2020-09-10
permalink: /posts/jupyter_with_gypsum_nodes/
tags:
  - jupyter
  - gypsum
  - umass
---

# Running Jupyter notebooks on Gypsum

### What is Gypsum?

Gypsum is a large cluster of computers (nodes) belonging to University of Massachusetts Amherst, each of which contains between four and eight NVIDIA GPU cards. There are 75 nodes with four NVIDIA Titan-X GPUs, 25 nodes with four NVIDIA M40 GPUs, 53 nodes with eight NVIDIA GeForce GTX 1080 Ti GPUs and 25 nodes with eight NVIDIA GeForce RTX 2080 Ti GPUs. Gypsum was purchased with funds generously provided by the  [Massachusetts Technology Collaborative](https://masstech.org/) .

As specified on [Gypsum Cluster Documentation](https://gypsum-docs.cs.umass.edu), the Gypsum hardware cluster consists of:
* 25 compute nodes with 4 NVIDIA Tesla M40 GPUs, 12 cores (2 x Xeon E5-2620 v3 2.40 GHz), 256G of RAM, and 256G SSD for local disk.
* 75 compute nodes with 4 NVIDIA TITAN X GPUs, 12 cores (2 x Xeon E5-2620 v3 2.40 GHz), 256G of RAM, and 256G SSD for local disk.
* 53 compute nodes with 8 NVIDIA GeForce GTX 1080 Ti GPUs, 24 cores (2 x Xeon Silver 4116 2.10 GHz), 384G of RAM, and 256G SSD for local disk.
* 25 compute nodes with 8 NVIDIA GeForce RTX 2080 Ti GPUs, 24 cores (2 x Xeon Silver 4116 2.10 GHz), 384G of RAM, and 256G SSD for local disk.
* A 325T shared file system accessible by all of the compute nodes.
* A 325T backup system.

### Why use Jupyter initially instead of running scripts directly on gypsum?

Typically, for running machine learning experiments on the GPU cluster, it is often helpful to realise that we are making measurable progress. Depending on your setup, it is possible to visualize the preliminary experiments you are performing to ensure that the gpu intensive task you are about to run doesn’t go waste because of trivial errors in implementation. Jupyter notebook often comes handy at times for these initial experimentations. 
This setup will guide you through running Cuda enabled torch for experiments that you need to run on research nodes but can visualize them on your local machine with port forwarding.

Before we begin, one thing to keep in mind about research clusters is the concept of a login node vs a compute node. When you ssh into your cluster, you are immediately in a login node, which is where you do all your main file editing and manipulation. These nodes usually don’t have the memory required for intense compute jobs, which is where the compute nodes come in. You typically submit jobs via job schedulers like SLURM (UMass uses this) or PBS to those compute nodes.

### Steps:

1. Login to your remote user:
`ssh <user>@gypsum.cs.umass.edu`. Make sure that you have setup RSA keys or connected to UMass VPN before you login.
2. First you want to begin by installing anaconda distribution on your remote user.
Download the shell script using wget as (download latest):
`wget https://repo.anaconda.com/archive/Anaconda3-2020.07-Linux-x86_64.sh`
`bash Anaconda3-2020.07-Linux-x86_64.sh`
This will install the anaconda distribution and you should answer yes on a prompt to set the path variables automatically.
You can verify the installation by first running `source .bashrc` or restarting the shell window and then running python/conda /jupyter commands to check if the distribution installed correctly.
3. The next step is to create virtual environment to install specific dependencies your project might need. It’s often helpful to keep one common virtual environment which contains most of the packages to avoid wasting time for setting up environments each time you want to work on a new project.
`conda create -n local_env`
`conda create -n local_env python=3.8`
You can use the latter if you want to specify the python version you want to use.
Activate the environment by running, `conda activate local_env`.
Now, you can install dependencies in the current environment by activating the environment and running Conda install commands. Eg,
`conda install pytorch torchvision cudatoolkit=10.1 -c pytorch` will install CUDA enabled pytorch.
3. Before moving on to install ipython kernels in the virtual environment, it is important to check that your pip version points to the anaconda distribution by running `pip -V`. If it does not, you may need to deactivate your virtual environment and activate it again. It is important to do this to prevent changing packages on the system wide pip (If you still try to do, it will throw permission denied error).
`conda deactivate` deactivates the current environment. You can also remove the existing environment by using `conda env remove -n myenv`.
4. Now that you pip points to your anaconda distribution, install python kernel in your virtual environment:
First switch to your virtual environment, `conda activate local_env`
Then run `pip install --user ipykernel`. It will also print the directory in which this kernel was installed.
To check the kernel list, you can print it using `jupyter kernelspec list`.
5. (Skip this step for now and come back later after port-forwarding step. I have put this here for now because it is more in flow with the previous steps.)
Now, to make this kernel available in your jupyter notebook UI, run
`python -m ipykernel install --user --name local_env --display-name "Local GPU env"`.
You should now see this kernel in your Jupyter notebook under Kernel/Change kernel in the toolbar.
6. Using port forwarding:
You can run Tmux sessions for running jupyter notebooks but I will not use them in this tutorial. If you want to use GPU (which should be the reason you are using Gypsum. If not, then please use Swarm cluster to run CPU-only jobs), then you need to use slurm commands to allocate resources. 
I won’t put all the slurm queues here (you can find them in the Gypsum documentation) but you should use titanx-short if you are using short timed GPU intensive notebooks. Run these commands on remote user,
`srun --pty -p titanx-short --gres=gpu:1 --mem=4000MB /bin/bash`
`export XDG_RUNTIME_DIR=""`
Allocate the queue, # gpu, memory as per you needs but be careful not to keep the job idle for long as this could prevent other users from using the resources if the queue is full.

Port forwarding step:

`ssh -R <port1>:localhost:<port2> gypsum.cs.umass.edu -f -N`
`jupyter notebook --port <port2> --no-browser`

On you /local machine/, run
`ssh -L  <port1>:localhost:<port1> <user>@gypsum.cs.umass.edu -f -N`

Then go to you browser and enter the url as `localhost:<port1>`. You will be prompted to put the token or password. Scroll down and set a password to avoid putting tokens on every login. 

Finally, go back to step 5, and change your kernel. 
You should now be able to run GPU from remote node on the jupyter notebook running in your browser. I found this to be quite helpful as you can upload the files in jupyter using it’s UI instead of doing an scp. 

### Misc:

Side note when installing CUDA enabled pytorch:
Use 	`10.1` version of CUDA as the NVIDIA drivers for `10.2` version are not yet installed on Gypsum (as of 28 Sept 2020). 
