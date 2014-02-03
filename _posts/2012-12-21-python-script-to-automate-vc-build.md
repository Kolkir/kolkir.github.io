---
title: Python script to automate VC++ build
layout: post
tags: 
---


Here is example of the Python script which can be used to automate VC++
compilation and build process. It sequentially build VC Solutions and
write short results to console window. If there were errors during
compilation it will show them(only errors) and asks about continuing
general build process.

    buildDirDebug    = "D:\\development\\Debug"
    buildDirRelease  = "D:\\development\\Release"

    binaryDir = "SourceCode\\Bin"
    codeDir   = "SourceCode"
    devenv    = "\"C:\\Program Files (x86)\\Microsoft Visual Studio 10.0\\Common7\\IDE\\devenv.com\""
    logFile   = "build.log"

    # path relative to "codeDir"
    solutions = [
    "Proj1\solution1.sln",
    "Proj2\solution2.sln",
    "Proj3\solution3.sln",
    ]

    buildModeDebug   = "Debug"
    buildModeRelease = "Release"

    #------------------------------------------------------------------------------------------------------

    import sys
    import subprocess
    from threading  import Thread
    from Queue import Queue, Empty
    import colorama
    from colorama import Fore, Back, Style

    buildDir  = buildDirDebug
    buildMode = buildModeDebug

    answer = raw_input('Debug/Release?[d/r]:')
    if answer.lower() == "r":
        buildDir  = buildDirRelease
        buildMode = buildModeRelease

    colorama.init()

    outEncoding = "cp866"

    spaces = "    "

    def enqueue_output(out, queue):
        for line in iter(out.readline, b''):
            queue.put(line)
        out.close()

    outFile = open(logFile,"wb")

    def RunCommand(strCommand):
        cmd = subprocess.Popen(strCommand, shell=True, bufsize=1, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        q = Queue()
        t = Thread(target=enqueue_output, args=(cmd.stdout, q))
        t.start()

        while True:
            code = cmd.poll()
            try:
                line = q.get_nowait()
            except Empty:
                pass
                #skip
            else:           
                outFile.write(line)
                text = line.decode(outEncoding)
                if text.find("error ") != -1 :
                    print(text)
            if (code != None):
                break

        outFile.flush()
        resultStr = ""
        if code != 0:
            for line in cmd.stderr:
                text = line.decode(outEncoding)
                resultStr += "\n" + spaces + spaces + text

        return code == 0, resultStr

    def DoCommand(strName, strCommand):
        print(strName + "\n")
        ok, resStr = RunCommand(strCommand)
        if ok:
            print(spaces + Fore.GREEN + "Done")
        else:
            print(spaces + Fore.RED + "Failed: " + strCommand)
            print(resStr)

            cont = False
            answer = raw_input('Continue?[y/n]:')
            if answer.lower() == "n":
                exit(1)

        print(Fore.RESET + Back.RESET + Style.RESET_ALL)

    DoCommand("Clear binaries", "rmdir /s /q \"" + buildDir + "\\" + binaryDir + "\"")
    DoCommand("SVN clear", "\"C:\\Program Files\\TortoiseSVN\\bin\\TortoiseProc.exe\" /command:cleanup /path:\"" + buildDir + "\" /noui /delunversioned /delignored")
    DoCommand("Creating bin directory", "mkdir \"" + buildDir + "\\" + binaryDir + "\"")

    for s in solutions:
        DoCommand("Compile  - " + s, devenv + " \"" + buildDir + "\\" + codeDir + "\\" + s + "\"" + " /rebuild \"" + buildMode + "|x64\"")

    raw_input('Press any key to finish')
