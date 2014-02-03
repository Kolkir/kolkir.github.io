---
title: C++0x multi-threading
layout: post
tags: C++11
---


If you want to play with new functionality for multi-threading in C++
for free you need gcc-4.5 and Linux, MinGW currently doesn't support
this functionality (but it has all required headers). To compile
programm you need to specify complier flag -std=c++0x and linker flag
-pthread. VC++ currently have not this functionality too. Here is small
example:

    #include <thread>
    #include <mutex>
    #include <iostream>
    #include <unistd.h>

    class MyJob
    {
    public:
        MyJob(std::mutex* m) : m(m){}
        void operator()()
        {
            for (int i = 0; i < 5; ++i)
            {
                {
                    std::lock_guard<std::mutex> lock(*m);
                    std::thread::id id = std::this_thread::get_id();
                    std::cout << "Thread " << id << std::endl;
                }
                sleep(1);
            }
        }
    private:
        std::mutex* m;
    };

    int main()
    {
        std::mutex m;
        std::thread t1{MyJob(&m)};
        std::thread t2{MyJob(&m)};

        t1.join();
        t2.join();

        return 0;
    }
