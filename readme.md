# TestStack.Seleno

## What is Seleno?
Seleno helps you to write automated UI tests in the right way by implementing Page Objects and Page Components and by reading and writing web page data using strongly typed view models. It uses Selenium Web Driver for browser automation.

## How do I use it?

1. Install-Package TestStack.Seleno
	* If you are using ASP.NET MVC then there are some helper methods for that that require you to install MVC if you want to use them: Install-Package Microsoft.AspNet.Mvc
	* If you installed MVC then you will also need to add binding redirects: Add-BindingRedirect

2. Create an assembly-level test fixture (if your unit test library supports it, otherwise creating a normal test fixture that will be guaranteed to run first should be enough because Seleno unloads itself when the app domain finishes) that looks something like this NUnit / ASP.NET web application example:

        [SetUpFixture]
        public class AssemblyFixture
        {
            [SetUp]
            public void SetUp()
            {
                SelenoHost.Run("Name.Of.Your.Web.Project", 12346, c => c
                    .UsingLoggerFactory(new ConsoleFactory())
                );
                // If you are using MVC then do this where RouteConfig is the class that registers your routes in the "Name.Of.Your.Web.Project" project
                // If you aren't using MVC then don't include this line
                RouteConfig.RegisterRoutes(RouteTable.Routes);
            }
        }
	* The `123456` is the port number you want the site to run on - it can be anything you want, just make it unique and unused
	* The `c` variable is a fluent configurator - chain method calls off of it to configure the different parts of Seleno
	* By default it uses Firefox so you will need to install that
	* You might need to run Visual Studio / your test runner as an admin if you can an error when the port tries to get registered by IIS Express

3. Create page objects by extending `Page`, or if you want to use strongly-typed view models, `Page<T>`, e.g.:

        public class HomePage : Page
        {
            public Form1Page GoToRegisterPage()
            {
                return Navigate().To<RegisterPage>(By.LinkText("Register"));
            }
        }
        
        public class RegisterPage : Page<RegisterModel>
        {
            public HomePage RegisterUser(RegisterModel registerModel)
            {
                Input().Model(registerModel);
                return Navigate().To<HomePage>(By.CssSelector("input[type=submit]"));
            }
        }
	* Seleno provides a DSL that hides you from most of Selenium Web Driver, feel free to make use of intellisense within your page object to experiment with what's possible
	* There are some links to advanced usage instructions and tutorials below

4. If you want to wrap common components of your pages then create components by extending `UiComponent`, e.g.:

        public class HomePage : Page
        {
            public RegisterPage GoToRegisterPage()
            {
                return Navigate().To<RegisterPage>(By.LinkText("Register"));
            }
    
            public LoginPanel LoginPanel
            {
                get { return GetComponent<LoginPanel>(); }
            }
        }
        
        public class LoginPanel : UiComponent
        {
            public bool IsLoggedIn
            {
                get { return Find().OptionalElement(By.Id("login-panel")) == null; }
            }
    
            public string LoggedInUserName
            {
                get { return Find().Element(By.Id("login-username")).Text; }
            }
        }
        
        public class RegisterPage : Page<RegisterModel>
        {
            public HomePage RegisterUser(RegisterModel registerModel)
            {
                Input().Model(registerModel);
                return Navigate().To<HomePage>(By.CssSelector("input[type=submit]"));
            }
        }

5. Create automated tests that use your page objects, e.g. this NUnit example:

        class RegistrationTests
        {
            [Test]
            public void GivenAUserIsntRegistered_WhenRegisteringThem_TheyEndUpBackOnTheHomepageAndLoggedIn()
            {
                var page = SelenoHost.NavigateToInitialPage<HomePage>()
                    .GoToRegisterPage()
                    .RegisterUser(ObjectMother.NewUser);
    
                Assert.That(page.Title, Is.EqualTo("Home"));
                Assert.That(page.LoginPanel.IsLoggedIn, Is.True);
                Assert.That(page.LoginPanel.LoggedInUserName, Is.EqualTo(ObjectMother.NewUser.UserName));
            }
        }

## Tutorials / advanced usage
Stay tuned!

## Background information on Page Objects

### What are Page Objects?
The Page Object design pattern is a way of encapsulating the interaction with an individual page in your application in a single object. It makes heavy use of OO principles to enable code reuse and improve maintenance. Rather than having tests that are a series of Selenium commands that are sent to the server, your tests become a series of interactions with objects that represent a page (or part of one).

### How does it work?
The usage of the Page Object design pattern creates a strong separation of concerns between  tests and Page Objects. The tests specify what should happen and the Page Objects encapsulate how it happens. 
* Tests are very procedural. They only interact with the Page Objects and make assertions. They should not have any implementation details, such as Selenium calls, whatsoever. 
* Page Objects encapsulate all of the interaction with Selenium, and all of the logic of that interaction. There are no test assertions in the Page Objects but they can throw exceptions.

### What are the benefits of using Page Objects?
* Separating test specification from test implementation makes tests more robust. If you change how a feature works, you just have to change the test, rather than every test that uses the feature.
* Maintenance is easier because you only have to change things in one place.
* Tests are more readable as they just work with Page Objects and make assertions. They do not have any Selenium code as this is hidden away in the Page Object. 

## Authors and Contributors

### Authors
* Mehdi Khalili (@MehdiK)
* Michael Whelan (@mwhelan)

### Contributors
* Rob Moore (@robdmoore)
* Franck Theolade (@Gwayaboy)
* Rhys Campbell (@RhysC)

## Problems / Contributions
If you have any problems with Seleno, please [raise an issue](https://github.com/TestStack/TestStack.Seleno/issues). We are also happy to take pull requests; if you are thinking of contributing a major change feel free to raise an issue first and we can talk through and provide assistance.
