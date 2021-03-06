# 一些常用函数

- [FindWindow](https://docs.microsoft.com/en-us/windows/desktop/api/winuser/nf-winuser-findwindoww)

```
HWND FindWindowW(
  LPCWSTR lpClassName,
  LPCWSTR lpWindowName
);
```

- [VirtualProtect]()

修改内存页属性

```
BOOL VirtualProtect(
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  flNewProtect,
  PDWORD lpflOldProtect
);
```

- [VirtualAllocEx](https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-virtualallocex)

在指定进程的虚拟空间保留或提交内存区域，除非指定MEM_RESET参数，否则将该内存区域置0。

```
LPVOID VirtualAllocEx(
  HANDLE hProcess,
  LPVOID lpAddress,				// 想要预定的地址，一般传NULL让系统自动找即可
  SIZE_T dwSize,
  DWORD  flAllocationType,		// 要预定区域还是要调拨物理存储器
  DWORD  flProtect				// 给区域指定保护属性
);

BOOL VirtualFreeEx(
  HANDLE hProcess,
  LPVOID lpAddress,
  SIZE_T dwSize,
  DWORD  dwFreeType
);
```

- [ReadProcessMemory/WriteProcessMemory](https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-readprocessmemory)

对另一个进程的地址空间进行读写

```
BOOL ReadProcessMemory(
  HANDLE  hProcess,
  LPCVOID lpBaseAddress,
  LPVOID  lpBuffer,
  SIZE_T  nSize,
  SIZE_T  *lpNumberOfBytesRead
);

BOOL WriteProcessMemory(
  HANDLE  hProcess,
  LPVOID  lpBaseAddress,
  LPCVOID lpBuffer,
  SIZE_T  nSize,
  SIZE_T  *lpNumberOfBytesWritten
);
```

- [GetModuleHandle](https://docs.microsoft.com/en-us/windows/desktop/api/libloaderapi/nf-libloaderapi-getmodulehandlew)

获取模块的句柄

```
HMODULE GetModuleHandleW(
  LPCWSTR lpModuleName
);
```

例：

```
HMODULE hModule = GetModuleHandle(L"Kernel32");
```

- [GetModuleInformation](https://docs.microsoft.com/zh-cn/windows/win32/api/psapi/nf-psapi-getmoduleinformation?redirectedfrom=MSDN)

获取指定模块信息

```
BOOL GetModuleInformation(
  HANDLE       hProcess,
  HMODULE      hModule,
  LPMODULEINFO lpmodinfo,
  DWORD        cb
);
```

例：

```
MODULEINFO modinfo = { 0 };
GetModuleInformation(GetCurrentProcess(), GetModuleHandleW(L"test.exe"), &modinfo, sizeof(MODULEINFO));
```

- [GetProcAddress](https://docs.microsoft.com/en-us/windows/desktop/api/libloaderapi/nf-libloaderapi-getprocaddress)

从dll中获取导出函数

```
FARPROC GetProcAddress(
  HMODULE hModule,
  LPCSTR  lpProcName
);
```

例：

```
pGNSI = (PGNSI) GetProcAddress(
	GetModuleHandle(TEXT("kernel32.dll")),
	"GetNativeSystemInfo");
```

- [LoadLibrary](https://docs.microsoft.com/en-us/windows/desktop/api/libloaderapi/nf-libloaderapi-loadlibraryw)

加载模块

```
HMODULE LoadLibraryW(
  LPCWSTR lpLibFileName
);
```

- [EnumProcess](https://docs.microsoft.com/zh-cn/windows/desktop/api/psapi/nf-psapi-enumprocesses)

```
BOOL EnumProcesses(
  DWORD   *lpidProcess,
  DWORD   cb,
  LPDWORD lpcbNeeded
);
```

- [OpenProcess](https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-openprocess)

```
HANDLE OpenProcess(
  DWORD dwDesiredAccess,
  BOOL  bInheritHandle,
  DWORD dwProcessId
);
```

- [EnumProcessModules](https://docs.microsoft.com/zh-cn/windows/desktop/api/psapi/nf-psapi-enumprocessmodules)

```
BOOL EnumProcessModules(
  HANDLE  hProcess,
  HMODULE *lphModule,
  DWORD   cb,
  LPDWORD lpcbNeeded
);
```

- [GetModuleBaseName](https://docs.microsoft.com/zh-cn/windows/desktop/api/psapi/nf-psapi-getmodulebasenamew)

```
DWORD GetModuleBaseNameW(
  HANDLE  hProcess,
  HMODULE hModule,
  LPWSTR  lpBaseName,
  DWORD   nSize
);
```

- [CreateToolhelp32Snapshot](https://docs.microsoft.com/en-us/windows/desktop/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot)

CreateToolhelp32Snapshot可以通过获取进程信息为指定的进程、进程使用的堆[HEAP]、模块[MODULE]、线程建立一个快照。说到底，可以获取系统中正在运行的进程信息，线程信息，等。

```
HANDLE CreateToolhelp32Snapshot(
  DWORD dwFlags,			// //用来指定“快照”中需要返回的对象，可以是TH32CS_SNAPPROCESS等
  DWORD th32ProcessID		// //一个进程ID号，用来指定要获取哪一个进程的快照，当获取系统进程列表或获取 当前进程快照时可以设为0
);
```

# 注入

## 注册表注入DLL

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows`

![注册表注入DLL](/images/Fri-Jun--7-17:15:45-2019_834633.png "注册表注入DLL")

AppInit_DLLs 键的值可能会包含一个DLL文件名或一组DLL的文件名（通过空格或逗号分隔）。第一个DLL的文件名可以包含路径，但其他DLL包含的路径则将被忽略（可将自己的dll放在Windows系统目录，这样就不必指明路径了）

为了能让系统使用这个注册表项，还必须创建一个名为 LoadAppInit_DLLs，类型为 DWORD 的注册表项，值设为1

当User32.dll被映射到一个新的进程时，会收到 **DLL_PROCESS_ATTACH**通知，在对它处理时，会取得上述注册表键的值，并调用 LoadLibrary 来载入这个字符串指定的每个DLL。（注意，由于被注入的DLL是在进程生命周期的早期被载入的，因此在调用函数时应该慎重，有些系统dll可能还没被加载，若直接调用可能会导致问题甚至系统蓝屏）

这种方法比较方便，但只会被映射到使用了 User32.dll 的进程中，所有基于 GUI 的应用程序都使用了 User32.dll ，但大多数基于 CUI 的应用程序不会使用它。且会被映射到每个基于GUI的应用程序中，若dll代码有问题可能导致更严重的问题

## Windows挂钩注入DLL

SetWindowsHookEx函数是微软提供给程序开发人员进行消息拦截的一个API。不过，他的功能不仅可以用作消息拦截，还可以进行DLL注入。

```
SetWindowsHookExW(
    _In_ int idHook,
    _In_ HOOKPROC lpfn,
    _In_opt_ HINSTANCE hmod,
    _In_ DWORD dwThreadId);

BOOL
WINAPI
UnhookWindowsHookEx(
    _In_ HHOOK hhk);
```

## 远程线程注入DLL

windows提供了一些函数来让一个进程对另一个进程进行操作

```
HANDLE CreateRemoteThread(
  HANDLE                 hProcess,
  LPSECURITY_ATTRIBUTES  lpThreadAttributes,		// 线程安全对象，默认可传入NULL
  SIZE_T                 dwStackSize,				// 指定线程可以为其线程栈使用多少地址空间
  LPTHREAD_START_ROUTINE lpStartAddress,			// 新线程执行的线程函数的地址
  LPVOID                 lpParameter,
  DWORD                  dwCreationFlags,			// 指定标志来控制线程的创建。0/CREATE_SUSPENDED
  LPDWORD                lpThreadId					// 存储分配给新线程的ID
);
```

CreateRemoteThread允许在目标进程中创建一个新的线程，我们可以在这个线程中运行自己的代码。注入dll可以让这个线程运行LoadLibrary来加载目标dll

大致如下：

```
int injectLib()
{
	DWORD pid = 0;				// 进程pid
	WCHAR dllName[200] = { 0 };		// 要注入的dll
	HANDLE hProcess = NULL;		// 进程句柄
	HANDLE hThread = NULL;
	LPVOID pszLibFileRemote = NULL;

	printf("please input pid: ");
	scanf_s("%d", &pid);
	printf("please input target dll name: ");
	wscanf_s(L"%s", &dllName, sizeof(dllName));

	printf("\n[+] target pid: 0x%x\n", pid);
	wprintf(L"[+] target dll: %s\n", dllName);

	__try {
		hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pid);

		if (!hProcess) {
			ERROR("openprocess error!");
		}

		printf("[+] process open success: 0x%x\n", pid);

		int cch = 1 + lstrlenW(dllName);
		int cb = cch * sizeof(WCHAR);

		pszLibFileRemote = VirtualAllocEx(hProcess, NULL, cb, MEM_COMMIT, PAGE_READWRITE);

		if (!pszLibFileRemote) {
			ERROR("virtualAlloc error!");
		}

		printf("[+] virtualAlloc: %p\n", pszLibFileRemote);

		// copy the DLL's pathname to the remote process address space
		if (!WriteProcessMemory(hProcess, pszLibFileRemote, dllName, cb, NULL)) {
			ERROR("WriteProcessMemory error: 0x%x", GetLastError());
		}
		wprintf(L"[+] WriteProcessMemory Success: %s\n", dllName);

		LPTHREAD_START_ROUTINE pfnLoadLibrary = (LPTHREAD_START_ROUTINE)GetProcAddress(
			GetModuleHandle(L"kernel32"), "LoadLibraryW");
		if (!pfnLoadLibrary) {
			ERROR("loadlibrary load error: 0x%x", GetLastError());
		}

		printf("[+] LoadLibraryW: %p\n", pfnLoadLibrary);

		hThread = CreateRemoteThread(hProcess, NULL, 0, pfnLoadLibrary, pszLibFileRemote, 0, NULL);

		if (!hThread) {
			ERROR("CreateRemoteThread error: 0x%x", GetLastError());
		}

		// Wait for the remote thread to terminate
		WaitForSingleObject(hThread, INFINITE);

		puts("[+] inject success!");
	}
	__finally {
		if (hThread) {
			CloseHandle(hThread);
		}
		if (pszLibFileRemote) {
			VirtualFreeEx(hProcess, pszLibFileRemote, 0, MEM_RELEASE);
		}
		if (hProcess) {
			CloseHandle(hProcess);
		}
	}
	return 0;
}
```

## 使用木马DLL来注入DLL

windows下dll的加载顺序大致为：

- EXE所在目录

- 当前目录GetCurrentDirectory()

- 系统目录GetSystemDirectory()

- WINDOWS目录GetWindowsDirectory()

- 环境变量 PATH 所包含的目录。

我们可以把进程必然会载入的一个dll替换掉，比如进程会载入 demo.dll，我们可以创建自己的 demo.dll ，将原有的进行重命名。或者在exe所在目录下创建一个名称为系统dll的dll，使其优先加载我们的dll。当然我们需要在我们自己的dll中导出原有dll的符号，这点可通过函数转发器很方便的实现：

```
// 下面这个pragma告诉链接器，正在编译的dll应该输出一个名为 SomeFunc 的函数，但实际实现SomeFunc的是另一个名为SomeOtherFunc的函数，这个函数被包含在一个名为DllWork的模块中
#pragma comment(linker, "/export:SomeFunc=DllWork.SomeOtherFunc")
```

## 修改应用程序的导入段来注入DLL

导入段包含了一个模块所需的所有DLL的名称，我们可以在文件的导入段中找到那个要被替换的DLL的名称，对它进行修改或在导入段中添加我们的dll

# HOOK

## VEH hook (Vectored Exception Handling）

> VEH 的中文名字： 向量化异常处理（Vectored Exception Handling）
> 1.VEH 最早出现在XP上 因为只有XP及以上Window版本才支持
> 目前 Windows 平台下实现和使用的异常处理机制主要有 4 种：
> 　　筛选器异常处理，结构化异常处理(Structure Exception Handler, SEH)，向量化异常处理(Vectored Exception Handler, VEH)，C++异常处理(C++ Exception Handler, C++EH)。
> 其中 前三种为操作系统提供的异常处理机制，最后一种为C++提供的
>
> VEH通过使用 Win32 API 函数 AddVectoredExceptionHandler可注册新的异常处理函数，函数的参数就是指向 EXCEPTION_POINTERS 结构的指针。同时，增加了函数地址的注册处理程序链表。由于系统中使用一个链表来存储矢量异常处理程序，程序可以安装尽可能多的向量处理器，只要有必要。
> 在用户模式下发生异常时，异常处理分发函数在内部会先调用遍历 VEH 记录链表的函数， 如果没有找到可以处理异常的注册函数，再开始遍历 SEH 注册链表。
> 二者之间具体联系：VEH优先权高于SHE，只有所有VEH全不处理某个异常的时候，异常处理权才会到达SHE。只要目标程序中没有利用VEH，那么，你所设计的VEH将是第一个得到控制者。现在采用SEH作为异常处理的普通C/C++程序对你将不会再有干扰，可以通过使用VEH来进行hook处理了。
> 如果存在调试器，那么控制权转向将会发生新的变化。当异常发生后，首先通知的将会是调试器，调试器不处理才会再返回控制权给VEH；如果VEH不处理，再返回给SHE；若SEH不处理，再给调试器一个机会，如果还不处理，则交由系统处理。
>

参考：[ (向量化异常处理)VEH hook](https://bbs.pediy.com/thread-190668.htm)

















