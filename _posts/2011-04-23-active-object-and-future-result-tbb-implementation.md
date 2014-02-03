---
title: Active object and future result TBB implementation
layout: post
tags: Multithreading
---


After the reading articles about [future
results](http://drdobbs.com/cpp/226700179) and [active
objects](http://drdobbs.com/go-parallel/article/showArticle.jhtml?articleID=225700095)
I have implemented a class which realize these concepts, asthreading
library i used TBB. Here is a code:

Active.h:

    #ifndef _ACTIVE_OBJECT_H_
    #define _ACTIVE_OBJECT_H_

    #include "Types.h"

    #include <functional>
    #include <memory>

    #include <tbb/tbb.h>

    namespace Threading
    {

    template <class T>
    class FutureResult
    {
    public:
        FutureResult()
        {
            isAvailable = false;
            haveError = false;
        }

        void SetValue(const T& val)
        {
            tbb::mutex::scoped_lock lock(guard);
            value = val;
            isAvailable = true;
            haveError = false;
        }

        T GetValue() const
        {
            tbb::mutex::scoped_lock lock(guard);
            isAvailable = false;
            haveError = false;
            return value;
        }

        bool IsAvailable() const
        {
            return isAvailable;
        }

        bool HaveError() const
        {
            return haveError;
        }

        std::string GetErrorReport() const
        {
            tbb::mutex::scoped_lock lock(guard);
            return errorReport;
        }

        void SetErrorReport(const std::string& error)
        {
            tbb::mutex::scoped_lock lock(guard);
            errorReport = error;
        }

    private:
        T value;
        std::string errorReport;
        mutable tbb::atomic<bool> isAvailable;
        mutable tbb::atomic<bool> haveError;
        mutable tbb::mutex guard;
    };

    template <class R, class F, class O>
    class FunctionRunner : public tbb::task
    {
    public:
        FunctionRunner(R r, F f, O* p)
            : result(r)
            , func(f)
            , ptr(p)
        {}
        virtual task* execute()  
        {
            try
            {
                result->SetValue((ptr->*func)());
            }
            catch(std::exception& e)
            {
                result->SetErrorReport(e.what());
            }
            return 0;
        }
        R result;
        F func;
        O* ptr;
    };

    class Active

    {

    public:

        Active();

        ~Active();

        template<class R, class F, class O>

        std::shared_ptr<FutureResult<R> > ExecuteParallel(F f, O* p)

        {

            std::shared_ptr<FutureResult<R> > result(new FutureResult<R>());

            tbb::task& tbbTask = *(new(parent->allocate_child())

                                   FunctionRunner<decltype(result),F,O>(result, f, p));  

            parent->increment_ref_count();  

            parent->spawn(tbbTask);  

            return result;

        }

        void WaitAll() const;

    private:

        Active(const Active&);

        void operator= (const Active&);

    private:

        tbb::empty_task* parent;

    };

    }

    #endif

Active.cpp:

    #include "Active.h"

    namespace Threading
    {
    /*******************************************************************************/
    Active::Active()

    {

        parent = new( tbb::task::allocate_root() ) tbb::empty_task;  

        parent->set_ref_count(1);

    }

    /*******************************************************************************/

    Active::~Active() 

    {

        parent->wait_for_all();
        parent->destroy(*parent);

    }
    /*******************************************************************************/
    void Active::WaitAll() const
    {
        parent->wait_for_all();
        parent->set_ref_count(1);
    }
    /*******************************************************************************/
    }
