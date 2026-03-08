# Guide: Rescuing and Recovering Data from Old Hard Disks on Linux 1.0

This guide describes a practical workflow for rescuing data from old hard disks
using Linux tools such as ddrescue, TestDisk, PhotoRec, find, and exiftool.

The process emphasizes safety, reproducibility, and minimizing risk to the
original disk.

Typical use cases include:

- Recovering old photos and videos from retired disks
- Recovering files from disks removed from old laptops
- Extracting data from disks that may have been reformatted
- Digital archaeology on disks from the 1990s–2010s

This guide assumes:

- You are using a Linux system (Ubuntu, Kubuntu, Debian, etc.)
- You have connected the disk via SATA, IDE/PATA adapter, or USB bridge
- You have enough storage space to create a full disk image

----------------------------------------------------------------------
1. Core Principle
----------------------------------------------------------------------

Always follow this order:

1. Connect the disk
2. Identify the device
3. Do not mount it read-write
4. Clone/image the disk first
5. Work only on the image afterwards
6. Recover files from the image
7. Sort and filter the recovered files

Golden rule:

Never perform recovery work directly on the only copy of the original disk
unless absolutely necessary.

----------------------------------------------------------------------
2. Install Required Tools
----------------------------------------------------------------------

Recommended tools:

- ddrescue
- testdisk
- photorec
- smartmontools
- exiftool

Install them on Ubuntu/Kubuntu:

sudo apt update
sudo apt install testdisk gddrescue smartmontools libimage-exiftool-perl

Notes:

- "testdisk" includes PhotoRec
- "gddrescue" provides ddrescue
- "smartmontools" provides smartctl
- "libimage-exiftool-perl" provides exiftool

----------------------------------------------------------------------
3. Connect the Disk Safely
----------------------------------------------------------------------

Before connecting the disk:

- Ensure you have enough free space for a full disk image
- Use the correct adapter for the disk type
- Avoid repeatedly plugging/unplugging the disk

Disk types:

SATA disks
- SATA data + SATA power connectors

Laptop IDE (PATA) disks
- 44-pin connector

Desktop IDE disks
- 40-pin ribbon + Molex power

When connecting:

- Align connectors carefully
- Ensure correct orientation
- Check jumper settings if using IDE

Listen for disk behavior:

Good signs
- smooth spin-up
- steady rotation

Bad signs
- repetitive clicking
- spin-up/spin-down cycles
- grinding noises

If mechanical distress occurs, stop stressing the disk.

----------------------------------------------------------------------
4. Identify the Disk
----------------------------------------------------------------------

List block devices:

lsblk

Example output:

sdb           8:16   0 298.1G  0 disk
├─sdb1        8:17   0   200M  0 part
├─sdb2        8:18   0 297.3G  0 part
└─sdb3        8:19   0 619.9M  0 part

Here the disk device is:

/dev/sdb

Check kernel detection messages:

sudo dmesg | tail -50

Always double-check device names before running recovery commands.

Confusing source and destination disks can destroy data.

----------------------------------------------------------------------
5. Avoid Mounting the Disk
----------------------------------------------------------------------

Do not open the disk in a graphical file manager before imaging.

Mounting read-write can modify filesystem metadata.

For a simple and safe workflow, avoid mounting entirely until after
a full disk image is created.

----------------------------------------------------------------------
6. Check Disk Health (SMART)
----------------------------------------------------------------------

Run a SMART check:

sudo smartctl -a /dev/sdX

Example:

sudo smartctl -a /dev/sdb

If SMART data does not appear when using a USB adapter, try:

sudo smartctl -a -d sat /dev/sdX

Key attributes to examine:

Reallocated_Sector_Ct
Current_Pending_Sector
Offline_Uncorrectable

Healthy disks often show:

Reallocated_Sector_Ct = 0
Current_Pending_Sector = 0
Offline_Uncorrectable = 0
SMART overall-health test result: PASSED

Even disks with some errors may still be recoverable.

----------------------------------------------------------------------
7. Prepare Recovery Workspace
----------------------------------------------------------------------

Create a recovery directory:

mkdir -p ~/recovery/diskname

Check free disk space:

df -h ~

You generally need:

- space for the disk image
- space for recovered files
- extra system margin

Example:

A 320 GB disk may require ~500 GB free space to work comfortably.

Important:

Never store recovery images or recovered files on the disk you are trying to
recover from, even if there is other free partitions. 

----------------------------------------------------------------------
8. Create a Disk Image with ddrescue
----------------------------------------------------------------------

Imaging the disk is the most important step.

Basic command:

sudo ddrescue -f -n /dev/sdX ~/recovery/diskname/disk.img ~/recovery/diskname/disk.log

Example:

sudo ddrescue -f -n /dev/sdb ~/recovery/macdisk/disk.img ~/recovery/macdisk/disk.log

Meaning:

/dev/sdX     = source disk
disk.img     = disk image file
disk.log     = recovery log

Options:

-f   allow writing to existing output file
-n   fast first pass without retries

ddrescue advantages:

- reads disk sequentially
- skips slow sectors initially
- records progress in log file
- allows resuming interrupted imaging

If a USB adapter causes resets or unstable reads, you may try:

sudo ddrescue -f -d -n /dev/sdX disk.img disk.log

The -d option sometimes improves compatibility with USB bridges.

After imaging, verify the image exists:

ls -lh ~/recovery/diskname/disk.img

----------------------------------------------------------------------
9. Work on the Image, Not the Disk
----------------------------------------------------------------------

Once imaging completes successfully:

- unplug the original disk if possible
- perform recovery only on the disk image

Example image:

~/recovery/diskname/disk.img

----------------------------------------------------------------------
10. Decide Between TestDisk and PhotoRec
----------------------------------------------------------------------

Two recovery approaches exist.

TestDisk

Use when:

- filesystem may still exist
- partitions may be damaged
- you want original filenames

PhotoRec

Use when:

- files were deleted long ago
- disk was reformatted
- filesystem metadata is lost

Summary:

Filesystem recovery -> TestDisk
Deep raw recovery -> PhotoRec

----------------------------------------------------------------------
11. Try TestDisk First (If Filesystem May Exist)
----------------------------------------------------------------------

Run TestDisk on the image:

sudo testdisk ~/recovery/diskname/disk.img

Capabilities:

- rebuild partition tables
- repair boot sectors
- browse filesystem
- copy files with original names

Best case:

You can navigate directories and copy files normally.

If no usable filesystem remains, proceed to PhotoRec.

----------------------------------------------------------------------
12. Run PhotoRec for Deep File Recovery
----------------------------------------------------------------------

Run PhotoRec on the disk image:

sudo photorec ~/recovery/diskname/disk.img

Typical workflow inside PhotoRec:

1. Select disk image
2. Select partition table type
3. Enter "File Opt"
4. Disable all file types
5. Enable only desired types

Recommended formats for photo recovery:

jpg
png
bmp
mov
mp4
3gp

Optional:

zip (sometimes useful for archived images)

Select output directory and start scan.

----------------------------------------------------------------------
13. Expect Large Amounts of Junk Files
----------------------------------------------------------------------

PhotoRec recovers files by signature.

It does not preserve:

- original filenames
- directory structure

Recovered files typically appear as:

f123456.jpg
f123457.mov

Inside directories like:

recup_dir.1
recup_dir.2
recup_dir.3

Recovered data may include:

- browser cache
- icons
- thumbnails
- UI graphics
- fragments

Filtering is therefore essential.

----------------------------------------------------------------------
14. Inspect and Count Recovered Files
----------------------------------------------------------------------

Count JPG files:

find ~/recovery/diskname/photorec_output -type f -name "*.jpg" | wc -l

List common file types:

find ~/recovery/diskname/photorec_output -type f | sed 's/.*\.//' | sort | uniq -c | sort -nr | head

Inspect EXIF metadata:

exiftool file.jpg

Or:

exiftool -DateTimeOriginal -Model -Megapixels file.jpg

----------------------------------------------------------------------
15. Filter by File Size (Heuristic)
----------------------------------------------------------------------

Real phone photos are often much larger than browser junk.

Typical rough sizes:

Browser/UI images
1 KB – 100 KB

Phone photos
300 KB – several MB

Preview files smaller than 150 KB:

find photorec_output -type f -name "*.jpg" -size -150k

If they appear to be junk, delete:

find photorec_output -type f -name "*.jpg" -size -150k -delete

Note:

150 KB is a heuristic threshold, not a guarantee.

----------------------------------------------------------------------
16. Filter by Image Resolution
----------------------------------------------------------------------

Create a directory for high-resolution photos:

mkdir -p ~/recovery/diskname/highres_photos

Filter photos with resolution above 1 megapixel:

exiftool -if 'defined $Megapixels and $Megapixels > 1' \
-ext jpg -r photorec_output \
-copyto ~/recovery/diskname/highres_photos

This removes many tiny UI images.

----------------------------------------------------------------------
17. Filter by Camera Model (Optional)
----------------------------------------------------------------------

Inspect camera models present:

exiftool -Model -r photorec_output | sort | uniq -c | sort -nr | head

Example camera identifiers:

iPhone
Nokia

Optional filter:

mkdir -p ~/recovery/diskname/phone_photos

exiftool -if '$Model =~ /iPhone|Nokia/' \
-ext jpg -r photorec_output \
-copyto ~/recovery/diskname/phone_photos

Note:

Some recovered photos may lack EXIF model metadata.

----------------------------------------------------------------------
18. Filter by Date Range
----------------------------------------------------------------------

Example: extract photos from June 2011 to December 2012.

mkdir -p ~/recovery/diskname/iphone_2011_2012

exiftool -if 'defined $DateTimeOriginal and $DateTimeOriginal ge "2011:06:01 00:00:00" and $DateTimeOriginal lt "2013:01:01 00:00:00"' \
-ext jpg -r phone_photos \
-copyto ~/recovery/diskname/iphone_2011_2012

If no files match:

Verify metadata exists using:

exiftool file.jpg

----------------------------------------------------------------------
19. Rename Files by EXIF Date
----------------------------------------------------------------------

PhotoRec filenames are meaningless.

Rename using EXIF timestamps:

exiftool '-FileName<DateTimeOriginal' \
-d %Y%m%d_%H%M%S%%-c.%%e \
~/recovery/diskname/iphone_2011_2012

Example result:

20110718_142201.jpg
20121231_235944.jpg

----------------------------------------------------------------------
20. Useful Linux File Commands
----------------------------------------------------------------------

Find files recursively:

find directory -name "*.jpg"

Delete matching files:

find directory -name "*.tmp" -delete

Copy matching files:

find directory -name "*.mp4" -exec cp -t videos {} +

Count files:

find directory -name "*.jpg" | wc -l

Mental model:

grep -> search inside files
find -> search for files
wc -> count results
| -> pipe results to next command

----------------------------------------------------------------------
21. Delete Entire Directory Trees
----------------------------------------------------------------------

Equivalent to old DOS deltree:

rm -r directory

Interactive deletion:

rm -ri directory

Force deletion:

rm -rf directory

Use -rf carefully.

----------------------------------------------------------------------
22. Kubuntu Trash Location
----------------------------------------------------------------------

Trash directory:

~/.local/share/Trash/

Subdirectories:

files/
info/

If permissions prevent deletion:

sudo rm -rf ~/.local/share/Trash/files/*
sudo rm -rf ~/.local/share/Trash/info/*

----------------------------------------------------------------------
23. Fix File Ownership After Recovery
----------------------------------------------------------------------

Files created with sudo may belong to root.

Fix ownership:

sudo chown -R "$USER":"$USER" ~/recovery

----------------------------------------------------------------------
24. Linux Tools vs Commercial Recovery Software
----------------------------------------------------------------------

Commercial tools (Disk Drill, etc.) integrate:

- disk scanning
- partition repair
- deep file carving
- preview interface

Linux tools provide similar capabilities through separate tools:

ddrescue
testdisk
photorec
exiftool

Advantages of Linux workflow:

- excellent disk imaging with ddrescue
- flexible automation
- powerful filtering tools
- scriptable workflows

Commercial tools often provide easier interfaces.

----------------------------------------------------------------------
25. Recommended Workflow Summary
----------------------------------------------------------------------

1. Connect disk
2. Identify device with lsblk
3. Check SMART status
4. Create recovery directory
5. Image disk with ddrescue
6. Work only on the disk image
7. Attempt filesystem recovery with TestDisk
8. Run PhotoRec if necessary
9. Filter recovered files by size/resolution/date
10. Rename files by EXIF timestamp

----------------------------------------------------------------------
26. Final Advice
----------------------------------------------------------------------

Successful recovery often involves two phases:

Extraction
and
Filtering

PhotoRec may recover tens of thousands of files.

Filtering by:

size
resolution
EXIF date
camera model

can reduce the dataset dramatically and reveal the meaningful photos.
