---
title: "Part One: Executing tests on Sauce Labs using *fastlane* and *saucectl* - XCUITest"
categories:
  - Blog
tags:
  - Sauce Labs
  - iOS
  - XCUITest
  - fastlane
  - fastlane-plugin-saucectl
---
  
This post is part one of a series of blog posts where I will explain how you can to utilise ***[fastlane](https://fastlane.tools/)*** and [Sauce Labs](https://docs.saucelabs.com/) for the execution of XCUITests on the Sauce Real Device Cloud platform.

In part one you’ll learn the following:
1. Set up *fastlane*
2. Install *fastlane-plugin-saucectl*
3. Upload application `ipa` files to Sauce Labs storage
4. Configure your tests for Sauce Labs execution
5. Install `saucectl` agnostic test orchestrator
6. Execute tests on Sauce Labs Real Device platform

## Set up *fastlane*

***[fastlane](https://fastlane.tools/)*** lets you configure lanes to support your team’s deployment workflows. You can use a single command to run a lane.

### Installing *fastlane*
The preferred method to install ***fastlane***, is with [Bundler](https://bundler.io/). ***fastlane*** supports Ruby versions 2.5 or newer, please ensure you have the minimum version of newer installed.

It is recommended that you use Bundler and create a Gemfile to define your dependency on ***fastlane***. This will clearly define the ***fastlane*** version to be used and its dependencies and will also speed up fastlane execution.

To install Bundler execute the following within terminal:
```shell
   gem install bundler
```

Next, within he root directory of your XCode project create a Gemfile.
```shell
  cd your_ios_app_project
  touch Gemfile
```

Add the following content to the newly created Gemfile:
```ruby
    source "https://rubygems.org"
    gem "fastlane"
```

Next to install fastlane and create `Gemfile.lock` within terminal execute the following: 
```shell
   bundle install
```

Navigate to your iOS application project and run:
```shell
  fastlane init
```

fastlane will automatically detect your project, and ask for any missing information. If the installation was sucessful a ***fastlane***  directory will be created, like the following:

<p align="center">
  <img width="600" height="200" src="/assets/images/fastlane-installation.png">
</p>

## Install fastlane-plugin-saucectl

To install the `fastlane-plugin-saucectl`, navigate to your iOS application project run:

```shell
  fastlane add_plugin saucectl
```

Now that the fastlane-plugin-sauce dependency is installed, you should see an auto-generated  `PluginFile`.
<p align="center">
  <img width="2000" height="800" src="/assets/images/install_fastlane_saucectl.png">
</p>

🎉 You're now ready to start prepping your tests for Sauce Labs execution. 

## Using fastlane-plugin-saucectl
I assume if you're reading this post that you have already set up your UI Test project within Xcode, and I will not go into detail on this subject within this post. 
However, there are some requirements that you must satisfy in order to use the features of the fastlane-plugin-saucectl plugin.

### Test/Spec classes
`fastlane-plugin-saucectl` will scan your UI Test target for test classes, therefore your test class names must proceed with Spec, Specs, Tests, or Test. For example `ExampleSpec`, `ExampleSpecs`, `ExampleTest`, or `ExampleTests`.

```swift
import XCTest

class ExampleTests {
    
    func testExample() throws {
        // Test code
    }
}
```

## Set up your test lanes for Sauce Labs execution

🚫 Before you proceed with the next steps you must have built your application for real device testing. This will generate an `MyApplication.ipa` and `MyApplicationRunner.ipa`.

Navigate to your `fastlane/Fastfile` that you created earlier, and we can begin configuring our lanes for UI test execution via Sauce Labs.

`fastlane-plugin-saucectl` has the following available actions:

| Available Actions   | Description                                                                                                                                                                                                  |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `install_saucectl`    | Downloads the Sauce Labs saucectl cli binary for test execution. Optionally specify the [version](https://github.com/saucelabs/saucectl/releases/) you wish to install or automatically download the latest. |
| `sauce_upload`        | Upload test artifacts to sauce labs storage                                                                                                                                                                  | 
| `sauce_config`        | Create SauceLabs configuration file for test execution based on given parameters                                                                                                                             |
| `sauce_runner`        | Execute automated tests on sauce labs platform via saucectl binary for specified configuration                                                                                                               | 
| `delete_from_storage` | Delete test artifacts from sauce labs storage by storage id or group id                                                                                                                                      |
| `sauce_apps`          | Returns the set of files by specific app id that have been uploaded to Sauce Storage by the requester                                                                                                        |
| `sauce_devices`       | Returns a list of Device IDs for all devices in the data center that are currently free for testing.                                                                                                         |
| `disabled_tests`      | Fetches any disabled ui test cases (for android searches for @Ignore tests, and for ios skipped tests within an xcode test plan). Plan is to use this in the future for generating pretty HTML reports       | 

## Upload test artifacts to sauce labs storage

The first lane within your `Fastfile` you can create is an `upload_to_sauce` lane. This lane will use the [`sauce_upload`](https://ianrhamilton.github.io/fastlane-plugin-saucectl/upload/) action from the `fastlane-plugin-saucectl`. 

Although the `saucectl` runner will automatically upload your `ipa` files as part of test execution, you may wish to upload apps as part of your development pipelines for manual live testing, or later delete your `ipa` files by a specific `id` - this lane can be very useful.

The `sauce_upload` action will return the `id` of the application from sauce storage.
```ruby
    desc "Upload app and test runner to Sauce Labs storage"
      lane :upload_to_sauce do
          @app_id = sauce_upload(platform: 'ios',
                                 app: 'SauceLabs-Demo-App.ipa',
                                 file: 'SauceLabs-Demo-App.ipa',
                                 region: 'eu',
                                 sauce_username: ENV['SAUCE_USERNAME'],
                                 sauce_access_key: ENV['SAUCE_ACCESS_KEY'])

          @test_app_id = sauce_upload(platform: 'ios',
                                      app: 'SauceLabs-Demo-App-Runner.XCUITest.ipa',
                                      file: 'SauceLabs-Demo-App-Runner.XCUITest.ipa',
                                      region: 'eu',
                                      sauce_username: ENV['SAUCE_USERNAME'],
                                      sauce_access_key: ENV['SAUCE_ACCESS_KEY'])
end
```

### Help

Information and help for the `sauce_upload` action can be printed out by running the following command:

```shell
  fastlane action sauce_upload
```

Now that your lane is complete, you can now upload your apps to sauce storage. You can do this by executing the following command:
```shell
    bundle exec fastlane ios upload_to_sauce
```

```shell
ian@Ians-MBP-2 my-demo-app-ios % bundle exec fastlane ios upload_to_sauce
[✔] 🚀 
+--------------------------+---------+---------------------------------------------------------------+
|                                            Used plugins                                            |
+--------------------------+---------+---------------------------------------------------------------+
| Plugin                   | Version | Action                                                        |
+--------------------------+---------+---------------------------------------------------------------+
| fastlane-plugin-saucectl | 0.1.3   | sauce_apps, disabled_tests, install_saucectl, sauce_devices,  |
|                          |         | delete_from_storage, sauce_upload, sauce_runner, sauce_config |
+--------------------------+---------+---------------------------------------------------------------+

[15:45:25]: ------------------------------
[15:45:25]: --- Step: default_platform ---
[15:45:25]: ------------------------------
[15:45:25]: Driving the lane 'ios upload_to_sauce' 🚀
[15:45:25]: --------------------------
[15:45:25]: --- Step: sauce_upload ---
[15:45:25]: --------------------------
[15:45:25]: ⏳ Uploading "SauceLabs-Demo-App.ipa" upload to Sauce Labs.
[15:45:27]: ✅ Successfully uploaded app to sauce labs: 
 {"item": {"id": "6a038d70-61ec-49b0-b62c-860ff55f8b01", "owner": {"id": "c1706f2cef92450d87218e658bad3e52", "org_id": "d34523477ebf4956aed6d043bedb1a91"}, "name": "SauceLabs-Demo-App.ipa", "upload_timestamp": 1650023127, "etag": "CMuYpJP/lfcCEAE=", "kind": "ios", "group_id": 1753787, "size": 3495570, "description": null, "metadata": {"identifier": "com.saucelabs.mydemoapp.ios", "name": "My Demo App", "version": "1", "is_test_runner": false, "icon": "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AAAAAAAAAMDlmDXWWGONNdZY4+Nf/ggAnl7eWmONNdZYY4011lhjjTXWWGONNdZYY4011lhjjTXWWGONNdZYY4011ni8uXY5DgC/fjz76dtvrLHGGmusscYaa6yxxhprrLHGp9f3AODpu1trrLHGGmussQYAAAAAAAAAAAAAwAkAAAAAAAAAAAAA4OHZlTXWWGONNdb423e3AAAAAAAAAAAAAAAAAOB898oaa6yxxhprrPF4c+1yHAB+/Xj207ffWGONNdZYY4011lhjjTXWWGONNS7HAQAeb66tscYaa6yxxocXzwEAAAAAAAAAAAAAnAAAAAAAAAAAAAAAPrx4bo011lhjjTUe/s8VAAAAAAAAAAAAAAAAgF8eZo011lhjjTXWeLy5djkOAL9+PPvp22+sscYaa6yxxhprrLHGGmusscYaf/3dbwHA/7x/Z4011lhjjTXW+PmH3wMAAAAAAAAAAAAAOAEAAAAAAAAAAAAAnO9eWWONNdZYY401Pr99AwAAAAAAAAAAAAAAAJ6+u7XGGmusscYajzfXLscBAD68eG6NNdZYY4011lhjjTXWWGONNdZY49////8BgKfvbq2xxhprrLHGGp/fvgEAAAAAAAAAAAAAcAIAAAAAAAAAAAAA+Pz2jTXWWGONNdZY4+m7W5fjcDkOl+NwOQ6X43A5DpfjcDkOl+NwOQ6X43A5DpfjcDkOl+NwOQ6X43A5DpfjcDkOv348A4BfHmaNNdZYY401Hm+uXY4DADy9vLXGGmusscYaa6yxxhprrLHGGmus8XhzDQDg8ebaGmusscYaazzeXAMAAAAAAAAAAAAAgBMAAAAAAAAAAAAAADw8u7LGGmusscYaa6yxxhprrLHGGmusscYaa6yxxhprrLHGGmtcjgMAPH13a4011lhjjceba5fjAABPL2+tscYaa6yxxhprrLHGGmusscYaa6zx6fU9APj047011lhjjTXWWOPp5S0AAAAAAAAAAAAAADgBAAAAAAAAAAAAAMDff/i9NdZYY4011lhjjTXWWGONNdZYY4011lhjjTXWWGONNdZ4enkLAH55mDXWWGONNR6eXbkcBwB4enlrjTXWWGONNdZYY4011lhjjTXWWGONx5trAACPN9fWWGONNdZYY43Pb98AAAAAAAAAAAAAAIATAAAAAAAAAAAAAAB8fvvGGmusscYaa6yxxhprrLHGGmusscYaa6yxxhprrLHGGmtcjgMAPH13a4011ljj4dmVL+/fAYCnl7fWWGONNdZYY4011lhjjTXWWGONNdZY43IcAODTj/fWWGONNdZYY43Hm2sAAAAAAAAAAAAAAAAnAAAAAAAAAAAAAACADy+eW2ONNdZYY4011lhjjTXWWGONNdZYY4011lhjjTXW+PmH7wHALw+zxhprrPHw7MqX9+8AwNPLW2usscYaa6yxxhprrLHGGmusscYaa6zxj7s/AAB4vLm2xhprrLHGGmt8en0PAAAAAAAAAAAAAADgBAAAAAAAAAAAAAAA8PHPf7TGGmusscYaa6yxxhprrLHGGmusscYaa6yxxhprPN5cuxwHAPjw4rk11ljj4dmVL+/fAYCnl7fWWGONNdZYY4011lhjjTXWWGONNdZY46dvvwEAcL57ZY011lhjjTXWeLy5BgAAAAAAAAAAAAAAACcAAAAAAAAAAAAAAICv57OHZ1fWWGONNdZYY4011lhjjTXWWGONNdZYY4011ljjH3d/AACf376xxhprPDy78uX9OwDw9PLWGmusscYaa6yxxhprrLHGGmusscYaazzeXLscBwD45WHWWGONNdZYY401/nH3BwAAAAAAAAAAAAAAAHACAAAAAAAAAAAAAACA890ra6yxxhprrLHGGmusscYaa6yxxhprrLHGGms83lz7ej4DgMeba2us8fDsypf37wDA33/4vTXWWGONNdZYY4011lhjjTXWWGONNdZ4eHblchwAAB5vrq2xxhprrLHGGo831wAAAAAAAAAAAAAAAABOAAAAAAAAAAAAAAAA8PV89nhzbY011lhjjTXWWGONNdZYY4011lhjjTXWWOPT63sA8OnHe2us8fDsypf37wDA+e6VNdZYY4011lhjjTXWWGONNdZYY4011nh4duXL+3cAAP7+w++tscYaa6yxxhprfHp9DwAAAAAAAAAAAAAAAOAEAAAAAAAAAAAAAAAA8PntG2usscYaa6yxxhprrLHGGmusscYaa6yxxuPNNQCAx5tra6zx7//6TwBwvntljTXWWGONNdZYY4011lhjjTXWWGONNR5vrl2OAwDA+e6VNdZYY4011lhjjZ/+4xsAAAAAAAAAAAAAAAAAcAIAAAAAAAAAAAAAAACADy+eW2ONNdZYY4011lhjjTXWWGONNdZYY41Pr+8BwKcf762xxqcf7wHA+e6VNdZYY4011lhjjTXWWGONNdZYY4011vjp229cjgMAwMc//9Eaa6yxxhprrLHGGpfjAAAAAAAAAAAAAAAAAAAnAAAAAAAAAAAAAAAAALgc8/DsyhprrLHGGmusscYaa6yxxhprrLHG4801AIDHm2trfPrxHgCc715ZY4011lhjjTXWWGONNdZYY4011lhjjZ9/+B4AAHz88x+tscYaa6yxxhprrPHPv/wJAAAAAAAAAAAAAAAAAMAJAAAAAAAAAAAAAAAAAODjX/5kjTXWWGONNdZYY4011lhjjTXWWOPT63sA8OnHe2t8+vEeAJzvXlljjTXWWGONNdZYY4011lhjjTXWWOPDi+c+v30DAADOd6+sscYaa6yxxhprrPH08hYAAAAAAAAAAAAAAAAAAJwAAAAAAAAAAAAAAAAAAODnH763xhprrLHGGmusscYaa6yxxhqPN9cAAB5vrn368R4AnO9eWWONNdZYY4011lhjjTXWWGONNdb48OK5z2/fAACAXz+e/f2H31tjjTXWWGONNdZY4/Hm2tfzGQAAAAAAAAAAAAAAAAAAnAAAAAAAAAAAAAAAAAAA4Ov57L+//cYaa6yxxhprrLHGGmusscYan17fA4CPf/6jTz/eA4Dz3StrrLHGGmusscYaa6yxxhprrPHw7MqHF899fvsGAADALw/z07ffWGONNdZYY4011ljj8eba5TgAAAAAAAAAAAAAAAAAAACcAAAAAAAAAAAAAAAAAAAALsc83lxbY4011lhjjTXWWGONNT68eA4AfnmYz2/fAIDz3StrrLHGGmusscYaa6yxxhqPv7n28w/f+9fre1/PZwAAAPCPu1cenl1ZY4011lhjjTXWWOPh2ZXLcQAAAAAAAAAAAAAAAAAAAIATAAAAAAAAAAAAAAAAAAAAXI55vLm2xhprrLHGGmusscYan9++AQAAAOe7V9ZYY4011ljj6eWtf72+96/X9/71+t6//+s/fXn/ztfzGQAAAAB8fvvG4821NdZYY4011lhjjTXWeHh25cv7dwAAAAAAAAAAAAAAAAAAAABOAAAAAAAAAAAAAAAAAAAAAJdjHm+urbHGGmusscYaazy9vAUAAADnu1fWWGONNdZY4+nlLQAAAAAAAIB//9d/+vDiuTXWWGONNdZYY4011ljj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AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAIATAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADACQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA4AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAHACAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA4AQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAnAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAE4AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAnAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAACA/wUbzcGiN2W89wAAAABJRU5ErkJggg==", "short_version": "1.3.0", "is_simulator": false, "min_os": "14.3", "target_os": "14.5", "test_runner_plugin_path": null, "device_family": ["phone", "tablet"]}, "access": {"team_ids": ["95adcc1eb0ab459c9c6f3aabcf129079"], "org_ids": []}, "sha256": "bfee551b6944a68d309ea7a48a3439dfc906723f5b6d98c12e1f565545d5be3e"}}
[15:45:27]: --------------------------
[15:45:27]: --- Step: sauce_upload ---
[15:45:27]: --------------------------
[15:45:27]: ⏳ Uploading "SauceLabs-Demo-App-Runner.XCUITest.ipa" upload to Sauce Labs.
[15:45:29]: ✅ Successfully uploaded app to sauce labs: 
 {"item": {"id": "dfa8c632-0d48-4be1-891e-8dd738e8967a", "owner": {"id": "c1706f2cef92450d87218e658bad3e52", "org_id": "d34523477ebf4956aed6d043bedb1a91"}, "name": "SauceLabs-Demo-App-Runner.XCUITest.ipa", "upload_timestamp": 1650023130, "etag": "CJeBvJT/lfcCEAE=", "kind": "ios", "group_id": 1754003, "size": 4261221, "description": null, "metadata": {"identifier": "com.saucelabs.mydemoapp.ios.My-Demo-AppUITests.xctrunner", "name": "My Demo AppUITests-Runner", "version": "1", "is_test_runner": true, "icon": null, "short_version": "1.0", "is_simulator": false, "min_os": "9.0", "target_os": "14.5", "test_runner_plugin_path": "PlugIns/My Demo AppUITests.xctest", "device_family": ["phone", "tablet"]}, "access": {"team_ids": ["95adcc1eb0ab459c9c6f3aabcf129079"], "org_ids": []}, "sha256": "5ef3133033818ace89f376d4cc07110a21fe0175286dc7d3097536749493f166"}}

+------+------------------+-------------+
|           fastlane summary            |
+------+------------------+-------------+
| Step | Action           | Time (in s) |
+------+------------------+-------------+
| 1    | default_platform | 0           |
| 2    | sauce_upload     | 2           |
| 3    | sauce_upload     | 2           |
+------+------------------+-------------+

[15:45:29]: fastlane.tools finished successfully 🎉
```

Now that we've uploaded our `ipa` files, you can log in to your sauce labs account and view the uploaded artefacts.
<p align="center">
  <img width="2000" height="800" src="/assets/images/app_upload.png">
</p>

## Generate saucectl config
[`saucectl`](https://docs.saucelabs.com/mobile-apps/automated-testing/espresso-xcuitest/xcuitest/) relies on a YAML specification file to determine exactly which tests to run, how to run them, and on which device and OS version. 
The generation and customisation of the required `saucectl` config to run your XCUITest tests can be automated using the `sauce_config` action.

### Execute tests by test runner
When building your application for real device testing an application runner `ipa` file will be generated. This `ipa` will contain all of your tests within the XCode Test Target.

To generate a `config.yml` file for iOS real devices based on test runner ipa file you simply need to add a new `lane` in your `Fastfile`, for example:

```ruby
desc "Create saucectl configuration for test runner"
lane :generate_sauce_config do
    sauce_config({platform: 'ios',
                  kind: 'xcuitest',
                  app: 'path/to/myTestApp.ipa',
                  test_app: 'path/to/TestRunner.ipa',
                  region: 'eu',
                  devices: [ {name: 'iPhone 11'}]
             })
end
```

### Help

Information and help for the `sauce_config` action can be printed out by running the following command:

```shell
  fastlane action sauce_config
```

For detailed explanation of parameters used here, please read the official docs [here](https://ianrhamilton.github.io/fastlane-plugin-saucectl/config/).

Now that your`generate_sauce_config` `lane` is complete execute the following command:
```shell
    bundle exec fastlane ios generate_sauce_config
```

```shell
ian@Ians-MBP-2 my-demo-app-ios % bundle exec fastlane ios generate_sauce_config
[✔] 🚀 
+--------------------------+---------+---------------------------------------------------------------+
|                                            Used plugins                                            |
+--------------------------+---------+---------------------------------------------------------------+
| Plugin                   | Version | Action                                                        |
+--------------------------+---------+---------------------------------------------------------------+
| fastlane-plugin-saucectl | 0.1.3   | sauce_apps, disabled_tests, install_saucectl, sauce_devices,  |
|                          |         | delete_from_storage, sauce_upload, sauce_runner, sauce_config |
+--------------------------+---------+---------------------------------------------------------------+

[12:28:18]: ------------------------------
[12:28:18]: --- Step: default_platform ---
[12:28:18]: ------------------------------
[12:28:18]: Driving the lane 'ios generate_sauce_config' 🚀
[12:28:18]: --------------------------
[12:28:18]: --- Step: sauce_config ---
[12:28:18]: --------------------------
[12:28:18]: Creating saucectl config .....🚕💨
[12:28:18]: Successfully created saucectl config ✅

+------+------------------+-------------+
|           fastlane summary            |
+------+------------------+-------------+
| Step | Action           | Time (in s) |
+------+------------------+-------------+
| 1    | default_platform | 0           |
| 2    | sauce_config     | 0           |
+------+------------------+-------------+

[12:28:18]: fastlane.tools finished successfully 🎉
ian@Ians-MBP-2 my-demo-app-ios % 
```

In the root of your application's directory you will now find a directory named `.sauce` which contains something like the following:
<details>
<summary>config.yml</summary>

{% highlight yaml %}
apiVersion: v1alpha
kind: xcuitest
retries: 0
sauce:
region: eu-central-1
concurrency: 1
xcuitest:
app: SauceLabs-Demo-App.ipa
testApp: SauceLabs-Demo-App-Runner.XCUITest.ipa
artifacts:
download:
when: always
match:
- junit.xml
directory: "./artifacts/"
reporters:
junit:
enabled: true
suites:
- name: xcuitest-iPhone_11_15_real
  testOptions:
  devices:
    - id: iPhone_11_15_real
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType: PHONE
      private: true

{% endhighlight %}

</details>
<br>

## Alternative Suite Options
There are several options available for generating test suites. Below I will explain these options.

### Xcode Test Plans

Apple added test plans to Xcode 11 to make it easier to execute tests with varying test configurations. Running your tests with different environments or localizations is a great way to catch more bugs.

At the time of writing, it is not possible to execute tests by Xcode’s Test Plan using the standalone `saucectl` runner, however using this plugin it is possible to generate suites based on your specified Test Plan.

When you create a Test Plan in Xcode a file is generated with the extension `.xctestplan`. The test plan is simply a JSON file containing the test configurations. The `fastlane-plugin-saucectl` plugin will read the `json` file of the specified test plan and collect `selectedTests`, or if your test plan contains `skippedTests` it will gather all tests within your UITest target and then simply remove them.

If your UI Test target contains many tests and you’re using Test Plans differentiate between test runs - this can be a huge benefit to your project.

We can do this by simply adding an additional parameter to the `generate_sauce_config` action we created above, named `test_plan`.
```ruby
desc "Create saucectl configuration for Xcode Test Plan"
lane :generate_sauce_config do
    sauce_config({platform: 'ios',
                  kind: 'xcuitest',
                  app: 'path/to/myTestApp.ipa',
                  test_app: 'path/to/TestRunner.ipa',
                  region: 'eu',
                  test_plan: 'UITests',
                  devices: [ {name: 'iPhone 11'}]
             })
end
```

Below is an example of a test plan containing skipped classes:

<details>
<summary>UITests.xctestplan</summary>

{% highlight json %}
{
"configurations" : [
{
"id" : "5949275D-70AC-45E9-AE6C-D7ED27FD2438",
"name" : "Configuration 1",
"options" : {

      }
    }
],
"defaultOptions" : {
"testTimeoutsEnabled" : true
},
"testTargets" : [
{
"skippedTests" : [
"NavigationTest",
"ProductDetailsTest"
],
"target" : {
"containerPath" : "container:My Demo App.xcodeproj",
"identifier" : "A2C826A72706266600616582",
"name" : "My Demo AppUITests"
}
}
],
"version" : 1
}
{% endhighlight %}

</details>
<br>

By executing the `generate_sauce_config` lane the following config will be generated:
<details>
<summary>config.yml</summary>

{% highlight ruby %}
apiVersion: v1alpha
kind: xcuitest
retries: 0
sauce:
region: eu-central-1
concurrency: 1
xcuitest:
app: SauceLabs-Demo-App.ipa
testApp: SauceLabs-Demo-App-Runner.XCUITest.ipa
artifacts:
download:
when: always
match:
- junit.xml
directory: "./artifacts/"
reporters:
junit:
enabled: true
suites:
- name: xcuitest-uitests
  testOptions:
  class:
    - My_Demo_AppUITests.ProductListingPageTest/testProductListingPageAddItemToCart
    - My_Demo_AppUITests.ProductListingPageTest/testProductListingPageAddMultipleItemsToCart
      devices:
    - id: iPhone_11_15_real
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType: PHONE
      private: true

{% endhighlight %}

</details>


### Sharding Test Plans
Espresso and Sauce Labs have their own implementation of test sharding for parallel execution, **this is not the same**. 

The `fastlane-plugin-saucectl` supports cross platform sharding, and this implementation will gather test classes and distribute evenly between specified devices. 

<p align="center">
  <img width="2000" height="800" src="/assets/images/saucectl_ios_test_shard_test_plan.png">
</p>

Again, we can simply modify the existing `action` that we created above with an additional parameter named `test_distribution`.
```ruby
desc "Create saucectl configuration for Test Plan"
lane :generate_sauce_config do
  sauce_config({platform: 'ios',
                kind: 'xcuitest',
                app: 'path/to/myTestApp.ipa',
                test_app: 'path/to/TestRunner.ipa',
                region: 'eu',
                test_plan: 'EnabledUITests',
                test_distribution: 'shard',
                devices: [ {name: 'iPhone RDC One'}, {id: 'iphone_rdc_two'} ],
               })
end 
```
<details>
<summary>config.yml</summary>

{% highlight yaml %}
apiVersion: v1alpha
kind: xcuitest
retries: 0
sauce:
region: eu-central-1
concurrency: 1
xcuitest:
app: SauceLabs-Demo-App.ipa
testApp: SauceLabs-Demo-App-Runner.XCUITest.ipa
artifacts:
download:
when: always
match:
- junit.xml
directory: "./artifacts/"
reporters:
junit:
enabled: true
suites:
- name: xcuitest-iphone rdc one-shard-1
  testOptions:
  class:
    - My_Demo_AppUITests.ProductListingPageTest/testProductListingPageAddItemToCart
      devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType: PHONE
      private: true
- name: xcuitest-iphone_rdc_two-shard-2
  testOptions:
  class:
    - My_Demo_AppUITests.ProductListingPageTest/testProductListingPageAddMultipleItemsToCart
      devices:
    - id: iphone_rdc_two
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType: PHONE
      private: true

{% endhighlight %}
</details>
<br>

As you can see from the above `config.yml` the tests that are enabled (or not skipped) within the Test Plan have been split between the two devices.

### Test Distribution options
One of the main drawbacks executing tests as a single suite is the long running videos on Sauce Labs. For example if you run a single suite on a single device and your test run takes 30 minutes (or longer), Sauce Labs will generate a video 30 minutes in length. This makes it very difficult to find the exact moment that a failure occurred. 

Therefore, I try to break by suites down by test class or by test case to shorten videos. This way if a test run contains a failure, the shortened video can be attached to defects. This obviously has it's drawbacks - for example the additional time added on for waiting for a device to become available from the previous test run. I would rather add slightly more time to the run in order to save time scrolling through lengthy videos for failures. This does work great on Android Virtual Devices (emulators) however at the time of writing Sauce Labs does not support this for iOS platform.

The [saucectl plugin](https://ianrhamilton.github.io/fastlane-plugin-saucectl/config/#xcuitest-test-distribution-options) has the capabilities to scan a UI Test target for test classes, and test cases. The plugin will then instruct `saucectl` to treat each specified option as a suite per specified real device(s).

#### Distribute tests by class
<p align="center">
  <img width="2000" height="800" src="/assets/images/saucectl_test_ios_testclass.drawio.png">
</p>

```ruby
desc "Create saucectl configuration distributing by class"
lane :generate_sauce_config do
  sauce_config(platform: 'ios',
               kind: 'xcuitest',
               app: "SauceLabs-Demo-App.ipa",
               test_app: "SauceLabs-Demo-App-Runner.XCUITest.ipa",
               region: 'eu',
               test_target: "My Demo AppUITests",
               test_distribution: 'class',
               devices: [ {name: 'iPhone RDC One'} ])
end 
```

The plugin will look through the specified UI Test Target and gather all test classes and create a suite for each `class`. The following config will be generated:
<details>
<summary>config.yml</summary>

{% highlight yaml %}
apiVersion: v1alpha
kind: xcuitest
retries: 0
sauce:
region: eu-central-1
concurrency: 1
xcuitest:
app: SauceLabs-Demo-App.ipa
testApp: SauceLabs-Demo-App-Runner.XCUITest.ipa
artifacts:
download:
when: always
match:
- junit.xml
directory: "./artifacts/"
reporters:
junit:
enabled: true
suites:
- name: xcuitest-my_demo_appuitests.navigationtest
  testOptions:
  class: My_Demo_AppUITests.NavigationTest
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType: PHONE
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productlistingpagetest
  testOptions:
  class: My_Demo_AppUITests.ProductListingPageTest
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest
  testOptions:
  class: My_Demo_AppUITests.NavigationTest
  devices:
    - id: iphone_rdc_two
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType: PHONE
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest
  devices:
    - id: iphone_rdc_two
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productlistingpagetest
  testOptions:
  class: My_Demo_AppUITests.ProductListingPageTest
  devices:
    - id: iphone_rdc_two
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true 
{% endhighlight %}

</details>

#### Distribute tests by test case
<p align="center">
  <img width="2000" height="800" src="/assets/images/saucectl_test_ios_testclass.drawio.png">
</p>

Although this distribution method is available it is not recommended for long running test suites. Hopefully in the future Sauce Labs will support virtual device testing for XCUITest, at that point this option will be useful as you can utilize your VM capacity. 

For now you can consider this as an experimental feature or for running a very small number of tests.

```ruby
desc "Create saucectl configuration distributing by testCase"
lane :generate_sauce_config do
  sauce_config(platform: 'ios',
               kind: 'xcuitest',
               app: "SauceLabs-Demo-App.ipa",
               test_app: "SauceLabs-Demo-App-Runner.XCUITest.ipa",
               region: 'eu',
               test_target: "My Demo AppUITests",
               test_distribution: 'testCase',
               devices: [ {name: 'iPhone RDC One'} ])
end 
```

The plugin will look through the specified test target and gather all test classes and their containing test cases and create a suite for each `testCase`:
<details>
<summary>config.yml</summary>

{% highlight yaml %}
apiVersion: v1alpha
kind: xcuitest
retries: 0
sauce:
region: eu-central-1
concurrency: 1
xcuitest:
app: SauceLabs-Demo-App.ipa
testApp: SauceLabs-Demo-App-Runner.XCUITest.ipa
artifacts:
download:
when: always
match:
- junit.xml
directory: "./artifacts/"
reporters:
junit:
enabled: true
suites:
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatetocart
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateToCart
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType: PHONE
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatetomore
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateToMore
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatemoretowebview
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateMoreToWebview
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatemoretoabout
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateMoreToAbout
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatemoretoqrcode
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateMoreToQRCode
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatemoretogeolocation
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateMoreToGeoLocation
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatemoretodrawing
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateMoreToDrawing
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatefromcarttocatalog
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateFromCartToCatalog
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.navigationtest/testnavigatecarttocatalog
  testOptions:
  class: My_Demo_AppUITests.NavigationTest/testNavigateCartToCatalog
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetails
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetails
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailsprice
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsPrice
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailshighlights
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsHighlights
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailsdecreasenumberofitems
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsDecreaseNumberOfItems
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailsincreasenumberofitems
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsIncreaseNumberOfItems
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailsdefaultcolor
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsDefaultColor
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailscolorsswitch
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsColorsSwitch
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailsratesselection
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsRatesSelection
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productdetailstest/testproductdetailsaddtocart
  testOptions:
  class: My_Demo_AppUITests.ProductDetailsTest/testProductDetailsAddToCart
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productlistingpagetest/testproductlistingpageadditemtocart
  testOptions:
  class: My_Demo_AppUITests.ProductListingPageTest/testProductListingPageAddItemToCart
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
- name: xcuitest-my_demo_appuitests.productlistingpagetest/testproductlistingpageaddmultipleitemstocart
  testOptions:
  class: My_Demo_AppUITests.ProductListingPageTest/testProductListingPageAddMultipleItemsToCart
  devices:
    - name: iPhone RDC One
      orientation: portrait
      options:
      carrierConnectivity: false
      deviceType:
      private: true
{% endhighlight %}

</details>
<br>

#### Test Only
If you would like to create a configuration that would only run specific tests or classes, you can do so by adding a [`test_class`](https://ianrhamilton.github.io/fastlane-plugin-saucectl/config/#test_class) parameter to the `sauce_config` action. This parameter takes an array of strings containing your classes.
```ruby
desc "Create saucectl configuration for specific test classes"
lane :generate_sauce_config do
  sauce_config(platform: 'ios',
               kind: 'xcuitest',
               app: "SauceLabs-Demo-App.ipa",
               test_app: "SauceLabs-Demo-App-Runner.XCUITest.ipa",
               region: 'eu',
               test_class: ['My_Demo_AppUITests.ProductDetailsTest', 'My_Demo_AppUITests.ProductListingPageTest/testProductListingPageAddMultipleItemsToCar'],
               devices: [ {name: 'iPhone RDC One'} ])
end 
```

# Install the saucectl binary
Now that you have generated your test configuration, you can install the `saucectl` test runner.

To generate the runner you can use the built in `install_saucectl` action. You can now create a `lane` for the installation.

```ruby
desc "Install Sauce Labs toolkit dependencies"
  lane :install do
    install_saucectl 
end
```

If you would like to test using a specific version of `saucectl` you can add the version to the `lane`:
```ruby
desc "Install Sauce Labs toolkit dependencies"
lane :install_toolkit do
  install_saucectl(version: '0.86.0')
end 
```

This will generate the `saucectl` runner file in the root of your project. This is required to execute tests on Sauce Labs. See the official [docs](https://ianrhamilton.github.io/fastlane-plugin-saucectl/install/) for more information.

# Execute tests

Now we have everything in place to execute tests using the `sauce_runner` action 🎉

Create a new `lane` for the test execution.

```ruby
desc "Execute UI tests on sauce labs platform"
lane :execute_tests do
  sauce_runner
end
```

By default the `saucectl` runner file will check if your `ipa` has previously been updated, else it will upload the files. 

# Putting it all together
Now we have created all of our `lanes` we can create a lane that put's it all together.

```ruby
desc "Execute ui tests on Sauce Labs real device cloud"
  lane :ui_tests do 
    generate_sauce_config 
    install
    execute_tests 
  end
```

To execute the `ui_tests` lane, simply run:
```shell
bundle exec fastlane ios ui_tests
```

This will cruise through each of your created `lanes` 😎

# Summary
`fastlane-plugin-saucectl` automates every aspect of your development and testing workflow for Sauce Labs execution. The flexible options for configuration are a huge benefit.

Check out the example [`Fastfile`](https://github.com/ianrhamilton/fastlane-plugin-saucectl/blob/main/fastlane/Fastfile#L80) in the fastlane-plugin-saucectl repository.

I hope you enjoyed part-one - if you did please show your support and give the repo a star, share, or even contribute! 🙏 

In part-two I will explain how you can use the `fastlane-plugin-saucectl` plugin to automate the set up and configuration for Android `espresso` UI tests on Sauce Labs.
