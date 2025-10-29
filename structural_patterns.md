# Структурные паттерны проектирования

Отвечают за построение удобных в поддержке иерархий классов.

<details>
<summary>
  Adapter
</summary>

**Адаптер** - это структурный паттерн проектирования, который позволяет объектам с несовместимыми интерфейсами работать вместе.

<details>
<summary>
  Проблема
</summary>

Главный герой: Александр - опытный путешественник, который годами путешествовал по Европе на своём надежном автомобиле "Феникс", который реализует конкретный интерфейс управления. На этой машине он умеет: рулить `Steer()`, давать газ `Accelerate()` и тормозить `Brake()`

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Car {
  <<interface>>
  + Steer()
  + Accelerate()
  + Brake()
}

class Traveler {
  + Travel(distance: int)
}

class CarFenix {
  ...
  + Steer()
  + Accelerate()
  + Brake()
}

Car <|.. CarFenix
Traveler --> Car
```

Александр решает отправиться в путешествие по Сахаре. Автомобиль "Феникс" беспомощно застревает в песках. Александр понимает, что единственный способ продолжить путь - нанять верблюда у местных бедуинов.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Camel {
  ...
  + PullReins(direction: string)
  + KickHeels()
  + CommandStop()
}
```

Главный герой не умеет управлять верблюдом, он не знает его команд. Для поворота нужно тянуть за поводья в нужную сторону, для движения вперед - легонько ударить пятками по бокам верблюда, а для остановки - натянуть поводья.

</details>

<details>
<summary>
  Решение
</summary>

Бедуины предлагают Александру создать **адаптер**. Это объект-переводчик, который трансформирует интерфейс или данные одного объекта в такой вид, чтобы он стал понятен другому объекту

В нашем случае это специальное седло, оснащенное рулём и педалями, которое преобразует автомобильные команды в верблюжьи:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Adapter {
  - camel: Camel
  + Steer()
  + Accelerate()
  + Brake()
}

class Camel {
  ...
  + PullReins(direction: string)
  + KickHeels()
  + CommandStop()
}

Adapter o--> Camel
```

- `Steer("left")` → Поворот руля налево → `PullReins("left")` (механизм тянет левый повод)
- `Accelerate()` → Нажатие педали газа → `KickHeels()` (механизм легенько ударяет пятками по бокам верблюда)
- `Brake()` → Нажатие педали тормоза → `CommandStop()` (механизм натягивает поводья)

Итоговое управление нашего путешественника будет выглядить следующим образом:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Car {
  <<interface>>
  + Steer()
  + Accelerate()
  + Brake()
}

class Traveler {
  + Travel(distance: int)
}

class CarFenix {
  ...
  + Steer()
  + Accelerate()
  + Brake()
}

class Adapter {
  - camel: Camel
  + Steer()
  + Accelerate()
  + Brake()
}

class Camel {
  ...
  + PullReins(direction: string)
  + KickHeels()
  + CommandStop()
}

Car <|.. CarFenix
Traveler --> Car
Adapter --> Camel
Car <|.. Adapter
```

Заметим, что в данной реализации используется **ассоциация**. Адаптер содержит ссылку на служебный объект(Camel).

</details>

**Общая диаграмма паттерна через агрегацию:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Client

class IClient["Client Interface"]
IClient : + Method(data)

class Adapter {
  - adaptee: Service
  + Method(data)
}

class Service {
  ...
  + ServiceMethod(SpecialData)
}

Client --> IClient
IClient <|.. Adapter
Adapter --> Service
```

**Общая диаграмма паттерна через наследование:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Client

class ExistingClass["Existing Class"]
ExistingClass : ...
ExistingClass : + Method(data)

class Service {
  ...
  + ServiceMethod(special_data)
}

class Adapter {
  ...
  + Method(data)
}

Client --> ExistingClass
ExistingClass <-- Adapter
Service <|-- Adapter
```

В данном случае адаптеру не нужен вложенный объект, так как он может наследовать как часть существующего класса так и часть сервиса.

</details>

<details>
<summary>
  Bridge
</summary>

**Мост** — это структурный паттерн проектирования, который разделяет один или несколько классов на две отдельные иерархии — абстракцию и реализацию, позволяя изменять их независимо друг от друга.

<details>
<summary>
  Проблема
</summary>

У вас есть базовый класс `Принтер` с двумя видами принтеров: `Лазерный` и `Струйный`. Теперь вы хотите добавить поддержку разных типов печати: `цветной` и `чёрно-белой`.

Если пойти простым путём наследования, придётся создать четыре отдельных класса:
- `ЛазерныйЦветной`
- `ЛазерныйЧернобелый`
- `СтруйныйЦветной`
- `СтруйныйЧернобелый`

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Printer {
  <<abstract>>
  + Print()
}

class LaserColor {
  + Print()
}

class LaserMonochrome {
  + Print()
}

class InkjetColor {
  + Print()
}

class InkjetMonochrome {
  + Print()
}

Printer <|-- LaserColor
Printer <|-- LaserMonochrome
Printer <|-- InkjetColor
Printer <|-- InkjetMonochrome
```

- `Printer` = Принтер
- `LaserColor` = ЛазерныйЦветной
- `LaserMonochrome` = ЛазерныйЧернобелый
- `InkjetColor` = СтруйныйЦветной
- `InkjetMonochrome` = СтруйныйЧернобелый

На данный момент у нас `2 принтера` x `2 печати`, всего 4 подкласса. При добавлении новых видов принтеров и типов печати количество комбинаций будет расти. Так, если добавим еще один тип печати, то получится `3 х 2 = 6` подклассов. И это очень печально.

</details>

<details>
<summary>
  Решение
</summary>

Паттерн Мост предлагает отказаться от создания множества комбинаций через наследование. Вместо этого мы разделяем наши характеристики на две независимые части и связываем их через "мост".

Разделяем на две самостоятельные иерархии:

- Первая иерархия - типы принтеров (лазерный, струйный)
- Вторая иерархия - типы печати (цветная, чёрно-белая)

И заменяем наследование агрегацией:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
direction LR
class Printer
class PrintingType
Printer o--> PrintingType
```

Таким образом, мы разделили систему: тип печати стал отдельной иерархией с классами `Color` и `Monochrome`. Класс `Printer` теперь содержит ссылку на объект `PrintingType` и может поручать ему работу по обработке цветности. Эта связь между принтером и типом печати и есть тот самый "мост".

Главное преимущество: когда мы добавляем новые типы печати, классам принтеров не нужны изменения, и наоборот - новые принтеры легко работают с существующими типами печати.

Итак, **абстракция** — это образный слой управления чем-либо(`Printer`). Он не делает работу самостоятельно, а делегирует её слою реализации(`PrintingType`).

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Printer {
  <<abstract>>
  - printing_type: PrintingType
  + Printer(printing_type: PrintingType)
  + Print(document: String)*
  + SetPrintingType(printing_type: PrintingType)
}

class LaserPrinter {
  + Print(document: String)
}

class InkjetPrinter {
  + Print(document: String)
}

class PrintingType {
  <<interface>>
  + ApplyPrinting(document: String)*
}

class ColorPrinting {
  + ApplyPrinting(document: String)
}

class MonochromePrinting {
  + ApplyPrinting(document: String)
}

Printer <|-- LaserPrinter
Printer <|-- InkjetPrinter
Printer o--> PrintingType
PrintingType <|.. ColorPrinting
PrintingType <|.. MonochromePrinting
```

Реализация используя псевдокод:

**Абстракция:**

```pseudocode
class Printer {
  field printing_type: PrintingType  // Мост к реализации

  Constructor(printing_type: PrintingType) {
    this.printing_type = printing_type
  }

  abstract function Print(document: String)

  function SetPrintingType(printing_type: PrintingType) {
    this.printing_type = printing_type  // Меняем реализацию на лету!
  }
}

class LaserPrinter extends Printer {
  function Print(document: String) {
    Print("Лазерная печать: ")
    printing_type.ApplyPrinting(document)  // Делегируем реализацию
  }
}

class InkjetPrinter extends Printer {
  function Print(document: String) {
    Print("Струйная печать: ")
    printing_type.ApplyPrinting(document)  // Делегируем реализацию
  }
}
```

**Реализация:**

```pseudocode
interface PrintingType {
  function ApplyPrinting(document: String)
}

class ColorPrinting implements PrintingType {
  function ApplyPrinting(document: String) {
    Print("ЦВЕТНАЯ печать: " + document)
  }
}

class MonochromePrinting implements PrintingType {
  function ApplyPrinting(document: String) {
    Print("ЧЁРНО-БЕЛАЯ печать: " + document)
  }
}
```

**Клиент:**

```pseudocode
laser_сolor = new LaserPrinter(new ColorPrinting())
inkjet_mono = new InkjetPrinter(new MonochromePrinting())

// Работаем через абстракцию
laser_сolor.Print("Отчёт компании")
// Вывод: "Лазерная печать: ЦВЕТНАЯ печать: Отчёт компании"

inkjet_mono.Print("Черновик документа")
// Вывод: "Струйная печать: ЧЁРНО-БЕЛАЯ печать: Черновик документа"

// Меняем реализацию на лету!
laser_сolor.SetPrintingType(new MonochromePrinting())
laser_сolor.Print("Новый документ")
// Вывод: "Лазерная печать: ЧЁРНО-БЕЛАЯ печать: Новый документ"
```

</details>

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class Abstraction {
  - impl: Implementation
  + Feature1()
  + Feature2()
}

class RefinedAbstraction["Refined Abstraction"] {
  ...
  + FeatureN()
}

class Implementation {
  <<interface>>
  + Method1()
  + Method2()
  + Method3()
}

class ConcreteImpl["Concrete Implementations"]

Client --> Abstraction
Abstraction <|-- RefinedAbstraction
Abstraction o--> Implementation
Implementation <|.. ConcreteImpl
```

</details>

<details>
<summary>
  Composite
</summary>

**Компоновщик** — это структурный паттерн проектирования, который позволяет сгруппировать множество объектов в древовидную структуру, а затем работать с ней так, как будто это единичный объект.

<details>
<summary>
  Проблема
</summary>

Представим, что мы разрабатываем свою файловую систему. Она состоит из папок и файлов. Папка может содержать другие папки и файлы.

Мы решили, что хотим добавить функционал для подсчета размера, независимо папка это или файл.

**Пример файловой системы:**
```text
📁 Документы [5130 KB]
  📁 Работа [430 KB]
    📄 Резюме.pdf (250 KB)
    📄 Отчёт.docx (180 KB)
  📁 Фотографии [4700 KB]
    📄 Отпуск.jpg (1500 KB)
  📄 Проект.pptx (3200 KB)
```

Если решать задачу напрямую, то потребуется открыть все папки, перебрать все файлы внутри и посчитать их суммарный размер. Но это слишком сложно, так как структура папок и их содержимое могут быть неизвестны заранее.

Кроме того, невозможно предсказать количество уровней вложенности папок. Поэтому обойти такую структуру простым циклом не получится.

</details>


<details>
<summary>
  Решение
</summary>

Паттерн Компоновщик предлагает элегантное решение этой проблемы. Мы создаём общий интерфейс для всех элементов файловой системы — как для файлов, так и для папок. Этот интерфейс объявляет операции, которые поддерживаются как отдельными файлами, так и целыми группами папок.

Файл просто вернет свой размер, а папка запросит размер у вложенных файлов и папок и вернет их сумму. Вложенные папки, в свою очередь, также будут рекурсивно перебирать свои собственные вложенные элементы, запрашивая их размеры и суммируя результаты.

Нам, как клиенту, не придется теперь открывать все папки вручную, мы просто будем запрашивать размер у файловой системы, не думая о ее структуре.

**Общий интерфейс компонентов файловой системы:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class FileSystemComponent {
  <<interface>>
  + GetSize() int
}
```

Определяет метод для получения размера компонентов.

**Компоненты файловой системы:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class File {
  ...
  - size: int
  + GetSize() int
}
```

**File (Файл)** - конечный элемент системы, "лист" дерева. Содержит конкретные данные и просто возвращает свой размер при вызове `GetSize()`.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Folder {
  ...
  - children: FileSystemComponent[]
  + GetSize() int
  + Add(FileSystemComponent)
  + Remove(FileSystemComponent)
  + GetChild(int) FileSystemComponent
}
```

`Folder (Папка)` - компоновщик, может содержать другие компоненты. При вызове `GetSize()` рекурсивно обходит всех детей и суммирует их размеры. Управляет коллекцией через `Add()`, `Remove()`, `GetChild()`.

**Общая структура:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class FileSystemComponent {
  <<interface>>
  + GetSize() int
}

class File {
  ...
  - size: int
  + GetSize() int
}

class Folder {
  ...
  -children: FileSystemComponent[]
  + GetSize() int
  + Add(FileSystemComponent)
  + Remove(FileSystemComponent)
  + GetChild(int) FileSystemComponent
}

Client --> FileSystemComponent
FileSystemComponent <|.. File
FileSystemComponent <|.. Folder
Folder *--> FileSystemComponent
```

**Реализация в псевдокоде:**

**File:**

```pseudocode
class File {
  field size: int

  function GetSize() : int {
    return this.size  // Просто возвращаем свой размер
  }
}
```

**Folder:**

```pseudocode
class Folder {
  field children: Array FileSystemComponent

  function GetSize() : int {
    total = 0
    for each child in this.children {
      total = total + child.GetSize()  // Рекурсивный вызов!
    }
    return total
  }

  function Remove(component: FileSystemComponent) {
    this.children.Remove(component)
  }

  function GetChild(index: int) : FileSystemComponent {
    return this.children.Get(index)
  }
}
```

**Client:**

```pseudocode
function CreateFileSystem() : FileSystemComponent{
  // Создаём файлы
  resume = new File("Резюме.pdf", 250)
  report = new File("Отчёт.docx", 180)
  vacation_photo = new File("Отпуск.jpg", 1500)
  project = new File("Проект.pptx", 3200)

  // Создаём папки
  documents = new Folder("Документы")
  work_folder = new Folder("Работа")
  photos_folder = new Folder("Фотографии")

  // Добавляем файлы в папки
  work_folder.Add(resume)
  work_folder.Add(report)
  photos_folder.Add(vacation_photo)

  // Строим иерархию
  documents.Add(work_folder)
  documents.Add(photos_folder)
  documents.Add(project)  // Файл напрямую в корневую папку

  return documents
}

// Клиенту не нужно знать о вложенности!
function ClientCode() {
  file_system = CreateFileSystem()
  total_size = file_system.GetSize()
}
```

</details>

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client
class Component {
  <<interface>>
  + Execute()
}

class Leaf {
  ...
  + Execute()
}

class SubLeaf {
  ...
  + Execute()
}

class Composite {
  - children: Component[]
  + Add(cmpnt: Component)
  + Remove(cmpnt: Component)
  + GetChildren() Component[]
  + Execute()
}

Client --> Component
Component <|.. Leaf
Leaf <|-- SubLeaf
Component <|.. Composite
Component <--o Composite
```

</details>

<details>
<summary>
  Decorator
</summary>

**Декоратор** — это структурный паттерн проектирования, который позволяет динамически добавлять объектам новую функциональность, оборачивая их в полезные «обёртки».

<details>
<summary>
  Проблема
</summary>

Мы работаем над системой для кофейни, которая позволяет формировать заказы на кофе. Основой системы является класс `Coffee` с методом `GetDescription()`, который возвращает описание напитка, и методом `GetCost()`, который рассчитывает его стоимость.

Изначально в кофейне был только базовый кофе без молока и добавок.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Coffee {
  ...
  + GetDescription() String
  + GetCost() double
}
```

Сторонние программы (терминалы заказов) создавали объекты кофе и использовали их для формирования заказов.

Одного базового напитка клиентам мало. Некоторые хотели бы добавлять в кофе больше молока. Другие просили карамельный сироп. Третьи хотели взбитые сливки или шоколад.

Каждый тип добавки живёт в собственном подклассе. Сначала мы добавим каждый из этих типов добавок в программу, унаследовав их от базового класса `Coffee`.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
Coffee <|-- MilkCoffee
Coffee <|-- CaramelCoffee
Coffee <|-- WhippedCreamCoffee
Coffee <|-- ChocolateCoffee
```

Теперь клиент выбирал один тип напитка с одной добавкой, который и использовался в заказе.

Но многие клиенты хотели бы заказать, например, "латте с молоком, карамельным сиропом и взбитыми сливками"!

Попытка реализовать все возможные комбинации добавок через наследование приведёт к созданию громоздкой и неудобной иерархии классов, количество которых растёт в геометрической прогрессии.

</details>

<details>
<summary>
  Решение
</summary>

Паттерн Декоратор предлагает заменить наследование агрегацией (или композицией). Мы помещаем исходный объект в специальную обёртку, которая вызывает поведение у вложенного объекта и добавляет свою функциональность.

В нашем кофейном примере мы помещаем базовый напиток`Coffee` в специальную обёртку (например, "с молоком"). Мгновенно напиток приобретает новые вкусовые ноты, а его ценность закономерно возрастает на 20 рублей.

Поскольку и исходный объект, и обёртка реализуют одинаковый интерфейс, клиент может работать с ними одинаково.

При этом можно создавать цепочки из нескольких обёрток - например, Кофе → Молоко → Карамель → Сливки.

**Итоговая структура классов кофейни c применением паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class BaseCoffee {
  <<interface>>
  + GetDescription() String
  + GetCost() double
}

class Coffee {
  + GetDescription() String
  + GetCost() double
}

class CoffeeDecorator {
  <<abstract>>
  - wrapped: BaseCoffee
  + GetDescription() String
  + GetCost() double
}

class MilkDecorator {
  + GetDescription() String
  + GetCost() double
}

class CaramelDecorator {
  + GetDescription() String
  + GetCost() double
}

Client --> BaseCoffee
BaseCoffee <|.. Coffee
BaseCoffee <|.. CoffeeDecorator
BaseCoffee <--o CoffeeDecorator
CoffeeDecorator <|-- MilkDecorator
CoffeeDecorator <|-- CaramelDecorator
```

**Псевдокод**

**Класса `Coffee` вместе с интерфейсом:**

```pseudocode
// Базовый интерфейс для всех кофейных напитков
interface BaseCoffee {
  function GetDescription() : String
  function GetCost() : double
}

// Конкретный класс кофе
class Coffee implements BaseCoffee {
  function GetDescription() : String {
    return "Кофе"
  }

  function GetCost() : double {
    return 100.0
  }
}
```

**Обертки для класса `Coffee`:**

```pseudocode
// Абстрактный декоратор
abstract class CoffeeDecorator implements BaseCoffee {
  protected wrapped: BaseCoffee

  Constructor(coffee: BaseCoffee) {
    this.wrapped = coffee
  }

  function GetDescription() : String {
    return wrapped.GetDescription()
  }

  function GetCost() : double {
    return wrapped.GetCost()
  }
}

// Конкретные декораторы
class MilkDecorator extends CoffeeDecorator {
  function GetDescription() : String {
    return wrapped.GetDescription() + ", молоко"
  }

  function GetCost() : double {
    return wrapped.GetCost() + 20.0
  }
}

class CaramelDecorator extends CoffeeDecorator {
  function GetDescription() : String {
    return wrapped.GetDescription() + ", карамель"
  }

  function GetCost() : double {
    return wrapped.GetCost() + 30.0
  }
}

```

**Клиентский код:**

```pseudocode
function Main() {
    // Простой кофе
    simple_coffee = new Coffee()

    // Кофе с молоком
    coffee_with_milk = new MilkDecorator(new Coffee())

    // Кофе с молоком и карамелью
    fancy_coffee = new CaramelDecorator(
                    new MilkDecorator(
                      new Coffee()
                    )
                  )

  // Можем дополнить кофе на лету
  simple_coffee = new MilkDecorator(simple_coffee)
  simple_coffee = new CaramelDecorator(simple_coffee)
  // Кофе -> Кофе с Молоком -> Кофе с Молоком и Карамелью
}
```

</details>

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

сlass Client

class Component {
  <<interface>>
  + Execute()
}

class ConcComponent["Concrete Component"] {
  ...
  + Execute()
}

class BaseDecorator["Base Decorator"] {
  - wrappee: Component
  + BaseDecorator(comp: Component)
  + Execute()
}

class ConcDecorator["Concrete Decorator"] {
  ...
  + Execute()
  + Extra()
}

Client --> Component
Component <|.. ConcComponent
Component <|.. BaseDecorator
Component <--o BaseDecorator
BaseDecorator <|-- ConcDecorator
```

</details>

<details>
<summary>
  Facade
</summary>

**Фасад** — это структурный паттерн проектирования, который предоставляет простой интерфейс к сложной системе классов, библиотеке или фреймворку.

<details>
<summary>
  Проблема
</summary>

Представим, что у нас есть умный дом, состоящий из множества систем:

- Система освещения (LightingSystem)
- Система климат-контроля (ClimateControlSystem)
- Система мультимедиа (MultimediaSystem)
- Система безопасности (SecuritySystem)
- Система штор (CurtainSystem)

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class LightingSystem {
  + SetBrightness(int percentage)
  + SetColor(string color)
  + TurnOn()
  + TurnOff()
}

class ClimateControlSystem {
  + SetTemperature(double temperature)
  + SetMode(string mode)
}

class MultimediaSystem {
  + PlayMusic(string genre, int volume)
  + PlayMovie(string title)
  + SetVolume(int percentage)
  + Stop()
}

class SecuritySystem {
  + SetDoNotDisturb(bool enabled)
  + LockDoors()
  + ActivateAlarm()
  + DisableAlarm()
}

class CurtainSystem {
  + OpenAll()
  + CloseAll()
  + SetPosition(int percentage)
}
```

Мы захотели создать уютную атмосферу для вечернегго отдыха, это включает в себя:

1. Приглушить свет до 30%
2. Установить температуру 23°C
3. Включить джазовую музыку на 25% громкости
4. Активировать режим "Не беспокоить"
5. Закрыть шторы

Но для этого нам приходиться обращаться к каждой системе отдельно и настраивать все самим:

```pseudocode
lighting.SetBrightness(30)
climate.SetTemperature(23)
multimedia.PlayMusic("Jazz", 25)
security.SetDoNotDisturb(true)
curtains.CloseAll()
```

Это неудобно и требует знания всех деталей каждой системы!

</details>

<details>
<summary>
  Решение
</summary>

Паттерн Фасад предлагает создать единый упрощённый интерфейс для взаимодействия со сложной системой, состоящей из множества компонентов. Он действует как "интеллектуальный посредник", который скрывает всю внутреннюю сложность от пользователя.

Вместо того чтобы вручную управлять каждой подсистемой умного дома отдельно, мы создаём специальный класс-фасад `SmartHomeFacade`, который внутри выполнит всю последовательность действий за нас.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class LightingSystem {
  + SetBrightness(int percentage)
  + SetColor(string color)
  + TurnOn()
  + TurnOff()
}

class ClimateControlSystem {
  + SetTemperature(double temperature)
  + SetHumidity(int percentage)
  + SetMode(string mode)
}

class MultimediaSystem {
  + PlayMusic(string genre, int volume)
  + PlayMovie(string title)
  + SetVolume(int percentage)
  + Stop()
}

class SecuritySystem {
  + SetDoNotDisturb(bool enabled)
  + LockDoors()
  + ActivateAlarm()
  + DisableAlarm()
}

class CurtainSystem {
  + OpenAll()
  + CloseAll()
  + SetPosition(int percentage)
}

class SmartHomeFacade {
  - lighting: LightingSystem
  - climate: ClimateControlSystem
  - multimedia: MultimediaSystem
  - security: SecuritySystem
  - curtains: CurtainControlSystem
  + ActivateEveningMode()
  + ActivateMorningMode()
  + ActivateCinemaMode()
  + ActivateAwayMode()
}

SmartHomeFacade --> LightingSystem
SmartHomeFacade --> ClimateControlSystem
SmartHomeFacade --> MultimediaSystem
SmartHomeFacade --> SecuritySystem
SmartHomeFacade --> CurtainSystem
```

Теперь нам достаточно вызвать метод объекта `SmartHomeFacade` для создания уютной атмосферы:

```pseudocode
home_manager = new SmartHomeFacade()
home_manager.ActivateEveningMode()
```

</details>

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Facade {
  - links_to_subsystem_objects
  - optional_additional_facade
  + SubsytemOperation()
}

class AdditFacade["Additional Facade"] {
  ...
  + AnotherOperation()
}

class SubClass1["Subsystem class"]
class SubClass2["Subsystem class"]
class SubClass3["Subsystem class"]
class SubClass4["Subsystem class"]
class SubClass5["Subsystem class"]
class SubClass6["Subsystem class"]

Client --> Facade
Facade --> AdditFacade
Facade ..> SubClass1
Facade ..> SubClass2
Facade ..> SubClass3
Facade ..> SubClass4

AdditFacade ..> SubClass3
AdditFacade ..> SubClass4
AdditFacade ..> SubClass5
AdditFacade ..> SubClass6
```

</details>

<details>
<summary>
  Flyweight
</summary>

**Легковес** — это структурный паттерн проектирования, который позволяет вместить бóльшее количество объектов в отведённую оперативную память. Легковес экономит память, разделяя общее состояние объектов между собой, вместо хранения одинаковых данных в каждом объекте.

<details>
<summary>
  Проблема
</summary>

Мы пишем игру и в ней есть локация "Лес", там очень много деревьев, около 10 000. Каждое дерево имеет следующие данные:

- Тип (дуб, сосна, берёза) ~ 40 байт
- Текстуру ~ 2.7 МБ
- 3D-сетку ~ 263 КБ
- Здоровье ~ 4 байт
- Возраст ~ 4 байт
- Позицию (x, y, z) ~ 12 байт
- Состояние (здоровое/больное) ~ 1 байт

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
direction LR

class Game {
  - forest: List~Tree~
  - player: Player
  + Start()
  + Update()
  + RenderForest()
}

class Tree {
  - type: String
  - texture: Texture
  - mesh: Mesh
  - health: Int
  - age: Int
  - position: Vec3
  - is_sick: Bool
  + Render()
}

Game o--> Tree
```

Если посчитаем общий размер, то получим , что одно дерево занимает `3'100'555` байт
У нас `10'000` таких деревьев: `10000` x `3100555` = `28.9`ГБ 😱

Нам явно необходимо провести какие-то оптимизации.

</details>

<details>
<summary>
  Решение
</summary>

Паттерн Легковес предлагает разделить данные на две категории:

1. **Внутреннее состояние (разделяемое):**
   - Тип (дуб, сосна, берёза) ~ 40 байт
   - Текстура ~ 2.7 МБ
   - 3D-сетка ~ 263 КБ

2. **Внешнее состояние (уникальное):**
   - Здоровье ~ 4 байта
   - Возраст ~ 4 байта
   - Позиция (x, y, z) ~ 12 байт
   - Состояние (здоровое/больное) ~ 1 байт

**Внутреннее состояние** - это неизменяемые данные, которые определяют саму сущность объекта и одинаковы для всех экземпляров одного типа. Эти данные не зависят от контекста использования и могут безопасно разделяться между множеством объектов.

**Внешнее состояние** - это изменяемые данные, которые уникальны для каждого конкретного экземпляра объекта и зависят от контекста его использования. Эти данные определяют индивидуальные характеристики объекта в конкретный момент времени и не могут разделяться между разными объектами.

Ключевая идея Легковеса заключается в том, что внешнее состояние не хранится внутри класса, а передаётся в методы через параметры при каждом вызове. Такой подход кардинально меняет архитектуру — теперь нам требуется значительно меньше объектов, поскольку они различаются только внутренним состоянием, а количество возможных вариаций внутреннего состояния обычно невелико.

Поместим внутренее состояние в класс `TreeType` (он будет являться легковесом), получим:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Game {
  - forest: Tree []
  - player: Player
  + Start()
  + Update()
  + RenderForest()
}

class TreeType {
  - type: String
  - texture: Texture
  - mesh: Mesh
  + Render(position: Vec3, health: Int, age: Int, is_sick: Bool)
}

class Tree {
  - tree_type: TreeType
  - health: Int
  - age: Int
  - position: Vec3
  - is_sick: Bool
  + Render()
}

Game o--> Tree
Game o--> TreeType
Tree --> TreeType
```

В классе игры мы по-прежнему храним массив объектов `Tree`, однако теперь это значительно более лёгкие сущности. Каждый `Tree` выступает в роли контекста-хранилища, содержащего лишь уникальные экземплярные данные и компактную ссылку на объёмный, но общий для многих объектов `TreeType`.

Расчитаем потребление памяти:

`TreeType` размер: 40 байт + 2.7 МБ + 263 КБ ≈ 3.1 МБ
3 типа деревьев × 3.1 МБ = 9.3 МБ

`Tree` размер: 8 + 4 + 4 + 12 + 1 + 24 = 53 байта
10 000 деревьев × 53 байта = 530 000 байт ≈ 0.53 МБ

Всего потребление памяти составит: 9.83 МБ

**Фабрика Легковесов:**

Для удобстав работы с легковесами и контекстами можно создать отдельно фабрику `TreeTypeFactory`, которая будет отвечать за создание и повтороное использование объектов `TreeType`. Плюсы такого подхода в том, что если легковес уже создан, то нам не придется искать его, или создавать еще такой же, за нас это сделает фабрика.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Game {
  - forest: Tree []
  - player: Player
  + Start()
  + Update()
  + CreateForest()
  + RenderForest()
}

class Tree {
  - tree_type: TreeType
  - health: Int
  - age: Int
  - position: Vec3
  - is_sick: Bool
  + Tree()
  + Render()
}

class TreeType {
  - type: String
  - texture: Texture
  - mesh: Mesh
  + Render(position: Vec3, health: Int, age: Int, is_sick: Bool)
}

class TreeTypeFactory {
  - tree_types: Map~String, TreeType~
  + GetTreeType(type: String, texture: Texture, mesh: Mesh) TreeType
}

Game o--> Tree
Tree --> TreeType
Tree --> TreeTypeFactory : использует
TreeTypeFactory o--> TreeType : создаёт и хранит
```

Фабрика содержит словарь `tree_types`, где ключом является уникальный идентификатор, а значением — соответствующий объект `TreeType`.

**Псевдокод:**

**TreeFactory:**

```pseudocode
class TreeFactory {
  // Хранит созданные легковесы для повторного использования
  field tree_types: Map<String, TreeType>

  function GetTreeType(type: String, texture: Texture, mesh: Mesh) : TreeType {
    // Создаём уникальный ключ на основе внутреннего состояния
    key = type + ":" + texture.GetHash() + ":" + mesh.GetHash()

    // Если легковес не существует, то создаем его
    if (!tree_types.ContainsKey(key)) {
      tree_types[key] = new TreeType(type, texture, mesh)
    }

    return tree_types[key]
  }
}
```

**TreeType (легковес):**

```pseudocode
class TreeType {
  field type: String
  field texture: String
  field mesh: String

  Constructor(type: String) {
    this.type = type
    this.texture = type + "_texture.png"
    this.mesh = type + "_mesh.obj"
  }

  function Render(x: Int, y: Int, health: Int) {
    // Логика рендеринга
  }
}
```

**TreeType (легковес):**

```pseudocode
class Tree {
  field tree_type: TreeType
  field health: Int
  field x: Int
  field y: Int

  Constructor(type: String, x: Int, y: Int) {
    this.tree_type = TreeTypeFactory.GetTreeType(type)
    this.health = 100
    this.x = x
    this.y = y
  }

  function Render() {
    tree_type.Render(x, y, health)
  }

  function Grow() {
    health = health - 1  // Стареем
  }
}
```

**Game:**

```pseudocode
class Game {
  field trees: Tree[]
  field is_running: Bool = false

  function Start() {
    this.is_running = true
    this.CreateForest()
  }

  function CreateForest() {
    for i from 1 to 10000 {
      type = "Дуб"
      x = Random(0, 1000)
      y = Random(0, 1000)

      tree = new Tree(type, x, y)
      trees.Add(tree)
    }
  }

  function Update() {
    if not this.is_running {
        return
    }

    // Обновляем состояние деревьев
    for each tree in trees {
      if Random(0, 100) < 10 {  // 10% шанс
        tree.Grow()
      }
    }
  }

  function RenderForest() {
    if not this.is_running {
      return
    }

    for each tree in trees {
      tree.Render()
    }
  }

  function Stop() {
    this.is_running = false
  }
}
```

</details>


**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class FlyweightFactory {
  - cache: Flyweight[]
  + GetFlyweight(repeatingState): Flyweight
}

class Flyweight {
  - repeating_state
  + Operation(unique_state)
}

class Context {
  - unique_state
  - flyweight
  + Context(repeating_state, unique_state)
  + Operation()
}

Client *--> Context
Context --> FlyweightFactory
Context --> Flyweight
FlyweightFactory o--> Flyweight
```

</details>

Proxy
**Заместитель** — это структурный паттерн проектирования, который позволяет подставлять вместо реальных объектов специальные объекты-заменители. Эти объекты перехватывают вызовы к оригинальному объекту, позволяя сделать что-то до или после передачи вызова оригиналу.

Проблема
Решение

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Client

class ServiceInterface {
  <<interface>>
  + Operation()
}

class Proxy {
  - real_service: Service
  + Proxy(s: Service)
  + CheckAccess()
  + Operation()
}

class Service {
  ...
  + Operation()
}

Client --> ServiceInterface
ServiceInterface <|.. Proxy
ServiceInterface <|.. Service
Proxy o--> Service
```
