---
title: "Scala, sbt, Cucumber and selenium-webdriver: Part Two"
categories:
  - Blog
tags:
  - Scala
  - SBT
  - WebDriver
  - Cucumber
  - Page Objects
---

## Background / The Problem
In the previous example we stepped through a very basic guide to creating your first scala, cucumber and webDriver test. This may have been enough to get you started, but there’s a lot of room for improvement. If you have struggled to build upon the example outlined in part-one, and would like to build a maintainable framework rather than just a test, read on.<br>

## The Solution
We abstract all page level actions, and elements into [Page Objects](https://martinfowler.com/bliki/PageObject.html) and utilize selenium web-diver’s PageFactory. For example each page of your application has it’s own ‘page’ class which lists all locators and common methods related to these pages. This way when your app code changes causing test failures you will only have to update your test code in one place.<br>

## Example
In this example we will expand upon my previous post and upgrade it by abstracting the webDriver instance and create page-objects.<br>

### Driver Class
In order to use a single instance of webDriver we abstract the driver and wrap it within an object, often called a webDriver singleton. The purpose of this is to control object creation, limiting the number of objects to one.
```scala
object Driver {
 
   var driver: WebDriver = _
 
   def getDriver: WebDriver = {
     if (driver == null) {
     Firefox.firefoxProfile.setAcceptUntrustedCertificates(true)
     driver = Firefox.webDriver
    }
    driver
   }

}
```

### The Base class
Next we implement a base-class.
```scala
object Base {
 
  val BASE_URl     = "https://wordpress.com"
 
  val LOAD_TIMEOUT = 30
 
  val REFRESH_RATE = 2
 
}
 
abstract class Base[T] {
 
  lazy val webDriver = getDriver
 
  def initPage(clazz: Class[T]): T = {
    val page = PageFactory.initElements(getDriver, clazz)
    val pageLoadCondition = page.asInstanceOf[Base[T]].getPageLoadCondition
    waitForPageToLoad(pageLoadCondition)
    page
  }
 
  def navigateToPage(url: String) {
    getDriver.navigate().to(BASE_URl + url)
  }
 
 private def waitForPageToLoad(pageLoadCondition: ExpectedCondition[WebElement]) {
    val wait = new FluentWait(getDriver).withTimeout(LOAD_TIMEOUT, TimeUnit.SECONDS)
      .pollingEvery(REFRESH_RATE, TimeUnit.SECONDS)
    wait until pageLoadCondition
  }
 
  def getPageLoadCondition: ExpectedCondition[WebElement]
 
}
```

The Base object and Base class are common to all pages, including methods and page level helpers that each page-object inherit.<br>

The base object is fairly self explanatory, listing fields for your base URL, load timeout etc.<br>

The Base class includes the Base object and contains a method `initPage`, which will pass an instance of `webDriver` from the `getDriver` method created above in order to initialize a page-object (for example Login page) and any WebElement fields that are listed within that page-object.<br>

Next we have a `waitForPageToBeLoaded` method, which waits for a condition, for example an element to be visible on page before proceeding. This condition will be created from within our page-objects (more below).

### The Page Class
```scala
class Login extends Base[Login] {
  def initPage: Login = {
    navigateToPage("/login")
    new Login().initPage(classOf[Login])
  }
 
  @FindBy(id = "user_login")
  var loginField: WebElement =_
 
  @FindBy(id = "user_pass")
  var passwordField: WebElement =_
 
  @FindBy(id = "wp-submit")
  var loginButton: WebElement =_
 
  @FindBy(id = "login_error")
  var loginError: WebElement =_
 
  override def getPageLoadCondition: ExpectedCondition[WebElement] = {
    ExpectedConditions.visibilityOf(loginField)
  }
 
  def login(userName: String, password: String) {
    loginField.sendKeys(userName)
    passwordField.sendKeys(password)
    loginButton.click()
  }
}
```

The above code snippet is a simple example of the WordPress Login page.

- First of all we name our page class, naming it by the page name of your application under test. In this case the first page that we navigate to on the WordPress website is the Login page, hence why I have named the class Login.
- The Login page extends (inherits) the Base class created above. The Login page is a subclass of Base, i.e. Base[Login] (more information on Type and polymorphism here) This way the initPage method and wait condition are executed when the Login page is initialized.
- Next we use PageFactory to declare some fields, listing page level elements (for example submit button).
- In order to wait for this page to be loaded, we create our page load condition method. In this case we are waiting for the visibility of the login field.
Next we create any common methods related to the page, in this case we have a common login method, passing userName and password as String type parameters.

### The Step Definitions Class
Once we have created our page objects, we are now ready to access them from within our Step Definitions.

The page-objects must declared with a [lazy val](https://stackoverflow.com/questions/7484928/what-does-a-lazy-val-do).

For example:
```scala
class MyFirstPageObjectTestSteps extends ScalaDsl with EN {
 
  lazy val wpLogin = new Login().initPage
  lazy val wpHome  = new Dashboard().initPage
}
```

The reason why we use `lazy val` is; we only want to initialize a page when we need to. Otherwise it will cause tests to fail as the pages will be initialized each time we execute a test, or a step definitions class is used due to waiting for page level elements in our page classes (`getPageLoadCondion` method). Essentially pages should only accessed if, and when we require them.

### Accessing page-objects from Step Definitions

```scala
class MyFirstPageObjectTestSteps extends ScalaDsl with EN {
 
  lazy val wpLogin = new Login().initPage
  lazy val wpHome  = new Dashboard().initPage
 
  Given( """^I login with (valid|invalid) credentials to Add WordPress.com blog$""") { (negate: String) =>
    if (negate.equals("valid")) {
      wpLogin.login("info@example.co.uk", "valid")
    } else {
      wpLogin.login("info@example.co.uk", "invalid")
    }
  }
 
  When( """^I (should|should not) be logged in$""") { (negate: String) =>
    if (negate.equals("should")) {
      assertTrue("Wordpress Dashboard not displayed", wpHome.blogTitle().contains("infoatrubygemtsl"))
    } else {
      assertTrue("Error message was not displayed", wpLogin.loginErrorMessage().equals("ERROR: The password you entered for the email or username info@rubygemtsl.co.uk is incorrect. Lost your password?"))
    }
  }
}

```

## Summary
Following the above example you should be well on your way to building a robust, well maintainable test framework. It is important to remember that the more time invested in building maintainable frameworks, the better. A reliable test suite will enable tests to be executed on demand/periodically via Continuous Integration in the run up to a release, as well as freeing a tester use their knowledge and testing skills execute exploratory based testing.<br>

The natuaral next steps for a useful and maintainable test pack would be to extend the driver class to cover cross-browser and mobile web based testing, as well as introduce some of the features of cucumber such as Before/After hooks.