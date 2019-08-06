# What is the difference between string and String?

***

## В чём разница между string и String в C#?

Ответ на самом деле очень прост: string — это просто псевдоним (alias) для System.String 
т.е. технически, никакой разницы нет. Так же, как и нет разницы между int и System.Int32.

### Рекомендовал использовать string для переменных и String для вызовов методов:
Пример:
```csharp  
string name = "everyone";
string msg = String.Format("Hi, {0}!", name);
Console.WriteLine(msg);
Console.ReadKey();
```

В последних версиях рекомендация была изменена: 
рекомендуется всегда использовать алиас string, в том числе при вызове методов (string.Format()).

Ещё один нюанс: если в какой-нибудь библиотеке будет объявлен тип Foo.String, то использование String может привести к конфликтам, аиспользование string всегда будет однозначно.
Однако этот случай скорее невероятен, потому что объявление подобного типа запрещается Framework Design Guidelines и здравой логикой.