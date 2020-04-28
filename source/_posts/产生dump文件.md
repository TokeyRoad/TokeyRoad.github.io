---
title: 产生dump文件
date: 2020-04-28 09:31:07
tags:
  - C++
categories: 
- C++
copyright: true
---

#### 程序崩溃或发生异常时产生dump文件

核心API是：

```C++
CreateFile()
MinDumpWriteDump()
```

需要包含的头文件：

```C++
#include <DbgHelp.h>
#pragma comment(lib, "dbghelp.lib")
```

代码如下：

```C++
#define _CRT_SECURE_NO_WARNINGS
#include <windows.h>
#include <stdlib.h>
#include <string>

#include <DbgHelp.h>

#pragma comment(lib, "dbghelp.lib")



LONG WINAPI MyCustomUnhandledFilter(struct _EXCEPTION_POINTERS *lpExceptionInfo)
{
	LONG iRet = EXCEPTION_EXECUTE_HANDLER;

	TCHAR szDumpFileName[MAX_PATH] = { 0 };

	SYSTEMTIME st = { 0 };
	GetLocalTime(&st);
	wsprintf(szDumpFileName, "%04d-%02d-%02d-%02d-%02d-%02d.dmp", st.wYear, st.wMonth, st.wDay, st.wHour, st.wMinute, st.wMinute);

	HANDLE hDumpFile = CreateFile(szDumpFileName, GENERIC_WRITE, FILE_SHARE_READ, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);

	if (hDumpFile == INVALID_HANDLE_VALUE)
	{
		DWORD dwErrorID = GetLastError();
		printf("Failed to create dump file, error ID: %d\n", dwErrorID);

		return iRet;
	}

	MINIDUMP_EXCEPTION_INFORMATION MindumpExceptionInfo = { 0 };
	MindumpExceptionInfo.ThreadId = GetCurrentThreadId();
	MindumpExceptionInfo.ExceptionPointers = lpExceptionInfo;
	MindumpExceptionInfo.ClientPointers = FALSE;

	BOOL bRet = MiniDumpWriteDump(GetCurrentProcess(), GetCurrentProcessId(), hDumpFile, MiniDumpNormal, &MindumpExceptionInfo, NULL, NULL);

	if (bRet)
	{
		printf("Succeeded to create dump file!\n");
	}
	else
	{
		printf("Failed to create dump file!\n");
	}

	CloseHandle(hDumpFile);

	return iRet;
}

void crash(int a)
{
	int i = 1;
	int j = a;
	//！！程序崩溃的地方
	i /= j;
}

int main() {
	SetUnhandledExceptionFilter(MyCustomUnhandledFilter);
	char buf[10];
	memset(buf, 0, sizeof(buf) * sizeof(char));
	crash(0);
	strcpy(buf, "123456789");

	system("pause");
	return 0;
}
```

