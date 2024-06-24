```
vinnar@my-Virt:~$ cd the_everglades/
vinnar@my-Virt:~/the_everglades$ ll
total 2732
drwxrwxr-x 10 vinnar vinnar    4096 Jun 15 21:57 ./
drwxr-xr-x 31 vinnar vinnar    4096 Jun 15 18:35 ../
-rw-rw-r--  1 vinnar vinnar 2728935 Jan  4  2022 alpine-mini-rootfs.tar.gz
drwxr-xr-x 19 vinnar vinnar    4096 Jan 28  2021 alpine_root_dir/
-rwxrwxrwx  1 vinnar vinnar      98 Dec  5  2021 check_in_guests.sh*
drwxrwxr-x  6 vinnar vinnar    4096 Apr 11  2022 fitness_fans/
drwxrwxr-x  3 vinnar vinnar    4096 Feb  6  2022 Fitness_Fans_Ameneties/
drwxr-xr-x 21 root   root      4096 Jan 28  2021 Fitness_Fans_Area/
-rw-r--r--  1 root   root       308 Feb  5  2022 FitnessFans.txt
-rwxrwxr-x  1 vinnar vinnar    1170 Feb  5  2022 make_fitness_fans_area.sh*
-rwxrwxrwx  1 vinnar vinnar    1349 Apr  3  2022 make_potter_fans_area.sh*
drwxrwxr-x  8 vinnar vinnar    4096 Jan 28  2022 overlay_fs_example/
drwxrwxr-x  6 vinnar vinnar    4096 Apr 11  2022 potter_fans/
drwxrwxr-x  3 vinnar vinnar    4096 Feb  6  2022 Potter_Fans_Ameneties/
drwxrwxr-x  3 vinnar vinnar    4096 Jun 15 20:53 Potter_Fans_Area/
-rw-rw-r--  1 vinnar vinnar     331 Apr  2  2022 PotterFans.txt
-rwxr-xr-x  1 vinnar vinnar     564 Feb  7  2022 sam_potter_fan.sh*
vinnar@my-Virt:~/the_everglades$ cd overlay_fs_example/
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./*
./lower_dir:
total 20
drwxrwxr-x 2 vinnar vinnar 4096 Jan 27  2022 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   18 Jan 27  2022 ll1.txt
-rw-rw-r-- 1 vinnar vinnar   20 Jan 27  2022 ll2.txt

./lower-dir-2:
total 12
drwxrwxr-x 2 vinnar vinnar 4096 Jan 28  2022 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   22 Jan 28  2022 ll21.txt

./lower-dir-3:
total 12
drwxrwxr-x 2 vinnar vinnar 4096 Jan 28  2022 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   22 Jan 28  2022 ll31.txt

./merged_dir:
total 8
drwxrwxr-x 2 vinnar vinnar 4096 Jan 27  2022 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
```

#I failed to notice that the instructor had left files behind in the upper directory that shouldn't be there (`ll1.txt`, `ll2.txt`):

```
./upper_dir:
total 24
drwxrwxr-x 2 vinnar vinnar 4096 Jan 27  2022 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   46 Jan 27  2022 ll1.txt
c--------- 1 root   root   0, 0 Jan 27  2022 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   16 Jan 27  2022 ul1.txt
-rw-rw-r-- 1 vinnar vinnar   20 Jan 27  2022 ul2.txt

./work_dir:
total 12
drwxrwxr-x 3 vinnar vinnar 4096 Jan 28  2022 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
d--------- 2 root   root   4096 Jan 28  2022 work/
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ sudo mount -t overlay -o lowerdir=./lower_dir/,upperdir=./upper_dir/,workdir=./work_dir/ none ./merged_dir/
```

#Here I edited a .txt file to check if the lower layer was truly RO.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vim ./lower_dir/ll1.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ cat !$
cat ./lower_dir/ll1.txt
File in lower dir

is this really RO?
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ cat ./merged_dir/ll1.txt 
File in lower dir

is this really RO?
```

#So the file was writable, and the changes propagated to the copy in the merged layer.


#Curious, I edited files in the merged layer to see if any changes propagated.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vim ./merged_dir/ll2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vim ./merged_dir/ul2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./upper_dir/
total 24
drwxrwxr-x 2 vinnar vinnar 4096 Jun 19 21:18 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   58 Jun 19 21:18 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   16 Jan 27  2022 ul1.txt
-rw-rw-r-- 1 vinnar vinnar   58 Jun 19 21:18 ul2.txt
```

#Wait, what?  `ll1.txt` is now missing from the upper layer directory, and `ll2.txt` is no longer `c---------` or from 2022...

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ egrep "" ./*/ll2.txt
./lower_dir/ll2.txt:File-2 in lower dir
./merged_dir/ll2.txt:File-2 in lower dir
./merged_dir/ll2.txt:
./merged_dir/ll2.txt:this is modified from the merged dir
./upper_dir/ll2.txt:File-2 in lower dir
./upper_dir/ll2.txt:
./upper_dir/ll2.txt:this is modified from the merged dir
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ egrep "" ./*/ul2.txt
./merged_dir/ul2.txt:File-2 in upper dir
./merged_dir/ul2.txt:
./merged_dir/ul2.txt:this is modified from the merged dir
./upper_dir/ul2.txt:File-2 in upper dir
./upper_dir/ul2.txt:
./upper_dir/ul2.txt:this is modified from the merged dir
```

#Changes in the merged layer propagate to the upper layer, but not the lower layer.  So that explains the change to `ll2.txt` in the upper layer, but I still have no idea why `ll1.txt` disappeared from the upper.


#Then I tried editing a file in the upper layer to see what would happen.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ egrep "" ./*/ul1.txt
./merged_dir/ul1.txt:File-2 in Upper
./upper_dir/ul1.txt:File-2 in Upper
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./upper_dir/ul1.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ egrep "" ./*/ul1.txt
./merged_dir/ul1.txt:File-2 in Upper
./upper_dir/ul1.txt:File-2 in Upper
./upper_dir/ul1.txt:
./upper_dir/ul1.txt:modifying the copy in the upper dir
```

#Looks like edits in the upper layer don't propagate.


#I tried editing in the merged directory again.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vim ./merged_dir/ll2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ egrep "" ./*/ll2.txt
./lower_dir/ll2.txt:File-2 in lower dir
./merged_dir/ll2.txt:File-2 in lower dir
./merged_dir/ll2.txt:
./merged_dir/ll2.txt:this is modified from the merged dir, TWICE!
./upper_dir/ll2.txt:File-2 in lower dir
./upper_dir/ll2.txt:
./upper_dir/ll2.txt:this is modified from the merged dir, TWICE!
```

#Subsequent edits in the merged layer still propagate to upper.


#I tested if creating a new file in the lower layer would propagate.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./lower_dir/ll_created_after_overlay.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./lower_dir/
total 24
drwxrwxr-x 2 vinnar vinnar 4096 Jun 19 21:35 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   43 Jun 19 21:16 ll1.txt
-rw-rw-r-- 1 vinnar vinnar   20 Jan 27  2022 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   30 Jun 19 21:35 ll_created_after_overlay.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./upper_dir/
total 24
drwxrwxr-x 2 vinnar vinnar 4096 Jun 19 21:30 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   66 Jun 19 21:30 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   53 Jun 19 21:26 ul1.txt
-rw-rw-r-- 1 vinnar vinnar   58 Jun 19 21:18 ul2.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./merged_dir/
total 32
drwxrwxr-x 1 vinnar vinnar 4096 Jun 19 21:30 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   43 Jun 19 21:16 ll1.txt
-rw-rw-r-- 1 vinnar vinnar   66 Jun 19 21:30 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   30 Jun 19 21:35 ll_created_after_overlay.txt
-rw-rw-r-- 0 vinnar vinnar   16 Jan 27  2022 ul1.txt
-rw-rw-r-- 1 vinnar vinnar   58 Jun 19 21:18 ul2.txt
```

#Like editing a file, creating a new one in the lower layer propagates to the merged layer.


#I tried creating a file in the upper layer.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./upper_dir/ul_created_after_overlay.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./upper_dir/
total 28
drwxrwxr-x 2 vinnar vinnar 4096 Jun 19 21:38 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   66 Jun 19 21:30 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   53 Jun 19 21:26 ul1.txt
-rw-rw-r-- 1 vinnar vinnar   58 Jun 19 21:18 ul2.txt
-rw-rw-r-- 1 vinnar vinnar   51 Jun 19 21:38 ul_created_after_overlay.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./lower_dir/
total 24
drwxrwxr-x 2 vinnar vinnar 4096 Jun 19 21:35 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   43 Jun 19 21:16 ll1.txt
-rw-rw-r-- 1 vinnar vinnar   20 Jan 27  2022 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   30 Jun 19 21:35 ll_created_after_overlay.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./merged_dir/
total 36
drwxrwxr-x 1 vinnar vinnar 4096 Jun 19 21:38 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   43 Jun 19 21:16 ll1.txt
-rw-rw-r-- 1 vinnar vinnar   66 Jun 19 21:30 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   30 Jun 19 21:35 ll_created_after_overlay.txt
-rw-rw-r-- 0 vinnar vinnar   16 Jan 27  2022 ul1.txt
-rw-rw-r-- 1 vinnar vinnar   58 Jun 19 21:18 ul2.txt
-rw-rw-r-- 1 vinnar vinnar   51 Jun 19 21:38 ul_created_after_overlay.txt
```

#So *creating a file* in the upper layer will propagate it to the merged layer, but *editing an existing file* does not.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./upper_dir/ul*
./upper_dir/ul1.txt:File-2 in Upper
./upper_dir/ul1.txt:
./upper_dir/ul1.txt:modifying the copy in the upper dir
./upper_dir/ul2.txt:File-2 in upper dir
./upper_dir/ul2.txt:
./upper_dir/ul2.txt:this is modified from the merged dir
./upper_dir/ul_created_after_overlay.txt:initial text for UL file created after the overlay
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./merged_dir/ul*
./merged_dir/ul1.txt:File-2 in Upper
./merged_dir/ul2.txt:File-2 in upper dir
./merged_dir/ul2.txt:
./merged_dir/ul2.txt:this is modified from the merged dir
./merged_dir/ul_created_after_overlay.txt:initial text for UL file created after the overlay
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./merged_dir/ll1.txt 
-rw-rw-r-- 1 vinnar vinnar 43 Jun 19 21:16 ./merged_dir/ll1.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/ll1.txt
./lower_dir/ll1.txt:File in lower dir
./lower_dir/ll1.txt:
./lower_dir/ll1.txt:is this really RO?
./merged_dir/ll1.txt:File in lower dir
./merged_dir/ll1.txt:
./merged_dir/ll1.txt:is this really RO?
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./upper_dir/
total 28
drwxrwxr-x 2 vinnar vinnar 4096 Jun 19 23:11 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   66 Jun 19 21:30 ll2.txt
-rw-rw-r-- 1 vinnar vinnar   53 Jun 19 21:26 ul1.txt
-rw-rw-r-- 1 vinnar vinnar   58 Jun 19 21:18 ul2.txt
-rw-rw-r-- 1 vinnar vinnar   51 Jun 19 21:38 ul_created_after_overlay.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./upper_dir/ll1.txt
ls: cannot access './upper_dir/ll1.txt': No such file or directory
```

#`ll1.txt` is definitely gone from upper.


#Was going to test this again but with two overlay FSs.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ sudo umount -l ./merged_dir 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ cd ..
vinnar@my-Virt:~/the_everglades$ ll
total 2732
drwxrwxr-x 10 vinnar vinnar    4096 Jun 15 21:57 ./
drwxr-xr-x 31 vinnar vinnar    4096 Jun 20 14:28 ../
-rw-rw-r--  1 vinnar vinnar 2728935 Jan  4  2022 alpine-mini-rootfs.tar.gz
drwxr-xr-x 19 vinnar vinnar    4096 Jan 28  2021 alpine_root_dir/
-rwxrwxrwx  1 vinnar vinnar      98 Dec  5  2021 check_in_guests.sh*
drwxrwxr-x  6 vinnar vinnar    4096 Apr 11  2022 fitness_fans/
drwxrwxr-x  3 vinnar vinnar    4096 Feb  6  2022 Fitness_Fans_Ameneties/
drwxr-xr-x 21 root   root      4096 Jan 28  2021 Fitness_Fans_Area/
-rw-r--r--  1 root   root       308 Feb  5  2022 FitnessFans.txt
-rwxrwxr-x  1 vinnar vinnar    1170 Feb  5  2022 make_fitness_fans_area.sh*
-rwxrwxrwx  1 vinnar vinnar    1349 Apr  3  2022 make_potter_fans_area.sh*
drwxrwxr-x  8 vinnar vinnar    4096 Jan 28  2022 overlay_fs_example/
drwxrwxr-x  6 vinnar vinnar    4096 Apr 11  2022 potter_fans/
drwxrwxr-x  3 vinnar vinnar    4096 Feb  6  2022 Potter_Fans_Ameneties/
drwxrwxr-x  3 vinnar vinnar    4096 Jun 15 20:53 Potter_Fans_Area/
-rw-rw-r--  1 vinnar vinnar     331 Apr  2  2022 PotterFans.txt
-rwxr-xr-x  1 vinnar vinnar     564 Feb  7  2022 sam_potter_fan.sh*
```

#Here I was testing having muliple overlay FSs with a common directory as well as using multiple directories for the lower layer.

```
vinnar@my-Virt:~/the_everglades$ sudo mount -t overlay -o lowerdir=./alpine_root_dir/:./Potter_Fans_Ameneties/:./potter_fans/lower_dir/,upperdir=./potter_fans/upper_dir/,workdir=./potter_fans/work_dir/ none potter_fans/merged_dir/
vinnar@my-Virt:~/the_everglades$ sudo mount -t overlay -o lowerdir=./alpine_root_dir/:./Fitness_Fans_Ameneties/:./fitness_fans/lower_dir/,upperdir=./fitness_fans/upper_dir/,workdir=./fitness_fans/work_dir/ none fitness_fans/merged_dir/
vinnar@my-Virt:~/the_everglades$ ll ./*_fans/merged_dir/*
-rwxr-xr-x 1 vinnar vinnar  446 Apr 11  2022 ./fitness_fans/merged_dir/bryan_fitness_fan.sh*
-rwxr-xr-x 1 vinnar vinnar  962 Apr 11  2022 ./fitness_fans/merged_dir/connie_fitness_fan.sh*
-rw-r--r-- 1 root   root    380 Apr 11  2022 ./fitness_fans/merged_dir/FitnessFans.txt
-rwxr-xr-x 1 vinnar vinnar  632 Apr 11  2022 ./potter_fans/merged_dir/alice_potter_fan.sh*
-rw-r--r-- 1 root   root    391 Apr 11  2022 ./potter_fans/merged_dir/PotterFans.txt
-rwxr-xr-x 1 vinnar vinnar  564 Apr 11  2022 ./potter_fans/merged_dir/sam_potter_fan.sh*

./fitness_fans/merged_dir/ameneties:
total 20
drwxrwxr-x 5 vinnar vinnar 4096 Feb  6  2022 ./
drwxrwxr-x 1 vinnar vinnar 4096 Apr 11  2022 ../
drwxrwxr-x 2 vinnar vinnar 4096 Jan 26  2022 Fitness_Ent_Center/
drwxrwxr-x 2 vinnar vinnar 4096 Jan 26  2022 Fitness_Hangout_Area/
drwxrwxr-x 2 vinnar vinnar 4096 Jan 26  2022 Healthy_Sandwiches/
```

#<snip a bunch of system directories>

```
./potter_fans/merged_dir/ameneties:
total 20
drwxrwxr-x 5 vinnar vinnar 4096 Feb  6  2022 ./
drwxrwxr-x 1 vinnar vinnar 4096 Apr 11  2022 ../
drwxrwxr-x 2 vinnar vinnar 4096 Jan 26  2022 Burgers_and_Fries/
drwxrwxr-x 2 vinnar vinnar 4096 Jan 26  2022 Potter_Ent_Center/
drwxrwxr-x 2 vinnar vinnar 4096 Jan 26  2022 Potter_Hangout_Area/
```

#<snip a bunch of system directories>

```
vinnar@my-Virt:~/the_everglades$ ll ./*_fans/merged_dir/
./fitness_fans/merged_dir/:
total 112
drwxrwxr-x 1 vinnar vinnar 4096 Apr 11  2022 ./
drwxrwxr-x 6 vinnar vinnar 4096 Apr 11  2022 ../
drwxrwxr-x 5 vinnar vinnar 4096 Feb  6  2022 ameneties/
drwxr-xr-x 1 vinnar vinnar 4096 Apr 11  2022 bin/
-rwxr-xr-x 1 vinnar vinnar  446 Apr 11  2022 bryan_fitness_fan.sh*
-rwxr-xr-x 1 vinnar vinnar  962 Apr 11  2022 connie_fitness_fan.sh*
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 dev/
drwxr-xr-x 1 vinnar vinnar 4096 Apr 11  2022 etc/
-rw-r--r-- 1 root   root    380 Apr 11  2022 FitnessFans.txt
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 home/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 lib/
drwxr-xr-x 5 vinnar vinnar 4096 Jan 28  2021 media/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 mnt/
drwxrwxr-x 2 vinnar vinnar 4096 Apr 11  2022 old_root/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 opt/
dr-xr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 proc/
drwx------ 1 vinnar vinnar 4096 Apr 11  2022 root/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 run/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 sbin/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 srv/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 sys/
drwxrwxr-x 2 vinnar vinnar 4096 Jan 28  2021 tmp/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 usr/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 var/

./potter_fans/merged_dir/:
total 112
drwxrwxr-x 1 vinnar vinnar 4096 Apr 11  2022 ./
drwxrwxr-x 6 vinnar vinnar 4096 Apr 11  2022 ../
-rwxr-xr-x 1 vinnar vinnar  632 Apr 11  2022 alice_potter_fan.sh*
drwxrwxr-x 5 vinnar vinnar 4096 Feb  6  2022 ameneties/
drwxr-xr-x 1 vinnar vinnar 4096 Apr 11  2022 bin/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 dev/
drwxr-xr-x 1 vinnar vinnar 4096 Apr 11  2022 etc/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 home/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 lib/
drwxr-xr-x 5 vinnar vinnar 4096 Jan 28  2021 media/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 mnt/
drwxrwxr-x 2 vinnar vinnar 4096 Apr 11  2022 old_root/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 opt/
-rw-r--r-- 1 root   root    391 Apr 11  2022 PotterFans.txt
dr-xr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 proc/
drwx------ 1 vinnar vinnar 4096 Apr 11  2022 root/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 run/
-rwxr-xr-x 1 vinnar vinnar  564 Apr 11  2022 sam_potter_fan.sh*
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 sbin/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 srv/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 sys/
drwxrwxr-x 2 vinnar vinnar 4096 Jan 28  2021 tmp/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 usr/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 var/
```

#Creating a file in the directory of the lower layer which is common to both of the overlay FSs to see what happens.

```
vinnar@my-Virt:~/the_everglades$ vi ./alpine_root_dir/file_in_alpine_rd_lowerdir.txt
vinnar@my-Virt:~/the_everglades$ grep "" ./*_fans/merged_dir/file_in_alpine_rd_lowerdir.txt
./fitness_fans/merged_dir/file_in_alpine_rd_lowerdir.txt:where does this file that's in the alpine root directory (which is one of the dirs making up the lower layer for both overlay FSs) show up?
./potter_fans/merged_dir/file_in_alpine_rd_lowerdir.txt:where does this file that's in the alpine root directory (which is one of the dirs making up the lower layer for both overlay FSs) show up?
```

#What if I edit that file?

```
vinnar@my-Virt:~/the_everglades$ !vi
vi ./alpine_root_dir/file_in_alpine_rd_lowerdir.txt
vinnar@my-Virt:~/the_everglades$ grep "" ./*_fans/merged_dir/file_in_alpine_rd_lowerdir.txt
./fitness_fans/merged_dir/file_in_alpine_rd_lowerdir.txt:where does this file that's in the alpine root directory (which is one of the dirs making up the lower layer for both overlay FSs) show up?
./potter_fans/merged_dir/file_in_alpine_rd_lowerdir.txt:where does this file that's in the alpine root directory (which is one of the dirs making up the lower layer for both overlay FSs) show up?
vinnar@my-Virt:~/the_everglades$ !vi
vi ./alpine_root_dir/file_in_alpine_rd_lowerdir.txt
vinnar@my-Virt:~/the_everglades$ cat ./fitness_fans/merged_dir/file_in_alpine_rd_lowerdir.txt 
where does this file that's in the alpine root directory (which is one of the dirs making up the lower layer for both overlay FSs) show up?
```

#There was no propagation of changes made to this file in the lower layer, even after several edits.


#I decided to test creating and then editing a lower layer file that was from a directory that was not common to both overlay FSs.

```
vinnar@my-Virt:~/the_everglades$ vi ./fitness_fans/lower_dir/file_in_fitnessfans_lowerdir.txt
vinnar@my-Virt:~/the_everglades$ grep "" ./fitness_fans/*/file_in_fitnessfans_lowerdir.txt
./fitness_fans/lower_dir/file_in_fitnessfans_lowerdir.txt:initial text of fitness fans lower dir file
./fitness_fans/merged_dir/file_in_fitnessfans_lowerdir.txt:initial text of fitness fans lower dir file
vinnar@my-Virt:~/the_everglades$ !vi
vi ./fitness_fans/lower_dir/file_in_fitnessfans_lowerdir.txt
vinnar@my-Virt:~/the_everglades$ grep "" ./fitness_fans/*/file_in_fitnessfans_lowerdir.txt
./fitness_fans/lower_dir/file_in_fitnessfans_lowerdir.txt:initial text of fitness fans lower dir file
./fitness_fans/lower_dir/file_in_fitnessfans_lowerdir.txt:
./fitness_fans/lower_dir/file_in_fitnessfans_lowerdir.txt:moar text
./fitness_fans/merged_dir/file_in_fitnessfans_lowerdir.txt:initial text of fitness fans lower dir file
```

#Here too no edits after creation propagate.  Is it because there's multiple overlay FSs?  I decided to remove one to test.

```
vinnar@my-Virt:~/the_everglades$ sudo umount -l ./fitness_fans/merged_dir 
vinnar@my-Virt:~/the_everglades$ vi ./potter_fans/lower_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt
vinnar@my-Virt:~/the_everglades$ grep "" ./potter_fans/*/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt 
./potter_fans/lower_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt:initial text or w/e
./potter_fans/merged_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt:initial text or w/e
vinnar@my-Virt:~/the_everglades$ !vi
vi ./potter_fans/lower_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt
vinnar@my-Virt:~/the_everglades$ grep "" ./potter_fans/*/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt 
./potter_fans/lower_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt:initial text or w/e
./potter_fans/lower_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt:
./potter_fans/lower_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt:moar text!
./potter_fans/merged_dir/file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt:initial text or w/e
```

#The result was the same: initial creation propagates, but further edits do not.

```
vinnar@my-Virt:~/the_everglades$ ll ./potter_fans/lower_dir/
total 20
drwxrwxr-x 2 vinnar vinnar 4096 Jun 20 19:48 ./
drwxrwxr-x 6 vinnar vinnar 4096 Apr 11  2022 ../
-rwxr-xr-x 1 vinnar vinnar  632 Apr 11  2022 alice_potter_fan.sh*
-rw-rw-r-- 1 vinnar vinnar   32 Jun 20 19:48 file_in_potter_fans_lower_dir_after_removal_of_fitnessfans.txt
-rwxr-xr-x 1 vinnar vinnar  564 Apr 11  2022 sam_potter_fan.sh*
```

#Here I edited a preexisting file which is a shell script.

```
vinnar@my-Virt:~/the_everglades$ vi ./potter_fans/lower_dir/alice_potter_fan.sh 
vinnar@my-Virt:~/the_everglades$ head !$
head ./potter_fans/lower_dir/alice_potter_fan.sh
#!/bin/bash

#adding comment to original file to see if propagates

file_name="PotterFans.txt"

if [ -f "$file_name" ]; then
	rm -f $file_name
fi

vinnar@my-Virt:~/the_everglades$ head ./potter_fans/merged_dir/alice_potter_fan.sh 
#!/bin/bash

file_name="PotterFans.txt"

if [ -f "$file_name" ]; then
	rm -f $file_name
fi

# Wait for Sam to check-in
sleep 10
```

#Unlike .txt files, editing the shell script did NOT propagate any changes to the merged layer.

```
vinnar@my-Virt:~/the_everglades$ sudo umount -l ./potter_fans/merged_dir 
vinnar@my-Virt:~/the_everglades$ ll ./alpine_root_dir/
total 80
drwxr-xr-x 19 vinnar vinnar 4096 Jun 20 19:39 ./
drwxrwxr-x 10 vinnar vinnar 4096 Jun 15 21:57 ../
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 bin/
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 dev/
drwxr-xr-x 15 vinnar vinnar 4096 Apr  3  2022 etc/
-rw-rw-r--  1 vinnar vinnar  158 Jun 20 19:38 file_in_alpine_rd_lowerdir.txt
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 home/
drwxr-xr-x  7 vinnar vinnar 4096 Jan 28  2021 lib/
drwxr-xr-x  5 vinnar vinnar 4096 Jan 28  2021 media/
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 mnt/
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 opt/
dr-xr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 proc/
drwx------  2 vinnar vinnar 4096 Jan 28  2021 root/
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 run/
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 sbin/
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 srv/
drwxr-xr-x  2 vinnar vinnar 4096 Jan 28  2021 sys/
drwxrwxr-x  2 vinnar vinnar 4096 Jan 28  2021 tmp/
drwxr-xr-x  7 vinnar vinnar 4096 Jan 28  2021 usr/
drwxr-xr-x 12 vinnar vinnar 4096 Jan 28  2021 var/
```

#I decided to try editing a file and then recreating the overlay FS without any directories shared with other overlay FSs.

```
vinnar@my-Virt:~/the_everglades$ vi ./alpine_root_dir/file_in_alpine_rd_lowerdir.txt 
vinnar@my-Virt:~/the_everglades$ ll ./fitness_fans/merged_dir/
total 8
drwxrwxr-x 2 vinnar vinnar 4096 Apr 11  2022 ./
drwxrwxr-x 6 vinnar vinnar 4096 Apr 11  2022 ../
vinnar@my-Virt:~/the_everglades$ sudo mount -t overlay -o lowerdir=./alpine_root_dir/,upperdir=./fitness_fans/upper_dir/,workdir=./fitness_fans/work_dir/ none fitness_fans/merged_dir/
vinnar@my-Virt:~/the_everglades$ ll ./fitness_fans/merged_dir/
total 104
drwxrwxr-x 1 vinnar vinnar 4096 Apr 11  2022 ./
drwxrwxr-x 6 vinnar vinnar 4096 Apr 11  2022 ../
drwxr-xr-x 1 vinnar vinnar 4096 Apr 11  2022 bin/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 dev/
drwxr-xr-x 1 vinnar vinnar 4096 Apr 11  2022 etc/
-rw-rw-r-- 1 vinnar vinnar   61 Jun 20 20:17 file_in_alpine_rd_lowerdir.txt
-rw-r--r-- 1 root   root    380 Apr 11  2022 FitnessFans.txt
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 home/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 lib/
drwxr-xr-x 5 vinnar vinnar 4096 Jan 28  2021 media/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 mnt/
drwxrwxr-x 2 vinnar vinnar 4096 Apr 11  2022 old_root/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 opt/
dr-xr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 proc/
drwx------ 1 vinnar vinnar 4096 Apr 11  2022 root/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 run/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 sbin/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 srv/
drwxr-xr-x 2 vinnar vinnar 4096 Jan 28  2021 sys/
drwxrwxr-x 2 vinnar vinnar 4096 Jan 28  2021 tmp/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 usr/
drwxr-xr-x 1 vinnar vinnar 4096 Jan 28  2021 var/
vinnar@my-Virt:~/the_everglades$ cat ./fitness_fans/merged_dir/file_in_alpine_rd_lowerdir.txt 
Ok so this is the initial text before creating the overlayfs
vinnar@my-Virt:~/the_everglades$ vi ./alpine_root_dir/file_in_alpine_rd_lowerdir.txt 
vinnar@my-Virt:~/the_everglades$ !cat
cat ./fitness_fans/merged_dir/file_in_alpine_rd_lowerdir.txt 
Ok so this is the initial text before creating the overlayfs
vinnar@my-Virt:~/the_everglades$ vi ./alpine_root_dir/file_in_alpine_rd_lowerdir.txt 
vinnar@my-Virt:~/the_everglades$ vi ./alpine_root_dir/file_in_alpine_rd_lowerdir.txt 
vinnar@my-Virt:~/the_everglades$ cat ./fitness_fans/merged_dir/file_in_alpine_rd_lowerdir.txt 
Ok so this is the initial text before creating the overlayfs
```

#So this time edits made to a file in the lower layer that existed prior to the creation of the overlay FS did *not* propagate to the merged layer.

```
vinnar@my-Virt:~/the_everglades$ sudo umount -l ./fitness_fans/merged_dir 
vinnar@my-Virt:~/the_everglades$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            2.9G     0  2.9G   0% /dev
tmpfs           589M  1.7M  587M   1% /run
/dev/sda5        49G   26G   21G  57% /
tmpfs           2.9G   84M  2.8G   3% /dev/shm
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           2.9G     0  2.9G   0% /sys/fs/cgroup
/dev/loop0      128K  128K     0 100% /snap/bare/5
/dev/loop2       64M   64M     0 100% /snap/core20/2318
/dev/loop3       56M   56M     0 100% /snap/core18/2823
/dev/loop4       62M   62M     0 100% /snap/core20/1611
/dev/loop1       56M   56M     0 100% /snap/core18/2538
/dev/loop5       71M   71M     0 100% /snap/core22/188
/dev/loop6      219M  219M     0 100% /snap/gnome-3-34-1804/93
/dev/loop7      219M  219M     0 100% /snap/gnome-3-34-1804/77
/dev/loop12      92M   92M     0 100% /snap/gtk-common-themes/1535
/dev/loop15      12M   12M     0 100% /snap/nmap/2655
/dev/loop9      9.7M  9.7M     0 100% /snap/htop/3417
/dev/loop11      82M   82M     0 100% /snap/gtk-common-themes/1534
/dev/loop14     347M  347M     0 100% /snap/gnome-3-38-2004/115
/dev/loop8      9.8M  9.8M     0 100% /snap/htop/4407
/dev/loop10      75M   75M     0 100% /snap/core22/1380
/dev/loop13     350M  350M     0 100% /snap/gnome-3-38-2004/143
/dev/loop18      55M   55M     0 100% /snap/snap-store/558
/dev/loop17      39M   39M     0 100% /snap/snapd/21759
/dev/loop16      12M   12M     0 100% /snap/nmap/3363
/dev/sda1       511M  4.0K  511M   1% /boot/efi
tmpfs           589M   92K  589M   1% /run/user/1001
none            2.9G  164K  2.9G   1% /run/qemu
/dev/loop20      13M   13M     0 100% /snap/snap-store/1113
/dev/loop21     506M  506M     0 100% /snap/gnome-42-2204/176
vmhgfs-fuse     466G  436G   30G  94% /mnt/hgfs
vinnar@my-Virt:~/the_everglades$ cd ./overlay_fs_example/
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll
total 32
drwxrwxr-x  8 vinnar vinnar 4096 Jan 28  2022 ./
drwxrwxr-x 10 vinnar vinnar 4096 Jun 15 21:57 ../
drwxrwxr-x  2 vinnar vinnar 4096 Jun 20 19:25 lower_dir/
drwxrwxr-x  2 vinnar vinnar 4096 Jan 28  2022 lower-dir-2/
drwxrwxr-x  2 vinnar vinnar 4096 Jan 28  2022 lower-dir-3/
drwxrwxr-x  2 vinnar vinnar 4096 Jan 27  2022 merged_dir/
drwxrwxr-x  2 vinnar vinnar 4096 Jun 20 19:25 upper_dir/
drwxrwxr-x  3 vinnar vinnar 4096 Jun 19 21:16 work_dir/
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./lower_dir/
total 20
drwxrwxr-x 2 vinnar vinnar 4096 Jun 20 19:25 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   43 Jun 19 21:16 ll1.txt
-rw-rw-r-- 1 vinnar vinnar   20 Jan 27  2022 ll2.txt
```

#Editing the files to reset them to their original state because I forgot to make a backup copy first.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./lower_dir/ll1.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ cat ./lower_dir/*
Duplicate in lower layer
File in lower dir
File-2 in lower dir
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./upper_dir/*
./upper_dir/dup.txt:Duplicate in upper layer
./upper_dir/ll2.txt:File-2 in lower dir
./upper_dir/ll2.txt:
./upper_dir/ll2.txt:this is modified from the merged dir, TWICE!
./upper_dir/ul1.txt:File-2 in Upper
./upper_dir/ul1.txt:
./upper_dir/ul1.txt:modifying the copy in the upper dir
./upper_dir/ul2.txt:File-2 in upper dir
./upper_dir/ul2.txt:
./upper_dir/ul2.txt:this is modified from the merged dir
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./upper_dir/ll2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./upper_dir/ll1.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./upper_dir/ul1.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./upper_dir/*
./upper_dir/dup.txt:Duplicate in upper layer
./upper_dir/ll2.txt:File-2 in lower dir
./upper_dir/ul1.txt:File-2 in Upper
./upper_dir/ul2.txt:File-2 in upper dir
./upper_dir/ul2.txt:
./upper_dir/ul2.txt:this is modified from the merged dir
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ rm -f ./upper_dir/ll2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./upper_dir/*
./upper_dir/dup.txt:Duplicate in upper layer
./upper_dir/ul1.txt:File-2 in Upper
./upper_dir/ul2.txt:File-2 in upper dir
./upper_dir/ul2.txt:
./upper_dir/ul2.txt:this is modified from the merged dir
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./upper_dir/ul2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./merged_dir/*
grep: ./merged_dir/*: No such file or directory
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll
total 32
drwxrwxr-x  8 vinnar vinnar 4096 Jan 28  2022 ./
drwxrwxr-x 10 vinnar vinnar 4096 Jun 15 21:57 ../
drwxrwxr-x  2 vinnar vinnar 4096 Jun 20 20:28 lower_dir/
drwxrwxr-x  2 vinnar vinnar 4096 Jan 28  2022 lower-dir-2/
drwxrwxr-x  2 vinnar vinnar 4096 Jan 28  2022 lower-dir-3/
drwxrwxr-x  2 vinnar vinnar 4096 Jan 27  2022 merged_dir/
drwxrwxr-x  2 vinnar vinnar 4096 Jun 20 20:30 upper_dir/
drwxrwxr-x  3 vinnar vinnar 4096 Jun 19 21:16 work_dir/
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./merged_dir/
total 8
drwxrwxr-x 2 vinnar vinnar 4096 Jan 27  2022 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
```

#Trying the overlay FS again with the original directories I was using in my first attempt to doublecheck functionality.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ sudo mount -t overlay -o lowerdir=./lower_dir/,upperdir=./upper_dir/,workdir=./work_dir/ none ./merged_dir/
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vim ./lower_dir/ll1.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll !$
ll ./lower_dir/ll1.txt
-rw-rw-r-- 1 vinnar vinnar 54 Jun 20 20:33 ./lower_dir/ll1.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ cat !$
cat ./lower_dir/ll1.txt
File in lower dir

edit after creating the overlay fs
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/ll1.txt
./lower_dir/ll1.txt:File in lower dir
./lower_dir/ll1.txt:
./lower_dir/ll1.txt:edit after creating the overlay fs
./merged_dir/ll1.txt:File in lower dir
./merged_dir/ll1.txt:
./merged_dir/ll1.txt:edit after creating the overlay fs
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vim ./lower_dir/ll1.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/ll1.txt
./lower_dir/ll1.txt:File in lower dir
./lower_dir/ll1.txt:
./lower_dir/ll1.txt:edit after creating the overlay fs
./lower_dir/ll1.txt:
./lower_dir/ll1.txt:second edit after creating overlay fs
./merged_dir/ll1.txt:File in lower dir
./merged_dir/ll1.txt:
./merged_dir/ll1.txt:edit after creating the overlay fs
```

#Again, the first edit in the lower layer is propagated to the merged layer, however any subsequent edits do not propagate.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./lower_dir/
total 20
drwxrwxr-x 2 vinnar vinnar 4096 Jun 20 20:36 ./
drwxrwxr-x 8 vinnar vinnar 4096 Jan 28  2022 ../
-rw-rw-r-- 1 vinnar vinnar   25 Jan 27  2022 dup.txt
-rw-rw-r-- 1 vinnar vinnar   93 Jun 20 20:36 ll1.txt
-rw-rw-r-- 1 vinnar vinnar   20 Jan 27  2022 ll2.txt
```

#Just in case the editor was having any effect I tried `vi` instead of `vim`.  On this VM they are separate binaries (`vi` is not a symlink to `vim`).

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./lower_dir/ll2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/ll2.txt
./lower_dir/ll2.txt:File-2 in lower dir
./lower_dir/ll2.txt:
./lower_dir/ll2.txt:first edit after creating overlay fs; note this was edited in `vi` not `vim`
./merged_dir/ll2.txt:File-2 in lower dir
./merged_dir/ll2.txt:
./merged_dir/ll2.txt:first edit after creating overlay fs; note this was edited in `vi` not `vim`
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./lower_dir/ll2.txt 
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/ll2.txt
./lower_dir/ll2.txt:File-2 in lower dir
./lower_dir/ll2.txt:
./lower_dir/ll2.txt:first edit after creating overlay fs; note this was edited in `vi` not `vim`
./lower_dir/ll2.txt:
./lower_dir/ll2.txt:second edit in vi
./merged_dir/ll2.txt:File-2 in lower dir
./merged_dir/ll2.txt:
./merged_dir/ll2.txt:first edit after creating overlay fs; note this was edited in `vi` not `vim`
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vim ./lower_dir/ll1.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/ll1.txt
./lower_dir/ll1.txt:File in lower dir
./lower_dir/ll1.txt:
./lower_dir/ll1.txt:edit after creating the overlay fs
./lower_dir/ll1.txt:
./lower_dir/ll1.txt:second edit after creating overlay fs
./lower_dir/ll1.txt:
./lower_dir/ll1.txt:3rd edit
./merged_dir/ll1.txt:File in lower dir
./merged_dir/ll1.txt:
./merged_dir/ll1.txt:edit after creating the overlay fs
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ ll ./*/ll1.txt
-rw-rw-r-- 1 vinnar vinnar 103 Jun 20 20:39 ./lower_dir/ll1.txt
-rw-rw-r-- 0 vinnar vinnar  54 Jun 20 20:33 ./merged_dir/ll1.txt
```

#Testing file creation in the lower layer.

```
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./lower_dir/new_file_in_lower.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/new_file_in_lower.txt
./lower_dir/new_file_in_lower.txt:initial text of newly created file in lower layer dir
./merged_dir/new_file_in_lower.txt:initial text of newly created file in lower layer dir
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ vi ./lower_dir/new_file_in_lower.txt
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ grep "" ./*/new_file_in_lower.txt
./lower_dir/new_file_in_lower.txt:initial text of newly created file in lower layer dir
./lower_dir/new_file_in_lower.txt:
./lower_dir/new_file_in_lower.txt:first edit after initial text, again done to the copy in lower layer dir
./merged_dir/new_file_in_lower.txt:initial text of newly created file in lower layer dir
vinnar@my-Virt:~/the_everglades/overlay_fs_example$ 
```

#Like edits, the when a file is created in the lower layer it propagates to the merged layer, but any subsequent edits do not propagate.
