+++
title = "Proper Factory Reset"
date = "2026-11-12"


[taxonomies]
tags=["android"]

[extra]
comment = true
+++

In this blog, I’ll walk you through how to properly wipe your Android phone before selling it or handing it off. So your data doesn’t go along for the ride with the phone too :p 

# Prerequisites

    A linux system with adb installed 
    USB cable (which supports data transfer)
    Backup of your data (optional)
    Have your phone charged above 50% atleast

Remove accounts & disable lock screen (preventing FRP)

Remove the google accounts and the OEM accounts. Disable the lock screen and set it to none.

# Check encryption state
```bash
adb shell getprop ro.crypto.state
```
> if the output says `encrypted` = configured and ready

> if the output says `unencrypted` = try to enable it if available in settings

# Overwrite, then wipe

We have to overwrite free space on the user partition by copying large 
files into `/sdcard`, then delete them.

> random file set on linux
```bash
dd if=/dev/urandom of=~/bigfile.bin bs=1M count=1024 status=progress
```

> now run this script until you fill the android
```bash
#!/bin/bash
FILE=~/bigfile.bin
i=1
echo "Pushing random files until device reports full..."
while true; do
  echo "Pushing copy #$i..."
  if adb push "$FILE" /sdcard/bigfile_${i}.bin >/dev/null 2>&1; then
    i=$((i+1))
  else
    echo "Push failed — likely full. Stopping."
    break
  fi
done
```

# Check free space
```bash
adb shell df -h /sdcard || true
```

# Purge 
remove the files we just pushed
```bash
adb shell rm /sdcard/bigfile_*.bin
```
> now reboot
```bash
adb reboot
```
now just perform a normal factory reset how you normally would and you're all done.

# Automation
I was bored so i made a script to automate this process, you can find the link [here](https://github.com/0xnafq/Wipe-script)

> You have to enable `USB-Debugging` in developer settings, then check if it's recognized in adb.
```bash
adb devices
```
output should be like 
```bash
> adb devices
List of devices attached
df5bXXXX        device
```
and when the device is recognized, just run the script with `sudo`(important)
```bash
sudo ./wipe.sh 
```
and then the magic would happen 
```bash
> sudo ./wipe.sh
Checking device...
df5bXXXX        device

941621248 bytes (942 MB, 898 MiB) copied, 2 s, 470 MB/s
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 2.28577 s, 470 MB/s
Pushing files until storage full...
pushed #1
pushed #2
pushed #3
pushed #4
pushed #5
pushed #6
pushed #7
pushed #8
pushed #9
pushed #10
pushed #11
pushed #12
pushed #13
pushed #14
```
this might go on for a while and it would end like this.
```bash
pushed #87
pushed #88
pushed #89
pushed #90
pushed #91
pushed #92
pushed #93
pushed #94
pushed #95
pushed #96
pushed #97
Stopped at copy #98
Deleting pushed files...
Empty Files app Trash manually if needed.
Now run factory reset via phone UI or recovery.

>
```
so it pushes the 1gb file to the device till it fills up and then you can do the factory reset via ui as you normally do and you can repeat it if you want to.

# Conclusion
A proper factory reset doesn't just delete data, it also ensures the data is overwritten first so the old data can't be recovered. Filling the device with random data, deleting it and then performing a standard reset, you significantly reduce the risk of data leakage. It just adds another layer of security before handing off your phone or selling it.


