main function disassembly : 
```asm 
│           0x00001128      55             push rbp
│           0x00001129      4889e5         mov rbp, rsp
│           0x0000112c      4883ec10       sub rsp, 0x10
│           0x00001130      bf02000000     mov edi, 2                  ; int64_t arg1
│           0x00001135      e8dfffffff     call sym.add
│           0x0000113a      8945fc         mov dword [var_4h], eax
│           0x0000113d      8b45fc         mov eax, dword [var_4h]
│           0x00001140      c9             leave
└           0x00001141      c3             ret
```


preparing the stack frame for the new entered function (main) : 
```asm
   0x555555555128 <main+0000>      push   rbp   --> callee base pointer
   0x555555555129 <main+0001>      mov    rbp, rsp --> rbp = rsp (getting a new stack frame addr) 
   0x55555555512c <main+0004>      sub    rsp, 0x10 --> reserving/allocating 16 bytes for the function . 
```
the compiler allocates needed memory for all the variables inside the function (including args if they are passed
from the registers. the reason is if we store local variables in rbp-4 and rbp-8 .... we would overrite them if if push the rip of the next instruction + the rbp of the current function (the call instruction saves a rip and jumps to the function address ) )



preparing args for before calling the add function : 
```asm
    0x555555555130 <main+0008>      mov    edi, 0x2 --> passing the 2 as the first argument for add function
    0x555555555135 <main+000d>      call   0x555555555119 <add> --> calling the function 
```
    calling a function = storing the next IP and then jumping to the function location .

argument passing convention  (may change program to program. not something universal):

1st arg --> edi
2st arg --> esi
3st arg --> edx
4st arg --> ecx
5st arg --> r8d
6st arg --> r9d
7st arg and the rest --> pushed into the stack (ill get to that later.)

```asm 
   0x55555555513a <main+0012>      mov    DWORD PTR [rbp-0x4],  eax --> the res of function add is 
                                                                stored at eax. we are saving it into 
                                                                rbp-4 (its the local variable "a" in
                                                                source code) 
   0x55555555513d <main+0015>      mov    eax, DWORD PTR [rbp-0x4] --> preparing the return value of the
                                                                        main function . 
   0x555555555141 <main+0019>      ret                  ---> returning to the callee function . 

```
sym.add function disassembly : 

```asm 
│           0x00001119      55             push rbp
│           0x0000111a      4889e5         mov rbp, rsp
│           0x0000111d      897dfc         mov dword [var_4h], edi     ; arg1
│           0x00001120      8b45fc         mov eax, dword [var_4h]
│           0x00001123      83c001         add eax, 1
│           0x00001126      5d             pop rbp
└           0x00001127      c3             ret
```



preparing the stack frame for the function call:
```asm
   0x555555555119 <add+0000>       push   rbp
 → 0x55555555511a <add+0001>       mov    rbp, rsp
```

puting the return value of the function into eax and then restoring the callee rbp (main's rbp) and returning to the callee 
```asm 
│           0x00001123      83c001         add eax, 1
│           0x00001126      5d             pop rbp
└           0x00001127      c3             ret
```
ret instruction pops the stored rip of the program which was pushed into the stack via call instruction . 
it pops it into the rip to change the flow of program back to the callee function 

note : u cant directly pop something in rip . u can pop it in a register and jmp to it . so ret is actually equivalent to 2 insuctions . 
