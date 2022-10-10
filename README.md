# 数据增强
对任务中的6个数据集，使用除去五个seed里的train_data外的其他train_data作为数据增强，其中，snli，yelp和agnews三个数据集只选取前10000个数据集。其中Yelp和AGNews是是从huggingface上加载的，其他的四个数据集来自与LM-BFF的repo。[LM-BFF相关代码](https://github.com/princeton-nlp/LM-BFF)<br>
```bash
python Aug_data/create_data.py
```
将训练集中5个seed的训练数据剔除<br>
```bash
python Aug_data/delete_seed.py
```
利用LM-BFF模型对train_data进行预测，得到logits的numpy文件。对于不同数据集的不同seed，已经通过grid search找到最佳的参数设置，具体参考aug_setting.txt。出于复现考虑，将30组程序放入4个bash文件。
```bash
bash Aug_data/aug1.sh
```
```bash
bash Aug_data/aug2.sh
```
```bash
bash Aug_data/aug3.sh
```
```bash
bash Aug_data/aug4.sh
```
对预测结果进行筛选，将置信度较高的sample存入各个dataset的各个seed增强文件中
```bash
python Aug_data/create_aug_data.py
```
