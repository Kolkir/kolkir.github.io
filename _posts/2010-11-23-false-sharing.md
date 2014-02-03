---
title: False sharing
layout: post
tags: Multithreading
---


Here is "False sharing" example, you can try it in VS with enabled
OpenMP flag. Also if you enable speed optimization for compiler single
thread version will be faster, but you can see false sharing effect too.

    #include <iostream>
    #include <ctime>
    #include <omp.h>

    using std::cout;
    using std::endl;

    const double M=8e8;

    //single thread work to do
    void one_thread() 
    {
        long int n1=1, n2=1;
        for(; n1<M; n1+=n1%3);
        for(; n2<M; n2+=n2%3);
        cout << "Sum = " << n1 + n2 << endl;
    }

    //multi thread work to do
    void multi_thread() 
    {
        struct T
        {
            long int n1;
            char chuck[64];//should be commented to see false sharing effect
            long int n2;
        } t;
        t.n1 = 1;
        t.n2 = 1;
        #pragma omp parallel num_threads(2) shared(M,t)
        {
            #pragma omp sections
            {
                #pragma omp section
                for(; t.n1<M; t.n1+=t.n1%3);

                #pragma omp section
                for(; t.n2<M; t.n2+=t.n2%3);
            }
        }
        cout << "Sum = " << t.n1 + t.n2 << endl;
    }

    time_t start, end;
    double diff;

    int main(int argc, char* argv[]) 
    {
        time(&start);
        multi_thread();
        time(&end);
        diff=difftime(end, start);
        cout<<diff<<" seconds elapsed for multi thread calculation."<<endl;

        time(&start);
        one_thread();
        time(&end);
        diff=difftime(end, start);
        cout<<diff<<" seconds elapsed for 1 thread calculation."<<endl;
        return 0;
    }
