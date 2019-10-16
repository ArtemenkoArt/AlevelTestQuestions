# Shallow Vs Deep Copy

## Какая разница между поверхностным (shallow) и глубоким (deep) копированием?

При поверхностном копировании копируются значения полей класса, включая значения любых указателей или ссылок. При этом скопированные значения этих указателей и ссылок указывают на одни и те же объекты, что и в оригинальном объекте, что зачастую ведет к ошибкам. Отсюда и название такого метода копирования: мы копируем только указатели/ссылки, вместо того, чтобы делать копии этих внутренних объектов и ссылаться на них, собственно не углубляемся во внутреннюю структуру объекта. При глубоком копировании мы копируем значения полей не только на первом "уровне", но и заходим глубже, копируя все значения.

## Копирование объектов. Интерфейс ICloneable

Поскольку классы представляют ссылочные типы, то это накладывает некоторые ограничения на их использование. В частности:

```csharp
class Program
{
    static void Main(string[] args)
    {
        Person p1 = new Person { Name="Tom", Age = 23 };
        Person p2 = p1;
        p2.Name = "Alice";
        Console.WriteLine(p1.Name); // Alice
 
        Console.Read();
    }
}
 
class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

В данном случае объекты p1 и p2 будут указывать на один и тот же объект в памяти, поэтому изменения свойств в переменной p2 затронут также и переменную p1.
Чтобы переменная p2 указывала на новый объект, но со значениями из p1, мы можем применить клонирование с помощью реализации интерфейса **ICloneable**:

```csharp
public interface ICloneable
{
    object Clone();
}
```
Реализация интерфейса в классе Person могла бы выглядеть следующим образом:

```csharp
class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public object Clone()
    {
        return new Person { Name = this.Name, Age = this.Age };
    }
}
```
Использование:

```csharp
class Program
{
    static void Main(string[] args)
    {
        Person p1 = new Person { Name="Tom", Age = 23 };
        Person p2 = (Person)p1.Clone();
        p2.Name = "Alice";
        Console.WriteLine(p1.Name); // Tom
 
        Console.Read();
    }
}
```
Теперь все нормально копируется, изменения в свойствах p2 не сказываются на свойствах в p1.
Для сокращения кода копирования мы можем использовать специальный метод **MemberwiseClone()**, который возвращает копию объекта:

```csharp
class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public object Clone()
    {
        return this.MemberwiseClone();
    }
}
```
Этот метод реализует **поверхностное (неглубокое) копирование**. Однако данного копирования может быть недостаточно. Например, пусть класс Person содержит ссылку на объект Company:

```csharp
class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Company Work { get; set; }
 
    public object Clone()
    {
            return this.MemberwiseClone();
    }
}
 
class Company
{
    public string Name { get; set; }
}
```
В этом случае при копировании новая копия будет указывать на тот же объект Company:

```csharp
Person p1 = new Person { Name="Tom", Age = 23, Work= new Company { Name = "Microsoft" } };
Person p2 = (Person)p1.Clone();
p2.Work.Name = "Google";
p2.Name = "Alice";
Console.WriteLine(p1.Name); // Tom
Console.WriteLine(p1.Work.Name); // Google - а должно быть Microsoft
```
Поверхностное копирование работает только для свойств, представляющих примитивные типы, но не для сложных объектов. И в этом случае надо применять **глубокое копирование**:

```csharp
class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Company Work { get; set; }
 
    public object Clone()
    {
        Company company = new Company { Name = this.Work.Name };
        return new Person
        {
            Name = this.Name,
            Age = this.Age,
            Work = company
        };
    }
}
 
class Company
{
    public string Name { get; set; }
}
```