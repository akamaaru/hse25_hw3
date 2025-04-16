### Эпигеномика
# Домашнее задание №3. Отчёт
Целью данного домашнего задания является разбивка (аннотация) генома человека на разные типы эпигенетических состояний. Это будет сделано на основании данных о наличии различных гистоновых модификаций (гистоновый код) в соответствующих участках генома. Для выполнения этого задания мы будем работать с чтениями, полученными в ChIP-seq экспериментах из проекта [ENCODE](https://www.encodeproject.org/), которые были выровнены на геном человека (версия hg19). 

Автоматическая разбивка генома на эпигенетические типы будет осуществляться с помощью программы ChromHMM. В результате итеративной процедуры (алгоритм Баума-Велша) программа определит сочетание гистоновых меток, которые характерны для каждого из N разных эпигенетических типов (число N указывается пользователем при запуске программы). Наша задача будет заключаться в том, чтобы на основании косвенных независимых наблюдений вручную приписать каждому из N эпигенетических типов возможную биологическую функцию.

Работа выполнялась в блокноте [Google Colab](https://colab.research.google.com/drive/18cMAmYvewLoa0jr2PQRDhexm3osyFpb2?usp=sharing).

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

После отработки `ChromHMM` с опциями `BinarizeBAM` и `LearnModel` получим результат с разбиением на интервалы.

| | | | | |
|---|---|---|---|---|
| ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/emissions.png) | ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/transitions.png) | ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/overlap.png) |![](https://github.com/akamaaru/hse25_hw3/blob/main/img/RefSeqTES_neighborhood.png) | ![](https://github.com/akamaaru/hse25_hw3/blob/main/img/RefSeqTSS_neighborhood.png)|

Геномный браузер с пользовательским треком (я использовал `Huvec_15_expanded.bed` из результатов `ChromHMM`) в выбранной клеточной линией и метками:

![](https://github.com/akamaaru/hse25_hw3/blob/main/img/browser.png)

За что вообще отвечает `HUVEC`? Клеточная линия `HUVEC` (Human Umbilical Vein Endothelial Cells) — это эндотелиальные клетки, 
выстилающие внутреннюю поверхность пупочной вены. 
Для них характерны особые эпигенетические паттерны, связанные с сосудистой функцией, регуляцией воспаления, и прочие.

На основании этого (а также на основании графиков результата разбиения и информации из геномного браузера) можно вывести функции наших полученных состояний:

| № | Имя состояния | Биологическая роль  |
|-------|--------------------------------------|----------------------------------------------------------------------------|
| 1     | Active Promoter  | Активный промотор с высокой транскрипцией, открытая хроматиновая структура |
| 2     | Weak Promoter   | Слабо активный промотор, может активироваться при стимуляции               |
| 3     | Flanking Promoter  | Промоторная граница, возможная подготовка к транскрипции                   |
| 4     | Strong Enhancer   | Активный сосудистый энхансер, усиление экспрессии генов                    |
| 5     | Strong Enhancer   | Воспалительный или стимул-специфичный активный энхансер                    |
| 6     | Weak Enhancer  | Потенциальный (праймированный) энхансер, может стать активным              |
| 7     | Insulator  | Граница доменов (TAD), контроль пространственной регуляции генов           |
| 8     | Transcription Start Transition       | Переходная область от промотора к транскрибируемой зоне                    |
| 9     | Strong Transcription | Активная транскрипция вдоль тела гена                                  |
| 10    | Weak Transcribed    | Слабо транскрибируемые участки                                             |
| 11    | Polycomb Repression | Репрессированные эмбриональные/развивающиеся гены                        |
| 12    | Polycomb Repression (stable)         | Стабильная репрессия, возможно менее чувствительна к изменениям среды      |
| 13    | Heterochromatin  | Компактный неактивный хроматин, структура и молчание                       |
| 14    | Repeat/Artifact       | Повторы или участки без значимых гистоновых модификаций                    |
| 15    | Quiescent  | Неактивная хроматиновая зона, без выраженной функции                       |
