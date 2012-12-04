---
layout: post
title: "Understand ArraySizeHelper"
date: 2012-10-07 16:14
comments: true
categories: 
---

## Intro
In Chrome's source code, `arraysize` is a macro used to calculate the number of elements in one array. It can be applied to different kinds of arrays without indicate the arrays' type explicitly. The code is shown as below.

{% codeblock ArraySizeHelper lang:cpp %}
template <typename T, size_t N>
char (&ArraySizeHelper(T (&array)[N]))[N];
#define arraysize(array) (sizeof(ArraySizeHelper(array)))
{% endcodeblock %}

The code is short, but it seems not very easy for C++ beginner to learn how it works. In order to understand this tircky macro, first we need to understand 3 special language features respectively.

## Reference
Normally Reference is used in C++ to substitute Pointer in C in function call, so the function can have a clearer declaration and better data abstraction. But sometimes, the syntax of Reference is little complicated.

{% codeblock lang:cpp %}
// cliche
char c;
char* p;
char& r = c;

// dizzy code
char s[10];
char* pp[10];
char (*ps)[10];
// ?
// char& rr[10]; //error
char (&rs)[10] = s;

// how about this
char fc() { return 'a'; }
char* fp() { return 0; }
// char fs()[10]; //error
char& fr() { char c='a'; return c; }
char (&fr_decl());
char (&fr_impl()) { char c='a'; return c; }

// tricky
char (*pfc)() = fc;
char* (*pfp)() = fp;
// ??
char (&rfc)() = fc;
char& (&rfr)() = fr;
// ???
// char (&rfs)()[10]; //error
char (&frs())[10];
char (&frs_impl())[10] { char s[10]; return s; }
char (&(&rfrs)())[10] = frs;
{% endcodeblock %}

Through above examples, we can understand that `ArraySizeHelper` part involved in the macro code is actually a function declaration. For example, `ArraySizeHelper` in the code `int arr[10]; ArraySizeHelper(arr);` is equal with `char (&ArraySizeHelper(int (&array)[10]))[10]`, which is a function accepts a reference of an int array(size 10) as an parameter, and returns a reference of a char array(size 10) as result.

## Template
As we already know `ArraySizeHelper` is a function declaration, the template part in the macro is not hard to understand. According to the macro, the template for `ArraySizeHelper` can only accept `array type` as its parameter. It will generate specific declaration corresponding the parameter's type automatically.

{% codeblock lang:cpp %}
// simulate
float f[5];
float f2[5]
foo o[8];

ArraySizeHelper(f);
ArraySizeHelper(f2);
ArraySizeHelper(o);

// Template Instantiation...
char (&ArraySizeHelper(float (&array)[5]))[5];
char (&ArraySizeHelper(foo (&array)[8]))[8];
{% endcodeblock %}

One point worth mention is that directly call `ArraySizeHelper` will cause error in the program, because `ArraySizeHelper` has no implement code.

## Sizeof
`sizeof` is a very interesting keyword. We usually use it to compute the size of special user-defined structure, or use it in the bit operation. It is natural to think it acts like a function, but actually it will return the computed result during compiling time instead of runing time, so we must consider it separately as a special language feature.

The way `sizeof` used in the macro is very tricky, and I have not seen this usage in any other articles or codes before.

{% codeblock lang:cpp %}
// sizeof reference
char c;
int i;
char s[7];
int a[9];

char& rc = c;
int& ri = i;
char (&rs)[7] = s;
int (&ra)[9] = a;

int src = sizeof(rc);
int sri = sizeof(ri);

int ss = sizeof(s);
int srs = sizeof(rs);
int sa = sizeof(a);
int sra = sizeof(ra);

// sizeof function call
int fi();
char fc(int arg);
char (&frs(double arg))[3] {
    char s[3] = {'h', 'i', '\0'};
    std::cout<<s;
    return s;
}

// what will happen?
// int sfi = sizeof(fi); // error
int sfi = sizeof(fi());
int sfc = sizeof(fc(7));
int sfrs = sizeof(fs(2.2));

{% endcodeblock %}

From above examples, we can find out that `sizeof` will treat reference types the same as their referring types, and it will treat function calls the same as their return types. As mentioned above, `sizeof` is executed during compiling time, so it can accept function calls as its parameter without caring about the implement of those functions because it will not really call them.

## Last Shot
Now we can give an example to show how the ArraySizeHelper macro works.

{% codeblock lang:cpp %}
float f[5];

arraysize(f);

// equal with...
char (&ArraySizeHelper(float (&array)[5]))[5];
sizeof(ArraySizeHelper(f));

// equal with...
sizeof(char (&array)[5]);

// return 5
{% endcodeblock %}
