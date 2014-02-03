---
title: How to make reusable buffer for C++ streaming operations
layout: post
tags: C++, parsing, stream
---


Here is an example how make parsing number in C++ without temporary
string buffer, which sometimes needed when you use std::stringstream.

    #include <string>
    #include <vector>
    #include <streambuf>
    #include <sstream>
    #include <type_traits>

    struct mystreambuf : public std::streambuf 
    {
        mystreambuf(char_type* buffer, std::streamsize bufferLength)
        {
            setg(buffer, buffer, buffer + bufferLength);
        }
    protected:
        virtual pos_type seekoff(off_type offset, std::ios_base::seekdir direction, std::ios_base::openmode)
        {
            off_type curpos = gptr() - eback();
            std::streamsize bufferLength = egptr() - eback();
            off_type newpos = -1;

            if(direction == std::ios::cur) 
            {
                newpos = curpos + offset;
            }
            else if (direction == std::ios::end) 
            {
                newpos = bufferLength - offset;
            }
            else
            {
                newpos = offset;
            }

            setg(eback(), eback() + newpos, egptr());
            return newpos;
        }

        virtual pos_type seekpos(pos_type , std::ios_base::openmode )
        {
            return (std::streampos(std::_BADOFF));
        }
    };

    int main(int arcg, char* argv[])
    {
        auto msg = "123.45";
        auto length = sizeof(msg);
        std::vector<char> buffer(msg, msg + length);

        mystreambuf streamBuffer(buffer.data(), length);

        std::istream localStream(&streamBuffer);

        double res = 0;
        localStream.seekg(1, localStream.beg);
        localStream >> res;

        localStream.clear();
        localStream.seekg(0, localStream.beg);

        res = 0;
        localStream >> res;

        auto msg2 = "33.5";
        length = sizeof(msg2);
        buffer.assign(msg2, msg2 + length);

        localStream.clear();
        localStream.seekg(0, localStream.beg);

        res = 0;
        localStream >> res;

        return 0;
    }
