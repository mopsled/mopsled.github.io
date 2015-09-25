---
layout: post
title: "Log iOS method arguments with Frida"
description: ""
category:
tags: [reverse-engineering, frida]
---

## What is Frida?

[Frida](http://www.frida.re/) is a reverse engineering tool that can be used for dynamic code instrumentation. Frida works by injecting a JavaScript engine and console into a live process. With Frida, it's possible to dynamically expose and change behavior in memory, functions, and the [recently added Objective-C runtime](http://www.frida.re/docs/javascript-api/#objc) using JavaScript. Frida is available for debugging jailbroken iOS devices, along with native Linux, Windows, Mac, and Android processes.

This is a short guide to hook an Objective-C method with Frida, logging  method arguments to the console.

## Frida setup

The first step is to follow [Frida's iOS tutorial](http://www.frida.re/docs/ios/) to ensure that Frida can communicate with your iOS device. The major steps are to:

1. Install Frida on your computer
2. Install Frida on your jailbroken iOS device through Cydia
3. Test that Frida can communicate with your iOS device over USB

<pre>
$ <strong>frida-ps -U</strong>
PID  Name
---  ----------------
597  Audible
480  Camera
632  Downcast
563  LINE
133  Mail
601  Messenger
666  Settings
...</pre>

## Identify Objective-C method

For this example I'm injecting Frida into an app called [LINE](http://line.me/en/), an instant-messaging app from Japan. I used the [Hopper](http://www.hopperapp.com/) dissasembler to find a method in LINE I was interested in examining, `[MessageViewController sendMessageWithText:]`

![Hopper disassembly](/assets/images/frida-log-ios-method/line-disassembly.png)

Use Frida to identify the underlying function in-memory:

<pre>
$ <strong>frida -U LINE</strong>
    _____
   (_____)
    |   |    Frida 5.0.0 - A world-class dynamic instrumentation framework
    |   |
    |`-'|    Commands:
    |   |        help      -> Displays the help system
    |   |        object?   -> Display information about 'object'
    |   |        exit/quit -> Exit
    |   |
    |   |    More info at http://www.frida.re/docs/home/
    `._.'

[USB::iPod Touch 5G::LINE]-> <strong>ObjC.classes.MessageViewController</strong>
{
    "handle": "0x1913c48"
}
[USB::iPod Touch 5G::LINE]-> <strong>ObjC.classes.MessageViewController["- sendMessageWithText:"]</strong>
function</pre>

## Write Interceptor script to attach to method

Create a file called `interceptSendMessage.js` to hold the method-injecting data:

    var sendMessage = ObjC.classes.MessageViewController["- sendMessageWithText:"];

    Interceptor.attach(sendMessage.implementation, {
      onEnter: function(args) {
        // args[0] is self
        // args[1] is selector (SEL "sendMessageWithText:")
        // args[2] holds the first function argument, an NSString
        var message = ObjC.Object(args[2]);
        console.log("\n[MessageViewController sendMessageWithText:@\""
            + message.toString() + "\"]");
      }
    });

## Run the script

To tell Frida to connect to the LINE app on an iOS device over USB and run the script in `interceptSendMessage.js`, run this command:

<pre>
$ <strong>frida -U -l interceptSendMessage.js LINE</strong></pre>

After Frida has attached to the process, try sending a message:

![Hopper disassembly](/assets/images/frida-log-ios-method/frida-message-demonstration.png)

### Additional Information

## Modifying arguments in Interceptor

It's easy to dynamically modify arguments with Frida too. Here's an example script that will append the string `@" :)"` to the first method argument to `MessageViewController`'s `sendMessageWithText`:

    var sendMessage = ObjC.classes.MessageViewController["- sendMessageWithText:"];

    Interceptor.attach(sendMessage.implementation, {
      onEnter: function(args) {
        var message = ObjC.Object(args[2]);
        var modifiedMessage = message["- stringByAppendingString:"](" :)");
        args[2] = modifiedMessage;
      }
    });
