# Principles of OOP

**Инкапсуляция**, **наследование** и **полиморфизм** обычно задаются в качестве трех основных принципов объектно-ориентированных языков (OOП) и объектно-ориентированной методологии. Эти принципы в некоторой степени зависят от типа языка.

## Инкапсуляция

Есть два важных аспекта инкапсуляции:

- Ограничение доступа - например, предотвращение доступа одного объекта к внутреннему состоянию другого.
- Пространства имен / области действия - позволяют одному и тому же имени иметь разные значения в разных контекстах.

Инкапсуляция необходима для уменьшения связей между программными компонентами!!!

```csharp 
    public class Employee
    {
        private string _name;
        private int _age;

        public Employee()
        {
            _name = "John";
            _age = 14;
        }

        public void AboutMe()
        {
            Console.WriteLine($"My name is {_name} and i'm {_age} years old");
        }
    }
```

## Наследование 

ООП вводит концепцию наследования, согласно которой специализированные классы - без дополнительного кода - могут «копировать» атрибуты и поведение исходных классов, которые они специализируют. Если некоторые из этих атрибутов или поведения необходимо изменить, вы переопределяете их. Единственный исходный код, который вы изменяете, - это код, необходимый для создания специализированных классов. Исходный объект называется родительским, а новая специализация называется дочерним

```csharp 
    public class Person
    {
        protected string Name;
        protected int Age;

        public Person()
        {
            Name = "John";
            Age = 54;
        }

        public virtual void AboutMe()
        {
            Console.WriteLine($"My name is {Name} and i'm {Age} years old");
        }
    }

    public class Employee : Person
    {
        private double _salary;

        public Employee()
        {
            _salary = 1000.32;
        }

        public override void AboutMe()
        {
            base.AboutMe();
            Console.WriteLine($"I got a job and earns {_salary}, when i'm {Age} years old");
        }
    }
```

## Полиморфизм

Скажем, у нас есть родительский класс и несколько дочерних классов, которые наследуются от него. Иногда мы хотим использовать коллекцию - например, список - который содержит смесь всех этих классов. Или у нас есть метод, реализованный для родительского класса, но мы хотели бы использовать его и для детей.

Это может быть решено с помощью полиморфизма.

Проще говоря, полиморфизм дает возможность использовать класс точно так же, как и его родительский, поэтому нет путаницы с типами. Но каждый дочерний класс сохраняет свои собственные методы такими, какие они есть.

```csharp 
    public  class Person
    {

        protected string Name;
        protected int Age;

        public Person(string name, int age)
        {
            Name = name;
            Age = age;
        }

        public virtual void AboutMe()
        {
            Console.WriteLine($"My name is {Name} and i'm {Age} years old");
        }
    }

    public class Employee : Person
    {
        private double _salary;

        public Employee(string name, int age): base(name, age)
        {
            _salary = 1000.32;
        }

        public override void AboutMe()
        {
            base.AboutMe();
            Console.WriteLine($"I got a job and earns {_salary}, when i'm {Age} years old");
        }
    }

    public class Student : Person
    {
        private double _averageMark;

        public Student(string name, int age) : base(name, age)
        {
            _averageMark = 78.21;
        }

        public override void AboutMe()
        {
            base.AboutMe();
            Console.WriteLine($"I am a student with average mark {_averageMark}");
        }
    }
```

И теперь мы можем сделать так:


```csharp 
    Person employee = new Employee("John", 41);
    Person student = new Student("Alex", 18);

    List<Person> people = new List<Person>();

    people.Add(employee);
    people.Add(student);

    people.ForEach(x => x.AboutMe());
```
