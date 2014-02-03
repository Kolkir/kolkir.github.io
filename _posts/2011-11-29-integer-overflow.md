---
title: Integer overflow
layout: post
tags: Overflow
---


It have sense to never user clear unsigned types in loops in your
program to avoid overflow situation. But if you need big numbers range
from unsigned types, you can use wrappers like
[SafeInt](http://safeint.codeplex.com/). Good article how to use it you
can find [here](http://msdn.microsoft.com/en-us/library/ms972705).

And little example for VisualStudio:
```
    #include <limits>
    #include <iostream>
    #include <safeint.h>

    typedef unsigned long DWORD;

    using namespace std;

    void CheckOverflow(DWORD val)
    {
        if (val == numeric_limits<DWORD>::max())
        {
            cout << "overflow\n";
        }
        else
        {
            cout << "ok\n";
        }
    }

    int main()
    {
        DWORD val = -1;

        cout << "-1\n";
        CheckOverflow(val);

        val = -2;
        cout << "-2\n";
        CheckOverflow(val);

        int v = -2;
        msl::utilities::SafeInt<DWORD> a(0); 

        cout << "DWORD size: " << sizeof(DWORD) << "\n";
        cout << "SafeInt<DWORD> size: " << sizeof(msl::utilities::SafeInt<DWORD>) << "\n";

        a = v;//raise exception

        return 0;
    }

```
