# Overriding

## Полезные ссылки

[Виртуальные методы и свойства](https://metanit.com/sharp/tutorial/3.19.php)

## Виртуальные методы и свойства
При наследовании нередко возникает необходимость изменить в классе-наследнике функционал метода, который был унаследован от базового класса. В этом случае класс-наследник может переопределять методы и свойства базового класса.

Те методы и свойства, которые мы хотим сделать доступными для переопределения, в базовом классе помечается модификатором **virtual**. Такие методы и свойства называют виртуальными.

А чтобы переопределить метод в классе-наследнике, этот метод определяется с модификатором **override**. Переопределенный метод в класе-наследнике должен иметь тот же набор параметров, что и виртуальный метод в базовом классе.
Например, рассмотрим следующие классы:
```csharp
class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public Person(string firstName, string lastName)
    {
        FirstName = firstName;
        LastName = lastName;
    }
    public virtual void Display()
    {
        Console.WriteLine($"{FirstName} {LastName}");
    }
}
class Employee : Person
{
    public string Company { get; set; }
    public Employee(string firstName, string lastName, string company)
        : base(firstName, lastName)
    {
        Company = company;
    }
}
```
### Также как и методы, можно переопределять свойства:
```csharp
class Credit
{
    public virtual decimal Sum { get; set; }
}
class LongCredit : Credit
{
    private decimal sum;
    public override decimal Sum
    {
        get
        {
            return sum;
        }
        set
        {
            if(value > 1000)
            {
                sum = value;
            }
        }
    }
}
class Program
{
    static void Main(string[] args)
    {
        LongCredit credit = new LongCredit { Sum = 6000 };
        credit.Sum = 490;
        Console.WriteLine(credit.Sum);
        Console.ReadKey();
    }
}
```
### Ключевое слово base

Кроме конструкторов, мы можем обратиться с помощью ключевого слова base к другим членам базового класса. В нашем случае вызов base.Display(); будет обращением к методу Display() в классе Person:
```csharp
class Employee : Person
{
    public string Company { get; set; }
  
    public Employee(string lastName, string firstName, string company)
            :base(firstName, lastName)
    {
        Company = company;
    }
  
    public override void Display()
    {
        base.Display();
        Console.WriteLine($"работает в {Company}");
    }
}
```
### Запрет переопределения методов
Также можно запретить переопределение методов и свойств. В этом случае их надо объявлять с модификатором **sealed**:
```csharp
    public override sealed void Display()
    {
        Console.WriteLine($"{FirstName} {LastName} работает в {Company}");
    }
```
### Абстрактные методы (свойства так же)

```csharp
abstract class Person
{
...
    public abstract void Display();
}
 
class Client : Person
{
...
    public override void Display()
    {
        Console.WriteLine($"{Name} имеет счет на сумму {Sum}");
    }
}
```
