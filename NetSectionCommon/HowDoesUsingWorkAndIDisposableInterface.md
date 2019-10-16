# IDisposable Interface

Предоставляет механизм для освобождения неуправляемых ресурсов.

```csharp
[System.Runtime.InteropServices.ComVisible(true)]
public interface IDisposable
```
Производный
- BaseLock 
- DebuggerManager.IslandThread
- ProjectCollection
- BuildManager
- BaseDataEnvironment

Атрибуты
- ComVisibleAttribute


## Примеры
В следующем примере показано, как создать класс ресурсов, реализующий IDisposable интерфейс.
```csharp
using System;
using System.ComponentModel;

// The following example demonstrates how to create
// a resource class that implements the IDisposable interface
// and the IDisposable.Dispose method.

public class DisposeExample
{
    // A base class that implements IDisposable.
    // By implementing IDisposable, you are announcing that
    // instances of this type allocate scarce resources.
    public class MyResource: IDisposable
    {
        // Pointer to an external unmanaged resource.
        private IntPtr handle;
        // Other managed resource this class uses.
        private Component component = new Component();
        // Track whether Dispose has been called.
        private bool disposed = false;

        // The class constructor.
        public MyResource(IntPtr handle)
        {
            this.handle = handle;
        }

        // Implement IDisposable.
        // Do not make this method virtual.
        // A derived class should not be able to override this method.
        public void Dispose()
        {
            Dispose(true);
            // This object will be cleaned up by the Dispose method.
            // Therefore, you should call GC.SupressFinalize to
            // take this object off the finalization queue
            // and prevent finalization code for this object
            // from executing a second time.
            GC.SuppressFinalize(this);
        }

        // Dispose(bool disposing) executes in two distinct scenarios.
        // If disposing equals true, the method has been called directly
        // or indirectly by a user's code. Managed and unmanaged resources
        // can be disposed.
        // If disposing equals false, the method has been called by the
        // runtime from inside the finalizer and you should not reference
        // other objects. Only unmanaged resources can be disposed.
        protected virtual void Dispose(bool disposing)
        {
            // Check to see if Dispose has already been called.
            if(!this.disposed)
            {
                // If disposing equals true, dispose all managed
                // and unmanaged resources.
                if(disposing)
                {
                    // Dispose managed resources.
                    component.Dispose();
                }

                // Call the appropriate methods to clean up
                // unmanaged resources here.
                // If disposing is false,
                // only the following code is executed.
                CloseHandle(handle);
                handle = IntPtr.Zero;

                // Note disposing has been done.
                disposed = true;

            }
        }

        // Use interop to call the method necessary
        // to clean up the unmanaged resource.
        [System.Runtime.InteropServices.DllImport("Kernel32")]
        private extern static Boolean CloseHandle(IntPtr handle);

        // Use C# destructor syntax for finalization code.
        // This destructor will run only if the Dispose method
        // does not get called.
        // It gives your base class the opportunity to finalize.
        // Do not provide destructors in types derived from this class.
        ~MyResource()
        {
            // Do not re-create Dispose clean-up code here.
            // Calling Dispose(false) is optimal in terms of
            // readability and maintainability.
            Dispose(false);
        }
    }
    public static void Main()
    {
        // Insert code here to create
        // and use the MyResource object.
    }
}
```

## Комментарии

В основном этот интерфейс используется для высвобождения неуправляемых ресурсов. Сборщик мусора автоматически освобождает память, выделенную управляемому объекту, если этот объект больше не используется. Однако невозможно предсказать, когда произойдет сборка мусора. Более того, сборщик мусора не имеет сведений о неуправляемых ресурсах, таких как дескрипторы окон, или открытых файлах и потоках.

Dispose Используйте метод этого интерфейса для явного освобождения неуправляемых ресурсов в сочетании с сборщиком мусора. Потребитель объекта может вызвать этот метод, если объект больше не нужен.

### Предупреждение
Это коренное изменение для добавления IDisposable интерфейса в существующий класс. Поскольку уже существующие потребители типа не могут вызывать Dispose, нельзя быть уверенным, что неуправляемые ресурсы, удерживаемые типом, будут освобождены.

Поскольку реализация вызывается потребителем типа, когда ресурсы, принадлежащие экземпляру, больше не нужны, необходимо либо заключить управляемый объект SafeHandle в (рекомендуемый альтернативный вариант), либо переопределить IDisposable.Dispose Object.Finalizeзначение, чтобы освободить неуправляемые ресурсы в событии, которое пользователь забыл вызвать Dispose.

### Важно!
В .NET Framework C++ компилятор поддерживает детерминированное удаление ресурсов и не допускает прямую реализацию Dispose метода.

Подробное обсуждение использования этого интерфейса и Object.Finalize метода см. в разделах сборка мусора и Реализация метода Dispose .

## Использование объекта, реализующего IDisposable

Если приложение просто использует объект, реализующий IDisposable интерфейс, следует вызвать IDisposable.Dispose реализацию объекта, когда вы завершите его использование. В зависимости от языка программирования это можно сделать одним из двух способов:

С помощью языковой конструкции, такой как using оператор в C# и Visual Basic.

Путем заключения вызова IDisposable.Dispose в / реализацию try в блоке.finally

### Примечание
Документация по типам, которые IDisposable реализуют, обратите внимание на то, что Dispose в действительности и включает напоминание для вызова его реализации.

### Оператор C# и Visual Basic using
Если язык поддерживает конструкцию, такую как оператор using в C# , и оператор using в Visual Basic, его можно использовать вместо явного вызова IDisposable.Dispose . В следующем примере этот подход используется при определении WordCount класса, сохраняющего сведения о файле и количестве слов в нем.

```csharp
using System;
using System.IO;
using System.Text.RegularExpressions;

public class WordCount
{
   private String filename = String.Empty;
   private int nWords = 0;
   private String pattern = @"\b\w+\b"; 

   public WordCount(string filename)
   {
      if (! File.Exists(filename))
         throw new FileNotFoundException("The file does not exist.");
      
      this.filename = filename;
      string txt = String.Empty;
      using (StreamReader sr = new StreamReader(filename)) {
         txt = sr.ReadToEnd();
      }
      nWords = Regex.Matches(txt, pattern).Count;
   }
   
   public string FullName
   { get { return filename; } }
   
   public string Name
   { get { return Path.GetFileName(filename); } }
   
   public int Count 
   { get { return nWords; } }
}   
```
Эта using инструкция на самом деле является синтаксическим удобством. Во время компиляции языковой компилятор реализует промежуточный язык (IL) для try / finally блока.
Дополнительные сведения об using инструкции см. в разделах инструкция using или инструкции по использованию .

### Блок try/finally
Если язык программирования не using поддерживает конструкцию, подобную инструкции в C# или Visual Basic, или если вы предпочитаете не использовать ее, можно вызвать IDisposable.Dispose реализацию из блока try finally /оператор. finally Следующий пример заменяет using блок в предыдущем / примере try finally блоком.

```csharp
using System;
using System.IO;
using System.Text.RegularExpressions;

public class WordCount
{
   private String filename = String.Empty;
   private int nWords = 0;
   private String pattern = @"\b\w+\b"; 

   public WordCount(string filename)
   {
      if (! File.Exists(filename))
         throw new FileNotFoundException("The file does not exist.");
      
      this.filename = filename;
      string txt = String.Empty;
      StreamReader sr = null;
      try {
         sr = new StreamReader(filename);
         txt = sr.ReadToEnd();
      }
      finally {
         if (sr != null) sr.Dispose();     
      }
      nWords = Regex.Matches(txt, pattern).Count;
   }
   
   public string FullName
   { get { return filename; } }
   
   public string Name
   { get { return Path.GetFileName(filename); } }
   
   public int Count 
   { get { return nWords; } }
}   
```
Дополнительные сведения о шаблоне try см. в / finally разделе try... Перехватить... Оператор finally, try-finallyили try-finally.

### Использование IDisposable
Следует реализовывать IDisposable , только если тип использует неуправляемые ресурсы напрямую. Потребители вашего типа могут вызывать вашу IDisposable.Dispose реализацию для освобождения ресурсов, когда экземпляр больше не нужен. Чтобы обрабатывать случаи, в которых они не вызывают Disposeошибку, следует использовать класс, производный от SafeHandle , чтобы создать оболочку для неуправляемых ресурсов, Object.Finalize или переопределить метод для ссылочного типа. В любом случае используется Dispose метод для выполнения любой очистки, необходимой после использования неуправляемых ресурсов, таких как освобождение, освобождение или сброс неуправляемых ресурсов.

### Важно!
При определении базового класса, использующего неуправляемые ресурсы и имеющего или, скорее всего, подклассов, которые должны быть удалены, следует реализовать IDisposable.Dispose метод и предоставить вторую Disposeперегрузку, как обсуждалось в следующей раздела.


### IDisposable и иерархия наследования
Базовый класс с подклассами, которые должны быть уничтожены, IDisposable должен реализовываться следующим образом. Этот шаблон следует использовать при реализации IDisposable любого типа, который не является sealed (NotInheritable в Visual Basic).

- Он должен предоставлять один открытый, не виртуальный Dispose() метод и защищенный виртуальный Dispose(Boolean disposing) метод.

- Метод должен вызывать Dispose(true) и отключать завершение для повышения производительности. Dispose()

- Базовый тип не должен содержать все завершения.

Следующий фрагмент кода отражает шаблон удаления для базовых классов. Предполагается, что тип не переопределяет Object.Finalize метод.

```csharp
using Microsoft.Win32.SafeHandles;
using System;
using System.Runtime.InteropServices;

class BaseClass : IDisposable
{
   // Flag: Has Dispose already been called?
   bool disposed = false;
   // Instantiate a SafeHandle instance.
   SafeHandle handle = new SafeFileHandle(IntPtr.Zero, true);
   
   // Public implementation of Dispose pattern callable by consumers.
   public void Dispose()
   { 
      Dispose(true);
      GC.SuppressFinalize(this);           
   }
   
   // Protected implementation of Dispose pattern.
   protected virtual void Dispose(bool disposing)
   {
      if (disposed)
         return; 
      
      if (disposing) {
         handle.Dispose();
         // Free any other managed objects here.
         //
      }
      
      disposed = true;
   }
}
```
При переопределении Object.Finalize метода класс должен реализовать следующий шаблон.

```csharp
using System;

class BaseClass : IDisposable
{
   // Flag: Has Dispose already been called?
   bool disposed = false;
   
   // Public implementation of Dispose pattern callable by consumers.
   public void Dispose()
   { 
      Dispose(true);
      GC.SuppressFinalize(this);           
   }
   
   // Protected implementation of Dispose pattern.
   protected virtual void Dispose(bool disposing)
   {
      if (disposed)
         return; 
      
      if (disposing) {
         // Free any other managed objects here.
         //
      }
      
      // Free any unmanaged objects here.
      //
      disposed = true;
   }

   ~BaseClass()
   {
      Dispose(false);
   }
}
```
Подклассы должны реализовывать удаляемый шаблон следующим образом:

- Они должны переопределить Dispose(Boolean) и вызвать реализацию базового класса Dispose(Boolean).

- Они могут предоставлять завершение при необходимости. Метод завершения должен вызвать Dispose(false).

Обратите внимание, что производные классы сами IDisposable по себе не реализуют интерфейс и не Dispose включают метод без параметров. Они переопределяют только метод базового Dispose(Boolean) класса.
Следующий фрагмент кода отражает шаблон удаления для производных классов. Предполагается, что тип не переопределяет Object.Finalize метод.

```csharp
using Microsoft.Win32.SafeHandles;
using System;
using System.Runtime.InteropServices;

class DerivedClass : BaseClass
{
   // Flag: Has Dispose already been called?
   bool disposed = false;
   // Instantiate a SafeHandle instance.
   SafeHandle handle = new SafeFileHandle(IntPtr.Zero, true);

   // Protected implementation of Dispose pattern.
   protected override void Dispose(bool disposing)
   {
      if (disposed)
         return; 
      
      if (disposing) {
         handle.Dispose();
         // Free any other managed objects here.
         //
      }
      
      // Free any unmanaged objects here.
      //

      disposed = true;
      // Call base class implementation.
      base.Dispose(disposing);
   }
}
```