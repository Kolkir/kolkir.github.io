---
title: Again about rvalues
layout: post
tags: 
---


You can use rvalues for creating move semantics - remove copy
constructors for temporaries. Example:

    #include <iostream>

    class A
    {
    public:
        A()
        {
            x = new int[1];
            x[0] = 1;
            std::cout << "newn";
        }
        ~A()
        {
            delete[] x;
        }

        A(const A& a)
        {
            x = new int[1];
            x[0] = a.x[0];
            std::cout << "newn";
        }

       /* A(A&& a) //move constructor
        {
            x = a.x;
            a.x = 0;
        }*/
    private:
        int *x;
    };

    A GetA()
    {
        A a;
        return a;
    }
