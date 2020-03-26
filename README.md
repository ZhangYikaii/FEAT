# Few-Shot Learning via Embedding Adaptation with Set-to-Set Functions
The code repository for "[Few-Shot Learning via Embedding Adaptation with Set-to-Set Functions](https://arxiv.org/abs/1812.03664)" (Accepted by CVPR 2020) in PyTorch.

## Few-Shot Embedding Adaptation with Transformer

Learning with limited data is a key challenge for visual recognition. Many few-shot learning methods address this challenge by learning an instance embedding function from seen classes and apply the function to instances from unseen classes with limited labels. This style of transfer learning is task-agnostic: the embedding function is not learned optimally discriminative with respect to the unseen classes, where discerning among them leads to the target task. In this paper, we propose a novel approach to adapt the instance embeddings to the target classification task with a #set-to-set# function, yielding embeddings that are task-specific and are discriminative. We empirically investigated various instantiations of such set-to-set functions and observed the Transformer is most effective --- as it naturally satisfies key properties of our desired model. We denote this model as FEAT (few-shot embedding adaptation w/ Transformer) and validate it on both the standard few-shot classification benchmark and four extended few-shot learning settings with essential use cases, i.e., cross-domain, transductive, generalized few-shot learning, and low-shot learning. It archived consistent improvements over baseline models as well as previous methods, and established the new state-of-the-art results on two benchmarks. 

![Few-Shot Learning via Transformer](imgs/teaser.PNG)

![architecture compare](imgs/architecture.png)

### Prerequisites

The following packages are required to run the scripts:

- [PyTorch-0.4 and torchvision](https://pytorch.org)

- Package [tensorboardX](https://github.com/lanpa/tensorboardX)

- Dataset: please download dataset and put images into the folder data/[name of dataset, miniimagenet or cub]/images

- Pre-trained weights: please download the [pre-trained weights](https://drive.google.com/open?id=14Jn1t9JxH-CxjfWy4JmVpCxkC9cDqqfE) of the encoder if needed. The pre-trained weights can be downloaded by the script [download_weight.sh](download_weight.sh)

### Dataset

#### MiniImageNet Dataset

The MiniImageNet dataset is a subset of the ImageNet that includes a total number of 100 classes and 600 examples per class. We follow the [previous setup](https://github.com/twitter/meta-learning-lstm), and use 64 classes as SEEN categories, 16 and 20 as two sets of UNSEEN categories for model validation and evaluation respectively.

#### CUB Dataset
[Caltech-UCSD Birds (CUB) 200-2011 dataset](http://www.vision.caltech.edu/visipedia/CUB-200-2011.html) is initially designed for fine-grained classification. It contains in total 11,788 images of birds over 200 species. On CUB, we randomly sampled 100 species as SEEN classes, another two 50 species are used as two UNSEEN sets. We crop all images with given bounding box before training. We only test CUB with ConvNet backbone in our work.

#### TieredImageNet Dataset
[TieredImageNet](https://github.com/renmengye/few-shot-ssl-public) is a large-scale dataset  with more categories, which contains 351, 97, and 160 categoriesfor model training, validation, and evaluation, respectively. The dataset can also be download from [here](https://github.com/kjunelee/MetaOptNet).
We only test TieredImageNet with ResNet backbone in our work.

Check [this](https://github.com/Sha-Lab/FEAT/blob/master/data/README.md) for details of data downloading and preprocessing.

### Code Structures
To reproduce our experiments with FEAT, please use **train_fsl.py**. There are four parts in the code.
 - `model`: It contains the main files of the code, including the few-shot learning trainer, the dataloader, the network architectures, and baseline and comparison models.
 - `data`: Images and splits for the data sets.
 - `saves`: The pre-trained weights of different networks.
 - `checkpoints`: To save the trained models.

### Model Training and Evaluation
Please use **train_fsl.py** and follow the instructions blow. FEAT meta-learns the embedding adaptation process such that all the training instance embeddings in a task is adapted, based on their contextual task information, using Transformer. The file will automatically evaluate the model on the meta-test set with 10,000 tasks after given epoches.

#### Arguments
The train_fsl.py takes the following command line options (details are in the `model/utils.py`):

## Task Related Arguments ## 
- `dataset`: Option for the dataset (`MiniImageNet`, `TieredImageNet`, or `CUB`), default to `MiniImageNet`

- `way`: The number of classes in a few-shot task during meta-training, default to `5`

- `eval_way`: The number of classes in a few-shot task during meta-test, default to `5`

- `shot`: Number of instances in each class in a few-shot task during meta-training, default to `1`

- `eval_shot`: Number of instances in each class in a few-shot task during meta-test, default to `1`

- `query`: Number of instances in each class to evaluate the performance during meta-training, default to `15`

- `eval_query`: Number of instances in each class to evaluate the performance during meta-test, default to `15`

## Optimization Related Arguments ## 
- `max_epoch`: The maximum number of training epochs, default to `200`

- `episodes_per_epoch`: The number of tasks sampled in each epoch, default to `100`

- `num_eval_episodes`: The number of tasks sampled from the meta-val set to evaluate the performance of the model (note that we fix sampling 10,000 tasks from the meta-test set during final evaluation), default to `200`

- `lr`: Learning rate for the model, default to `0.0001` with pre-trained weights

- `lr_mul`: This is specially designed for set-to-set functions like FEAT. The learning rate for the top layer will be multiplied by this value (usually with faster learning rate). Default to `10`

- `lr_scheduler`: The scheduler to set the learning rate (`step`, `multistep`, or `cosine`), default to `step`

- `step_size`: The step scheduler to decrease the learning rate. Set it to a single value if choose the `step` scheduler, and provide multiple values when choose the `multistep` scheduler. Default to `20`

- `gamma`: Learning rate ratio for `step` or `multistep` scheduler, default to `0.2`

- `fix_BN`: Set the encoder to the evaluation mode during the meta-training. This parameter is useful when meta-learning with the WRN. Default to `False`

- `augment`: Whether to do data augmentation or not during meta-training, default to `False`

- `mom`: The momentum value for the SGD optimizer, default to `0.9`

- `weight_decay`: The weight_decay value for SGD optimizer, default to `0.0005`

## Model Related Arguments ## 
- `model_class`: The model to use during meta-learning. We provide implementations for baselines (`MatchNet` and `ProtoNet`), set-to-set functions (`BILSTM`, `DeepSet`, `GCN`, and our `FEAT`). We also include an instance-specific embedding adaptation approach `FEAT`, which is discussed in the old version of the paper. Default to `FEAT`

- `use_euclidean`: Use the euclidean distance or the cosine similarity to compute pairwise distances. We use the euclidean distance in the paper. Default to `False`

- `backbone_class`: Types of the encoder, i.e., the convolution network (`ConvNet`), ResNet-12 (`Res12`), or Wide ResNet (`WRN`), default to `ConvNet`

- `balance`: This is the balance weight for the contrastive regularizer. Default to `0`

- `temperature`: Temperature over the logits, we #divide# logits with this value. It is useful when meta-learning with pre-trained weights. Default to `1`

- `temperature2`: Temperature over the logits in the regularizer, we divide logits with this value. This is specially designed for the contrastive regularizer. Default to `1`

## Other Arguments ## 
- `orig_imsize`: Whether to resize the images before loading the data into the memory. `-1` means we do not resize the images and do not read all images into the memory. Default to `-1`

- `multi_gpu`: Whether to use multiple gpus during meta-training, default to `False`

- `gpu`: The index of GPU to use. Please provide multiple indexes if choose `multi_gpu`. Default to `0`

- `log_interval`: How often to log the meta-training information, default to every `50` tasks

- `eval_interval`: How often to validate the model over the meta-val set, default to every `1` epoch

- `save_dir`: The path to save the learned models, default to `./checkpoints`

Running the command without arguments will train the models with the default hyperparamters values. Loss changes will be recorded as a tensorboard file.

#### FEAT Approach

For example, to train the 1-shot/5-shot 5-way FEAT model with ConvNet backbone on MiniImageNet:

    $ python train_fsl.py  --max_epoch 200 --model_class FEAT --use_euclidean --backbone_class ConvNet --dataset MiniImageNet --way 5 --eval_way 5 --shot 1 --eval_shot 1 --query 15 --eval_query 15 --balance 1 --temperature 64 --temperature2 16 --lr 0.0001 --lr_mul 10 --lr_scheduler step --step_size 20 --gamma 0.5 --gpu 8 --init_weights ./saves/initialization/miniimagenet/con-pre.pth --eval_interval 1
	$ python train_fsl.py  --max_epoch 200 --model_class FEAT --use_euclidean --backbone_class ConvNet --dataset MiniImageNet --way 5 --eval_way 5 --shot 5 --eval_shot 5 --query 15 --eval_query 15 --balance 0.1 --temperature 32 --temperature2 64 --lr 0.0001 --lr_mul 10 --lr_scheduler step --step_size 20 --gamma 0.5 --gpu 14 --init_weights ./saves/initialization/miniimagenet/con-pre.pth --eval_interval 1

to train the 1-shot/5-shot 5-way FEAT model with ResNet-12 backbone on MiniImageNet:

    $ python train_fsl.py  --max_epoch 200 --model_class FEAT  --backbone_class Res12 --dataset MiniImageNet --way 5 --eval_way 5 --shot 1 --eval_shot 1 --query 15 --eval_query 15 --balance 0.01 --temperature 64 --temperature2 64 --lr 0.0002 --lr_mul 10 --lr_scheduler step --step_size 40 --gamma 0.5 --gpu 1 --init_weights ./saves/initialization/miniimagenet/Res12-pre.pth --eval_interval 1 --use_euclidean
	$ python train_fsl.py  --max_epoch 200 --model_class FEAT  --backbone_class Res12 --dataset MiniImageNet --way 5 --eval_way 5 --shot 5 --eval_shot 5 --query 15 --eval_query 15 --balance 0.1 --temperature 64 --temperature2 32 --lr 0.0002 --lr_mul 10 --lr_scheduler step --step_size 40 --gamma 0.5 --gpu 0 --init_weights ./saves/initialization/miniimagenet/Res12-pre.pth --eval_interval 1 --use_euclidean

to train the 1-shot/5-shot 5-way FEAT model with ResNet-12 backbone on TieredImageNet:

	$ python train_fsl.py  --max_epoch 200 --model_class FEAT  --backbone_class Res12 --dataset TieredImageNet --way 5 --eval_way 5 --shot 1 --eval_shot 1 --query 15 --eval_query 15 --balance 0.1 --temperature 64 --temperature2 64 --lr 0.0002 --lr_mul 10 --lr_scheduler step --step_size 20 --gamma 0.5 --gpu 0 --init_weights ./saves/initialization/tieredimagenet/Res12-pre.pth --eval_interval 1  --use_euclidean
	$ python train_fsl.py  --max_epoch 200 --model_class FEAT  --backbone_class Res12 --dataset TieredImageNet --way 5 --eval_way 5 --shot 5 --eval_shot 5 --query 15 --eval_query 15 --balance 0.1 --temperature 32 --temperature2 64 --lr 0.0002 --lr_mul 10 --lr_scheduler step --step_size 40 --gamma 0.5 --gpu 0 --init_weights ./saves/initialization/tieredimagenet/Res12-pre.pth --eval_interval 1  --use_euclidean

#### Results

Results on the MiniImageNet:
|  Setups  | 1-Shot 5-Way | 1-Shot 5-Way | 5-Shot 5-Way | 5-Shot 5-Way |
|:--------:|:------------:|:------------:|:------------:|:------------:|
| Backbone |    ConvNet   |    ResNet    |    ConvNet   |    ResNet    |
|:--------:|:------------:|:------------:|:------------:|:------------:|
| ProtoNet |     52.61    |     62.39    |     71.33    |     80.53    |
|  BILSTM  |     52.13    |     63.90    |     69.15    |     80.63    |
| DEEPSETS |     54.41    |     64.14    |     70.96    |     80.93    |
|    GCN   |     53.25    |     64.50    |     70.59    |     81.65    |
|:--------:|:------------:|:------------:|:------------:|:------------:|
|   FEAT   |     55.15    |     66.78    |     71.61    |     82.05    |

Results on the TieredImageNet with ResNet-12 backbone:
|  Setups  | 1-Shot 5-Way | 5-Shot 5-Way |
|:--------:|:------------:|:------------:|
| ProtoNet |     68.23    |     84.03    |
|  BILSTM  |     68.14    |     84.23    |
| DEEPSETS |     68.59    |     84.36    |
|    GCN   |     68.20    |     84.64    |
|:--------:|:------------:|:------------:|
|   FEAT   |     70.80    |     84.79    |

## .bib citation
If this repo helps in your work, please cite the following paper:

    @inproceedings{ye2020fewshot,
      author    = {Han-Jia Ye and
                   Hexiang Hu and
                   De-Chuan Zhan and
                   Fei Sha},
      title     = {Few-Shot Learning via Embedding Adaptation with Set-to-Set Functions},
      booktitle = {Computer Vision and Pattern Recognition (CVPR)},
      year      = {2020}
    }


## Acknowledgment
We thank following repos providing helpful components/functions in our work.
- [ProtoNet](https://github.com/cyvius96/prototypical-network-pytorch)

- [MatchingNet](https://github.com/gitabcworld/MatchingNetworks)

- [PFA](https://github.com/joe-siyuan-qiao/FewShot-CVPR/)

- [Transformer](https://github.com/jadore801120/attention-is-all-you-need-pytorch)

- [MetaOptNet](https://github.com/kjunelee/MetaOptNet/)

