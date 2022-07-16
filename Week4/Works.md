# Works 4


## 测试工作

part2

正常：

```
name: oe_test_password_blank
name: oe_test_ProcMgmt_vmstat
name: oe_test_ProcMgmt_top
name: oe_test_ProcMgmt_start_kill
name: oe_test_ProcMgmt_ps
name: oe_test_ProcMgmt_pgrep
```

失败：
```
初步判断需多个节点
oe_test_reboot

疑似本体问题
oe_test_ProcMgmt_sar
oe_test_password_change

疑似路径问题
oe_test_ProcMgmt_at
oe_test_disk_tuned_set
oe_test_disk_schedule_specific
oe_test_disk_schedule_udev
oe_test_disk_tuned_new
oe_test_disk_tuned_install

未知

oe_test_ProcMgmt_crontab_cronfile
oe_test_ProcMgmt_crontab_cmd
ps：以上两个前段执行正常均死在 grep: /root/test.txt: No such file or directory

oe_test_ProcMgmt_iostat
oe_test_nmcli_vlan
oe_test_nmcli_systemd_resolved
oe_test_nmcli_route
oe_test_nmcli_macsec
oe_test_nmcli_general
oe_test_nmcli_device
oe_test_nmcli_config_gw
oe_test_nmcli_config_connect
oe_test_nmcli_con_reload
oe_test_nmcli_cancel_auto
oe_test_nmcli_bridge
oe_test_nmcli_add_connect
oe_test_nmcli_Mgntconnect
oe_test_nmcli_8023link
oe_test_USBinfo
oe_test_server_git_install
oe_test_power_powertop_startup
oe_test_power_powertop_powerup
oe_test_power_powertop_calibrate
oe_test_power_powertop2tuned_optimize
oe_test_power_output_HTML
oe_test_power_measurement_internal
oe_test_performance_monitor_pcp
```


## 课程工作

- [openEuler RISC-V 应用编程技术 Chapter2 Lab1](https://github.com/KiritakeKumi/OpenEuler-Application-For-RISC-V-Programming/blob/main/Files/Chapter2/Lab1/Lab1.md)
- [openEuler RISC-V 应用编程技术 Chapter1 Lab2](https://github.com/KiritakeKumi/OpenEuler-Application-For-RISC-V-Programming/blob/main/Files/Chapter2/Lab2/Lab2.md)