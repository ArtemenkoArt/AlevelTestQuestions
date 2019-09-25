# SOLID, KISS, DRY

## SOLID

* **Принцип единственности ответственности**

* **Принцип открытости/закрытости**

* **Принцип замещения Лисков**

* **Принцип разделения интерфейса**

* **Принцип инверсии зависимости**

## Принцип единственности ответственности

НЕ ДОЛЖНО БЫТЬ БОЛЬШЕ ОДНОЙ ПРИЧИНЫ ДЛЯ ИЗМЕНЕНИЯ КЛАССА

Если у объекта много ответвенности, то и меняться он будет очень часто. Поэтому, если класс имеет больше одной ответственности, то это ведет к хрупкости дизайна и ошибкам в неожиданных местах при изменениях кода.

Валидация данных

Проблема:

```csharp
        static void Main(string[] args)
        {
            var product = new Product { Price = 100 };
            var isValid = product.IsValid();
        }

        public class Product
        {
            public int Price { get; set; }

            public bool IsValid()
            {
                return Price > 0;
            }
        }
```

Теперь наш объект Product начал использовать в некоем CustomerService, который считает валидным продукт с ценой больше 100 тыс. рублей. Что делать? Уже сейчас понятно, что нам придется изменять наш объект продукта, например, таким образом:

```csharp
        static void Main(string[] args)
        {
            var product = new Product { Price = 100 };
            var isValid = product.IsValid(true);
        }

        public class Product
        {
            public int Price { get; set; }

            public bool IsValid(bool isCustomerService)
            {
                if (isCustomerService == true)
                    return Price > 100000;

                return Price > 0;
            }
        }
```

Решение:

Стало очевидно, что при дальнейшем использовании объекта Product логика валидации его данных будет изменяться и усложняться. Видимо пора отдать ответственность за валидацию данных продукта другому объекту. Причем надо сделать так, чтобы сам объект продукта не зависел от конкретной реализации его валидатора. Получаем код:

```csharp
        static void Main(string[] args)
        {
            // обычное использование
            var product = new Product { Price = 100 };

            // используем объект продукта в новом сервисе
            var product1 = new Product(new CustomerServiceProductValidator()) { Price = 100 };

        }

        public interface IProductValidator
        {
            bool IsValid(Product product);
        }

        public class ProductDefaultValidator : IProductValidator
        {
            public bool IsValid(Product product)
            {
                return product.Price > 0;
            }
        }

        public class CustomerServiceProductValidator : IProductValidator
        {
            public bool IsValid(Product product)
            {
                return product.Price > 100000;
            }
        }

        public class Product
        {
            private readonly IProductValidator validator;

            public Product() : this(new ProductDefaultValidator())
            {
            }

            public Product(IProductValidator validator)
            {
                this.validator = validator;
            }

            public int Price { get; set; }

            public bool IsValid()
            {
                return validator.IsValid(this);
            }
        }
```

Имеем объект Product отдельно, а любое количество всяческих валидаторов отдельно.

## Принцип открытости/закрытости

ПРОГРАММНЫЕ СУЩНОСТИ (КЛАССЫ, ФУНКЦИИ И ТД.) ДОЛЖНЫ БЫТЬ ОТКРЫТЫ ДЛЯ РАСШИРЕНИЯ, НО ЗАКРЫТТЫ ДЛЯ ИЗМЕНЕНИЯ

Принцип открытости/закрытость как раз и дает понимание того, как оставаться достаточно гибкими в условиях постоянно меняющихся требований.

Проверка типа абстракции

Проблема:

У нас есть иерархия объектов с абстрактным родительским классом AbstractEntity и класс Repository, который использует абстракцию. При этом вызывая метод Save у Repository мы строим логику в зависимости от типа входного параметра:

```csharp
public abstract class AbstractEntity
{
}
  
public class AccountEntity : AbstractEntity
{
}
  
public class RoleEntity : AbstractEntity
{
}
  
public class Repository
{
    public void Save(AbstractEntity entity)
    {
        if (entity is AccountEntity)
        {
            // специфические действия для AccountEntity
        }
        if (entity is RoleEntity)
        {
            // специфические действия для RoleEntity
        }
    }
}
```
Решение:

Чтобы решить данную проблему, необходимо логику сохранения конкретных классов из иерархии AbstractEntity вынести в конкретные классы Repository. Для этого мы должны выделить интерфейс IRepository и создать хранилища AccountRepository и RoleRepository:

```csharp
public abstract class AbstractEntity
{
}
  
public class AccountEntity : AbstractEntity
{
}
  
public class RoleEntity : AbstractEntity
{
}
  
public interface IRepository<T> where T : AbstractEntity
{
    void Save(T entity);
}
  
public class AccountRepository : IRepository<AccountEntity>
{
    public void Save(AccountEntity entity)
    {
        // специфические действия для AccountEntity
    }
}
  
public class RoleRepository : IRepository<RoleEntity>
{
    public void Save(RoleEntity abstractEntity)
    {
        // специфические действия для RoleEntity
    }
}
```

## Принцип замещения Лисков

ДОЛЖНА БЫТЬ ВОЗМОЖНОСТЬ ВМЕСТО БАЗОВОГО ТИПА ПОДСТАВИТЬ ЛЮБОЙ ЕГО ПРОИЗВОДНЫЙ ТИП

Ошибочное наследование

Проблему, с который связан принцип Лисков, наглядно можно продемонстрировать на примере двух классов Прямоугольника и Квадрата. Пусть они будут выглядеть следующим образом:

```csharp
class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
     
    public int GetArea()
    {
        return Width * Height;
    }
}
 
class Square : Rectangle
{
    public override int Width
    {
        get
        {
            return base.Width;
        }
 
        set
        {
            base.Width = value;
            base.Height = value;
        }
    }
 
    public override int Height
    {
        get
        {
            return base.Height;
        }
 
        set
        {
            base.Height = value;
            base.Width = value;
        }
    }
}
```

На первый взгляд вроде все правильно, классы предельно простые, всего два свойства, и, казалось бы, сложно где-то ошибиться. Однако представим ситуацию, что в главной программе у нас следующий код:

```csharp
class Program
{
    static void Main(string[] args)
    {
        Rectangle rect = new Square();
        TestRectangleArea(rect);
 
        Console.Read();
    }
 
    public static void TestRectangleArea(Rectangle rect)
    {
        rect.Height = 5;
        rect.Width = 10;
        if (rect.GetArea() != 50)
            throw new Exception("Некорректная площадь!");
    }
}
```

Проблема заключается в том, что производный класс Square не ведет себя как базовый класс Rectangle, и поэтому его не следует наследовать от данного базового класса. В этом и есть практический смысл принципа Лисков. Производный класс, который может делать меньше, чем базовый, обычно нельзя подставить вместо базового, и поэтому он нарушает принцип подстановки Лисков.

## Принцип разделения интерфейса

КЛИЕНТЫ НЕ ДОЛЖНЫ ВЫНУЖДЕНО ЗАВИСЕТЬ ОТ МЕТОДОВ, КОТОРЫМИ НЕ ПОЛЬЗУЮТСЯ

Пример:

Допустим у нас есть интерфейс отправки сообщения:

```csharp
interface IMessage
{
    void Send();
    string Text { get; set;}
    string Subject { get; set;}
    string ToAddress { get; set; }
    string FromAddress { get; set; }
}
```
И пусть есть класс EmailMessage, который реализует этот интерфейс:
```csharp
class EmailMessage : IMessage
{
    public string Subject { get; set; }
    public string Text { get; set; }
    public string FromAddress { get; set; }
    public string ToAddress { get; set; }
 
    public void Send()
    {
        Console.WriteLine("Отправляем по Email сообщение: {0}", Text);
    }
}
```
Теперь определим класс, который бы отправлял данные по смс:
```csharp
class SmsMessage : IMessage
{
    public string Text { get; set; }
    public string FromAddress { get; set; }
    public string ToAddress { get; set; }
        
    public string Subject
    {
        get
        {
            throw new NotImplementedException();
        }
 
        set
        {
            throw new NotImplementedException();
        }
    }
 
    public void Send()
    {
        Console.WriteLine("Отправляем по Sms сообщение: {0}", Text);
    }
}
```
Здесь мы уже сталкиваемся с небольшой проблемой: свойство Subject, которое определяет тему сообщения, при отправке смс не указывается, поэтому оно в данном классе не нужно. Таким образом, в классе SmsMessage появляется избыточная функциональность, от которой класс SmsMessage начинает зависеть.

Это не очень хорошо, но посмотрим дальше. Допустим, нам надо создать класс для отправки голосовых сообщений.

Класс голосовой почты также имеет отправителя и получателя, только само сообщение передается в виде звука, что на уровне C# можно выразить в виде массива байтов. И в этом случае было бы неплохо, если бы интерфейс IMessage включал бы в себя дополнительные свойства и методы для этого, например:

```csharp
interface IMessage
{
    void Send();
    string Text { get; set;}
    string ToAddress { get; set; }
    string Subject { get; set; }
    string FromAddress { get; set; }
 
    byte[] Voice { get; set; }
}
```
Тогда класс голосовой почты VoiceMessage мог бы выглядеть следующим образом:
```csharp
class VoiceMessage : IMessage
{
    public string ToAddress { get; set; }
    public string FromAddress { get; set; }
    public byte[] Voice { get; set; }
 
    public string Text
    {
        get
        {
            throw new NotImplementedException();
        }
 
        set
        {
            throw new NotImplementedException();
        }
    }
 
    public string Subject
    {
        get
        {
            throw new NotImplementedException();
        }
 
        set
        {
            throw new NotImplementedException();
        }
    }
 
    public void Send()
    {
        Console.WriteLine("Передача голосовой почты");
    }
}
```

И здесь опять же мы сталкиваемся с ненужными свойствами. Плюс нам надо добавить новое свойство в предыдущие классы SmsMessage и EmailMessage, причем этим классам свойство Voice в принципе то не нужно. В итоге здесь мы сталкиваемся с явным нарушением принципа разделения интерфейсов.

Для решения возникшей проблемы нам надо выделить из классов группы связанных методов и свойств и определить для каждой группы свой интерфейс:
```csharp
interface IMessage
{
    void Send();
    string ToAddress { get; set; }
    string FromAddress { get; set; }
}
interface IVoiceMessage : IMessage
{
    byte[] Voice { get; set; }
}
interface ITextMessage : IMessage
{
    string Text { get; set; }
}
 
interface IEmailMessage : ITextMessage
{
    string Subject { get; set; }
}
 
class VoiceMessage : IVoiceMessage
{
    public string ToAddress { get; set; }
    public string FromAddress { get; set; }
 
    public byte[] Voice { get; set; }
    public void Send()
    {
        Console.WriteLine("Передача голосовой почты");
    }
}
class EmailMessage : IEmailMessage
{
    public string Text { get; set; }
    public string Subject { get; set; }
    public string FromAddress { get; set; }
    public string ToAddress { get; set; }
 
    public void Send()
    {
        Console.WriteLine("Отправляем по Email сообщение: {0}", Text);
    }
}
 
class SmsMessage : ITextMessage
{
    public string Text { get; set; }
    public string FromAddress { get; set; }
    public string ToAddress { get; set; }
    public void Send()
    {
        Console.WriteLine("Отправляем по Sms сообщение: {0}", Text);
    }
}
```
## Принцип инверсии зависимости

«Зависимость на Абстракциях. Нет зависимости на что-то конкретное.»

Смотрим первый пример (Валидация данных) на наличие выделения каких-либо абстракций. Мы создали новый интерфейс IProductValidator, что и является нашей абстракцией валидации данных. Поэтому наша валидация данных никак теперь не влияет на класс Product.


## Вывод

Не забываем про такую вещь как интерфейсы в С#. И пользуемся ею максимально грамотно, для соблюдения правил SOLID.

## Keep it simple, stupid

* **Не имеет смысла реализовывать дополнительные функции, которые не будут использоваться вовсе или их использование крайне маловероятно**

* **Не стоит перегружать интерфейс лишними опциями, которые не будут нужны большенству пользователей**

* **Не стоит подключать огромную библиотеку, если вам нужно всего пару функций**

* **Данные нужно обрабатыввать с той точностью, которой достаточно для качественного решения задачи**

* **Не имеет смысла бес­пре­дельно уве­ли­чи­вать уро­вень абстрак­ции, надо уметь вовремя оста­но­вить­ся**

## Don’t repeat yourself

* **Не пишите один и тот же самый код по нескольку раз! Если вам приходится это делать, то явно вы что-то упустили из виду. Остановитесь, просмотрите свой код и найдите выход, как можно избавиться от дублирования кода!**