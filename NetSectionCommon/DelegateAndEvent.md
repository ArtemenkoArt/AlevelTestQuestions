# 1.0 Делегаты
## 1.1 Определение делегатов
## 1.2 Присвоение ссылки на метод
## 1.3 Соответствие методов делегату
## 1.4 Добавление методов в делегат
## 1.5 Объединение делегатов
## 1.6 Вызов делегата
## 1.7 Делегаты как параметры методов
## 1.8 Обобщенные делегаты

# 2.0 События
## 2.1 Класс данных события AccountEventArgs

# 3.0 Action, Predicate, Func
## 3.1 Action
## 3.2 Predicate
## 3.3 Func


### Полезные ссылки

[Делегаты](https://metanit.com/sharp/tutorial/3.13.php)
[События](https://metanit.com/sharp/tutorial/3.14.php)
[Action, Predicate, Func](https://metanit.com/sharp/tutorial/3.33.php)
[Делегаты и события в .NET](https://habr.com/ru/post/198694/)

Делегаты представляют такие объекты, которые указывают на методы. То есть делегаты - это указатели на методы и с помощью делегатов мы можем вызвать данные методы.

## 1.1 Определение делегатов

Для объявления делегата используется ключевое слово delegate, после которого идет возвращаемый тип, название и параметры. Например:
```csharp
delegate void Message();
```

Делегат Message в качестве возвращаемого типа имеет тип void (то есть ничего не возвращает) и не принимает никаких параметров. Это значит, что этот делегат может указывать на любой метод, который не принимает никаких параметров и ничего не возвращает.

Рассмотрим примение этого делегата:
```csharp
class Program
{
    delegate void Message(); // 1. Объявляем делегат
 
    static void Main(string[] args)
    {
        Message mes; // 2. Создаем переменную делегата
        if (DateTime.Now.Hour < 12)
        {
            mes = GoodMorning; // 3. Присваиваем этой переменной адрес метода
        }
        else
        {
            mes = GoodEvening;
        }
        mes(); // 4. Вызываем метод
        Console.ReadKey();
    }
    private static void GoodMorning()
    {
        Console.WriteLine("Good Morning");
    }
    private static void GoodEvening()
    {
        Console.WriteLine("Good Evening");
    }
}
```

Здесь сначала мы определяем делегат:
```csharp
delegate void Message(); // 1. Объявляем делегат
```

В данном случае делегат определяется внутри класса, но также можно определить делегат вне класса внутри пространства имен.
Для использования делегата объявляется переменная этого делегата:
```csharp
Message mes; // 2. Создаем переменную делегата
```

С помощью свойства DateTime.Now.Hour получаем текущий час. И в зависимости от времени в делегат передается адрес определенного метода. Обратите внимание, что методы эти имеют то же возвращаемое значение и тот же набор параметров (в данном случае отсутствие параметров), что и делегат.
```csharp
mes = GoodMorning; // 3. Присваиваем этой переменной адрес метода
```

Затем через делегат вызываем метод, на который ссылается данный делегат:
```csharp
mes(); // 4. Вызываем метод
```

Вызов делегата производится подобно вызову метода.
Посмотрим на примере другого делегата:
```csharp
class Program
{
    delegate int Operation(int x, int y);
     
    static void Main(string[] args)
    {
        // присваивание адреса метода через контруктор
        Operation del = Add; // делегат указывает на метод Add
        int result = del(4,5); // фактически Add(4, 5)
        Console.WriteLine(result);
 
        del = Multiply; // теперь делегат указывает на метод Multiply
        result = del(4, 5); // фактически Multiply(4, 5)
        Console.WriteLine(result);
 
        Console.Read();
    }
    private static int Add(int x, int y)
    {
        return x+y;
    }
    private static int Multiply (int x, int y)
    {
        return x * y;
    }
}
```

В данном случае делегат Operation возвращает значение типа int и имеет два параметра типа int. Поэтому этому делегату соответствует любой метод, который возвращает значение типа int и принимает два параметра типа int. В данном случае это методы Add и Multiply. То есть мы можем присвоить переменной делегата любой из этих методов и вызывать.

Поскольку делегат принимает два параметра типа int, то при его вызове необходимо передать значения для этих параметров: del(4,5).

Делегаты необязательно могут указывать только на методы, которые определены в том же классе, где определена переменная делегата. Это могут быть также методы из других классов и структур.
```csharp
class Math
{
    public int Sum(int x, int y) { return x + y; }
}
class Program
{
    delegate int Operation(int x, int y);
 
    static void Main(string[] args)
    {
        Math math = new Math();
        Operation del = math.Sum;
        int result = del(4, 5);     // math.Sum(4, 5)
        Console.WriteLine(result);  // 9
 
        Console.Read();
    }
}
```

## 1.2 Присвоение ссылки на метод

Выше переменной делегата напрямую присваивался метод. Есть еще один способ - создание объекта делегата с помощью конструктора, в который передается нужный метод:
```csharp
class Program
{
    delegate int Operation(int x, int y);
 
    static void Main(string[] args)
    {
        Operation del = Add;
        Operation del2 = new Operation(Add);
 
        Console.Read();
    }
    private static int Add(int x, int y) { return x + y; }
}
```
Оба способа равноценны.

## 1.3 Соответствие методов делегату

Как было написано выше, методы соответствуют делегату, если они имеют один и тот же возвращаемый тип и один и тот же набор параметров. Но надо учитывать, что во внимание также принимаются модификаторы ref и out. Например, пусть у нас есть делегат:
```csharp
delegate void SomeDel(int a, double b);
```

Этому делегату соответствует, например, следующий метод:
```csharp
void SomeMethod1(int g, double n) { }
```

А следующие методы НЕ соответствуют:
```csharp
int SomeMethod2(int g, double n) { }
void SomeMethod3(double n, int g) { }
void SomeMethod4(ref int g, double n) { }
void SomeMethod5(out int g, double n) { g = 6; }
```

## 1.4 Добавление методов в делегат

В примерах выше переменная делегата указывала на один метод. В реальности же делегат может указывать на множество методов, которые имеют ту же сигнатуру и возвращаемые тип. Все методы в делегате попадают в специальный список - список вызова или invocation list. И при вызове делегата все методы из этого списка последовательно вызываются. И мы можем добавлять в этот спиок не один, а несколько методов:
```csharp
class Program
{
    delegate void Message();
 
    static void Main(string[] args)
    {
        Message mes1 = Hello;
        mes1 += HowAreYou;  // теперь mes1 указывает на два метода
        mes1(); // вызываются оба метода - Hello и HowAreYou
        Console.Read();
    }
    private static void Hello()
    {
        Console.WriteLine("Hello");
    }
    private static void HowAreYou()
    {
        Console.WriteLine("How are you?");
    }
}
```

В данном случае в список вызова делегата mes1 добавляются два метода - Hello и HowAreYou. И при вызове mes1 вызываются сразу оба этих метода.

Для добавления делегатов применяется операция +=. Однако стоит отметить, что в реальности будет происходить создание нового объекта делегата, который получит методы старой копии делегата и новый метод, и новый созданный объект делеагата будет присвоен переменной mes1.

При добавлении делегатов следует учитывать, что мы можем добавить ссылку на один и тот же метод несколько раз, и в списке вызова делегата тогда будет несколько ссылок на один и то же метод. Соответственно при вызове делегата добавленный метод будет вызываться столько раз, сколько он был добавлен:

```csharp
Message mes1 = Hello;
mes1 += HowAreYou;
mes1 += Hello;
mes1 += Hello;
 
mes1();

//Консольный вывод:
// Hello
// How are you?
// Hello
// Hello
```

Подобным образом мы можем удалять методы из делегата с помощью операции -=:

```csharp
static void Main(string[] args)
{
    Message mes1 = Hello;
    mes1 += HowAreYou;
    Message mes2 = HowAreYou;
    Message mes3 = mes1 + mes2;
    mes1(); // вызываются все методы из mes1 и mes2
    mes1 -= HowAreYou;  // удаляем метод HowAreYou
    mes1(); // вызывается метод Hello
     
    Console.Read();
}
```

При удалении методов из делегата фактически будет создаватья новый делегат, который в списке вызова методов будет содержать на один метод меньше.

При удалении следует учитывать, что если делегат содержит несколько ссылок на один и тот же метод, то операция -= начинает поиск с конца списка вызова делегата и удаляет только первое найденное вхождение. Если подобного метода в списке вызова делегата нет, то операция -= не имеет никакого эффекта.


## 1.5 Объединение делегатов

Делегаты можно объединять в другие делегаты. Например:

```csharp
class Program
{
    delegate void Message();
 
    static void Main(string[] args)
    {
        Message mes1 = Hello;
        Message mes2 = HowAreYou;
        Message mes3 = mes1 + mes2; // объединяем делегаты
        mes3(); // вызываются все методы из mes1 и mes2
         
        Console.Read();
    }
    private static void Hello()
    {
        Console.WriteLine("Hello");
    }
    private static void HowAreYou()
    {
        Console.WriteLine("How are you?");
    }
}
```

В данном случае объект mes3 представляет объединение делегатов mes1 и mes2. Объединение делегатов значит, что в список вызова делегата mes3 попадут все методы из делегатов mes1 и mes2. И при вызове делегата mes3 все эти методы одновременно будут вызваны.


## 1.6 Вызов делегата

В примерах выше делегат вызывался как обычный метод. Если делегат принимал параметры, то при ее вызове для параметров передавались необходимые значения:
```csharp
class Program
{
    delegate int Operation(int x, int y);
    delegate void Message();
 
    static void Main(string[] args)
    {
        Message mes = Hello;
        mes();
        Operation op = Add;
        op(3, 4);
        Console.Read();
    }
    private static void Hello() { Console.WriteLine("Hello"); }
    private static int Add(int x, int y) { return x + y; }
}
```

Другой способ вызова делегата представляет метод Invoke():
```csharp
class Program
{
    delegate int Operation(int x, int y);
    delegate void Message();
 
    static void Main(string[] args)
    {
        Message mes = Hello;
        mes.Invoke();
        Operation op = Add;
        op.Invoke(3, 4);
        Console.Read();
    }
    private static void Hello() { Console.WriteLine("Hello"); }
    private static int Add(int x, int y) { return x + y; }
}
```

Если делегат принимает параметры, то в метод Invoke передаются значения для этих параметров.

Следует учитывать, что если делегат пуст, то есть в его списке вызова нет ссылок ни на один из методов (то есть делегат равен Null), то при вызове такого делегата мы получим исключение, как, например, в следующем случае:

```csharp
Message mes = null;
//mes();        // ! Ошибка: делегат равен null
 
Operation op = Add;
op -= Add;      // делегат op пуст
op(3, 4);       // !Ошибка: делегат равен null
```
Поэтому при вызове делегата всегда лучше проверять, не равен ли он null. Либо можно использовать метод Invoke и оператор условного null:
```csharp
Message mes = null;
mes?.Invoke();        // ошибки нет, делегат просто не вызывается
 
Operation op = Add;
op -= Add;          // делегат op пуст
op?.Invoke(3, 4);   // ошибки нет, делегат просто не вызывается
```

Если делегат возвращает некоторое значение, то возвращается значение последнего метода из списка вызова (если в списке вызова несколько методов). Например:

```csharp
class Program
{
    delegate int Operation(int x, int y);
     
    static void Main(string[] args)
    {
        Operation op = Subtract;
        op += Multiply;
        op += Add;
        Console.WriteLine(op(7, 2));    // Add(7,2) = 9
        Console.Read();
    }
    private static int Add(int x, int y) { return x + y; }
    private static int Subtract(int x, int y) { return x - y; }
    private static int Multiply(int x, int y) { return x * y; }
}
```

## 1.7 Делегаты как параметры методов

Также делегаты могут быть параметрами методов:
```csharp
class Program
{
    delegate void GetMessage();
 
    static void Main(string[] args)
    {
        if (DateTime.Now.Hour < 12)
        {
            Show_Message(GoodMorning);
        }
        else
        {
             Show_Message(GoodEvening);
        }
        Console.ReadLine();
    }
    private static void Show_Message(GetMessage _del)
    {
        _del?.Invoke();
    }
    private static void GoodMorning()
    {
        Console.WriteLine("Good Morning");
    }
    private static void GoodEvening()
    {
        Console.WriteLine("Good Evening");
    }
}
```

## 1.8 Обобщенные делегаты

Делегаты могут быть обобщенными, например:

```csharp
delegate T Operation<T, K>(K val);
 
class Program
{
    static void Main(string[] args)
    {
        Operation<decimal, int> op = Square;
 
        Console.WriteLine(op(5));
        Console.Read();
    }
 
    static decimal Square(int n)
    {
        return n * n;
    }
}
```

# События

События сигнализируют системе о том, что произошло определенное действие.
События объявляются в классе с помощью ключевого слова event, после которого идет название делегата:

```csharp
// Объявляем делегат
public delegate void AccountStateHandler(string message);
// Событие, возникающее при выводе денег
public event AccountStateHandler Withdrawn;
```

Связь с делегатом означает, что метод, обрабатывающий данное событие, должен принимать те же параметры, что и делегат, и возвращать тот же тип, что и делегат.
Итак, посмотрим на примере. Для этого возьмем класс Account из прошлой темы и изменим его следующим образом:

```csharp
class Account
{
    // Объявляем делегат
    public delegate void AccountStateHandler(string message);
    // Событие, возникающее при выводе денег
    public event AccountStateHandler Withdrawn;
    // Событие, возникающее при добавление на счет
    public event AccountStateHandler Added;
 
    int _sum; // Переменная для хранения суммы
 
    public Account(int sum)
    {
        _sum = sum;
    }
 
    public int CurrentSum
    {
        get { return _sum; }
    }
 
    public void Put(int sum)
    {
        _sum += sum;
        if (Added != null)
            Added($"На счет поступило {sum}");
    }
    public void Withdraw(int sum)
    {
        if (sum <= _sum)
        {
            _sum -= sum;
            if (Withdrawn != null)
                Withdrawn($"Сумма {sum} снята со счета");
        }
        else
        {
            if (Withdrawn != null)
                Withdrawn("Недостаточно денег на счете");
        }
    }
}
```

Здесь мы определили два события: Withdrawn и Added. Оба события объявлены как экземпляры делегата AccountStateHandler, поэтому для обработки этих событий потребуется метод, принимающий строку в качестве параметра.

Затем в методах Put и Withdraw мы вызываем эти события. Перед вызовом мы проверяем, закреплены ли за этими событиями обработчики (if (Withdrawn != null)). Так как эти события представляют делегат AccountStateHandler, принимающий в качестве параметра строку, то и при вызове событий мы передаем в них строку.

Теперь используем события в основной программе:

```csharp
class Program
{
    static void Main(string[] args)
    {
        Account account = new Account(200);
        // Добавляем обработчики события
        account.Added += Show_Message;
        account.Withdrawn += Show_Message;
 
        account.Withdraw(100);
        // Удаляем обработчик события
        account.Withdrawn -= Show_Message;
 
        account.Withdraw(50);
        account.Put(150);
 
        Console.ReadLine();
    }
    private static void Show_Message(string message)
    {
        Console.WriteLine(message);
    }
} 
```

Для прикрепления обработчика события к определенному событию используется операция += и соответственно для открепления - операция -=: событие += метод_обработчика_события. Опять же обращаю внимание, что метод обработчика должен иметь такие же параметры, как и делегат события, и возвращать тот же тип. В итоге мы получим следующий консольный вывод:

```csharp
// Сумма 100 снята со счета
// На счет поступило 150
```

Кроме использованного выше способа прикрепления обработчиков есть и другой с использованием делегата. Но оба способа будут равноценны:

```csharp
account.Added += Show_Message;
account.Added += new Account.AccountStateHandler(Show_Message);
```

## 1.1 Класс данных события AccountEventArgs

Нередко при возникновении события обработчику события требуется передать некоторую информацию о событии. Например, добавим и в нашу программу новый класс AccountEventArgs со следующим кодом:

```csharp
class AccountEventArgs
{
    // Сообщение
    public string Message{get;}
    // Сумма, на которую изменился счет
    public int Sum {get;}
 
    public AccountEventArgs(string mes, int sum)
    {
        Message = mes;
        Sum = sum;
    }
}
```

Данный класс имеет два свойства: Message - для хранения выводимого сообщения и Sum - для хранения суммы, на которую изменился счет.
Теперь применим класс AccoutEventArgs, изменив класс Account следующим образом:

```csharp
class Account
{
    // Объявляем делегат
    public delegate void AccountStateHandler(object sender, AccountEventArgs e);
    // Событие, возникающее при выводе денег
    public event AccountStateHandler Withdrawn;
    // Событие, возникающее при добавлении на счет
    public event AccountStateHandler Added;
 
    int _sum; // Переменная для хранения суммы
 
    public Account(int sum)
    {
        _sum = sum;
    }
 
    public int CurrentSum
    {
        get { return _sum; }
    }
 
    public void Put(int sum)
    {
        _sum += sum;
        if (Added != null)
            Added(this, new AccountEventArgs($"На счет поступило {sum}", sum));
    }
    public void Withdraw(int sum)
    {
        if (_sum >= sum)
        {
            _sum -= sum;
            if (Withdrawn != null)
                Withdrawn(this, new AccountEventArgs($"Сумма {sum} снята со счета", sum));
        }
        else
        {
            if (Withdrawn != null)
                Withdrawn(this, new AccountEventArgs("Недостаточно денег на счете", sum));
        }
    }
}
```

По сравнению с предыдущей версией класса Account здесь изменилось только количество параметров у делегата и соответственно количество параметров при вызове события. Теперь они также принимают объект AccountEventArgs, который хранит информацию о событии, получаемую через конструктор.
Теперь изменим основную программу:

```csharp
class Program
{
    static void Main(string[] args)
    {
        Account account = new Account(200);
        // Добавляем обработчики события
        account.Added += Show_Message;
        account.Withdrawn += Show_Message;
 
        account.Withdraw(100);
        // Удаляем обработчик события
        account.Withdrawn -= Show_Message;
 
        account.Withdraw(50);
        account.Put(150);
 
        Console.ReadLine();
    }
    private static void Show_Message(object sender, AccountEventArgs e)
    {
        Console.WriteLine($"Сумма транзакции: {e.Sum}");
        Console.WriteLine(e.Message);
    }
} 
```
По сравнению с предыдущим вариантом здесь мы только изменяем количество параметров и сущность их использования в обработчике Show_Message.


# 3 Action, Predicate и Func

В .NET есть несколько встроенных делегатов, которые используются в различных ситуациях. И наиболее используемыми, с которыми часто приходится сталкиваться, являются Action, Predicate и Func.

## 3.1 Action

Делегат Action является обобщенным, принимает параметры и возвращает значение void:
```csharp
public delegate void Action<T>(T obj)
```

Данный делегат имеет ряд перегруженных версий. Каждая версия принимает разное число параметров: от Action<in T1> до Action<in T1, in T2,....in T16>. Таким образом можно передать до 16 значений в метод.
Как правило, этот делегат передается в качестве параметра метода и предусматривает вызов определенных действий в ответ на произошедшие действия. Например:

```csharp
static void Main(string[] args) 
{
    Action<int, int> op;
    op = Add;
    Operation(10, 6, op);
    op = Substract;
    Operation(10, 6, op);
 
    Console.Read();
}
 
static void Operation(int x1, int x2, Action<int, int> op)
{
    if (x1 > x2)
        op(x1, x2);
}
 
static void Add(int x1, int x2)
{
    Console.WriteLine("Сумма чисел: " + (x1 + x2));
}
 
static void Substract(int x1, int x2)
{
    Console.WriteLine("Разность чисел: " + (x1 - x2));
}
```

## 3.2 Predicate

Делегат Predicate<T>, как правило, используется для сравнения, сопоставления некоторого объекта T определенному условию. В качестве выходного результата возвращается значение true, если условие соблюдено, и false, если не соблюдено:

```csharp
Predicate<int> isPositive = delegate (int x) { return x > 0; };
 
Console.WriteLine(isPositive(20));
Console.WriteLine(isPositive(-20));
```
В данном случае возвращается true или false в зависимости от того, больше нуля число или нет.

## 3.3 Func

Еще одним распространенным делегатом является Func. Он возвращает результат действия и может принимать параметры. Он также имеет различные формы: от Func<out T>(), где T - тип возвращаемого значения, до Func<in T1, in T2,...in T16, out TResult>(), то есть может принимать до 16 параметров.

```csharp
TResult Func<out TResult>()
TResult Func<in T, out TResult>(T arg)
TResult Func<in T1, in T2, out TResult>(T1 arg1, T2 arg2)
TResult Func<in T1, in T2, in T3, out TResult>(T1 arg1, T2 arg2, T3 arg3)
TResult Func<in T1, in T2, in T3, in T4, out TResult>(T1 arg1, T2 arg2, T3 arg3, T4 arg4)
//...........................................
```

Данный делегат также часто используется в качестве параметра в методах:

```csharp
static void Main(string[] args) 
{
    Func<int, int> retFunc = Factorial;
    int n1 = GetInt(6, retFunc);
    Console.WriteLine(n1);  // 720
     
    int n2 = GetInt(6, x=> x *x);
    Console.WriteLine(n2); // 36
     
    Console.Read();
}
 
static int GetInt(int x1, Func<int, int> retF)
{
    int result = 0;
    if (x1 > 0)
        result = retF(x1);
    return result;
}
static int Factorial(int x)
{
    int result = 1;
    for (int i = 1; i <= x; i++)
    {
        result *= i;
    }
    return result;
}
```

Метод GetInt() в качестве параметра принимает делегат Func<int, int>, то есть ссылку на метод, который принимает число int и возвращает также значение int.

При первом вызове метода GetInt() ему передается ссылка на метод вычисления факториала. Во втором случае передается лямбда-выражение x=> x*x, то есть опять же выражение, которое принимает параметр int x и возвращает результат x * x.