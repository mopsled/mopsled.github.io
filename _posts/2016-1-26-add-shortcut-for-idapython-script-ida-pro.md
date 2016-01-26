---
layout: post
title: "Add Shortcut for IDAPython script in IDA"
description: ""
category:
tags: [ida, idapython, reverse-engineering]
---

This post describes a way to associate an IDAPython script with a hotkey in IDA. The hotkeys are defined in Python and only require a small bit of bootstrap IDC code to initialize.

## Modify startup script

First, modify IDA's startup to run the custom hotkey code. IDA's startup script is located in the `ida.idc` file which can be found at `<IDADIR>/idc/ida.idc`.

At the bottom of the file, edit the `main()` function with the line below. Change the directory (e.g. `C:\\Users\\Alec`) to point at a path where you will store your shortcut script.

`ida.idc`

    static main(void)
    {
        // ...
        RunPythonStatement("execfile(r'C:\\Users\\Alec\\ida-shortcuts.py')");
    }

## Create hotkey script

Create your Python hotkey script at the path specified in `ida.idc`. I named this file `ida-shortcuts.py`. Here's a simple exmaple that binds `<CTRL>`+`<ALT>`+`<SHIFT>`+`H` to the `say_hi` function.

`ida-shortcuts.py`

    import idaapi

    def say_hi():
        print "Hi!"

    idaapi.CompileLine('static py_say_hi() { RunPythonStatement("say_hi()"); }')

    AddHotkey("Ctrl+Alt+Shift+H", 'py_say_hi')

To run an external script instead of in-file code, use the `execfile` function:

`ida-shortcuts.py` (with `execfile`)

<pre><code>import idaapi

def say_hi():
    <strong>execfile(r"C:\Users\Alec\say_hi.py")</strong>

idaapi.CompileLine('static py_say_hi() { RunPythonStatement("say_hi()"); }')

AddHotkey("Ctrl+Shift+H", 'py_say_hi')</code></pre>

## Multiple hotkeys

If several IDAPython shortcuts are defined, repeating the steps above for each function can be tedius. The script below uses `ida_run_python_function` and `idaapi.CompileLine` to compile new functions. To use the script below, define a python function and use the `AddHotkey` function with your function name like this:

<pre><code><strong>AddHotkey("Ctrl+Alt+Shift+H", ida_run_python_function("say_hi"));</strong></code></pre>

`ida-shortcuts.py` (with `ida_run_python_function`)

<pre><code>import idaapi
compiled_functions = {}
def ida_run_python_function(func_name):
    if func_name not in compiled_functions:
        ida_func_name = "py_%s" % func_name
        idaapi.CompileLine('static %s() { RunPythonStatement("%s()"); }' 
            % (ida_func_name, func_name))
        compiled_functions[func_name] = ida_func_name
    return ida_func_name

def say_hi():
    execfile(r"C:\Users\Alec\say_hi.py")

def search_asserts():
    execfile(r"C:\Users\Alec\search_asserts.py")

<strong>AddHotkey("Ctrl+Alt+Shift+H", ida_run_python_function("say_hi"));</strong>
<strong>AddHotkey("Ctrl+Shift+A", ida_run_python_function("search_asserts"));</strong>
</code></pre>