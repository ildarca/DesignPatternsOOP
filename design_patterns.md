
## Принципы проектирования архитектуры

### Повторное использование кода

###

### SOLID

<details>
<summary>
  Single Responsibility Principle
</summary>

**Принцип единственной ответственности** -
`класс должен быть ответственным только за одну конкретную функцию или задачу.`

Если класс решает много задач, то получается много связанного кода, что влечет плохую читабельность. Так же приходится изменять его каждый раз, когда одна из частей класса ломается. При этом есть риск сломать остальные части класса.

В классе `User` используется метод для сохранения в базу данных. При изменении работы с базами данных этот класс тоже будет изменен.

```plantuml
class User {
  - name
  - email
  + GetName()
  + GetEmail()
  + SaveToDatabase()
}
```

Решение: для базы данных создадим отдельный класс и пернесем сохранения пользователя в него.

```plantuml
class User {
  - name
  - email
  + GetName()
  + GetEmail()
}

class DataBase {
  ...
  + SaveToDatabase(User)
}

DataBase .> User
```

</details>

<details>
<summary>
  Open/Closed Principle
</summary>

**Принцип открытости/закрытости** -
`Расширяйте классы, но не изменяйте их первоначальный код`

**Открытый** - класс доступный для расширения. Есть возможность расширить набор его операций или добавить к нему новые поля, создав собсвенный подкласс.

**Закрытый** - класс с окончательно определенным интерфейсов, он не будет изменяться в будещем и готов для использования другими классами.

Если класс был окончательно написан и протестирован, в дальнейшем изменять его не желательно и если требуется расширение, то делать это только за счет добавления подклассов, не изменяя код родительского класса.

**Проблема:** Рассмотрим класс для обработки заказов, который содержит метод `GetShippingCost()`, рассчитывающий стоимость доставки. В текущей реализации при добавлении нового способа доставки приходится модифицировать этот метод, что нарушает принцип открытости/закрытости.

```plantuml
class Order {
  - shipping
  + GetTotal()
  + GetShippingCost()
}

note right of Order::GetShippingCost()
  if (shipping == "ground") {
    return 10; // 10 $
  }
  if (shipping == "air") {
    return 20; // 20 $
  }
end note

```

**Решение:** Вместо одного метода, который работает со всеми способами доставки, создадим отдельные классы `Ground` и `Air`, реализующие общий интерфейс доставки. Каждый класс будет самостоятельно рассчитывать стоимость доставки для своего способа. В классе `Order` мы будем вызывать методы этих классов доставки через общий интерфейс. Таким образом, при добавлении нового способа доставки в будущем достаточно будет создать соответствующий класс, реализующий этот интерфейс, без необходимости изменять существующие методы в классе `Order`.

```plantuml
class Order {
  - shipping: Shipping
  + GetTotal()
  + GetShippingCost()
}

interface Shipping {
  + GetCost()
}

class Ground {
  ...
  + GetCost()
}

class Air {
  ...
  + GetCost()
}

Order o-> Shipping
Shipping <|.. Air
Shipping <|.. Ground

note left of Order::GetShippingCost()
  return shipping.GetCost();
end note

note right of Air::GetCost()
  подсчитывает цену для заказов
end note

```


</details>

<details>
<summary>
  Liskov Substitution Principle
</summary>

**Принцип подстановки Лисков** -
`Подклассы должны дополнять, а не заменять функционал базового класса`.

Требования к переопределенным в подклассах методам:
<details>
<summary>
  Т1
</summary>

Типы параметров метода подкласса должны `совпадать` или быть более `абстрактными`, чем типы параметров базового метода.

**Пример:**
- Базовый класс содержит метод `AssignDriver(CarDriver d)`, который позволяет назначать водителей для автомобилей. Клиентский код всегда передает в метод водителя машины.
- **Хорошо:** Мы создали подкласс и переопределили метод назначения для любого водителя: `AssignDriver(Driver d)`. При передаче клиентским кодом водителя автомобиля новый метод сможет его назначить, ведь он умеет работать со всеми типами водителей.
- **Плохо:** Мы создали подкласс и переопределили метод только для гоночных водителей: `AssignDriver(RaceCarDriver d)`. Клиентский код все так же подаст обычного водителя. Но метод умеет работать только с гоночными водителями, и клиентский код сломается.

</details>

<details>
<summary>
  Т2
</summary>

Тип возвращаемого значения метода подкласса должен `совпадать` или быть `подтипом` возвращаемого значения базового метода.

Здесь все так же как и в первом требовании, но наоборот.

- **Базовый метод:** `GetUser(): User`. Клиентский код ожидает на выходе объект пользователя с базовыми полями _like_ и _email_.

- **Хорошо:** Метод подкласса: `GetUser(): AdminUser`. Клиентский код получит администратора, который является пользователем, но с дополнительными правами. Всё будет работать корректно.

- **Плохо:** Метод подкласса: `GetUser(): Object`. Клиентский код сломается, так как получит непонятный объект (возможно, строку или число), у которого нет полей _like_ и _email_, необходимых для работы.

</details>

<details>
<summary>
  Т3
</summary>

Метод подкласса не должен выбрасывать исключения, которые не свойственны базовому методу.

При переопределении метода нельзя выбрасывать новые типы исключений, которых нет в базовом методе. Клиентский код уже настроен на обработку конкретных исключений базового класса. Если подкласс добавит новое исключение, оно может не перехватиться и "уронить" клиентскую программу.

</details>

<details>
<summary>
  Т4
</summary>

Метод подкласса не должен ужесточать `пред-условие`.

Например, если базовый метод принимает любые целые числа, то подкласс не может ограничить работу только положительными значениями. Это нарушит ожидания клиентского кода.

</details>

<details>
<summary>
  Т5
</summary>

Метод подкласса не должен ослаблять `пост-условие`.

Если базовый метод гарантирует закрытие файлов после выполнения, то подкласс не может нарушать это обязательство, оставляя файлы открытыми. Клиентский код полагается на поведение базового класса и может некорректно завершить работу с висящими файловыми дескрипторами.

</details>

<details>
<summary>
  Т6
</summary>

Инварианты класса должны остаться без изменений.

**Инвариант** — это внутреннее правило объекта, которое никогда не нарушается. Например:

- У класса **Прямоугольник**: `ширина > 0 и высота > 0`
- У класса **Пользователь**: `email содержит "@"`
- У класса **КорзинаПокупок**: `общая_сумма >= 0`

Если базовый класс гарантирует, что у него `баланс >= 0`, то наследник не может разрешить отрицательный баланс. Все "правила жизни" объекта должны сохраняться.

</details>

<details>
<summary>
  Т7
</summary>

Подкласс не должен изменять значения приватных полей базового класса.

Это возможно в языке программирования Python, где нет четкой защиты полей.

</details>

</details>

<details>
<summary>
  Interface Segregation Principle
</summary>

Принцип разделения интерфейса -
`Клиенты не должны зависеть от методов, которые они не используют.`

"Толстые" интерфейсы необходимо разделять на более мелкие и специализированные, решающие конкретную задачу.

**Пример:**

Данным двум классам приходится реализовывать методы не относящиеся к их задачам. Для программиста нужны только методы для написания и тестирования кода, а для повара только метод готовки.

```plantuml
interface Worker {
  + Code()
  + Test()
  + Cooking()
}

class Programmer {
  + <color:green>Code()</color>
  + <color:green>Test()</color>
  + <color:red>Cooking()</color>
}

class Cook {
  + <color:red>Code()</color>
  + <color:red>Test()</color>
  + <color:green>Cooking()</color>
}

Worker <|.. Programmer
Worker <|.. Cook
```

Решение: разделим интерфейс `Worker` на тонкие `Coder`, `Tester` и `Chef`. Теперь классам не придется реализовывать лишние методы.

```plantuml
interface Coder {
  + Code()
}

interface Tester {
  + Test()
}

interface Chef {
  + Cooking()
}

class Programmer {
  + Code()
  + Test()
}

class Cook {
  + Cooking()
}

Coder <|.. Programmer
Tester <|.. Programmer
Chef <|.. Cook
```

</details>

<details>
<summary>
  Dependency Inversion Principle
</summary>

Принцип инверсии зависимостей -
`Классы верхних уровней не должны зависеть от классов нижних уровней. Оба должны зависеть от абстракций. Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.`

Классы `нижнего уровня` — это исполнители: они выполняют конкретные поручения(сохранить данные, прочитать данные и т.д.), в то время как классы `верхнего уровня` — управленцы, которые координируют работу и реализуют бизнес-процессы.

Как реализовать принцип:

1. Определите интерфейс с операциями, которые бизнес-логика требует от низкоуровневых компонентов
2. Сделайте бизнес-логику зависимой от этого интерфейса, а не от конкретных реализаций — это создаёт гибкую связь
3. Обеспечьте соответствие низкоуровневых классов созданному интерфейсу — теперь они зависят от контракта, определённого бизнес-логикой

**Пример:**

Есть рабочий, который умеет работать только с одним конкретным станком. Если станок сломался и нужно поставить новый - рабочий не сможет с ним работать, потому что новый станок работает по-другому.

```plantuml
class Worker {
  + Work()
}

class OldMachine {
  + StartOldWay()
}

Worker -> OldMachine : умеет работать только так
```

Решение: создадим универсальный пульт управления. Теперь рабочий учится работать с пультом, а не со станком. Любой станок можно подключить к этому пульту.

```plantuml
class Worker {
  + Work()
}

interface Controller {
  + Start()
  + Stop()
}

class UniversalController implements Controller {
  + Start()
  + Stop()
}

class OldMachine {
  + StartOldWay()
}

class NewMachine {
  + StartNewWay()
}

Worker -> Controller
UniversalController --> OldMachine
UniversalController --> NewMachine
```

</details>

## Паттерны проектирования

### Порождающие паттерны

Паттерны отвечают за удобное и безопасное создание новых объектов или даже целых семейт объектов.

Factory Method
Фабричный метод -

Abstract Factory
Абстрактная фабрика - это порождающий паттерн проектирования, который позволяет создавать семейства связных объектов, не привязываясь к конкретным классам создаваемых объектов.

<details>
<summary>
  Проблема
</summary>

Представим, что мы пишем магазин автомобилей. Магазин занимается продажей семейства седанов, внедорожников, спорткаров от разных производителей: Toyota, BMW.

На ранних этапах наш код создания автомобилей будет выглядить вот так:

```cpp
Car* CreateSedan(string brand) {
  if (brand == "Toyota") {
    return new ToyotaSedan();
  } else if (brand == "BMW") {
    return new BMWSedan();
  }
}

Car* CreateSUV(string brand) {
  if (brand == "Toyota") {
    return new ToyotaSUV();
  } else if (brand == "BMW") {
    return new BMWSUV();
  }
}

Car* CreateSportsCar(string brand) {
  if (brand == "Toyota") {
    return new ToyotaSportsCar();
  } else if (brand == "BMW") {
    return new BMWSportsCar();
  }
}
```

**Какие проблемы возникают?**

1. Клиент заказывает автомобили `BMW`, но получает Седан(BMW 5 Series), Внедорожник(Toypta RAV4), Спорткар(BMW M8). Клиент растроится. А ошибка произошла во время создания автомобилей:

```cpp
Car* sedan = CreateSedan("BMW");      // BMW 5 Series
Car* suv = CreateSUV("Toyota");       // Toypta RAV4 - ОШИБКА!
Car* sports = CreateSportsCar("BMW"); // BMW M8
```

2. Если мы захотим расширить парк машин, то придется изменять существующий код создания авто.
3. Дублирование кода.

</details>

<!-- <details>
<summary> -->
  Решение
<!-- </summary> -->

Для начала паттерн предлагает выделить общие интерфейсы для отдельных продуктов семейсв. Так каждое семейство автомобилей получат общий интерфейс `Седан`, `Внедорожник`, `Спорткар`. Например:

```plantuml
interface SUV {
  + Drive()
  + OffRoad()
}

class ToyotaSUV implements SUV{
  ...
  + Drive()
  + OffRoad()
}

class BMWSUV implements SUV{
  ...
  + Drive()
  + OffRoad()
}
```

Далее необходимо создать **абстракную фабрику**. Это общий интерфейс, который будет содержать методы создания всех автомобилей семейства: `CrateSedan()`, `CreateSUV()`, `CreateSportCar()`.

```plantuml
interface CarFactory {
  + CrateSedan()
  + CreateSUV()
  + CreateSportCar()
}
```

Для каждого бренда семейства мы должны создать свою собственную фабрику, реализуя абстрактный интерфейс.

```plantuml
interface CarFactory {
  + CrateSedan(): Sedan
  + CreateSUV(): SUV
  + CreateSportCar(): SportCar
}

class BMWFactory implements CarFactory {
  ...
  + CrateSedan(): Sedan
  + CreateSUV(): SUV
  + CreateSportCar(): SportCar
}

class ToyotaFactory implements CarFactory {
  ...
  + CrateSedan(): Sedan
  + CreateSUV(): SUV
  + CreateSportCar(): SportCar
}
```

Клиентский код работает только через общие интерфейсы:

- Клиент использует `CarFactory`, не зная конкретной фабрики
- Клиент использует `Sedan`/`SUV`/`SportsCar`, не зная конкретных моделей
- Не важно, какая фабрика - Toyota или BMW
- Важно, что все автомобили совместимы и одного бренда

**Пример:**

```cpp
// Клиенту безразлично, какая фабрика
void ClientCode(CarFactory& factory) { // Любая фабрика: Toyota или BMW
  // Фабрика сама "знает" какие модели создавать
  Sedan* sedan = factory.CreateSedan();     // Toyota Camry или BMW 5 Series
  SUV* suv = factory.CreateSUV();           // Toyota RAV4 или BMW X5
}
```

Можно легко заменять фабрики, не меняя клиентский код. Все созданные автомобили гарантированно совместимы друг с другом.

**Замечание:** фабрика создается отдельно - обычно через конфигурацию или системные настройки.

**Общая диаграмма паттерна:**

```plantuml
@startuml

interface AbstractFactory {
  + CreateProductA(): AbstractProductA
  + CreateProductB(): AbstractProductB
}

interface AbstractProductA {
  + OperationA()
}

interface AbstractProductB {
  + OperationB()
}

class ConcreteFactory1 implements AbstractFactory {
  + CreateProductA(): AbstractProductA
  + CreateProductB(): AbstractProductB
}

class ConcreteFactory2 implements AbstractFactory {
  + CreateProductA(): AbstractProductA
  + CreateProductB(): AbstractProductB
}

class ProductA1 {
  + OperationA()
}

class ProductB1 {
  + OperationB()
}

class ProductA2{
  + OperationA()
}

class ProductB2 {
  + OperationB()
}

ProductB2 ..|> AbstractProductB
ProductA2 ...|> AbstractProductA


ProductB1 ..|> AbstractProductB
ProductA1 ...|> AbstractProductA


ConcreteFactory1 --> ProductA1
ConcreteFactory1 --> ProductB1

ConcreteFactory2 --> ProductB2
ConcreteFactory2 --> ProductA2


class Client {
  - factory: AbstractFactory
  + Client(factory)
  + operate()
}

Client --> AbstractFactory
@enduml
```

</details>

Builder
Строитель

Prototype
Прототип

Singleton
Одиночка

