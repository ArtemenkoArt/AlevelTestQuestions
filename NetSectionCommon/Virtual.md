# How does Virtual modifier work?

## Полезные ссылки
[Virtual Method in C#](https://www.c-sharpcorner.com/UploadFile/3d39b4/virtual-method-in-C-Sharp/)

[C# Virtual Keyword](https://www.tutlane.com/tutorial/csharp/csharp-virtual-keyword)

[What is virtual class in C# with an example?](https://www.quora.com/What-is-virtual-class-in-C-with-an-example)

[C# Abstract Properties, Virtual Properties and Access Modifiers](http://www.cunningplanning.com/post/csharp-abstract-virtual-access-properties/)

[virtual](https://docs.microsoft.com/ru-ru/dotnet/csharp/language-reference/keywords/virtual)

[override](https://docs.microsoft.com/ru-ru/dotnet/csharp/language-reference/keywords/override)

[Виртуальные методы и свойства](https://metanit.com/sharp/tutorial/3.19.php)



##C# Virtual Keyword

Ключевое слово ***virtual*** используется для изменения объявлений методов, свойств, индексаторов и событий и разрешения их переопределения в производном классе. Например, этот метод может быть переопределен любым наследующим его классом:

```csharp 
public class Users
{
    public virtual void GetInfo()
    {
        Console.WriteLine("Base Class");
    }
}
```

В коде мы определили ***GetInfo*** метод с виртуальным ключевым словом таким образом, ***GetInfo*** метод может быть переопределен любым производным классом, который наследует свойства от базового класса пользователей.

При вызове виртуального метода тип времени выполнения объекта проверяется на переопределение члена. Вызывается переопределение члена в самом дальнем классе. Это может быть исходный член, если никакой производный класс не выполнял переопределение этого члена.

По умолчанию методы не являются виртуальными. Такой метод переопределить невозможно.

Нельзя использовать модификатор ***virtual*** с модификаторами ***static***, ***abstract***, ***private***, или ***override***. 

> Ниже приведен пример использования виртуального ключевого слова для переопределения членов класса в производном классе

```csharp 

using System; 

namespace Tutlane
{
        // Base Class

    public class BClass
    {
        public virtual string Name { get; set; }
        public virtual void GetInfo()
        {
            Console.WriteLine("Learn C# Tutorial");
        }
    }

    // Derived Class

    public class DClass : BClass
    {
        private string name;
        public override string Name
        {
            get
            {
                return name;
            }
            set
            {
                if (!string.IsNullOrEmpty(value))
                {
                    name = value;
                }
                else
                {
                    name = "No Value";
                }
            }
        }
        public override void GetInfo()
        {
            Console.WriteLine("Welcome to Tutlane");
        }
    }
    class Program
    {
        static void Main(string[] args)
        {
            DClass d = new DClass();
            d.GetInfo();
            BClass b = new BClass();
            b.GetInfo();
            d.Name = string.Empty;
            Console.WriteLine(d.Name);
            Console.WriteLine("\nPress Enter Key to Exit..");
            Console.ReadLine();
        }
    }
}

```
Если вы посмотрите на приведенный выше пример, мы переопределяем методы  и  свойства базового класса ( BClass ), которые мы определили с помощью виртуального ключевого слова в производном классе ( DClass ) с помощью ключевого слова override.

## override
Модификатор ***override*** требуется для расширения или изменения абстрактной или виртуальной реализации унаследованного метода, свойства, индексатора или события.

Метод ***override*** предоставляет новую реализацию члена, унаследованного от базового класса. Метод, переопределенный объявлением ***override***, называется переопределенным базовым методом. Переопределенный базовый метод должен иметь ту же сигнатуру, что и метод ***override***.

Невиртуальный или статический метод нельзя переопределить. Переопределенный базовый метод должен иметь тип ***virtual***, ***abstract*** или ***override***.
Объявление ***override*** не может изменить доступность метода ***virtual***. Методы ***override*** и ***virtual*** должны иметь одинаковый модификатор уровня доступа.

Модификаторы new, static и virtual нельзя использовать для изменения метода override.
Переопределяющее объявление свойства должно задавать такие же модификатор уровня доступа, тип и имя, которые имеются у унаследованного свойства, а переопределенное свойство должно иметь тип virtual, abstract или override.

>В этом примере класс Square должен предоставить переопределенную реализацию Area, так как Area является унаследованным от абстрактного класса ShapesClass.

```csharp 
abstract class Shape
{
    public abstract int GetArea();
}

class Square : Shape
{
    int side;

    public Square(int n) => side = n;

    // GetArea method is required to avoid a compile-time error.
    public override int GetArea() => side * side;

    static void Main() 
    {
        var sq = new Square(12);
        Console.WriteLine($"Area of the square = {sq.GetArea()}");
    }

    interface I
    {
        void M();
    }
    abstract class C : I
    {
        public abstract void M();
    }

}
// Output: Area of the square = 144

```


## Переопределение свойств

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
##Ключевое слово base
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

## Запрет переопределения методов
Также можно запретить переопределение методов и свойств. В этом случае их надо объявлять с модификатором sealed:

```csharp 
class Employee : Person
{
    public string Company { get; set; }
  
    public Employee(string firstName, string lastName, string company)
                : base(firstName, lastName)
    {
        Company = company;
    }
 
    public override sealed void Display()
    {
        Console.WriteLine($"{FirstName} {LastName} работает в {Company}");
    }
}
```
При создании методов с модификатором sealed надо учитывать, что sealed применяется в паре с override, то есть только в переопределяемых методах.

И в этом случае мы не сможем переопределить метод Display в классе, унаследованном от Employee.
