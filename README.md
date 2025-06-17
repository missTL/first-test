<div align="center">

<h1> <img src="assets/logo.png" style="vertical-align: -10px;" :height="50px" width="50px"> FutureSightDrive: Thinking Visually with Spatio-Temporal CoT for Autonomous Driving </h1>

<a href="https://arxiv.org/abs/2505.17685"><img src='https://img.shields.io/badge/arXiv-Paper-red?logo=arxiv&logoColor=white' alt='arXiv'></a>
<a href='https://miv-xjtu.github.io/FSDrive.github.io'><img src='https://img.shields.io/badge/Project_Page-Website-green?logo=googlechrome&logoColor=white' alt='Project Page'></a>

Shuang Zeng<sup>1,2</sup>,
[Xinyuan Chang](https://scholar.google.com.hk/citations?user=5OnPBVYAAAAJ&hl=zh-CN)<sup>1</sup>,
Mengwei Xie<sup>1</sup>,
Xinran Liu<sup>1</sup>,
Yifan Bai<sup>2,3</sup>,
Zheng Pan<sup>1</sup>,
Mu Xu<sup>1</sup>,
[Xing Wei](https://scholar.google.com.hk/citations?user=KNyC5EUAAAAJ&hl=zh-CN&oi=ao/)<sup>2</sup>,

<sup>1</sup>Amap, Alibaba Group,
<sup>2</sup>Xi’an Jiaotong University,
<sup>3</sup>DAMO Academy, Alibaba Group

Official implementation of FSDrive - a VLM can think visually about trajectory planning, advancing autonomous driving towards visual reasoning.

https://github.com/user-attachments/assets/a99a14a3-a892-4cbe-ac1f-66b777d9081b

</div>

## Table of Contents
- [🛠️ Installation](#-Installation)
- [📦 Data Preparation](#-Data-Preparation)
- [🚀 Training](#-Training)
- [🎯 Infer](#-Infer)
- [📈 Evaluation](#-Evaluation)
- [👀 Visualization](#-Visualization)
- [📜 Citing](#-Citing)
- [🙏 Acknowledgement](#-Acknowledgement)

## 🛠️ Installation

Create the required environment through the following steps:

```bash
git clone https://github.com/MIV-XJTU/FSDrive.git
cd FSDrive

conda create -n FSDrive python=3.10

cd LLaMA-Factory && pip install -e ".[torch,metrics,deepspeed,liger-kernel,bitsandbytes,vllm,qwen,modelscope]"

cd ..

pip install -r requirements.txt
```

## 📦 Data Preparation

1、Download nuScenes

Download the complete dataset from [nuScenes](https://www.nuscenes.org/nuscenes#download) and extract it to `./LLaMA-Factory/data/nuscenes`

Or establish a soft connection：

```bash
ln -s /path/to/your/nuscenes LLaMA-Factory/data
```

We used pre-cached data from the nuScenes dataset. The data can be downloaded at [Google Drive](). The file `cached_nuscenes_info.pkl` is located in the directory `./create_data`. The `metrics` folder is placed in the directory `./tools/data`.

2、Extract visual tokens

Separately extract the visual tokens of the front view from both the pre-trained and fine-tuned data, to facilitate supervised MLLM:

```bash
python MoVQGAN/pretrain_data.py
python MoVQGAN/sft_data.py
```

3、Construct data

Construct pre-training and fine-tuning data that conform to the LLaMA-Factory format respectively:

```bash
python create_data/pretrain_data.py
python create_data/sft_data.py --split train # Change to "val" for constructing the validation set
```

Follow the [LLaMA-Factory tutorial](https://github.com/hiyouga/LLaMA-Factory/blob/main/data/README.md) and add the dataset information in the file `./LLaMA-Factory/data/dataset_info.json`.

## 🚀 Training
Enter the working directory of LLaMA-Factory:
```bash
cd LLaMA-Factory
```

1、Pre-train

First, pre-train the VLM to activate its visual generation capabilities:
```bash
llamafactory-cli train ../configs/pretrain.yaml
```

2、SFT

Then, based on the pre-trained parameters, fine-tune the VLM to think visually about trajectory planning:
```bash
llamafactory-cli train ../configs/sft.yaml
```
## 🎯 Infer
Run the following command in the LLaMA-Factory directory to infer test dataset:
```bash
python scripts/vllm_infer.py \ 
--model_name_or_path saves/qwen2_vl-7b/sft \
--dataset val_cot_motion \
--template qwen2_vl \
--cutoff_len 32768 \
--max_new_tokens 2048 \
--max_samples 100000 \
--image_resolution 524288 \
--save_name results.jsonl \
--temperature 0.1 \
--top_p 0.1 \
--top_k 10
```

## 📈 Evaluation
First, under the FSDrive directory, match the predicted results with the tokens to facilitate the evaluation:
```bash
cd ..

python tools/match.py \
--pred_trajs_path ./LLaMA-Factory/results.jsonl \
--token_traj_path ./LLaMA-Factory/data/train_cot_motion.json
```

Then evaluate the L2 and collision rate indicators for the end-to-end trajectory planning:
```bash
python tools/evaluation/evaluation.py \
# Change to "stp3" and use the ST-P3 calculation method
--metric uniad \  
--result_file ./LLaMA-Factory/eval_traj.json
```

## 👀 Visualization
Use the following command under the FSDrive directory to visualize the trajectory:
```bash
python tools/visualization/visualize_planning.py \
--pred-trajs-path ./LLaMA-Factory/results.jsonl \
--tokens-path ./LLaMA-Factory/eval_traj.json \  
--output-path ./vis_traj
```

Use the following command under the FSDrive directory to restore the visual tokens to the pixel space and visualize the CoT:
```bash
python ./MoVQGAN/vis.py \
--input_json ./LLaMA-Factory/eval_traj.json \
--output_dir ./vis_cot
```

## 📜 Citing

If you find FSDrive is useful in your research or applications, please consider giving us a star 🌟 and citing it by the following BibTeX entry.

```
@article{zeng2025FSDrive,
      title={FutureSightDrive: Thinking Visually with Spatio-Temporal CoT for Autonomous Driving},
      author={Shuang Zeng and Xinyuan Chang and Mengwei Xie and Xinran Liu and Yifan Bai and Zheng Pan and Mu Xu and Xing Wei},
      journal={arXiv preprint arXiv:2505.17685},
      year={2025}
      }
```

## 🙏 Acknowledgement
Our work is primarily based on the following codebases:[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory), [MoVQGAN](https://github.com/ai-forever/MoVQGAN), [GPT-Driver](https://github.com/PointsCoder/GPT-Driver), [Agent-Driver](https://github.com/USC-GVL/Agent-Driver). We are sincerely grateful for their work.
