
# Принципы проектирования архитектуры

## Повторное использование кода

##

## SOLID

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

**Закрытый** - класс с окончательно определенным интерфейсов, он не будет изменяться в будещем и  готов для использования другими классами.

```plantuml
class Order {
  - shipping
  + GetTotal()
  + GetShippingCost()
}
```

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
  L
</summary>
Принцип открытости/закрытости -

</details>

<details>
<summary>
  I
</summary>
Принцип открытости/закрытости -

</details>

<details>
<summary>
  D
</summary>
Принцип открытости/закрытости -

</details>


##
