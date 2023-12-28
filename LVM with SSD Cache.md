Date: 13 Dec, 2023 1:14PM
Reference: https://blog.delouw.ch/2020/01/29/using-lvm-cache-for-storage-tiering/

## Using LVM cache for storage tiering

SSDs are small, expensive but fast. HDDs are large and cheap, but slow. Let’s combine the two technologies to get the speed of SSDs with the price and size of HDDs. This can be achieved with storage tiering using LVM cache.

## Hardware vs. Software solutions

There are so-called “Hybrid HDDs” on the market. The SSD part is relatively small and you can not tune that cache or getting any statistics about cache hits and cache misses. Further, modern SSD provides a NVMe interface which is much faster than the old school SATA ports. That makes Hybrid HDD/SDD quite useless. A software solution is much faster and more flexible.

## dm-cache vs. dm-writecache

There are two implementations available, dm-cache and dm-writecache. This article will focus on dm-cache.

### dm-cache

Provides both, write and read cache and is used where not only write operations are critical but also read operations. Use cases are very versatile, can be everything from VM storage to file servers and the like. Another benefit of dm-cache over dm-writecache is that the cache can be created, activated and destroyed online.

### dm-writecache

Dm-writecache provides only write cache, but no read cache. dm-writecache has less overhead because there will be no promotion/demotion of data for read caches, performance tests show slightly better results with dm-writecache over dm-cache. Use cases are from my point of view a bit limited as there is no read cache. If you need the maximum possible performance on write operations, then dm-writecache is your choice. Be aware that for creating the cache, the LV must be offline.

I tested this with Fedora 31, RHEL 8.1 and 8.2 beta. It will not work with RHEL8.1 but with RHEL 8.2 beta. On GA it will probably be in the status Technical Preview which means it is not fully supported.

```
dmesg|tail -2
[  343.446581] TECH PREVIEW: DM writecache may not be fully supported.
               Please review the provided documentation for limitations.
```

Enterprise users should stick to dm-cache.

## cachepool vs. cache volumes

There are two kinds of methods of how the cache is used: cachepool and cache volumes (cachevol). The main difference is that cachepool is using two logical volumes to store the actual cache and the cache metadata, where cachevol is using one device for both. If you have more than one LV to cache it may be better to use cachepool as you can place the metadata on a different device which will provide a better over-all throughput.

## Writeback vs. writethrough caching

Writeback means that the write commit happens when the data is written to the cache device only. Writethrough means that the commit happens when the data is written to the backend device as well. So writethrough is more or less useless, write operations are probably even slower (at least latency wise) that with no caching at all.

You may consider using redundant cache devices when using writeback cache mode to ensure resilience. This can be done using a software raid or a hardware raid controller.

## Performance

In my test setup I just made some very basic performance tests with _dd_ which is not really a benchmark.

Write speed to the LV was about 340 Mbyte/s, read speed approx. 410 Mbyte/s which is quite slower than the native speed to the used SATA SSD (Samsung SSD 840) but a massive gain compared to the native speed of the HDDs.

With more modern devices (NVMe PCIe) you can expect much better performance.

## Assumptions

- Your slow HDD device is /dev/vdb
- Your fast SDD device is /dev/vdc

## Lets do it

### Create the PVs

```
[root@f31-lvmtest ~]# pvcreate /dev/vdb
  Physical volume "/dev/vdb" successfully created.
[root@f31-lvmtest ~]# pvcreate /dev/vdc
  Physical volume "/dev/vdc" successfully created.
[root@f31-lvmtest ~]#
```

### Create the VG and LV

```
[root@f31-lvmtest ~]# vgcreate vg_data /dev/vdb /dev/vdc
  Volume group "vg_data" successfully created
[root@f31-lvmtest ~]# lvcreate -L 10G -n lv_data vg_data /dev/vdb
  Logical volume "lv_data" created.
[root@f31-lvmtest ~]#
```

The created logical volume “lv_data” is later also referenced as the “Origin LV” in manpages and other documentation available.

### Creating the cache pool and meta data LVs

```
[root@f31-lvmtest ~]# lvcreate -n cachepool_meta -L 100M vg_data /dev/vdc
  Logical volume "cachepool_meta" created.
[root@f31-lvmtest ~]# lvcreate -n cachepool -L 5G vg_data /dev/vdc
  Logical volume "cachepool" created.
[root@f31-lvmtest ~]#
```

Be aware that the cache pool and metadata LVs can reside on different devices to speed up things even more.

### Assemble the cache pool and attach it to the origin LV

```
[root@f31-lvmtest ~]# lvconvert --type cache-pool --poolmetadata cachepool_meta vg_data/cachepool
  Converted vg_data/cachepool and vg_data/cachepool_meta to cache pool.
[root@f31-lvmtest ~]# lvconvert --type cache --cache-pool cachepool --cachemode writethrough vg_data/lv_data
  Logical volume vg_data/lv_data is now cached.
[root@f31-lvmtest ~]#
```

Now the lv_data LV is cached. Lets have a look to the output of _lsblk_

```
[root@f31-lvmtest ~]# lsblk 
NAME                            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                              11:0    1 1024M  0 rom  
vda                             252:0    0   20G  0 disk 
├─vda1                          252:1    0    1G  0 part /boot
└─vda2                          252:2    0   19G  0 part 
  ├─fedora-root         253:0    0   15G  0 lvm  /
  └─fedora-swap         253:1    0    2G  0 lvm  [SWAP]
vdb                             252:16   0  100G  0 disk 
└─vg_data-lv_data_corig         253:5    0   10G  0 lvm  
  └─vg_data-lv_data             253:2    0   10G  0 lvm  
vdc                             252:32   0  100G  0 disk 
├─vg_data-cachepool_cpool_cdata 253:3    0    5G  0 lvm  
│ └─vg_data-lv_data             253:2    0   10G  0 lvm  
└─vg_data-cachepool_cpool_cmeta 253:4    0  100M  0 lvm  
  └─vg_data-lv_data             253:2    0   10G  0 lvm 
[root@f31-lvmtest ~]#
```

## Useful tools

The lvm command has some options to display cache statistics but they are not very nice to read.

```
f31-lvmtest:~# lvs -a -o +devices,cache_total_blocks,cache_used_blocks,cache_dirty_blocks,cache_read_hits,cache_read_misses,cache_write_hits,cache_write_misses,segtype
  LV                      VG             Attr       LSize   Pool              Origin          Data%  Meta%  Move Log Cpy%Sync Convert SyncAction Mismatches Devices                  CacheTotalBlocks CacheUsedBlocks  CacheDirtyBlocks CacheReadHits    CacheReadMisses  CacheWriteHits   CacheWriteMisses Type      
  root                    fedora -wi-ao----  15.00g                                                                                                 /dev/vda2(0)                                                                                                                                    linear    
  swap                    fedora -wi-ao----   2.00g                                                                                                 /dev/vda2(3840)                                                                                                                                 linear    
  [cachepool_cpool]       vg_data        Cwi---C---   5.00g                                   0.01   0.68            0.00                                   cachepool_cpool_cdata(0)            81920                8                0                5               51                0                0 cache-pool
  [cachepool_cpool_cdata] vg_data        Cwi-ao----   5.00g                                                                                                 /dev/vdc(25)                                                                                                                                    linear    
  [cachepool_cpool_cmeta] vg_data        ewi-ao---- 100.00m                                                                                                 /dev/vdc(0)                                                                                                                                     linear    
  lv_data                 vg_data        Cwi-a-C---  10.00g [cachepool_cpool] [lv_data_corig] 0.01   0.68            0.00                                   lv_data_corig(0)                    81920                8                0                5               51                0                0 cache     
  [lv_data_corig]         vg_data        owi-aoC---  10.00g                                                                                                 /dev/vdb(0)                                                                                                                                     linear    
  [lvol0_pmspare]         vg_data        ewi------- 100.00m                                                                                                 /dev/vdb(2560)                                                                                                                                  linear    
f31-lvmtest:~#
```

It is recommended to create an alias for this options:

```
echo "alias lvs-cache='lvs -a -o +devices,cache_total_blocks,cache_used_blocks,cache_dirty_blocks,cache_read_hits,cache_read_misses,cache_write_hits,cache_write_misses,segtype'" >> ~/.bashrc
```

For a more pretty output of this information I found this shell script: 
[https://github.com/standard-error/lvmcache-statistics/blob/master/lvmcache-statistics.sh](https://github.com/standard-error/lvmcache-statistics/blob/master/lvmcache-statistics.sh).

The LV is hardcoded, lets change that:

```
--- lvmcache-statistics.sh.orig 2020-01-29 11:41:57.477584497 +0100
+++ lvmcache-statistics.sh      2020-01-28 12:17:01.667578929 +0100
@@ -24,7 +24,8 @@
 ##################################################################
 set -o nounset
 
-LVCACHED=/dev/vg00/lvol0
+#LVCACHED=/dev/vg00/lvol0
+LVCACHED=$1
 
 RESULT=$(dmsetup status ${LVCACHED})
 if [ $? -ne 0 ]; then
```

Afterwards you can provide your LV as a parameter:

```
[root@f31-lvmtest ~]# ./lvmcache-statistics.sh /dev/vg_data/lv_data
------------------------------------
LVM Cache report of /dev/vg_data/lv_data
------------------------------------
- Cache Usage: 6.3% - Metadata Usage: 1.4%
- Read Hit Rate: 86.0% - Write Hit Rate: 80.4%
- Demotions/Promotions/Dirty: 0/60180/0
- Features in use: metadata2 writethrough no_discard_passdown 
[root@f31-lvmtest ~]#
```

## What happens when the cache disk failes?

In write-through cache mode: No data lost. In write-back mode: Partial loss of data. All data that was not yet written to Origin LV is lost. Means: In writeback mode you better have redundancy or a fast backup/recovery solution. There may also be workloads where data loss is not important at all.

When using single non-redundant cache devices, ensure proper monitoring of the SSD with smartctl or similar SMART monitoring software and replace it as soon as blocks are relocated which means that the SSD is near the end of its life cycle.

## Removing the cache

In the case of a failure of the cache device or you just want to replace it with a faster device, un-caching can be done, even online.

### Cache device is operational

If the cache device is operational its straight forward:

```
f31-lvmtest:~# lvconvert --uncache vg_data/lv_data
  Logical volume "cachepool_cpool" successfully removed
  Logical volume vg_data/lv_data is not cached.
f31-lvmtest:~#
```

It also removes the cachepool and cachepool metadata LVs from the VG.

### Cache device failure

In the case of a failure this looks as following:

```
lvconvert --uncache vg_data/lv_data
  WARNING: Couldn't find device with uuid TDYR1f-4QWp-Z8n4-FgaS-T7ni-NVai-3hO3J9.
  WARNING: VG vg_data is missing PV TDYR1f-long-guid-3hO3J9 (last written to /dev/vdc).
  WARNING: Couldn't find device with uuid TDYR1f-4QWp-Z8n4-FgaS-T7ni-NVai-3hO3J9.
  WARNING: Cache pool data logical volume vg_data/cachepool_cpool_cdata is missing.
  WARNING: Cache pool metadata logical volume vg_data/cachepool_cpool_cmeta is missing.
Do you really want to uncache vg_data/lv_data with missing LVs? [y/n]: y
  WARNING: Couldn't find device with uuid TDYR1f-4QWp-Z8n4-FgaS-T7ni-NVai-3hO3J9.
  Logical volume "cachepool_cpool" successfully removed
  Logical volume vg_data/lv_data is not cached.
```

Ensure you remove the failed device from the VG as well:

```
vgreduce vg_data --removemissing
```

Afterwards you need to activate the LV again:

```
[root@f31-lvmtest ~]# lvchange -a y /dev/vg_data/lv_data
```

## Conclusion

dm-cache brings for both, read and write operations, a huge performance gain which makes it realy worth using it. Use writeback cache for better performance and depending on the resilience requirements, use redundant cache devices (and of course redundant HDDs as well).

Happy caching!