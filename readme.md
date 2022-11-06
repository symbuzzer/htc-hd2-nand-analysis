1-UNDERSTANDING HTC HD2 AMAGLDR'S NAND DESIGN, USING "LAST 24 MB OF NAND" FEATURE  
  
MANIFEST  
each nand block's size is 128kb so eu hd2's full nand size is 1000 sector.(hexdecimal[512*1024/128])  
magldr's cfg starting 216. block (decimal[2169]*128/1024=66mb). It is on 216, 217 and 218. blocks.  
this area includes ipl, radio rom (which is 23.8mb), (h)spl (which is 0.5mb) and magldr (which is 0.5mb) and other which are I have no idea.  
so android's first partition starts 219. blocks.  
every boot process; spl erases last 24 mb of nand. (which starts from f40 to 1000)(1000-hexdecimal[24*1024/128])  
so we couldnt use first 218 and last 0c blocks. We only can use from 219 to f40 blocks, which is size 420mb for eu. (decimal[f40-219]*128/1024)  
for tmous we have more 512mb too. so we can use tmous nand for android 932mb. (420+512)  
  
THEORY  
with magldr, we can use last 24mb for android too. so we have 444mb usable nand memory for eu, 956mb usable nand memory for tmous.  
probably magldr blocks spl erasing process when phone is booting. but sometimes magldr cant do that. for example if you can use reset button which is under battery cover or flash anyhing from spl via any pc app, last 24mb will erased.  
so if we want to use last 24mb with android, we wont do these. If we do, our last partition (it is data probably) will corrupted.  
  
  
2-UNDERSTANDING HTC HD2 AMAGLDR'S PARTITION STRUCTURE  
  
Hi dear friends. I writed a note about aMagldr's partiton system last week and I wanted to share it with you.  
If you dont know what is mtty, please look this post on xda for installation PC drivers and how to use it: http://forum.xda-developers.com/showthread.php?t=623356  
After this step, you can communicate with your phones spl via mtty brigde. It is like fastboot or adb, but it is more limited.  
aMagldr has own mtty commands. If you select "USB TTY" option from its menu, you can apply them. For example "help" and "ad help" commands open help aMagldr's commands list.  
Oldie aMagldr 1.08 hadnt got DAF installer for installing recovery or android kernel & rom and nand partitioning. In this version, developers must to use mtty commands. You can see this progress from here: http://www.mobile01.com/topicdetail.php?f=224&t=1827891  
We can still make partitions with this method, but we need to understanding aMagldr's structure before it. So I tried to understanding this method and finally I did it. There isnt any pratical brief for us I know. But maybe, someone deals with it, who knows;)  
  
EXAMPLE INPUT:  
ad addpart misc 8 50 41  
ad addpart recovery 3E 62 D  
ad addpart boot 28 50 49  
ad addpart system 398 50 41  
ad addpart userdata 921 50 51  
  
(I dont use cache partition, so I neednt "ad addpart cache ...")  
  
EXAMPLE OUTPUT:  
Name Size R.Size Start Type Flags  
misc | 8 | 8 | 219 | 50 | 41  
recovery | 3E | 3E | 221 | 62 | D  
boot | 28 | 28 | 25F | 61 | 49  
system | 398 | 395 | 287 | 50 | 41  
userdata | 921 | 91D | 61F | 50 | 51  
  
THESE MEAN...  
  
general formula:  
-size = decimal(Size)x 128 KB  
-type:  
--ya = 50  
--raw = ?  
--yboot = 61  
--rboot = ?  
--rrecov = 62  
-flag:  
--hr = 41  
--ro = ?  
--nospr = ?  
--asize =  
--ro|hr = 49  
--ro|nospr = D  
--ro|nospr|hr = ?  
--asize|hr = 51  
(Known vales are enough for nearly all android roms, so no need to find others)  
  
1) misc:  
-size is: 8x128= 1024 KB = 1 MB (8 is decimal value of 8)  
-type is: ya  
-flag is: hr  
  
2) recovery:  
-size is: 62x128= 7936 KB = 7,750 MB (62 is decimal value of 3E)  
-type is: rrecov  
-flag is: ro|nospr  
  
3) boot:  
-size is: 40x128= 5120 KB = 5 MB (40 is decimal value of 28)  
-type is: yboot  
-flag is: ro|hr  
  
4) system:  
-size is: 920x128= 117760 KB = 115 MB (920 is decimal value of 398)  
-type is: ya  
-flag is: hr  
  
5) userdata:  
-size is: 2337x128= 299136 KB = 292,125 MB (2337 is decimal value of 921)  
-type is: ya  
-flag is: asize|hr  
  
PARTITION TABLE FOR DAF'S FLASH.CFG FILE:  
misc ya|hr 1M  
recovery rrecov|ro|nospr filesize recovery-raw.img  
boot yboot|ro|hr 5M  
system ya|hr 115M  
userdata ya|asize|hr allsize  
(my recovery-raw.img file size is 7,750)  
  
BAD BLOCKS:  
We can see them with this formula:  
Number of Bad Blocks = decimal(Size)-decimal(R. Size)  
If R.Size = Size, there isnt any bad block on this partition  
1) I see; I have no bad blocks on my misc, recovery and boot partitions. Because, their R. Size's = Size's  
2) system:  
-have 920-917 = 3 bad blocks (920 is decimal value of 398, 917 is decimal value of 395)  
3) data:  
-have 2337-2333 = 4 bad blocks (2337 is decimal value of 921, 2333 is decimal value of 91d)  
