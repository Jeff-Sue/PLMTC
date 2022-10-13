# PLMTC (PLMTuningCompetition)
This is the implementation for the PLMTuningCompetition.
## 容器环境
```bash
docker run -it --gpus 1 plm_cls:code /bin/bash
```
进入docker容器后，cd /home/plm_cls_docker,开始运行代码
## 数据增强
进入工作环境：
```bash
conda activate lmbff
```
对任务中的6个数据集，使用除去五个seed里的train_data外的其他train_data作为数据增强，其中，snli，yelp和agnews三个数据集只选取前10000个数据集。其中Yelp和AGNews是是从huggingface上加载的，其他的四个数据集来自于LM-BFF的repo。[LM-BFF相关代码](https://github.com/princeton-nlp/LM-BFF)<br>
```bash
python LB_aug/create_data.py
```
将训练集中5个seed的训练数据剔除<br>
```bash
python LB_aug/delete_seed.py
```
利用LM-BFF模型对train_data进行预测，得到logits的numpy文件。对于不同数据集的不同seed，已经通过grid search找到最佳的参数设置，具体参考aug_setting.txt。出于复现考虑，将30组程序放入4个bash文件。
```bash
bash LB_aug/aug1.sh
```
```bash
bash LB_aug/aug2.sh
```
```bash
bash LB_aug/aug3.sh
```
```bash
bash LB_aug/aug4.sh
```

## 分类器训练
在进行训练之前，需要进行环境的变更：

```bash
conda deactivate
conda activate lmaas
```

分别训练不同task所有seed的分类器，首先需要对文件目录进行调整：

```bash
cd LB_aug/data

mv k-shot/ag_news AGNews
mv k-shot/MRPC MRPC
mv k-shot/SNLI SNLI
mv k-shot/SST-2 SST-2
mv k-shot/trec TREC
mv k-shot/yelp Yelp

# cd into LB_aug
cd ../   
python create_aug_data.py    #create augment.tsv
```
进入再运行不同task的bash文件完成classifier的训练：

```bash
cd ..
sh train_cls_agnews_allseed.sh
sh train_cls_mrpc_allseed.sh
sh train_cls_sst2_allseed.sh
sh train_cls_yelp_allseed.sh
sh train_cls_snli_allseed.sh
sh train_cls_trec_allseed.sh
```

最后对每一个task进行predict，得到结果并保存在prediction文件夹中：
```bash
python test_submit.py
```
