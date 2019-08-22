# IEnumerable Vs IQueryable

## Полезные ссылки

[What is the difference between IQueryable<T> and IEnumerable<T>?](https://stackoverflow.com/questions/252785/what-is-the-difference-between-iqueryablet-and-ienumerablet)<br/>
[Интерфейсы IEnumerable и IEnumerator](https://metanit.com/sharp/tutorial/4.11.php)<br/>
[IEnumerable<T> и IQueryable<T>, в чем разница?](https://habr.com/ru/post/108407/)<br/>
[IEnumerable и IQueryable в Entity Framework](https://metanit.com/sharp/entityframework/1.4.php)<br/>
## IEnumerable

Как мы знаем, основной для большинства коллекций является реализация интерфейсов IEnumerable и IEnumerator. Благодаря такой реализации мы можем перебирать объекты в цикле foreach::

```csharp  
foreach(var item in перечислимый_объект)
{
     
}
```
Перебираемая коллекция должна реализовать интерфейс IEnumerable.

Интерфейс IEnumerable имеет метод, возвращающий ссылку на другой интерфейс - перечислитель:

```csharp  
public interface IEnumerable
{
    IEnumerator GetEnumerator();
}
```
А интерфейс IEnumerator определяет функционал для перебора внутренних объектов в контейнере:

 ```csharp  
public interface IEnumerator
{
    bool MoveNext(); // перемещение на одну позицию вперед в контейнере элементов
    object Current {get;}  // текущий элемент в контейнере
    void Reset(); // перемещение в начало контейнера
}
  ```
  Метод **MoveNext()** перемещает указатель на текущий элемент на следующую позицию в последовательности. Если последовательность еще не закончилась, то возвращает true. Если же последовательность закончилась, то возвращается false.

Свойство **Current** возвращает объект в последовательности, на который указывает указатель.

Метод **Reset()** сбрасывает указатель позиции в начальное положение.

### Реализация IEnumerable и IEnumerator

 ```csharp  
class Week
    {
        string[] days = { "Monday", "Tuesday", "Wednesday", "Thursday",
                            "Friday", "Saturday", "Sunday" };
 
        public IEnumerator<string> GetEnumerator()
        {
            return new WeekEnumerator(days);
        }
    }
    class WeekEnumerator : IEnumerator<string>
    {
        string[] days;
        int position = -1;
        public WeekEnumerator(string[] days)
        {
            this.days = days;
        }
         
        public string Current
        {
            get
            {
                if (position == -1 || position >= days.Length)
                    throw new InvalidOperationException();
                return days[position];
            }
        }
 
        object IEnumerator.Current => throw new NotImplementedException();
         
        public bool MoveNext()
        {
            if(position < days.Length - 1)
            {
                position++;
                return true;
            }
            else
                return false;
        }
 
        public void Reset()
        {
            position = -1;
        }
        public void Dispose() { }
    }
    class Program
    {
        static void Main(string[] args)
        {
            Week week = new Week();
            foreach(var day in week)
            {
                Console.WriteLine(day);
            }
            Console.Read();
        }
    }
  ```
## IEnumerable и IQueryable в Entity Framework
Методы расширений LINQ могут возвращать два объекта: **IEnumerable** и **IQueryable**. С одной стороны, интерфейс **IQueryable** наследуется от **IEnumerable**, поэтому по идее объект **IQueryable** это и есть также объект **IEnumerable**. Но реальность несколько сложнее. Между объектами этих интерфейсов есть разница в плане функциональности, поэтому они не взаимозаменяемы.

Интерфейс **IEnumerable** находится в пространстве имен **System.Collections**. Объект **IEnumerable** представляет набор данных в памяти и может перемещаться по этим данным только вперед. Запрос, представленный объектом **IEnumerable**, выполняется немедленно и полностью, поэтому получение данных приложением происходит быстро.

При выполнении запроса **IEnumerable** загружает все данные, и если нам надо выполнить их фильтрацию, то сама фильтрация происходит на стороне клиента.

Интерфейс IQueryable располагается в пространстве имен **System.Linq**. Объект **IQueryable** предоставляет удаленный доступ к базе данных и позволяет перемещаться по данным как в прямом порядке от начала до конца, так и в обратном порядке. В процессе создания запроса, возвращаемым объектом которого является **IQueryable**, происходит оптимизация запроса. В итоге в процессе его выполнения тратится меньше памяти, меньше пропускной способности сети, но в то же время он может обрабатываться чуть медленнее, чем запрос, возвращающий объект **IEnumerable**.

Для примера возьмем два вроде бы идентичных выражения. Объект **IEnumerable**:

```csharp  
IEnumerable<Phone> phoneIEnum = db.Phones;
var phones=phoneIEnum.Where(p => p.Id > id).ToList();
```
Здесь запрос будет иметь следующий вид:
```sql 
SELECT
    [Extent1].[Id] AS [Id], 
    [Extent1].[Name] AS [Name], 
    [Extent1].[Company] AS [Company] 
    FROM [dbo].[Phones] AS [Extent1]
```
Фильтрация результата, обозначенная с помощью метода Where(p => p.Id > id) будет идти уже после выборки из бд в самом приложении.

Чтобы совместить фильтры, нам надо было сразу применить метод **Where**: db.Phones.Where(p => p.Id > id);

Объект **IQueryable**:

```csharp  
IQueryable<Phone> phoneIQuer = db.Phones;
var phones=phoneIQuer.Where(p => p.Id > id).ToList();
```
Здесь запрос будет иметь следующий вид:
```sql 
SELECT
    [Extent1].[Id] AS [Id], 
    [Extent1].[Name] AS [Name], 
    [Extent1].[Company] AS [Company] 
    FROM [dbo].[Phones] AS [Extent1]
    WHERE [Extent1].[Id] >3
```
### Выводы:
Таким образом, все методы суммируются, запрос оптимизируется, и только потом происходит выборка из базы данных.

Что же лучше использовать? Все зависит от конкретной ситуации. Если разработчику нужен весь набор возвращаемых данных, то лучше использовать **IEnumerable**, предоставляющий максимальную скорость. Если же нам не нужен весь набор, а то только некоторые отфильтрованные данные, то лучше применять **IQueryable**.
