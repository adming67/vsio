- Create initial memory tmpfs mount point.
```bash
mkdir ramtest

sudo mount tmpfs -t tmpfs ramtest

cd ramtest
```

- Test write speed
```bash
dd if=/dev/zero of=data_tmp bs=1M count=512  # write to ramtest
```

```Output
512+0 records in
512+0 records out
536870912 bytes (537 MB, 512 MiB) copied, 0.183532 s, 2.9 GB/s
```

- Test read speed
```bash
dd if=data_tmp of=/dev/null bs=1M count=512
```

```Output
512+0 records in
512+0 records out
536870912 bytes (537 MB, 512 MiB) copied, 0.11144 s, 4.8 GB/s
```

- Unmount the tmpfs mount point.
```bash
cd ~

sudo umount /home/adming/ramtest
```
