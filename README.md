### Эпигеномика
# Домашнее задание №3. Отчёт
Целью данного домашнего задания является разбивка (аннотация) генома человека на разные типы эпигенетических состояний. Это будет сделано на основании данных о наличии различных гистоновых модификаций (гистоновый код) в соответствующих участках генома. Для выполнения этого задания мы будем работать с чтениями, полученными в ChIP-seq экспериментах из проекта [ENCODE](https://www.encodeproject.org/), которые были выровнены на геном человека (версия hg19). 

Автоматическая разбивка генома на эпигенетические типы будет осуществляться с помощью программы ChromHMM. В результате итеративной процедуры (алгоритм Баума-Велша) программа определит сочетание гистоновых меток, которые характерны для каждого из N разных эпигенетических типов (число N указывается пользователем при запуске программы). Наша задача будет заключаться в том, чтобы на основании косвенных независимых наблюдений вручную приписать каждому из N эпигенетических типов возможную биологическую функцию.

Работа выполнялась в [блокноте Google Colab](https://colab.research.google.com/drive/18cMAmYvewLoa0jr2PQRDhexm3osyFpb2?usp=sharing).

## Основное задание
В качестве клеточной линии выберем `HUVEC` (так как `BJ` из предыдущего задания не было в списке).

Скачаем контрольным экспериментом и выравнивание 10-ти разных гистоновых модификаций:
|  Гистонная метка | Название файла  |
|---|---|
| Control | wgEncodeBroadHistoneHuvecControlStdAlnRep1.bam |
|Ezh239875 |	wgEncodeBroadHistoneHuvecEzh239875AlnRep1.bam	|
| H2az	| wgEncodeBroadHistoneHuvecH2azAlnRep1.bam |
|	H3k09me3	| wgEncodeBroadHistoneHuvecH3k09me3AlnRep1.bam |
|	H3k4me1Std|	wgEncodeBroadHistoneHuvecH3k4me1StdAlnRep1.bam |
|	H3k9acStd	| wgEncodeBroadHistoneHuvecH3k9acStdAlnRep1.bam |
|	H3k9me1Std|	wgEncodeBroadHistoneHuvecH3k9me1StdAlnRep1.bam |
|	H3k27acStd	| wgEncodeBroadHistoneHuvecH3k27acStdAlnRep1.bam |
|	H3k27me3Std	| wgEncodeBroadHistoneHuvecH3k27me3StdAlnRep1.bam |
|	H3k36me3Std	| wgEncodeBroadHistoneHuvecH3k36me3StdAlnRep1.bam |
|	H3k79me2	| wgEncodeBroadHistoneHuvecH3k79me2AlnRep1.bam |

Запустим `ChromHMM` с опциями `BinarizeBAM`:
```
java -mx5000M -jar /content/ChromHMM/ChromHMM.jar \
  BinarizeBam -b 200  /content/ChromHMM/CHROMSIZES/hg19.txt /content/ cellmarkfiletable.txt   binarizedData
```
Далее обучим на обработанных данных с опцией `LearnModel`, ставим целью разбивку на 15 типов:
```
java -mx5000M -jar /content/ChromHMM/ChromHMM.jar LearnModel -b 200 /content/binarizedData/ /content/learnModelData 15 hg19
```

После отработки `ChromHMM` получим результат с разбиением на интервалы и с графиками, описывающими разбиение:

| | | | | |
|---|---|---|---|---|
| ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/emissions.png) | ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/transitions.png) | ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/overlap.png) |![](https://github.com/akamaaru/hse25_hw3/blob/main/img/RefSeqTES_neighborhood.png) | ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/RefSeqTSS_neighborhood.png)|

Геномный браузер с пользовательским треком (я использовал `Huvec_15_expanded.bed` из результатов `ChromHMM`) в выбранной клеточной линией и метками:

![](https://github.com/akamaaru/hse25_hw3/blob/main/img/browser.png)

За что вообще отвечает `HUVEC`? Клеточная линия `HUVEC` (Human Umbilical Vein Endothelial Cells) — это эндотелиальные клетки, 
выстилающие внутреннюю поверхность пупочной вены. 
Для них характерны особые эпигенетические паттерны, связанные с сосудистой функцией, регуляцией воспаления, и прочие.

На основании этого (а также на основании графиков результата разбиения и информации из геномного браузера) можно вывести функции наших полученных состояний:

| № | Имя состояния | Соответствующая метка | Биологическая роль  |
|-------|------------------------------------|------------------------------------------------|--------------------------------------------------------------|
| 1     | Active Promoter (Endothelial)      | H3K9ac, H3K27ac, H2A.Z                          | Активный промотор сосудистых генов                           |
| 2     | Weak Promoter                      | H3K9ac (низкий уровень)                        | Слабо активный промотор                                      |
| 3     | Promoter Flanking                  | H2A.Z, H3K4me1                                 | Границы промоторов, поддержка активации                      |
| 4     | Strong Enhancer (Vascular)         | H3K4me1, H3K27ac                               | Сильный энхансер, возможно сосудистый                        |
| 5     | Strong Enhancer (Inflammatory)     | H3K4me1, H3K27ac                               | Сильный энхансер воспалительных генов                        |
| 6     | Weak Enhancer                      | H3K4me1                                        | Потенциальный/неактивный энхансер                            |
| 7     | Insulator (CTCF region)            | -                                             | Разделение TAD, регуляция контактов                          |
| 8     | Transcription Start Transition     | H3K27ac, H3K36me3                              | Переход от промотора к телу гена                             |
| 9     | Strong Transcription               | H3K36me3, H3K79me2                             | Тело активно транскрибируемых генов                          |
| 10    | Weak Transcribed                   | Низкие уровни H3K36me3                         | Слабо транскрибируемые гены                                  |
| 11    | Polycomb Repression (Developmental)| H3K27me3, EZH2                                 | Репрессия генов развития                                     |
| 12    | Polycomb Repression (Stable)       | H3K27me3, EZH2                                 | Устойчивая Polycomb-репрессия                                |
| 13    | Heterochromatin                    | H3K9me3, H3K9me1                               | Гетерохроматин, структурные или молчащие регионы             |
| 14    | Repeat/Artifact                    | -                                                | Повторы, технические артефакты                               |
| 15    | Quiescent/Low Activity             | -                                              | Без признаков активности                                     |

Теперь в наш `Huvec_15_expanded.bed` добавим названия, которые мы дали. 
Получится файл `Huvec_15_annotated.bed`, который можно найти в той же папке с результатами.
Откроем эту разметку в геномном браузере:

![](https://github.com/akamaaru/hse25_hw3/blob/main/img/browser_annotated.png)

## Бонусное задание
Теперь, когда у нас есть предсказания, можно оценить их качество.
Для этого, обучим модель уже на других репликах взятых гистоновых модификаций, 
а потом с помощью программы `SAGAconf` посчитаем R2-score для каждой из модификаций, 
указав программе как результат разбиения первой группы реплик, так и второй.

Так как при обучении модели в основной части задания апостериорные вероятности не вычислялись, пришлось запустить программу во второй раз для каждой из реплик:
```
java -mx1600M -jar ChromHMM/ChromHMM.jar \
  LearnModel -printposterior /content/binarizedData/ /content/learnModelDataPosterior 15 hg19
java -mx1600M -jar ChromHMM/ChromHMM.jar \
  LearnModel -printposterior /content/binarizedData2/ /content/learnModelDataPosterior2 15 hg19
```
Парсим данные с помощью `SAGAconf_parser`:
```
python SAGAconf/SAGAconf_parser.py --out_format bed --saga chmm \
  learnModelDataPosterior/POSTERIOR/ 200 SAGAconfRes/rep1
python SAGAconf/SAGAconf_parser.py --out_format bed --saga chmm \
  learnModelDataPosterior2/POSTERIOR/ 200 SAGAconfRes/rep2
```
Теперь запускаем сам `SAGAconf`:
```
python SAGAconf/SAGAconf.py \
  SAGAconfRes/rep1/parsed_posterior.bed SAGAconfRes/rep2/parsed_posterior.bed SAGAconfRes/res
```

Получилась следующая таблица:

| №           | R2-score          |
|------------|-------------------|
| 1          | 0.7581801068035559|
| 2          | 0.02559859545371701|
| 3          | 0.6268033730235885|
| 4          | 0.8626236376000166|
| 5          | 0.7218704355797874|
| 6          | 0.6151001177590957|
| 7          | 0.33923432153924526|
| 8          | 0.4836659289035733|
| 9          | 0.5103336876914248|
| 10         | 0.7667971837423139|
| 11         | 0.7057605643333116|
| 12         | 0.423487085817551 |
| 13         | 0.9398634501331234|
| 14         | 0.9808090946948198|
| 15         | 0.4984481865705773|

Обычно порогом доверия считается значение около 0.5. 
Тогда, по этой таблице можно узнать, что типам 2, 7, 8, 12, 15 мы доверять не будем.
Остальные имеют R2-score > 0.5, и им доверять можно больше.
