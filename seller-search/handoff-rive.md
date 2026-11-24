# Анимация поиска продавца · спецификация Rive-файла

**Версия спеки 1.1** · 24 ноября 2026 · вариант F (утверждён)

Как подключить файл `seller-search.riv`: что в нём лежит, как вписать его в экран и как проверить результат. Анимация полностью собрана в файле, приложению ничего настраивать не нужно: только показать и запустить.

### 0. Что реализовать

Показать `seller-search.riv` на экране ожидания «Запрос автозапчастей» над текстом «Ищем продавцов…» через официальный Rive runtime. Артборд `SellerSearch`, стейт-машина `State Machine 1`, оба по умолчанию. Готово, когда анимация крутится без остановки всё время, пока нет откликов. Когда приходит первый отклик, экран целиком меняется на список откликов: доигрывать анимацию до конца не нужно.

Сервис открывается во вебвью, поэтому основной способ подключения — web runtime (раздел 6).

**Если спеку читает ИИ-ассистент.** Правила, которые чаще всего нарушают при встраивании:

- Всё рисует скрипт Rive (Luau). Нужен runtime с поддержкой Rive Scripting. На старом runtime файл загрузится, но артборд останется пустым.
- Запускать стейт-машину `State Machine 1`, а не анимацию `Idle`. `Idle` — пустой таймлайн, он только держит машину активной.
- Вью-модели, inputs стейт-машины и events в файле нет. Привязывать и задавать ничего не нужно, auto-bind не нужен.
- Настройки анимации (скорость, число продавцов, цвета) зашиты во входы скрипта внутри файла. Из приложения их не поменять. Если это понадобится, файл нужно пересобрать (раздел 3).
- Fit — Contain, квадратная область. Сцена сама вписывается в любую область, но рассчитана на квадрат.
- Не обрезать область рисования и не масштабировать canvas стилями без `resizeDrawingSurfaceToCanvas()`: точки 1,4 px станут мыльными.

### 1. Файл

| Поле | Значение |
|---|---|
| Файл | `seller-search.riv` |
| Размер | 75 365 байт |
| SHA-256 | `ce38027ca75f19533793dc2d7b2932ff8fc79d4d3904e7999e9b0a44b2954f6b` |
| Исходник в Rive | «Анимация поиска продавца», file id 2747104, экспорт из редактора 24.11.2026 |
| Ресурсы внутри | скрипт `SellerSearch` (Luau), шрифт `GolosText` (Golos Text Regular, только буквы 14 имён), 14 фото продавцов 192 × 192, WebP 75 % |

**Где взять файл.** GitHub, прямые ссылки без входа: [seller-search.riv](https://github.com/kolesamotion/motion-library/raw/main/seller-search/seller-search.riv) · [seller-search.zip](https://github.com/kolesamotion/motion-library/raw/main/seller-search/seller-search.zip) (внутри `seller-search.riv`, эта спека `handoff-rive.md`, исходник `scene.rml` и `rive.yaml`) · папка [seller-search](https://github.com/kolesamotion/motion-library/tree/main/seller-search). Те же кнопки есть на странице «Анимация поиска продавца», раздел «Разработка». После скачивания сверьте SHA-256 с таблицей.

Файл в Rive руками не правят без нового экспорта и обновления этой спеки: после правки SHA-256 меняется.

### 2. Состав файла

| Объект | Имя | Что делает |
|---|---|---|
| Артборд | `SellerSearch` | 360 × 360, фон прозрачный, по умолчанию. Его и подключать |
| Лейаут | `Root` | растянут на весь артборд |
| Скрипт-лейаут | `Search` | скрипт `SellerSearch` рисует всё: сетку точек, линзу, волны, продавцов и имена |
| Стейт-машина | `State Machine 1` | по умолчанию; одно состояние `Idle` по кругу |
| Анимация | `Idle` | пустой таймлайн 1 с по кругу, чтобы стейт-машина не останавливалась |
| Вью-модель, inputs, events | нет | приложению нечего задавать и нечего слушать |

### 3. Настройки внутри файла

Входы скрипта на узле `Search`. Они уже выставлены под вариант F и из приложения не меняются.

| Вход | Значение | Что значит |
|---|---|---|
| `speed` | 0.8 | темп всей анимации |
| `count` | 1 | сколько продавцов видно одновременно (в демо можно посмотреть 2 и 3) |
| `showNames` | true | имя под аватаркой |
| `accent` | `#0077FF` | синева линзы, волны и бутона |
| `dotColor` | `#B2B8C6` | цвет точек сетки |
| `seed` | 0 | 0 — новый случайный расклад при каждом запуске; число больше 0 закрепляет расклад и порядок |

Если продукту понадобится менять что-то из приложения (например, `count`), моушн-дизайнер вынесет это во вью-модель и выпустит новую версию файла и спеки.

### 4. Что происходит в анимации

- **Интро.** Сетка точек собирается за 1,5 с. Первый продавец появляется примерно через 1,5 с после старта.
- **Продавцы.** Одновременно виден один. Новый вырастает из точки сразу до полного размера за 0,8 с, пока предыдущий уменьшается и исчезает. Новый продавец появляется каждые 2,8 с.
- **Круг.** 14 продавцов, каждый появляется один раз за круг, круг длится 39,4 с. Порядок тасуется заново каждый круг, места появления подбираются заново при каждом запуске, на стыке кругов никто не выходит дважды подряд.
- **Точки.** Сетка 11 px, точки 1,4 px `#B2B8C6` в круге радиусом 176 px с мягким краем. Сетка медленно «дышит», по ней ходит синяя линза 44 px, от продавца расходится волна. Точки у аватарки не огибают её, а гаснут за 5 px до края.
- **Аватарка.** Круг 54 px, имя Golos Text 14 px под ней.
- **Шов.** Кадр зависит только от времени, швов и рывков в петле нет. Анимация не кончается и не подаёт сигналов о завершении.

Все времена даны при `speed` 0.8, то есть так, как анимация идёт в файле.

### 5. Размер и размещение

Сцена нарисована в квадрате 360 × 360 dp. В макете (экран «Запрос автозапчастей», Figma `oRHggPsVAoQpTH36SL9oq9`, узел 356:89473) зона анимации тоже 360 × 360 во всю ширину экрана, сразу под ней текст «Ищем продавцов…» и «Сообщим, когда появятся первые отклики».

- Область Rive = зона анимации из макета, 360 × 360 на экране шириной 360. На более широких экранах держите квадрат по ширине экрана.
- Fit — Contain, выравнивание по центру. Неквадратная область тоже работает: сцена впишется по меньшей стороне и встанет по центру, по краям будет виден фон экрана.
- `Fit.Layout` тоже поддерживается: сцена вписывается внутри артборда любого размера.
- Фон артборда прозрачный: под анимацией виден фон экрана, отдельно задавать его не нужно.
- Анимация рассчитана на светлый фон: имена под аватарками тёмные (`#242933`), точки сетки `#B2B8C6`. На тёмном экране имена читаться не будут; для тёмной темы нужна отдельная версия файла.

### 6. Подключение

Общие шаги для всех платформ:

1. Скачать `seller-search.riv` (раздел 1), положить в ресурсы приложения и загружать оттуда.
2. Выбрать артборд `SellerSearch` и стейт-машину `State Machine 1` (оба по умолчанию).
3. Fit Contain, автозапуск.

Нужен runtime с поддержкой Rive Scripting. Проверено на web `@rive-app/webgl2` 2.50.0 и 2.55.0 и `@rive-app/canvas` 2.50.0. На iOS (`RiveRuntime`) и Android (`rive-android`) возьмите версию с поддержкой скриптов и сверьте с демо на странице.

Web (вебвью), проверенный пример:

```js
const r = new rive.Rive({
  src: "seller-search.riv",
  canvas: document.getElementById("seller-search"), // квадрат 360 × 360 CSS px
  artboard: "SellerSearch",
  stateMachines: "State Machine 1",
  autoplay: true,
  layout: new rive.Layout({ fit: rive.Fit.Contain, alignment: rive.Alignment.Center }),
  onLoad: () => r.resizeDrawingSurfaceToCanvas()
});
window.addEventListener("resize", () => r.resizeDrawingSurfaceToCanvas());
// экран сменился на список откликов:
// r.cleanup();
```

iOS (SwiftUI, RiveRuntime со скриптами), набросок, сверить с версией SDK:

```swift
let vm = RiveViewModel(fileName: "seller-search", stateMachineName: "State Machine 1", fit: .contain, artboardName: "SellerSearch")
vm.view().frame(width: 360, height: 360)
```

Android (rive-android со скриптами), набросок, сверить с версией SDK. В `res/raw` дефис в имени недопустим, поэтому файл кладётся как `seller_search.riv`:

```kotlin
riveView.setRiveResource(R.raw.seller_search, artboardName = "SellerSearch", stateMachineName = "State Machine 1", fit = Fit.CONTAIN)
```

### 7. Поведение

- Анимация запускается при показе экрана и крутится, пока нет откликов.
- Пришёл первый отклик: экран меняется целиком, Rive-вью выгружается. Выходной анимации нет и не нужно.
- Ушли с экрана и вернулись: начните заново. Анимация стартует с интро и новым раскладом, это ожидаемо.
- Свёрнутое приложение или скрытый вебвью: Rive-вью можно поставить на паузу, при возвращении продолжить.
- Reduced motion (iOS Reduce Motion, Android «Удалить анимации», web `prefers-reduced-motion`): в файле нет готового статичного кадра, при t = 0 сетка ещё не собрана. Что показывать в этом режиме, решает продуктовый дизайнер; по умолчанию можно оставить анимацию, она спокойная и без резких движений.
- Событий для аналитики, звука и хаптики нет.
- Нагрузка: от 20 до 47 отрисовок на кадр, расчёт кадра около 0,6 мс на MacBook Air и около 2 мс на кадр целиком в web runtime без GPU.

### 8. Проверка

- Через 1,5 с после старта появляется первый продавец, сетка к этому моменту собрана.
- На экране одновременно один продавец полного размера; новый появляется примерно каждые 2,8 с, пока предыдущий уходит.
- За 40 с показываются все 14 имён: Серик, Андрей, Айман, Алексей, Данияр, Маржан, Сергей, Болат, Максим, Ольга, Аружан, Виктор, Арман, Айдос.
- Точки чёткие, не размытые; аватарка круглая, без квадратных углов.
- Если область анимации пустая дольше 2 с, runtime не поддерживает скрипты: обновите его.
- Под анимацией виден фон экрана, серого квадрата нет.
- SHA-256 файла совпадает с таблицей в разделе 1.

### 9. Исходник

Файл собран как проект Rive CLI: `rive.yaml` (связь с файлом в Rive), `scene.rml` (сцена, ниже), скрипт `SellerSearch.luau`, фото и шрифт. `rive.yaml` и `scene.rml` лежат в архиве. Скрипт и ассеты хранятся в самом файле Rive и в папке проекта у моушн-дизайнера. Итоговый `.riv` берётся только из экспорта редактора (Export, For runtime): сборка CLI вкладывает фото без сжатия и не годится для выдачи.

<details class="md-fold"><summary>Показать scene.rml · 47 строк</summary>

```xml
<Rive version="1" kind="fragment">
    <Artboard defaultStateMachineId="0:7" styleId="0:5" width="360" height="360" name="SellerSearch" id="0:2">
        <LayoutComponentStyle name="Artboard Style" id="0:5"/>

        <LayoutComponent width="360" height="360" styleId="0:31" name="Root" id="0:30">
            <LayoutComponentStyle layoutWidthScaleType="fill" layoutHeightScaleType="fill" widthUnitsValue="auto" heightUnitsValue="auto" name="Root Style" id="0:31"/>
            <ScriptedLayout scriptAssetId="0:80" name="Search" id="0:32">
                <ScriptInputNumber propertyValue="0.8" name="speed" id="0:21"/>
                <ScriptInputBoolean propertyValue="true" name="showNames" id="0:22"/>
                <ScriptInputColor propertyValue="FF0077FF" name="accent" id="0:23"/>
                <ScriptInputColor propertyValue="FFB2B8C6" name="dotColor" id="0:24"/>
                <ScriptInputNumber propertyValue="0" name="seed" id="0:25"/>
                <ScriptInputNumber propertyValue="1" name="count" id="0:26"/>
            </ScriptedLayout>
        </LayoutComponent>

        <StateMachine name="State Machine 1" id="0:7">
            <StateMachineLayer name="Layer 1" id="0:8">
                <AnyState x="200" y="-120" id="0:116"/>
                <ExitState x="400" y="-120" id="0:117"/>
                <EntryState id="0:118">
                    <StateTransition stateToId="0:12" id="0:119"/>
                </EntryState>
                <AnimationState x="200" animationId="0:6" id="0:12"/>
            </StateMachineLayer>
        </StateMachine>

        <LinearAnimation loopValue="loop" duration="60" name="Idle" id="0:6"/>
    </Artboard>

    <ScriptAsset file="SellerSearch.luau" name="SellerSearch" id="0:80" codeFileId="0:121"/>
    <FontAsset includeInExport="true" file="fonts/GolosText-Regular.ttf" name="GolosText" id="0:90"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/01_serik.png" name="01_serik" id="0:100"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/02_andrey.png" name="02_andrey" id="0:101"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/03_aiman.png" name="03_aiman" id="0:102"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/04_aleksey.png" name="04_aleksey" id="0:103"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/05_daniyar.png" name="05_daniyar" id="0:104"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/06_marzhan.png" name="06_marzhan" id="0:105"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/07_sergey.png" name="07_sergey" id="0:106"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/08_bolat.png" name="08_bolat" id="0:107"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/09_maksim.png" name="09_maksim" id="0:108"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/10_olga.png" name="10_olga" id="0:109"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/11_aruzhan.png" name="11_aruzhan" id="0:110"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/12_viktor.png" name="12_viktor" id="0:111"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/13_arman.png" name="13_arman" id="0:112"/>
    <ImageAsset format="webp" quality="0.75" includeInExport="true" file="avatars/14_aidos.png" name="14_aidos" id="0:113"/>
</Rive>
```

</details>
