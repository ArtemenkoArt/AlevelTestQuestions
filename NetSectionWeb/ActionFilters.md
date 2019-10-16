# Фильтры действий

Фильтры действий - это фильтры, которые могут применяться для любых целей. Встроенный интерфейс для создания фильтра данного типа, IActionFilter, показан в примере ниже:
```csharp
namespace System.Web.Mvc
{
    public interface IActionFilter
    {
        void OnActionExecuting(ActionExecutingContext filterContext);
        void OnActionExecuted(ActionExecutedContext filterContext);
    }
}
```
В интерфейсе IActionFilter определены два метода. Инфраструктура ASP.NET MVC Framework вызывает метод OnActionExecuting() перед обращением к методу действия, а метод OnActionExecuted() - после обращения к методу действия.

## Реализация метода OnActionExecuting()
Метод OnActionExecuting() вызывается перед обращением к методу действия. Этой возможностью можно воспользоваться для инспектирования запроса и принятия решения об отмене запроса, модификации запроса или о начале некоторой деятельности, которая включает вызов действия. Данный метод принимает в качестве параметра объект ActionExecutingContext, который является подклассом класса ControllerContext и определяет два дополнительных свойства:

ActionDescriptor
Предоставляет детали о методе действия

Result
Результат для метода действия: фильтр может отменить запрос, установив для этого свойства значение, отличное от null

Фильтр можно использовать для отмены запроса, установив свойство Result параметра в результат действия. Для демонстрации этого в папке Infrastructure примера проекта создается специальный класс фильтра действия по имени CustomActionAttribute, код которого приведен в примере ниже:
```csharp
using System.Web.Mvc;

namespace Filter.Infrastructure
{
    public class CustomActionAttribute : FilterAttribute, IActionFilter
    {
        public void OnActionExecuting(ActionExecutingContext filterContext)
        {
            if (filterContext.HttpContext.Request.IsLocal)
            {
                filterContext.Result = new HttpNotFoundResult();
            }
        }

        public void OnActionExecuted(ActionExecutedContext filterContext)
        {
            // не реализован
        }
    }
}
```
В этом примере метод OnActionExecuting() применяется для проверки, поступил ли запрос с локальной машины. Если это так, пользователю возвращается ответ 404 - Not Found (не найдено).

В этом примере можно заметить, что для создания работающего фильтра реализовывать оба метода, определенные в интерфейсе IActionFilter, не обязательно. Будьте осторожны, чтобы не сгенерировать исключение NotImplementedException, которое Visual Studio добавляет в класс при реализации интерфейса. Инфраструктура ASP.NET MVC Framework вызывает оба метода в фильтре действия, и если возникнет исключение, будут активизированы фильтры исключений. Если никакой логики в метод добавлять не нужно, просто оставьте его пустым.

Фильтр действия применяется подобно любому другому атрибуту. Для демонстрации работы фильтра действия, созданного в примере, мы добавили в контроллер Home новый метод действия:
```csharp
using System;
using System.Web.Mvc;
using Filter.Infrastructure;

namespace Filter.Controllers
{
    public class HomeController : Controller
    {
        // ...

        [CustomAction]
        public string FilterTest()
        {
            return "Это метод действия FilterTest в контроллере Home";
        }
    }
}
```
Чтобы протестировать фильтр, запустите приложение и перейдите на URL вида /Home/FilterTest. Запрос из браузера будет, естественно, локальным подключением, что приведет к генерации специальным фильтром действия ошибки 404 - Not Found:

![](https://professorweb.ru/my/ASP_NET/mvc/level5/files/img51500.jpg)

Если вы хотите удостовериться, что именно фильтр сгенерировал ошибку, просто удалите атрибут CustomAction из метода действия FilterTest() в контроллере Home и запустите приложение снова.

## Реализация метода OnActionExecuted()
Фильтр можно также использовать для решения задачи, которая предусматривает выполнение метода действия. В качестве простого примера в папке Infrastructure создан новый файл класса по имени ProfileActionAttribute.cs, в котором определен класс, предназначенный для измерения времени выполнения метода действия. Код этого фильтра показан в примере ниже:
```csharp
using System.Diagnostics;
using System.Web.Mvc;

namespace Filter.Infrastructure
{
    public class ProfileActionAttribute : FilterAttribute, IActionFilter
    {
        private Stopwatch timer;

        public void OnActionExecuting(ActionExecutingContext filterContext)
        {
            timer = Stopwatch.StartNew();
        }

        public void OnActionExecuted(ActionExecutedContext filterContext)
        {
            timer.Stop();
            if (filterContext.Exception == null)
            {
                filterContext.HttpContext.Response.Write(
                    string.Format("<div>Время работы метода действия: {0:F6}</div>",
                        timer.Elapsed.TotalSeconds));
            }
        }
    }
}
```
В данном примере метод OnActionExecuting() используется для запуска таймера (с применением класса высокоточного таймера Stopwatch из пространства имен System.Diagnostics). Метод OnActionExecuted() вызывается, когда метод действия завершен. В примере ниже показано, как применить этот атрибут к контроллеру Home. (Предыдущий фильтр был удален, поэтому локальные запросы перенаправляться не будут.)
```csharp
// ...
[ProfileAction]
public string FilterTest()
{
    return "Это метод действия FilterTest в контроллере Home";
}
// ...
```
Запустив приложение и перейдя на URL вида /Home/FilterTest, будет получен результат, показанный на рисунке:

![](https://professorweb.ru/my/ASP_NET/mvc/level5/files/img51501.jpg)

Обратите внимание, что информация профилирования выводится в браузере перед результатами метода действия. Причина в том, что фильтр действия выполняется после того, как метод действия завершен, но перед обработкой результата.

В качестве параметра методу OnActionExecuted() передается экземпляр класса ActionExecutedContext. В этом классе определен ряд полезных свойств, описанных в таблице ниже (описаны свойства кроме аналогичных ActionDescriptor и Result).

Свойства класса ActionExecutedContext
Имя	                Тип	           Описание
Canceled	        bool           Возвращает true, если действие было отменено другим фильтром

Exception	      Exception	       Возвращает объект исключения, сгенерированного другим фильтром или методом действия

ExceptionHandled	bool           Возвращает true, если исключение было обработано

Свойство Exception возвращает исключение, которое сгенерировал метод действия, а свойство ExceptionHandled указывает, обработал ли это исключение другой фильтр.

Свойство Canceled возвратит true, если другой фильтр отменил запрос (за счет установки значения для свойства Result) после момента, когда был вызван метод OnActionExecuting() фильтра. Метод OnActionExecuted() будет по-прежнему вызываться, но только чтобы можно было провести очистку и освободить занятые ресурсы.

