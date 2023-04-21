# Execution
There are two modes of execution: `interactif` and `batch`. 

### 1. Interactif

This mode is made to: 

- deal with the different files
- deal with the accounts
- execute code with `srun`

To run it in an interactive mode, see [http://www.idris.fr/jean-zay/gpu/jean-zay-gpu-exec_interactif.html](http://www.idris.fr/jean-zay/gpu/jean-zay-gpu-exec_interactif.html). 
### TODO : INTERACTIF MODE

### 2. Batch

This mode enables to: 

- launch a command and exit the session
- overpass the limit of times of the interactive mode

The batch mode can be used with the **Slurm** software.



### Example with a python code

We will see an example of how to run a code on gpu in batch mode.

1. Connect to jean zay, go to working directory and create an example directory.

```bash
$ ssh jean_zay
$ cd $WORK
$ mkdir example
```

2. Import the example code. Let’s say we have the following python code that shows torch tensors in `[test.py](http://test.py)` file.

```python
import torch
import torch.nn as nn
device = torch.device("cuda")
inputs = torch.randn(10).to(device)
model = nn.Linear(10, 2).to(device)
outputs = model(inputs)
print(f"OUTPUT : {outputs}")
```

3. As this is a pytorch script, we will look at the pytorch modules: 

```bash
$ module avail pytorch
--------------------------------------------------- /gpfslocalsup/pub/modules-idris-env4/modulefiles/linux-rhel8-skylake_avx512 ----------------------------------------------------
pytorch-cpu/py3/1.4.0             pytorch-gpu/py3/1.3.1             pytorch-gpu/py3/1.7.0                        pytorch-gpu/py3/1.8.1   pytorch-gpu/py3/1.13.0  
pytorch-cpu/py3/1.7.1             pytorch-gpu/py3/1.3.1+nccl-2.5.6  pytorch-gpu/py3/1.7.0+hvd-0.21.0             pytorch-gpu/py3/1.9.0   pytorch-gpu/py3/2.0.0   
pytorch-gpu/py3/1.1               pytorch-gpu/py3/1.4.0             pytorch-gpu/py3/1.7.1                        pytorch-gpu/py3/1.10.0  
pytorch-gpu/py3/1.2.0             pytorch-gpu/py3/1.5.0             pytorch-gpu/py3/1.7.1+nccl-2.8.3-1           pytorch-gpu/py3/1.10.1  
pytorch-gpu/py3/1.3               pytorch-gpu/py3/1.5.1             pytorch-gpu/py3/1.7.1-cellpose-cellprofiler  pytorch-gpu/py3/1.11.0  
pytorch-gpu/py3/1.3.0+nccl-2.5.6  pytorch-gpu/py3/1.6.0             pytorch-gpu/py3/1.8.0                        pytorch-gpu/py3/1.12.1
```

We will choose the `pytorch-gpu/py3/2.0.0` module.

4. Create a slurm script in `launch_script.slurm`: 

```bash
#!/bin/bash
#SBATCH --job-name=example_code          # nom du job
#SBATCH --nodes=1                    # on demande un noeud
#SBATCH --ntasks-per-node=1          # avec une tache par noeud (= nombre de GPU ici)
#SBATCH --gres=gpu:1                 # nombre de GPU par noeud (max 8 avec gpu_p2, gpu_p4, gpu_p5)
#SBATCH --cpus-per-task=10           # nombre de CPU par tache (1/4 des CPU du noeud 4-GPU)
#SBATCH --hint=nomultithread         # hyperthreading desactive
#SBATCH --time=00:01:00              # temps maximum d'execution demande (HH:MM:SS)
#SBATCH --output=example%j.out      # nom du fichier de sortie
#SBATCH --error=example%j.out       # nom du fichier d'erreur (ici commun avec la sortie)
 
# Nettoyage des modules charges en interactif et herites par defaut
module purge
  
# Chargement des modules
module load pytorch-gpu/py3/2.0.0
   
# Echo des commandes lancees
set -x
    
python -u test.py
```

5. Launch the script: 

```bash
$ sbatch launch_script.slurm
Submitted batch job 999439
```

6. We can now see the queue using: 

```bash
$ squeue --user=<ulg77ts>
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   999439   gpu_p13 example_  ulg77ts  R       0:08      1 r10i2n7
```

7. Once the code has ended, we can look at the log file `example999439.out` file: 

```bash
$ cat example999439.out
Loading pytorch-gpu/py3/2.0.0
Loading requirement: cuda/11.7.1 nccl/2.12.12-1-cuda cudnn/8.5.0.96-11.7-cuda
   gcc/8.5.0 openmpi/4.1.1-cuda intel-mkl/2020.4 magma/2.7.0-cuda sox/14.4.2
       sparsehash/2.0.3 libjpeg-turbo/2.1.3
+ python -u test.py
OUTPUT : tensor([0.1613, 0.8026], device='cuda:0', grad_fn=<AddBackward0>)
```

It has loaded the module and launched the command. The output is the tensor as expected.

8. Now let’s say we want to import a python package that is not available in the module. For the example we will import a logger : `loguru`. 

We can do it using either pip or conda. We will use pip. You need to create a symoblic link in order to install the libraries in the `$WORK` directory, and not `$HOME` (done by default): 

```bash
$ mkdir $WORK/.local
$ ln -s $WORK/.local $HOME
```

Now we can load the module: 

```bash
$ module load pytorch-gpu/py3/2.0.0
Loading pytorch-gpu/py3/2.0.0
Loading requirement: cuda/11.7.1 nccl/2.12.12-1-cuda cudnn/8.5.0.96-11.7-cuda gcc/8.5.0 openmpi/4.1.1-cuda intel-mkl/2020.4 magma/2.7.0-cuda sox/14.4.2 sparsehash/2.0.3
   libjpeg-turbo/2.1.3
```

Then, we can install the library: 

```bash
$ pip install --user --no-cache-dir loguru
```

We can modify the python file to test the importation: 

```python
import torch
import torch.nn as nn
from loguru import logger
logger.info("LOGURU TEST")
device = torch.device("cuda")
inputs = torch.randn(10).to(device)
model = nn.Linear(10, 2).to(device)
outputs = model(inputs)
print(f"OUTPUT : {outputs}")
```

We can add it to the slurm config file: 

```bash
#!/bin/bash
#SBATCH --job-name=example_code          # nom du job
#SBATCH --nodes=1                    # on demande un noeud
#SBATCH --ntasks-per-node=1          # avec une tache par noeud (= nombre de GPU ici)
#SBATCH --gres=gpu:1                 # nombre de GPU par noeud (max 8 avec gpu_p2, gpu_p4, gpu_p5)
#SBATCH --cpus-per-task=10           # nombre de CPU par tache (1/4 des CPU du noeud 4-GPU)
#SBATCH --hint=nomultithread         # hyperthreading desactive
#SBATCH --time=00:01:00              # temps maximum d'execution demande (HH:MM:SS)
#SBATCH --output=example%j.out      # nom du fichier de sortie
#SBATCH --error=example%j.out       # nom du fichier d'erreur (ici commun avec la sortie)
                        
# Nettoyage des modules charges en interactif et herites par defaut
module purge
                         
# Chargement des modules
module load pytorch-gpu/py3/2.0.0
pip install --user --no-cache-dir loguru
                          
# Echo des commandes lancees
set -x

python -u test.py
```

And launch the command: 

```bash
$ sbatch launch_script.slurm
```

The result would show: 

```bash
$ cat output999700.out
Loading pytorch-gpu/py3/2.0.0
 Loading requirement: cuda/11.7.1 nccl/2.12.12-1-cuda cudnn/8.5.0.96-11.7-cuda
     gcc/8.5.0 openmpi/4.1.1-cuda intel-mkl/2020.4 magma/2.7.0-cuda sox/14.4.2
         sparsehash/2.0.3 libjpeg-turbo/2.1.3
+ python -u test.py
2023-04-21 17:42:53.328 | INFO     | __main__:<module>:4 - LOGURU TEST
OUTPUT : tensor([-0.9672,  0.8543], device='cuda:0', grad_fn=<AddBackward0>)
```

We see that the `loguru` has been imported and the code ran with success ! 

## Sources: 

- [Mono GPU slurm script](http://www.idris.fr/jean-zay/gpu/jean-zay-gpu-exec_mono_batch.html)
- [Own python environment](http://www.idris.fr/jean-zay/gpu/jean-zay-gpu-python-env.html)
