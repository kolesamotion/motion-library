# AI-иконка · спецификация Rive-файла

**Версия спеки 1.1** · 9 ноября 2026 · GD-11154 · только для новых экранов

Как подключить файл `ai_icon.riv`: что в нём лежит, какие свойства задавать и как вписать иконку в макет. Иконка анимируется сама, нужно только выбрать цвет и паузу между повторами.

### 0. Что реализовать

Показать на экране файл `ai_icon.riv` через официальный Rive runtime платформы. Запустить стейт-машину `icon_button_sm` и задать во вью-модели `AiIconButton` два свойства: `color` (цвет звёзд) и `pause` (пауза между повторами в мс). Готово, когда иконка стоит на месте статичной иконки из макета, совпадает с ней в покое и повторяет анимацию с нужной паузой.

Старые экраны остаются на прежнем файле. Этот файл нужен только для новых экранов, обратная совместимость с прежними артбордами `ai_icon_button` и `ai_icon_loader` не нужна: их в файле нет.

**Если спеку читает ИИ-ассистент.** Правила, которые чаще всего нарушают при встраивании:

- Запускать стейт-машину `icon_button_sm`, а не анимацию `icon` напрямую. Пауза и смена цвета живут в стейт-машине.
- Цвет и пауза задаются свойствами вью-модели (`color`, `pause`), а не inputs стейт-машины. Inputs в файле нет.
- `pause` задаётся в миллисекундах, а не в секундах: 2000 — это 2 с.
- Включить auto-bind (привязать экземпляр вью-модели по умолчанию). Без привязки иконка играет белой без паузы, но поменять ничего нельзя.
- Звёзды занимают не весь артборд, а около 66 % по центру. Чтобы совпасть с иконкой 24 × 24 из макета, область Rive делается 32 × 32 с тем же центром (раздел 4).
- Не обрезать область рисования по краю звёзд: в движении звёзды выходят за рамки покоя.

### 1. Файл

| Поле | Значение |
| --- | --- |
| Файл | `ai_icon.riv` |
| Размер | 3 606 байт |
| SHA-256 | `9479cf66441a735bead4f0be206435031eacb8396335b6927dc09cc5f9961afa` |
| Исходник в Rive | `ai_icon_prod`, file id 2270880, ревизия «v7 default pause 0» |
| Ресурсы | нет: без картинок, шрифтов, текста, звука и скриптов |

**Где взять файл.** На странице «AI-иконка», вкладка «Разработка», кнопка «Скачать .riv (zip)». В архиве `ai_icon.zip`: `ai_icon.riv`, эта спека `handoff-rive.md`, исходник `scene.rml` и `rive.yaml`. После распаковки сверьте SHA-256 с таблицей.

Файл в Rive руками не правят без новой ревизии и обновления этой спеки: после правки SHA-256 меняется.

### 2. Состав файла

| Объект | Имя | Что делает |
| --- | --- | --- |
| Артборд | `ai_icon` | 32 × 32, фон прозрачный, по умолчанию. Его и подключать |
| Вложенный артборд | `ai_icon_art` | сама иконка: три звезды. Вписан в `ai_icon` с fit contain, поэтому иконка масштабируется под любой размер |
| Стейт-машина | `icon_button_sm` | по умолчанию; движение → пауза → движение по кругу, отдельный слой отвечает за цвет |
| Анимация | `icon` | движение 2 с (120 кадров при 60 fps) |
| Анимация | `color_gradient` | вращение градиента внутри звёзд, 2 с, по кругу |
| Вью-модель | `AiIconButton`, экземпляр `Default` | два свойства, значения передаются во вложенный артборд автоматически |
| Свойство | `color` (enum `AiIconColor`) | `white` · `grey` · `gradient`, по умолчанию `white` |
| Свойство | `pause` (number, мс) | пауза между повторами, по умолчанию 0 (без остановки) |

Цикл: анимация `icon` проигрывает движение 2 с, затем стейт-машина переходит в покой на `pause` мс и запускает движение заново с первого кадра. Полный цикл = 2000 мс + `pause`. По умолчанию `pause` = 0: иконка двигается без остановки, цикл 2 с. Events нет, обратной связи из файла в приложение нет.

### 3. Цвета

| `color` | Звёзды | Где использовать |
| --- | --- | --- |
| `white` | `#FFFFFF` | на градиентной плашке Kolesa AI (`#4E33FE → #0077FF → #00D4FF`), на тёмном фоне |
| `grey` | `#A2AAB8` (Neutral 75) | на неактивной кнопке (фон `#F0F2F5`), например «Улучшаю описание…» |
| `gradient` | градиент `#00D4FF → #0077FF → #4E33FE`, вращается | на белом фоне, крупно: замена прежнего лоадера |

Прозрачность звёзд одинаковая во всех цветах: большая 100 %, верхняя маленькая 80 %, нижняя маленькая 40 %. Она уже заложена в файл, задавать её не нужно.

### 4. Размер и размещение

Иконка масштабируется пропорционально под любой квадратный размер. Fit — Contain (или Layout), выравнивание по центру. Неквадратная область тоже работает: иконка впишется по меньшей стороне и встанет по центру.

Звёзды в покое занимают 66 % артборда по центру (отступ 17 % с каждой стороны). В макетах иконка нарисована в рамке 24 × 24, где звёзды занимают около 21 × 21. Поэтому:

| Слот в макете | Область Rive | Сдвиг области от слота |
| --- | --- | --- |
| 24 × 24 (кнопка, плашка Kolesa AI) | 32 × 32 | −4, −4 (тот же центр) |
| 16 × 16 | 21.3 × 21.3 | −2.7, −2.7 |
| 80 × 80 (лоадер) | 106.7 × 106.7 | −13.3, −13.3 |

Общее правило: область Rive = слот × 4/3, центр совпадает с центром слота. Так иконка совпадает с макетами, где звёзды занимают около 88 % рамки. Прежний артборд `ai_icon_loader` рисовал звёзды мельче: чтобы повторить его один в один, область 80 × 80 = слоту.

Фон прозрачный. Область больше слота, поэтому соседний текст и края плашки не должны её обрезать: отключите обрезку у родителей (`clipsToBounds = false`, `clipChildren = false`, `overflow: visible`) или учитывайте поля при вёрстке.

### 5. Готовые настройки

| Где | `color` | `pause` | Размер слота |
| --- | --- | --- | --- |
| Плашка Kolesa AI в шторке «Проверяю фотографии» | `white` | 0 | 24 × 24 |
| Кнопка «Указать параметры авто» на экране «Оценка авто» | `white` | 2000 | 24 × 24 |
| Неактивная кнопка «Улучшаю описание…» | `grey` | 0 | 24 × 24 |
| Лоадер «Проверяем параметры авто» в «Оценке авто» | `gradient` | 2000 | 80 × 80 |

### 6. Подключение

Общие шаги для всех платформ:

1. Скачать `ai_icon.riv` (раздел 1), положить в ресурсы приложения и загружать оттуда.
2. Выбрать артборд `ai_icon` и стейт-машину `icon_button_sm` (оба по умолчанию).
3. Привязать экземпляр вью-модели по умолчанию (auto-bind).
4. Записать `color` (строка значения enum) и `pause` (число в мс). Свойства можно менять на лету, например переключить цвет при смене состояния кнопки.

Нужен runtime с поддержкой Data Binding (вью-модели). Проверено на web `@rive-app/canvas` 2.43.1. На iOS (`RiveRuntime`) и Android (`rive-android`) возьмите актуальную версию с Data Binding и сверьте с демо на этой странице.

Web, проверенный пример:

```js
const r = new rive.Rive({
  src: "ai_icon.riv",
  canvas: document.getElementById("ai-icon"), // 32 × 32 CSS px для слота 24 × 24
  artboard: "ai_icon",
  stateMachines: "icon_button_sm",
  autoplay: !matchMedia("(prefers-reduced-motion: reduce)").matches,
  autoBind: true,
  layout: new rive.Layout({ fit: rive.Fit.Contain }),
  onLoad: () => {
    r.resizeDrawingSurfaceToCanvas();
    const vm = r.viewModelInstance;
    vm.enum("color").value = "grey"; // white | grey | gradient
    vm.number("pause").value = 0; // мс, 0 по умолчанию
  }
});
```

iOS (SwiftUI, RiveRuntime с Data Binding), набросок, сверить с версией SDK:

```swift
let vm = RiveViewModel(fileName: "ai_icon", stateMachineName: "icon_button_sm", fit: .contain, artboardName: "ai_icon")
// после загрузки: экземпляр по умолчанию
if let instance = vm.riveModel?.riveFile.viewModelNamed("AiIconButton")?.createDefaultInstance() {
  vm.riveModel?.stateMachine?.bind(viewModelInstance: instance)
  instance.enumProperty(fromPath: "color")?.value = "grey"
  instance.numberProperty(fromPath: "pause")?.value = 0
}
vm.view().frame(width: 32, height: 32)
```

Android (rive-android с Data Binding), набросок, сверить с версией SDK:

```kotlin
riveView.setRiveResource(R.raw.ai_icon, artboardName = "ai_icon", stateMachineName = "icon_button_sm", fit = Fit.CONTAIN, autoBind = true)
val vmi = riveView.controller.stateMachines.first().viewModelInstance!!
vmi.getEnumProperty("color").value = "grey"
vmi.getNumberProperty("pause").value = 0f
```

### 7. Поведение

- Иконка запускается при показе и повторяется бесконечно, пока виден экран или идёт процесс.
- Когда иконка уходит с экрана, Rive-вью можно поставить на паузу или выгрузить. При возвращении начните с начала: первый кадр совпадает с покоем, скачка не будет.
- Смена `color` срабатывает сразу, без перехода. Смена `pause` действует со следующей паузы.
- Reduced motion (iOS Reduce Motion, Android «Удалить анимации», web `prefers-reduced-motion`): стейт-машину не запускать, показать первый кадр — это поза покоя, как в макете. Цвет при этом задаётся так же.
- Событий для аналитики и хаптики нет.

### 8. Проверка

- Цикл: 2000 мс движения + `pause`. При `pause` = 0 (по умолчанию) полный цикл 2000 мс, при `pause` = 2000 — 4000 мс.
- В покое иконка в Rive-вью совпадает с иконкой из макета. Если звёзды мельче или крупнее, проверьте, что область 4/3 от слота, а fit Contain.
- Значения по умолчанию после auto-bind: `color` = `white`, `pause` = 0.
- Без привязки вью-модели иконка тоже играет: белая, без паузы.

### 9. Исходник текстом: RML и YAML

Rive CLI хранит проект как текст, его удобно читать, сравнивать между версиями и держать в git. Ниже исходник файла после `rive create --from-remote-file=2270880`, проверка `rive . --verify`: 0 ошибок, 0 предупреждений. Сборка `.riv`: `rive . --publish=local` (скриптов нет, подпись не нужна).

<details class="md-fold"><summary>Показать scene.rml · 555 строк</summary>

```xml
<Rive version="1" kind="fragment">
    <Artboard defaultStateMachineId="1:9370" viewModelId="9:10" viewModelInstanceId="9:11" isComponent="true" spaceId="1:10082" width="32" height="32" styleId="1:9302" name="ai_icon_art" id="1:9176">
        <LayoutComponent width="32" height="32" styleId="1:9266" name="Icon" id="1:9265">
            <Node x="18.5" y="16" name="IconPSR" id="1:9267">
                <Shape x="4" y="-6.5" opacity="0" name="Gradient Small" complexDependency="true" id="9:600">
                    <Ellipse width="7" height="7" name="Path" id="9:601"/>
                    <Fill fillRule="clockwise" name="Fill 1" id="9:602">
                        <LinearGradient startX="-1.5850391" startY="-4.2000122" endX="2.1091685" endY="3.5181141" name="Component" id="9:603">
                            <GradientStop colorValue="FF00D4FF" name="Component" id="9:604"/>
                            <GradientStop colorValue="FF4E33FE" position="1" name="Component" id="9:605"/>
                            <GradientStop colorValue="FF0077FF" position="0.51428574" name="Component" id="9:606"/>
                        </LinearGradient>
                    </Fill>
                    <ScaleConstraint maxValueY="0.94999999" minValue="0.89999998" maxValue="0.94999999" targetId="1:9268" name="Scale" id="9:607"/>
                    <ClippingShape sourceId="1:9268" id="9:608"/>
                    <TranslationConstraint targetId="1:9268" name="Translation" id="9:609"/>
                </Shape>
                <Shape x="2.2450638" y="7.2623124" opacity="0.40000001" name="Small" id="1:9268">
                    <Star innerRadius="0.23" points="4" cornerRadius="4" width="10.2" height="10.2" name="Path" id="1:9269"/>
                    <Fill fillRule="clockwise" name="Fill 1" id="1:9270">
                        <SolidColor colorValue="FFFFFFFF" name="Component" id="1:9271"/>
                    </Fill>
                </Shape>
                <Shape x="-5.5" opacity="0" name="Gradient Medium" complexDependency="true" id="9:610">
                    <Ellipse width="8.3999996" height="8.3999996" name="Path" id="9:611"/>
                    <Fill fillRule="clockwise" name="Fill 1" id="9:612">
                        <LinearGradient startX="-1.5850391" startY="-4.2000122" endX="2.1091685" endY="3.5181141" name="Component" id="9:613">
                            <GradientStop colorValue="FF00D4FF" name="Component" id="9:614"/>
                            <GradientStop colorValue="FF4E33FE" position="1" name="Component" id="9:615"/>
                            <GradientStop colorValue="FF0077FF" position="0.51428574" name="Component" id="9:616"/>
                        </LinearGradient>
                    </Fill>
                    <ScaleConstraint maxValueY="0.94999999" minValue="0.89999998" maxValue="0.94999999" targetId="1:9272" name="Scale" id="9:617"/>
                    <ClippingShape sourceId="1:9272" id="9:618"/>
                    <TranslationConstraint targetId="1:9272" name="Translation" id="9:619"/>
                </Shape>
                <Shape x="4" y="-6.5" opacity="0.80000001" name="Medium" id="1:9272">
                    <Star innerRadius="0.2" points="4" cornerRadius="4" width="12.5" height="12.5" name="Path" id="1:9273"/>
                    <Fill fillRule="clockwise" name="Fill 1" id="1:9274">
                        <SolidColor colorValue="FFFFFFFF" name="Component" id="1:9275"/>
                    </Fill>
                </Shape>
                <Shape x="-5.5" opacity="0" name="Gradient Big" complexDependency="true" id="9:620">
                    <Ellipse width="16" height="16" name="Path" id="9:621"/>
                    <Fill fillRule="clockwise" name="Fill 1" id="9:622">
                        <LinearGradient startX="-2.1901889" startY="-7.9999866" endX="3.1897931" endY="7.7217431" name="Component" id="9:623">
                            <GradientStop colorValue="FF00D4FF" name="Component" id="9:624"/>
                            <GradientStop colorValue="FF4E33FE" position="1" name="Component" id="9:625"/>
                            <GradientStop colorValue="FF0077FF" position="0.51428574" name="Component" id="9:626"/>
                        </LinearGradient>
                    </Fill>
                    <ScaleConstraint maxValueY="0.94999999" minValue="0.89999998" maxValue="0.94999999" targetId="1:9276" name="Scale" id="9:627"/>
                    <ClippingShape sourceId="1:9276" id="9:628"/>
                    <TranslationConstraint targetId="1:9276" name="Translation" id="9:629"/>
                </Shape>
                <Shape x="-5.5" name="Big" id="1:9276">
                    <Star innerRadius="0.25999999" points="4" cornerRadius="4" width="21.200001" height="21.200001" name="Path" id="1:9277"/>
                    <Fill fillRule="clockwise" name="Fill 1" id="1:9278">
                        <SolidColor colorValue="FFFFFFFF" name="Component" id="1:9279"/>
                    </Fill>
                </Shape>
            </Node>
            <LayoutComponentStyle edgeConstraints="16" layoutAlignmentType="center" name="Layout Style" id="1:9266"/>
            <ForegroundLayoutDrawable name="Fills and Strokes Foreground" id="1:9414"/>
        </LayoutComponent>
        <LayoutComponentStyle layoutAlignmentType="center" name="Layout Style" id="1:9302"/>
        <StateMachine name="icon_button_sm" id="1:9370">
            <StateMachineLayer name="Main" id="1:9371">
                <AnimationState animationId="1:9311" x="201" y="2" reset="true" id="1:9325">
                    <StateTransition stateToId="9:30" enableExitTime="true" exitTimeIsPercetange="true" duration="0" exitTime="50" id="9:31">
                        <TransitionViewModelCondition id="0:10084">
                            <TransitionPropertyViewModelComparator id="0:10085">
                                <BindablePropertyEnum id="0:10086">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10087"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:21" id="0:10088"/>
                        </TransitionViewModelCondition>
                        <DataBindContext sourcePathIds="9:10-9:13" propertyKey="158" converterId="0:0" id="0:10083"/>
                    </StateTransition>
                    <StateTransition stateToId="9:30" enableExitTime="true" exitTimeIsPercetange="true" duration="0" exitTime="50" id="9:33">
                        <TransitionViewModelCondition id="0:10090">
                            <TransitionPropertyViewModelComparator id="0:10091">
                                <BindablePropertyEnum id="0:10092">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10093"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:22" id="0:10094"/>
                        </TransitionViewModelCondition>
                        <DataBindContext sourcePathIds="9:10-9:13" propertyKey="158" converterId="0:0" id="0:10089"/>
                    </StateTransition>
                    <StateTransition stateToId="9:30" enableExitTime="true" exitTimeIsPercetange="true" duration="0" exitTime="50" id="9:34">
                        <TransitionViewModelCondition id="0:10107">
                            <TransitionPropertyViewModelComparator id="0:10108">
                                <BindablePropertyEnum id="0:10109">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10110"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:23" id="0:10111"/>
                        </TransitionViewModelCondition>
                        <DataBindContext sourcePathIds="9:10-9:13" propertyKey="158" converterId="0:0" id="0:10112"/>
                    </StateTransition>
                    <StateTransition stateToId="9:30" enableExitTime="true" exitTimeIsPercetange="true" duration="0" exitTime="50" id="0:10133">
                        <DataBindContext sourcePathIds="9:10-9:13" propertyKey="158" converterId="0:0" id="0:10134"/>
                    </StateTransition>
                </AnimationState>
                <ExitState x="480" id="1:9372"/>
                <EntryState id="1:9373">
                    <StateTransition stateToId="1:9325" id="1:9326"/>
                </EntryState>
                <AnyState x="400" id="1:9374"/>
                <AnimationState animationId="9:40" x="201" y="120" id="9:30">
                    <StateTransition stateToId="1:9325" id="9:32"/>
                </AnimationState>
            </StateMachineLayer>
            <StateMachineLayer name="Color" id="9:60">
                <EntryState id="9:61">
                    <StateTransition stateToId="9:62" id="9:66"/>
                </EntryState>
                <AnimationState animationId="9:41" x="201" y="2" id="9:62">
                    <StateTransition stateToId="9:69" id="9:751">
                        <TransitionViewModelCondition id="0:10113">
                            <TransitionPropertyViewModelComparator id="0:10114">
                                <BindablePropertyEnum id="0:10115">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10116"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:23" id="0:10117"/>
                        </TransitionViewModelCondition>
                    </StateTransition>
                    <StateTransition stateToId="9:63" id="9:67">
                        <TransitionViewModelCondition id="0:10095">
                            <TransitionPropertyViewModelComparator id="0:10096">
                                <BindablePropertyEnum id="0:10097">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10098"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:22" id="0:10099"/>
                        </TransitionViewModelCondition>
                    </StateTransition>
                </AnimationState>
                <AnimationState animationId="9:42" x="584" y="113" id="9:63">
                    <StateTransition stateToId="9:69" id="9:752">
                        <TransitionViewModelCondition id="0:10118">
                            <TransitionPropertyViewModelComparator id="0:10119">
                                <BindablePropertyEnum id="0:10120">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10121"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:23" id="0:10122"/>
                        </TransitionViewModelCondition>
                    </StateTransition>
                    <StateTransition stateToId="9:62" id="9:68">
                        <TransitionViewModelCondition id="0:10100">
                            <TransitionPropertyViewModelComparator id="0:10101">
                                <BindablePropertyEnum id="0:10102">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10103"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:21" id="0:10104"/>
                        </TransitionViewModelCondition>
                    </StateTransition>
                </AnimationState>
                <AnyState x="400" id="9:64"/>
                <ExitState x="480" id="9:65"/>
                <AnimationState animationId="9:43" x="201" y="240" id="9:69">
                    <StateTransition stateToId="9:62" id="9:753">
                        <TransitionViewModelCondition id="0:10123">
                            <TransitionPropertyViewModelComparator id="0:10124">
                                <BindablePropertyEnum id="0:10125">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10126"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:21" id="0:10127"/>
                        </TransitionViewModelCondition>
                    </StateTransition>
                    <StateTransition stateToId="9:63" id="9:754">
                        <TransitionViewModelCondition id="0:10128">
                            <TransitionPropertyViewModelComparator id="0:10129">
                                <BindablePropertyEnum id="0:10130">
                                    <DataBindContext sourcePathIds="9:10-9:12" propertyKey="637" id="0:10131"/>
                                </BindablePropertyEnum>
                            </TransitionPropertyViewModelComparator>
                            <TransitionValueEnumComparator value="9:22" id="0:10132"/>
                        </TransitionViewModelCondition>
                    </StateTransition>
                </AnimationState>
            </StateMachineLayer>
        </StateMachine>
        <LinearAnimation duration="1" name="rest" id="9:40">
            <KeyedObject objectId="1:9268" id="9:401">
                <KeyedProperty property="opacity" id="9:402">
                    <KeyFrameDouble value="0.40000001" id="9:403"/>
                </KeyedProperty>
                <KeyedProperty property="scaleX" id="9:404">
                    <KeyFrameDouble value="1" id="9:405"/>
                </KeyedProperty>
                <KeyedProperty property="scaleY" id="9:406">
                    <KeyFrameDouble value="1" id="9:407"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9272" id="9:411">
                <KeyedProperty property="opacity" id="9:412">
                    <KeyFrameDouble value="0.80000001" id="9:413"/>
                </KeyedProperty>
                <KeyedProperty property="scaleX" id="9:414">
                    <KeyFrameDouble value="1" id="9:415"/>
                </KeyedProperty>
                <KeyedProperty property="scaleY" id="9:416">
                    <KeyFrameDouble value="1" id="9:417"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9276" id="9:421">
                <KeyedProperty property="opacity" id="9:422">
                    <KeyFrameDouble value="1" id="9:423"/>
                </KeyedProperty>
                <KeyedProperty property="scaleX" id="9:424">
                    <KeyFrameDouble value="1" id="9:425"/>
                </KeyedProperty>
                <KeyedProperty property="scaleY" id="9:426">
                    <KeyFrameDouble value="1" id="9:427"/>
                </KeyedProperty>
            </KeyedObject>
        </LinearAnimation>
        <LinearAnimation duration="1" name="color_white" id="9:41">
            <KeyedObject objectId="1:9271" id="9:431">
                <KeyedProperty property="colorValue" id="9:432">
                    <KeyFrameColor value="FFFFFFFF" id="9:433"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9275" id="9:434">
                <KeyedProperty property="colorValue" id="9:435">
                    <KeyFrameColor value="FFFFFFFF" id="9:436"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9279" id="9:437">
                <KeyedProperty property="colorValue" id="9:438">
                    <KeyFrameColor value="FFFFFFFF" id="9:439"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:600" id="9:700">
                <KeyedProperty property="opacity" id="9:7001">
                    <KeyFrameDouble id="9:7002"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:610" id="9:701">
                <KeyedProperty property="opacity" id="9:7011">
                    <KeyFrameDouble id="9:7012"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:620" id="9:702">
                <KeyedProperty property="opacity" id="9:7021">
                    <KeyFrameDouble id="9:7022"/>
                </KeyedProperty>
            </KeyedObject>
        </LinearAnimation>
        <LinearAnimation duration="120" loopValue="loop" name="color_gradient" id="9:43">
            <KeyedObject objectId="1:9271" id="9:720">
                <KeyedProperty property="colorValue" id="9:7201">
                    <KeyFrameColor value="00FFFFFF" id="9:7202"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9275" id="9:721">
                <KeyedProperty property="colorValue" id="9:7211">
                    <KeyFrameColor value="00FFFFFF" id="9:7212"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9279" id="9:722">
                <KeyedProperty property="colorValue" id="9:7221">
                    <KeyFrameColor value="00FFFFFF" id="9:7222"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:600" id="9:730">
                <KeyedProperty property="opacity" id="9:7301">
                    <KeyFrameDouble value="1" id="9:7302"/>
                </KeyedProperty>
                <KeyedProperty property="rotation" id="9:7401">
                    <KeyFrameDouble value="-3.1415927" interpolationType="linear" id="9:7402"/>
                    <KeyFrameDouble value="3.1415927" interpolationType="linear" frame="120" id="9:7403"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:610" id="9:731">
                <KeyedProperty property="opacity" id="9:7311">
                    <KeyFrameDouble value="1" id="9:7312"/>
                </KeyedProperty>
                <KeyedProperty property="rotation" id="9:7411">
                    <KeyFrameDouble value="-1.5707964" interpolationType="linear" id="9:7412"/>
                    <KeyFrameDouble value="4.712389" interpolationType="linear" frame="120" id="9:7413"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:620" id="9:732">
                <KeyedProperty property="opacity" id="9:7321">
                    <KeyFrameDouble value="1" id="9:7322"/>
                </KeyedProperty>
                <KeyedProperty property="rotation" id="9:7421">
                    <KeyFrameDouble interpolationType="linear" id="9:7422"/>
                    <KeyFrameDouble value="6.2831855" interpolationType="linear" frame="120" id="9:7423"/>
                </KeyedProperty>
            </KeyedObject>
        </LinearAnimation>
        <LinearAnimation duration="1" name="color_grey" id="9:42">
            <KeyedObject objectId="1:9271" id="9:441">
                <KeyedProperty property="colorValue" id="9:442">
                    <KeyFrameColor value="FFA2AAB8" id="9:443"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9275" id="9:444">
                <KeyedProperty property="colorValue" id="9:445">
                    <KeyFrameColor value="FFA2AAB8" id="9:446"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9279" id="9:447">
                <KeyedProperty property="colorValue" id="9:448">
                    <KeyFrameColor value="FFA2AAB8" id="9:449"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:600" id="9:710">
                <KeyedProperty property="opacity" id="9:7101">
                    <KeyFrameDouble id="9:7102"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:610" id="9:711">
                <KeyedProperty property="opacity" id="9:7111">
                    <KeyFrameDouble id="9:7112"/>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="9:620" id="9:712">
                <KeyedProperty property="opacity" id="9:7121">
                    <KeyFrameDouble id="9:7122"/>
                </KeyedProperty>
            </KeyedObject>
        </LinearAnimation>
        <LinearAnimation duration="240" loopValue="loop" name="icon" id="1:9311">
            <KeyedObject objectId="1:9276" id="1:9312">
                <KeyedProperty property="scaleY" id="1:9313">
                    <KeyFrameDouble value="1" interpolationType="cubicValue" id="1:9318">
                        <CubicValueInterpolator x1="0.55574715" y1="0.9909659" x2="0.66932106" y2="0.089283317" id="1:9382"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble interpolationType="cubicValue" frame="30" id="1:9317">
                        <CubicValueInterpolator autoBezier="true" x1="0.33010167" y1="-0.089127451" x2="0.67002815" y2="0.29501405" id="1:9383"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="0.46000001" interpolationType="cubicValue" frame="60" id="1:9316">
                        <CubicValueInterpolator autoBezier="true" x1="0.33003047" y1="0.62501526" x2="0.66983974" y2="0.91085672" id="1:9384"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="90" id="1:9315">
                        <CubicValueInterpolator autoBezier="true" x1="0.32994655" y1="1.0890856" x2="0.67000002" id="1:9385"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="120" id="1:9314">
                        <CubicValueInterpolator autoBezier="true" y1="1" id="1:9386"/>
                    </KeyFrameDouble>
                </KeyedProperty>
                <KeyedProperty property="scaleX" id="1:9319">
                    <KeyFrameDouble value="1" interpolationType="cubicValue" id="1:9324">
                        <CubicValueInterpolator x1="0.55720401" y1="0.98961163" x2="0.66932106" y2="0.089283317" id="1:9377"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble interpolationType="cubicValue" frame="30" id="1:9323">
                        <CubicValueInterpolator autoBezier="true" x1="0.33010167" y1="-0.089127451" x2="0.67002815" y2="0.29501405" id="1:9378"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="0.46000001" interpolationType="cubicValue" frame="60" id="1:9322">
                        <CubicValueInterpolator autoBezier="true" x1="0.33003047" y1="0.62501526" x2="0.66983974" y2="0.91085672" id="1:9379"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="90" id="1:9321">
                        <CubicValueInterpolator autoBezier="true" x1="0.32994655" y1="1.0890856" x2="0.67000002" id="1:9380"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="120" id="1:9320">
                        <CubicValueInterpolator autoBezier="true" y1="1" id="1:9381"/>
                    </KeyFrameDouble>
                </KeyedProperty>
                <KeyedProperty property="opacity" id="9:502">
                    <KeyFrameDouble value="1" interpolationType="cubic" id="9:503">
                        <CubicEaseInterpolator id="9:504"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="0.40000001" interpolationType="linear" frame="60" id="9:505"/>
                    <KeyFrameDouble value="1" interpolationType="linear" frame="90" id="9:506"/>
                    <KeyFrameDouble value="1" interpolationType="cubic" frame="120" id="9:507">
                        <CubicEaseInterpolator id="9:508"/>
                    </KeyFrameDouble>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9272" id="1:9333">
                <KeyedProperty property="scaleY" id="1:9334">
                    <KeyFrameDouble value="1" interpolationType="cubicValue" id="1:9339">
                        <CubicValueInterpolator x1="0.54245162" y1="1.0074543" x2="0.66959006" y2="2.0652051" id="1:9392"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1.9" interpolationType="cubicValue" frame="30" id="1:9338">
                        <CubicValueInterpolator autoBezier="true" x1="0.33245215" y1="1.7337739" x2="0.66755527" y2="0.1695468" id="1:9393"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble interpolationType="cubicValue" frame="60" id="1:9337">
                        <CubicValueInterpolator autoBezier="true" x1="0.33037651" y1="-0.16849202" x2="0.6696161" y2="0.71480805" id="1:9394"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="0.88" interpolationType="cubicValue" frame="90" id="1:9336">
                        <CubicValueInterpolator autoBezier="true" x1="0.32982737" y1="1.0449136" x2="0.67000002" y2="0.96039999" id="1:9395"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="120" id="1:9335">
                        <CubicValueInterpolator autoBezier="true" y1="1" id="1:9396"/>
                    </KeyFrameDouble>
                </KeyedProperty>
                <KeyedProperty property="scaleX" id="1:9340">
                    <KeyFrameDouble value="1" interpolationType="cubicValue" id="1:9345">
                        <CubicValueInterpolator x1="0.54244101" y1="1.0058284" x2="0.66959006" y2="2.0652051" id="1:9387"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1.9" interpolationType="cubicValue" frame="30" id="1:9344">
                        <CubicValueInterpolator autoBezier="true" x1="0.33245215" y1="1.7337739" x2="0.66755527" y2="0.1695468" id="1:9388"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble interpolationType="cubicValue" frame="60" id="1:9343">
                        <CubicValueInterpolator autoBezier="true" x1="0.33037651" y1="-0.16849202" x2="0.6696161" y2="0.71480805" id="1:9389"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="0.88" interpolationType="cubicValue" frame="90" id="1:9342">
                        <CubicValueInterpolator autoBezier="true" x1="0.32982737" y1="1.0449136" x2="0.67000002" y2="0.96039999" id="1:9390"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="120" id="1:9341">
                        <CubicValueInterpolator autoBezier="true" y1="1" id="1:9391"/>
                    </KeyFrameDouble>
                </KeyedProperty>
                <KeyedProperty property="opacity" id="9:512">
                    <KeyFrameDouble value="0.80000001" interpolationType="cubic" id="9:513">
                        <CubicEaseInterpolator id="9:514"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="linear" frame="30" id="9:515"/>
                    <KeyFrameDouble value="0.80000001" interpolationType="cubic" frame="120" id="9:516">
                        <CubicEaseInterpolator id="9:517"/>
                    </KeyFrameDouble>
                </KeyedProperty>
            </KeyedObject>
            <KeyedObject objectId="1:9268" id="1:9346">
                <KeyedProperty property="scaleY" id="1:9347">
                    <KeyFrameDouble value="1" interpolationType="cubicValue" id="1:9352">
                        <CubicValueInterpolator x1="0.50735068" y1="1.0099971" x2="0.6702522" y2="0.99555635" id="1:9402"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1.2" interpolationType="cubicValue" frame="30" id="1:9351">
                        <CubicValueInterpolator autoBezier="true" x1="0.33051002" y1="1.4049162" x2="0.6694721" y2="2.4383168" id="1:9403"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="2.24" interpolationType="cubicValue" frame="60" id="1:9350">
                        <CubicValueInterpolator autoBezier="true" x1="0.33339268" y1="2.0399644" x2="0.66662538" y2="0.20669226" id="1:9404"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble interpolationType="cubicValue" frame="90" id="1:9349">
                        <CubicValueInterpolator autoBezier="true" x1="0.33045036" y1="-0.20487922" x2="0.37190115" y2="0.9989" id="1:9405"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="120" id="1:9348">
                        <CubicValueInterpolator y1="1" id="1:9406"/>
                    </KeyFrameDouble>
                </KeyedProperty>
                <KeyedProperty property="scaleX" id="1:9353">
                    <KeyFrameDouble value="1" interpolationType="cubicValue" id="1:9358">
                        <CubicValueInterpolator x1="0.5085237" y1="1.0082796" x2="0.6702522" y2="0.99555635" id="1:9397"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1.2" interpolationType="cubicValue" frame="30" id="1:9357">
                        <CubicValueInterpolator autoBezier="true" x1="0.33051002" y1="1.4049162" x2="0.6694721" y2="2.4383168" id="1:9398"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="2.24" interpolationType="cubicValue" frame="60" id="1:9356">
                        <CubicValueInterpolator autoBezier="true" x1="0.33339268" y1="2.0399644" x2="0.66662538" y2="0.20669226" id="1:9399"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble interpolationType="cubicValue" frame="90" id="1:9355">
                        <CubicValueInterpolator autoBezier="true" x1="0.33045036" y1="-0.20487922" x2="0.37190115" y2="0.99886328" id="1:9400"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="1" interpolationType="cubicValue" frame="120" id="1:9354">
                        <CubicValueInterpolator y1="1" id="1:9401"/>
                    </KeyFrameDouble>
                </KeyedProperty>
                <KeyedProperty property="opacity" id="9:522">
                    <KeyFrameDouble value="0.40000001" interpolationType="cubic" id="9:523">
                        <CubicEaseInterpolator id="9:524"/>
                    </KeyFrameDouble>
                    <KeyFrameDouble value="0.80000001" interpolationType="linear" frame="30" id="9:525"/>
                    <KeyFrameDouble value="1" interpolationType="linear" frame="60" id="9:526"/>
                    <KeyFrameDouble value="0.40000001" interpolationType="cubic" frame="120" id="9:527">
                        <CubicEaseInterpolator id="9:528"/>
                    </KeyFrameDouble>
                </KeyedProperty>
            </KeyedObject>
        </LinearAnimation>
    </Artboard>
    <Artboard defaultStateMachineId="9:810" viewModelId="9:10" viewModelInstanceId="9:11" isComponent="true" spaceId="1:10082" width="32" height="32" styleId="9:801" x="-60" name="ai_icon" id="9:800">
        <LayoutComponentStyle layoutAlignmentType="center" id="9:801"/>
        <LayoutComponent width="32" height="32" styleId="9:803" name="Fit" id="9:802">
            <LayoutComponentStyle layoutWidthScaleType="fill" layoutHeightScaleType="fill" widthUnitsValue="auto" heightUnitsValue="auto" id="9:803"/>
            <NestedArtboardLeaf fit="contain" artboardId="1:9176" isStateful="true" name="ai_icon_art" id="9:804">
                <NestedStateMachine animationId="1:9370" id="9:805"/>
                <ViewModelInstance viewModelId="9:10" exports="true" name="ai_icon_art" id="9:806">
                    <ViewModelInstanceEnum propertyValue="9:21" playbackValue="9:21" viewModelPropertyId="9:12" id="9:807">
                        <DataBindContext sourcePathIds="9:10-9:12" propertyKey="560" id="9:808"/>
                    </ViewModelInstanceEnum>
                    <ViewModelInstanceNumber propertyValue="0" viewModelPropertyId="9:13" id="9:809">
                        <DataBindContext sourcePathIds="9:10-9:13" propertyKey="575" id="9:819"/>
                    </ViewModelInstanceNumber>
                </ViewModelInstance>
            </NestedArtboardLeaf>
        </LayoutComponent>
        <StateMachine name="icon_button_sm" id="9:810">
            <StateMachineLayer name="Layer 1" id="9:811">
                <AnyState x="400" id="9:812"/>
                <ExitState x="480" id="9:813"/>
                <EntryState id="9:814">
                    <StateTransition stateToId="9:815" id="9:816"/>
                </EntryState>
                <AnimationState animationId="9:817" x="140" id="9:815"/>
            </StateMachineLayer>
        </StateMachine>
        <LinearAnimation duration="1" name="idle" id="9:817"/>
    </Artboard>
    <ComponentAsset artboardId="1:9176" name="Button 2.1 2" id="1:9413"/>
    <ComponentAsset artboardId="9:800" name="ai_icon" id="9:820"/>
    <CubicEaseInterpolator id="0:717"/>
    <CubicEaseInterpolator id="0:718"/>
    <CubicEaseInterpolator id="1:8077"/>
    <CubicEaseInterpolator id="1:8078"/>
    <CubicEaseInterpolator id="1:8079"/>
    <CubicEaseInterpolator id="1:8080"/>
    <CubicEaseInterpolator id="1:8081"/>
    <CubicEaseInterpolator id="1:8082"/>
    <CubicEaseInterpolator id="1:8083"/>
    <CubicEaseInterpolator id="1:8084"/>
    <CubicEaseInterpolator id="1:8085"/>
    <CubicEaseInterpolator id="1:8086"/>
    <CubicValueInterpolator autoBezier="true" x1="0.33000001" y1="13.584" x2="0.66304642" y2="15.991311" id="0:805"/>
    <CubicValueInterpolator autoBezier="true" x1="0.33000001" y1="18.799999" x2="0.64206475" y2="23.710323" id="0:806"/>
    <CubicValueInterpolator x1="0.52108109" y1="9.211854" x2="0.69744486" y2="5.3664546" id="1:8351"/>
    <CubicValueInterpolator y2="0" id="1:8352"/>
    <CubicValueInterpolator y1="2.2450638" y2="-5.5" id="1:8353"/>
    <CubicValueInterpolator x1="0.61695224" y1="-0.97529334" x2="0.59640372" y2="5.3246155" id="1:8354"/>
    <CubicValueInterpolator y1="-5.5" y2="-5.5" id="1:8355"/>
    <CubicValueInterpolator x1="0.44111022" y1="-6.5504375" x2="0.66803235" y2="0.5478912" id="1:8356"/>
    <CubicValueInterpolator x1="0.29204792" y1="7.9812336" x2="0.5393101" y2="8.1697865" id="1:8357"/>
    <CubicValueInterpolator x1="0.29589483" y1="-4.0080523" x2="0.614779" y2="-10.760186" id="1:8358"/>
    <CubicValueInterpolator y1="7.2623124" x2="0.77823192" y2="6.1883359" id="1:8359"/>
    <CubicValueInterpolator x1="0.23482788" y1="-6.0276947" x2="0.65327579" y2="-1.6933435" id="1:8360"/>
    <CubicValueInterpolator x1="0.46981755" y1="9.7457886" x2="0.79169607" y2="5.7007356" id="1:8361"/>
    <CubicValueInterpolator x1="0.39926806" y1="-3.4625819" x2="0.59113115" y2="-8.7239351" id="1:8362"/>
    <CubicValueInterpolator y1="4" y2="4" id="1:8363"/>
    <CubicValueInterpolator x1="0.44045627" y1="-4.601747" x2="0.78069687" y2="1.0113586" id="1:8364"/>
    <CubicValueInterpolator y1="-6.5" y2="-6.5" id="1:8365"/>
    <CubicValueInterpolator x1="0.48380229" y1="9.5772047" x2="0.6811958" y2="4.4451156" id="1:8366"/>
    <CubicValueInterpolator x1="0.41018993" y1="-4.2728267" x2="0.57680851" y2="6.46" id="1:8367"/>
    <CubicValueInterpolator x1="0.31269115" y1="-2.928472" x2="0.51708037" y2="-4.3499346" id="1:8368"/>
    <CubicValueInterpolator x1="0.41265151" y1="8.0400753" x2="0.61670154" y2="4.0537543" id="1:8369"/>
    <CubicValueInterpolator x1="0.37511307" y1="-0.71731943" x2="0.58363008" y2="-6.3507833" id="1:8370"/>
    <CubicValueInterpolator x1="0.36476648" y1="-3.39237" x2="0.59541774" y2="-5.9667473" id="1:8371"/>
    <CubicValueInterpolator x1="0.22578511" y1="-3.8977079" x2="0.79092741" y2="-8.2315855" id="1:8372"/>
    <CubicValueInterpolator x1="0.27891073" y1="-4.1918402" x2="0.63429195" y2="1.9443156" id="1:8373"/>
    <CubicValueInterpolator x1="0.30328164" y1="8.9802952" x2="0.67748266" y2="7.2155948" id="1:8374"/>
    <DataEnumCustom name="AiIconColor" id="9:20">
        <DataEnumValue key="white" value="white" id="9:21"/>
        <DataEnumValue key="grey" value="grey" id="9:22"/>
        <DataEnumValue key="gradient" value="gradient" id="9:23"/>
    </DataEnumCustom>
    <Space name="Space 1" id="1:10082"/>
    <ViewModel defaultInstanceId="9:11" name="AiIconButton" id="9:10">
        <ViewModelPropertyEnumCustom enumId="9:20" name="color" id="9:12"/>
        <ViewModelPropertyNumber name="pause" id="9:13"/>
        <ViewModelInstance exports="true" name="Default" id="9:11">
            <ViewModelInstanceEnum propertyValue="9:21" playbackValue="9:21" viewModelPropertyId="9:12" id="0:10105"/>
            <ViewModelInstanceNumber propertyValue="0" viewModelPropertyId="9:13" id="0:10106"/>
        </ViewModelInstance>
    </ViewModel>
</Rive>
```

</details>

`rive.yaml` — конфиг проекта Rive CLI:

```yaml
name: src
logs:
  file: build/rive.log
  problems: build/problems.log
main: "ai_icon"
push:
  projectId: 789242
  fileId: 2270880
```

### 10. Машиночитаемая спека (JSON)

<details class="md-fold"><summary>Показать JSON</summary>

```json
{
 "name": "Kolesa AI icon · Rive",
 "version": "1.1",
 "date": "2026-11-09",
 "ticket": "GD-11154",
 "file": {
  "name": "ai_icon.riv",
  "bytes": 3606,
  "sha256": "9479cf66441a735bead4f0be206435031eacb8396335b6927dc09cc5f9961afa",
  "rive_file": "ai_icon_prod",
  "rive_file_id": 2270880,
  "revision": "v7 default pause 0"
 },
 "artboard": {
  "name": "ai_icon",
  "size": [
   32,
   32
  ],
  "background": "transparent",
  "nested": "ai_icon_art (fit contain)"
 },
 "state_machine": "icon_button_sm",
 "inputs": [],
 "events": [],
 "view_model": {
  "name": "AiIconButton",
  "instance": "Default",
  "auto_bind": true,
  "properties": {
   "color": {
    "type": "enum",
    "enum": "AiIconColor",
    "values": [
     "white",
     "grey",
     "gradient"
    ],
    "default": "white"
   },
   "pause": {
    "type": "number",
    "unit": "ms",
    "default": 0,
    "min": 0
   }
  }
 },
 "colors": {
  "white": "#FFFFFF",
  "grey": "#A2AAB8",
  "gradient": [
   "#00D4FF",
   "#0077FF",
   "#4E33FE"
  ]
 },
 "star_opacity": {
  "big": 1.0,
  "top_small": 0.8,
  "bottom_small": 0.4
 },
 "cycle": {
  "motion_ms": 2000,
  "pause_ms": "pause",
  "total_ms": "2000 + pause",
  "loop": "infinite",
  "fps": 60
 },
 "layout": {
  "fit": "contain",
  "alignment": "center",
  "stars_rest_bbox_fraction": [
   0.171,
   0.171,
   0.829,
   0.833
  ],
  "area_per_slot": "slot * 4/3, same center",
  "example_slot_24": {
   "area": [
    32,
    32
   ],
   "offset": [
    -4,
    -4
   ]
  },
  "example_slot_80": {
   "area": [
    106.67,
    106.67
   ],
   "offset": [
    -13.33,
    -13.33
   ]
  }
 },
 "presets": [
  {
   "where": "Kolesa AI plate in 'checking photos' sheet",
   "color": "white",
   "pause": 0,
   "slot": [
    24,
    24
   ]
  },
  {
   "where": "'enter car details' button on Car valuation screen",
   "color": "white",
   "pause": 2000,
   "slot": [
    24,
    24
   ]
  },
  {
   "where": "disabled button 'improving description'",
   "color": "grey",
   "pause": 0,
   "slot": [
    24,
    24
   ]
  },
  {
   "where": "loader replacement",
   "color": "gradient",
   "pause": 2000,
   "slot": [
    80,
    80
   ]
  }
 ],
 "reduced_motion": "do not start the state machine; show first frame (rest pose)"
}
```

</details>
