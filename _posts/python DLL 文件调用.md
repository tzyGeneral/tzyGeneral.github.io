# python DLL 文件调用



最近想python直接调用fiddler的api，需要使用dll文件。下面给出python调用DLL文件遇到的一些问题.

> ctypes官网：https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#module-ctypes



#### 错误：OSError: [WinError 193] %1 不是有效的 Win32 应用程序。

```shell
Traceback (most recent call last):
  File "G:/AutoTool/driver/Instrument_dynamic.py", line 12, in <module>
    dll = cdll.LoadLibrary("Dll1.dll")
  File "E:\python\python3\lib\ctypes\__init__.py", line 426, in LoadLibrary
    return self._dlltype(name)
  File "E:\python\python3\lib\ctypes\__init__.py", line 348, in __init__
    self._handle = _dlopen(self._name, mode)
OSError: [WinError 193] %1 不是有效的 Win32 应用程序。

```

遇到这个问题的时候，是因为程序当当前的编辑运行环境不匹配，拿DLL文件调用来说，你在64位的python环境下调用32位的DLL文件肯定是不行的，这时你需要把python环境换成32位的。



#### 1. C/C++的DLL文件编写，简单的小示例。

```c++
#include "stdafx.h"
#include <windows.h>
#include <iostream>
typedef struct _SimpleStruct
{
	int    nNo;
	float  fVirus;
	char   szBuffer[2];
} SimpleStruct, *PSimpleStruct;
typedef const SimpleStruct*  PCSimpleStruct;
 
extern "C" int  __declspec(dllexport) PrintStruct(PSimpleStruct simp);
int PrintStruct(PSimpleStruct simp)
{
	printf("nMaxNum=%f,szContent=%s", simp->fVirus, simp->szBuffer);
	return simp->nNo;
}
```



#### python调用编写好的DLL文件，可以帮助理解DLL文件的调用。

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
# author:SingWeek
 
from ctypes import *
class SimpStruct(Structure):
    _fields_ = [("nNo", c_int),
                ("fVirus", c_float),
                ("szBuffer", c_wchar)]#“”中的命名可以自己指定
 
dll = CDLL("Dll1.dll")
# print(dll.PrintStruct)
simple = SimpStruct()
simple.nNo = 16
simple.fVirus = 3.1415926
simple.szBuffer = "x"
out=dll.PrintStruct(byref(simple))
print(out)

```



#### 3、python下通过from ctypes import *调用DLL文件的注意事项：

​        想要正确调用dll文件，你要找到正确的调用约定，必须查看C头文件或要调用的函数的文档。整数、字节对象和（unicode）字符串是唯一可以在这些函数调用中直接用作参数的本地Python对象。它们都不作为C语言NULL指针传递，字节对象和字符串作为指针传递给包含它们的数据的内存块（char*或wchar_t*）。Python整数作为平台默认的C int类型传递，它们的值被屏蔽以适合于C类型。在继续使用其他参数类型调用函数之前，我们必须了解更多关于ctypes数据类型的信息。

​        Python ctypes 数据类型表

| **ctypes type**                                              | **C type**                             | **Python type**          |
| ------------------------------------------------------------ | -------------------------------------- | ------------------------ |
| [c_bool](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_bool) | _Bool                                  | bool (1)                 |
| [c_char](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_char) | char                                   | 1-character bytes object |
| [c_wchar](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_wchar) | wchar_t                                | 1-character string       |
| [c_byte](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_byte) | char                                   | int                      |
| [c_ubyte](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_ubyte) | unsigned char                          | int                      |
| [c_short](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_short) | short                                  | int                      |
| [c_ushort](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_ushort) | unsigned short                         | int                      |
| [c_int](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_int) | int                                    | int                      |
| [c_uint](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_uint) | unsigned int                           | int                      |
| [c_long](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_long) | long                                   | int                      |
| [c_ulong](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_ulong) | unsigned long                          | int                      |
| [c_longlong](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_longlong) | __int64 or long long                   | int                      |
| [c_ulonglong](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_ulonglong) | unsigned __int64 or unsigned long long | int                      |
| [c_size_t](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_size_t) | size_t                                 | int                      |
| [c_ssize_t](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_ssize_t) | ssize_t or Py_ssize_t                  | int                      |
| [c_float](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_float) | float                                  | float                    |
| [c_double](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_double) | double                                 | float                    |
| [c_longdouble](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_longdouble) | long double                            | float                    |
| [c_char_p](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_char_p) | char * (NUL terminated)                | bytes object or None     |
| [c_wchar_p](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_wchar_p) | wchar_t * (NUL terminated)             | string or None           |
| [c_void_p](https://docs.python.org/3.6/library/ctypes.html?highlight=ctypes#ctypes.c_void_p) | void *                                 | int or None              |

##### 4. function ‘test’ not found!，是因为dll文件中没有对应函数，The specified module could not be found或者function ‘***’ not found

```shell
AttributeError: function 'test' not found
```



##### 5. Procedure called with not enough arguments (** bytes missing) or wrong calling convention，在调用DLL文件中的对应函数的时候，参数传递出错，可能是参数少了，也可能是参数类型不对等引起的，此时需要核对DLL文件中的参数结构形式。

```shell
ValueError: Procedure called with not enough arguments(108 bytes missing)or worong calling convention
```



##### 6. 首先有这个函数会出现一串地址print(cdll.kernel32.GetModuleHandleA)，如果向dll调用的函数中传入的参数发生错误，会出现如下问题。

```shell
OSError: exception: access violation reading 0x000000020
```



##### 7. **重点**：

​    在dll文件中的调用方式有几种，他们分别对应不同的约定。其中cdll用于加载遵守cdecl标准函数调用约定的连接库；windll用于加载遵循stdcall调用约定的动态链接库，oledll和windll完全相同，只是会默认其载入的函数会统一返回一个windows result错误编码。在python中除了cdll，还可以用pydll功能相同，大小写的区别在于小写需要添加LoadLibrary进行dll文件调用；大写的可以直接将dll文件名放进局可以了。

​    stdcall调用约定：stdcall很多时候被称为pascal调用约定，因为pascal是早期很常见的一种教学用计算机程序设计语言，其语法严谨，使用的函数调用约定就是stdcall。在Microsoft C++系列的C/C++编译器中，常常用PASCAL宏来声明这个调用约定，类似的宏还有WINAPI和CALLBACK。stdcall调用约定声明的语法为(以前文的那个函数为例）：int __stdcall function(int a,int b)

​    stdcall的调用约定意味着：1）参数从右向左压入堆栈，2）函数自身修改堆栈 3)函数名自动加前导的下划线，后面紧跟一个@符号，其后紧跟着参数的尺寸。注意不同编译器会插入自己的汇编代码以提供编译的通用性，但是大体代码如此。

​    cdecl调用约定：cdecl调用约定又称为C调用约定，是C语言缺省的调用约定，它的定义语法是：

int function (int a ,int b) //不加修饰就是C调用约定，int __cdecl function(int a,int b)//明确指出C调用约定。cdecl调用约定的参数压栈顺序是和stdcall是一样的，参数首先由右向左压入堆栈。所不同的是，函数本身不清理堆栈，调用者负责清理堆栈。