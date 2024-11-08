# 部署 MiniGPT-V

## 报错1

name ‘cuda_setup’ is not defined

解决方法

```bash
pip install -U bitsandbytes
```

参考：https://blog.csdn.net/dzysunshine/article/details/130491896 an   https://github.com/Vision-CAIR/MiniGPT-4/issues/117

## 报错2

torch.cuda.OutOfMemoryError: CUDA out of memory. Tried to allocate 136.00 MiB (GPU 0; 23.68 GiB total capacity; 2.49 GiB already allocated; 63.75 MiB free; 2.67 GiB reserved in total by PyTorch) If reserved memory is >> allocated memory try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF

尝试1

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 python demo.py --cfg-path eval_configs/minigpt4_eval.yaml  --gpu-id 0
```

失败：torch.cuda.OutOfMemoryError: CUDA out of memory. Tried to allocate 136.00 MiB (GPU 0; 23.68 GiB total capacity; 2.49 GiB already allocated; 63.75 MiB free; 2.67 GiB reserved in total by PyTorch) If reserved memory is >> allocated memory try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF

尝试2 

```bash
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:64
```

失败：torch.cuda.OutOfMemoryError: CUDA out of memory. Tried to allocate 68.00 MiB (GPU 0; 23.68 GiB total capacity; 2.71 GiB already allocated; 15.75 MiB free; 2.71 GiB reserved in total by PyTorch) If reserved memory is >> allocated memory try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF

尝试3

```bash
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:32
```

失败： CUDA out of memory. Tried to allocate 68.00 MiB (GPU 0; 23.68 GiB total capacity; 2.71 GiB already allocated; 15.75 MiB free; 2.71 GiB reserved in total by PyTorch) If reserved memory is >> allocated memory try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF

尝试4：change to a smaller model

```python
from huggingface_hub import hf_hub_download
import os

# Hugging Face 模型路径和目标路径
model_repo = "Vision-CAIR/vicuna-7b"
destination_path = "/home/louey/MiniGPT-4/vicuna7b"

# 创建目标文件夹（如果不存在）
os.makedirs(destination_path, exist_ok=True)

# 文件列表
files_to_download = [
    "pytorch_model-00001-of-00003.bin",
    "pytorch_model-00002-of-00003.bin",
    "config.json",
    "generation_config.json",
    "pytorch_model.bin.index.json",
    "special_tokens_map.json",
    "tokenizer.model",
    "tokenizer_config.json",
]

# 下载文件
for file_name in files_to_download:
    hf_hub_download(repo_id=model_repo, filename=file_name, local_dir=destination_path)
```

```bash
unset PYTORCH_CUDA_ALLOC_CONF;

python /home/louey/MiniGPT-4/hg.py
```

成功


# merge

```python
import torch
from transformers import AutoTokenizer, AutoModelForCauimport torch
from transformers import AutoTokenizer, AutoModelForCausalLM

# 设置模型路径
vicuna_model_path = "/home/louey/MiniGPT-4/vicuna7b"
minigpt4_checkpoint_path = "/home/louey/MiniGPT-4/prerained_minigpt4_7b.pth"
output_model_path = "/home/louey/MiniGPT-4/minigpt4/mergedMiniGPT4Vicuna7B"

# 检查是否有可用的 GPU
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 加载 Vicuna 7B 模型和 tokenizer
tokenizer = AutoTokenizer.from_pretrained(vicuna_model_path)
model = AutoModelForCausalLM.from_pretrained(vicuna_model_path).to(device)

# 加载 MiniGPT-4 检查点
checkpoint = torch.load(minigpt4_checkpoint_path, map_location=device)

# 融合模型和检查点
model.load_state_dict(checkpoint, strict=False)

# 保存融合后的模型和 tokenizer
model.save_pretrained(output_model_path)
tokenizer.save_pretrained(output_model_path)

print(f"Model and tokenizer have been saved to {output_model_path}")
```
失败： Traceback (most recent call last):
。。。
  File "/home/louey/anaconda3/envs/minigptv/lib/python3.9/site-packages/torch/nn/modules/module.py", line 1143, in convert
    return t.to(device, dtype if t.is_floating_point() or t.is_complex() else None, non_blocking)
torch.cuda.OutOfMemoryError: CUDA out of memory. Tried to allocate 172.00 MiB (GPU 0; 23.68 GiB total capacity; 2.59 GiB already allocated; 141.75 MiB free; 2.59 GiB reserved in total by PyTorch) If reserved memory is >> allocated memory try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF

try

```bash
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:32
```
```
import os
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

# 设置 PYTORCH_CUDA_ALLOC_CONF 环境变量
os.environ["PYTORCH_CUDA_ALLOC_CONF"] = "max_split_size_mb:128"

# 设置模型路径
import os
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

# 设置 PYTORCH_CUDA_ALLOC_CONF 环境变量
os.environ["PYTORCH_CUDA_ALLOC_CONF"] = "max_split_size_mb:128"

# 设置模型路径
vicuna_model_path = "/home/louey/MiniGPT-4/vicuna7b"
minigpt4_checkpoint_path = "/home/louey/MiniGPT-4/prerained_minigpt4_7b.pth"
output_model_path = "/home/louey/MiniGPT-4/minigpt4/mergedMiniGPT4Vicuna7B"

# 检查是否有可用的 GPU
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 释放未使用的 GPU 内存
torch.cuda.empty_cache()

# 加载 Vicuna 7B 模型和 tokenizer
tokenizer = AutoTokenizer.from_pretrained(vicuna_model_path)
model = AutoModelForCausalLM.from_pretrained(vicuna_model_path).to(device)

# 加载 MiniGPT-4 检查点
checkpoint = torch.load(minigpt4_checkpoint_path, map_location=device)

# 融合模型和检查点
model.load_state_dict(checkpoint, strict=False)

# 保存融合后的模型和 tokenizer
model.save_pretrained(output_model_path)
tokenizer.save_pretrained(output_model_path)

print(f"Model and tokenizer have been saved to {output_model_path}")
# evaluation

try

```bash
CUDA_VISIBLE_DEVICES=2,4,6 python /home/louey/LLaVA/model_vqa.py \
--model-path /home/louey/MiniGPT-4/minigpt4 \
--image-folder /home/louey/LLaVA/data/images \
--question-file /home/louey/LLaVA/data/validation/question.jsonl \
--answers-file /home/louey/LLaVA/data/validation/answer5mini.jsonl \
--conv-mode llava_v1 \
--num-chunks 1 \
--chunk-idx 0 \
--temperature 0.3 \
--top_p 0.9 \
--num_beams 5
```

(model_vqa.py is from llava)





<font size='5'>**MiniGPT-v2: Large Language Model as a Unified Interface for Vision-Language Multi-task Learning**</font>

Jun Chen, Deyao Zhu, Xiaoqian Shen, Xiang Li, Zechun Liu, Pengchuan Zhang, Raghuraman Krishnamoorthi, Vikas Chandra, Yunyang Xiong☨, Mohamed Elhoseiny☨

☨equal last author

<a href='https://minigpt-v2.github.io'><img src='https://img.shields.io/badge/Project-Page-Green'></a> <a href='https://arxiv.org/abs/2310.09478.pdf'><img src='https://img.shields.io/badge/Paper-Arxiv-red'></a>  <a href='https://huggingface.co/spaces/Vision-CAIR/MiniGPT-v2'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Spaces-blue'> <a href='https://minigpt-v2.github.io'><img src='https://img.shields.io/badge/Gradio-Demo-blue'></a> [![YouTube](https://badges.aleen42.com/src/youtube.svg)](https://www.youtube.com/watch?v=atFCwV2hSY4)


<font size='5'> **MiniGPT-4: Enhancing Vision-language Understanding with Advanced Large Language Models**</font>

Deyao Zhu*, Jun Chen*, Xiaoqian Shen, Xiang Li, Mohamed Elhoseiny

*equal contribution

<a href='https://minigpt-4.github.io'><img src='https://img.shields.io/badge/Project-Page-Green'></a>  <a href='https://arxiv.org/abs/2304.10592'><img src='https://img.shields.io/badge/Paper-Arxiv-red'></a> <a href='https://huggingface.co/spaces/Vision-CAIR/minigpt4'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Spaces-blue'></a> <a href='https://huggingface.co/Vision-CAIR/MiniGPT-4'><img src='https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model-blue'></a> [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1OK4kYsZphwt5DXchKkzMBjYF6jnkqh4R?usp=sharing) [![YouTube](https://badges.aleen42.com/src/youtube.svg)](https://www.youtube.com/watch?v=__tftoxpBAw&feature=youtu.be)

*King Abdullah University of Science and Technology*

## 💡 Get help - [Q&A](https://github.com/Vision-CAIR/MiniGPT-4/discussions/categories/q-a) or [Discord 💬](https://discord.gg/5WdJkjbAeE)

<font size='4'> **Example Community Efforts Built on Top of MiniGPT-4 ** </font> 
  
* <a href='https://github.com/waltonfuture/InstructionGPT-4?tab=readme-ov-file'><img src='https://img.shields.io/badge/Project-Page-Green'></a> **InstructionGPT-4**: A 200-Instruction Paradigm for Fine-Tuning MiniGPT-4 Lai Wei, Zihao Jiang, Weiran Huang, Lichao Sun, Arxiv, 2023

* <a href='https://openaccess.thecvf.com/content/ICCV2023W/CLVL/papers/Aubakirova_PatFig_Generating_Short_and_Long_Captions_for_Patent_Figures_ICCVW_2023_paper.pdf'><img src='https://img.shields.io/badge/Project-Page-Green'></a> **PatFig**: Generating Short and Long Captions for Patent Figures.", Aubakirova, Dana, Kim Gerdes, and Lufei Liu, ICCVW, 2023 


* <a href='https://github.com/JoshuaChou2018/SkinGPT-4'><img src='https://img.shields.io/badge/Project-Page-Green'></a> **SkinGPT-4**: An Interactive Dermatology Diagnostic System with Visual Large Language Model, Juexiao Zhou and Xiaonan He and Liyuan Sun and Jiannan Xu and Xiuying Chen and Yuetan Chu and Longxi Zhou and Xingyu Liao and Bin Zhang and Xin Gao,  Arxiv, 2023 


* <a href='https://huggingface.co/Tyrannosaurus/ArtGPT-4'><img src='https://img.shields.io/badge/Project-Page-Green'></a> **ArtGPT-4**: Artistic Vision-Language Understanding with Adapter-enhanced MiniGPT-4.",  Yuan, Zhengqing, Huiwen Xue, Xinyi Wang, Yongming Liu, Zhuanzhe Zhao, and Kun Wang, Arxiv, 2023 


</font>

## News
[Oct.31 2023] We release the evaluation code of our MiniGPT-v2.  

[Oct.24 2023] We release the finetuning code of our MiniGPT-v2.

[Oct.13 2023] Breaking! We release the first major update with our MiniGPT-v2

[Aug.28 2023] We now provide a llama 2 version of MiniGPT-4

## Online Demo

Click the image to chat with MiniGPT-v2 around your images
[![demo](figs/minigpt2_demo.png)](https://minigpt-v2.github.io/)

Click the image to chat with MiniGPT-4 around your images
[![demo](figs/online_demo.png)](https://minigpt-4.github.io)


## MiniGPT-v2 Examples

![MiniGPT-v2 demos](figs/demo.png)



## MiniGPT-4 Examples
  |   |   |
:-------------------------:|:-------------------------:
![find wild](figs/examples/wop_2.png) |  ![write story](figs/examples/ad_2.png)
![solve problem](figs/examples/fix_1.png)  |  ![write Poem](figs/examples/rhyme_1.png)

More examples can be found in the [project page](https://minigpt-4.github.io).



## Getting Started
### Installation

**1. Prepare the code and the environment**

Git clone our repository, creating a python environment and activate it via the following command

```bash
git clone https://github.com/Vision-CAIR/MiniGPT-4.git
cd MiniGPT-4
conda env create -f environment.yml
conda activate minigptv
```


**2. Prepare the pretrained LLM weights**

**MiniGPT-v2** is based on Llama2 Chat 7B. For **MiniGPT-4**, we have both Vicuna V0 and Llama 2 version.
Download the corresponding LLM weights from the following huggingface space via clone the repository using git-lfs.

|                            Llama 2 Chat 7B                             |                                           Vicuna V0 13B                                           |                                          Vicuna V0 7B                                          |
:------------------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------------:
[Download](https://huggingface.co/meta-llama/Llama-2-7b-chat-hf/tree/main) | [Downlad](https://huggingface.co/Vision-CAIR/vicuna/tree/main) | [Download](https://huggingface.co/Vision-CAIR/vicuna-7b/tree/main) 


Then, set the variable *llama_model* in the model config file to the LLM weight path.

* For MiniGPT-v2, set the LLM path 
[here](minigpt4/configs/models/minigpt_v2.yaml#L15) at Line 14.

* For MiniGPT-4 (Llama2), set the LLM path 
[here](minigpt4/configs/models/minigpt4_llama2.yaml#L15) at Line 15.

* For MiniGPT-4 (Vicuna), set the LLM path 
[here](minigpt4/configs/models/minigpt4_vicuna0.yaml#L18) at Line 18

**3. Prepare the pretrained model checkpoints**

Download the pretrained model checkpoints


| MiniGPT-v2 (after stage-2) | MiniGPT-v2 (after stage-3) | MiniGPT-v2 (online developing demo)| 
|------------------------------|------------------------------|------------------------------|
| [Download](https://drive.google.com/file/d/1Vi_E7ZtZXRAQcyz4f8E6LtLh2UXABCmu/view?usp=sharing) |[Download](https://drive.google.com/file/d/1HkoUUrjzFGn33cSiUkI-KcT-zysCynAz/view?usp=sharing) | [Download](https://drive.google.com/file/d/1aVbfW7nkCSYx99_vCRyP1sOlQiWVSnAl/view?usp=sharing) |


For **MiniGPT-v2**, set the path to the pretrained checkpoint in the evaluation config file 
in [eval_configs/minigptv2_eval.yaml](eval_configs/minigptv2_eval.yaml#L10) at Line 8.



| MiniGPT-4 (Vicuna 13B) | MiniGPT-4 (Vicuna 7B) | MiniGPT-4 (LLaMA-2 Chat 7B) |
|----------------------------|---------------------------|---------------------------------|
| [Download](https://drive.google.com/file/d/1a4zLvaiDBr-36pasffmgpvH5P7CKmpze/view?usp=share_link) | [Download](https://drive.google.com/file/d/1RY9jV0dyqLX-o38LrumkKRh6Jtaop58R/view?usp=sharing) | [Download](https://drive.google.com/file/d/11nAPjEok8eAGGEG1N2vXo3kBLCg0WgUk/view?usp=sharing) |

For **MiniGPT-4**, set the path to the pretrained checkpoint in the evaluation config file 
in [eval_configs/minigpt4_eval.yaml](eval_configs/minigpt4_eval.yaml#L10) at Line 8 for Vicuna version or [eval_configs/minigpt4_llama2_eval.yaml](eval_configs/minigpt4_llama2_eval.yaml#L10) for LLama2 version.   



### Launching Demo Locally

For MiniGPT-v2, run
```
python demo_v2.py --cfg-path eval_configs/minigptv2_eval.yaml  --gpu-id 0
```

For MiniGPT-4 (Vicuna version), run

```
python demo.py --cfg-path eval_configs/minigpt4_eval.yaml  --gpu-id 0
```

For MiniGPT-4 (Llama2 version), run

```
python demo.py --cfg-path eval_configs/minigpt4_llama2_eval.yaml  --gpu-id 0
```


To save GPU memory, LLMs loads as 8 bit by default, with a beam search width of 1. 
This configuration requires about 23G GPU memory for 13B LLM and 11.5G GPU memory for 7B LLM. 
For more powerful GPUs, you can run the model
in 16 bit by setting `low_resource` to `False` in the relevant config file:

* MiniGPT-v2: [minigptv2_eval.yaml](eval_configs/minigptv2_eval.yaml#6) 
* MiniGPT-4 (Llama2): [minigpt4_llama2_eval.yaml](eval_configs/minigpt4_llama2_eval.yaml#6)
* MiniGPT-4 (Vicuna): [minigpt4_eval.yaml](eval_configs/minigpt4_eval.yaml#6)

Thanks [@WangRongsheng](https://github.com/WangRongsheng), you can also run MiniGPT-4 on [Colab](https://colab.research.google.com/drive/1OK4kYsZphwt5DXchKkzMBjYF6jnkqh4R?usp=sharing)


### Training
For training details of MiniGPT-4, check [here](MiniGPT4_Train.md).

For finetuning details of MiniGPT-v2, check [here](MiniGPTv2_Train.md)


### Evaluation
For finetuning details of MiniGPT-v2, check [here](eval_scripts/EVAL_README.md)  


## Acknowledgement

+ [BLIP2](https://huggingface.co/docs/transformers/main/model_doc/blip-2) The model architecture of MiniGPT-4 follows BLIP-2. Don't forget to check this great open-source work if you don't know it before!
+ [Lavis](https://github.com/salesforce/LAVIS) This repository is built upon Lavis!
+ [Vicuna](https://github.com/lm-sys/FastChat) The fantastic language ability of Vicuna with only 13B parameters is just amazing. And it is open-source!
+ [LLaMA](https://github.com/facebookresearch/llama) The strong open-sourced LLaMA 2 language model.


If you're using MiniGPT-4/MiniGPT-v2 in your research or applications, please cite using this BibTeX:
```bibtex


@article{chen2023minigptv2,
      title={MiniGPT-v2: large language model as a unified interface for vision-language multi-task learning}, 
      author={Chen, Jun and Zhu, Deyao and Shen, Xiaoqian and Li, Xiang and Liu, Zechu and Zhang, Pengchuan and Krishnamoorthi, Raghuraman and Chandra, Vikas and Xiong, Yunyang and Elhoseiny, Mohamed},
      year={2023},
      journal={arXiv preprint arXiv:2310.09478},
}

@article{zhu2023minigpt,
  title={MiniGPT-4: Enhancing Vision-Language Understanding with Advanced Large Language Models},
  author={Zhu, Deyao and Chen, Jun and Shen, Xiaoqian and Li, Xiang and Elhoseiny, Mohamed},
  journal={arXiv preprint arXiv:2304.10592},
  year={2023}
}
```


## License
This repository is under [BSD 3-Clause License](LICENSE.md).
Many codes are based on [Lavis](https://github.com/salesforce/LAVIS) with 
BSD 3-Clause License [here](LICENSE_Lavis.md).
