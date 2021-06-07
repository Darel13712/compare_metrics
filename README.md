# Сравнение реализаций метрик

| lib        | HitRate@20 | MAP@20 | MRR@20 | NDCG@20 | Precision@20 | Recall@20 | RocAuc@20 |
| ---------- | ---------- | ------ | ------ | ------- | ------------ | --------- | --------- |
| replay     | 0.4746     | 0.0393 | 0.1863 | 0.0933  | 0.0575       | 0.0963    | 0.2832    |
| rs_metrics | 0.4746     | 0.0317 | 0.1863 | 0.0933  | 0.0575       | 0.0963    | —         |
| daisyrec   | 0.474570   | FAIL   | 0.2748 | 1.0     | 0.0575       | 0.0963    | FAIL      |
| recbole    | 0.474570   | 0.0392 | 0.1863 | 0.0932  | 0.0575       | 0.0963    | 0.28317*  |
| elliot     | 0.474570   | 0.0729 | 0.1863 | 0.0898  | 0.0575       | 0.0963    | 0.62689   |
| betaRecsys | -          | 0.0316 | -      | 0.0932  | 0.0575       | 0.0963    | 0.51050   |
|            |            |        |        |         |              |           |           |
|            |            |        |        |         |              |           |           |
|            |            |        |        |         |              |           |           |
|            |            |        |        |         |              |           |           |
|            |            |        |        |         |              |           |           |
|            |            |        |        |         |              |           |           |
|            |            |        |        |         |              |           |           |



## Описание файлов

- `pred.csv` – предсказание модели
- `test.csv` — тестовые данные
- `res.csv` — результаты метрик в различных библиотеках



## Что хочется получить 

Надо дополнить файл `res.csv` значениями метрик, посчитанными разными инструментами. Осторожно с порядком аргументов. Он может быть `true, pred` или наоборот.



### Список метрик 

Метрики считаем для `k=20`.

- hitrate
- precision
- recall
- map
- mrr
- ndcg
- roc-auc

Если встретятся ещё интересные метрики, можно их тоже добавить, но вроде это основные.



- [x] replay
- [ ] [Beta-Recsys](https://github.com/beta-team/beta-recsys)
- [ ] [DaisyRec](https://github.com/AmazingDD/daisyRec)
- [ ] [RecBole](https://github.com/RUCAIBox/RecBole)
- [ ] [MSRecommenders](https://github.com/microsoft/recommenders)
- [ ] [elliot](https://github.com/sisinflab/elliot)
- [ ] [NeuRec](https://github.com/wubinzzu/NeuRec)
- [ ] [surprise](https://github.com/NicolasHug/Surprise)
- [ ] [OpenRec](https://github.com/ylongqi/openrec)
- [ ] [RecsysPytorch](https://github.com/yoongi0428/RecSys_PyTorch)
- [ ] [RecSys19 Evaluation](https://github.com/MaurizioFD/RecSys2019_DeepLearning_Evaluation)

