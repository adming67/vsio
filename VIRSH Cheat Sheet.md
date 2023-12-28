`virsh` is a command-line interface for managing virtualized guests using the libvirt library. It is commonly used with virtualization platforms such as KVM (Kernel-based Virtual Machine). Here's a cheat sheet with some commonly used `virsh` commands:

### Connect to the Hypervisor
```bash
virsh connect qemu:///system   # Connect to the local system hypervisor
virsh connect qemu+ssh://user@hostname/system  # Connect to a remote system hypervisor
```

### List Domains (Virtual Machines)
```bash
virsh list               # List all active domains
virsh list --all         # List all domains, including inactive ones
```

### Domain Information
```bash
virsh dominfo <domain>   # Display information about a specific domain
virsh domstats <domain>  # Display statistics for a specific domain
```

### Start/Stop/Reboot Domains
```bash
virsh start <domain>     # Start a domain
virsh autostart <domain> # Set a domain to start automatically on system startup
                         # Use --disable option to disable autostart
virsh shutdown <domain>  # Gracefully shutdown a domain
virsh destroy <domain>   # Forcefully stop a domain (equivalent to Force Off)
virsh reboot <domain>    # Reboot a domain
```

### View Console
```bash
virsh console <domain>   # Connect to the console of a domain
                         # --force option may also be used.
```

### Managing Devices
```bash
virsh attach-device <domain> <xml-file>  # Attach a device to a domain
virsh detach-device <domain> <xml-file>  # Detach a device from a domain
```

### Snapshot Management
```bash
virsh snapshot-create-as <domain> <snapshot-name>  # Create a snapshot
virsh snapshot-list <domain>                        # List snapshots
virsh snapshot-revert <domain> <snapshot-name>       # Revert to a specific snapshot
virsh snapshot-delete <domain> <snapshot-name>       # Delete a snapshot
```

### Virtual Network Management
```bash
virsh net-list                 # List virtual networks
virsh net-info <network>       # Display information about a virtual network
virsh net-start <network>      # Start a virtual network
virsh net-destroy <network>    # Stop a virtual network
virsh net-autostart <network>  # Enable autostart for a virtual network
```

### Miscellaneous
```bash
virsh help                    # Display help for virsh commands
virsh version                 # Display version information
```

This is a basic cheat sheet, and there are many more options and features available in `virsh`. You can always refer to the manual pages (`man virsh`) or the official documentation for more detailed information and options.

## Examples

#### Virsh display node information

```shell
sudo virsh nodeinfo
```

```Output
CPU model:           x86_64
CPU(s):              8
CPU frequency:       3769 MHz
CPU socket(s):       1
Core(s) per socket:  4
Thread(s) per core:  2
NUMA cell(s):        1
Memory size:         16227192 KiB
```

#### Get a List of OSes

```bash
osinfo-query os
```
- This can be installed using following command:
```bash
sudo nala install libosinfo-bin
```

#### Get a List All Domains

```bash
virsh list --all
```

```Output
 28   winxi-1       running
 29   ubsvr201      running
 -    fedora-39     shut off
 -    mint          shut off
 -    ubuntu23.10   shut off
 -    winx          shut off
 -    winx-clone1   shut off
 -    winx-clone2   shut off
 -    winx-clone3   shut off
 -    winxi         shut off
```

#### Get a List of Active Domains

```Output
 28   winxi-1       running
 29   ubsvr201      running
```

#### Rename a Domain

```bash
virsh domrename ubsvr201 apacheserver01
```

#### Change Domain Boot Disk

```bash
virsh edit ubsvr201   # Syntax: virsh edit <domain>
                      # Scroll to the <devices> section and modify <source file=''/>
                      # Example:
                      # <source file='/var/lib/libvirt/images/==apacheserver01.qcow2=='/>
```

#### Get Domain Information
```bash
virsh dominfo ubsvr201  # Syntax: virsh dominfo <domain>
```

```Output
Id:             29
Name:           ubsvr201
UUID:           c2b85e0e-8ede-44ab-9ac3-6c20acb941eb
OS Type:        hvm
State:          running
CPU(s):         4
CPU time:       16541.4s
Max memory:     2097152 KiB
Used memory:    2097152 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: apparmor
Security DOI:   0
Security label: libvirt-c2b85e0e-8ede-44ab-9ac3-6c20acb941eb (enforcing)
```

#### Stop All Running VMs

```bash
for i in `sudo virsh list | grep running | awk '{print $2}'` do
  sudo virsh shutdown $i 
done
```

#### Remove a Domain

```bash
sudo virsh destroy test 2> /dev/null 
sudo virsh undefine test                  # --remove-all-storage option my be appended
sudo virsh pool-refresh default 
sudo virsh vol-delete --pool default test.qcow2
```

#### Suspend a Domain

```bash
virsh suspend ubsvr201
```
- Remember it will still consume system RAM. Disk and network I/O will not occur while the guest is suspended.

#### Resume a Domain

```bash
virsh resume ubsvr201
```

#### Save a Domain

```bash
virsh save ubsvr201
```

#### Restore a Domain

```bash
virsh restore ubsvr201
```

#### List Volumes in a Pool

```bash
virsh vol-list --pool default
virsh vol-list --pool images
```

