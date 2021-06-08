# Сравнение реализаций метрик
## Разногласия
MAP:
Разное усреднение в AP
- replay и recBole усредняет по k
- rs_metrics и betaRecsys по количеству релевантных
- daisyRec первоначально падал, выдает что-то третье, меньше всех
- elliot?? Очень большое значение

HitRate:
- betaRecsys ?

MRR:
- daisyRec ищет не первый релевантный, а суммирует все. т.е. он не ограничен единицей

NDCG:
- daisyRec почему-то выдал единицу
- elliot почему-то маленькое значение

RocAuc:
- daisyRec упал
- elliot и beta почему-то большие значения


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
- [x] [Beta-Recsys](https://github.com/beta-team/beta-recsys)
- [x] [DaisyRec](https://github.com/AmazingDD/daisyRec)
- [x] [RecBole](https://github.com/RUCAIBox/RecBole)
- [ ] [MSRecommenders](https://github.com/microsoft/recommenders)
- [x] [elliot](https://github.com/sisinflab/elliot)
- [ ] [NeuRec](https://github.com/wubinzzu/NeuRec)
- [ ] [surprise](https://github.com/NicolasHug/Surprise)
- [ ] [OpenRec](https://github.com/ylongqi/openrec)
- [ ] [RecsysPytorch](https://github.com/yoongi0428/RecSys_PyTorch)
- [ ] [RecSys19 Evaluation](https://github.com/MaurizioFD/RecSys2019_DeepLearning_Evaluation)

