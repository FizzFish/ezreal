---
layout: default
---

## 灵感
通过分析Linux内核master分支代码：
```sh
git --grep "use after free" or "use-after-free"
```
发现49%的UAF漏洞与设备驱动相关，65%的漏洞是通过动态分析得到的，而动态分析是有局限性的。

## 思路
文章可以发现lock/unlock对范围过小的问题。对于两个可以并行的代码段，由于锁的临界区范围问题，可能会导致UAF问题，即一段代码解引用另一段代码释放的指针地址。

## 算法
1. Local-Global Strategy

1.1 get the pairs of possible concurrently executed functions
```c
pos_func_pair_set = 0
lock_call_set = GetLockCall()
for i:= 0 to SizeOf(lock_call_set) - 1 do
    lock_call1 := lock_call_set[i]
    lock_var1 := GetLockVar(lock_call1)
    caller_func1 := GetCallerFunc(lock_call1)
    for j := i + 1 to Sizeof(lock_call_set) - 1 do
        lock_call2 := lock_call_set[j]
        lock_var2 := GetLockVar(lock_call2)
        caller_func2 := GetCallerFunc(lock_call2)
        if lock_var1 is aliased to lock_var2 then
            func_pair := <caller_func1, caller_func2>
            Add func_pair to pos_func_pair_set
        endif
    endfor
endfor
return pos_func_pair_set
```
1.2 Filer out may-false pairs of concurrently executed functions
```c
foreach func_pair in pos_func_pair_set do
    <caller_func1, caller_func2> := GetFuncPair(func_pair)
    func_set1 := GetAncestorFunc(caller_func1)
    func_set2 := GetAncestorFunc(caller_func2)
    if func_set1 && func_set2 != 0 then
        Delete func_pair from pos_func_pair_set
    endif
end foreach
return pos_func_pair_set
```
1.3 Get local concurrent interface pairs
```c
local_interface_pair_set := 0
foreach func_pair in pos_func_pair_set do
    <caller_func1, caller_func2> := GetFuncPair(func_pair)
    interface_set1 := GetDriverInterface(caller_func1)
    interface_set2 := GetDriverInterface(caller_func2)
    foreach interface1 in interface_set1 do
        foreach interface2 in interface_set2 do
            if interface1 == interface2 then
                continue
            endif
            interface_pair := <interface1, interface2>
            Add interface_pair to local_interface_pair_set
        end foreach
    end foreach
end foreach
return local_interface_pair_set
```
1.4 Get glocal concurrent interface pairs
```c
global_interface_pair_set := 0
local_interface_pair_info_set := GatherLocalInterfacePairSet()
foreach interface_pair in local_interface_pair_set_info do
    conc_num := GetFileNumOfConcInterfacePair(interface_pair)
    file_num := GetFileNumOfInterfacePair(interface_pair)
    ratio := conc_num / file_num
    if ratio >= R then
        Add interface_pair to global_interface_pair_set
    endif
end foreach
return glocal_interface_pair_set
```
2. HandleFunc(func, caller_sum, lockset_caller, path_info_caller)
```c
if func exists in path_info_caller then
    return
endif
func_sum := FindFuncSummary(func)
if func_sum == 0 then
    func_sum := AnalyzeFuncSummary(func)
    StoreFuncSymmary(func_sum)
endif
access_info_set := GetAccessInfoSet(func_sum)
foreach access_info access_info_set do
    lockset := lockset_caller + GetLockSet(access_info)
    path_info := path_info_caller + GetPathInfo(access_info)
    SetLockSet(access_info, lockset)
    SetPathInfo(access_info, path_info)
    AddAccessInfo(access_info, caller_sum)
end foreach
```
    
