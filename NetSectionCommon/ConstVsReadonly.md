# Const vs Readonly

## �������� ������

[Const and Readonly(metanit)](https://metanit.com/sharp/tutorial/3.3.php)

## Const

��������� ������������� ��� �������� ����� ��������, ������� �� ������ ���������� � ���������. ��� ����������� �������� ������������ �������� ����� const.

Const:

* **��������� �� ����� ���� ���������� ��� static, ��� �� ��������� static**

* **��������� ������ ���� ������������������� ��� �����������, ��� ��� �� ����� ���������� const ������ ���� ��� ����������**


* **����� ����������� �������� ��������� �� ����� ���� ��������**

```csharp
class MathLib
{
    public const double PI=3.141;
}
 
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine(MathLib.PI);
    }
}
```
## Readonly

���� readonly ����� ���������������� ��� �� ���������� ���� �� ������ ������, ���� ���������������� � �������� � ������������. ���������������� ��� �������� �� �������� � ������ ������ ������, ����� ������ ��������� �� ��������.�������� ����� readonly.

Readonly:

* **����� ���� ���������� ��� static**

* **�������� ���� ������������� �� ����� ���������� ���������(��� �������� �������)**

* **����� ���� ������������������ ���� ��� ����������, ���� � ������������**

```csharp
public class A
{
    public readonly double number1 = 12.21;

    public A()
    {
        number1 = 42.42;
    }
}

public class Program
{
    static void Main(string[] args)
    {

        var tempA = new A();

        Console.WriteLine(tempA.number1);
    }
}
```
## �����
* **Const ������ ���� ���������� �� ����� ����������, � readonly ����� ���� ���������� �� ����� ���������� ���������.**

* **���������������� const ����� ������ ��� �� �����������. Readonly ����� ���������������� ���� ��� ��� �����������, ���� � ������������ ������.**

* **Const �� ����� ���� ������������. Readonly ����� ���� ������������.**