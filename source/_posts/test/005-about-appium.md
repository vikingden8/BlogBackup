---
title: About Appium
date: 2017-05-31 00:03:36
tags:
categories: "测试开发"
---

### Introduction to Appium

Appium is an open-source tool for automating **native**, **mobile web**, and **hybrid** applications on iOS and Android platforms. Native apps are those written using the iOS, Android, or Windows SDKs. Mobile web apps are web apps accessed using a mobile browser (Appium supports Safari on iOS and Chrome or the built-in ‘Browser’ app on Android). Hybrid apps have a wrapper around a “webview” – a native control that enables interaction with web content. Projects like **Phonegap**, make it easy to build apps using web technologies that are then bundled into a native wrapper, creating a hybrid app.

Importantly, Appium is “cross-platform”: it allows you to write tests against multiple platforms (iOS, Android, Windows), using the same API. This enables code reuse between iOS, Android, and Windows testsuites.

For specific information about what it means for Appium to “support” its platforms, and automation modalities, please see the [platform support](http://appium.io/slate/en/master/?ruby#appium-platform-support) doc.

<!--more-->

### APPIUM PHILOSOPHY

Appium was designed to meet mobile automation needs according to a philosophy outlined by the following four tenets:

* You shouldn’t have to recompile your app or modify it in any way in order to automate it.

* You shouldn’t be locked into a specific language or framework to write and run your tests.

* A mobile automation framework shouldn’t reinvent the wheel when it comes to automation APIs.

* A mobile automation framework should be open source, in spirit and practice as well as in name!

### APPIUM DESIGN

So how does the structure of the Appium project live out this philosophy? We meet requirement #1 by using vendor-provided automation frameworks under the hood. That way, we don’t need to compile in any Appium-specific or third-party code or frameworks to your app. This means you’re testing the same app you’re shipping. The vendor-provided frameworks we use are:

* iOS: 9.3+ Apple’s XCTest

* Android 4.2+: Google’s UiAutomator

* Android 2.3+: Google’s Instrumentation. (Instrumentation support is provided by bundling a separate project, Selendroid)

* Windows: Microsoft’s WinAppDriver

We meet requirement #2 by wrapping the vendor-provided frameworks in one API, the WebDriver API. WebDriver (aka “Selenium WebDriver”) specifies a client-server protocol (known as the JSON Wire Protocol). Given this client-server architecture, a client written in any language can be used to send the appropriate HTTP requests to the server. There are already clients written in every popular programming language. This also means that you’re free to use whatever test runner and test framework you want; the client libraries are simply HTTP clients and can be mixed into your code any way you please. In other words, Appium & WebDriver clients are not technically “test frameworks” – they are “automation libraries”. You can manage your test environment any way you like!

We meet requirement #3 in the same way: WebDriver has become the de facto standard for automating web browsers, and is a W3C Working Draft. Why do something totally different for mobile? Instead we have extended the protocol with extra API methods useful for mobile automation.

It should be obvious that requirement #4 is a given – you’re reading this because [Appium is open source](https://github.com/appium/appium).

### APPIUM CONCEPTS

#### Client/Server Architecture

Appium is at its heart a webserver that exposes a REST API. It receives connections from a client, listens for commands, executes those commands on a mobile device, and responds with an HTTP response representing the result of the command execution. The fact that we have a client/server architecture opens up a lot of possibilities: we can write our test code in any language that has a http client API, but it is easier to use one of the Appium client libraries. We can put the server on a different machine than our tests are running on. We can write test code and rely on a cloud service like Sauce Labs to receive and interpret the commands.

#### Session

Automation is always performed in the context of a session. Clients initiate a session with a server in ways specific to each library, but they all end up sending a POST /session request to the server, with a JSON object called the 'desired capabilities’ object. At this point the server will start up the automation session and respond with a session ID which is used for sending further commands.

#### Desired Capabilities

Desired capabilities are a set of keys and values (i.e., a map or hash) sent to the Appium server to tell the server what kind of automation session we’re interested in starting up. There are also various capabilities which can modify the behavior of the server during automation. For example, we might set the platformName capability to iOS to tell Appium that we want an iOS session, rather than an Android or Windows one. Or we might set the safariAllowPopups capability to true in order to ensure that, during a Safari automation session, we’re allowed to use JavaScript to open up new windows. See the capabilities doc for the complete list of capabilities available for Appium.

#### Appium Server

Appium is a server written in Node.js. It can be built and installed from source or installed directly from NPM:
```sh
 $ npm install -g appium
 ```

#### Appium Clients

There are client libraries (in Java, Ruby, Python, PHP, JavaScript, and C#) which support Appium’s extensions to the WebDriver protocol. When using Appium, you want to use these client libraries instead of your regular WebDriver client. You can view the full list of libraries [here](http://appium.io/slate/en/master/?ruby#list-of-client-libraries-with-appium-server-support).

#### [Appium Desktop](https://github.com/appium/appium-desktop/releases/tag/v1.0.2-beta.2)

There exist GUI wrappers around the Appium server that can be downloaded. These come bundled with everything required to run the Appium server, so you don’t need to worry about Node. They also come with an [Inspector](), which enables you to check out the hierarchy of your app. This can come in handy when writing tests.
