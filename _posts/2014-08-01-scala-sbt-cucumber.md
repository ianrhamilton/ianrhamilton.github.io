---
title: "Scala, sbt, Cucumber and selenium-webdriver: Part One"
categories:
  - Blog
tags:
  - Scala
  - SBT
  - WebDriver
  - Cucumber
---

## Background
Recently I started a contract where I was asked to build a maintainable automation framework using Sbt, Scala, cucumber and webDriver for a web application based on the Play framework. Although I have lots of experience of web based projects using Ruby, Cucumber and webDriver, I’ve never used (or even thought of using) Scala.<br>

After a few days of pain I got there in the end and I have now fully integrated cucumber into my project. I found it very difficult to find any recent online resources, blogs etc that would help with this set up. If your looking to set this up yourself, then hopefully you will find this example useful, and read on...<br>

## Prerequisite
[IntelliJ IDEA Comminuity Edition](https://www.jetbrains.com/idea/?host=intellij.co.uk) is great as a tool to support automated test development. It supports Scala and offers a scala-cucumber plugin, enabling us to write our cucumber feature files using the Gerkin syntax (Given, When, Then, And, But), as well as providing syntax and error highlighting for feature files and step definitions.<br>

## IntelliJ Setup
- Open InteliJ:
<p align="center">
  <img width="600" height="200" src="/assets/images/intelij-open.png">
</p>

- Select Configure:
<p align="center">
  <img width="600" height="200" src="/assets/images/intelij-configure.png">
</p>

- Select Plugins:
<p align="center">
  <img width="600" height="200" src="/assets/images/intelij-plugins.png">
</p>

- Select Browse Repositories…, enter “cucumber into the search field:
<p align="center">
  <img width="600" height="200" src="/assets/images/plugins-browser.png">
</p>

Install the Cucumber for Scala plugin and restart InteliJ.

## Creating your first sclala project
Once you have installed the required plugins, open InteliJ.

- Select "Create New Project" 
- Select "Scala, and SBT"
- Click next

If you **do not** have these options you may need to **install the sbt plugin**.

- Name your project, for example `my_first_cucumber_test`
- Select "Finish"

InteliJ will open and create all of the required files and folders for us, including the build.sbt:
<p align="center">
  <img width="600" height="200" src="/assets/images/first-project.png">
</p>

Now we have the project set up, we can go ahead and delete anything that we don’t need, Go to the `src/test` directory and delete any `java` directories, keeping the `scala` directories.

<p align="center">
  <img width="600" height="200" src="/assets/images/resources.png">
</p>

## Project set up
Open the build.sbt folder. When prompted with the following message, select "Enable auto- import".
<p align="center">
  <img width="600" height="200" src="/assets/images/auto-import.png">
</p>

Next, add dependencies for scalatest, cucumber, junit, and selenium-webdriver to your `build.sbt` file.

```sbt
name := "my_first_cucumber_test"
 
version := "1.0"
 
libraryDependencies ++= Seq(
  "org.scalatest" % "scalatest_2.11" % "2.2.0" % "test",
  "org.scala-lang" % "scala-library" % "2.11.1",
  "info.cukes" % "cucumber-scala_2.11" % "1.1.8",
  "info.cukes" % "cucumber-junit" % "1.1.8",
  "info.cukes" % "cucumber-picocontainer" % "1.1.8",
  "junit" % "junit" % "4.11" % "test",
  "com.novocode" % "junit-interface" % "0.10" % "test",
  "org.seleniumhq.selenium" % "selenium-java" % "2.42.2")
```

Once dependencies are installed we’re ready to create our first feature file.

### Writing Features
Features are written in a plain text format named Gerkin. Feature files live in the `src/test/resources` directory. Navigate to `resources` directory, right click and select "new directory" and create a `features` folder.

Now we have our features folder we can go ahead and create our first feature file.
Right click, select "new file" and give it a name ending in .feature, like the following:

<p align="center">
  <img width="600" height="200" src="/assets/images/new-feature.png">
</p>

```gherkin
  Feature: My First cucumber feature
 
  As a tester,
  I would like to utilize cucumber,
  So that I can create BDD style selenium-webdriver tests.
 
  Scenario: Google search, using selenium
    Given I have navigated to google
    When I search for "selenium"
    Then the page title should be selenium - Google Search

```

Now you have created your first Feature, well done! Now lets write some code...

## Step Definitions
Navigate to the `test/scala` directory, right click and create a new package called `stepdefs`.

<p align="center">
  <img width="600" height="200" src="/assets/images/new-feature-2.png">
</p>

Now we must create a cucumber runner class in order to execute our tests and generate our step definitions. Right click on the step definition package and select new "Scala class" and add the following:

```scala
package stepdefs
 
import cucumber.api.junit.Cucumber
import cucumber.api.junit.Cucumber.Options
import org.junit.runner.RunWith
 
@RunWith(classOf[Cucumber])
@Options(
  features = Array("src/test/resources/features"),
  glue = Array("stepdefs"),
  format = Array("pretty", "html:target/cucumber-report"),
  tags = Array("@wip")
)
class RunCucumber {
}
```

As the first scenario is "work in progress", go ahead and tag your first scenario as `@wip` so it can be picked up when executing the cucumber runner that we just created.

<p align="center">
  <img width="600" height="200" src="/assets/images/new-feature-3.png">
</p>

### Execute the tests
Go to the cucumber runner file that we created, right click and select "Run Cucumber"
<p align="center">
  <img width="600" height="200" src="/assets/images/execute-tests.png">
</p>

The scenario will now be executed, however as there are no step definitions the scenario cannot be ran, it will execute and spit out the empty step definitions:
<p align="center">
  <img width="600" height="200" src="/assets/images/passing-test.png">
</p>

For some reason I get java and scala step definitions. Ignore them, we only need the scala ones (obviously):
<p align="center">
  <img width="600" height="200" src="/assets/images/new-step-defs.png">
</p>
Now create new scala class within the stepdefs package:
<p align="center">
  <img width="600" height="200" src="/assets/images/steps-defs-class.png">
</p>
Now we can simply copy/paste our new step definitions, and add required plugins. Step definitions must extend Scala DSL with EN. Like the following:
```scala
package stepdefs
 
import cucumber.api.PendingException
import cucumber.api.scala.{ScalaDsl, EN}
import org.scalatest.Matchers
 
class MyFirstCucumberTest extends ScalaDsl with EN with Matchers {
 
  Given("""^I have navigated to google$"""){ () =>
    //// Write code here that turns the phrase above into concrete actions
    throw new PendingException()
  }
 
  When("""^I search for "(.*?)"$"""){ (arg0:String) =>
    //// Write code here that turns the phrase above into concrete actions
    throw new PendingException()
  }
 
  Then("""^the page title should be selenium - Google Search$"""){ () =>
    //// Write code here that turns the phrase above into concrete actions
    throw new PendingException()
  }
 
}
```

Simply fill in the in the pending step definitions with selenium commands, something the following:

```scala
package stepdefs
 
import java.util.concurrent.TimeUnit
 
import cucumber.api.scala.{ScalaDsl, EN}
import org.openqa.selenium.By
import org.openqa.selenium.firefox.FirefoxDriver
import org.scalatest.Matchers
 
class MyFirstCucumberTest extends ScalaDsl with EN with Matchers{
 
  val driver = new FirefoxDriver()
  driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS)
 
  Given("""^I have navigated to google$"""){ () =>
    driver.navigate().to("http://www.google.com")
  }
 
  When("""^I search for "(.*?)"$"""){ (searchTerm:String) =>
    driver.findElement(By.id("gbqfq")).sendKeys(searchTerm)
    driver.findElement(By.id("gbqfb")).click();
  }
 
  Then("""^the page title should be "(.*?)"$"""){ (title:String) =>
    driver.getTitle.shouldEqual(title)
  }
 
}
```
It is also possible to execute our tests via command line: sbt ‘test-only stepdefs.RunCucumber’

### Summary
What I’ve covered here is a very basic guide to getting started, however hopefully it is enough to get you started. If not, ask, I’m happy to help where possible.<br>

Now we’re up and running the next step is to increase maintainability by abstracting the selenium driver, and build our page objects.<br>

As usual, all example code can be found at my GitHub, [here](https://github.com/ianrhamilton/scala-cucumber-webdriver-example).