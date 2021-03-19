# TODO

* Маркдаун
* Границы предложений
* Форматы
* https://gist.github.com/gruber/249502 (URL)
* https://daringfireball.net/2010/07/improved_regex_for_matching_urls
* https://m.habr.com/post/429548/ - Плагин «Rainbow CSV» как альтернатива Excel (см детектирование разделителя значений полей)
* https://m.habr.com/post/343116/ - Как я написал приложение, которое за 15 минут делало то же самое, что и регулярн...
* https://www.regular-expressions.info/examplesprogrammer.html
* https://www.regular-expressions.info/examples.html?wlr=1
* https://regular-expressions.mobi/creditcard.html?wlr=1
* https://github.com/vapniks/regex-collection/blob/master/regex-collection.sh
* https://github.com/dukei/any-balance-providers/blob/master/modules/ab/source/vendor/string.js
* https://docs.oracle.com/javase/1.5.0/docs/api/java/util/regex/Pattern.html
* https://github.com/rin-nas/php5-utf8/blob/master/UTF8.php
* https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS
* https://regex101.com/r/sEGPFE/4/ - захват хоста в тексте для обнаружения ссылок
* https://regex101.com/r/I5In0d/1/ - захват русской и английской буквы (и в обратном порядке) для обнаружения текста на кириллице и латинице


JSON
* https://github.com/rin-nas/php-json/blob/master/JSON.php
* http://seriot.ch/parsing_json.php
* https://m.habr.com/ru/company/mailru/blog/314014/ Парсинг JSON — это минное поле
* https://stackoverflow.com/questions/2583472/regex-to-validate-json
* https://codegolf.stackexchange.com/questions/474/write-a-json-validator

```
Тип данных CIDR (бесклассовая маршрутизация интернет домена, Classless Internet Domain Routing) следует соглашению для сетевых адресов IPv4 и IPv6. Вот несколько примеров:
192.168.100.128/25
10.1.2.3/32
2001:4f8:3:ba:2e0:81ff:fe22:d1f1/128
::ffff:1.2.3.0/128
 Тип данных MACADDR может использоваться для хранения MAC-адресов для идентификации оборудования, таких как 08-00-2b-01-02-03.
```

http://blog.stevenlevithan.com/archives/parseuri

```js
// parseUri 1.2.2
// (c) Steven Levithan <stevenlevithan.com>
// MIT License

function parseUri (str) {
	var	o   = parseUri.options,
		m   = o.parser[o.strictMode ? "strict" : "loose"].exec(str),
		uri = {},
		i   = 14;

	while (i--) uri[o.key[i]] = m[i] || "";

	uri[o.q.name] = {};
	uri[o.key[12]].replace(o.q.parser, function ($0, $1, $2) {
		if ($1) uri[o.q.name][$1] = $2;
	});

	return uri;
};

parseUri.options = {
	strictMode: false,
	key: ["source","protocol","authority","userInfo","user","password","host","port","relative","path","directory","file","query","anchor"],
	q:   {
		name:   "queryKey",
		parser: /(?:^|&)([^&=]*)=?([^&]*)/g
	},
	parser: {
		strict: /^(?:([^:\/?#]+):)?(?:\/\/((?:(([^:@]*)(?::([^:@]*))?)?@)?([^:\/?#]*)(?::(\d*))?))?((((?:[^?#\/]*\/)*)([^?#]*))(?:\?([^#]*))?(?:#(.*))?)/,
		loose:  /^(?:(?![^:@]+:[^:@\/]*@)([^:\/?#.]+):)?(?:\/\/)?((?:(([^:@]*)(?::([^:@]*))?)?@)?([^:\/?#]*)(?::(\d*))?)(((\/(?:[^?#](?![^?#\/]*\.[^?#\/.]+(?:[?#]|$)))*\/?)?([^?#\/]*))(?:\?([^#]*))?(?:#(.*))?)/
	}
};

```

```
SELECT to_char(last_event_at, 'YYYY-MM-DD') as calls_date, COUNT(*) AS cnt
FROM cts__cdr
WHERE last_event_at > '2021-01-01'
GROUP BY calls_date
ORDER BY calls_date ASC
limit 100;

SELECT last_event_at::date as calls_date, COUNT(*) AS cnt
FROM cts__cdr
WHERE last_event_at > '2021-01-01'
GROUP BY calls_date
ORDER BY calls_date ASC
limit 100;
```
