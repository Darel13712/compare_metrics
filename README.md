# Сравнение реализаций метрик
## Разногласия
MAP:
Разное усреднение в AP
- replay и recBole усредняет по k
- rs_metrics и betaRecsys по количеству релевантных
- daisyRec первоначально падал, выдает что-то третье, меньше всех
- elliot?? Очень большое значение
- в Suprise нет метрики
- 
HitRate:
- betaRecsys ?
- в Suprise нет метрики

MRR:
- daisyRec ищет не первый релевантный, а суммирует все. т.е. он не ограничен единицей
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall
- в Suprise нет метрики

NDCG:
- daisyRec почему-то выдал единицу
- elliot почему-то маленькое значение
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall
- в Suprise нет метрики

ROC AUC:
Глобально используют несколько подходов:
1) Средний roc auc по каждому пользователю. roc auc для пользователя, у которого нет ground truth, выставляется равным нулю.
2) Берем все скоры всех пользователей и все ground truth для этих предиктов, считаем ROC AUC.
3) Тоже скор по всем предиктам независимо от пользователей. В отличие от предыдущего способа здесь к negatives добалвяются все айтемы, которые не вошли в предсказанный топ.
4) Group Area Under the Curve: считает AUC для каждого пользователя и взвешивает при усреднении
https://elliot.readthedocs.io/en/latest/guide/metrics/accuracy.html#elliot.evaluation.metrics.accuracy.AUC.gauc.GAUC
5) Limited AUC: усредняет по пользователям. Такой же, как в 4м пункте, но positives ограничиваются топом, в 3м positives берутся все.
https://wiki.epfl.ch/edicpublic/documents/Candidacy%20exam/Evaluation.pdf

В ситуации, когда мы сами подали отрезанный топ:
- 1й и 5й способы совпадают (если наличие заполнения нулями одинаково).

- replay и recBole используют 1й спрособ.
- openRec по сути использует 1й, но умеет считать только ROC AUC для каждого пользователя в отдельности. Сам усреднил, наны заполнил нулями.
- betaRecsys и MSRecommenders использут один код, который считает по 2му способу.
- elliot имеет в арсенале 3, 4, 5 способы. в Однако, отрбрасывает всех пользователей, у которых нет ground trues (другие библиотеки иногда заполняют такие нулями).
В контексте ограничения предиктов нами топ 20, AUC и LAUC совпадают.
- в Suprise нет метрики
- daisyRec упал
- rs_metrics?

Precision:
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall.
- в openRec реализация осталась на tf1 в модуле legacy, хотя библиотека переехала на tf2.

Recall:
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall.

## Примечания
beta recsys скопировали себе код MS Recommeders один в один.

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
- [x] [surprise](https://github.com/NicolasHug/Surprise)
- [x] [OpenRec](https://github.com/ylongqi/openrec)
- [ ] [RecsysPytorch](https://github.com/yoongi0428/RecSys_PyTorch)
- [ ] [RecSys19 Evaluation](https://github.com/MaurizioFD/RecSys2019_DeepLearning_Evaluation)

