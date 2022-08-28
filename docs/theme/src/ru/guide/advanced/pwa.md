---
title: PWA
icon: setting
category:
  - Продвинутые
tag:
  - Продвинутые
  - PWA
---

Тема обеспечивает прогрессивную поддержку веб-приложений [^pwa-intro] через встроенный [`vuepress-plugin-pwa2`][pwa2], и по умолчанию он отключен.

[^pwa-intro]: **Введение в PWA**

    PWA, полное название Progressive Web app. Стандарт PWA установлен W3C.

    Это позволяет сайтам устанавливать сайт как приложение на поддерживаемой платформе через браузер, поддерживающий эту функцию.

::: info

`vuepress-theme-hope` передает `plugins.pwa` в параметрах темы в качестве параметров плагина для `vuepress-plugin-pwa2`.

Если вы используете этот плагин, мы рекомендуем вам установить `shouldPrefetch: false` в файле конфигурации VuePress.

:::

<!-- more -->

## Прямое включение <Badge text="Не рекомендуется" type="warning" />

Вы можете установить для `plugins.pwa` значение `true` в параметрах темы, чтобы тема автоматически генерировала необходимую конфигурацию и быстро включала плагины. Однако мы рекомендуем вам вручную установить некоторые параметры, следуя приведенным ниже инструкциям.

## Введение

Service Worker [^service-worker] (сокращенно SW) в основном используется для кэширования и проксирования контента сайта.

[^service-worker]: **Введение в Service Worker**

    1. Service Worker получит и кэширует все зарегистрированные в нем файлы в процессе регистрации.

    1. После завершения регистрации Service Worker активируется и начинает проксировать и контролировать все ваши запросы.

    1. Всякий раз, когда вы хотите инициировать запрос доступа через браузер, Service Worker проверит, существует ли он в своем собственном списке кеша, если он существует, он напрямую вернет кешированный результат, в противном случае он вызовет свой собственный метод fetch для получения Это. Вы можете использовать настраиваемый метод выборки, чтобы полностью контролировать результат запроса ресурсов на веб-странице, например предоставлять резервную веб-страницу в автономном режиме.

    1. Каждый раз, когда пользователь повторно открывает сайт, Service Worker будет запрашивать ссылку при его регистрации. Если будет обнаружена новая версия Service Worker, она обновится и начнет кэшировать список ресурсов, зарегистрированных в новом Service Worker. После того, как обновление содержимого будет успешно получено, Service Worker инициирует событие update. Пользователь может быть уведомлен через это событие, например, в правом нижнем углу будет отображаться всплывающее окно, сообщающее пользователю о наличии нового контента и позволяющее пользователю инициировать обновление.

Этот плагин автоматически зарегистрирует Service Worker через `workbox-build`. Чтобы лучше контролировать то, что Service Worker может предварительно кэшировать, подключаемый модуль предоставляет следующие конфигурации.

::: tip

Если вы опытный пользователь, вы также можете установить `plugins.pwa.generateSwConfig` в параметрах темы напрямую, чтобы передать параметры в `workbox-build`.

:::

## Управление кешем

В зависимости от требования installable [^installable], плагин предоставляет соответствующие параметры для управления кешем.

[^installable]: **Устанавливаемый**

    Чтобы сайт мог быть зарегистрирован как PWA, сайт должен сам успешно зарегистрировать действительный сервис-воркер и в то же время добавить действительный файл манифеста и объявить его.

    У каждой платформы или браузера есть требования к размеру кэша Service Worker. Когда размер файла кеша Service Worker слишком велик, сайт будет помечен как не подлежащий установке. Для Safari порог составляет 50 МБ, некоторые браузеры будут устанавливать меньше или больше значений (30 МБ, 70 МБ, 80 МБ), а Chrome отметит порог в 100 МБ.

    Файл манифеста должен содержать как минимум `name` (или `short_name`) `icons` `start_url`

    ::: note

    Начиная с Chrome 93, Service Worker должен содержать эффективные события выборки для управления автономными запросами.

    Однако в настоящее время плагин по умолчанию не содержит соответствующей логики обработки, поэтому на устройствах Android с Chrome 93 или более поздней версии сайт не будет отображать запрос на установку.

    :::

### Кэш по умолчанию

По умолчанию плагин предварительно кэширует все файлы `js` `css` и `svg`. Кешируются только домашняя страница и 404 `html`.

В то же время плагин будет кэшировать файлы шрифтов: `**/*.{woff,woff2,eot,ttf,otf}`.

### Кэш изображений

Вы можете кэшировать изображения сайта, установив для параметра `plugins.pwa.cachePic` значение `true`.

Если ваш сайт небольшой, а изображения в основном представляют собой критические описания и вы хотите, чтобы они отображались в автономном режиме, установите для этого параметра значение `true`.

::: info Распознавание изображений

Мы распознаем изображения по расширению файла. Любые файлы, оканчивающиеся на `.png`, `.jpg`, `.jpeg`, `.gif`, `.bmp`, `.webp` будут рассматриваться как изображения.

:::

### HTML-кэш

Если у вас есть небольшие сайты и вы хотите сделать документ полностью доступным в автономном режиме, вы можете установить для `plugins.pwa.cacheHTML` значение `true`, чтобы кэшировать все HTML-файлы.

::: tip Почему по умолчанию кешируются только домашняя страница и страница 404?

Хотя VuePress генерирует HTML-файлы через SSR[^ssr] для всех страниц, эти файлы в основном используются для SEO[^seo] и позволяют напрямую настраивать серверную часть без SPA[^spa]. Перейдите по любой ссылке.

[^ssr]: **SSR**: **S**erver **S**ide **R**endering,
[^seo]: **SEO**: **S**earch **E**ngine **O**ptimization.
[^spa]: **SPA**: **S**ingle **P**age **A**pplication, большинство из них имеют только домашнюю страницу и используют режим истории для обработки маршрутизации вместо фактического перехода между страницы.

VuePress — это, по сути, SPA. Это означает, что вам нужно только кэшировать домашнюю страницу и войти с домашней страницы, чтобы получить доступ ко всем страницам в обычном режиме. Следовательно, отсутствие кэширования других HTML по умолчанию может эффективно уменьшить размер кэша (на 40% меньше по размеру) и ускорить скорость обновления ПО.

Но в этом есть и недостаток. Если пользователь входит на сайт непосредственно с не домашней страницы, файл HTML для первой страницы все равно необходимо загрузить из Интернета. Кроме того, в автономной среде пользователи могут заходить только через домашнюю страницу, а затем самостоятельно переходить на соответствующую страницу. Если они напрямую обращаются к ссылке, появится недоступная подсказка.

:::

### Контроль размера

Чтобы предотвратить попадание больших файлов в список предварительного кэширования, все файлы размером более 2 МБ или изображения размером более 1 МБ будут удалены.

Вы можете настроить максимальный размер файла кэша (единица измерения: КБ) с помощью параметра `plugins.pwa.maxSize` или изменить предельный размер изображения (единица измерения: КБ) с помощью `plugins.pwa.maxPicSize`.

## Управление обновлениями

Мы предоставляем опцию `plugins.pwa.update` , чтобы контролировать, как пользователи получают обновления.

Значение по умолчанию для параметра `update` равно `"available"`, что означает, что при наличии нового контента новый SW будет автоматически установлено в фоновом режиме, а всплывающее окно предложит пользователю, что новый контент доступен. готово после завершения установки ПО. Пользователи могут выбрать, следует ли немедленно обновить страницу для просмотра нового контента.

При поведении по умолчанию пользователи по-прежнему будут читать старый контент до того, как ПО будет готово, и им не будет предложено. Если ваш проект все еще находится на стадии разработки, и вы хотите предупредить пользователя о том, что он может читать устаревший контент, вы можете установить для этого параметра значение `"hint"`. Это позволяет пользователям получать уведомления о публикации нового контента в течение нескольких секунд после посещения документов. Но негативным эффектом этого является то, что если пользователь решит выполнить обновление до того, как новый SW будет готово, ему потребуется получить все ресурсы страницы из Интернета до того, как новый SW установит и будет управлять страницей.

Если ваши документы стабильны или вы ведете блог и не слишком заботитесь о том, чтобы пользователи сразу же получали последнюю версию, вы можете установить для этого параметра значение `"disabled"`, что означает, что новый SW будет установлено полностью автоматически. в фоновом режиме и начать ждать, когда все страницы, контролируемые старой версией ПО, будут закрыты, новый SW начнет брать на себя управление и предоставлять пользователям новый контент при следующем посещении. Этот параметр может предотвратить отвлечение пользователей всплывающим окном в правом нижнем углу во время посещения.

Чтобы ускорить доступ пользователей в условиях слабой сети или ее отсутствия через ПО, а также чтобы пользователи всегда получали доступ к новому контенту, вы можете установить для этой опции значение `"force"`. Действие этой опции заключается в отмене регистрации старого ПО при обнаружении новый SW и обновлении страницы, чтобы убедиться, что пользователь просматривает новейший контент. Но мы настоятельно рекомендуем не использовать эту опцию без необходимости, так как после выпуска новый SW все пользователи столкнутся с неожиданным внезапным обновлением в течение нескольких секунд после входа на сайт, и им придется получить доступ к документу через Интернет и установить все последнее SW.

### Всплывающее окно с запросом на обновление

При обнаружении нового контента (обнаружено новый SW), в правом нижнем углу появится всплывающее окно с запросом на обновление, которое позволит пользователю обновить и применить.

::: tip пользовательское всплывающее окно

Если вас не устраивает всплывающее окно по умолчанию, вы можете написать свой собственный компонент. Вам необходимо зарегистрировать свой собственный всплывающий компонент глобально и передать имя компонента в параметр `plugins.pwa.hintComponent`.

:::

### Обновить готовое всплывающее окно

Когда новый контент будет готов (новый SW успешно установлен и начал ждать), в правом нижнем углу появится всплывающее окно готовности обновления, которое позволит пользователю обновить и применить.

::: tip пользовательское всплывающее окно

Если вас не устраивает всплывающее окно по умолчанию, вы можете написать свой собственный компонент. Вам необходимо зарегистрировать свой всплывающий компонент глобально и передать имя компонента в параметр `plugins.pwa.updateComponent`.

:::

## Генерация манифеста

Чтобы обеспечить возможность установки PWA, сайт должен сгенерировать файл манифеста и объявить действительный адрес файла манифеста [^manifest] через `<link>`.

[^manifest]: **Файл манифеста**

    Файл манифеста использует формат JSON и отвечает за объявление различной информации о PWA, такой как имя, описание, иконка и действия быстрого доступа.

    Чтобы ваш сайт был зарегистрирован как PWA, вам необходимо соответствовать основным спецификациям манифеста, чтобы браузер рассматривал сайт как устанавливаемое PWA и разрешал пользователям устанавливать его.

    ::: info

    Стандарты и спецификации манифеста смотрите в [Манифесте W3C](https://w3c.github.io/manifest/)

    :::

Плагин автоматически сгенерирует для вас файл манифеста `manifest.webmanifest` в выходном каталоге, а также добавит оператор адреса манифеста в каждый HTML-код `<head>`.

Если у вас уже есть файл `manifest.webmanifest` или `manifest.json` в `.vuepress/public`, плагин прочитает и объединит его с окончательным манифестом.

### Автоматическая генерация

Плагин будет использовать информацию из API-интерфейса плагина VuePress и максимально задавать запасной вариант для полей в манифесте. Таким образом, вам не нужно устанавливать большинство полей манифеста.

Если следующие поля не установлены, они попытаются вернуться к следующим предустановленным значениям по порядку.

| Опции                       | Значение по умолчанию                                                                                   |
| --------------------------- | ------------------------------------------------------------------------------------------------------- |
| name                        | `siteConfig.title` \|\| `siteConfig.locales['/'].title` \|\| `"Site"`                                   |
| short_name                  | `siteConfig.title` \|\| `siteConfig.locales['/'].title` \|\| `"Site"`                                   |
| description                 | `siteConfig.description` \|\| `siteConfig.locales['/'].description` \|\| `"A site built with vuepress"` |
| lang                        | `siteConfig.locales['/'].lang` \|\| `"en-US"`                                                           |
| start_url                   | `siteConfig.base`                                                                                       |
| scope                       | `siteConfig.base`                                                                                       |
| display                     | `"standalone"`                                                                                          |
| theme_color                 | `"#46bd87"`                                                                                             |
| background_color            | `"#ffffff"`                                                                                             |
| orientation                 | `"portrait-primary"`                                                                                    |
| prefer_related_applications | `false`                                                                                                 |

Для полных элементов конфигурации см. [Файл определения типа манифеста](https://github.com/vuepress-theme-hope/vuepress-theme-hope/blob/main/packages/pwa2/src/shared/manifest.ts).

### Ручная настройка

Вы можете вручную указать содержимое манифеста с помощью `plugins.pwa.manifest` в параметрах темы.

::: tip Приоритет

Опция `plugins.pwa.manifest` в теме имеет наивысший приоритет, за ней следуют файлы манифеста, которые могут находиться в папке `public`.

:::

**Вы должны, по крайней мере, установить действительный значок с помощью `manifest.icons` в `plugins.pwa` или других параметров, связанных со значком, в плагине PWA.**

::: warning

Спецификация возможности установки [^installable] требует, чтобы в манифесте был объявлен хотя бы одна действительная иконка.

Таким образом, если вы не настроите `manifest.icons` в `plugins.pwa`, посетители смогут пользоваться только автономным доступом, обеспечиваемым кешем Service Worker, но не смогут установить ваш сайт как PWA.

Кроме того, плагин по умолчанию ничего не обрабатывает в манифесте, а выводит как есть. Это означает, что если вы планируете развертывание в подкаталоге, вы должны добавить префикс URL-адреса к URL-адресам манифеста самостоятельно.

Но если все, что вам нужно, находится в базовой папке, вы можете установить `appendBase: true` в `plugins.pwa`, чтобы плагин мог добавлять `base` к любым ссылкам в нем.

:::

## Другие опции

Плагин также предоставляет другие параметры, связанные с PWA, такие как значок плитки Microsoft и настройки цвета, значок Apple и так далее.

Вы можете установить их по мере необходимости. Подробные параметры смотрите в [Конфиг PWA](../../config//plugins/pwa.md).

## Дальнейшее чтение

Для получения более подробной информации смотрите:

- [Документация по плагину PWA][pwa2]
- [Google PWA](https://web.dev/progressive-web-apps/)
- [MDN PWA](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)
- [Спецификация манифеста W3C](https://w3c.github.io/manifest/)

[pwa2]: https://vuepress-theme-hope.github.io/v2/pwa/