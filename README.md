# MVP-Clean_Architecture

### Run Requirements

* Xcode 9.1

## ⚡️Generatus⚡️
Для более комфортной разработки вы можете использовать
*Generatus: https://github.com/Ryasnoy/Generatus


### High Level Layers

#### MVP Concepts
##### Presentation Logic
* `View` - делегирует события взаимодействия с пользователем в `Presenter` и отображает данные, переданные `Presenter`
* Все `UIViewController`, `UIView`, `UITableViewCell` , сабклассы принадлежат слою `View`
* Обычно `View` является пассивным - оно не должно содержать сложной логики, поэтому в большинстве случаев нам не нужно писать Unit Tests для него.
* `Presenter` - содержит логику представления и сообщает `View`, что необходимо отобразить
* Чаще всего нам необходим один `Presenter` на одно представление (view controller)
* Он не ссылается на конкретный тип `View`, а скорее ссылается на протокол `View`, который обычно реализуется подклассом `UIViewController`
* Он должен быть простым классом `Swift` и не ссылаться ни на какие классы `iOS` фреймворков - это упрощает повторное использование, возможно, в приложении `macOS`
* Он должен покрываться Unit Tests
* `Configurator` - вводит граф зависимостей для объекта представления (view controller)
* Вы можете с легкостью использовать DI (dependency injection) библиотеки. К сожалению DI библиотеки еще недостаточно зрелы в `iOS` / `Swift`
* Обычно он содержит очень простую логику и нам не нужно писать для этого Unit Tests
* `Router` - содержит логику навигации от одного представления (view controller) к другому
* В некоторых гайдах можно встретить такое название как  `FlowController`
* Очень сложно поддается написанию Unit Tests поскольку содержит много ссылок на классы `iOS` фреймворков. Поэтому нужно стараться делаеть его максимально простым.
* Обычно он ссылается только на `Presenter`, но из-за метода `func prepare (for segue: UIStoryboardSegue, sender: Any?)`  иногда ссылаемся на него и во `ViewController`


#### Clean Architecture Concepts
##### Application Logic

* `UseCase / Interactor` - содержит бизнес-логику для конкретного варианта использования в приложении
* На него ссылается `Presenter`.  `Presenter`  может ссылаться на несколько  `UseCases`  поскольку обычно используется несколько вариантов использования на одном экране
* Он манипулирует `Entities` и связывается с `Gateways`  для извлечения / сохранения объектов
* Протоколы  `Gateway` должны быть определены в слоях `Application Logic`  и реализованы в `Gateways & Framework Logic`
* Разделение, описанное выше, гарантирует, что `Application Logic`  зависит от абстракций, а не от фактических фреймворков / реализаций
* The separation described above ensures that the `Application Logic` depends on abstractions and not on actual frameworks / implementations
* Он должен покрываться Unit Tests
* `Entity` - простые `Swift` классы / структуры
* Модели объектов, используемых вашим приложением, такие как `Order`, `Product`, `Shopping Cart` и тд

##### Gateways & Framework Logic

* `Gateway` -  содержит фактическую реализацию протоколов, определенных в слое  `Application Logic`
* Мы можем реализовать например, протокол  `LocalPersistenceGateway` используя  `CoreData` or `Realm`
* Мы можем реализовать например, протокол  `ApiGateway` используя `URLSession` or `Alamofire`
* Мы можем реализовать например, протокол  `UserSettings` используя  `UserDefaults`
* Он должен покрываться Unit Tests
* `Persistence / API Entities` - содержат представления, специфичные для структуры
* Например, у нас может быть `CoreDataOrder` который является подклассом  `NSManagedObject`
*  `CoreDataOrder` не будет передан на слой  `Application Logic` ,  но слой  `Gateways & Framework Logic` "преобразует" его в сущность `Order` определенный в слое `Application Logic`
* `Framework specific APIs` - содержит реализации специфичных для `iOS` API, таких как датчики / bluetooth / camera

### Demo Application Details
Демо проект находится по ссылке:
```
https://github.com/FortechRomania/ios-mvp-clean-architecture
```
* Демо-приложения пытаются выявить довольно сложный набор функций, который оправдывает использование представленных выше понятий
* Были написаны следующие __Unit Tests__:
* `BooksPresenterTest` - демонстрирует, как вы можете протестировать логику `Presenter`
* `DeleteBookUseCaseTest` - демонстрирует, как вы можете протестировать прикладную / бизнес-логику, а также как протестировать асинхронный код, который использует обработчики завершения и `NotificationCenter`
* `CacheBooksGatewayTest` - демонстрирует, как вы можете протестировать политику кэширования
* `CoreDataBooksGatewayTest` - демонстрирует, как вы можете протестировать `CoreData`
* `ApiClientTest` - демонстрирует, как вы можете протестировать слой API / Networking вашего приложения, заменив стек `URLSession`
* __Code comments__ можно найти в нескольких классах, выделяя различные дизайнерские решения или ссылаясь на последующие ресурсы
* Структура проекта пытается имитировать концепцию __Screaming Architecture__, которая может быть найдена в разделе ссылок
* Высокоуровневая диаграмма UML:
![High level UML diagram](https://github.com/FortechRomania/ios-mvp-clean-architecture/blob/master/CleanArchitecture.png)

### Debatable Design Decisions

Предоставление большого количества мобильных приложений - это тонкий клиент поверх API и большинство из них содержат небольшую бизнес-логику (поскольку большинство бизнес-логик найдено в API), некоторые из концепций «чистой архитектуры» могут быть спорным в мобильном мире. Ниже вы можете найти:

* Создание представления для каждого уровня (API, CoreData) может показаться излишним. Если ваше приложение в значительной степени зависит от API, который находится под вашим контролем, тогда может возникнуть смысл моделировать как объект, так и представление API, используя тот же класс. Однако вы не должны допускать представление персистентности (например, подкласс `NSManagedObject`) в других слоях [`Parse`](https://techcrunch.com/2016/01/28/facebook-shutters-its-parse-developer-platform/)
* Если вы обнаружите, что в большинстве случаев ваши `UseCase / Interactor`  просто делегируют действия в `Gateways`, возможно, вам не нужно делать `UseCase / Interactor` , вы можете использовать `Gateways` непосредственно в `Presenter`
* Если вы хотите еще больше усилить разделение слоев, вы можете рассмотреть возможность перемещения всех слоев в свои собственные модули
* Некоторые могут подумать о том, что создание методов `display (xyz: String)` на `CellView` является чрезмерным и что передача объекта `CellViewModel`  во  `CellView` и представление конфигурации с моделью просмотра является более простым. Если вы хотите, чтобы `View` сохранялась как пассивный элемент, вы можете создать методы, но постарайтесь сохранить простую логику

Вышеуказанный список определенно не является полным, и если у вас есть спорные вопросы, пожалуйста, создайте issue, и мы можем обсудить ее и включить в список выше.

Имейте в виду, что вам не нужно принимать все проектные решения заранее и что вы можете реорганизовать их во время своей работы.

Обсудите все дизайнерские решения с членами вашей команды и убедитесь, что вы все согласны.

### Полезные ссылки

https://github.com/FortechRomania/ios-mvp-clean-architecture/blob/master/readme.md#useful-resources

### Информация

Оригинал: - https://github.com/FortechRomania/ios-mvp-clean-architecture
Если вы нашли ошибку, создайте issue или PR с исправлением.


### Перевод
Oleg Ryasnoy
Сообщество iOS Разработчиков в Telegram: https://t.me/devios

