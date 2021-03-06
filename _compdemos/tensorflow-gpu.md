---
title: Running Tensorflow (and Keras) on GPUs
last_modified_at: 2019-05-25
main_author: Dirk Petersen
primary_reviewers: fizwit, vortexing
---

Running tensorflow with GPUs has become easier in 2019. We can either use it on Gizmo as the latest GizmoJ class nodes are equipped with GPUs or on the [Koshu cluster](/compdemos/cluster_koshuBeta/) which is running in Google's cloud.

## GPU Tensorflow in a Python Environment

### GPU Tensorflow with the standard Python

load a current version of Python 3.6 on Rhino using the ml shortcut for module load and then use pip3 to install Tensorflow in your user home directory. 

```
    ~$ ml Python/3.6.7-foss-2016b-fh2
    ~$ pip3 install --user --upgrade tensorflow-gpu
``` 

then create a small python test script:

```
    echo "#! /usr/bin/env python3" > ~/tf-test.py
    echo "import tensorflow" >> ~/tf-test.py
    echo "from tensorflow.python.client import device_lib" >> ~/tf-test.py
    echo "print(tensorflow.__version__)" >> ~/tf-test.py
    echo "print(tensorflow.__path__)" >> ~/tf-test.py
    echo "print(device_lib.list_local_devices())" >> ~/tf-test.py
    chmod +x ~/tf-test.py
```

and run it on Gizmo with `--gres=gpu` to select GizmoJ nodes


``` 
    ~$ sbatch -p largenode -c 6 --mem 33000 -o out.txt --gres=gpu ~/tf-test.py
    ~$ tail -f out.txt
```

or you run it on Koshu which has slightly more powerful GPUs

```
    ~$ sbatch -M koshu -p gpu --gres=gpu -o out.txt ~/tf-test.py
    Submitted batch job 27074536 on cluster koshu
    ~$ tail -f out.txt
    1.12.0
    ['/home/petersen/.local/lib/python3.6/site-packages/tensorflow', '...']
    ...
    physical_device_desc: "device: 0, name: Tesla V100-SXM2-16GB, pci bus id: 0000:00:04.0, compute capability: 7.0"
```

if you want to switch back to the non-GPU version of Tensorflow just uninstall the GPU version you installed under .local 

```
    ~$ pip3 uninstall tensorflow-gpu
    Uninstalling tensorflow-gpu-1.13.1:
    Would remove:
        /home/petersen/.local/bin/freeze_graph
        /home/petersen/.local/bin/saved_model_cli
        /home/petersen/.local/bin/tensorboard
        /home/petersen/.local/bin/tf_upgrade_v2
        /home/petersen/.local/bin/tflite_convert
        /home/petersen/.local/bin/toco
        /home/petersen/.local/bin/toco_from_protos
        /home/petersen/.local/lib/python3.6/site-packages/tensorflow/*
        /home/petersen/.local/lib/python3.6/site-packages/tensorflow_gpu-1.13.1.dist-info/*
    Proceed (y/n)?
```

### GPU Tensorflow in a virtual environment

Python virtual environments are useful for advanced users who would like to work with multiple versions of python packages. It is important to understand that the virtual env is tied to the Python environment you have previously loaded using the `ml` command. Let's load a recent Python and create a virtual environment called `mypy`

```
    ~$ ml Python/3.6.7-foss-2016b-fh2
    ~$ python3 -m venv mypy
    ~$ source ./mypy/bin/activate
    (mypy) petersen@rhino3:~$ which pip3
    /home/petersen/mypy/bin/pip3
```

Now that you have our own environment you can install packages with pip3. Leave out the --user option in this case because you want to install the package under the virtual environment and not under ~/.local 

```
    (mypy) petersen@rhino3:~$ pip3 install --upgrade tensorflow-gpu
``` 

Now you can just continue with the example from `GPU Tensorflow with the standard Python`. After you are done with your virtual environment you can just run the `deactivate` script. No need to uninstall the tensorflow package:

```
    (mypy) petersen@rhino3:~$ deactivate 
    ~$ 
```


### GPU Tensorflow from a singularity container  

To run in a Singularity container, you need to start with a Docker image containing a modern Python and the _tensorflow-gpu_ package installed.  The [Tensorflow Docker images](https://hub.docker.com/r/tensorflow/tensorflow/) are all set up and ready.

After that load Singularity:

    ml Singularity

> Note that there is a _singularity_ module (note the case) that is
> out-of-date.  Loading _Singularity_ (with caps) will give you the most modern
> release which is typically what you want.
> 

After that, the only change is to enable NVIDIA support by adding the `--nv`
flag to `singularity exec`:

    singularity exec --nv docker://tensorflow/tensorflow:latest-gpu-py3 python3 ...

Sample code is available in the [slurm-examples repository](https://github.com/FredHutch/slurm-examples/tree/master/tensorflow-gpu).

## Tensorflow from R

Scientific Computing maintains custom builds of R and Python.
Python modules with fh suffixes have Tensorflow since version 3.6.1.
Only Python3 releases have the Tensorflow package. To use Tensorflow from
R, use the FredHutch Modules for R and Python.
Example Setup

```
    ml R
    ml Python/3.6.7-foss-2016b-fh2

    # Start R
    R
    # R commands
    pyroot = Sys.getenv("EBROOTPYTHON")
    pypath <- paste(pyroot, sep = "/", "bin/python")
    Sys.setenv(TENSORFLOW_PYTHON=pypath)
    library(tensorflow)
    tf_version()
    sess = tf$Session()
    hello <- tf$constant('Hello, TensorFlow!')
    sess$run(hello)
    # Sample output: b'Hello, TensorFlow!'
```

## Troubleshooting 

First verify that you have a GPU active (e.g. Tesla V100) as well as CUDA V 10.0 or newer

```
    petersen@koshuk0:~$ nvidia-smi 
    Thu Dec 27 14:44:19 2018       
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 410.79       Driver Version: 410.79       CUDA Version: 10.0     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla V100-SXM2...  Off  | 00000000:00:04.0 Off |                    0 |
    | N/A   39C    P0    34W / 300W |      0MiB / 16130MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
```

If you see any issues here you need to contact `SciComp` to have the error corrected 

Also please email `SciComp` to request further assistance 
