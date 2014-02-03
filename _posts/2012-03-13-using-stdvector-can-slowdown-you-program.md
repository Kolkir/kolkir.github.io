---
title: Using std::vector can slowdown you program
layout: post
tags: 
---


I've reviewed some code for matrices manipulation, and saw that it was
written with raw C array, I've changed it with std::vector and got big
slowdown in matrices coping. After that i wrote test program and saw
that the std::vector::assign method is very ineffective for the full
container coping, and std::vector::operator= is too ineffective than
simple memcpy(). Maybe i do something wrong? I alway thought that i can
use std::vector instead C array in C++. But now i see that i have pay
for that.

    #define _SECURE_SCL 0

    #include "timer.h"
    #include <iostream>
    #include <vector>

    int main()
    {
        cpptask::Timer timer;

        int arrLen = 256;
        int len = 1000000;
        {
            double* m1 = new double[arrLen];
            double* m2 = new double[arrLen];

            timer.Start();

            for(int i = 0; i < len; ++i)
            {
                if (i % 2)
                {
                    memcpy(m1, m2, arrLen * sizeof(double)); 
                }
                else
                {
                    memcpy(m2, m1, arrLen * sizeof(double));
                }
            }

            std::cout << "Array " << timer.End() << "\n";

            delete[] m1;
            delete[] m2;
        }
        {
            std::vector<double> m1(arrLen);
            std::vector<double> m2(arrLen);

            timer.Start();

            for(int i = 0; i < len; ++i)
            {
                if (i % 2)
                {
                    //m1.assign(m2.begin(), m2.end());
                    m1 = m2;
                }
                else
                {
                    m2 = m1;
                    //m2.assign(m1.begin(), m1.end());
                }
            }

            std::cout << "Vector " << timer.End() << "\n";
        }
    }
