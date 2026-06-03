# OVS-Switch-Config-Bug
We explain how a simple wrong config can crash the switch

This bug appears when a user writes an arbitrarily large integer value to n-revalidator-threads in the Open_vSwitch other_config map via OVSDB transaction, then creates a bridge to trigger datapath initialization.

```
ovs-vsctl set Open_vSwitch . other_config:n-revalidator-threads=1000
ovs-vsctl add-br br-attack-test &
```  
