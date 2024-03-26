

1\. [dump](https://so.csdn.net/so/search?q=dump&spm=1001.2101.3001.7020)文件生成
----------------------------------------------------------------------------

利用下面程序（debug x86编译，release模式下需修改工程部分配置以生成pdb信息），模拟程序崩溃。

```null
static LONG WINAPI CretateDmpFile(struct _EXCEPTION_POINTERS* pExPointInfo)	char szFile[1240] = { 0 };	strcpy_s(szFile, "file.dmp");	HANDLE hFile = CreateFileA(szFile, GENERIC_WRITE, FILE_SHARE_WRITE, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);if (hFile != INVALID_HANDLE_VALUE)		MINIDUMP_EXCEPTION_INFORMATION miniExInfo;		miniExInfo.ThreadId = ::GetCurrentThreadId();		miniExInfo.ExceptionPointers = pExPointInfo;		miniExInfo.ClientPointers = NULL;		BOOL bOk = MiniDumpWriteDump(GetCurrentProcess(), GetCurrentProcessId(), hFile, MiniDumpNormal, &miniExInfo, NULL, NULL);return EXCEPTION_EXECUTE_HANDLER;	SetUnhandledExceptionFilter(CretateDmpFile);
```

编译成功后，运行windbgTest.exe，生成如下图所示的一个file.dmp文件。

![](https://img-blog.csdnimg.cn/20201216225007999.png)

2\. [windbg](https://so.csdn.net/so/search?q=windbg&spm=1001.2101.3001.7020)设置
------------------------------------------------------------------------------

### 2.1 设置符号路径

file->Symbol file path;

![](https://img-blog.csdnimg.cn/20201216230518852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19kb25n,size_16,color_FFFFFF,t_70)

设置符号路径为:E:\\c++\\windbg\\test1\\windbgTest\\Debug;SRV\*E:\\c++\\windbg\\test1\\windbgTest\\Debug\\symbols\*http://msdl.microsoft.com/download/symbols，可设置多个路径，用分号隔开。

1）E:\\c++\\windbg\\test1\\windbgTest\\Debug：程序pdb文件路径

2）E:\\c++\\windbg\\test1\\windbgTest\\Debug\\symbols：符号生成路径

3）http://msdl.microsoft.com/download/symbols: 微软的PE文件对应的符号文件。

### 2.2 设置源文件路径

file->Source FIle Path

![](https://img-blog.csdnimg.cn/20201216225556129.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19kb25n,size_16,color_FFFFFF,t_70)

### 2.3  打开file.dmp文件和源文件

执行.ecxr可切换到异常发生时的堆栈。

![](https://img-blog.csdnimg.cn/20201216230029781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19kb25n,size_16,color_FFFFFF,t_70)

也可通过执行!analyze -v获取分析信息,可查找到源码中出现问题的位置。

![](https://img-blog.csdnimg.cn/20201216230605170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d3d19kb25n,size_16,color_FFFFFF,t_70)

3\. 参考资料
--------

一篇很不错的介绍windbg基础的文章: [https://www.cnblogs.com/kekec/archive/2012/11/14/2755924.html](https://www.cnblogs.com/kekec/archive/2012/11/14/2755924.html)

[**http://www.debuginfo.com/resources.html**](http://www.debuginfo.com/resources.html)

[**http://www.debuginfo.com/articles/debuginfomatch.html**](http://www.debuginfo.com/articles/debuginfomatch.html)

[**http://www.debuginfo.com/articles/gendebuginfo.html**](http://www.debuginfo.com/articles/gendebuginfo.html)

**后续将继续更新如何断点调试源代码和发生异常时的堆栈分析。**