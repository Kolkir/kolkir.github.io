---
title: Simple unit test framework for C++
layout: post
tags: UnitTesting
---


I want to create very simple unit test framework for C++ which can be
used with Visual Studio. I mean that it must have some integration to
IDE. When i searched for some existent frameworks i didn't find
anything useful and open source. Also i want this framework should be
very lightweight, and test creation should take very little time,
because instead test creation doesn't have sense.\
So here is first version, it use new features from C++0x, and was
created in Visual Studio2010 Express.It produce output to standard
Output window, and you can click on failed assertion message to move you
code editor to required code line.

Test.h:

    #pragma once 

    #include < sstream>

    #define CHECK(v) self->Assert(##v, L#v, __FILE__, __LINE__);

    #define CHECK_EQUALS(v1, v2) self->AssertEquals(##v1,##v2, L#v1, L#v2, __FILE__, __LINE__);

    #define CHECK_NOT_EQUALS(v1, v2) self->AssertNotEquals(##v1,##v2, L#v1, L#v2, __FILE__, __LINE__);

    #define TEST(name) void name(testengine::Test* self)

    namespace testengine
    {
         class Test
         {
         public: 
             Test() : done(true) {self = this;}
             virtual ~Test(){}
             virtual void Setup(){}
             virtual void TearDown(){}
             virtual void Run() = 0; 

             void Assert(bool rez, const std::wstring& msg, const std::string& fileName, size_t line)
             {
                 if (!rez)
                 {
                         std::wstringstream buf;
                         buf < < fileName.c_str() < < L"(" < < line < < ") : Assertation failed : " < < msg.c_str() < < std::endl;
                         report += buf.str();
                         done = false;
                 } 
             }

             template < class T1, class T2> void AssertEquals(T1 v1, T2 v2, const std::wstring& msg1, const std::wstring& msg2, const std::string& fileName, size_t line)
             {
                 std::wstringstream buf;
                 buf < < msg1 < < L" == " < < msg2;
                 Assert(v1 == v2, buf.str(), fileName, line);
             }

             template < class T1, class T2> void AssertNotEquals(T1 v1, T2 v2, const std::wstring& msg1, const std::wstring& msg2, const std::string& fileName, size_t line)
             {
                 std::wstringstream buf;
                 buf < < msg1 < < L" != " < < msg2;
                 Assert(v1 != v2, buf.str(), fileName, line);
             }

             const std::wstring GetReport() {return report;}

             bool IsDone() {return done;}
         protected: 
             Test* self;
         private: 
             std::wstring report;
             bool done;
         };
    }

TestExecuter.h:

    #pragma once 

    #include "Test.h"

    #include <memory>
    #include <vector>
    #include <string>
    #include <tuple>
    #include <sstream>
    #include <functional>

    #include < windows.h>

    namespace testengine
    {
    typedef std::function< void(Test*)> TestFunction;

    class SingleFunctionTest : public Test
    {
    public:    
        SingleFunctionTest(TestFunction func) : func(func){} virtual void Run() 
        {
            func(this); 
        }

    private:    
        TestFunction func;
    };

    class TestExecuter
    {
    public:
        void AddTest(TestFunction func, const std::wstring& name) 
        {
            tests.push_back(std::make_tuple(name, new SingleFunctionTest(func))); 
        }

        template< class T> void AddTest(const std::wstring& name) 
        {
            tests.push_back(std::make_tuple(name, new T));  
        }

        void RunTests() 
        {
            OutputDebugStringW(L"=============================================================================n");        
            OutputDebugStringW(L"Start running tests :n"); 
            auto i = tests.begin(), e = tests.end(); 
            size_t passed = 0; 
            size_t failed = 0; 
            for (;i != e; ++i) 
            {
                OutputDebugStringW(L"-----------------------------------------------------------------------------n"); 
                bool go = true; 
                std::wstringstream buf;            
                buf < <  std::get< 0>(*i); 
                try 
                { 
                    std::get< 1>(*i)->Setup();  
                } 
                catch(...) 
                {
                    buf < <  L" - Fails : Unhandled exception while Setupn";
                    OutputDebugStringW(buf.str().c_str());
                    go = false; ++failed; 
                }

                if (go) 
                { 
                    try { std::get< 1>(*i)->Run();  
                } 
                catch(...)
                {
                    buf < <  L" - Fails : Unhandled exception while Runn";
                    OutputDebugStringW(buf.str().c_str());
                    go = false; ++failed; 
                }
            }
            if (go) 
            {  
                try 
                { 
                    std::get< 1>(*i)->TearDown();  
                } 
                catch(...) 
                {
                    buf < <  L" - Fails : n";
                    buf < <  std::get< 1>(*i)->GetReport();
                    buf < <  L"Unhandled exception while TearDownn";
                    OutputDebugStringW(buf.str().c_str());
                    go = false; ++failed; 
                }
            }

            if (go)
            { 
                if (!std::get< 1>(*i) ->IsDone()) 
                {
                    buf < <  L" - Fails : n" < <  std::get< 1>(*i)->GetReport();
                    OutputDebugStringW(buf.str().c_str());
                    ++failed;
                }
                else 
                {
                    buf < <  L" - OKn";
                    OutputDebugStringW(buf.str().c_str()); ++passed;
                }
            }
        }
        OutputDebugStringW(L"-----------------------------------------------------------------------------n"); 
        std::wstringstream buf;
        buf < <  L"All tests finished : passed = " < <  passed < <  L" failed = " < <  failed < <  std::endl;
        OutputDebugStringW(buf.str().c_str());
        OutputDebugStringW(L"=============================================================================n"); }

    private: 
        std::vector< std::tuple< std::wstring, std::shared_ptr< Test> > > tests;
    };
    }

main.cpp:

    #include "TestEngineTestExecuter.h"

    class MyTest1 : public testengine::Test
    {
    public: 
        virtual void Run()
        {
            CHECK(1 == 0); 
        }
    };

    class MyTest2 : public testengine::Test
    {
    public: 
        virtual void Run()
        {
            CHECK(1 == 1);
        }
    };

    class MyTest3 : public testengine::Test
    {
    public: 
        virtual void Run()
        {
            CHECK_EQUALS(1, 2); 
        }
    };

    class MyTest4 : public testengine::Test
    {
    public: 
        virtual void TearDown()
        {
            throw 1; 
        }
        virtual void Run()
        {
            CHECK_EQUALS(3, 4); 
        }
    };

    class MyTest5 : public testengine::Test
    {
    public:
        virtual void Setup() { throw 1; }
        virtual void TearDown() { throw 1; }
        virtual void Run()
        {
            CHECK_EQUALS(3, 4); 
        }
    };

    class MyTest6 : public testengine::Test
    {
    public:
        virtual void TearDown() { throw 1; }
        virtual void Run() 
        { 
            throw 1;
            CHECK_EQUALS(3, 4); 
        }
    };

    TEST(MyTest7)
    {
        CHECK_NOT_EQUALS(4, 4);
    }

    int main(int argc, char* argv[])
    {
        testengine::TestExecuter testExec;

        testExec.AddTest< MyTest1>(L"MyTest1");
        testExec.AddTest< MyTest2>(L"MyTest2");
        testExec.AddTest< MyTest3>(L"MyTest3");
        testExec.AddTest< MyTest4>(L"MyTest4");
        testExec.AddTest< MyTest5>(L"MyTest5");
        testExec.AddTest< MyTest6>(L"MyTest6");
        testExec.AddTest(MyTest7, L"MyTest7");
        testExec.RunTests();
        return 0;
     }
