

Get a list of hosts that dont have a specific kernel

```shell
hammer --csv --no-headers host list \
--search 'os_major=9 AND (os_title ~ "RHEL" OR os_title ~ "Red") AND lifecycle_environments=Testing AND NOT facts.uname::release=5.14.0-687.34.1.el9_8.x86_64' \
--fields Name \
--organization ops_linux
```