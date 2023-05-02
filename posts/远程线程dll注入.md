---
layout: default
title:  远程线程dll注入
date:   2023-05-01 22:32:00 +0800
---

[返回主页](../)

# 简介

一个进程在另一个进程中创建线程的注入方式。

# 核心函数

创建在另一个进程的虚拟地址空间中运行的线程。

```c++
HANDLE CreateRemoteThread(
  [in]  HANDLE                 hProcess,
  [in]  LPSECURITY_ATTRIBUTES  lpThreadAttributes,
  [in]  SIZE_T                 dwStackSize,
  [in]  LPTHREAD_START_ROUTINE lpStartAddress,
  [in]  LPVOID                 lpParameter,
  [in]  DWORD                  dwCreationFlags,
  [out] LPDWORD                lpThreadId
);
```

# 代码

首先需要一个用来注入的dll。

```c++
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"

BOOL APIENTRY DllMain(HMODULE hModule,
                      DWORD ul_reason_for_call,
                      LPVOID lpReserved) {
  switch (ul_reason_for_call) {
    case DLL_PROCESS_ATTACH:
      MessageBox(NULL, L"success!", L"Congratulation", MB_OK);
    case DLL_THREAD_ATTACH:
      MessageBox(NULL, L"success!", L"Congratulation", MB_OK);
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
      break;
  }
  return TRUE;
}

```

远程线程dll注入核心代码

```c++
// clang-format off
#include <Windows.h>
#include <TlHelp32.h>
#include <iostream>
#include "tchar.h"
// clang-format on

char string_inject[] = "D:\\Workspace\\Inject\\x64\\Debug\\Inject.dll";

/**
 * 通过进程快照获取PID
 *
 * \param lpProcessName
 * \return
 */
DWORD _GetProcessPID(LPCTSTR lpProcessName) {
  DWORD Ret = 0;
  PROCESSENTRY32 p32;

  HANDLE lpSnapshot = ::CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

  if (lpSnapshot == INVALID_HANDLE_VALUE) {
    printf("获取进程快照失败, 请重试! Error[%d]", ::GetLastError());
    return Ret;
  }

  p32.dwSize = sizeof(PROCESSENTRY32);
  ::Process32First(lpSnapshot, &p32);

  do {
    if (!lstrcmp(p32.szExeFile, lpProcessName)) {
      Ret = p32.th32ProcessID;
      break;
    }
  } while (::Process32Next(lpSnapshot, &p32));

  ::CloseHandle(lpSnapshot);
  return Ret;
}

/**
 * 打开一个进程并为其创建一个线程
 *
 * \param pid
 * \param DllName
 * \return
 */
DWORD RemoteThreadInject(DWORD _Pid, LPCWSTR dllName) {
  // 打开进程
  HANDLE hProcess;
  HANDLE hThread;
  DWORD size = 0;
  BOOL write = 0;
  LPVOID pAllocMemory = nullptr;
  DWORD dllAddr = 0;
  FARPROC pThread;
  hProcess = ::OpenProcess(PROCESS_ALL_ACCESS, FALSE, _Pid);
  size = (_tcslen(dllName) + 1) * sizeof(TCHAR);

  // 远程申请空间
  pAllocMemory =
      ::VirtualAllocEx(hProcess, nullptr, size, MEM_COMMIT, PAGE_READWRITE);
  if (pAllocMemory == nullptr) {
    printf("VirtualAllocEx - Error!");
    return FALSE;
  }

  // 写入内存
  write = ::WriteProcessMemory(hProcess, pAllocMemory, dllName, size, NULL);
  if (write == FALSE) {
    printf("WriteProcessMemory - Error!");
    return FALSE;
  }

  // 获取LoadLibrary的地址
  pThread =
      ::GetProcAddress(::GetModuleHandle(L"kernel32.dll"), "LoadLibraryW");
  LPTHREAD_START_ROUTINE addr = (LPTHREAD_START_ROUTINE)pThread;

  // 在另一个进程中创建线程
  hThread =
      ::CreateRemoteThread(hProcess, NULL, 0, addr, pAllocMemory, 0, NULL);
  if (hThread == NULL) {
    printf("CreateRemoteThread - Error!");
    return FALSE;
  }

  // 等待线程函数结束，获得退出码
  WaitForSingleObject(hThread, -1);
  GetExitCodeThread(hThread, &dllAddr);

  // 释放dll空间
  VirtualFreeEx(hProcess, pAllocMemory, size, MEM_DECOMMIT);

  // 关闭线程句柄
  ::CloseHandle(hProcess);
  return TRUE;
}

int main() {
  DWORD PID = _GetProcessPID(L"test.exe");
  RemoteThreadInject(PID, L"D:\\Workspace\\Inject\\Debug\\Inject.dll");
}
```

写一个`test.exe`作为测试进程

```c++
#include <iostream>

int main() {
  std::cout << "Hello World!\n";
  getchar();
}
```