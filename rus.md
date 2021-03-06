# Предзагрузка ресурсов в HTML5

## Предугадывание действий пользователя и предзагрузка ресурсов для улучшения производительности

HTML5 предлагает новую и пока относительно неизвестную возможность под названием
«Предварительная загрузка» (*англ. — prefetch*), которая позволяет загружать
ресурсы заранее, чтобы обеспечить как можно более быстрый доступ к сайту без необходимости 
дожидаться загрузки всех запрашиваемых файлов.

Оптимизировать работу современных сайтов можно не только минимизируя
исходный объём загрузки и уменьшая размер загружаемых файлов, также дополнительное ускорение возможно за счёт более раннего начала загрузки нужных ресурсов.

«Предварительная загрузка» подразумевает загрузку файла до того, как он
понадобится, чтобы пользователям не приходилось ждать.
Вы можете выбрать, какой контент подлежит предварительной загрузке,
путем анализа поведения ваших пользователей
и попытаться заранее предугадать какие ресурсы им понадобятся.


## Кэш браузера

Кто-то может возразить: «у нас уже есть кэширование в браузере, нам не нужна
предварительная загрузка!» Однако, как подчеркнул Стив Саудерс (Steve Souders) в
своей замечательной статье [«Предварительный просмотр страниц»][1], кэширование
в браузере не поможет во многих ситуациях:

> **первое посещение** — кэш начинает работать только при повторных
посещениях сайта. У браузера нет возможности кэшировать какие-либо ресурсы
до того, как вы зайдёте на сайт в первый раз.

> **очистка** — кэш очищается чаще, чем вы думаете. В дополнение к
периодическим очисткам кеша самим пользователем, он также может очищаться
антивирусным ПО и вследствие ошибок в браузере. (Кэш [19% пользователей
Chrome][2] очищается, по крайней мере, раз в неделю из-за ошибки в браузере.)

> **замещение** — поскольку хранилище кэша общее для всех сайтов,
посещаемых пользователем, ресурсы одного сайта могут быть удалены, чтобы освободить место для ресурсов
другого.

> **истечение срока годности** — [у 69% загружаемых ресурсов нет
заголовков управления кэшем или они добавляются в кэш на
короткий промежуток времени][3]. Если пользователь повторно заходит
на такие страницы, и браузер определяет, что у ресурса истёк
срок годности, требуется HTTP-запрос для проверки обновлений. Даже если
согласно полученному ответу кэшированный ресурс всё ещё актуален, такие сетевые
задержки замедляют загрузку страниц, особенно на мобильных устройствах.

> **изменения** — даже если ресурсы сайта хранятся в кэше с предыдущего
посещения, вполне возможно, что на сайт были внесены изменения, и теперь используются
другие ресурсы.


## Прогнозирующий алгоритм Chrome

Собственно, Chrome уже делает попытки предсказать ваше поведение используя
историю браузера, эвристические алгоритмы и другие подсказки, чтобы
предугадать ваши запросы. Когда вы с утра открываете браузер, Chrome знает
какие сайты вы обычно посещаете и выполняет DNS-предзагрузку по именам хостов
(подробнее об этом чуть позже), кроме того, когда вы начинаете набирать
в поисковой строке что-то вроде «amaz», Chrome понимает, что вы, вероятно,
отправитесь на amazon.com и опять-таки начинает DNS-предзагрузку.

Если прогнозирующий алгоритм браузера уверен в своем
прогнозе, он даже может запустить предварительную отрисовку страницы, эта
функция получила название «мгновенные страницы» (instant pages).

<iframe width="560" height="315" src="https://www.youtube.com/embed/_Jn93FDx9oI" frameborder="0" allowfullscreen></iframe>

*[https://www.youtube.com/watch?feature=player_embedded&v=_Jn93FDx9oI][4]*

Chrome изучает топологию сети и ваши поведенческие паттерны
при посещении сайтов. Если ему удаётся справиться со своей работой,
он может избавить пользователя от сотен миллисекунд задержек при
переходе между страницами и приблизиться к святому граалю «мгновенной
загрузки страниц».

**Итак, если браузер делает за нас всё это, что ещё можно сделать, чтобы
улучшить его прогнозы и повысить производительность сайтов?**


## DNS-предзагрузка

Разрешение DNS-имен — это процесс, который выполняет браузер, чтобы конвертировать
домен/имя хоста в ip-адрес, требуемый для получения доступа к ресурсу (в ходе
этого процесса выполняется конвертация понятного пользователю адреса вроде
[http://www.medium.com][5] в [http://80.72.139.101][6]); он занимает некоторое
время и влияет на скорость загрузки страницы. Обычно поиск ip по имени занимает
60-120 мс., после чего следует распространение в прямом и обратном направлении
для осуществления «TCP-рукопожатия», которое может добавить еще 100-200 мс. к
отправке запроса на получение файла. На мобильных устройствах всё ещё дольше, 
частично из-за большей длительности этого цикла, которая составляет 200-1000 мс.

DNS-предзагрузка является безопасной, обратно-совместимой возможностью HTML5,
которая может положительно влиять на ощущаемую производительность. Особенно это
касается страниц, для которых загружается динамический контент с
разных доменов.

Предварительное разрешение/загрузка имён — это процесс определения IP-адреса
каждого ресурса на странице до того, как браузер сделает на него запрос. Его
целью является экономия времени на разрешении DNS-имени, когда запрос на ресурс
отправлен.

Если просмотреть исходный код amazon.com, вы увидите в верхней части главной
страницы следующий код:

    <meta http-equiv='x-dns-prefetch-control' content='on'>
    <link rel='dns-prefetch' href='http://g-ecx.images-amazon.com'>
    <link rel='dns-prefetch' href='http://z-ecx.images-amazon.com'>
    <link rel='dns-prefetch' href='http://ecx.images-amazon.com'>
    <link rel='dns-prefetch' href='http://completion.amazon.com'>
    <link rel='dns-prefetch' href='http://fls-na.amazon.com'>

Amazon.com использует DNS-предзагрузку для разрешения **нескольких доменных имен**,
с которых загружаются различные ресурсы вроде изображений, JavaScript
и CSS-файлов. Когда браузер встречает эти адреса, он в первую очередь
проверяет кэш и затем, если в кэше нет нужных ресурсов, конвертирует доменное
имя в соответствующий IP-адрес через запрос от DNS-сервера. Эти запросы
выполняются в фоновом режиме, чтобы избежать блокировки отрисовки страницы.

Как мы уже видели, в некоторых случаях Chrome делает это автоматически, но
даже и тогда мы можем помочь ему, сделав подсказки конкретно для нашего сайта.

DNS-запросы очень экономны — по сети отсылается всего лишь несколько сотен
байтов, так что риск невелик.

Заглянув под капот некоторых сайтов, я обнаружил что [net-a-porter][7],
[airbnb][8] и [csswizardry][9] также используют эту возможность.


## Предзагрузка ссылок

_Примечание Frontender Magazine: `rel="subresource"`
[более не поддерживается][rel-subresource-deprecation] ни одним браузером,
используйте вместо него `rel="preload"`._

От команды [MDN][10]:

> Предзагрузка ссылок — это алгоритм браузера, использующий периоды простоя
браузера для загрузки или предзагрузки документов, которые пользователь может
посетить в ближайшем будущем. Сайт предлагает браузеру несколько подсказок по
предзагрузке, и после окончания загрузки страницы браузер начинает незаметно
выполнять предзагрузку указанных документов, а затем сохраняет их в кэш. Когда
пользователь посещает один из таких предзагруженных документов,
документ может быть быстро получен из кэша браузера.

Это означает, что если мы уверены, что пользователь предпримет определённое
действие, мы можем заранее начать загрузку важных ресурсов, чтобы помочь
страницам открываться мгновенно.

Для Chrome мы можем указать важные ресурсы для каждой страницы, а также какие
файлы мы хотим предложить для предзагрузки для следующей:

    <link rel='subresource' href='critical.js'>
    <link rel='subresource' href='main.css'>

    <link rel='prefetch' href='secondary.js'>

Значение `subresource` атрибута `rel` указывает на критические ресурсы
для текущей страницы, **эти ссылки должны быть размещены
в верхней части страницы как можно выше и работать как средство ручной
корректировки для сканера предварительной загрузки в Chrome**.
Значение `prefetch` говорит браузеру, что нужно начать загружать файл
`secondary.js` когда он закончит со всеми критическими
подресурсами текущей страницы, при этом подсказки с `prefetch`
имеют минимальный приоритет.

Оптимальное время инициации предзагрузки зависит от установленного алгоритма
действий, текущего профиля подключения пользователя, доступных ресурсов
устройства и других переменных, зависящих от контекста. В результате это
решение делегируется браузеру пользователя.

Предзагружаться должны только те элементы, которые могут быть кэшированы.


## Предварительная отрисовка

Эта возможность на данный момент работает только в Chrome. Вместо того, чтобы
просить браузер загрузить файл, мы даём ему команду
загрузить и отрисовать целую страницу, однако она не показывается
пользователю до момента, пока она не потребуется. Это секретный
ингредиент функции мгновенной загрузки страниц от Google.

**Предварительная отрисовка — это сложная экспериментальная функция, и её
неверное использование может создать неудобства вашим
пользователям**, в том числе увеличить нагрузку на сеть, замедлить загрузку
других ссылок и обновление контента. Использовать её следует только
если вы практически полностью уверены в том, какую страницу
пользователь посетит следующей, и если это действительно
принесёт вашим посетителями реальную пользу.

Запустить предварительную отрисовку можно добавив элемент `link` со значением
`prerender` для атрибута `rel`, например:

    <link rel='prerender' href='http://www.pagetoprerender.com'>

Принцип работы состоит в том, что страница предварительно
отрисовывается в новой вкладке, однако она скрыта, пока
пользователь не запросит эту страницу. Так как браузер выполняет все скрипты на
предварительно отрисовываемой странице, это может привести к зависаниям
на текущей странице, счётчики зафиксируют просмотр страницы,
а также будут посланы запросы на контент, который пользователь может
никогда не увидеть.

Когда Chrome встречает такой элемент `<link>`, он рассматривает возможность
предварительной отрисовки заданной страницы. Это может произойти в
процессе парсинга страницы или позже, если элемент `<link>` вставляется на
страницу динамически с помощью JavaScript.


## Динамическая вставка подсказок по предзагрузке

Вы можете добавлять на страницу подсказки по предварительной загрузке,
когда становится очевидно, что действие пользователя потребует загрузки
дополнительного контента:

    var hint = document.createElement("link");
    hint.setAttribute(“rel”, ”prerender”);
    hint.setAttribute(“href”, ”next-page.html”);
     
    document.getElementsByTagName(“head”)[0].appendChild(hint);


## Поддержка браузерами

![Поддержка браузерами][Поддержка браузерами]

*Internet Explorer 9 поддерживает DNS-предзагрузку, но называет её просто
предзагрузкой. Для Internet Explorer 10+ DNS-предзагрузка и предзагрузка вообще -
понятия эквивалентные, в результате чего в обеих случаях выполняется
DNS-предзагрузка (из книги [Ильи Григорика (Ilya Grigorik)][11] [«Оптимизация
веб-производительности»][12]).*


## Как проверить поддержку предзагрузки

На данный момент не существует способа проверить поддержку
предзагрузки или предварительной отрисовки, а открытие
инструментов разработчика в Chrome отменит
все запросы на предзагрузку или предварительную отрисовку. Единственный
способ убедиться, что запрошенные вами ресурсы загружаются — это
проверить кэш браузера. В Chrome перейдите на **`chrome://cache/`**, в Firefox -
на **`about:cache`** и поищите файлы, которые вы пробовали предзагрузить.
Chrome позволяет увидеть была ли страница предварительно отрисована,
это можно проверить по адресу **`chrome://net-internals/#prerender`**.

У Google есть перечень ситуаций, в которых предварительная отрисовка отменяется:

* Инициация загрузки страницы
* Присутствие на странице тегов `audio` или `video`
* XMLHTTPRequests-запросы `POST`, `PUT` и `DELETE`
* HTTP-аутентификация
* HTTPS-страницы
* Страницы, для которых срабатывает предупреждение о наличии вредоносного ПО
* Наличие попапов или новых окон
* При определении использования большого количества ресурсов
* Открытие панели инструментов разработчика

*Пока что я не смог подтвердить отменяется ли предзагрузка во всех этих случаях*


## Что нас ждёт в будущем

В редакторском черновике по подсказкам предзагрузки ресурсов от 10 июля 2014
года вы можете увидеть подробный анализ того, как работает каждый из этих
элементов, и что DNS-предзагрузка будет называться «предварительное
подключение» (*англ. — «preconnect»*), а также то, что готовится к введению
новый элемент `preload`, который будет указывать на ресурсы, которые нужно
запросить как можно раньше.

[https://igrigorik.github.io/resource-hints/][13]


## Анализируйте архитектуру страниц, изучайте действия пользователей

Проведите анализ архитектуры вашего сайта и выясните как можно максимально
эффективно использовать предзагрузку: какие ресурсы являются наиболее важными
для каждой страницы? Какие действия запускают загрузку дополнительного контента?
Какой контент может блокировать рендеринг страницы?

Изучите действия пользователей на вашем сайте: какие страницы наиболее посещаемы?
Какой критический путь ваши пользователи проходят до конверсии? С какой частотой
совершаются определённые действия?

Большую часть этой информации можно получить с помощью инструментов для анализа страниц и,
собственно, понаблюдав за тем, как реальные посетители пользуются вашим сайтом.

**Всегда пользуйтесь инструментами вроде [webpagetest][14] для измерения
скорости вашего сайта до и после внесения изменений.**

Почему бы не начать предзагрузку следующей страницы после входа в веб-приложение?

Почему бы в процессе выполнения поиска не начать предзагрузку важных ресурсов для
страницы с результатами?

В процессе оформления заказа, пока пользователь заполняет форму,
почему бы не начать загрузку ресурсов для следующей страницы?

**Всегда сравнивайте сколько времени экономится для пользователя и какова при
этом дополнительная нагрузка на соединение с сетью**

> С большой силой приходит большая ответственность

[1]: http://www.stevesouders.com/blog/2013/11/07/prebrowsing/
[2]: https://plus.google.com/+WilliamChanPanda/posts/hsfVHq6wKxG
[3]: http://httparchive.org/interesting.php#caching
[4]: https://www.youtube.com/watch?feature=player_embedded&v=_Jn93FDx9oI
[5]: https://medium.com/
[6]: http://www.smashingmagazine.com/
[7]: http://www.net-a-porter.com/
[8]: https://www.airbnb.com/
[9]: http://csswizardry.com/
[10]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Link_prefetching_FAQ
[11]: https://www.igvita.com/
[12]: http://www.amazon.co.uk/High-Performance-Browser-Networking-performance/dp/1449344763
[13]: https://igrigorik.github.io/resource-hints/
[14]: http://www.webpagetest.org/

[Поддержка браузерами]: img/Fp1Q5A-ru.png
[rel-subresource-deprecation]: https://developers.google.com/web/updates/2016/03/link-rel-preload#goodbye-ltlink-relsubresource
