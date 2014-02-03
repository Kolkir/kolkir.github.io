---
title: C++ program compiled for x64 is slower than compiled for x86
layout: post
tags: 
---


I've wrote sample program to play with SSE optimizations. And received
some strange results:

1.  Compiled for x64 - my SSE optimizations didn't add any significant
    speed up, and some time are slower than regular code with compiler
    optimizations.
2.  Compiled for x86 - my SSE optimizations give some speed up (depends
    on processor). And total application execution time is much lesser
    than for x64, with my optimization or without.

As i understand x64 platform can give some performance improvements only
if you work with big amounts of memory, but heavy calculations
algorithms have sense to compile for x86.

I've used Visual Studio 2010 C++ compiler on Intel Core i5-2500 3.30Ghz.

But may be i make wrong conclusions?

Here is code i used for testing:

    #include <xmmintrin.h>

    #include "timer.h"

    #include <vector>
    #include <iostream>
    #include <algorithm>

    /********************DECLARATIONS************************************************/
    class Vector
    {
    public:
        Vector():x(0),y(0),z(0){}

        Vector(double x, double y, double z)
            : x(x)
            , y(y)
            , z(z)
        {
        }

        double x;
        double y;
        double z;
    };

    double Dot(const Vector& a, const Vector& b)
    {
        return a.x * b.x + a.y * b.y + a.z * b.z;
    }

    class Vector2
    {
    public:
        typedef double value_type;

        Vector2():x(0),y(0){}

        Vector2(double x, double y)
            : x(x)
            , y(y)
        {
        }

        double x;
        double y;
    };

    /******************************TESTS***************************************************/

    void Test(const std::vector<Vector>& m, std::vector<Vector2>& m2)
    {
        Vector axisX(0.3f, 0.001f, 0.25f);
        Vector axisY(0.043f, 0.021f, 0.45f);

        std::vector<Vector2>::iterator i2 = m2.begin();

        std::for_each(m.begin(), m.end(),
            [&](const Vector& v)
        {
            Vector2 r(0,0);
            r.x = Dot(axisX, v);
            r.y = Dot(axisY, v);

            (*i2) = r;
            ++i2;
        });
    }

    void TestSIMD(const std::vector<Vector>& m, std::vector<Vector2>& m2)
    {
        Vector axisX(0.3f, 0.001f, 0.25f);
        Vector axisY(0.043f, 0.021f, 0.45f);

        __m128d ax = _mm_set_pd(axisX.x, axisY.x);
        __m128d ay = _mm_set_pd(axisX.y, axisY.y);
        __m128d az = _mm_set_pd(axisX.z, axisY.z);

        std::vector<Vector2>::iterator i2 = m2.begin();

        std::for_each(m.begin(), m.end(),
            [&](const Vector& v)
        {
            __m128d x = _mm_set_pd(v.x, v.x);
            __m128d y = _mm_set_pd(v.y, v.y);
            __m128d z = _mm_set_pd(v.z, v.z);

            __m128d xx = _mm_mul_pd(x, ax);
            __m128d yy = _mm_mul_pd(y, ay);
            __m128d zz = _mm_mul_pd(z, az);

            __m128d xy = _mm_add_pd(xx, yy);
            __m128d xyz = _mm_add_pd(xy, zz);

            _mm_storeh_pd(&(*i2).x, xyz);
            _mm_storel_pd(&(*i2).y, xyz);
            ++i2;
        });
    }

    int main()
    {
        cpptask::Timer timer;

        int len2 = 300;
        size_t len = 5000000;
        std::vector<Vector> m;
        m.reserve(len);
        for (size_t i = 0; i < len; ++i)
        {
            m.push_back(Vector(i * 0.2345, i * 2.67, i * 0.98));
        }

        /***********************************************************************************/
        {
            std::vector<Vector2> m2(m.size());
            double time = 0;
            for (int i = 0; i < len2; ++i)
            {
                timer.Start();
                Test(m, m2);
                time += timer.End();
            }
            std::cout << "Dot product double - " << time / len2 << std::endl;
        }
        /***********************************************************************************/
        {
            std::vector<Vector2> m2(m.size());
            double time = 0;
            for (int i = 0; i < len2; ++i)
            {
                timer.Start();
                TestSIMD(m, m2);
                time += timer.End();
            }
            std::cout << "Dot product SIMD double - " <<  time / len2 << std::endl;
        }
        /***********************************************************************************/

        return 0;
    }
