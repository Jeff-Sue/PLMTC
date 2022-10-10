# 数据增强
## 数据准备
对任务中的6个数据集，使用除去五个seed里的train_data外的其他train_data作为数据增强，其中，snli，yelp和agnews三个数据集只选取前10000个数据集。以上6个数据集都是从huggingface上加载的。<br>
```bash
python Aug_data/create_data.py
```
