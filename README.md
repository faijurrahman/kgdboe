# kgdboe
A network interface for GDB for Linux Kernel

This module is a fork of https://github.com/sysprogs/kgdboe.git
For original tutorial please check here: http://sysprogs.com/VisualKernel/kgdboe/tutorial


Ensure you have the debugging symbols for your kernel on the debug machine. If you are using VisualKernel, it will fetch them automatically.
Download, unpack and build kgdboe on the target machine:
  git clone https://github.com/sysprogs/kgdboe.git
  cd kgdboe
  make -C /lib/modules/$(uname -r)/build M=$(pwd)



Check the Address of module_kallsyms_lookup:
  root@salvator-x:~/nfs/kgdboe-master# cat /proc/kallsyms | grep kallsyms_lookup_name
    ffff00000816e998 T module_kallsyms_lookup_name
    ffff00000816f340 T kallsyms_lookup_name
    ffff0000091237d0 r __ksymtab_kallsyms_lookup_name
    ffff000009134c1a r __kstrtab_kallsyms_lookup_name


Check the Name of Ethernet network device:
  root@salvator-x:~/nfs/kgdboe-master# ifconfig
    eth0      Link encap:Ethernet  HWaddr 2e:09:0a:00:a3:ff
              inet addr:192.168.0.122  Bcast:192.168.0.255  Mask:255.255.255.0
              inet6 addr: fe80::2c09:aff:fe00:a3ff/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:69437 errors:0 dropped:0 overruns:0 frame:0
              TX packets:28819 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:84707197 (80.7 MiB)  TX bytes:4963246 (4.7 MiB)
              Interrupt:115

    lo        Link encap:Local Loopback
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:65536  Metric:1
              RX packets:2 errors:0 dropped:0 overruns:0 frame:0
              TX packets:2 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:140 (140.0 B)  TX bytes:140 (140.0 B)


Load the kgdboe module:
  root@salvator-x:~/nfs/kgdboe-master# sudo insmod kgdboe.ko kallsyms_lookup_name_address=0xffff00000816f340 device_name=eth0

    Inspect the output of the dmesg command ensuring that kgdboe has initialized correctly:
    dmesg | tail



Start gdb on your debug machine. Run the following command to connect to kgdboe:
  target remote udp:<IP>:<port>
  Enjoy debugging!



Configuration parameters
  You can tweak the behavior of KGDBoE by specifying some parameters when loading the module. The syntax is simply as follows:

  insmod kgdboe.ko param1=val1 param2=val2 ...
  The following parameters are supported:
    Parameter	Default	Description
      device_name	        eth0	    Ethernet device to use for debugging.
      local_ip	          N/A	      Local IP address to bind to. Auto-detected if not specified.
      udp_port	          31337	    UDP port to use for debugging.
      force_single_core	  1       	Disable all cores except #0 when the module is loaded. This setting is recommended unless you are debugging SMP-specific issues, 
                                    as it avoids many synchronization problems. KGDBoE can reliably work in the SMP mode, but it has not been tested on all network drivers, so use caution 
                                    if you decide to disable this.
