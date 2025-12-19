## Build own's Environment
### Conda
```shell
$ module load gcc/10.2.0 cuda/11.4 cudampi/openmpi-4.1.1 python/3.7.1 cmake/3.16.9
$ conda create -n my_pytorch  
$ source activate my_pytorch
(my_pytorch) $  


# If you want to install torch / 
# for Python3.11
pip install torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128

# for Python3.10
pip install torch==2.5.1 torchvision==0.20.1 --index-url https://download.pytorch.org/whl/cu118

# IsaacSim # Recommmand: Just use image!
pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com

## Install check
python - <<'PY'
import torch
print(torch.__version__)
print(torch.cuda.is_available())
if torch.cuda.is_available():
    print(torch.cuda.get_device_name(0))
PY

```
