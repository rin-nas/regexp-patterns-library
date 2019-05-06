# Регулярные выражения

## Когда применять, а когда не применять регулярные выражения?

Если вы пишите прототип, или если скорость работы не принципиальна — используйте регулярные выражения. Это ускорит скорость разработки и уменьшит количество кода. Но, если итоговое решение будет работать медленно, то его всегда можно будет переписать без использования регулярных выражений.

## Диалекты
* [PCRE](http://www.pcre.org/current/doc/html/pcre2syntax.html), применяется в [PHP](http://php.net/) и др. ПО
* [ECMA 262](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Regular_Expressions), применяется в [JavaScript](https://developer.mozilla.org/docs/Web/JavaScript) (JS)
* есть и [другие](https://en.wikipedia.org/wiki/Comparison_of_regular_expression_engines) достойные внимания, но в этом документе не рассматриваются

## Функции и методы объектов для обработки строк регулярными выражениями в языках программирования
Тип обработки|JavaScript|PHP
-------------|----------|---
Проверка соответствия (валидация)| [`RegExp.test()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test)|[`preg_match()`](http://php.net/manual/function.preg-match.php), [`preg_grep()`](http://php.net/manual/function.preg-grep.php)
Поиск и захват подстрок| [`RegExp.exec()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec), [`String.match()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String/match), [`String.search()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String/search)|[`preg_match_all()`](http://php.net/manual/function.preg-match-all.php)
Поиск, захват и замена подстрок| [`String.replace()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String/replace)|[`preg_replace()`](http://php.net/manual/en/function.preg-replace.php), [`preg_replace_callback()`](http://php.net/manual/function.preg-replace-callback.php), [`preg_replace_callback_array()`](http://php.net/manual/function.preg-replace-callback-array.php), [`preg_filter()`](http://php.net/manual/function.preg-filter.php)
Поиск и разбиение на подстроки| [`String.split()`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String/split)|[`preg_split()`](http://php.net/manual/function.preg-split.php)

## Виды соответствия
* Полное — вся строка соответствует шаблону регулярного выражения, которое всегда начинается с `^` и заканчивается на `$`.
* Частичное — часть строки соответствует шаблону регулярного выражения. Наличие символов `^` и `$` опционально.

Полное соответствие обычно используют для [валидации](https://en.wikipedia.org/wiki/Data_validation) (проверки) строк на соответствие или не соответствие регулярному выражению. А частичное — для [синтаксического анализа](https://ru.wikipedia.org/wiki/Синтаксический_анализ) (парсинга) и [лексического анализа](https://ru.wikipedia.org/wiki/Лексический_анализ) (токенизации) строк.

Из частичного соответствия очень легко сделать полное, указав `^` в начале и `$` в конце. В обратную сторону — сложнее, т.к.  при необходимости необходимо явно указывать границы начала и конца искомой подстроки. Обычно это делают через `(?<!…)` и `(?!…)` соответственно.

## Почему регулярные выражения работают медленно?

Главная проблема медлительности — многочисленные откаты в случае неудачи в одной из веток поиска, которые могут привести к превышению максимального времени выполнения. Подробнее см. [Catastrophic Backtracking](https://www.regular-expressions.info/catastrophic.html). 
* Пример долго выполняющегося регулярного выражения [PCRE](https://regex101.com/r/jye7NW/3) и его оптимизированной версии, работающей почти в 10 раз быстрее: [PCRE](https://regex101.com/r/A9qf3S/1)

## Как писать быстро работающие регулярные выражения?

1. Используйте [атомарную группировку](https://www.regular-expressions.info/atomic.html) `(?>…)` и/или [однократные квантификаторы](https://www.regular-expressions.info/possessive.html): `…?+`, `…*+`, `…++`. В диалекте JS нет такой встроенной возможности, но можно использовать конструкцию типа `(?=(…))\1`, которая является аналогом.
2. Не используйте вложенные циклы типа `(…+)+`. Используйте конструкции типа `(?!…)(…)+`, `(?=…)(…)+`, `(…)+(?<!…)`, `(…)+(?<=…)`.
3. В случае отсутствия `^` задавайте явно границу начала совпадения через `\b` или `(?<!…)`. В случае отсутствия `$` задавайте явно границу совпадения через `\b` или `(?!…)`.
4. Замените нежадные квантификаторы типа `.*?` на жадные. Но часть рег. выражения нужно будет немного переписать и усложнить ради увеличения скорости.
5. Не используйте флаг `/u` (юникод в PCRE). Возможно, часть рег. выражения нужно будет немного переписать и усложнить ради увеличения скорости.

## Что делать, если диалект регулярные выражения не поддерживает нужной мне возможности?
1. Регулярные выражения PCRE пока поддерживают просмотр назад только с фиксированной длиной. Т.е. внутри `(?<!…)` нельзя использовать квантификаторы `?`, `*`, `+`, `{…,…}`. Но есть способ обойти это ограничение, используя [`\K`](http://www.pcre.org/current/doc/html/pcre2syntax.html#SEC11).
2. Регулярные выражения JS пока не поддерживают всех возможностей PCRE, но есть готовые решения:
    * Библиотека [`XRegExp`](https://github.com/slevithan/xregexp), которая расширяет стандартные возможности рег. выражений в JS до уровня PCRE.
    * `String.prototype.matchRecursive()` — реализация JavaScript метода [`String.match()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match) с учётом рекурсии: [JS Fiddle](https://jsfiddle.net/zqta1481/14/)
    * Генератор регулярного выражения до заданной глубины рекурсии `(?R)` (TODO сделать форму ввода): [JS Fiddle](https://jsfiddle.net/rea4sxgn/)

## Какие онлайн сервисы существуют для обработки строк регулярными выражениями?
Используйте сервис [`regex101.com`](https://regex101.com/r/iB63bg/2/) для написания и тестирования ваших регурярных выражений, а так же для поиска и замены подстрок через регулярные выражения в диалектах PRCE, JS, Python, Golang.

## Склад готовых регулярных выражений

Каждое регулярное выражение написано так, чтобы оно
* работало максимально быстро (по советам выше)
* было читабельным с выделением логических частей (используется флаг `/x`) и описанием в комментариях (`#`)

### Числа
* Целые: `/^[-−]? (?:[1-9]\d*|0) $/sx`
* Целые и десятичные: `/^[-−]? (?:[1-9]\d*|0) (?:\.\d+)? $/sx`
* Денежный формат: `/^[-−]? (?:[1-9]\d*|0) (?:\.\d{1,2})? $/sx`
* Число с разделением тысячных через запятую и десятичными: [PCRE](https://regex101.com/r/FH4QLj/4/)

### Дата и время

#### нестрогая валидация
* Дата в формате `ГГГГ-ММ-ДД`: `/^\d{4}-\d{2}-\d{2}$/sSX`
* Время в формате `ЧЧ:ММ:СС`: `/^\d{2}:\d{2}(:\d{2})?$/sSX`

#### строгая валидация
* Валидация даты в формате `ГГГГ-ММ-ДД` с проверкой корректности года, месяца и дня, но без проверки YYYY-02-29 в високосных годах (нужно доделать): [PCRE](https://regex101.com/r/WnauVT/6/)

### Форматы
* Валидация IPv4 + IPv6: [PCRE](https://regex101.com/r/eVEGRY/1/), [link](https://stackoverflow.com/questions/4460586/javascript-regular-expression-to-check-for-ip-addresses/26445549#26445549). В языках программирования есть готовый валидатор: PHP —  [`filter_var()`](http://php.net/manual/en/function.filter-var.php), NodeJS — [`net.isIP()`](https://nodejs.org/api/net.html#net_net_isip_input).
* Захват адреса электронной почты ([Email address](https://en.wikipedia.org/wiki/Email_address)) из текста: [PCRE](https://regex101.com/r/Q4dsL5/14)
* Захват многострочных комментариев (в 2 раза эффективнее, чем `/\* .*? \*/`): [PCRE](https://regex101.com/r/r2ESLq/2/)

### HTML
* Захват [xml и html сущностей](https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references): [PCRE](https://regex101.com/r/xhNZc5/4/), [JS](https://regex101.com/r/N5GFsv/1/) (strict); [PCRE](https://regex101.com/r/0OOLyS/1) ([non strict](https://stackoverflow.com/questions/15532252/why-is-reg-being-rendered-as-without-the-bounding-semicolon)).
См. так же список именованных сущностей в [JSON](https://www.w3.org/TR/html5/entities.json).
* Захват заданных html тегов с учётом их парности без вложенных таких же тегов: [PCRE](https://regex101.com/r/JVzBz2/3), [JS](https://regex101.com/r/CvlwKz/1)
* Захват заданных html тегов с учётом их парности и вложенности друг в друга (рекурсивно): [PCRE](https://regex101.com/r/jwH6O2/4), [JS](https://regex101.com/r/IVSo1x/1)
* Вставляет теги `</li>` там, где они пропущены (чинит некорректную вложенность тегов `<li>`): [PCRE](https://regex101.com/r/PFEWtT/7/)

### Обработка естественного языка
* Удаление начальных и конечных пробелов: [PCRE](https://regex101.com/r/JM7xw4/1)
* Валидация слова на английском языке во множественном числе: [PCRE](https://regex101.com/r/GtF2QA/9/)
* Валидация ФИО на русском или английском языке: [PCRE](https://regex101.com/r/GQ1xKK/17/)
* Выделение слов и чисел из текста [PCRE](https://regex101.com/r/fpu9Gb/2/)

### SQL
* Детектирование SQL на модификацию данных: [PCRE](https://regex101.com/r/CcSugS/14)
* Удаление комментариев из SQL (замена на пробел): [PCRE](https://regex101.com/r/SjGwyh/3)
* Разбиение SQL на несколько запросов по символу `;`: [PCRE](https://regex101.com/r/0A9MoC/5)

### Прочее
* Валидация (неполная, но быстрая) регулярного выражения на диалект ECMA 262 (JavaScript), есть проверка на уникальность флагов: [PCRE](https://regex101.com/r/iB63bg/3/)


### Валидация качества пароля

Регулярное выражение: [PCRE](https://regex101.com/r/MOWCV3/9)

Пароль должен соответствовать следующим требованиям:
1. длина от 8 до 64 символов
2. допустимы только английские буквы, цифры и знаки пунктуации
3. содержит хотябы одну цифру (0-9)
4. содержит хотябы одну английскую прописную букву (A-Z)
5. содержит хотябы одну английскую строчную букву (a-z)
6. содержит хотябы один знак пунктуации: `!"#$%&'()*+,-./:;<=>?@[\]^_``{|}~`
7. не имеют 6 и более подряд совпадающих символов типа "zzzzzz", "000000"
8. не имеет 6 и более подряд "плохих" последовательностей типа "123456", "abcdef", "qwerty"

### Парсинг CSV

* Захват только непустых значений колонок: [PCRE](https://regex101.com/r/ZRP2iA/2)
* Как распарсить CSV строку в таблицу? [PCRE](https://regex101.com/r/3yJVNR/2), [PostgreSQL](https://github.com/rin-nas/postgresql-patterns-library/blob/master/README.md#%D0%BA%D0%B0%D0%BA-%D1%80%D0%B0%D1%81%D0%BF%D0%B0%D1%80%D1%81%D0%B8%D1%82%D1%8C-csv-%D1%81%D1%82%D1%80%D0%BE%D0%BA%D1%83-%D0%B2-%D1%82%D0%B0%D0%B1%D0%BB%D0%B8%D1%86%D1%83)

### Детектирование бинарных данных

Ищем до первого найденного [управляющего символа](https://unicode-table.com/ru/#control-character) по регулярному выражению `/[\x00-\x1f](?<![\x08\x09\x0c\x0a\x0d])/` (одинаково в диалектах PCRE и JS)

[JSON](http://json.org/) не должен определяться как бинарный, поэтому исключаем все специальные символы по спецификации:
* `\b` represents the backspace character `U+0008`
* `\t` represents the character tabulation character `U+0009`
* `\f` represents the form feed character `U+000C`
* `\n` represents the line feed character `U+000A`
* `\r` represents the carriage return character `U+000D`
