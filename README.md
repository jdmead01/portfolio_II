# Portfolio II

Now that we've learned how to return Views (html pages) and reference static content (CSS, JavaScript, images, etc.), let's update our previous Portfolio assignment. The three routes should already be set up; we'll just have to update the return types for each of the methods and create 3 html pages:

* **"/"** - **Home Page**
  * Personal information
* **"/projects"** - **Projects Page**
  * A list of some of your projects
* **"/contact"** - **Contact Page**
  * A form where people visiting can send you information (don't worry about form functionality quite yet)
---
Have a navigation bar that links to each of the pages.

These wireframes just give you a rough outline--feel free to be creative and personalize it. Just be sure to incorporate static style sheets, images, and maybe even some JavaScript to make sure you're practicing those skills. (Don't get too carried away though--there will be more time to complete your portfolio when you graduate! 
***
![Alt text](https://s3.amazonaws.com/General_V88/boomyeah2015/codingdojo/curriculum/content/chapter/PortfolioII%28edit%29.png)
***

***Complete the following for this assignment***

* [ ] Update your 3 routes to return Views instead of Strings
* [ ] There should be a navbar with tabs to each section displayed at the top
* [ ] The Home page should have a picture, name, and about me section
* [ ] he Projects page should have at least 3 projects, each with a title, image, and small description
* [ ] The Contact page should have a form requesting the inquirer's name, email, and their message
* [ ] Remember all pages should have some styling!!
* [ ] (BONUS) Add some JavaScript for a more interactive user experience


***
# Rendering Views

The most common response for our controllers will be to serve a view, and ASP.NET Core makes serving them a breeze. Any time one of our Controllers returns a View, the framework attempts to use the name of your controller to find the appropriate view file. ASP.NET looks for a directory called "Views", then looks for a subdirectory of Views that matches the name of your controller, and if it fails to find the view there it will always look for a folder called ```Shared``` as a last resort.

Make sure your Views subdirectories have names that match your controller class names (Without "Controller")

For example ```UsersController``` would look for its views in the ```Users``` subdirectory (Views/Users). If a view needs to be rendered from multiple Controllers it must be in ```Shared``` (Views/Shared).  

Our view files will not be standard HTML files, as they will have a view engine for embedding C# code, and will have a ```.cshtml``` file extension.  However, for all intents and purposes, you may treat them exactly as you would normal HTML files. ASP.NET Core MVC will not read standard ```.html``` files without some extra configuration. 

Let's make some content to display. Create a new folder called ```Views``` in your project directory, then make a subdirectory that matches your controller name. Here, add a new file called "Index.cshtml"

![Alt text](https://s3.amazonaws.com/General_V88/boomyeah2015/codingdojo/curriculum/content/chapter/ViewStructure2.PNG)

**Index.cshtml**
```javascript
<!DOCTYPE html>
<html>
    <head>
        <meta charset='utf-8'>
        <title>Hello!</title>
    </head>
    <body>
        <h1>Hello ASP.NET Mvc!</h1>
    </body>
</html>
```

**ViewResult**

Now we need to tell our controller to actually serve the view, which you can do by calling a function found in the MVC library: View(). In fact, we will want to return this function call from an action in our controller that wants to serve a view. Remember, it is the responsibly of our controller's actions to return a valid server response, and the return type of the View() function itself is ViewResult: a type defined in the MVC library, built specifically for handing a view response. So because of this, we can list ViewResult as the return type of the method in charge of serving a view.  

If we invoke View() with no arguments, ASP will look to serve a view file with the name of the action making the call. So if the name of the action is Index, ASP will go hunting for a file called Index.cshtml within your project's Views directory.

You may also invoke View("FileName"), with the name of a specific view file (minus .cshtml).

**HelloController.cs**
```javascript
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
namespace YourNamespace.Controllers
{
    public class HelloController : Controller
    {
        [HttpGet]
        [Route("")]
        public ViewResult Index()
        {
            // will attempt to serve 
                // Views/Hello/Index.cshtml
            // or if that file isn't there:
                // Views/Shared/Index.cshtml
            return View();
        }
        [HttpGet]
        [Route("info")]
        public ViewResult Info()
        {
            // Same logic for serving a view applies
            // if we provide the the exact view name
            return View("Info");
        }
        // You may also serve the same view twice from additional actions
        [HttpGet("elsewhere")]
        public ViewResult Elsewhere()
        {
            return View("Index");
        }
    }
}
```
***
# Redirecting

Sometimes we want to redirect to other controller methods rather than rendering a view. In ASP.NET Core MVC, we'll use the ```RedirectToAction()``` response. Redirecting can also be done based on routes, but ```RedirectToAction``` is preferable because our routes are more arbitrary and subject to change than our controller method names.  The return type of RedirectToAction() is **RedirectToActionResult**, which you can use as the return type for your redirecting actions.

The ```RedirectToAction``` method accepts a string that corresponds to the name of another method (or "Action") in your controller.
```javascript
// Other code
public class FirstController : Controller
{
    //  Other code
    public RedirectToActionResult Method()
    {
        // Will redirect to the "OtherMethod" method
        return RedirectToAction("OtherMethod");
    }
    // Other code
    public ViewResult OtherMethod()
    {
        return View();
    }
}
```
If the method you want to redirect to expects route parameters,you can include them too. We pass an anonymous object as an additional argument:
```javascript
// Other code
public RedirectToActionResult Method()
{
    // The anonymous object consists of keys and values
    // The keys should match the parameter names of the method being redirected to
    return RedirectToAction("OtherMethod", new { Food = "sandwiches", Quantity = 5 });
}
 
[HttpGet]
[Route("other/{Food}/{Quantity}")]
public ViewResult OtherMethod(string Food, int Quantity)
{
    Console.WriteLine($"I ate {Quantity} {Food}.");
    // Writes "I ate 5 sandwiches."
}
```

As you can see, we can use this method to pass regular parameters, route parameters, or both.

If you want to redirect to another controller's method, we have to specify the controller name as well as the method name. Just like how our View folders have names like "Home" instead of "HomeController", when we refer to a controller we leave out the word "controller" from the name.
```javascript
// Other code
public class FirstController : Controller
{
    public RedirectToActionResult Method()
    {
        return RedirectToAction("OtherMethod", "Second");
    }
}
 
// In another file
public class SecondController : Controller
{
    public ViewResult OtherMethod()
    {
        return View();
    }
}
```
We can also pass arguments when redirecting to a different controller; simply add an anonymous object as a third argument to the RedirectToAction method.
***
# Returning JSON

JSON is used for performing AJAX operations with our front end, and it is the most common return format for APIs. ASP.NET Core MVC Controllers contain a method for serializing C# objects into JSON, so creating an API backend is a snap.

**YourController.cs**
```javascript
using Microsoft.AspNetCore.Mvc;
 
namespace YourNamespace.Controllers
{
    public class YourController : Controller
    {
        [HttpGet]
        [Route("")]
        public JsonResult Example()
        {
            // The Json method converts the object passed to it into JSON
            return Json(SomeObject);
        }
    }
}
```
The ```Json()``` method will accept any type of object for serialization, from simple values like ints to complex objects:

```javascript
// Other code
[HttpGet]
[Route("displayint")]
public JsonResult DisplayInt()
{
    return Json(34);
}
 
// Suppose we're working with the Human class we wrote in the previous chapter
[HttpGet]
[Route("displayhuman")]
public JsonResult DisplayHuman()
{
    return Json(new Human("Moose Phillips", 50, 5, 5));
}
```
DisplayHuman() would produce the following in your browser 

(image depicts Chrome with ```JSONView``` extension):

![Alt text](https://s3.amazonaws.com/General_V88/boomyeah2015/codingdojo/curriculum/content/chapter/display-human.png)

**Anonymous Objects**

If we want to return a group of values as JSON we have to write a class definition with appropriate fields to contain the data. But what if a different route is needed to return a slightly different number of values, or values of different types? We'd need to write an additional class for each configuration of values. Instead, we can use what are known as anonymous objects. **Anonymous objects** can be instantiated as a grouping of property names and values, without conforming to any class.

Here we construct an anonymous object with FirstName, LastName, and Age properties:
```javascript
// Other code
[HttpGet]
[Route("displayanon")]
public JsonResult DisplayAnon()
{
    var AnonObject = new {
    	FirstName = "Raz",
	LastName = "Aquato",
    	Age = 10
    };
    
    return Json(AnonObject);
}
```
Anonymous objects aren't just used to format information for JSON responses, they can be used anywhere you need them. They are best used in moderation however, defining classes provides much better code readability in most situations. If you ever find yourself constructing multiple anonymous objects with the same set of properties you definitely need a class!
***

# IActionResult

Note that each of our main Controller's response types - ViewResult, RedirectToActionResult, and JsonResult - limit our controller actions to only one of those three responses.  But what if we want to either respond with a View or a Redirect based on some condition?

Thankfully, there is an interface that is implemented by all action results - including, but not limited to, ViewResult, RedirectToActionResult, and JsonResult: IActionResult. Here's another wonderful example of how you can utilize OOP principles in ASP! Based on the fact that they all implement IActionResult, we can use this interface as the return type of your actions and return any conforming response.

```javascript
// In your controller class
[HttpGet("")]
public ViewResult Index()
{
    return View();
}
[HttpGet("{favoriteResponse}")]
public IActionResult ItDepends(string favoriteResponse)
{
    if(favoriteResponse == "Redirect")
    {
    	return RedirectToAction("Index");
    }
    else if(favoriteResponse == "Json")
    {
        return Json(new {FavoriteResponse = favoriteResponse});
    }
    return View();
}
```
***
# Static Files

Static files (css, javascript, images) aren't compiled the way that our .cs files are, and by default they aren't accessible to our app while it's running. In order to add styling and javascript to our apps, we need to provide some additional configuration to our Startup class.

```javascript
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    // other configurations
    // We will now add the UseStaticFiles() middleware, which allows our app to access our static files.
    app.UseStaticFiles();
    app.UseMvc();
}
```
ASP.NET Core Mvc apps will look for static files in a folder called ```wwwroot``` which we'll need to create ourselves within our main project folder. By convention ```wwwroot``` should contain subfolders for different types of static content. For example all of our css files should be in a subfolder called "css":

![alt text](http://s3.amazonaws.com/General_V88/boomyeah/company_209/chapter_3759/handouts/chapter3759_7239_AddingWwwroot.PNG)

Now that our application is configured properly, we can start using those static files in our views. Because our app knows by default where to look for our static files we only need to specify their location in relation to wwwroot:

**Index.cshtml**

```javascript
<!DOCTYPE html>
<html>
    <head>
        <meta charset='utf-8'>
        <title>Index</title>
        // In this context '~' refers to the wwwroot folder
        <link rel="stylesheet" href="~/css/style.css"/>
    </head>
    <body>
        <h1>Hello ASP.NET Mvc!</h1>
    </body>
</html>
```
***

# ViewBag

One way we can send data from our controllers to our views is with an object called ```ViewBag```. ViewBag is a dictionary that only persists over one view return, and does not persist across redirects, so it must be set in the method rendering the view we're sending data to. To use ViewBag, we define properties and assign values to them. In the following example, we're defining the property Example with the value "Hello World!":

**YourController.cs**

```javascript
//Other code
 
[HttpGet]
[Route("")]
public IActionResult Index()
{
    // Here we assign the value "Hello World!" to the property .Example
    // Property names are arbitrary and can be whatever you like
    ViewBag.Example = "Hello World!";
    return View();
}
```
When the resulting view is rendered, it will have access to the ViewBag property that we set.

**Index.cshtml**

```javascript
<h1>@ViewBag.Example</h1>
```
We can also use ViewBag in our front-end C# code blocks, although in general it is preferable to perform as much logic in the back-end as we can in order to keep it centralized.

**Index.cshtml**

```javascript
@{
    string LocalString = ViewBag.Example + " Good to see you!";
}
<h1>@LocalString</h1>
```

***


# Portfolio I
**Objectives**

* Get familiar with setting up ASP.NET Core project
* Get familiar with Controller routing
* Begin work on your personal portfolio site


Setting up a web application with the ASP.NET Core framework is almost as simple as it was to build a console application. After navigating to where you'd like your project to be saved, we will use the web argument (instead of console) with our dotnet new command:

Let’s set up three basic routes for a potential portfolio page. For now, let’s just have the routes return string values to make sure we’ve got the routes set up properly. We’ll update this project in the next assignment, once we know how to render actual html pages.


![Alt text](https://s3.amazonaws.com/General_V88/boomyeah2015/codingdojo/curriculum/content/chapter/PortfolioI.png)

# Starting a Project **BEGIN HERE**

**terminal**

```javascript 
dotnet new web --no-https -o ProjectName
```

**What's with the _--no-https_?**

By default, ASP will want to run your web applications using HTTPS. This is great for security, but will create some friction upfront - as you would need to then generate https certs for your local browsers. For now, it's best to turn this option off.

***Program.cs***

```javascript
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```
As is, running ```dotnet run``` will deliver a "Hello World!" message at localhost:5000.

# Error Feedback

Another tool available to us, particularly for when we don't know where exactly our app is failing. To receive more explicit error messages about the state of our app, we just need to make sure our work environment is set to **Development mode**. Notice in the Configure method of our **Startup file (Startup.cs)** the following condition:
```javascript

    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

```
Production mode, which is the version of our project we want our end users to see. We wouldn't want our users to see in plain text our erroneous lines of code (i.e. a Developer Exception Page), but we certainly want to see them!

To put our current work environment into Development mode, we need to change an environment variable for .NET.

Enter into the terminal:
```javascript
    export ASPNETCORE_ENVIRONMENT=Development
```

_#### After changing any environment variables, you should always restart any terminals you have open (including VS Code).###_

**Dotnet Watch**

One more helpful tool is the ```Dotnet Watcher```. Running our application with this tool tells our project to automatically rebuild and run after we make changes to our ```.cs``` files. Though not crucial, this tool can save us some time during our learning experience. 

To run the Watcher tool, simply run:

```dotnet watch run```

**Adding MVC**

Do you remember where we go to configure our application? In the Startup class, we'll need to do two things to configure this piece of middleware. First, we need to tell our application that we'll be using the MVC service (more on services later!). Then we need to tell it when we want to actually use that service:

```javascript
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();  //add this line
    }
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // some code removed for brevity
        app.UseMvc();   //add this line, replacing the app.run lines of code
    }
```

---
# Services and Dependency Injection

***Overview***

It is now time to better understand all that is going on inside of our Startup class's ConfigureServices method, which is, in fact, a central aspect of how this framework is wired. Every time we add a new line to this method, with something like:

```javascript
    public static void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();
    }
```
* we are adding a so-called "service" into our application's Service Container, seen here as IServiceCollection.
  * This makes the specified service available to the rest of your application in a number of ways. In this example here, we are simply saying that our app wants to make use of the framework-provided MVC services, which, by and large, are handled internally by the framework itself to make the whole thing go. 

***Dependency Injection***

The basic premise of dependency injection is that rather than make tightly-coupled associations between classes using inheritance, you can provide objects that any class might need, known here as **services**, to a shared **container**, and anytime a class requires one of these services, you can **inject** that object into the class. Now what does this all actually look like in your code? You can use a type of DI, called **Constructor Injection**, to bring any service into your class by accepting an object of the desired service type into the Constructor method of that class.

To see an example of this, let's bring a service into our Startup class's constructor. One you will be working with soon is a framework-service called ```IHostingEnvironment```, which is an object that will tell you more about the machine that is hosting the application. While in development, this will be your local PC; when you deploy, this will be a remote web server.
```javascript
    public class Startup
    {
        // other code in Startup
        public Startup(IHostingEnvironment env)
        {
            // run this in the debugger, and inspect the "env" object! You can use this object to tell you // the root path of your application, for the purposes of reading from local files, and for
            // checking environment variables - such as if you are running in Development or Production
            
            Console.WriteLine(env.ContentRootPath);
            Console.WriteLine(env.IsDevelopment());
        }
    }
```
# Controllers

The controllers will be where we manage routing. All controllers, by convention, must be found in a folder called Controllers. A good rule of thumb is to have a separate controller for each major kind of model our application will handle. For example, a "UsersController" should handle creating users, logging in/out, etc, while a "ProductsController" would be responsible for creating and modifying products. For our first example, we'll just have a controller called "HelloController":

![Alt text](http://s3.amazonaws.com/General_V88/boomyeah/company_209/chapter_3759/handouts/chapter3759_7232_HellowWorldStructure.PNG)

The basic structure of each controller file will look like this:

**HelloController.cs**
```javascript
    using Microsoft.AspNetCore.Mvc;
    namespace YourNamespace.Controllers     //be sure to use your own project's namespace!
    {
        public class HelloController : Controller   //remember inheritance??
        {
            //for each route this controller is to handle:
            [HttpGet]       //type of request
            [Route("")]     //associated route string (exclude the leading /)
            public string Index()
            {
                return "Hello World from HelloController!";
            }
        }
    }
```

There may seem like a lot going on here, but it will help to remember we are seeing an example of applied OOP.  HelloController is simply a class inheriting from the MVC Controller class, and thus it receives all the functionality it needs to perform its central duties as a Controller: to intercept incoming requests, and to provide server responses to those requests.

***Actions***

ASP has a special name for a Controller method that delivers a server response: Action.  And while you may include any number of fields, methods, and properties in your Controller classes, they will mostly be comprised of actions: such as Index in the provided example.  The two parts that make up an action - what separates it from any other method - is that it is matched with a route and that it delivers a valid response.  A string is a valid response, and will be served as text content to the client.  By virtue of the two lines of square bracket notation directly above (more on this shortly!), GET requests to localhost:5000/ will be matched with our Index action, which in turn will deliver back its text response.


![Alt text](https://s3.amazonaws.com/General_V88/boomyeah2015/codingdojo/curriculum/content/chapter/hello-from-controller.png)

---
# Routing

The ASP library supports a few ways to handle routing, but in this course we will use something called _Attributes_ to do this.  The reason for this is that _Attribute_ Routing allows us to be explicit and exact about which routes, and which types of requests, we are mapping to our actions.  

_Attribute_ syntax is comprised of square brackets that are listed above - and apply added functionality to - classes and class members (such as our Controllers and Controller actions).  _Attributes_ are special types of classes that will have the word Attribute, in the class name; however when we use them, we can omit the word _Attribute_.

**[RouteAttribute]**

We can match specific url strings to our Controller actions by providing a RouteAttribute above the method itself.  We provide this attribute a string for the url we are looking to handle (leading slash not required).  For example, **[Route("")]** will match ```localhost:5000```, or the root url.  **[Route("users")]** will match ```localhost:5000/users```, and **[Route("users/more"]** will match ```localhost:5000/users/more```.  Make sure each route you list is unique!  You will receive a server error if two identical routes are used.

**[HttpMethodAttribute]**

We can match specific HTTP method types to our Controller actions, much like we do with a RouteAttribute, by listing the particular HTTP verb attribute above the method.  If we are looking to handle a GET request, for example, we can use **[HttpGet]**.  POST requests are handled by **[HttpPost]**, and so on.  If no HttpMethodAttribute is listed, and a RouteAttribute is provided, GET requests will be used as the default HTTP method type for that action.

**[HttpMethodAttribute(Route)]**

If you wish, you can combine a RouteAttribute and a HttpMethodAttribute, by providing a route string to a HttpMethodAttribute.  For example **[HttpGet("")]** will be matched with GET requests to ```localhost:5000```.

Let's see some examples for this to sink in:

**YourController.cs**

```javascript
// Other code
[HttpGet]
[Route("")]
// GET requests to "localhost:5000" go here
public string Index()
{
    // Method body
}
[HttpGet("about")]
// GET requests to "localhost:5000/about" go here
public string About()
{
    // Method body
}
[HttpPost]
[Route("submission")]
// POST requests to "localhost:5000/submission" go here
// (Don't worry about form submissions for now, we will get to those later!)
public string FormSubmission(string formInput)
{
    // Method body
}
```
**Route Parameters**

```javascript
// Other code
[HttpGet]
[Route("{name}")]
public string Index(string name)
{
    return $"Hello {name}!";
}
[HttpGet]
[Route("pizza/{topping}")]
public string PizzaParty(string topping)
{
    return $"Who's ready for a {topping} Pizza!";
}
[HttpGet]
[Route("user/{name}/{location}/{age}")]
public string UserInfo(string name, string location, int age)
{
    return $"{name}, aged {age}, is from {location}";
}
```
---