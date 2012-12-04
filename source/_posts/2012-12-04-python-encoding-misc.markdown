---
layout: post
title: "Python Encoding Misc"
date: 2012-12-04 14:55
comments: true
categories: 
---

## Intro
There are several common problems about the encoding in Python. Here are my notes for them. All experiments are carried out in `Windows`.

## Interpreter
Python's interpreter cannot automatically detect source files' encoding, so any Non-ASCII characters will break the execution even they are just in comment.
{% codeblock cp936.py lang:python %}
# This will fail
# 你好
{% endcodeblock %}
We have to explicitly declare the encoding. The following is an example for `cp936`.
{% codeblock cp936.py lang:python %}
# coding=cp936
# This is ok
# 你好
{% endcodeblock %}
One exception is that the interpreter can handle the `utf-8` file with the `BOM` header correctly, even without the specific declaration.
The interpreter cannot handle the `unicode` files such as those created by `Notepad` in Windows, even with the declaration.
If the file's encoding is not consistent with the declaration, there may be other encoding/decoding problems.

## Print
We usually use Python's `Print` function to output strings to `sys.stdout`, but this function cannot handle the string's encoding properly either. If the paramater is an `unicode PyString`, `Print` can encode it into the system's local encoding and output correctly. Here are two examples.
{% codeblock cp936.py lang:python %}
# coding=cp936
# unicode string, this is ok
s=u"你好"
print s
{% endcodeblock %}

{% codeblock utf8.py lang:python %}
# coding=utf-8
# unicode string, this is ok
s=u"你好"
print s
{% endcodeblock %}
If the parameter is not a `unicode PyString`, `Print` just treat it as `ASCII` and send to `sys.stdout`. This will be fine if the parameter use the same as the system's local encoding, but if it will cause error in other circumstances.
{% codeblock cp936.py lang:python %}
# coding=cp936
# same as local encoding, this is ok on my machine
s="你好"
print s
{% endcodeblock %}

{% codeblock utf8.py lang:python %}
# coding=utf-8
# This will be wrong
s="你好"
print s
# This will be wrong too
s="\xE4\xBD\xA0\xE5\xA5\xBD" # the same sequence
print s
{% endcodeblock %}
We must decode the parameter into `unicode` to solve this problem.
{% codeblock utf8.py lang:python %}
# coding=utf-8
# This will be ok
s="你好"
print s.decode("utf-8")
# This will be ok too
s="\xE4\xBD\xA0\xE5\xA5\xBD" # the same sequence
print unicode(s, "utf-8")
{% endcodeblock %}

## Embedded Python
In some tasks we need to do some interaciton between `C` and `Python`, there are also some tricky encoding problems. The following is a `C` program calling the `Print` function.
{% codeblock print.c lang:python %}
#include <Python.h>

int main(int argc, char* argv[])
{   
    const char* str = "# coding=utf-8\n" "s='你好'\n" "print unicode(s, 'utf-8')";
    Py_Initialize();
    PyRun_SimpleString(str);
    Py_Finalize();
    return 0;
}
{% endcodeblock %}
If we save this source file as `utf-8` and compile it with `Visual C++`, the result of the program will be wrong. But the following program will be right.
{% codeblock print.c lang:python %}
#include <Python.h>

int main(int argc, char* argv[])
{   
    // even the file is saved as utf-8, the string here will be compiled into local encoding
    const char* str = "# coding=cp936\n" "s='你好'\n" "print unicode(s, 'cp936')";
    Py_Initialize();
    PyRun_SimpleString(str);
    Py_Finalize();
    return 0;
}
{% endcodeblock %}
The problem is that the compiler will convert the string into local encoding without caring about the encoding of the file. What's worse, I cannot find any way to change this setting. So to solve this problem, we should convert strings to sepcial encoding manually but not let the compiler do it.

## Conclusion
In conclusion, to avoid other similar problems, we should only use `unicode` strings in Python, and use resource files instead of writing strings directly in the source files.
