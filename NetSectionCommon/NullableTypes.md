# Nullable types

## Nullable-типы

Значение null по умолчанию могут принимать только объекты ссылочных типов. Однако в различных ситуациях бывает удобно, чтобы объекты числовых типов данных имели значение null, то есть были бы не определены. Стандартный пример - работа с базой данных, которая может содержать значения null. И мы можем заранее не знать, что мы получим из базы данных - какое-то определенное значение или же null. Для этого надо использовать знак вопроса ? после типа значений. Например:

```csharp
int? z = null;
bool? enabled = null;
```
Но фактически запись ? является упрощенной формой использования структуры **System.Nullable<T>**. Параметр T в угловых скобках представляет универсальный параметр, вместо которого в конкретной задача уже подставляется конкретный тип данных. Следующие виды определения переменных будут эквивалентны:

```csharp
int? z1 = 5;
bool? enabled1 = null;
Double?  d1 = 3.3;
 
Nullable<int> z2 = 5;
Nullable<bool> enabled2 = null;
Nullable<System.Double> d2 = 3.3;
```
Для всех типов Nullable определено два свойства: **Value**, которое представляет значение объекта, и **HasValue**, которое возвращает true, если объект Nullable хранит некоторое значение.
Причем свойство Value хранит объект того типа, которым типизируется Nullable:

```csharp
class Program
{
    static void Main(string[] args)
    {
        int? x = 7;
        Console.WriteLine(x.Value); // 7
        Nullable<State> state = new State() { Name = "Narnia" };
        Console.WriteLine(state.Value.Name);    // Narnia
 
        Console.ReadLine();
    }
}
struct State
{
    public string Name { get; set; }
}
```
Однако если мы попробуем получить значение переменной, которая равна null, то мы столкнемся с ошибкой:

```csharp
int? x = null;
Console.WriteLine(x.Value); // Ошибка
State? state = null;
Console.WriteLine(state.Value.Name);    // ошибка
```
В этом случае необходимо выполнять проверку на наличие значения с помощью еще одного свойства HasValue:

```csharp
int? x = null;
if(x.HasValue)
    Console.WriteLine(x.Value);
else
    Console.WriteLine("x is equal to null");
 
State? state = null;
if(state.HasValue)
    Console.WriteLine(state.Value.Name);
else
    Console.WriteLine("state is equal to null");
```
Также надо учитывать, что структура Nullable применяется только для типов значений, поскольку ссылочные типы итак могут иметь значение null:

```csharp
class Program
{
    static void Main(string[] args)
    {
        Nullable<State> state = null; // State - значимый тип
        // Nullable<Country> country = null;  Ошибка - Country - ссылочный тип
 
        Console.ReadLine();
    }
}
class Country
{
    public string Name { get; set; }
}
struct State
{
    public string Name { get; set; }
}
```
Также структура Nullable с помощью метода GetValueOrDefault() позволяет использовать значение по умолчанию (для числовых типов это 0), если значение не задано:

```csharp
int? figure = null;
Console.WriteLine(figure.GetValueOrDefault()); // по умолчанию используется 0
Console.WriteLine(figure.GetValueOrDefault(10)); // по умолчанию используется 10
```
##Равенство объектов
При проверке объектов на равенство следует учитывать, что они равны не только, когда они имеют ненулевые значения, которые совпадают, но и когда оба объекта равны null:

```csharp
int? x1 = null;
int? x2 = null;
if (x1 == x2)
    Console.WriteLine("объекты равны");
else
    Console.WriteLine("объекты не равны");
    
```
Если же один объект имеет значение null, а другой нет, или же оба объекта имеют разные значения, то они не равны.

##Преобразование типов Nullable
Рассмотрим возможные преобразования:
-   явное преобразование от T? к T
```csharp
int? x1 = null;
if(x1.HasValue)
{
    int x2 = (int)x1;
    Console.WriteLine(x2);
}
```
-   неявное преобразование от T к T?

```csharp
int x1 = 4;
int? x2 = x1;
Console.WriteLine(x2);
```
-   неявные расширяющие преобразования от V к T?
```csharp
int x1 = 4;
long? x2 = x1;
Console.WriteLine(x2);
```
-   явные сужающие преобразования от V к T?
```csharp
long x1 = 4;
int? x2 = (int?)x1;
```
-   Подобным образом работают преобразования от V? к T?
```csharp
long? x1 = 4;
int? x2 = (int?)x1;
```
-   явные преобразования от V? к T
```csharp
long? x1 = 4;
int x2 = (int)x1;
```
 
