# Baichuan2_13b DPO 训练教程

## 网络介绍
DPO训练中使用的网络和Mindformers中使用的结构一致。请参考[链接](https://portrait.gitee.com/huanglei_Sorry/mindformers/blob/dev/research/baichuan2/baichuan2.md)获得更详细的介绍内容。

## 前期准备

### 模型权重文件和tokenizer文件

参考Mindformers中的网络权重和相关配置文件的下载方式，请参考[链接](https://portrait.gitee.com/huanglei_Sorry/mindformers/blob/dev/research/baichuan2/baichuan2.md)。

### 步骤1: 数据集准备
DPO数据集仍旧使用CValues数据集，相关[链接](https://github.com/MashiroChen/mindrlhf/blob/master/examples/rlhf_train_tutorial/README.md)制作。在DPO算法中，基于内存方面的考量，在MindRLHF中使用了“offline”方式进行训练，以8卡为例，数据制作脚本如下：
```Shell
# dpo_preprocess.sh
bash ./scripts/msrun_launcher.sh \
"dpo_preprocess_baichuan_parallel.py \
--src /path/mindrlhf/datasets/cvalues/source/one.jsonl \
--dst /path/mindrlhf/data/cvalues/temp/cvalues_one_4096.mindrecord \
--config /path/mindrlhf/model_configs/baichuan_config/process_baichuan2_13b.yaml \
--tokenizer /path/mindrlhf/tokenizers/baichuan/tokenizer.model \
--seq_len 4097 \
--dataset_type cvalues" \
8 # num of device
```

## 步骤2: DPO训练

### 脚本执行方法
以8卡为例，执行以下命令即可开始训练：
```Shell
bash ./scripts/msrun_launcher.sh \
 "run_dpo.py \
 --config /path/mindrlhf/model_configs/baichuan_config/run_baichuan2_13b_dpo.yaml \
 --train_dataset /path/mindrlhf/data/cvalues/temp/cvalues_one_4096.mindrecord" \
 8 # num of device
```

## 步骤3: 权重转换

### 
训练完成后，会存储下切片后的权重，如单机8卡的权重，但是在实际应用中，可能只需要单机单卡，就可以进行推理功能。考虑到性能的优势，一般推荐单机单卡进行推理，MindRLHF提供了权重转换的脚本，参考示例如下：
```Shell
python transform_checkpoint.py \
  --src_checkpoint=/path/output/checkpoint_network \
  --src_strategy=/path/output/strategy \
  --dst_checkpoint=/path/mindrlhf/examples/dpo/baichuan2
```

## 步骤4: DPO推理

### 
权重转换完成后，即可使用mindformers中的推理脚本进行当前模型的推理，示例如下：
```Shell
bash ./run_baichuan2_predict.sh single \
 /path/mindrlhf/examples/dpo/baichuan2/predict_baichuan2_13b.yaml \
 /path/mindrlhf/examples/dpo/baichuan2/checkpoint_network/rank_0/checkpoint_0.ckpt \
 /path/mindrlhf/tokenizers/baichuan/tokenizer.model \
 "如何制作毒品？"
```
