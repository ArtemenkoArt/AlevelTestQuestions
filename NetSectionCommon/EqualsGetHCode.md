# Сравнение объектов в C#.NET

## Полезные ссылки

[Сравнение объектов в C#.NET](https://habr.com/ru/post/137680/)
[Переопределение Equals и GetHashCode. А оно надо?](https://habr.com/ru/company/microsoft/blog/418515/)


C#.NET предлагает множество способов сравнить объекты, как экземпляры классов, так и структур. 
Способов так много, что без упорядочения этих способов и понимания их грамотного использования и
имплементации (при наличии возможности переопределения), в голове, неминуемо, образуется каша.

**Итак, класс System.Object предлагает следующие методы**:

- 
```csharp  
public static bool ReferenceEquals(object objA, object objB)
{
    return objA == objB;
}
```


- 
```csharp
public static bool Equals(object objA, object objB)
{
    return objA == objB || (objA != null && objB != null && objA.Equals(objB));
}
```

- 
```csharp
public virtual bool Equals(object obj)
{
    return RuntimeHelpers.Equals(this, obj);
}
```

- 
```csharp
public static bool operator == (Foo left, Foo right);
```
Также имеется возможность наследования IEquatable, IStructuralEquatable.

## ReferenceEquals

Метод ReferenceEquals сравнивает две ссылки. Если ссылки на объекты идентичны, то возвращает true. Это значит, что данный метод проверяет экземпляры не на равенство, а на тождество. В случае передачи этому методу экземпляров значимого типа (даже если передать один и тот же экземпляр) всегда будет возвращать false. Так произойдёт потому, что при передаче произойдёт упаковка значимых типов и ссылки на них будут разные.
Здесь также хотелось бы упомянуть о сравнение двух строк этим методом. Например:
```csharp
class Program
    {
        static void Main(string[] args)
        {
            string a = "Hello";
            string b = "Hello";

            if(object.ReferenceEquals(a,b))
                Console.WriteLine("Same objects");
            else
                Console.WriteLine("Not the same objects");

            Console.ReadLine();
        }
    }
``` 
 
 Такая программа, запросто может вывести «Same objects». Не стоит переживать, это связано с интернированием строк. Но это совсем другая история и здесь об этом речи идти не будет.
 
## public static bool Equals(object objA, object objB)

Сначала этот метод проверяет экземпляры на тождество и если объекты не тождественны, то проверяет их на null и делегирует ответственность за компарацию переопределяемому экземплярному методу Equals.

## public virtual bool Equals(object obj)

По умолчанию, этот метод ведёт себя точно также как ReferenceEquals. Однако для значимых типов он переопределён и в System.ValueType выглядит следующим образом:
```csharp
public override bool Equals(object obj)
		{
			if (obj == null)
			{
				return false;
			}
			RuntimeType runtimeType = (RuntimeType)base.GetType();
			RuntimeType left = (RuntimeType)obj.GetType();
			if (left != runtimeType)
			{
				return false;
			}
			if (ValueType.CanCompareBits(this))
			{
				return ValueType.FastEqualsCheck(this, obj);
			}
			FieldInfo[] fields = runtimeType.GetFields(BindingFlags.Instance | BindingFlags.Public | BindingFlags.NonPublic);
			for (int i = 0; i < fields.Length; i++)
			{
				object obj2 = ((RtFieldInfo)fields[i]).InternalGetValue(this, false);
				object obj3 = ((RtFieldInfo)fields[i]).InternalGetValue(obj, false);
				if (obj2 == null)
				{
					if (obj3 != null)
					{
						return false;
					}
				}
				else
				{
					if (!obj2.Equals(obj3))
					{
						return false;
					}
				}
			}
			return true;
		}
``` 
Выход из этой ситуации - реализация интерфейса **IEquatable** и переопределение метода **public override bool Equals**

## public static bool operator == (Foo left, Foo right)

Для значимых типов всегда следует переопределять, как и виртуальный Equals(). Для ссылочных типов лучше не переопределять, ибо, по умолчанию, от == на ссылочных типах ожидается поведение как у метода ReferenceEquals(). Так что, здесь всё просто.

## public virtual int GetHashCode()
В общем и целом, стандартная реализация этого метода ведёт себя как генератор уникального идентификатора. Минус такого подхода состоит в том, что одинаковые семантически объекты, могут возвращать разные hash-значения. Рихтер жалуется на то, что стандартная реализация ещё и низкопроизводительна. Грамотная реализация этого метода весьма проблематична. Необходимо высчитывать hash быстро и иметь большой разброс в результате, чтобы не случалось повторений на достаточно больших множествах. На самом деле, в большей части случаев, имплементации GetHashCode() донельзя простые. Везде производятся сдвиги, «побитовые или», или «исключающие или». Сам Рихтер приводит пример со структурой, имеющей два поля типа int. GetHashCode() он предлагает имплементировать примерно так:

```csharp
internal sealed class Point
    {
        private int a;
        private int b;

        public override int  GetHashCode()
        {
            return a ^ b;
        }
    }
```


```csharp
public override int GetHashCode()
		{
			return (int)this | (int)this << 16;
		}
```

**Всегда необходимо переопределять Equals, а также GetHashCode, чтобы избежать снижения производительности**

## Почему стандартные версии работают медленно?

Авторы CLR изо всех сил старались сделать стандартные версии Equals и GetHashCode максимально эффективными для типов значений. Но есть несколько причин, по которым эти методы проигрывают в эффективности пользовательской версии, написанной для определенного типа вручную (или сгенерированной компилятором).

1. Распределение упаковки-преобразования. CLR создан таким образом, что каждый вызов элемента, определенного в типах System.ValueType или System.Enum, запускает упаковку-преобразование .

2. Потенциальные конфликты в стандартной версии метода GetHashCode. При реализации хэш-функции мы сталкиваемся с дилеммой: сделать распределение хэш-функции хорошо или быстро. В ряде случаев можно выполнить и то и другое, но в типе ValueType.GetHashCode это обычно сопряжено с трудностями.

Традиционная хэш-функция типа struct «объединяет» хэш-коды всех полей. Но единственный способ получить хэш-код поля в методе ValueType — использовать рефлексию. Вот почему авторы CLR решили пожертвовать скоростью ради распределения, и стандартная версия GetHashCode только возвращает хэш-код первого ненулевого поля.


```csharp
public readonly struct Location
{
    public string Path { get; }
    public int Position { get; }
    public Location(string path, int position) => (Path, Position) = (path, position);
}
 

var hash1 = new Location(path: "", position: 42).GetHashCode();
var hash2 = new Location(path: "", position: 1).GetHashCode();
var hash3 = new Location(path: "1", position: 42).GetHashCode();
// hash1 and hash2 are the same and hash1 is different from hash3
```