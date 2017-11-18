---
title: JDK1.6 Runtime.getRuntime().exec(cmd)执行错误：Cannot allocate memory
tags:
  - C++
  - exec
  - Java
  - 源码
id: 57
categories:
  - 笔记
date: 2014-08-21 10:34:59
---

<span style="color: #000000;">最近遇到一个问题，在程序中通过Runtime.getRuntime().exec(cmd)调用Linux中的脚本文件执行脚本抛出了异常：
</span>

<pre class="lang:default decode:true ">java.io.IOException: Cannot run program "sh": java.io.IOException: error=12, Cannot allocate memory  
        at java.lang.ProcessBuilder.start(ProcessBuilder.java:459)  
        at java.lang.Runtime.exec(Runtime.java:593)  
        at java.lang.Runtime.exec(Runtime.java:431)  
        at java.lang.Runtime.exec(Runtime.java:328)  
    ...  
        at org.springframework.scheduling.support.DelegatingErrorHandlingRunnable.run(DelegatingErrorHandlingRunnable.java:51)  
        at org.springframework.scheduling.concurrent.ReschedulingRunnable.run(ReschedulingRunnable.java:81)  
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:441)  
        at java.util.concurrent.FutureTask$Sync.innerRun(FutureTask.java:303)  
        at java.util.concurrent.FutureTask.run(FutureTask.java:138)  
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:98)  
        at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:207)  
        at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)  
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)  
        at java.lang.Thread.run(Thread.java:619)  
Caused by: java.io.IOException: java.io.IOException: error=12, Cannot allocate memory  
        at java.lang.UNIXProcess.&lt;init&gt;(UNIXProcess.java:148)  
        at java.lang.ProcessImpl.start(ProcessImpl.java:65)  
        at java.lang.ProcessBuilder.start(ProcessBuilder.java:452)  
        ... 15 more</pre>

<!--more-->
很是奇怪，应为该程序在其它机器上运行时并没有此异常抛出。
Cannot allocate memory说明无法分配内存。
查看两台机器的运行环境，JVM的配置是相同的，只有机器的配置不同，正常运行的机器是32G内存，异常机器是16G内存，难道真是内存不足造成的？
该程序正常运行峰值最多占用4G内存，JVM开启的最大内存是6644m，理论上应该不会出现内存不足的情况。
最后通过在网上查找，找到了大概的问题所在：Java程序调用外部程序时可能需要分配跟父进程同等大小的内存。
为什么会这么分配内存呢？
看看源码：
java.lang.ProcessImpl

<pre class="lang:java decode:true">private ProcessImpl(String cmd[],  
            String envblock,  
            String path,  
            boolean redirectErrorStream)  
    throws IOException  
    {  
        // The [executablePath] is not quoted for any case.          
        String executablePath = getExecutablePath(cmd[0]);  

        // We need to extend the argument verification procedure   
        // to guarantee the only command file execution for implicit [cmd.exe]  
        // run.  
        String upPath = executablePath.toUpperCase();  
        boolean isCmdFile = (upPath.endsWith(".CMD") || upPath.endsWith(".BAT"));  

        StringBuilder cmdbuf = new StringBuilder(80);  

        // Quotation protects from interpretation of the [path] argument as   
        // start of longer path with spaces. Quotation has no influence to   
        // [.exe] extension heuristic.  
        cmdbuf.append('"');  
        cmdbuf.append(executablePath);  
        cmdbuf.append('"');  

        for (int i = 1; i &lt; cmd.length; i++) {  
            cmdbuf.append(' ');  
            String s = cmd[i];  
            if (needsEscaping(isCmdFile, s)) {  
                cmdbuf.append('"');  
                cmdbuf.append(s);  

                // The code protects the [java.exe] and console command line   
                // parser, that interprets the [\"] combination as an escape   
                // sequence for the ["] char.   
                //     http://msdn.microsoft.com/en-us/library/17w5ykft.aspx  
                //  
                // If the argument is an FS path, doubling of the tail [\]   
                // char is not a problem for non-console applications.  
                //  
                // The [\"] sequence is not an escape sequence for the [cmd.exe]  
                // command line parser. The case of the [""] tail escape   
                // sequence could not be realized due to the argument validation   
                // procedure.  
                if (!isCmdFile &amp;&amp; s.endsWith("\\")) {  
                    cmdbuf.append('\\');  
                }  
                cmdbuf.append('"');  
            } else {  
                cmdbuf.append(s);  
            }  
        }  
    String cmdstr = cmdbuf.toString();  

    stdin_fd  = new FileDescriptor();  
    stdout_fd = new FileDescriptor();  
    stderr_fd = new FileDescriptor();  

    handle = create(cmdstr, envblock, path, redirectErrorStream,  
            stdin_fd, stdout_fd, stderr_fd);  

    java.security.AccessController.doPrivileged(  
        new java.security.PrivilegedAction() {  
        public Object run() {  
        stdin_stream =  
            new BufferedOutputStream(new FileOutputStream(stdin_fd));  
        stdout_stream =  
            new BufferedInputStream(new FileInputStream(stdout_fd));  
        stderr_stream =  
            new FileInputStream(stderr_fd);  
        return null;  
        }  
    });  
    }  
    private native long create(String cmdstr,  
                   String envblock,  
                   String dir,  
                   boolean redirectErrorStream,  
                   FileDescriptor in_fd,  
                   FileDescriptor out_fd,  
                   FileDescriptor err_fd)  
    throws IOException;</pre>

调用了JNI方法，继续查JNI的源码：
Windows JVM 实现：

<pre class="lang:c++ decode:true">JNIEXPORT jlong JNICALL  
Java_java_lang_ProcessImpl_create(JNIEnv *env, jclass ignored,  
                                  jstring cmd,  
                                  jstring envBlock,  
                                  jstring dir,  
                                  jboolean redirectErrorStream,  
                                  jobject in_fd,  
                                  jobject out_fd,  
                                  jobject err_fd)  
{  
    HANDLE inRead   = 0;  
    HANDLE inWrite  = 0;  
    HANDLE outRead  = 0;  
    HANDLE outWrite = 0;  
    HANDLE errRead  = 0;  
    HANDLE errWrite = 0;  
    SECURITY_ATTRIBUTES sa;  
    PROCESS_INFORMATION pi;  
    STARTUPINFO si;  
    LPTSTR  pcmd      = NULL;  
    LPCTSTR pdir      = NULL;  
    LPVOID  penvBlock = NULL;  
    jlong ret = 0;  
    OSVERSIONINFO ver;  
    jboolean onNT = JNI_FALSE;  
    DWORD processFlag;  

    ver.dwOSVersionInfoSize = sizeof(ver);  
    GetVersionEx(&amp;ver);  
    if (ver.dwPlatformId == VER_PLATFORM_WIN32_NT)  
        onNT = JNI_TRUE;  

    sa.nLength = sizeof(sa);  
    sa.lpSecurityDescriptor = 0;  
    sa.bInheritHandle = TRUE;  

    if (!(CreatePipe(&amp;inRead,  &amp;inWrite,  &amp;sa, PIPE_SIZE) &amp;&amp;  
          CreatePipe(&amp;outRead, &amp;outWrite, &amp;sa, PIPE_SIZE) &amp;&amp;  
          CreatePipe(&amp;errRead, &amp;errWrite, &amp;sa, PIPE_SIZE))) {  
        win32Error(env, "CreatePipe");  
        goto Catch;  
    }  

    assert(cmd != NULL);  
    pcmd = (LPTSTR) JNU_GetStringPlatformChars(env, cmd, NULL);  
    if (pcmd == NULL) goto Catch;  

    if (dir != 0) {  
        pdir = (LPCTSTR) JNU_GetStringPlatformChars(env, dir, NULL);  
        if (pdir == NULL) goto Catch;  
        pdir = (LPCTSTR) JVM_NativePath((char *)pdir);  
    }  

    if (envBlock != NULL) {  
        penvBlock = onNT  
            ? (LPVOID) ((*env)-&gt;GetStringChars(env, envBlock, NULL))  
            : (LPVOID) JNU_GetStringPlatformChars(env, envBlock, NULL);  
        if (penvBlock == NULL) goto Catch;  
    }  

    memset(&amp;si, 0, sizeof(si));  
    si.cb = sizeof(si);  
    si.dwFlags = STARTF_USESTDHANDLES;  
    si.hStdInput  = inRead;  
    si.hStdOutput = outWrite;  
    si.hStdError  = redirectErrorStream ? outWrite : errWrite;  

    SetHandleInformation(inWrite, HANDLE_FLAG_INHERIT, FALSE);  
    SetHandleInformation(outRead, HANDLE_FLAG_INHERIT, FALSE);  
    SetHandleInformation(errRead, HANDLE_FLAG_INHERIT, FALSE);  

    if (redirectErrorStream)  
        SetHandleInformation(errWrite, HANDLE_FLAG_INHERIT, FALSE);  

    if (onNT)  
        processFlag = CREATE_NO_WINDOW | CREATE_UNICODE_ENVIRONMENT;  
    else  
        processFlag = selectProcessFlag(env, cmd);  

    /* Java and Windows are both pure Unicode systems at heart. 
     * Windows has both a legacy byte-based API and a 16-bit Unicode 
     * "W" API.  The Right Thing here is to call CreateProcessW, since 
     * that will allow all process-related information like command 
     * line arguments to be passed properly to the child.  We don't do 
     * that currently, since we would first have to have "W" versions 
     * of JVM_NativePath and perhaps other functions.  In the 
     * meantime, we can call CreateProcess with the magic flag 
     * CREATE_UNICODE_ENVIRONMENT, which passes only the environment 
     * in "W" mode.  We will fix this later. */  

    ret = CreateProcess(0,           /* executable name */  
                        pcmd,        /* command line */  
                        0,           /* process security attribute */  
                        0,           /* thread security attribute */  
                        TRUE,        /* inherits system handles */  
                        processFlag, /* selected based on exe type */  
                        penvBlock,   /* environment block */  
                        pdir,        /* change to the new current directory */  
                        &amp;si,         /* (in)  startup information */  
                        &amp;pi);        /* (out) process information */  

    if (!ret) {  
        win32Error(env, "CreateProcess");  
        goto Catch;  
    }  

    CloseHandle(pi.hThread);  
    ret = (jlong)pi.hProcess;  
    (*env)-&gt;SetLongField(env, in_fd,  IO_handle_fdID, (jlong)inWrite);  
    (*env)-&gt;SetLongField(env, out_fd, IO_handle_fdID, (jlong)outRead);  
    (*env)-&gt;SetLongField(env, err_fd, IO_handle_fdID, (jlong)errRead);  

 Finally:  
    /* Always clean up the child's side of the pipes */  
    closeSafely(inRead);  
    closeSafely(outWrite);  
    closeSafely(errWrite);  

    if (pcmd != NULL)  
        JNU_ReleaseStringPlatformChars(env, cmd, (char *) pcmd);  
    if (pdir != NULL)  
        JNU_ReleaseStringPlatformChars(env, dir, (char *) pdir);  
    if (penvBlock != NULL) {  
        if (onNT)  
            (*env)-&gt;ReleaseStringChars(env, envBlock, (jchar *) penvBlock);  
        else  
            JNU_ReleaseStringPlatformChars(env, dir, (char *) penvBlock);  
    }  
    return ret;  

 Catch:  
    /* Clean up the parent's side of the pipes in case of failure only */  
    closeSafely(inWrite);  
    closeSafely(outRead);  
    closeSafely(errRead);  
    goto Finally;  
}</pre>

JVM Linux实现：

<pre class="lang:c++ decode:true ">#ifndef __solaris__  
#undef fork1  
#define fork1() fork()  
#endif  

JNIEXPORT jint JNICALL  
Java_java_lang_UNIXProcess_forkAndExec(JNIEnv *env,  
                                       jobject process,  
                                       jbyteArray prog,  
                                       jbyteArray argBlock, jint argc,  
                                       jbyteArray envBlock, jint envc,  
                                       jbyteArray dir,  
                                       jboolean redirectErrorStream,  
                                       jobject stdin_fd,  
                                       jobject stdout_fd,  
                                       jobject stderr_fd)  
{  
    int errnum;  
    int resultPid = -1;  
    int in[2], out[2], err[2], fail[2];  
    const char **argv = NULL;  
    const char **envv = NULL;  
    const char *pprog     = getBytes(env, prog);  
    const char *pargBlock = getBytes(env, argBlock);  
    const char *penvBlock = getBytes(env, envBlock);  
    const char *pdir      = getBytes(env, dir);  

    in[0] = in[1] = out[0] = out[1] = err[0] = err[1] = fail[0] = fail[1] = -1;  

    assert(prog != NULL &amp;&amp; argBlock != NULL);  
    if (pprog     == NULL) goto Catch;  
    if (pargBlock == NULL) goto Catch;  
    if (envBlock  != NULL &amp;&amp; penvBlock == NULL) goto Catch;  
    if (dir       != NULL &amp;&amp; pdir      == NULL) goto Catch;  

    /* Convert pprog + pargBlock into a char ** argv */  
    if ((argv = NEW(const char *, argc + 2)) == NULL)  
        goto Catch;  
    argv[0] = pprog;  
    initVectorFromBlock(argv+1, pargBlock, argc);  

    if (envBlock != NULL) {  
        /* Convert penvBlock into a char ** envv */  
        if ((envv = NEW(const char *, envc + 1)) == NULL)  
            goto Catch;  
        initVectorFromBlock(envv, penvBlock, envc);  
    }  

    if ((pipe(in)   &lt; 0) ||  
        (pipe(out)  &lt; 0) ||  
        (pipe(err)  &lt; 0) ||  
        (pipe(fail) &lt; 0)) {  
        throwIOException(env, errno, "Bad file descriptor");  
        goto Catch;  
    }  

    resultPid = fork1();  
    if (resultPid &lt; 0) {  
        throwIOException(env, errno, "Fork failed");  
        goto Catch;  
    }  

    if (resultPid == 0) {  
        /* Child process */  

        /* Close the parent sides of the pipe. 
           Give the child sides of the pipes the right fileno's. 
           Closing pipe fds here is redundant, since closeDescriptors() 
           would do it anyways, but a little paranoia is a good thing. */  
        /* Note: it is possible for in[0] == 0 */  
        close(in[1]);  
        moveDescriptor(in[0], STDIN_FILENO);  
        close(out[0]);  
        moveDescriptor(out[1], STDOUT_FILENO);  
        close(err[0]);  
        if (redirectErrorStream) {  
            close(err[1]);  
            dup2(STDOUT_FILENO, STDERR_FILENO);  
        } else {  
            moveDescriptor(err[1], STDERR_FILENO);  
        }  
        close(fail[0]);  
        moveDescriptor(fail[1], FAIL_FILENO);  

        /* close everything */  
        if (closeDescriptors() == 0) { /* failed,  close the old way */  
            int max_fd = (int)sysconf(_SC_OPEN_MAX);  
            int i;  
            for (i = FAIL_FILENO + 1; i &lt; max_fd; i++)  
                close(i);  
        }  

        /* change to the new working directory */  
        if (pdir != NULL &amp;&amp; chdir(pdir) &lt; 0)  
            goto WhyCantJohnnyExec;  

        if (fcntl(FAIL_FILENO, F_SETFD, FD_CLOEXEC) == -1)  
            goto WhyCantJohnnyExec;  

        execvpe(argv[0], argv, envv);  

    WhyCantJohnnyExec:  
        /* We used to go to an awful lot of trouble to predict whether the 
         * child would fail, but there is no reliable way to predict the 
         * success of an operation without *trying* it, and there's no way 
         * to try a chdir or exec in the parent.  Instead, all we need is a 
         * way to communicate any failure back to the parent.  Easy; we just 
         * send the errno back to the parent over a pipe in case of failure. 
         * The tricky thing is, how do we communicate the *success* of exec? 
         * We use FD_CLOEXEC together with the fact that a read() on a pipe 
         * yields EOF when the write ends (we have two of them!) are closed. 
         */  
        errnum = errno;  
        write(FAIL_FILENO, &amp;errnum, sizeof(errnum));  
        close(FAIL_FILENO);  
        _exit(-1);  
    }  

    /* parent process */  

    close(fail[1]); fail[1] = -1; /* See: WhyCantJohnnyExec */  

    switch (readFully(fail[0], &amp;errnum, sizeof(errnum))) {  
    case 0: break; /* Exec succeeded */  
    case sizeof(errnum):  
        waitpid(resultPid, NULL, 0);  
        throwIOException(env, errnum, "Exec failed");  
        goto Catch;  
    default:  
        throwIOException(env, errno, "Read failed");  
        goto Catch;  
    }  

    (*env)-&gt;SetIntField(env, stdin_fd,  IO_fd_fdID, in [1]);  
    (*env)-&gt;SetIntField(env, stdout_fd, IO_fd_fdID, out[0]);  
    (*env)-&gt;SetIntField(env, stderr_fd, IO_fd_fdID, err[0]);  

 Finally:  
    /* Always clean up the child's side of the pipes */  
    closeSafely(in [0]);  
    closeSafely(out[1]);  
    closeSafely(err[1]);  

    /* Always clean up fail descriptors */  
    closeSafely(fail[0]);  
    closeSafely(fail[1]);  

    free(argv); http://  
    free(envv);  

    releaseBytes(env, prog,     pprog);  
    releaseBytes(env, argBlock, pargBlock);  
    releaseBytes(env, envBlock, penvBlock);  
    releaseBytes(env, dir,      pdir);  

    return resultPid;  

 Catch:  
    /* Clean up the parent's side of the pipes in case of failure only */  
    closeSafely(in [1]);  
    closeSafely(out[0]);  
    closeSafely(err[0]);  
    goto Finally;  
}</pre>

linux下的实现（line:57）是调用fork1()函数，就是调用fork指令
fork子进程要传递父进程的所有句柄资源，fork后面紧跟exec，一般copy整个父进程空间的数据到子进程中。
所以fork需要分配与父进程相同的内存。因此，当我调用在程序中调用exec()时，需要一次性分配6644m的内存，系统内存早已不足。

参考：
[http://www.w3china.org/blog/more.asp?name=hongrui&amp;id=44142](http://www.w3china.org/blog/more.asp?name=hongrui&amp;id=44142 "http://www.w3china.org/blog/more.asp?name=hongrui&amp;id=44142")
[http://www.myexception.cn/program/1234642.html](http://www.myexception.cn/program/1234642.html "http://www.myexception.cn/program/1234642.html")