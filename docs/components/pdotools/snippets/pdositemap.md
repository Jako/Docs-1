# pdoSitemap

Быстрый сниппет генерации карты сайта для поисковых систем (sitemap.xml). Понимает параметры сниппета [GoogleSitemap][1] (конвертирует в собственные) и легко его заменяет.

Главная особенность - очень высокая скорость, по сравнению с аналогом. На сайте [bezumkin.ru][2] при 1700 страниц скорость генерации возросла в ***12 раз***, с 8.4 сек. до 0.7.

По умолчанию отключена проверка прав на доступ к документам. Это легко исправить включением параметра **&checkPermissions** (Внимание: замедляет работу!):

``` modx
[[!pdoSitemap?
    &checkPermissions=`list`
]]
```

По возможности ресурсы лучше исключать из карты параметрами сниппета.

## Параметры

*pdoSitemap* принимает все параметры [pdoTools][3] и некоторые свои:

| Параметр           | По умолчанию                                    | Описание                                                                                                           |
|--------------------|-------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| **&sitemapSchema** | `<http://www.sitemaps.org/schemas/sitemap/0.9>` | Схема карты сайта.                                                                                                 |
| **&forceXML**      | `1`                                             | Принудительно выводить страницу как XML.                                                                           |
| **&priorityTV**    |                                                 | Дополнительное поле, в котором указывается [приоритет][4] документа. Нужно добавить его в параметр **&includeTVs** |

### Шаблоны

> &tpl

``` xml
@INLINE <url>\n\t
<loc>[[+url]]</loc>\n\t
<lastmod>[[+date]]</lastmod>\n\t
<changefreq>[[+update]]</changefreq>\n\t
<priority>[[+priority]]</priority>\n
</url>
```

> &tplWrapper

``` xml
@INLINE <?xml version=\"1.0\" encoding=\"[[++modx_charset]]\"?>\n<urlset xmlns=\"[[+schema]]\">\n[[+output]]\n</urlset>
```

Приоритет и [частота обновления][5] для поисковиков устанавливаются в зависимости от последней даты изменения документа:

| Времени с последнего обновления документа | Приоритет | Частота обновления |
|-------------------------------------------|-----------|--------------------|
| Менее суток назад                         | 1.0       | daily              |
| Более суток и менее недели назад          | 0.75      | weekly             |
| Более недели и менее месяца назад         | 0.5       | weekly             |
| Более месяца назад                        | 0.25      | monthly            |

### Инструкция по созданию файла sitemap.xml

1. Создайте новый документ в корне сайта. На вкладке ***Документ*** выберите пустой шаблон, укажите Заголовок документа (не важно какой), и псевдоним ***sitemap***. Проверьте, чтобы стояли галочки «Опубликован» и «Не показывать в меню».
2. Перейдите на вкладку ***Настройки*** и укажите «Тип содержимого» - «XML».
3. Уберите галочку с «Использовать HTML-редактор» и сохраните документ.
4. В содержимом ресурса вызовите только сниппет pdoSitemap (см. Примеры ниже).

## Примеры

Обычный вывод карты сайта для текущего контекста. В большинстве случаев этого может быть достаточно:

``` modx
[[pdoSitemap]]
```

Генерируем карту сайта только из определённый контейнеров:

``` modx
[[pdoSitemap?
    &parents=`10`
]]
```

Исключаем ресурсы с id = 15 и 25, вместе с их потомками:

``` modx
[[pdoSitemap?
    &parents=`10, -15,-25`
]]
```

А теперь исключаем id = 15 с потомками, а 25 - без:

``` modx
[[pdoSitemap?
    &resources=`-25`
    &parents=`-15,10`
]]
```

Добавляем еще один контекст:

``` modx
[[pdoSitemap?
    &resources=`-25`
    &parents=`-15,10`
    &context=`web,catalog`
]]
```

А вот так можно посмотреть лог выборки карты:

``` modx
[[pdoSitemap?
    &resources=`-25`
    &parents=`-15,10`
    &context=`web,catalog`
    &showLog=`1`
    &forceXML=`0`
]]
```

[1]: http://rtfm.modx.com/extras/revo/googlesitemap
[2]: http://bezumkin.ru/sitemap.xml
[3]: /components/pdotools/general-parameters
[4]: http://www.sitemaps.org/ru/protocol.html#prioritydef
[5]: http://www.sitemaps.org/ru/protocol.html#changefreqdef