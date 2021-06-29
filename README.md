# Сравнение реализаций метрик
## Разногласия
**MAP:**<br/>
Различия идут от формулы average precision (AP):<br/>

1) AP@K = (сумма precision@i для i, где 0 < i < K и i-й айтем в предсказании - релевантный) / K : daisyRec (мы пофиксили маленький баг(*))
2) AP@K = (сумма precision@i для i, где 0 < i < K и i-й айтем в предсказании - релевантный) / (число релевантных объектов из теста) : rs_metrics, betaRecsys.
3) AP@K = (сумма precision@i для i, где 0 < i < K и i-й айтем в предсказании - релевантный) / [min(число релевантных объектов из теста, K)](https://github.com/wubinzzu/NeuRec/blob/master/evaluator/backend/python/metric.py#L40) : replay, recBole, neuRec. В формуле используется [cumsum](https://github.com/wubinzzu/NeuRec/blob/master/evaluator/backend/python/metric.py#L39), из-за которого в формуле AP знаменатель получается как сумма из минимумов.

4) AP@K = [все](https://github.com/sisinflab/elliot/blob/master/elliot/evaluation/metrics/accuracy/map/map.py#L69) (precision@i, где 0 < i < K) усредняются : elliot. С учетом того, что используются все precision, это равносильно делению на K. MAP не приравнивается нулю для пользователей, у которых нет релевантных айтемов в тесте - для них просто [не выдается](https://github.com/sisinflab/elliot/blob/master/elliot/evaluation/metrics/accuracy/map/map.py#L98) MAP, поэтому уреднение MAP всех пользователей дает число выше. 
<br/>

**HitRate:**<br/>
- в NeuRec бага в коде - должно быть "in" вместо "==": https://github.com/wubinzzu/NeuRec/blob/c33333df028d861473ff050338c974e5f4bb5dc5/evaluator/backend/python/metric.py#L12
- https://github.com/MaurizioFD/RecSys2019_DeepLearning_Evaluation/blob/master/Base/Evaluation/Evaluator.py#L339

**MRR:**<br/>
Определяется как обратная позиция первой релевантной рекомендации в списке первых K рекомендаций. Это значение усредняется по всем пользователям. 
- daisyRec ищет не первый релевантный, а суммирует все обратные релевантные ранги для каждого пользователя.

**NDCG:**<br/>
DCG имеет 2 реализации, которые дают одинаковый скор в случае, когда предсказанные скоры бинарные, альтернативная формула учитвает порядок элементов в списке путем домножения релевантности элемента на вес равный обратному логарифму номера позиции: https://en.wikipedia.org/wiki/Discounted_cumulative_gain#Discounted_Cumulative_Gain<br/>
- elliot использует альтернативную DCG (https://github.com/sisinflab/elliot/blob/master/elliot/evaluation/metrics/accuracy/ndcg/ndcg.py#L125)
- daisyRec использует альтернативную формулу DCG.
- betaRec, MSRecommenders, openRec используют обычную формулу.
- RecBole и Neurec на вход просто принимают просто индексы релевантных или сами списки айтемов, то есть не воспринимает величину рейтинга, использует обычную формулу.
- openRec считают DCG вместро nDCG - в рассчете нет деления на iDCG, кроме того непонятно зачем использовали [экспоненту](https://github.com/ylongqi/openrec/blob/a00de2345844858194ef43ab6845342114a5be93/openrec/tf2/metrics/ranking_metrics.py#L33), которая ни на что не влияет.
- в RecSys2019_DeepLearning_Evaluation в формуле DCG в знаменателе логарифм натуральный, а не по основанию 2. Кроме этого для рассчета iDCG берутся все ground truth relevances, не обрезанные по K.


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

Итого:
- replay использует 1й спрособ.
- recBole 2й и 5й. 
- openRec по сути использует 1й, но умеет считать только ROC AUC для каждого пользователя в отдельности. Сам усреднил, наны заполнил нулями.
- betaRecsys и MSRecommenders использут один код, который считает по 2му способу.
- elliot имеет в арсенале 3, 4, 5 способы. в Однако, отрбрасывает всех пользователей, у которых нет ground trues (другие библиотеки иногда заполняют такие нулями).
В контексте ограничения предиктов нами топ 20, AUC и LAUC совпадают.
- daisyRec упал

**Precision:**<br/>
- в Suprise из метрик реализованы только rmse, mse, а в примерах есть код, который вне библиотеки считает precision и recall - его и использовал.
- в openRec реализация осталась на tf1 в модуле legacy, хотя библиотека переехала на tf2.

**Recall:**<br/>
- в Suprise из метрик реализованы только rmse, mse, в примерах есть код, который вне библиотеки считает precision и recall - его и использовал.

## Примечания
beta recsys скопировали себе код MS Recommeders один в один.<br/>
в Suprise из метрик реализованы только rmse, mse, остальные предлагается писать самим. В примерах есть только precision и recall.<br/>
в NeuRec есть 2 бекенда расчета метрик - Cython и Python, использовал код python-бекенд.

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

