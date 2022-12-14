<div align="center">

# Assignment 4 - Deployment for Demos



<a href="https://pytorch.org/get-started/locally/"><img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-ee4c2c?logo=pytorch&logoColor=white"></a>
<a href="https://pytorchlightning.ai/"><img alt="Lightning" src="https://img.shields.io/badge/-Lightning-792ee5?logo=pytorchlightning&logoColor=white"></a>
<a href="https://hydra.cc/"><img alt="Config: Hydra" src="https://img.shields.io/badge/Config-Hydra-89b8cd"></a>
<a href="https://github.com/ashleve/lightning-hydra-template"><img alt="Template" src="https://img.shields.io/badge/-Lightning--Hydra--Template-017F2F?style=flat&logo=github&labelColor=gray"></a><br>
[![Paper](http://img.shields.io/badge/paper-arxiv.1001.2234-B31B1B.svg)](https://www.nature.com/articles/nature14539)
[![Conference](http://img.shields.io/badge/AnyConference-year-4b44ce.svg)](https://papers.nips.cc/paper/2020)

</div>

## Description

We build cifar10 timm module, convert it into script and trace model. Upload docker image to dockerhub.


## How to run

Install dependencies

```bash
# clone project
git clone https://github.com/sushant097/TSAI-Assignment4-Deployment-for-Demos
cd TSAI-Assignment4-Deployment-for-Demos

# [OPTIONAL] create conda environment
conda create -n myenv python=3.9
conda activate myenv

# install pytorch according to instructions
# https://pytorch.org/get-started/

# install requirements
pip install -r requirements.txt
```

Train model with default configuration

```bash
# train on CPU
python src/train.py trainer=cpu

# train on GPU
python src/train.py trainer=gpu
```

Train model with chosen experiment configuration from [configs/experiment/](configs/experiment/)

```bash
python src/train.py experiment=experiment_name.yaml
```

You can override any parameter from command line like this

```bash
python src/train.py trainer.max_epochs=20 datamodule.batch_size=64
```

### Assignment Related
**To Train the Cifar10 Torch Script**

`python src/train_script.py experiment=cifar`

It saves the `model.script.pt` model inside `logs/train/runs/*`

Using best hyperparmeters searched on Assignment 3

```bash
best_params:
  model.optimizer._target_: torch.optim.SGD
  model.optimizer.lr: 0.03584594526879088
  datamodule.batch_size: 128
best_value: 0.8082000017166138
```

**To train the timm resnet18 model**
` python src/train_script.py experiment=example_timm trainer=gpu`

It exports and saves the trained resnet18 timm as scripted model and best torch model.

**Run the Tensorboard**
`tensorboard --logdir=logs/train/runs --bind_all`

##### Pack the inference scripted model inside the dockerize folder.

**Build the docker for only prediction**

`make build` or `docker build -t sushant097/my_repo:deploy dockerize/`


**Pull from the DockerHub**
`docker pull sushant097/my_repo:deploy`

**RUN the docker container**
`docker run -t -p 8080:8080 sushant097/my_repo:deploy`

**The uncompressed image size is: 1.14GB**

**My Dockerfile is:**
```dockerfile
FROM python:3.9.7-slim-buster

WORKDIR src

COPY requirements.txt requirements.txt


RUN pip install torch==1.11.0+cpu torchvision==0.12.0+cpu -f https://download.pytorch.org/whl/torch_stable.html



Run pip3 install -r requirements.txt \
    && rm -rf /root/.cache/pip

COPY ./logs/train/runs/2022-09-29_07-05-14/model.script.pt model.script.pt

COPY src/* src/

EXPOSE 8080

ENTRYPOINT ["python", "src/demo_scripted.py"]
```

### Push model, logs and data to google drive (using dvc)
1. Untrack logs/data from git : `git rm -r --cached logs` and `git rm -r --cached data`
2. Add logs to dvc: `dvc add logs` and `dvc add data`
2.1. `git add . ` and `dvc config core.autostage true` : As logs folder from being tracked by git and then let dvc take care of it
3. Add a remote: `dvc remote add gdrive gdrive://1nqYL96VXmfecRur899q5YmgQ3IOnk0m7`
4. Push logs and other tracked files by dvc in gdrive: `dvc push -r gdrive`
5. Now, whenever logs/data is deleted then, we can directly pull logs from dvc as: `dvc pull -r gdrive`

### Tensorboard 
`tensorboard --logdir logs/train --bind_all`

### Tensorboard Dev 
```bash
tensorboard dev upload --logdir logs \
    --name "My Cifar10 TSAI Assignment4 Deployment for Demos experiment" \
    --description "Visualization of Deployment"
```
**My Tensorboard logs at: https://tensorboard.dev/experiment/xORFVXtbRluBJ6CVN3GgjA/**


#### Packing Traced Model

**To train the timm resnet18 model**
`python src/train_trace.py experiment=example_timm trainer=gpu`