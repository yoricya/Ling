# Ling
Ling — это язык, который читается почти как книга, где переменные — это всегда ссылки.

### Объявление функций
Функции объявляются с ключевым словом `fn`, после которого указывается имя функции. Аргументы передаются после `with`, а их типы указываются после `as`. Возвращаемый тип пишется после `->`.

<b>Пример простой функции:</b>
```ling
fn greet with name as string -> string:
  return "Hello, " + name

fn main:
  println with (greet with "Yoricya")
```

__Функции без возвращаемого значения__
```ling
fn say_hello with name as string:
  println with "Hello, " + name
```
Если функция ничего не возвращает, не нужно указывать `->`.

__Функции без аргументов__
Функции без аргументов объявляются так же, просто без `with`:
```ling
fn print_message:
  println with "This is Ling!"

fn main:
  print_message //Вызываются так же без with
```

__Функции с несколькими аргументами__
```ling
fn sum with a as i32, b as i32 -> i32:
  return a + b

fn divide with a, b as i32 -> i32: // Можно обьединить аргументы под один тип
  return a / b

fn main:
  println with (sum with a as 1, b as 2) // Если в функции несколько аргументов то при вызове указываем каждый
  
```

### Объекты aka Структуры
Обьекты или структуры объявляются с ключевым словом `object` после которого следует имя обьекта.
```ling
object Person:
  age as i32
  name as immut string // Так же можно обьявлять их как immut (см Особенности переменных)
```

__Функции обьекта__

Объекты могут содержать функции, внутри можно обращаться к полям объекта через `self`.

```ling
object Person:
  name as string
  age as i32

  fn greet -> string:
    return "Hello, my name is " + self.name
```

__Инициализация__
```ling
object Counter:
  value as i32

  fn increment:
    self.value = self.value + 1

  fn get_value -> i32:
    return self.value

fn main:
  define counter = new Counter with value as 1 // Если не указать value то по умолчанию будет 0
  counter.increment
  println with counter.get_value  // 2
```

__Приватные поля__

По умолчанию поля доступны снаружи, но их можно сделать приватными с `private`. Тогда они будут доступны только внутри методов этого объекта.

```ling
object BankAccount:
  private balance as immut i32 // Обьявили как immut чтобы функция get_balance не могла передать оригинальную ссылку на value, но можно указать immut в return type у функции

  fn deposit with amount as i32:
    self.balance = self.balance + amount

  fn get_balance -> i32:
    return self.balance

fn main:
  define acc = new BankAccount

  acc.deposit with 100
  println with acc.get_balance  // 100

  println with acc.balance  // Ошибка! balance — приватное поле
```


### Особенности переменных
Все переменные в Ling — это ссылки. Это значит, что когда ты создаёшь переменную, она ссылается на данные, а не хранит их.
При передаче переменных в функцию или создании новой переменной на её основе будет использована ссылка на те же данные. То есть изменения в одной переменной могут отразиться на другой.

```ling
fn change_value with x as i32:
  x = 456  // меняем значение x

fn main:
  define a = 123
  define b = a  // b ссылается на те же данные, что и a

  println with a  // 123
  println with b  // 123

  change_value with a  // передаем a в функцию, меняем значение

  println with a  // 456
  println with b  // 456, потому что a и b ссылаются на одни и те же данные
```
`a` и `b` ссылаются на одни и те же данные. Когда мы передаем `a` в функцию `change_value`, она изменяет значение на которое ссылаются и `a` и `b`, поэтому после вызова функции оба равны 456

<b>Пример с immut переменными</b>
```ling
fn change_value with x as i32:
  x = 456  // меняем значение x

fn main:
  define immut a = 123
  define b = a  // b копирует данные из a

  println with a  // 123
  println with b  // 123

  change_value with a  // передаем a в функцию, меняем значение x

  println with a  // 123, так как переменная помеченная как immut не передала свою ссылку в функцию а скопировала себя и передала новую ссылку
  println with b  // 123, так как b — это копия данных из a
```

<b>Так же можно копировать переменные со стороны функции:</b>
```ling
fn change_value with x as immut i32:
  x = 456  // меняем значение x, извне не имеет никакого смысла так как меняем данные новой ссылки которая сделала функция для этого аргумента
```

<b>Можно указать возвращаемый тип как immut, тогда функция при возврате значения будет сама создавать новую ссылку не передавая оригинальную</b>
```ling
fn change_value with x as i32 -> immut i32:
  return x // Функция создаст новую ссылку скопирует туда x и вернет ее
```

<b>Без immut возвращается оригинальаня ссылка</b>
```ling
fn change_value with x as i32 -> i32:
  return x // Возвращаем ссылку на x
```

