# Измерение производительности сайта Ригла

## Network

Профиль загрузки ресурсов [HAR](./sources/www.rigla.ru.har)

### Неоптимальные места:
#### 1. Дублирование ресурсов:

1.1 Повторные запросы к API: `https://www.rigla.ru/graphql`   
Вероятно, стоит их группировать в один запрос (по возможности)
![graphql](./sources/graphql.png)

1.2 Несколькими запросами получаются подтипы одного шрифта:
    `https://www.rigla.ru/fonts/MyriadPro`
    ![MyriadPro](./sources/MyriadPro.png)

1.3 Перезапрос шрифта Merriweather weight 400 и 700:
    ![Merriweather](./sources/Merriweather.png)

   Запросы к шрифтам могут быть оптимизированы путем объединения запросов. Или вообще подумать над тем, куда нам столько шрифтов, нужны ли они нам все?
   ![Why why so much fonts](./sources/much-fonts.png)

1.4 Отправляется два запроса, отличающиеся лишь одним полем.    
Вероятно нам вообще не нужно делать тот запрос, что не содержит в себе поля asyncMetrics (второй):
    ![privacy-cs.mail.ru-1](./sources/mail.ru-1.png)
    ![privacy-cs.mail.ru-2](./sources/mail.ru-2.png)

#### 2. Лишний размер ресурса:

2.1 Размер изображений в каруселях слишком велик, картинки не оптимизированы. 
В верстке они используются в размере 150х139px, а грузятся в 810х750px 
![Too large images](./sources/large-images.png)
Элементарно в карусели картинки весят по 100-300kb, а их можно оптимизировать до 30-60kb, даже не меняя формат на более современный (webp).
![Optimization images](./sources/optimization-images.png)

2.2 Грузится неиспользуемый js код   
    Получается, что на главной странице у нас грузится скрипт карты, который используется на других страницах (не на главной). Это замедляет загрузку главной страницы. Стоит отдавать эти скрипты после загрузки контента страницы или вообще грузить отдельно.
    Помимо всего прочего он еще и большого размера.
    ![Maps script](./sources/maps.png)

#### 3. Медленно загружающиеся ресурсы:   
3.1 Фрагменты js кода слишком большие. Из-за чего загружаются очень долго прежде чем мы отобразим контент для пользователя - 1,9 сек.   
Некоторые скрипты разумно разделены на чанки, но присутствуют слишком большие фрагменты, что замедляет загрузку.
![Too long js](./sources/long-js.png)

3.2 Большие не оптимизированные картинки

#### 4. Ресурсы, блокирующие загрузку:
4.1 Js код, блокирующий отображение контента (скрипты карт и прочие скрипты, не относящиеся к главной странице грузятся по первому)
![Resources block loading](./sources/block-loading.png)

4.2 Загружаются изображения, которые не видны при первом открытии страницы (в каруселях и ниже на странице). Можно использовать lazy load на скролл, чтобы ускорить отображение контента для пользователя
![Hidden images](./sources/hidden-images.png)

#### 5. Что-то ещё:
5.1 не работает Back/Forward cache: на странице используется WebBluetooth API - его можно отключить
![Turned off BFC](./sources/bfc.png)

5.2 Использование устаревших форматов изображений современные форматы изображений - то есть форматы WebP и AVIF (более эффективное сжатие по сравнению с PNG/JPEG)

5.3 Большое смещение макета при загрузке контента - Можно пофиксить задав изображениям явным образом атрибуты width и height.

## Performance

Профиль загрузки [JSON](./sources/performance-trace-20240619T173131.json)

### Время в миллисекундах от начала навигации до событий

- First Paint (FP) - 383.9 ms

    ![First Paint](./sources/FP.png)
- First Contentful Paint (FCP) - 383.9 ms   

    ![FCP](./sources/FCP.png)
- DOM Content Loaded (DCL) - 638.7 ms

    ![DCL](./sources/DCL.png)
- Load - 737.1 ms

    ![Load](./sources/Load.png)
- Largest Contentful Paint (LCP) - 3 368.8 ms

    ![LCP](./sources/LCP.png)

<p align="center">

| Метрика                  | Время, мс |
|--------------------------|-----------|
| First Paint              | 383.9     |
| First Contentful Paint   | 383.9     |
| DCL                      | 638.7    |
| Load Event               | 737.1    |
| Largest Contentful Paint | 3368.8    |

</p>

### DOM-элемент на котором происходит LCP

```
<img src="https://cdn.rigla.ru/media/tagesjump_mvppopup_popup/image/r/i/rigla_638x900_96dpi.jpg" 
draggable="false" alt="картинка текущего банера" 
class="popup-metadata-type-slider__img" 
data-gtm-vis-recent-on-screen30172849_1043="4197" data-gtm-vis-first-on-screen30172849_1043="4197" data-gtm-vis-total-visible-time30172849_1043="100" data-gtm-vis-has-fired30172849_1043="1">
```

![DOM-element LCP](./sources/LCP-element.png)

### Сколько времени в миллисекундах тратится на разные этапы обработки документа

Этапы обработки документа занимают:

| Метрика    | Время, мс |
|------------|-----------|
| Loading    | 20        |
| Scripting  | 2550      |
| Rendering  | 287       |
| Painting   | 204       |

![Timings summary](./sources/timings.png)

## Coverage

Профиль загрузки страницы [JSON](./sources/Coverage.json)

Скриншот вкладки после загрузки страницы:
![Coverage tab](./sources/coverage-tab.png)

Неиспользуемый CSS: 45.2 kB
![Unused CSS](./sources/unused-css.png)

Неиспользуемый JS: 3 686 kB
![Unused JS](./sources/unused-js.png)

# Измерение производительности сайта с замедлением CPU 4x slowdown и эмуляцией сети Slow 3G

## Network

Профиль загрузки ресурсов [HAR](./sources/www.rigla.ru-slow.har)

### Неоптимальные места:

Неоптимальные места остались те же, только ситуация еще ухудшилась.

1. Запрос аналитики отправляется еще тогда, когда страница не загружена и блокирует загрузку необходимых ресурсов. По итогу мы ждали почти 40 секунд, но так и не получили результат. Вероятно стоит откладывать аналитические запросы на после загрузки основного контента страницы. 
![Analytics request](./sources/analytics.png)

2. Гигантские по размеру картинки, которые мы ждали по 10-50 секунд (!).
![Too long wait images](./sources/wait-images.png)

3. В целом очень большое количество запросов, сложное DOM-дерево, не оптимизированный контент не позволяют нам использовать сайт в условиях медленного соединения и слабого CPU.


## Performance

Профиль загрузки [JSON](./sources/performance-slow-trace-20240619T181729.json)

### Время в миллисекундах от начала навигации до событий

- First Paint (FP) - 23 671.0 ms

    ![First Paint](./sources/FP-slow.png)
- First Contentful Paint (FCP) - 23 671.0 ms

    ![FCP](./sources/FCP-slow.png)
- DOM Content Loaded (DCL) - 38 148.2 ms

    ![DCL](./sources/DCL-slow.png)
- Largest Contentful Paint (LCP) - 108 693.1 ms

    ![LCP](./sources/LCP-slow.png)
- Load - 117 525.1 ms

    ![Load](./sources/load-slow.png)

<p align="center">

| Метрика                  | Время, мс |
|--------------------------|-----------|
| First Paint              | 23 671.0  |
| First Contentful Paint   | 23 671.0  |
| DCL                      | 38 148.2  |
| Largest Contentful Paint | 108 693.1 |
| Load Event               | 117 525.1 |

</p>

### DOM-элемент на котором происходит LCP
`img.popup-metadata-type-slider__img`

![DOM-element LCP](./sources/LCP-element-slow.png)

### Сколько времени в миллисекундах тратится на разные этапы обработки документа

Этапы обработки документа занимают:

| Метрика    | Время, мс |
|------------|-----------|
| Loading    | 110       |
| Scripting  | 11 020    |
| Rendering  | 1 784     |
| Painting   | 2 172     |

![Timings summary](./sources/timings-slow.png)

## Coverage

Профиль загрузки страницы [JSON](./sources/Coverage-slow.json)

Скриншот вкладки после загрузки страницы
![Coverage tab](./sources/coverage-tab-slow.png)

Неиспользуемый CSS: 45.4 kB
![Unused CSS](./sources/unused-css-slow.png)

Неиспользуемый JS: 3 686 kB
![Unused JS](./sources/unused-js-slow.png)
