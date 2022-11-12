# PLMTC (PLMTuningCompetition)
This is the implementation for the PLMTuningCompetition.


解压后进入docker_submit目录， 运行
```bash
docker build -t plm_cls /bin/bash
```
进去容器后， 运行
```bash
cd /home/plm_cls_docker
```
以此进入工作目录，并运行以下代码：
## 数据增强
对任务中的6个数据集，使用除去五个seed里的train_data外的其他train_data作为数据增强，其中，snli，yelp和agnews三个数据集只选取前10000个数据集。其中Yelp和AGNews是是从huggingface上加载的，其他的四个数据集来自于LM-BFF的repo。[LM-BFF相关代码](https://github.com/princeton-nlp/LM-BFF)<br>
```bash
cd LB_aug/
```
```bash
/opt/conda/envs/lmaas/bin/python create_data.py
```
将训练集中5个seed的训练数据剔除<br>
```bash
/opt/conda/envs/lmaas/bin/python delete_seed.py
```
利用LM-BFF模型对train_data进行预测，得到logits的numpy文件。对于不同数据集的不同seed，先通过grid search找到最佳的参数设置，具体参考aug_setting.txt。
```bash
bash SNLI.sh
```
```bash
bash QQP.sh
```
```bash
bash QNLI.sh
```
```bash
bash SST-2.sh
```
```bash
bash DBPedia.sh
```
之后，对于每一个data的每一个seed，查找它的最佳参数，并加载对应的logits文件。
```bash
/opt/conda/envs/lmbff/bin/python tools/gather_result.py --condition "{'tag': 'exp', 'task_name': 'sst-2', 'few_shot_type': 'prompt-demo'}"
/opt/conda/envs/lmbff/bin/python tools/gather_result.py --condition "{'tag': 'exp', 'task_name': 'snli', 'few_shot_type': 'prompt-demo'}"
/opt/conda/envs/lmbff/bin/python tools/gather_result.py --condition "{'tag': 'exp', 'task_name': 'qnli', 'few_shot_type': 'prompt-demo'}"
/opt/conda/envs/lmbff/bin/python tools/gather_result.py --condition "{'tag': 'exp', 'task_name': 'qqp/f1', 'few_shot_type': 'prompt-demo'}"
/opt/conda/envs/lmbff/bin/python tools/gather_result.py --condition "{'tag': 'exp', 'task_name': 'dbpedia', 'few_shot_type': 'prompt'}"
```
## 分类器训练

分别训练不同task所有seed的分类器，首先需要对文件目录进行调整：

```bash
cd data

mv k-shot/ag_news AGNews
mv k-shot/MRPC MRPC
mv k-shot/SNLI SNLI
mv k-shot/SST-2 SST-2
mv k-shot/trec TREC
mv k-shot/yelp Yelp

# cd into LB_aug
cd ../

mv logits/ag_news logits/AGNews
mv logits/trec logits/TREC
mv logits/yelp logits/Yelp 

# create augment.tsv
/opt/conda/envs/lmaas/bin/python create_aug_data.py  
```

进入再运行不同task的bash文件完成classifier的训练：

```bash
cd ../
sh train_cls_agnews_allseed.sh
sh train_cls_mrpc_allseed.sh
sh train_cls_sst2_allseed.sh
sh train_cls_yelp_allseed.sh
sh train_cls_snli_allseed.sh
sh train_cls_trec_allseed.sh
```

最后对每一个task进行predict，得到结果并保存在prediction文件夹中：
```bash
/opt/conda/envs/lmaas/bin/python test_submit.py
```

