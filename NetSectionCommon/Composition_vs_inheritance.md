# Composition vs Inheritance

## Полезные ссылки

[Линк с табличкой Composition vs Inheritance](https://www.c-sharpcorner.com/UploadFile/ff2f08/inheritance-vs-composition/)

[Еще 1 классный пример](https://www.codeproject.com/Articles/80045/Composition-VS-Inheritance)

## Example

### Условие

Мы пишем игру, у нас могут быть разные типы монстров. Различные типы монстров могут быть в состоянии делать различные атаки - кусать, пинать или наносить удары.

Один из способов сделать это - создать базовый класс Monster и создать подклассы для BitingMonster, KickingMonster и PunchingMonster - при этом каждый подкласс обрабатывает детали для различных способов борьбы.

Но что вы будете делать, если у вас есть новый монстр, который умеет кусать и пинать?

Используя наследование, вы можете создать BitingKickingMonster. Это может наследоваться от Monster, BitingMonster или KickingMonster.

## Версия наследования

Давайте представим, что вы работаете в компании по разработке игр.

Вы создаете новую игру и создаете класс Monster с двумя свойствами - HitPoints и AttackDamage. Код может выглядеть так:

```csharp 
    public class Monster
    {
        public int HitPoints { get; set; }
        public int AttackDamage { get; set; }

        public Monster(int hitPoints, int attackDamage)
        {
            HitPoints = hitPoints;
            AttackDamage = attackDamage;
        }
    }
```

Затем ваш босс говорит вам, что в игре должны быть разные типы монстров. Один тип атакует укусом, второй ударом ногой, а третий ударом кулаком. Итак, вы превращаете класс Monster в базовый класс и создаете из него три новых подкласса: BitingMonster, KickingMonster и PunchingMonster.

Теперь у вас есть эти классы:

* **Monster.cs**

```csharp
    public class Monster
    {
        public int HitPoints { get; set; }
 
        public Monster(int hitPoints)
        {
            HitPoints = hitPoints;
        }
    } 
```
* **BitingMonster.cs**


```csharp 
    public class BitingMonster : Monster
    {
        public int BiteDamage { get; set; }
 
        public BitingMonster(int hitPoints, int biteDamage)
            : base(hitPoints)
        {
            BiteDamage = biteDamage;
        }
    }
```

* **KickingMonster.cs**
```csharp 
    public class KickingMonster : Monster
    {
        public int KickDamage { get; set; }
 
        public KickingMonster(int hitPoints, int kickDamage)
            : base(hitPoints)
        {
            KickDamage = kickDamage;
        }
    }
```


* **PunchingMonster.cs**
```csharp 
    public class PunchingMonster : Monster
    {
        public int PunchDamage { get; set; }
 
        public PunchingMonster(int hitPoints, int punchDamage)
            : base(hitPoints)
        {
            PunchDamage = punchDamage;
        }
    }
```

На следующий день ваш босс скажет вам, что им нужны новые типы монстров в игре, которые могут выполнять различные комбинации укусов, ударов руками и ногами. По мере того, как это становится все сложнее и сложнее, вы также создаете класс Factory для создания различных типов монстров.

Если вы продолжите подкласс, вы можете получить следующий код(+4 класса, +Factory):

* **BitingKickingMonster.cs**
```csharp
    public class BitingKickingMonster : BitingMonster
    {
        public int KickDamage { get; set; }
 
        public BitingKickingMonster(int hitPoints, int biteDamage, int kickDamage)
            : base(hitPoints, biteDamage)
        {
            KickDamage = kickDamage;
        }
    }
``` 
* **BitingKickingPunchingMonster.cs**
```csharp
    public class BitingKickingPunchingMonster : BitingKickingMonster
    {
        public int PunchDamage { get; set; }

        public BitingKickingPunchingMonster(int hitPoints, int biteDamage, int kickDamage, int punchDamage)
            : base(hitPoints, biteDamage, kickDamage)
        {
            PunchDamage = punchDamage;
        }
    }
```
* **BitingPunchingMonster.cs**
```csharp
    public class BitingPunchingMonster : BitingMonster
    {
        public int PunchDamage { get; set; }
 
        public BitingPunchingMonster(int hitPoints, int biteDamage, int punchDamage)
            : base(hitPoints, biteDamage)
        {
            PunchDamage = punchDamage;
        }
    }
```
* **KickingPunchingMonster.cs**
```csharp
    public class KickingPunchingMonster : KickingMonster
    {
        public int PunchDamage { get; set; }
 
        public KickingPunchingMonster(int hitPoints, int kickDamage, int punchDamage)
            : base(hitPoints, kickDamage)
        {
            PunchDamage = punchDamage;
        }
    }
```

* **MonsterFactory.cs**
```csharp
    public class MonsterFactory
    {
        public enum MonsterType
        {
            Horse, // BitingKickingMonster
            Orc, // BitingKickingPunchingMonster
            Crocodile, // BitingMonster
            MikeTyson, // BitingPunchingMonster
            Camel, // KickingMonster
            Kangaroo, // KickingPunchingMonster
            MantisShrimp //PunchingMonster
        }
 
        public static Monster CreateMonster(MonsterType monsterType)
        {
            switch(monsterType)
            {
                case MonsterType.Horse:
                    return new BitingKickingMonster(10, 5, 5);
                case MonsterType.Orc:
                    return new BitingKickingPunchingMonster(10, 5, 5, 5);
                case MonsterType.Crocodile:
                    return new BitingMonster(10, 5);
                case MonsterType.MikeTyson:
                    return new BitingPunchingMonster(10, 5, 5);
                case MonsterType.Camel:
                    return new KickingMonster(10, 5);
                case MonsterType.Kangaroo:
                    return new KickingPunchingMonster(10, 5, 5);
                case MonsterType.MantisShrimp:
                    return new PunchingMonster(10, 5);
                default:
                    throw new ArgumentException();
            }
        }
    }
```

Теперь тестим:

```csharp
    public static void Test_BitingMonster()
    {
        Monster crocodile = MonsterFactory.CreateMonster(MonsterFactory.MonsterType.Crocodile);

        Console.WriteLine(crocodile is BitingMonster); //true

    }

    public static void Test_BitingKickingMonster()
    {
        Monster horse = MonsterFactory.CreateMonster(MonsterFactory.MonsterType.Horse);

        Console.WriteLine(horse is BitingMonster);// true

        // This test will fail, because we cannot inherit from multiple base classes.
        Console.WriteLine(horse is KickingMonster);//false
    }
```

Если мы попытаемся определить «тип» объекта, определить, какие атаки он может выполнить, мы можем проверить только один базовый класс, а не второй.

После того, как вы закончите создавать все эти классы, ваш начальник отправит вам электронное письмо. Теперь вам нужно иметь монстров, которые также могут атаковать плевками.

Кобры будут кусать и плевать, верблюды будут пинать и плевать и т.д.

Вместо того, чтобы создавать больше подклассов, вы решаете попробовать использовать композицию.

## Версия композиции

В этом случае у нас будет один класс Monster, и мы «составим» его с подходящим поведением атаки для каждого типа монстров.

* **Monster.cs**
```csharp
    public class Monster
    {
        public enum AttackType
        {
            Biting,
            Kicking,
            Punching
        }
 
        public int HitPoints { get; set; }
        public Dictionary<AttackType, int> AttackTypes { get; set; }
 
        // These replace the functionality of checking an object's "type",
        // to see if it "is" a certain datatype (KickingMonster, BitingMonster, etc.)
        public bool CanBite => AttackTypes.ContainsKey(AttackType.Biting);
        public bool CanKick => AttackTypes.ContainsKey(AttackType.Kicking);
        public bool CanPunch => AttackTypes.ContainsKey(AttackType.Punching);
 
        public int BiteDamage
        {
            get
            {
                if(CanBite)
                {
                    return AttackTypes[AttackType.Biting];
                }
 
                throw new NotSupportedException("This monster cannot bite.");
            }
        }
 
        public int KickDamage
        {
            get
            {
                if(CanKick)
                {
                    return AttackTypes[AttackType.Kicking];
                }
 
                throw new NotSupportedException("This monster cannot kick.");
            }
        }
 
        public int PunchDamage
        {
            get
            {
                if(CanPunch)
                {
                    return AttackTypes[AttackType.Punching];
                }
 
                throw new NotSupportedException("This monster cannot punch.");
            }
        }
 
        public Monster(int hitPoints)
        {
            HitPoints = hitPoints;
            AttackTypes = new Dictionary<AttackType, int>();
        }
 
        public void AddAttackType(AttackType attackType, int amountOfDamage)
        {
            AttackTypes[attackType] = amountOfDamage;
        }
    }
```

С помощью этого класса Monster, когда мы создаем новый объект Monster, мы «составляем» его параметры Attack, вызывая функцию AddAttackType () - с AttackType, и количество урона, которое монстр наносит этой атакой.

Я добавил еще несколько свойств (CanBite, CanKick и CanPunch), чтобы было проще узнать, какие типы атак может выполнять монстр.

Этот класс также содержит все свойства урона (BiteDamage, KickDamage и PunchDamage).

С этими шестью свойствами мы можем составить объект Monster для атаки так, как мы хотим - что мы делаем в классе MonsterFactory ниже.

* **MonsterFactory.cs**

```csharp
    public class MonsterFactory
    {
        public enum MonsterType
        {
            Horse, // BitingKickingMonster
            Orc, // BitingKickingPunchingMonster
            Crocodile, // BitingMonster
            MikeTyson, // BitingPunchingMonster
            Camel, // KickingMonster
            Kangaroo, // KickingPunchingMonster
            MantisShrimp //PunchingMonster
        }
 
        public static Monster CreateMonster(MonsterType monsterType)
        {
            Monster monster;
 
            switch(monsterType)
            {
                case MonsterType.Horse:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    break;
                case MonsterType.Orc:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                case MonsterType.Crocodile:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    break;
                case MonsterType.MikeTyson:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                case MonsterType.Camel:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    break;
                case MonsterType.Kangaroo:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                case MonsterType.MantisShrimp:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                default:
                    throw new ArgumentException();
            }
 
            return monster;
        }
    }
```
Это немного сложнее, чем предыдущая фабрика, потому что именно здесь мы «составляем» объект Monster, чтобы действовать как объект BitingMonster (или любые другие атаки, которые может выполнять монстр).

* **TestMonsterFactory.cs**

```csharp
    public static void Test_BitingMonster()
    {
        Monster crocodile = MonsterFactory.CreateMonster(MonsterFactory.MonsterType.Crocodile);

    Console.WriteLine(crocodile.CanBite);//true
    Console.WriteLine(crocodile.CanKick);//false
        Console.WriteLine(crocodile.CanPunch);//false
    }


    public  static void Test_BitingKickingMonster()
    {
        Monster horse = MonsterFactory.CreateMonster(MonsterFactory.MonsterType.Horse);

        Console.WriteLine(horse.CanBite);//true
        Console.WriteLine(horse.CanKick);//true
        Console.WriteLine(horse.CanPunch);//false
    }
```

В этих тестах мы можем использовать свойства CanBite, CanKick и CanPunch для имитации проверки «типа», как это было бы в версии наследования.

При использовании метода композиции, когда наш босс говорит нам добавить новую атаку «Spit» вместо создания новых классов, нам нужно только:

- Добавьте новое значение «Spitting» в перечисление AttackType в Monster.cs
- Добавить новое свойство «CanSpit» в Monster.cs
- Добавить новое свойство «SpitDamage» в Monster.cs

Затем MonsterFactory.cs может создавать монстров, которые могут использовать новую атаку «плевок».

Код может выглядеть так:

* **Monster.cs**

```csharp
    public class Monster
    {
        public enum AttackType
        {
            Biting,
            Kicking,
            Punching,
            Spitting
        }
 
        public int HitPoints { get; set; }
        public Dictionary<AttackType, int> AttackTypes { get; set; }
 
        // These replace the functionality of checking an object's "type",
        // to see if it "is" a certain datatype (KickingMonster, BitingMonster, etc.)
        public bool CanBite => AttackTypes.ContainsKey(AttackType.Biting);
        public bool CanKick => AttackTypes.ContainsKey(AttackType.Kicking);
        public bool CanPunch => AttackTypes.ContainsKey(AttackType.Punching);
        public bool CanSpit => AttackTypes.ContainsKey(AttackType.Spitting);
 
        public int BiteDamage
        {
            get
            {
                if(CanBite)
                {
                    return AttackTypes[AttackType.Biting];
                }
 
                throw new NotSupportedException("This monster cannot bite.");
            }
        }
 
        public int KickDamage
        {
            get
            {
                if(CanKick)
                {
                    return AttackTypes[AttackType.Kicking];
                }
 
                throw new NotSupportedException("This monster cannot kick.");
            }
        }
 
        public int PunchDamage
        {
            get
            {
                if(CanPunch)
                {
                    return AttackTypes[AttackType.Punching];
                }
 
                throw new NotSupportedException("This monster cannot punch.");
            }
        }
 
        public int SpitDamage
        {
            get
            {
                if(CanSpit)
                {
                    return AttackTypes[AttackType.Spitting];
                }
 
                throw new NotSupportedException("This monster cannot spit.");
            }
        }
 
        public Monster(int hitPoints)
        {
            HitPoints = hitPoints;
            AttackTypes = new Dictionary<AttackType, int>();
        }
 
        public void AddAttackType(AttackType attackType, int amountOfDamage)
        {
            AttackTypes[attackType] = amountOfDamage;
        }
    }   
```

* **MonsterFactory.cs**

```csharp
    public class MonsterFactory
    {
        public enum MonsterType
        {
            Horse, // BitingKickingMonster
            Orc, // BitingKickingPunchingMonster
            Crocodile, // BitingMonster
            MikeTyson, // BitingPunchingMonster
            Cobra, // BitingSpittingMonster
            Camel, // KickingSpittingMonster
            Kangaroo, // KickingPunchingMonster
            MantisShrimp //PunchingMonster
        }
 
        public static Monster CreateMonster(MonsterType monsterType)
        {
            Monster monster;
 
            switch(monsterType)
            {
                case MonsterType.Horse:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    break;
                case MonsterType.Orc:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                case MonsterType.Crocodile:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    break;
                case MonsterType.MikeTyson:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                case MonsterType.Cobra:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Biting, 5);
                    monster.AddAttackType(Monster.AttackType.Spitting, 5);
                    break;
                case MonsterType.Camel:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    monster.AddAttackType(Monster.AttackType.Spitting, 5);
                    break;
                case MonsterType.Kangaroo:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Kicking, 5);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                case MonsterType.MantisShrimp:
                    monster = new Monster(10);
                    monster.AddAttackType(Monster.AttackType.Punching, 5);
                    break;
                default:
                    throw new ArgumentException();
            }
 
            return monster;
        }
    }
```

Обратите внимание, что Верблюд теперь может пнуть и плюнуть. Также появился новый монстр: кобра, которая может кусать и плевать.

Чтобы дать верблюду новую способность к атаке, нам нужно было добавить только одну линию.

Чтобы добавить кобру, нам нужно было только добавить ее в перечисление и добавить новый «падеж» (начиная со строки 45), чтобы создать кобру - составив монстра, который может кусать и плевать.

## Где я нашел это полезным!

Существует две распространенные ситуации, когда вы хотите рассмотреть возможность использования композиции вместо наследования: когда вам нужно сделать множественное наследование, и когда ваши подклассы начинают иметь свои собственные подклассы.

Это большие показатели того, что композиция может быть лучшим выбором.

Вы также можете комбинировать композицию объектов с шаблоном дизайна стратегии. Вместо того, чтобы составлять объект со значениями свойств (тип атаки и ущерб), вы также можете составить его, настроив, как он будет выполнять свои действия.