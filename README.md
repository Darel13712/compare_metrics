# The Comparison of Metric Implementations

This is a companion repository for the paper **Quality Metrics in Recommender Systems: Do We Calculate Consistently?** Presented as a LBR poster at RecSys'21.



## Files Description

- `pred.csv` – EASE model predictions
- `test.csv` — test data
- `res.csv` — metric values for different libraries
- `res_auc.csv` — extra table with different versions of AUC



### Metrics 

All metrics are calculated at depth cut-off  `k=20`, except for roc-auc

- hitrate
- precision
- recall
- map
- mrr
- ndcg
- roc-auc



# Libraries

- [RePlay](https://github.com/sberbank-ai-lab/RePlay)
- [Beta-Recsys](https://github.com/beta-team/beta-recsys)
- [DaisyRec](https://github.com/AmazingDD/daisyRec)
- [RecBole](https://github.com/RUCAIBox/RecBole)
- [MSRecommenders](https://github.com/microsoft/recommenders)
- [elliot](https://github.com/sisinflab/elliot)
- [NeuRec](https://github.com/wubinzzu/NeuRec)
- [surprise](https://github.com/NicolasHug/Surprise)
- [OpenRec](https://github.com/ylongqi/openrec)
- [RecsysPytorch](https://github.com/yoongi0428/RecSys_PyTorch)
- [RecSys19 Evaluation](https://github.com/MaurizioFD/RecSys2019_DeepLearning_Evaluation)
- [RS Metrics](https://github.com/Darel13712/rs_metrics)