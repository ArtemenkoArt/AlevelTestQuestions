# What are an abstract class and interface and what is the difference in using?

***

## Абстрактные классы

Абстрактный класс в объектно-ориентированном программировании — это базовый класс, который не предполагает создания экземпляров через вызов конструктора на прямую, но экземпляр абстрактного класса создается не явно при построении экземпляра производного конкретного класса.
Ключевое слово abstractможет использоваться с классами, методами, свойствами, индексаторами и событиями

### Возможности и ограничения абстрактных классов:
* **Экземпляр абстрактного класса создать нельзя через вызов конструктора напрямую, но экземпляр абстрактного класса создается неявно при построении экземпляра производного конкретного класса.**
* **Абстрактные классы могут содержать как абстрактные,так и не абстрактные члены.**
* **Не абстрактный (конкретный) класс, являющийся производным от абстрактного, должен содержать фактические реализации всех наследуемых абстрактных членов.**

### Возможности абстрактных методов:
* **Абстрактный метод является неявным виртуальным методом.**
* **Создание абстрактных методов допускается только в абстрактных классах**
* **Тело абстрактного метода отсутствует, создание метода просто заканчивается двоеточием, а после сигнатуры не нужно ставить фигурные скобки ({ }).**
* **Реализация предоставляется методом переопределения override, который является членом неабстрактного класса.**

Пример, эволюция постирушек :)))
```csharp  
public abstract class AbstractWashingObject
    {
        public void Wash()
        {
            Console.WriteLine("Everything is washed!");
        }

        public virtual void Sound()
        {
            Console.WriteLine("Sound of work!");
        }

        public abstract void Stop();
    }

    class Mom : AbstractWashingObject
    {
        public override void Sound()
        {
            Console.WriteLine("Do you want to eat?");
        }

        public override void Stop()
        {
            Console.WriteLine("I did it!");
        }
    }

    class Himself : AbstractWashingObject
    {
        public new void Sound()
        {
            Console.WriteLine("OMG o_O");
        }

        public override void Stop()
        {
            Console.WriteLine("I finished!");
        }
    }

    class Wife : AbstractWashingObject
    {
        public override void Sound()
        {
            Console.WriteLine("Where is your salary?");
        }

        public override void Stop()
        {
            Console.WriteLine("Oh I forgot to wash some of the things...");
        }
    }

    class WasherMachine : AbstractWashingObject
    {
        public void Sound()
        {
            Console.WriteLine("WasherMachine: Wggg-wggg!");
        }

        public override void Stop()
        {
            Console.WriteLine("Bi-bi-bi!");
        }
    }

    class Home
    {
        private AbstractWashingObject homeWashingObject = null;

        public Home(AbstractWashingObject objectWhoCanWash)
        {
            this.homeWashingObject = objectWhoCanWash;
        }

        public void DoJob()
        {
            homeWashingObject.Sound();
            homeWashingObject.Wash();
            homeWashingObject.Stop();
        }
    }

    class Program
    {
        static void DoWash()
        {
            Mom mother = new Mom();
            Himself himself = new Himself();
            Wife wife = new Wife();
            WasherMachine washerMachine = new WasherMachine();

            //
            Home home = new Home(mother);
            //Home home = new Home(himself);
            //Home home = new Home(wife);
            //Home home = new Home(washerMachine);

            home.DoJob();
        }

        static void Main(string[] args)
        {
            DoWash();
            Console.ReadKey();
        }
    }
```

***

## Интерфейсы

Интерфейс — семантическая и синтаксическая конструкция в коде программы, используемая для специфицирования услуг, предоставляемых классом или компонентом.
Интерфейс- стереотип, являющийся аналогом чистого абстрактного класса, в котором запрещена любая реализация.
Для имени интерфейса следует применять в качестве префикса букву "I". Это подсказывает, что данный тип является интерфейсом.

### Правила использования интерфейсов:
* **Невозможно создать экземпляр интерфейса.**
* **Интерфейсы и члены интерфейсов являются абстрактными. Интерфейсы не имеют реализации в C#.**
* **Интерфейс может содержать только абстрактные члены(методы, свойства, события или индексаторы).**
* **Члены интерфейсов автоматически являются открытыми, абстрактными, и они не могут иметь модификаторов доступа.**
* **Интерфейсы немогут содержать константы, поля, операторы, конструкторы экземпляров, деструкторы или вложенные типы(интерфейсы в томчисле).**
* **Класс или структура, которые реализуют интерфейс, должны реализовать члены этого интерфейса, указанные при его создании.**
* **Интерфейс может наследоваться от одного или нескольких базовых интерфейсов.**
* **Базовый класс также может реализовать члены интерфейса с помощью виртуальных членов.В этом случаеп роизводный класс может изменить поведение интерфейса путем переопределения виртуальных членов.**
* **Если класс реализует два интерфейса, содержащих член с одинаковой сигнатурой,то при реализации этогочлена в классе оба интерфейса будут использовать этот член для своей реализации.**
* **Если члены двух интерфейсовс одинаковой сигнатурой методов должны выполнять различные действия при их реализации, необходимо воспользоваться явной реализацией члена интерфейса — техникой явного указания в имени члена имени интерфейса, которому принадлежит данный член. Это достигается путем включения в имя члена класса имени интерфейса с точкой. Данный член в производном классе будет помечен по умолчанию как скрытый.**

### Преимущество использования интерфейсов:
* **Класс или структура может реализовать несколько интерфейсов (допустимо множественное наследование от интерфейсов).**
* **Есл икласс или структура реализует интерфейс,она получает только имена и сигнатуры метода.**
* **Интерфейсы определяют поведение экземпляров производных классов.**
* **Базовый класс может обладать ненужным функционалом, полученным от других его базовых классов,чего можно избежать, применяя интерфейсы.**

Пример, опять стирка:)))

```csharp
		interface IWash
        {
            void Wash();
            void Sound();
        }

        class Mother : IWash
        {
            public void Sound()
            {
                Console.WriteLine("Mother: Do you want to eat?");
            }

            public void Wash()
            {
                Console.WriteLine("Everything is washed!");
            }
        }

        class Himself : IWash
        {
            public void Sound()
            {
                Console.WriteLine("Himself: OMG o_O");
            }

            public void Wash()
            {
                Console.WriteLine("Everything is washed!");
            }
        }

        class Wife : IWash
        {
            public void Sound()
            {
                Console.WriteLine("Wife: Where is your salary?");
            }

            public void Wash()
            {
                Console.WriteLine("Everything is washed!");
            }
        }

        class WasherMachine : IWash
        {
            public void Sound()
            {
                Console.WriteLine("WasherMachine: Wggg-wggg!");
            }

            public void Wash()
            {
                Console.WriteLine("Everything is washed!");
            }
        }

        class Home
        {
            private IWash homeWashingObject = null;

            public Home(IWash objectWhoCanWash)
            {
                this.homeWashingObject = objectWhoCanWash;
            }

            public void DoJob()
            {
                homeWashingObject.Wash();
                homeWashingObject.Sound();
            }
        }

        static void DuWashAtHome()
        {
            Mother mother = new Mother();
            Himself himself = new Himself();
            Wife wife = new Wife();
            WasherMachine washerMachine = new WasherMachine();

            //
            Home home = new Home(mother);
            //Home home = new Home(himself);
            //Home home = new Home(wife);
            //Home home = new Home(washerMachine);
            home.DoJob();

            Console.ReadKey();
        }

        static void Main(string[] args)
        {
            DuWashAtHome();
            Console.ReadKey();
        }
```

***

## Абстрактные классы & Интерфейсы

Но места где можно постирать не ограничиваются вышеперечисленными. Например, мы можем постираться в прачечной. Вобщем получается довольно широкий круг объектов, которые связаны только тем, что являются местом для стирки и должны реализовать некоторый метод Wash().

Так как объекты малосвязанные между собой, то для определения общего для всех них функционала лучше определить интерфейс. Тем более некоторые из этих объектов могут существовать в рамках параллельных систем классификаций. Например, прачечная может быть классом в структуре системы сферы обслуживания "ServiceSector".

Пример, и снова стирка:)))
```csharp 
	public interface IWash
    {
        void Wash();
    }

    public abstract class ServiceSector
    {
    }

    public abstract class WashAtHome
    {
    }

    class Himself : WashAtHome, IWash
    {
        public void Wash()
        {
            Console.WriteLine("OMG o_O");
        }
    }

    public class Wife : WashAtHome, IWash
    {
        public void Wash()
        {
            Console.WriteLine("Wife: Where is your salary?");
        }
    }

    public class WashingHouse : ServiceSector, IWash
    {
        public void Wash()
        {
            Console.WriteLine("Always glad to see you!");
        }
    }

    class Program
    {
        static void DoWash()
        {
            IWash needToWash = new Himself();
            //IWash needToEat = new Wife();
            //IWash needToEat = new WashingHouse();

            needToWash.Wash();
        }

        static void Main(string[] args)
        {
            DoWash();
            Console.ReadKey();
        }
    }
  ```