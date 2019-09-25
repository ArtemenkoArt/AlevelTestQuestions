# Generic vs Non-generic

## Полезные ссылки

[Обобщенные типы и зачем они нужны](https://metanit.com/sharp/tutorial/3.12.php)

## В чем разница: generic and non-generic collection in C#?

## Generic Collection 

* **Классы Generic Collection находятся в System.Collections.Generics namespace.**

* **Generic Collection строго типизирована.**

* **Generic Collections хранят элементы внутри массивов их фактических типов.**

## Non-Generic Collection 

* **Классы Non-Generic Collection находятся в System.Collections namespace.**

* **Non-Generic Collection не является строго типизирована.**

* **Non-Generic Collection хранят элементы внутри массивов объектов, в них могут храниться данные любого типа.**

Generic Collections:List<T>, Dictionary<TKey,TValue>, SortedList<TKey,TValue>, Hashset<T>, Queue<T>, Stack<T> 

Non-Generic Collections: ArrayList, SortedList, Stack, Queue, Hashtable, BitArray

Пример Generic Collection:

```csharp  
var list = new List<int>();

list.Add(10);
list.Add(20);
list.Add(30);

list.ForEach(i => Console.WriteLine(i));
```

Пример Non-Generic Collection:

```csharp 
var list = new ArrayList();

list.Add(10);
list.Add("sad");
list.Add(true);

foreach(var i in list)
    Console.WriteLine(i);
```
## Обобщенные методы

Пример:
```csharp
        static void Main(string[] args)
        {


            int x = 7;
            int y = 25;
            Swap<int>(ref x, ref y);
            Console.WriteLine($"x={x}    y={y}");   // x=25   y=7

            string s1 = "hello";
            string s2 = "bye";
            Swap<string>(ref s1, ref s2);
            Console.WriteLine($"s1={s1}    s2={s2}"); // s1=bye   s2=hello

        }

        public static void Swap<T>(ref T x, ref T y)
        {
            T temp = x;
            x = y;
            y = temp;
        }
```

## Почему лучше работать с Generic Collections?
В случае работы с Non-Generic Collections мы сталкиваемся с такими явлениями как упаковка (boxing) и распаковка (unboxing).
Упаковка (boxing) предполагает преобразование объекта значимого типа (например, типа int) к типу object. При упаковке общеязыковая среда CLR обертывает значение в объект типа System.Object и сохраняет его в управляемой куче (хипе). Распаковка (unboxing), наоборот, предполагает преобразование объекта типа object к значимому типу. Упаковка и распаковка ведут к снижению производительности, так как системе надо осуществить необходимые преобразования.