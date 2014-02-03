---
title: Performance and memory alignment to cache line size boundaries
layout: post
tags: Multithreading
---


Here i would like to show an example of the application where you can
see that data aligned to the boundaries of cache line size is accessed
much more faster then unaligned:

    #include "timer.h"

    #include <algorithm>
    #include <iostream>

    class A
    {
    public:
        int x[18];
        A* next;
    };

    class B
    {
    public:
        int x[15];
        B* next;
    };

    template<class T>
    void TestFunc(size_t len, size_t cycles)
    {
        size_t l = sizeof(T);        
        std::cout << "Size = " << l << std::endl;

        T* m = new T[len];
        for(size_t i = 0; i < len; ++i)
        {
            m[i].next = 0;
            if (i > 0)
            {
                m[i - 1].next = &m[i];
            }
        }
        parallel::Timer timer;
        timer.Start();
        for (size_t i = 0; i < cycles; ++i)
        {
            T* item = &m[0];
            while (item != 0)
            {
                item->x[4] += 5;
                item = item->next;
            }
        }
        std::cout << cycles << " cycles in " << timer.End() << " msn";
        delete[] m;
    }

    int main(int /*argc*/, char* /*argv*/[])
    {      
        size_t len = 1024;
        size_t cycles = 5000;

        TestFunc<A>(len, cycles);
        TestFunc<B>(len, cycles);
        return 0;
    }

Here i assume that the cache line size is 64 bytes. Also I have founded
for myself dependency between structure size and access speed - Smaller
structures are processed quickly. You can look at this example under ADM
CodeAnalyst to see that there are no data cache misses for second
function.
