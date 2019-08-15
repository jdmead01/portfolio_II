# Portfolio I
**Objectives**

* Get familiar with setting up ASP.NET Core project
* Get familiar with Controller routing
* Begin work on your personal portfolio site






***Getting Started***

Setting up a web application with the ASP.NET Core framework is almost as simple as it was to build a console application. After navigating to where you'd like your project to be saved, we will use the web argument (instead of console) with our dotnet new command:

Let’s set up three basic routes for a potential portfolio page. For now, let’s just have the routes return string values to make sure we’ve got the routes set up properly. We’ll update this project in the next assignment, once we know how to render actual html pages.


![Alt text](https://s3.amazonaws.com/General_V88/boomyeah2015/codingdojo/curriculum/content/chapter/PortfolioI.png)


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
