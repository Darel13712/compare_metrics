# Сравнение реализаций метрик
## Разногласия
**MAP:**<br/>
Различия идут от формулы average precision (AP) - precision@K суммируются и делятся на разные числа, вот варианты:<br/>
Если не оговорено иное, AP пользователей, у которых нет релевантных объектов в тесте, заполняются нулями, и precision@K используются только те, где k-й элемент релевантный.
1) в знаменателе K: daisyRec (мы пофиксили маленький баг(*))
2) в знаменателе число релевантных объектов из теста: rs_metrics, betaRecsys
3) в знаменателе min(число релевантных объектов из теста, K): replay и recBole
4) elliot просто усредняет (https://github.com/sisinflab/elliot/blob/master/elliot/evaluation/metrics/accuracy/map/map.py#L69) и не заполняет нулями AP - MAP просто не учитывает их при усреднении (https://github.com/sisinflab/elliot/blob/master/elliot/evaluation/metrics/accuracy/map/map.py#L98), кроме того AP считает все precision@K, а не только где k-й элемент релевантный (https://github.com/sisinflab/elliot/blob/master/elliot/evaluation/metrics/accuracy/map/map.py#L69).
5) neuRec заполняет нулями precision@K, где k-й элемент нерелевантен (https://github.com/wubinzzu/NeuRec/blob/master/evaluator/backend/python/metric.py#L36), делит на число релевантных объектов из теста (r_num: https://github.com/wubinzzu/NeuRec/blob/master/evaluator/backend/python/metric.py#L40).
<br/>


**HitRate:**<br/>
- betaRecsys ?
- в Suprise нет метрики
- в NeuRec бага в коде - должно быть "in" вместо "==": https://github.com/wubinzzu/NeuRec/blob/c33333df028d861473ff050338c974e5f4bb5dc5/evaluator/backend/python/metric.py#L12

**MRR:**<br/>
Определяется как обратная позиция первой релевантной рекомендации в списке первых K рекомендаций. Это значение усредняется по всем пользователям. 
- daisyRec ищет не первый релевантный, а суммирует все обратные релевантные ранги для каждого пользователя.
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall.
- в Suprise нет метрики.

**NDCG:**<br/>
DCG имеет 2 реализации, которые дают одинаковый скор в случае, когда предсказанные скоры бинарные, альтернативная формула учитвает порядок элементов в списке путем домножения релевантности элемента на вес равный обратному логарифму номера позиции: https://en.wikipedia.org/wiki/Discounted_cumulative_gain#Discounted_Cumulative_Gain<br/>
- elliot использует альтернативную DCG и не считает NDCG для пользователей, у которых нет релевантных объектов в тесте (https://github.com/sisinflab/elliot/blob/master/elliot/evaluation/metrics/accuracy/ndcg/ndcg.py#L125), 

**ROC AUC:**<br/>
Глобально используют несколько подходов:
1) Считаем ROC AUC для предиктов каждого пользователя отдельно, усредняем. В случае отсутствия ground truth, ROC AUC для этого пользователя заполняется нулем.
3) Стакаем все скоры всех пользователей и все ground truth для этих предиктов, считаем ROC AUC.
4) Тоже скор по всем предиктам независимо от пользователей. В отличие от предыдущего способа здесь к negatives добалвяются все айтемы, которые не вошли в предсказанный топ.
5) Group Area Under the Curve: считает AUC для каждого пользователя и взвешивает при усреднении
https://elliot.readthedocs.io/en/latest/guide/metrics/accuracy.html#elliot.evaluation.metrics.accuracy.AUC.gauc.GAUC
5) Limited AUC: усредняет по пользователям. Такой же, как в 4м пункте, но positives ограничиваются топом, в 3м positives берутся все.
https://wiki.epfl.ch/edicpublic/documents/Candidacy%20exam/Evaluation.pdf

В ситуации, когда мы сами подали отрезанный топ:
- 1й и 5й способы совпадают (если наличие заполнения нулями одинаково).

По сути 1 и 2 способы различаются только в том, считать AUC по всем стакнутым или по отдельности для каждого пользователя. Для того, чтобы понять какой именно вариант релизован но не запускать весь пайплайн (иногда тяжело затолкать предикты), смотрел в код. Например:
- в RecBole в коде видно, что все предикты просто стакаются - нет никакого разделения по пользователям (https://github.com/RUCAIBox/RecBole/blob/master/recbole/evaluator/evaluators.py#L331).
- в OpenRec .

Итого:
- replay использует 1й спрособ.
- recBole 2й и 5й. 
- openRec по сути использует 1й, но умеет считать только ROC AUC для каждого пользователя в отдельности. Сам усреднил, наны заполнил нулями.
- betaRecsys и MSRecommenders использут один код, который считает по 2му способу.
- elliot имеет в арсенале 3, 4, 5 способы. в Однако, отрбрасывает всех пользователей, у которых нет ground trues (другие библиотеки иногда заполняют такие нулями).
В контексте ограничения предиктов нами топ 20, AUC и LAUC совпадают.
- в Suprise нет метрики
- daisyRec упал
- rs_metrics?

**Precision:**<br/>
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall.
- в openRec реализация осталась на tf1 в модуле legacy, хотя библиотека переехала на tf2.

**Recall:**<br/>
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall.

## Примечания
beta recsys скопировали себе код MS Recommeders один в один.
в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall
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
- [x] [MSRecommenders](https://github.com/microsoft/recommenders)
- [x] [elliot](https://github.com/sisinflab/elliot)
- [x] [NeuRec](https://github.com/wubinzzu/NeuRec)
- [x] [surprise](https://github.com/NicolasHug/Surprise)
- [x] [OpenRec](https://github.com/ylongqi/openrec)
- [ ] [RecsysPytorch](https://github.com/yoongi0428/RecSys_PyTorch)
- [ ] [RecSys19 Evaluation](https://github.com/MaurizioFD/RecSys2019_DeepLearning_Evaluation)

