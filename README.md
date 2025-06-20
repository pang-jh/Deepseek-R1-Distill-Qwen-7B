# 医疗问诊CoT蒸馏
本项目旨在对Deepseek-R1-Distill-Qwen-7B进行医疗CoT数据进行LoRA微调，让Deepseek-R1的慢思考能力在医学领域上进一步提升

## 相关技术
- Transformer架构：涉及到的模型均为Decoder-Only模型
- QLoRA：LoRA的量化版本
- CoT：思维链技术，即慢思考模式，先思考再回答。与之对应是直接输出（快思考）
- LLaMA-Factory： LLaMA-Factory是一个用于大模型微调的工具，支持多种微调方法，如LoRA微调、全量微调（full）以及冻结微调（freeze）等。它提供了灵活的配置选项，方便用户根据需求进行模型调优。 
蒸馏技术：Deepseek-R1-Distill-Qwen-7B就是利用Deepseek-R1满血版当老师，Qwen2.5-7B当学生蒸馏出来的。
- unsloth：集成unsloth优化框架实现2.3倍训练加速（主要是重写了很多CUDA算子） 

## 数据预处理
- 数据集地址：https://huggingface.co/datasets/FreedomIntelligence/medical-o1-reasoning-SFT/viewer/zh
- 对数据进行预处理，处理脚本：process.py

## 克隆LLaMA Factory仓库
- 仓库地址 https://github.com/hiyouga/LLaMA-Factory
- 克隆并安装依赖的linux命令
```bash
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e ".[torch,metrics]"
```
## 配置数据集
- 将数据放到data目录下
- 在data的data_info.json中注册数据集信息，添加如下信息
```bash
  "DISC_small": {
    "file_name": "DISC_small.json",
    "columns": {
      "prompt": "input",
      "response": "output"
    }
  }
```
## 下载模型文件
- 执行命令
```bash
modelscope download --model Qwen/Qwen2.5-3B-Instruct --local_dir qwen2.5-3b-instruct
```
## 配置训练yaml文件
- yaml文件位于examples/train_lora/llama3_lora_sft.yaml
- 配置代码块：llama3_lora_sft.yaml
## 执行训练命令
```bash
llamafactory-cli train examples/train_lora/llama3_lora_sft.yaml
```
## 推理
- 找到saves文件夹中想推理的checkpoint
- 修改yaml文件，位于examples/inference/llama3_lora_sft.yaml
```bash
model_name_or_path: qwen2.5-3b-instruct
adapter_name_or_path: saves/qwen2.5-3b-instruct/lora/1_try/checkpoint-20
template: qwen
infer_backend: huggingface  # choices: [huggingface, vllm]
trust_remote_code: true
```
- 执行推理命令llamafactory-cli webchat examples/inference/llama3_lora_sft.yaml
