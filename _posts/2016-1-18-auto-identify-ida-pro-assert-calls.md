---
layout: post
title: "Method name detection with assert in IDAPython"
description: ""
category:
tags: [reverse-engineering, mips, ida, idapython]
---

This post is a tutorial for creating an IDAPython script to identify unnamed functions. Some functions leak their original function names in `assert` calls or other error information. For this example, I'm using the home router TL-WDR3500's firmware, and a call to the `__assert` function as an example.

## Identifying method names

Several unnammed methods in this firmware use the `__assert` method. For example, the method `sub_4205C8` below calls `__assert` if a protocol value is invalid.

![IDA Pro sub_4205C8 disassembly](/assets/images/idapro-assert-script/sub_4205C8-assert.png)

The second parameter to `__assert` holds the source file's name (`$a1` = `"httpOutput.c"`), and the fourth parameter holds the calling method's name (`$a3` = `"httpStatusLine"`). In code, this looks something like this:

    int parser_parse_entity(request_id *reqId) {
      if (reqId->sProtocol < 2) {
         __assert("reqId->sProtocol < 2", "httpOutput.c", 0x48, "httpStatusLine");
      }
      // ...
    }


## Writing the script

The goal is to write a script that

- Identifies unnamed methods (`sub_*`) that call `__assert`
- Find the value of string containing the method name (stored in `$a3`)
- Print out all identified methods

### Identify unnamed methods

First, create a script file for IDAPython code. I'm calling mine `name-asserted-functions.py`.

<script src="https://gist.github.com/mopsled/25406c2cacef11d57b6d.js"></script>

To run the script, go to `File` -> `Script File...` in IDA Pro

<img src="/assets/images/idapro-assert-script/script-file.png" alt="Open file menu" style="width: 320px">l

Select the `name-asserted-functions.py` file:

<img src="/assets/images/idapro-assert-script/open-script.png" alt="Choose file dialog" style="width: 600px">   

The script may pause for a bit then dump the names of discovered `sub_*` functions:

<img src="/assets/images/idapro-assert-script/output-sub.png" alt="IDAPython evaluated output of unnamed functions" style="width:500px">

### Find functions that call `__assert`

Next, use IDAPython's [`NextHead`](https://www.hex-rays.com/products/ida/support/idapython_docs/idc-module.html#NextHead) to iterate through instructions in each function. On each function, look for code cross-references to `__assert` using IDAPython's [`CodeRefsFrom`](https://www.hex-rays.com/products/ida/support/idapython_docs/idautils-module.html#CodeRefsFrom) and [`GetFunctionName`](https://www.hex-rays.com/products/ida/support/idapython_docs/idc-module.html#GetFunctionName).

<script src="https://gist.github.com/mopsled/aeaca247cb74d58bd203.js"></script>

The output is a handful of functions that use `__assert`:

<img src="/assets/images/idapro-assert-script/output-sub-assert.png" alt="IDAPython evaluated output of functions with __assert references" style="width:500px">

### Find the `methodName` argument for `__assert`

__assert's function call looks like this:

    void __assert(char *message, char *file, int code, char *methodName);
    // e.g.
    __assert("reqId->sProtocol < 2", "httpOutput.c", 0x48, "httpStatusLine");

In order to find what's in the `methodName` argument, the script needs to determine what's stored in the fourth argument register, `$a3`. The code below steps backwards through the code using IDAPython's [`PrevHead`](https://www.hex-rays.com/products/ida/support/idapython_docs/idc-module.html#PrevHead), then uses [`GetOpnd`](https://www.hex-rays.com/products/ida/support/idapython_docs/idc-module.html#GetOpnd) to determine if a value is being loaded into `$a3`. Finally, the code uses IDAPython's [`GetOperandValue`](https://www.hex-rays.com/products/ida/support/idapython_docs/idc-module.html#GetOperandValue) and [`GetString`](https://www.hex-rays.com/products/ida/support/idapython_docs/idc-module.html#GetString) functions to find the value of the string and associate it with the unnamed function.

<script src="https://gist.github.com/mopsled/918513c87f02bff4498e.js"></script>

Running this script produces:

<img src="/assets/images/idapro-assert-script/output-names.png" alt="Final output of function names passed to __assert for each unnammed function" style="width: 600px">

Success!

### Bulk rewriting

To have the script automatically rename each function, IDAPython's [`MakeName`](https://www.hex-rays.com/products/ida/support/idapython_docs/idc-module.html#MakeName) function can be used with arguments for the function's address and the new name. I haven't included it in the code above.

    MakeName(function_address, function_new_name);
    // e.g.
    MakeName(0x4224D8, "httpReqLineParse");
