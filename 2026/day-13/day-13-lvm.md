Task 1: Check Current Storage

Run: lsblk, pvs, vgs, lvs, df -h

         command: lsblk
         NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
         loop0          7:0    0 27.8M  1 loop /snap/amazon-ssm-agent/12322
         loop1          7:1    0   74M  1 loop /snap/core22/2339
         loop2          7:2    0   74M  1 loop /snap/core22/2411
         loop3          7:3    0 48.1M  1 loop /snap/snapd/25935
         loop4          7:4    0 48.4M  1 loop /snap/snapd/26382
         loop5          7:5    0 28.2M  1 loop /snap/amazon-ssm-agent/13009
         nvme0n1      259:0    0    8G  0 disk 
         ├─nvme0n1p1  259:1    0    7G  0 part /
         ├─nvme0n1p14 259:2    0    4M  0 part 
         ├─nvme0n1p15 259:3    0  106M  0 part /boot/efi
         └─nvme0n1p16 259:4    0  913M  0 part /boot
         nvme1n1      259:5    0   10G  0 disk 

         command: pvs
         
         command: vgs
         
         command: lvs

         command: df -h
         Filesystem       Size  Used Avail Use% Mounted on
         /dev/root        6.8G  2.5G  4.3G  37% /
         tmpfs            456M     0  456M   0% /dev/shm
         tmpfs            183M  888K  182M   1% /run
         tmpfs            5.0M     0  5.0M   0% /run/lock
         efivarfs         128K  3.8K  120K   4% /sys/firmware/efi/efivars
         /dev/nvme0n1p16  881M  162M  657M  20% /boot
         /dev/nvme0n1p15  105M  6.2M   99M   6% /boot/efi
         tmpfs             92M   12K   92M   1% /run/user/1000

Task 2: Create Physical Volume

         command: pvcreate /dev/nvme1n1
                  pvs
                  
                   PV           VG Fmt  Attr PSize  PFree 
           /dev/nvme1n1    lvm2 ---  10.00g 10.00g

Task 3: Create Volume Group

         command: vgcreate devops-vg /dev/nvme1n1
                  vgs
                  
                   VG        #PV #LV #SN Attr   VSize   VFree  
           devops-vg   1   0   0 wz--n- <10.00g <10.00g

Task 4: Create Logical Volume

         command: lvcreate -L 500M -n app-data devops-vg
                  lvs
                  
                    LV       VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
           app-data devops-vg -wi-a----- 500.00m 

Task 5: Format and Mount

         command: mkfs.ext4 /dev/devops-vg/app-data
             mke2fs 1.47.0 (5-Feb-2023)
             Creating filesystem with 128000 4k blocks and 128000 inodes
             Filesystem UUID: 984b69ee-e9e7-4658-9b82-ddf73c9d067b
             Superblock backups stored on blocks: 
                     32768, 98304
    
    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (4096 blocks): done
    Writing superblocks and filesystem accounting information: done

         command: mkdir -p /mnt/app-data
         
         command: mount /dev/devops-vg/app-data /mnt/app-data
         
         command: df -h /mnt/app-data
                 
                  Filesystem                        Size  Used Avail Use% Mounted on
         /dev/mapper/devops--vg-app--data  452M   24K  417M   1% /mnt/app-data

Task 6: Extend the Volume

         command: lvextend -L +200M /dev/devops-vg/app-data
                  Size of logical volume devops-vg/app-data changed from 500.00 MiB (125 extents) to 700.00 MiB (175 extents).
                  Logical volume devops-vg/app-data successfully resized.
         
         command: resize2fs /dev/devops-vg/app-data
                  resize2fs 1.47.0 (5-Feb-2023)
                  Filesystem at /dev/devops-vg/app-data is mounted on /mnt/app-data; on-line resizing required
                  old_desc_blocks = 1, new_desc_blocks = 1
                  The filesystem on /dev/devops-vg/app-data is now 179200 (4k) blocks long.

         command: df -h /mnt/app-data
                  Filesystem                        Size  Used Avail Use% Mounted on
                  /dev/mapper/devops--vg-app--data  637M   24K  594M   1% /mnt/app-data

         
