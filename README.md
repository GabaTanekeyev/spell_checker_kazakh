# spell_checker_kazakh
spell_checker
# JamSpellKazakh

[![Build Status][travis-image]][travis] [![Release][release-image]][releases]

[travis-image]: https://travis-ci.org/bakwc/JamSpell.svg?branch=master
[travis]: https://travis-ci.org/bakwc/JamSpell

[release-image]: https://img.shields.io/badge/release-0.0.11-blue.svg?style=flat
[releases]: https://github.com/bakwc/JamSpell/releases

JamSpellKazakh - это библиотека для проверки орфографии со следующими функциями:

- **точность** - учитывает окружение слов (контекст) для лучшей коррекции
- **скорость** - около 5 тыс. слов в секунду
- **мульти-язычность** - написан на C ++ и доступен для многих языков с привязками swig

## Содержание
- [Benchmarks](#benchmarks)
- [Использование](#Использование)
  - [Python](#python)
- [Обучение](#обучение)

## Benchmarks

<table>
  <tr>
    <td></td>
    <td>Errors</td>
    <td>Top 7 Errors</td>
    <td>Fix Rate</td>
    <td>Top 7 Fix Rate</td>
    <td>Broken</td>
    <td>Speed<br>
(words/second)</td>
  </tr>
  <tr>
    <td>JamSpell</td>
    <td>4.27%</td>
    <td>1.59%</td>
    <td>78.75%</td>
    <td>83.87%</td>
    <td>0.58%</td>
    <td>250474</td>
  </tr>
  <tr>
    <td>Dummy</td>
    <td>17.84%</td>
    <td>17.84%</td>
    <td>0.00%</td>
    <td>0.00%</td>
    <td>0.00%</td>
    <td>-</td>
  </tr>
</table>

Модель была обучена на 3 млн предложениях казахской литературы + 1.5 млн предложениях из новостных статей (казахский). 95% было использовано для обучения, 5% было использовано для оценки модели. Для формирования тестового текста из оригинального была использована [модель совершения ошибок](https://github.com/bakwc/JamSpell/blob/master/evaluate/typo_model.py). JamSpell была сравнена с моделью dummy (определение ошибок).

Мы использовали следующие метрики:
- **Errors** - процент слов с ошибками после проверки орфографии
- **Top 7 Errors** - процент слов отсутствует в топ-7 кандидатов
- **Fix Rate** - процент ошибочных слов, исправленных средством проверки правописания
- **Top 7 Fix Rate** - процент ошибочных слов, зафиксированных одним из 7 лучших кандидатов
- **Broken** - процент слов без ошибок, некоректно исправленных средством проверки правописания
- **Speed** - количество слов в секунду

Чтобы убедиться, что наша модель не направлена только на новости и казахскую литературу, мы проверили ее на статье из wikipedia - «Шерлок Холмс»:

<table>
  <tr>
    <td></td>
    <td>Errors</td>
    <td>Top 7 Errors</td>
    <td>Fix Rate</td>
    <td>Top 7 Fix Rate</td>
    <td>Broken</td>
    <td>Speed
(words per second)</td>
  </tr>
  <tr>
    <td>JamSpell</td>
    <td>9.32%</td>
    <td>2.48%</td>
    <td>70.95%</td>
    <td>78.21%</td>
    <td>4.83%</td>
    <td>2994</td>
  </tr>
  <tr>
    <td>Dummy</td>
    <td>18.53%</td>
    <td>18.53%</td>
    <td>0.00%</td>
    <td>0.00%</td>
    <td>0.00%</td>
    <td>-</td>
  </tr>
</table>

Подробнее о процедуре обучения можно узнать в разделе "[Обучение](#обучение)".

## Использование
### Python
1. Необходимо устновить ```swig3``` (Мы установили из стандартного репозитория командой ```apt-get install swig```)

2. Необходимо устновить ```jamspell```:
```bash
pip install jamspell
```
3. [Скачать](#download-models) или [обучить](#обучение) модель

4. Использование:

```python
import jamspell

corrector = jamspell.TSpellCorrector()
corrector.LoadLangModel('en.bin')

corrector.FixFragment('базалары ман қоймалары жанындағы тыйм салынған аймақтрды жәе қарулы күштердвің')
# u'базалары мен қоймалары жанындағы тыйым салынған аймақтарды және қарулы күштердің'

corrector.GetCandidates(['базалары', 'ман', 'қоймалары', 'жанындағы', 'тыйм', 'салынған', 'аймақтрды', 'жәе', 'қарулы', 'күштердвің'], 1)
# (u'мен', u'май', u'ма', u'жан', u'қан', ... )

corrector.GetCandidates(['базалары', 'ман', 'қоймалары', 'жанындағы', 'тыйм', 'салынған', 'аймақтрды', 'жәе', 'қарулы', 'күштердвің'], 4)
# (u'тыйым', u'тый', u'тым', u'тыйм', u'тыям', u'тыйма', u'түйм', u'тыйу', ...)
```

### Другие языки
Вы можете создавать расширения для других языков, используя [swig tutorial](http://www.swig.org/tutorial.html). Файл интерфейса swig - `jamspell.i`. 

## Обучение
Для обучения пользовательской модели вам необходимо:

1. Установить ```cmake``` (```apt-get install cmake```)

2. Склонировать и построить jamspell:
```bash
git clone https://github.com/bakwc/JamSpell.git
cd JamSpell
mkdir build
cd build
cmake ..
make
```

3. Подготовить текстовый файл с расширением utf-8 для обучения модели (пр. [```text_for_train.txt```](https://github.com/bakwc/JamSpell/blob/master/test_data/sherlockholmes.txt)) и дополнительный текстовый файл с алфавитом (пр. [```alphabet_kz.txt```](https://github.com/bakwc/JamSpell/blob/master/test_data/alphabet_en.txt))

4. Обучение модели:
```bash
./main/jamspell train ../test_data/alphabet_kz.txt ../test_data/text_for_train.txt model_kaz.bin
```
5. Чтобы проверить модель можно использовать скрипт ```evaluate/evaluate.py```:
```bash
python evaluate/evaluate.py -a alphabet_kz.txt -jsp model_kaz.bin -mx 50000 test_data.txt
```
6. Вы можете использовать ```evaluate/generate_dataset.py``` чтобы создать тестовую и обучающую выборку. Он поддерживает TXT-файлы, формат [Leipzig Corpora Collection](http://wortschatz.uni-leipzig.de/en/download/) fb2 книги.

## Модели
Первоисточник и правообладатель библиотеки [jamspell](https://github.com/bakwc/JamSpell), там вы можете ознакомиться с более подробным сравнительным анализом/описанием использования модели на других языках.
