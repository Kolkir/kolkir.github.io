---
title: Perfect forwarding full example
layout: post
tags: C++11
---


Here is an example of how to use rvalue references for forwarding:

    #include <functional>
    #include <iostream>
    #include <memory>

    class V
    {
    public: 
        explicit V(int x) :x(x) 
        {
           std::cout < <  "V()n"; 
        }
        int x;
    protected:
       V(const V&){}
       V& operator=(const V&){}
    };

    class A
    {
    public:
        A(V& v) 
        {
           std::cout < <  "A(V&) x = " < <  v.x < <  "n"; 
        }
    };

    template< typename T, typename Arg> std::shared_ptr< T> factory(Arg arg)
    {
        return std::shared_ptr< T>(new T(arg));
    } 

    template< typename T, typename Arg> std::shared_ptr< T> factory2(Arg&& arg)
    {
        // you can't use std::move for forwarding - compiler error 
        return std::shared_ptr< T>(new T(std::forward< V>(arg)));
    } 

    int main(int argc, char* argv[])
    {
        V v(1);

        A a(v);

       // compiler error //std::shared_ptr a1 = factory(v);

        std::shared_ptr< A> a1 = factory2< A>(v);

        return 0;
    }
