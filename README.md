# Регулярные выражения

## Диалекты
* [PCRE](http://www.pcre.org/current/doc/html/pcre2syntax.html), применяется в [PHP](http://php.net/) и др. ПО
* [ECMA 262](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions), применяется в [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript) (JS)
* есть и [другие](https://en.wikipedia.org/wiki/Comparison_of_regular_expression_engines) достойные внимания, но в этом документе не рассматриваются

## Методы обработки
Метод обработки|JavaScript|PHP
---------------|----------|---
Проверка соответствия (валидация)| [`RegExp.test()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test)|[`preg_match()`](http://php.net/manual/function.preg-match.php), [`preg_grep()`](http://php.net/manual/function.preg-grep.php)
Поиск и захват подстрок| [`RegExp.exec()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec), [`String.match()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match), [`String.search()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/search)|[`preg_match_all()`](http://php.net/manual/function.preg-match-all.php)
Поиск, захват и замена подстрок| [`String.replace()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace)|[`preg_replace()`](http://php.net/manual/en/function.preg-replace.php), [`preg_replace_callback()`](http://php.net/manual/function.preg-replace-callback.php), [`preg_replace_callback_array()`](http://php.net/manual/function.preg-replace-callback-array.php), [`preg_filter()`](http://php.net/manual/function.preg-filter.php)
Поиск и разбиение на подстроки| [`String.split()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split)|[`preg_split()`](http://php.net/manual/function.preg-split.php)

## Виды соответствия
* Полное — вся строка соответствует шаблону регулярного выражения, которое всегда начинается с `^` и заканчивается на `$`.
* Частичное — часть строки соответствует шаблону регулярного выражения. Наличие символов `^` и `$` опционально.

Из частичного соответствия очень легко сделать полное, указав `^` в начале и `$` в конце. В обратную сторону — сложнее, т.к.  при необходимости необходимо явно указывать границы начала и конца подстроки. Обычно это делают через `(?<!…)` и `(?!…)` соответственно

## Почему регулярные выражения работают медленно?

Главная проблема медлительности — многочисленные откаты в случае неудачи в одной из веток поиска, которые могут привести к превышению максимального времени выполнения. Подробнее см. [Catastrophic Backtracking](https://www.regular-expressions.info/catastrophic.html). 
* Пример долго выполняющегося рег. выражения [PCRE](https://regex101.com/r/jye7NW/3) и его оптимизированной версии, работающей почти в 10 раз быстрее: [PCRE](https://regex101.com/r/A9qf3S/1)

## Как писать быстро работающие регулярные выражения?

1. Используйте [атомарную группировку](https://www.regular-expressions.info/atomic.html) `(?>…)` и/или [однократные квантификаторы](https://www.regular-expressions.info/possessive.html): `…?+`, `…*+`, `…++`. В диалекте JS нет такой встроенной возможности, но можно использовать конструкцию типа `(?=(…))\1`, которая является аналогом.
2. Не используйте вложенные циклы типа `(…+)+`. Используйте конструкции типа `(?!…)(…)+`, `(?=…)(…)+`, `(…)+(?<!…)`, `(…)+(?<=…)`.
3. В случае отсутствия `^` задавайте явно границу начала совпадения через `\b` или `(?<!…)`. В случае отсутствия `$` задавайте явно границу совпадения через `\b` или `(?!…)`.
4. Замените нежадные квантификаторы типа `.*?` на жадные. Но часть рег. выражения нужно будет немного переписать и усложнить ради увеличения скорости.

## Советы
1. Регулярные выражения PCRE пока поддерживают только фиксированный просмотр назад. Т.е. внутри `(?<!…)` нельзя использовать
квантификаторы `?`, `*`, `+`, `{…,…}`. Но есть способ обойти это ограничение, используя [`\K`](http://www.pcre.org/current/doc/html/pcre2syntax.html#SEC11).
2. Регулярные выражения JS пока не поддерживают всех возможностей PCRE, но есть готовые решения:
    * Библиотека [`XRegExp`](https://github.com/slevithan/xregexp), которая расширяет стандартные возможности рег. выражений в JS до уровня PCRE.
    * `String.prototype.matchRecursive()` — реализация JavaScript метода [`String.match()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match) с учётом рекурсии: [JS Fiddle](https://jsfiddle.net/zqta1481/14/)
    * Генератор регулярного выражения до заданной глубины рекурсии `(?R)` (TODO сделать форму ввода): [JS Fiddle](https://jsfiddle.net/rea4sxgn/)
3. Используйте сервис [`regex101.com`](https://regex101.com/r/iB63bg/2/) для написания и тестирования ваших регурярных выражений, а так же для поиска и замены подстрок через регулярные выражения в диалектах PRCE, JS, Python, Golang.


## Склад готовых регулярных выражений

Каждое регулярное выражение написано так, чтобы оно
* работало максимально быстро (по советам выше)
* было читабельным с выделением логических частей (используется флаг `/x`) и описанием в комментариях (`#`)

### Числа
* Целые: `/^-?+\d++$/sSX`
* Целые и десятичные: `/^-?+\d++(?>\.\d+)?$/sSX`
* Денежный формат: `/^-?+\d++(?>\.\d{1,2})?$/sSX`

### Дата и время

#### нестрогая валидация
* Дата в формате `ГГГ-ММ-ДД`: `/^\d{4}-\d{2}-\d{2}$/sSX`
* Время в формате `ЧЧ:ММ:СС`: `/^\d{2}:\d{2}(:\d{2})?$/sSX`


### Форматы
* Валидация IPv4 + IPv6: [PCRE](https://regex101.com/r/eVEGRY/1/), [link](https://stackoverflow.com/questions/4460586/javascript-regular-expression-to-check-for-ip-addresses/26445549#26445549). В языках программирования есть готовый валидатор: PHP —  [`filter_var()`](http://php.net/manual/en/function.filter-var.php), NodeJS — [`net.isIP()`](https://nodejs.org/api/net.html#net_net_isip_input).
* Захват Email из текста: [PCRE](https://regex101.com/r/Q4dsL5/13)
* Валидация даты в формате ГГГГ-ММ-ДД без проверки YYYY-02-29 в високосных годах (нужно доделать): [PCRE](https://regex101.com/r/WnauVT/6/)
* Захват многострочных комментариев (в 2 раза эффективнее, чем `/\* .*? \*/`): [PCRE](https://regex101.com/r/r2ESLq/2/)

### HTML
* Захват [xml и html сущностей](https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references): [PCRE](https://regex101.com/r/xhNZc5/4/), [JS](https://regex101.com/r/N5GFsv/1/) (strict); [PCRE](https://regex101.com/r/0OOLyS/1) ([non strict](https://stackoverflow.com/questions/15532252/why-is-reg-being-rendered-as-without-the-bounding-semicolon)).
См. так же список именованных сущностей в [JSON](https://www.w3.org/TR/html5/entities.json).
* Захват заданных html тегов с учётом их парности без вложенных таких же тегов: [PCRE](https://regex101.com/r/JVzBz2/3), [JS](https://regex101.com/r/CvlwKz/1)
* Захват заданных html тегов с учётом их парности и вложенности друг в друга (рекурсивно): [PCRE](https://regex101.com/r/jwH6O2/4), [JS](https://regex101.com/r/IVSo1x/1)
* Вставляет теги `</li>` там, где они пропущены (чинит некорректную вложенность тегов `<li>`): [PCRE](https://regex101.com/r/PFEWtT/7/)

### Обработка естественного языка
* Валидация слова на английском языке во множественном числе: [PCRE](https://regex101.com/r/GtF2QA/8/)
* Валидация ФИО на русском или английском языке: [PCRE](https://regex101.com/r/GQ1xKK/14/)

### SQL
* Детектирование SQL на модификацию данных: [PCRE](https://regex101.com/r/CcSugS/14)
* Удаление комментариев из SQL: [PCRE](https://regex101.com/r/SjGwyh/3)
* Разбиение SQL на несколько запросов по символу `;`: [PCRE](https://regex101.com/r/z4WOwx/1/)

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

* Захват только непустых значений колонок: [PCRE](https://regex101.com/r/ZRP2iA/1)

### Детектирование бинарных данных

Ищем до первого найденного [управляющего символа](https://unicode-table.com/ru/#control-character) по регулярному выражению `/[\x00-\x1f](?<![\x08\x09\x0c\x0a\x0d])/` (одинаково в диалектах PCRE и JS)

[JSON](http://json.org/) не должен определяться как бинарный, поэтому исключаем все специальные символы по спецификации:
* `\b` represents the backspace character `U+0008`
* `\t` represents the character tabulation character `U+0009`
* `\f` represents the form feed character `U+000C`
* `\n` represents the line feed character `U+000A`
* `\r` represents the carriage return character `U+000D`

## TODO

* Маркдаун,
* Границы предложений
* Форматы
* https://gist.github.com/gruber/249502 (URL)
* https://daringfireball.net/2010/07/improved_regex_for_matching_urls
* https://m.habr.com/post/429548/ - Плагин «Rainbow CSV» как альтернатива Excel
* https://m.habr.com/post/343116/ - Как я написал приложение, которое за 15 минут делало то же самое, что и регулярн...
* https://www.regular-expressions.info/examplesprogrammer.html
* https://www.regular-expressions.info/examples.html?wlr=1
* https://regular-expressions.mobi/creditcard.html?wlr=1
* https://github.com/vapniks/regex-collection/blob/master/regex-collection.sh
* https://github.com/dukei/any-balance-providers/blob/master/modules/ab/source/vendor/string.js

```
Тип данных CIDR (бесклассовая маршрутизация интернет домена, Classless Internet Domain Routing) следует соглашению для сетевых адресов IPv4 и IPv6. Вот несколько примеров:
192.168.100.128/25
10.1.2.3/32
2001:4f8:3:ba:2e0:81ff:fe22:d1f1/128
::ffff:1.2.3.0/128
 Тип данных MACADDR может использоваться для хранения MAC-адресов для идентификации оборудования, таких как 08-00-2b-01-02-03.
```
