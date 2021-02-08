# 设置coredump生成目录

ulimit -c unlimited && echo /nfsroot/core_%e_sig%s_pid%p > /proc/sys/kernel/core_pattern