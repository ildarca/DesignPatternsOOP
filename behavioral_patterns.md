# Поведенческие паттерны проектирования

Решают задачи эффективного и безопасного взаимодействия между объектами программы.

<details>
<summary>
  Chain of Responsibility
</summary>

**Цепочка обязанностей** — это поведенческий паттерн проектирования, который позволяет передавать запросы последовательно по цепочке обработчиков. Каждый последующий обработчик решает, может ли он обработать запрос сам и стоит ли передавать запрос дальше по цепи.

<details>
<summary>
  Проблема
</summary>

Мы создаем банковскую систему. И в нее поступают заявки от клиентов на различные операции. Каждая заявка должна проходить несколько этапов проверки:

- Проверка формата данных
- Проверка лимитов
- Проверка безопасности
- Проверка кредитной истории
- Финальное одобрение

Заявки поступают в класс `LoanApplication` и обрабатываются в его методе `Process()`.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class LoanApplication {
  + Process(client_data, amount, duration) String
  - ValidateFormat(client_data, amount, duration) Boolean
  - CheckLimits(amount, duration) Boolean
  - SecurityCheck(client_data) Boolean
  - CheckCreditHistory(client_data) Boolean
}
```

```pseudocode
class LoanApplication {
  function Process(application: Application) : String {
    // Проверка формата данных
    if (!this.ValidateFormat(application)) {
      return "Invalid format"
    }

    // Проверка финансовых лимитов
    if (!this.CheckLimits(application)) {
      return "Limit exceeded"
    }

    // Проверка безопасности
    if (!this.SecurityCheck(application)) {
      return "Security issues"
    }

    // Проверка кредитной истории
    if (!this.CheckCreditHistory(application)) {
      return "Poor credit history"
    }

    // Финальное одобрение
    return "Approved"
  }

  function ValidateFormat(application: Application) : Boolean {
    // Проверяет корректность заполнения полей
    return application.name != null
      and application.amount > 0
      and application.duration > 0
  }

  function CheckLimits(application: Application) : Boolean {
    // Проверяет не превышены ли лимиты
    return application.amount <= 1000000
      and application.duration <= 60
  }

  function SecurityCheck(application: Application) : Boolean {
    // Проверяет безопасность операции
    return !application.client.isBlacklisted()
      and !application.hasFraudIndicators()
  }

  function CheckCreditHistory(application: Application) : Boolean {
    // Проверяет кредитную историю клиента
    return application.client.creditScore >= 650
      and application.client.hasNoDelinquencies()
  }
}
```

Если реализовать всё в одном классе, получится огромный метод с множеством условий. При добавлении новых проверок придется модифицировать существующий код, нарушая принцип открытости/закрытости.

Такой подход сложно поддерживать и расширять.

</details>

<details>
<summary>
  Решение
</summary>

Паттерн Цепочка обязанностей предлагает превратить отдельные поведения в объекты-обработчики. Каждый обработчик содержит метод для выполнения одной конкретной проверки. Данные для таких проверок будут передаваться в метод через параметры. Вместо монолитного метода `Process()` мы создаем отдельные обработчики с методом `Handle()`.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class FormatHandler {
  + Handle(application: Application) Result
}

class LimitHandler {
  + Handle(application: Application) Result
}

class SecurityHandler {
  + Handle(application: Application) Result
}
```

Обработчики объединяются в цепочку, где каждый имеет ссылку на следующего в последовательности. Это позволяет обработчику выполнить собственную логику над запросом
и при необходимости передать запрос следующему звену цепи. Длина цепочки легко масштабируется. Обработчик может прервать передачу запроса дальше, что дает гибкость в некоторых сценариях.

Создадим абстрактный класс LoanHandler, который будет содержать общую логику для всех обработчиков, включая методы и поле-ссылку `next` на следующий обработчик:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class LoanHandler {
  <<abstract>>
  - next: LoanHandler
  + SetNext(handler: LoanHandler) LoanHandler
  + Handle(application: Application) Result
  # HandleNext(application: Application) Result
}

class FormatHandler {
  + Handle(application: Application) Result
}

class LimitHandler {
  + Handle(application: Application) Result
}

class SecurityHandler {
  + Handle(application: Application) Result
}

LoanHandler <|-- FormatHandler
LoanHandler <|-- LimitHandler
LoanHandler <|-- SecurityHandler
```

И для удобной работы клиентского кода с объекетами-обработчиками создадим общий интерфейс:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class IHandler["Handler"] {
  <<interface>>
  + SetNext(handler: Handler)
  + Handle(application: Application) Result
}

class LoanHandler {
  <<abstract>>
  - next: Handler
  + SetNext(handler: Handler)
  + Handle(application: Application) Result
  + HandleNext(application: Application) Result
}

class FormatHandler {
  + Handle(application: Application) Result
}

class LimitHandler {
  + Handle(application: Application) Result
}

class SecurityHandler {
  + Handle(application: Application) Result
}

IHandler <|.. LoanHandler
IHandler <--o LoanHandler
LoanHandler <|-- FormatHandler
LoanHandler <|-- LimitHandler
LoanHandler <|-- SecurityHandler
```

**Псевдокод:**

**Handler (интерфейс):**

```pseudocode
interface Handler {
  function SetNext(handler: Handler)
  function Handle(application: Application) : Result
}
```

**LoanHandler (абстрактный класс):**

```pseudocode
abstract class LoanHandler implements Handler {
  field next: Handler

  function SetNext(handler: Handler) {
    this.next = handler
  }

  abstract function Handle(application: Application) : Result

  function HandleNext(application: Application) : Result {
    if (this.next != null) {
      return this.next.Handle(application)
    }
    return Result("Approved")
  }
}
```

**FormatHandler:**

```pseudocode
class FormatHandler extends LoanHandler {
  function Handle(application: Application) : Result {
    if (!application.hasRequiredFields()) {
      return Result("Error: Invalid format")
    }
    return this.HandleNext(application)
  }
}
```

**LimitHandler:**

```pseudocode
class LimitHandler extends LoanHandler {
  function Handle(application: Application) : Result {
    if (application.amount > 1000000) {
      return Result("Error: Amount exceeds limit")
    }
    return this.HandleNext(application)
  }
}
```

**SecurityHandler:**

```pseudocode
class SecurityHandler extends LoanHandler {
  function Handle(application: Application) : Result {
    if (application.client.isBlacklisted()) {
      return Result("Error: Client blacklisted")
    }
    return this.HandleNext(application)
  }
}
```

**Использование:**

```pseudocode
format_handler = new FormatHandler()
limit_handler = new LimitHandler()
security_handler = new SecurityHandler()

format_handler.SetNext(limitHandler)
limit_handler.SetNext(securityHandler)

result = format_handler.Handle(application)
// Запускаем обработку через цепочку: Формат → Лимиты → Безопасность
```

</details>

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class Handler {
  <<interface>>
  + SetNext(h: Handler)
  + Handle(request)
}

class BaseHandler {
  <<abstract>>
  - next: Handler
  + SetNext(h: Handler)
  + Handle(request)
}

class ConcreteHandlerA {
  + Handle(request)
}

class ConcreteHandlerB {
  + Handle(request)
}

Client --> Handler
Handler <|.. BaseHandler
Handler <--o BaseHandler
BaseHandler <|-- ConcreteHandlerA
BaseHandler <|-- ConcreteHandlerB
```

</details>

Command
**Команда** — это поведенческий паттерн проектирования, который превращает запросы в объекты, позволяя передавать их как аргументы при вызове методов, ставить запросы в очередь, логировать их, а также поддерживать отмену операций.

Проблема

Представьте, что вы разрабатываете текстовый редактор, подобный Word или Google Docs. Вы уже реализовали класс `Document`, который содержит бизнес-логику работы с документом.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
class Document {
  - content: String
  - file_path: String
  + Save()
  + Open(path: String)
  + Undo()
}
```

Вам нужно реализовать различные элементы управления(кнопки, ползунки и т.д.) для выполнения операций с документом:

- Сохранить документ
- Открыть новый документ
- Отменить изменения

В нашей программе существует разные способы вызова этих операций:

- Кнопки на панели инструментов
- Пункты в меню (Файл → Сохранить)
- Горячие клавиши (Ctrl+S, Ctrl+O, Ctlr+Z)

Можно создать отдельные классы для каждого элемента управления:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Button {
  <<abstract>>
  + OnClick()
}

class MenuItem {
  <<abstract>>
  + OnSelect()
}

class Shortcut {
  <<abstract>>
  + OnKeyPress()
}

class SaveButton {
  + OnClick()
}

class SaveMenuItem {
  + OnSelect()
}

class SaveShortcut {
  + OnKeyPress()
}

class OpenButton {
  + OnClick()
}

class OpenMenuItem {
  + OnSelect()
}

class OpenShortcut {
  + OnKeyPress()
}

class UndoButton {
  + OnClick()
}

class UndoMenuItem {
  + OnSelect()
}

class UndoShortcut {
  + OnKeyPress()
}

class Document {
  - content: String
  - file_path: String
  + Save()
  + Open(path: String)
  + Undo()
}

Button <|-- SaveButton
Button <|-- OpenButton
Button <|-- UndoButton

MenuItem <|-- SaveMenuItem
MenuItem <|-- OpenMenuItem
MenuItem <|-- UndoMenuItem

Shortcut <|-- SaveShortcut
Shortcut <|-- OpenShortcut
Shortcut <|-- UndoShortcut

SaveButton --> Document
SaveMenuItem --> Document
SaveShortcut --> Document
OpenButton --> Document
OpenMenuItem --> Document
OpenShortcut --> Document
UndoButton --> Document
UndoMenuItem --> Document
UndoShortcut --> Document
```

```pseudocode
// Кнопка "Сохранить" на панели инструментов
class SaveButton extends Button {
  field document: Document

  function OnClick() {
    document.Save()
  }
}

// Пункт меню "Сохранить"
class SaveMenuItem extends MenuItem {
  field document: Document

  function OnSelect() {
    document.Save()
  }
}

// Горячие клавиши Ctrl+S
class SaveShortcut extends Shortcut {
  field document: Document

  function OnKeyPress() {
    document.Save()
  }
}

// И так для операций Открыть и Отменить...
```

Это очень плохой подход:

1. Мы создаем огромное количество подклассов
2. Один и тот же код повторяется в разных местах программы.
3. При внесении изменений, например, сохранения файлов придется переписывать код сразу в нескольких местах.

Решение

Паттерн Команда предлагает вынести операции в отдельные классы-команды с единым интерфейсом, содержащим всего один метод для выполнения операции. Это позволяет разным элементам интерфейса работать с любыми командами, не зная их конкретной реализации.

Благодаря такому подходу, одни и те же кнопки, пункты меню или горячие клавиши могут выполнять различные команды, просто подменяя объект команды. Сами команды содержат всю необходимую информацию для выполнения операции — параметры, данные получателя и бизнес-логику. Элементы интерфейса лишь инициируют выполнение команды, не заботясь о том, что именно происходит внутри и какие данные требуются для операции.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Command {
  <<interface>>
  + Execute()
}

class SaveCommand {
  - document: Document
  + Execute()
}

class OpenCommand {
  - document: Document
  + Execute()
}

class UndoCommand {
  - document: Document
  + Execute()
}

class Button {
  - command: Command
  + OnClick()
  + SetCommand(command: Command)
}

class MenuItem {
  - command: Command
  + OnSelect()
  + SetCommand(command: Command)
}

class Shortcut {
  - command: Command
  + OnKeyPress()
  + SetCommand(command: Command)
}

class Document {
  - content: String
  - file_path: String
  + Save()
  + Open(path: String)
  + Undo()
}

Command <|.. SaveCommand
Command <|.. OpenCommand
Command <|.. UndoCommand
Button --> Command
MenuItem --> Command
Shortcut --> Command
SaveCommand --> Document
OpenCommand --> Document
UndoCommand --> Document
```

**Псевдокод:**

**Command (интерфейс):**

```pseudocode
interface Command {
  function Execute()
}
```

**SaveCommand:**

```pseudocode
class SaveCommand implements Command {
  field document: Document

  constructor(document: Document) {
    this.document = document
  }

  function Execute() {
    document.Save()
  }
}
```

**OpenCommand:**

```pseudocode
class OpenCommand implements Command {
  field document: Document

  constructor(document: Document) {
    this.document = document
  }

  function Execute() {
    document.Open()
  }
}
```

**UndoCommand:**

```pseudocode
class UndoCommand implements Command {
  field document: Document

  constructor(document: Document) {
    this.document = document
  }

  function Execute() {
    document.Undo()
  }
}
```

**Button:**

```pseudocode
class Button {
  field command: Command

  function SetCommand(cmd: Command) {
    this.command = cmd
  }

  function OnClick() {
    if (command != null) {
      command.Execute()
    }
  }
}
```

**MenuItem:**

```pseudocode
class MenuItem {
  field command: Command

  function SetCommand(cmd: Command) {
    this.command = cmd
  }

  function OnSelect() {
    if (command != null) {
      command.Execute()
    }
  }
}
```

**Shortcut:**

```pseudocode
class Shortcut {
  field command: Command

  function SetCommand(cmd: Command) {
    this.command = cmd
  }

  function OnKeyPress() {
    if (command != null) {
      command.Execute()
    }
  }
}
```

**Использование:**

```pseudocode
document = new Document()

save_command = new SaveCommand(document)
open_command = new OpenCommand(document)
undo_command = new UndoCommand(document)

save_button = new Button()
save_menu_item = new MenuItem()
save_shortcut = new Shortcut()

save_button.SetCommand(save_command)
save_menu_item.SetCommand(save_command)
save_shortcut.SetCommand(save_command)

// Теперь все три элемента выполняют сохранение
save_button.OnClick()      // Сохраняет документ
save_menu_item.OnSelect()  // Сохраняет документ
save_shortcut.OnKeyPress() // Сохраняет документ
```

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class Receiver {
  + Operation(a, b, c)
}

class Invoker {
  - command: Command
  + SetCommand(command: Command)
  + ExecuteCommand()
}

class Command {
  <<interface>>
  + Execute()
}

class Command1 {
  - receiver: Receiver
  - params: Params
  + Command1(receiver: Receiver, params: Params)
  + Execute()
}

class Command2 {
  + Execute()
}

Client --> Invoker
Client --> Receiver
Client ..> Command1
Receiver <-- Command1
Invoker --> Command
Command1 ..|> Command
Command2 ..|> Command
```

Iterator
**Итератор** — это поведенческий паттерн проектирования, который даёт возможность последовательно обходить элементы составных объектов, не раскрывая их внутреннего представления.

Проблема

В языках программирования встречается множество различных контейнеров для хранения элементов: массив, связный список, хэш-таблица, дерево и т. д. Все они имеют разную внутренюю структуру и для каждого из них приходится использовать свой способ перебора по элементам.

Представьте библиотечную систему, которая хранит книги в разных структурах данных:

- Массив новых поступлений
- Связный список редких книг
- Хэш-таблица книг по авторам

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class BookArray {
  - books: Book[]
  + GetBook(index: int) Book
  + GetLength() int
}

class BookLinkedList {
  - head: Node
  + GetHead() Node
}

class Node {
  - data: Book
  - next: Node
  + GetData() Book
  + GetNext() Node
}

class BookHashTable {
  - buckets: Bucket[]
  + GetBuckets() Bucket[]
}

class Bucket {
  - books: Book[]
  + GetBooks() Book[]
}

class Book {
  - title: String
  + GetTitle() String
}

class Client

Client --> BookArray
Client --> BookLinkedList
Client --> BookHashTable
BookLinkedList --> Node
Node --> Book
BookArray --> Book
BookHashTable --> Bucket
Bucket --> Book
```

Код клиента:

```pseudocode
// Для массива
for (i = 0; i < array.GetLength(); i++) {
  book = array.GetBook(i)
  print(book.GetTitle())
}

// Для связного списка
current = linked_list.GetHead()
while (current != null) {
  book = current.data
  print(book.GetTitle())
  current = current.next
}

// Для хэш-таблицы
buckets = table.GetBuckets()
for each bucket in buckets {
  books = bucket.GetBooks()
  for each book in books {
    print(book.getTitle())
  }
}
```

Проблема данного подхода в том, что клиентский код должен знать внутреннее устройство каждого контейнера, клиенту видны все внутренние данные контейнеров. И при сменее структуры данных нужно переписывать весь клиентский код.

Решение

Паттерн Итератор предлагает вынести логику обхода коллекций в отдельные классы-итераторы. Вместо того чтобы сама коллекция управляла процессом обхода, для этого создаются специальные объекты-итераторы. Каждый такой итератор самостоятельно отслеживает текущее состояние обхода: какая позиция в коллекции, какие элементы остались, как двигаться дальше. Если нам нужен будет какой-нибудь новый способ обхода, то мы просто добавим новый класс-итератор.

Для начала создадим общий интерфейс Итераторов:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Iterator {
  <<interface>>
  + HasNext() Boolean
  + Next() Book
}
```

Это позволит клиентскому коду работать с любыми итераторами одинаково и обеспечит заменяемость разных реализаций итераторов.

То же самое проделаем и для самих контейнеров. Интерфейс `IterableCollection` будет гарантировать, что класс коллекции умеет создавать итератор для обхода своих элементов:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class IterableCollection {
  <<interface>>
  + CreateIterator() Iterator
}
```

Для библиотечной системы контейнеры с итераторами примут уже такой вид:

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class Iterator {
  <<interface>>
  + HasNext() Boolean
  + Next() Book
}

class BookArrayIterator {
  - books: Book[]
  - position: int
  + HasNext() Boolean
  + Next() Book
}

class LinkedListIterator {
  - list: LinkedList
  - current: Node
  + HasNext() Boolean
  + Next() Book
}

class HashTableIterator {
  - table: HashTable
  - bucketIndex: int
  - elementIndex: int
  + HasNext() Boolean
  + Next() Book
}

class IterableCollection {
  <<interface>>
  + CreateIterator() Iterator
}

class BookArray {
  - books: Book[]
  + CreateIterator() Iterator
}

class BookLinkedList {
  - list: LinkedList
  + CreateIterator() Iterator
}

class BookHashTable {
  - table: HashTable
  + CreateIterator() Iterator
}

IterableCollection <|.. BookArray
IterableCollection <|.. BookLinkedList
IterableCollection <|.. BookHashTable

BookArray --> BookArrayIterator
BookLinkedList --> LinkedListIterator
BookHashTable --> HashTableIterator

Client --> IterableCollection

BookArrayIterator ..|> Iterator
LinkedListIterator ..|> Iterator
HashTableIterator ..|> Iterator
```

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Client

class Iterator {
  <<interface>>
  + GetNext()
  + HasMore() bool
}

class ConcreteIterator {
  - collection: ConcreteCollection
  - iteration_state
  + ConcreteIterator(c: ConcreteCollection)
  + GetNext()
  + HasMore() bool
}

class IterableCollection {
  <<interface>>
  + CreateIterator() Iterator
}

class ConcreteCollection {
  ...
  + CreateIterator() Iterator
}

Client --> Iterator
Client --> IterableCollection
IterableCollection ..> Iterator
Iterator <|.. ConcreteIterator
IterableCollection <|.. ConcreteCollection
ConcreteIterator <--> ConcreteCollection
```

Mediator
**Посредник** — это поведенческий паттерн проектирования, который позволяет уменьшить связанность множества классов между собой, благодаря перемещению этих связей в один класс-посредник.

Проблема

Представьте команду разработки программного продукта, где участвуют заказчик, программисты, тестировщики и дизайнеры. В традиционном подходе все эти участники общаются напрямую друг с другом. Заказчик может одновременно давать разные указания программисту и дизайнеру, программист напрямую сообщает тестировщику о готовности задачи, тестировщик шлет баг-репорты прямо программисту, а дизайнер согласует макеты со всеми сразу.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram
direction LR

class Customer {
  + ChangeRequirements()
  + RequestFeature()
}

class Programmer {
  + CompleteTask()
  + RequestClarification()
}

class Tester {
  + ReportBug()
  + VerifyFix()
}

class Designer {
  + DeliverDesign()
  + UpdateLayout()
}

Customer --> Programmer
Customer --> Tester
Customer --> Designer
Programmer --> Tester
Programmer --> Designer
Programmer --> Customer
Tester --> Programmer
Tester --> Customer
Designer --> Programmer
Designer --> Customer
```

Такой прямой способ коммуникации быстро приводит к хаосу. Заказчик может непоследовательно формулировать требования, давая разные формулировки разным участникам. Программист, получая запросы из нескольких источников, не понимает, какие из них приоритетны. Тестировщик и дизайнер могут получить противоречивые инструкции. Возникает ситуация, когда левая рука не знает, что делает правая - программист реализует одну функциональность, в то время как дизайнер уже переработал макеты, а заказчик и вовсе передумал.

```pseudocode
class Customer {
  field programmer: Programmer
  field tester: Tester
  field designer: Designer

  function ChangeRequirements(new_requirements: String) {
    // Заказчик сам решает, кого уведомлять
    programmer.UpdateRequirements(new_requirements)
    designer.UpdateRequirements(new_requirements)
    tester.UpdateTestCases(new_requirements)
    // Возможны разные формулировки для разных участников
  }

  function RequestFeature(feature: String) {
    // Запросы отправляются напрямую
    programmer.ImplementFeature(feature)
    designer.CreateDesign(feature)
  }
}
```

Решение

Паттерн Посредник предлагает радикально изменить эту схему, введя центральную фигуру - менеджера проектов. В этом случае все участники перестают общаться напрямую. Вместо этого каждый из них регистрируется у менеджера и далее все коммуникации происходят исключительно через него.

Для менеджера создадим интерфейс, который он будет реализовывать, чтобы если что можно было заменять менеджеров. И для удобства работы сделаем общий интерфейс и для всех участников.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class TeamMember {
  <<interface>>
  + SetMediator(mediator: Mediator)
  + ReceiveMessage(message: String)
  + GetRole() String
}

class Mediator {
  <<interface>>
  + Notify(sender: TeamMember, event: String, data: Any)
}
```

Когда заказчик меняет требования, он не бежит к каждому участнику команды, а просто уведомляет менеджера. Менеджер, получив эту информацию, сам решает, кого и в какой последовательности уведомить. Он может сначала сообщить дизайнеру о необходимости пересмотреть макеты, затем программисту - о изменениях в техническом задании, и наконец тестировщику - о необходимости актуализировать тест-кейсы.

Аналогично, когда программист завершает задачу, он не пишет всем подряд, а просто сообщает менеджеру о готовности. Менеджер уже сам решает, что тестировщик должен начать тестирование, а заказчику можно отправить уведомление о завершении этапа. Если тестировщик находит ошибку, он сообщает о ней менеджеру, и тот уже назначает программисту задачу на исправление, параллельно информируя заказчика о возникшей задержке.

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class Mediator {
  <<interface>>
  + Notify(sender: TeamMember, event: String, data: Any)
}

class ProjectManager {
  - customer: Customer
  - programmer: Programmer
  - tester: Tester
  - designer: Designer
  + ProjectManager()
  + Notify(sender: TeamMember, event: String, data: Any)
  + RegisterMember(member: TeamMember)
}

class TeamMember {
  <<interface>>
  + SetMediator(mediator: Mediator)
  + ReceiveMessage(message: String)
  + GetRole() String
}

class Customer {
  + ChangeRequirements(newRequirements: String)
  + RequestFeature(feature: String)
  + SetMediator(mediator: Mediator)
  + ReceiveMessage(message: String)
  + GetRole() String
}

class Programmer {
  + CompleteTask(task: String)
  + RequestClarification(question: String)
  + SetMediator(mediator: Mediator)
  + ReceiveMessage(message: String)
  + GetRole() String
}

class Tester {
  + ReportBug(bugDescription: String)
  + VerifyFix(bugId: String)
  + SetMediator(mediator: Mediator)
  + ReceiveMessage(message: String)
  + GetRole() String
}

class Designer {
  + DeliverDesign(design: String)
  + UpdateLayout(layout: String)
  + SetMediator(mediator: Mediator)
  + ReceiveMessage(message: String)
  + GetRole() String
}

Mediator <|.. ProjectManager
ProjectManager o--> Customer
ProjectManager o--> Programmer
ProjectManager o--> Tester
ProjectManager o--> Designer
Customer ..|> TeamMember
Programmer ..|> TeamMember
Tester ..|> TeamMember
Designer ..|> TeamMember
```

Участники команды больше не связаны друг с другом напрямую - они знают только менеджера и общаются только через него. Это значительно упрощает изменение состава команды или процессов взаимодействия, поскольку не требует перестройки множественных связей между участниками.

**Псевдокод:**

**ProjectManager:**

```pseudocode
class ProjectManager implements Mediator {
  field customer: Customer
  field programmer: Programmer
  field tester: Tester
  field designer: Designer

  function ProjectManager() {
    // Создаем участников команды в конструкторе
    this.customer = new Customer()
    this.programmer = new Programmer()
    this.tester = new Tester()
    this.designer = new Designer()

    // Регистрируем себя как посредника
    this.customer.SetMediator(this)
    this.programmer.SetMediator(this)
    this.tester.SetMediator(this)
    this.designer.SetMediator(this)
  }

  function RegisterMember(member: TeamMember) {
    // Метод может остаться для обратной совместимости,
    // но в этой реализации не используется
  }

  function Notify(sender: TeamMember, event: String, data: Any) {
    if (sender.GetRole() == "Programmer" and event == "task_completed") {
      // Программист завершил задачу
      this.tester.ReceiveMessage("Задача '" + data + "' готова к тестированию")
      this.customer.ReceiveMessage("Этап разработки '" + data + "' завершен")
    }
    else if (sender.GetRole() == "Tester" and event == "bug_reported") {
      // Тестировщик нашел баг
      this.programmer.ReceiveMessage("Обнаружен баг: " + data)
      this.customer.ReceiveMessage("Выявлена проблема, возможна задержка: " + data)
    }
    else if (sender.GetRole() == "Customer" and event == "requirements_changed") {
      // Заказчик изменил требования
      this.programmer.ReceiveMessage("Требования обновлены: " + data)
      this.designer.ReceiveMessage("Учтите новые требования в дизайне: " + data)
      this.tester.ReceiveMessage("Актуализируйте тест-кейсы: " + data)
    }
    else if (sender.GetRole() == "Designer" and event == "design_ready") {
      // Дизайнер завершил макет
      this.programmer.ReceiveMessage("Дизайн готов для реализации: " + data)
      this.tester.ReceiveMessage("Дизайн утвержден, готовьте тесты: " + data)
    }
  }
}
```

**Programmer:**

```pseudocode
class Programmer implements TeamMember {
  field mediator: Mediator

  function SetMediator(mediator: Mediator) {
    this.mediator = mediator
  }

  function GetRole() : String {
    return "Programmer"
  }

  function CompleteTask(task: String) {
    print("Программист завершил задачу: " + task)
    if (mediator != null) {
      mediator.Notify(this, "task_completed", task)
    }
  }

  function RequestClarification(question: String) {
    print("Программист запрашивает уточнение: " + question)
    if (mediator != null) {
      mediator.Notify(this, "clarification_needed", question)
    }
  }

  function ReceiveMessage(message: String) {
    print("Программист получил сообщение: " + message)
  }
}
```

**Использование:**

```pseudocode
// Теперь все просто - создаем менеджера
manager = new ProjectManager()

// Участники уже созданы и зарегистрированы
// Получаем доступ к участникам через менеджера (если нужно)
programmer = manager.programmer
customer = manager.customer

// Работаем через посредника
programmer.CompleteTask("Форма регистрации")
// Автоматически уведомляются тестировщик и заказчик

customer.ChangeRequirements("Добавить темную тему")
// Автоматически уведомляются программист, дизайнер и тестировщик
```

**Общая диаграмма паттерна:**

```mermaid
%%{init: {'theme': 'dark', 'class': {'hideEmptyMembersBox': true}}}%%
classDiagram

class ComponentA {
  - m: Mediator
  + OperationA()
}

class ComponentB {
  - m: Mediator
  + OperationB()
}

class ComponentC {
  - m: Mediator
  + OperationC()
}

class ComponentD {
  - m: Mediator
  + OperationD()
}

class Mediator {
  <<interface>>
  + Notify(sender)
}

class ConcreteMediator {
  - componentA
  - componentB
  - componentC
  - componentD
  + Notify(sender)
  + ReactOnA()
  + ReactOnB()
  + ReactOnC()
  + ReactOnD()
}

Mediator <-- ComponentA
Mediator <-- ComponentB
Mediator <-- ComponentC
Mediator <-- ComponentD
ComponentA <--o ConcreteMediator
ComponentB <--o ConcreteMediator
Mediator <|.. ConcreteMediator
ComponentC <--o ConcreteMediator
ComponentD <--o ConcreteMediator
```

Memento
**Снимок** — это поведенческий паттерн проектирования, который позволяет сохранять и восстанавливать прошлые состояния объектов, не раскрывая подробностей их реализации.

Проблема
Решение

Observer
**Наблюдатель** — это поведенческий паттерн проектирования, который создаёт механизм подписки, позволяющий одним объектам следить и реагировать на события, происходящие в других объектах.

Проблема
Решение

State
**Состояние** — это поведенческий паттерн проектирования, который позволяет объектам менять поведение в зависимости от своего состояния. Извне создаётся впечатление, что изменился класс объекта.

Проблема
Решение

Strategy
**Стратегия** — это поведенческий паттерн проектирования, который определяет семейство схожих алгоритмов и помещает каждый из них в собственный класс, после чего алгоритмы можно взаимозаменять прямо во время исполнения программы

Проблема
Решение

Template Method
**Шаблонный метод** — это поведенческий паттерн проектирования, который определяет скелет алгоритма, перекладывая ответственность за некоторые его шаги на подклассы. Паттерн позволяет подклассам переопределять шаги алгоритма, не меняя его общей структуры.

Проблема
Решение

Visitor
**Посетитель** — это поведенческий паттерн проектирования, который позволяет добавлять в программу новые операции, не изменяя классы объектов, над которыми эти операции могут выполняться.

Проблема
Решение
