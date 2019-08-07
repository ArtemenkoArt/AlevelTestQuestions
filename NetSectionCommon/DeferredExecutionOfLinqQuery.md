# Deferred Execution of LINQ Query

## Отложенное выполнение запроса LINQ

Отложенное выполнение означает, что выполнение выражения задерживается до тех пор, пока его фактическое значение, фактически не требуется. Это значительно повышает производительность, избегая ненужного выполнения.
Отложенное выполнение применимо к любой коллекции в памяти, а также к LINQ, таким как LINQ-to-SQL, LINQ-to-Entities, LINQ-to-XML и т.д.
Разберемся с отложенным выполнением, используя следующий пример:

```csharp 
public class Emploee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
}

class Program
{
    static void Main(string[] args)
    {
        IList<Emploee> emploeeList = new List<Emploee>()
        {
            new Emploee() { Id = 1, Age = 50 , Name = "Ivan Petukhov"} ,
            new Emploee() { Id = 2, Age = 25 , Name = "Petr Vorobyov"} ,
            new Emploee() { Id = 3, Age = 43 , Name = "Svetlana Dyatlova"} ,
            new Emploee() { Id = 4, Age = 68 , Name = "Grigory Ptentsov"} ,
            new Emploee() { Id = 5, Age = 70 , Name = "Irina Golubeva"}
        };

        var teenAgerEmploee = from s in emploeeList // <--- Запрос НЕ выполняется здесь
                                where s.Age > 18 && s.Age < 60
                                select s;

        foreach (Emploee teenEmploee in teenAgerEmploee) // <--- Запрос выполняется ЗДЕСЬ
        {
            Console.WriteLine("Emploee Name: {0}", teenEmploee.Name);
        }
        Console.ReadKey();
    }
}
```
Результат:

    Emploee Name: Ivan Petukhov
    Emploee Name: Petr Vorobyov
    Emploee Name: Svetlana Dyatlova

В приведенном выше примере можето видеть, что запрос выполняется, когда выполняется итерация с использованием цикла foreach. Это называется отложенным исполнением. LINQ обрабатывает коллекцию emploeeList, когда фактически получает доступ к каждому объекту из коллекции и что-то с ним делает (в данном случае, выводит в консоль имя).

### Отложенное выполнение возвращает последние данные
Чтобы проверить, возвращает ли отложенное выполнение самые последние данные каждый раз, добавим еще одного сотрудника после цикла foreach и проверим список сотрудников:

```csharp 
IList<Emploee> emploeeList = new List<Emploee>()
    {
        new Emploee() { Id = 1, Age = 50 , Name = "Ivan Petukhov"} ,
        new Emploee() { Id = 2, Age = 25 , Name = "Petr Vorobyov"} ,
        new Emploee() { Id = 3, Age = 43 , Name = "Svetlana Dyatlova"} ,
        new Emploee() { Id = 4, Age = 68 , Name = "Grigory Ptentsov"} ,
        new Emploee() { Id = 5, Age = 70 , Name = "Irina Golubeva"}
    };

var nonRetiredList = from s in emploeeList
                        where s.Age > 18 && s.Age < 60
                        select s;

foreach (Emploee emploee in nonRetiredList) // <--- Запрос выполняется ЗДЕСЬ
{
    Console.WriteLine("Emploee v.1 Name: {0}", emploee.Name);
}

Console.WriteLine(new String('-',30));

emploeeList.Add(new Emploee { Id = 6, Age = 19, Name = "Vasya Pupkin" }); // <<--- Добавляем нового сотрудника

foreach (Emploee teenEmploee in nonRetiredList) // <--- Запрос выполняется ЗДЕСЬ, и возвращает обновленные данные
{
    Console.WriteLine("Emploee v.2 Name: {0}", teenEmploee.Name);
}
```

Результат:

    Emploee v.1 Name: Ivan Petukhov
    Emploee v.1 Name: Petr Vorobyov
    Emploee v.1 Name: Svetlana Dyatlova
    ------------------------------
    Emploee v.2 Name: Ivan Petukhov
    Emploee v.2 Name: Petr Vorobyov
    Emploee v.2 Name: Svetlana Dyatlova
    Emploee v.2 Name: Vasya Pupkin

Как видно, второй цикл снова выполняет запрос и возвращает самые последние данные. Отсроченное выполнение переоценивает каждое выполнение; это называется ленивой оценкой. Это одно из главных преимуществ отложенного выполнения: оно всегда дает вам самые последние данные.
Так как сама переменная запроса никогда не содержит результатов запроса, ее можно выполнять так часто, как необходимо. Например, если база данных непрерывно обновляется отдельным приложением. В приложении можно создать один запрос, получающий последние данные, и его можно выполнять повторно с некоторым интервалом для извлечения каждый раз разных результатов.

## Принудительное немедленное выполнение

Запросы, выполняющие статистические функции над диапазоном исходных элементов, должны сначала выполнить итерацию этих элементов. Примерами таких запросов являются Count, Max, Average и First. Они выполняются без явного оператора foreach, поскольку сам запрос должен использовать foreach для возвращения результата. Нужно обратить внимание, что такой тип запросов возвращает одиночное значение, а не коллекцию IEnumerable.
Чтобы принудительно вызвать немедленное выполнение любого запроса и кэшировать его результаты, можно вызвать метод ToList<TSource>илиToArray<TSource>.
Можно также принудительно выполнить запрос, поместив цикл foreach сразу после выражения запроса. Однако вызов ToListилиToArrayтакже кэширует все данные в одной коллекции объектов.

## Реализация отложенного выполнения

Можно реализовать отложенное выполнение для своих пользовательских методов расширения для IEnumerable, используя ключевое слово yield в C #.
Например, вы можете реализовать пользовательский метод расширения GetNonRetiredEmploee для коллекции IEnumerable, который возвращает список всех сотрудников, соответствующих параметрам отбора.

```csharp 
public static class EnumerableExtensionMethods
{
    public static IEnumerable<Emploee> GetNonRetiredEmploee(this IEnumerable<Emploee> source)
    {
        foreach (Emploee emploee in source)
        {
            Console.WriteLine($"Processed emploee: {emploee.Name}");

            if (emploee.Age > 18 && emploee.Age < 65)
            {
                yield return emploee;
            }
        }
    }
}
```
Обратите внимание, что мы выводим имя сотрудника в консоль всякий раз, когда вызывается GetNonRetiredEmploee().
Теперь можно использовать этот метод расширения, как показано ниже:

```csharp 
IList<Emploee> emploeeList = new List<Emploee>()
    {
        new Emploee() { Id = 1, Age = 50 , Name = "Ivan Petukhov"} ,
        new Emploee() { Id = 2, Age = 25 , Name = "Petr Vorobyov"} ,
        new Emploee() { Id = 3, Age = 43 , Name = "Svetlana Dyatlova"} ,
        new Emploee() { Id = 4, Age = 68 , Name = "Grigory Ptentsov"} ,
        new Emploee() { Id = 5, Age = 70 , Name = "Irina Golubeva"}
    };

var nonRetiredList = from s in emploeeList.GetNonRetiredEmploee() // <--- Запрос НЕ выполняется здесь
                    select s;

foreach (Emploee emploee in nonRetiredList) // <--- Запрос выполняется ЗДЕСЬ
{
    Console.WriteLine("Emploee Name: {0}", emploee.Name);
}
```
Результат:

    Processed emploee: Ivan Petukhov
    Emploee Name: Ivan Petukhov
    Processed emploee: Petr Vorobyov
    Emploee Name: Petr Vorobyov
    Processed emploee: Svetlana Dyatlova
    Emploee Name: Svetlana Dyatlova
    Processed emploee: Grigory Ptentsov
    Processed emploee: Irina Golubeva

Как можето видеть из выходных данных, GetNonRetiredEmploee() вызывается, когда перебирается nonRetiredList с помощью цикла foreach.
Таким образом, можно создавать собственные методы, используя ключевое слово yield, чтобы получить преимущество отложенного выполнения.
