# Process-hiding
任务管理器的“结束任务”实际上就是强制终止进程，它所使用的是一个叫做TerminateProcess()的Win32 API函数，它的定义：
BOOL TerminateProcess(
HANDLE hProcess, // 将被结束进程的句柄
UINT uExitCode // 指定进程的退出码
);
Hook TerminateProcess()函数，每次TerminateProcess()被调用的时候先判断企图结束的进程是否是我的进程，如果是的话就简单地返回一个错误码就可以了。
如何根据hProcess判断它是否是我的进程的句柄：在我的进程当中先获得我的进程的句柄，然后通过进程间通讯机制传递给钩子函数，与hProcess进行比较不就行了？错！因为句柄是一个进程相关的值，不同进程中得到的我的进程的句柄的值在进程间进行比较是无意义的。
HANDLE OpenProcess(
DWORD dwDesiredAccess, // 希望获得的访问权限
BOOL bInheritHandle, // 指明是否希望所获得的句柄可以继承
DWORD dwProcessId // 要访问的进程ID
);
在调用TerminateProcess()之前，必先调用OpenProcess()，而OpenProcess()的参数表中的dwProcessId是在系统范围内唯一确定的。得出结论：要Hook的函数不是TerminateProcess()而是OpenProcess()，在每次调用OpenProcess()的时候，我们先检查dwProcessId是否为我的进程的ID(利用进程间通讯机制)，如果是的话就简单地返回一个错误码就可以了，任务管理器拿不到我的进程的句柄，故无法结束进程。
