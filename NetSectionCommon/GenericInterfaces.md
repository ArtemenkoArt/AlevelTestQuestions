# Обобщенные интерфейсы

## Полезные ссылки

[Обобщенные интерфейсы](https://metanit.com/sharp/tutorial/3.40.php)
[Generic Interfaces](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-interfaces)

Как и классы, интерфейсы могут быть обобщенными:

```csharp  
interface IUser<T>
{
    T Id { get; }
}
class User<T> : IUser<T>
{
    T _id;
    public User(T id)
    {
        _id = id;
    }
    public T Id { get { return _id; } }
}
```
Интерфейс IUser типизирован параметром T, который при реализации интерфейса используется в классе User. В частности, переменная _id определена как T, что позволяет нам использовать для id различные типы.

Определим две реализации: одна в качестве параметра будет использовать тип int, а другая - тип string:

```csharp  
IUser<int> user1 = new User<int>(6789);
Console.WriteLine(user1.Id);    // 6789
 
IUser<string> user2 = new User<string>("12345");
Console.WriteLine(user2.Id);    // 12345
```
Также при реализации интерфейса мы можем явным образом указать, какой тип будет использоваться для параметра T:

 ```csharp  
class IntUser : IUser<int>
{
    int _id;
    public IntUser(int id)
    {
        _id = id;
    }
    public int Id { get { return _id; } }
}
  ```
  
