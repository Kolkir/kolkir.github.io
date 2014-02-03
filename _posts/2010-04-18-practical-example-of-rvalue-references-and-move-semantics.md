---
title: Practical example of Rvalue references and move semantics
layout: post
tags: C++11
---


Here is simple example how to use move semantics to optimize you code
-you can override + operator to reduce constructor numbers:

    class A
    {
    public:
        A(int x)
          : x(x)
        {
            std::cout << "A(int x)\n";
        }

        A(const A& a)
           : x(a.x)
        {
            std::cout << "A(const A&)\n";
        }

        A& operator=(const A& a)
        {
            x = a.x;
            return *this;
        }

        A operator+(const A& a)
        {
            return A(x + a.x);
        }

    private:
        int x;
    };

    class B
    {
    public:
        B(int x)
           : x(x)
        {
            std::cout <<  "B(int x)\n";
        }

        B(const B&amp; b)
           : x(b.x)
        {
            std::cout <<  "B(const B&)\n";
        }

        B(B&& b) 
           : x(std::move(b.x))
        {
            std::cout <<  "B(B&&)\n";
        }

        B&; operator=(const B& b)
        {
            x = b.x;
            return *this;
        }

        B& operator=(B&& b)
        {
            x = std::move(b.x);
            return *this;
        }

        B operator+(const B& b)
        {
            return B(x + b.x);
        }

        B&& operator+(B&& b)
        {
            b.x += x;
            return std::move(b);
        }

    private:
        int x;
    };

    int main(int argc, char* argv[])
    {
        A a1(5);
        A a2 = a1 + 2 + 3;

        B b1(5);
        B b2 = b1 + 2 + 3;
        return 0;
    }
