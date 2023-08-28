---
title: "How To Expand Ubuntu 20.04 LTS Filesystem Volume on Hyper-V"
datePublished: Mon Aug 28 2023 11:26:43 GMT+0000 (Coordinated Universal Time)
cuid: cllusnvg4000k08lb63c7hi2m
slug: how-to-expand-ubuntu-2004-lts-filesystem-volume-on-hyper-v

---

I tried existing solutions on 18.04 and found it doesn't work with a 'nothing to do' output. Some further searching had similar-but-slightly-different steps and needing to merge a few different bits-and-pieces from other solutions on the web to make work.

Setup: Hyper-V; VHDX file as hard disk; Ubuntu-18.04

Steps:

1. Expand the VDHX file via Hyper-V as mentioned in existing solutions and then inside the VM:
    
2. `fdisk -l`
    
    See which partition is the current Ubuntu setup - should be obvious based on size (in my case was sda3)
    
3. `growpart /dev/sda 3`
    
    Note the space as mentioned.
    
4. `pvresize /dev/sda3`
    
    This is the step which isn't mentioned in a lot of places; its the intemediate step that allows the logical volume extension step work.
    
5. `lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv`
    
    The `/dev/` part can seen in the fdisk output from step 1.
    
6. `resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv`
    
    After the prep above, this step now works. Takes a couple of moments and afterwards can verify with `df -h` that the partition is expanded.
    

src = [https://superuser.com/questions/1716141/how-to-expand-ubuntu-20-04-lts-filesystem-volume-on-hyper-v](https://superuser.com/questions/1716141/how-to-expand-ubuntu-20-04-lts-filesystem-volume-on-hyper-v)