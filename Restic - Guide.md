# Restic - The Ideal Backup Solution
Deep Grewal - December 7, 2020

___
## Why Restic?
The backup solution space is crowded.  There are a multitude of applications, commands, and methodologies that are available.  Choosing the right one is a daunting task that can definitely leave your head spinning.  After having tried a few of the options out there, I have settled with Restic and here's why.  

### Open Source
First and foremost, Restic is completely open source.  This is important especially when you are choosing a solution that will handle your very own data, which often times is personal and sensitive in nature.  That's the whole reason that it is being backed up in the first place...because you deem it as important.  So, when it comes to backups, it is a requirement that the solution be non-proprietary, open source, and auditable.  There should not be any concern of nefarious data harvesting or data siphoning to an overly interested third-party.  Your data should remain **your** data.   

### Deduplication
Although storage space is inexpensive these days, it is always wise to use the available storage space sparingly and efficiently.  Restic does just that.  During the creation of a backup (known as a snapshot), Restic intelligently ensures that deduplication is occurring.  This prevents the backing up of entire copies of the same files or objects over and over again with each additional snapshot.  The end result, of course, is less space used for backups when compared to more traditional backup methods which use storage space quite greedily.

### Encryption
Backups which are not encrypted simply aren't secure.  It is easy to create a backup and forget about it, especially if the backup was created on unencrypted removable storage or unencrypted cloud storage.  Anyone who gets a hold of this storage can now view the data fairly easily and do some very bad things with it.  This is why it is important to have encrypted backups.  Restic, by default, encrypts backups regardless of their destination.    

### Scale and Scope
Some backup tools are limited in their functionality and compatibility.  They can only perform a backup a certain way or can only store the backup on certain locations.  They only run on certain operating systems.  Restic, on the other hand, is extremely flexible and diverse.  Restic is compatible with Linux, BSD, Mac, and Windows.  In addition to being able to create backups to local, SFTP, and REST servers, it is also capable of backing up to major cloud storage providers such as Backblaze B2, Wasabi, Openstack Swift, Amazon S3, Google Cloud, Microsoft Azure, etc..

### Simplicity and Sophistication
Restic provides you with a set of commands that can be used as building blocks for a variety of backup strategies, from simple to complex.  Each command is well documented and can be viewed using man pages or the `--help` parameter after the command.  The project website also contains excellent documentation and samples at https://restic.readthedocs.io/en/stable/.  This abundance of information promotes the creation of scripts (Perl, Python, BASH, etc.) and cron jobs that can further streamline and automate the process of creating routine backups.

### Snapshots
If you have worked with virtual machines or the Btrfs file system, the concept of snapshots should be very familiar.  The output or product of a Restic backup is a snapshot.  Each snapshot is a basic unit within Restic and contains a record of files, directories, and objects as they existed when the snapshot was created.  Snapshots can be compared, restored, mounted, and deleted.  They promote incremental backup strategies which are space-efficient.
___
## Setting the Stage
This guide will detail the process of using Restic.  We will generate data for backing up, download and install Restic, create backups, compare backups, restore backups, and delete unneeded backups.  For the purposes of this example, I have created an Ubuntu Server 20.04 virtual machine.  

### Generating Data for a Backup
In order to demonstrate Restic, we should create some subdirectories and files that are "important" and worthy of being backed up.

First, let's navigate to the home directory and create the `files` and `notes` subdirectories.

```
deep@ubuntu-vm:~$ cd ~

deep@ubuntu-vm:~$ mkdir files notes

deep@ubuntu-vm:~$ ls
files  notes
```

Next, let's create some files within each of these subdirectories and verify their creation.

```
deep@ubuntu-vm:~$ touch files/file{1..3}.txt

deep@ubuntu-vm:~$ touch notes/note{1..3}.txt

deep@ubuntu-vm:~$ ls *
files:
file1.txt  file2.txt  file3.txt

notes:
note1.txt  note2.txt  note3.txt
```

### Installing Restic
Restic can be installed using the `apt install` command.

```
deep@ubuntu-vm:~$ sudo apt install restic
Reading package lists... Done
Building dependency tree
Reading state information... Done
Suggested packages:
  libjs-jquery libjs-underscore
The following NEW packages will be installed:
  restic
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 8524 kB of archives.
After this operation, 25.7 MB of additional disk space will be used.
Get:1 http://us.archive.ubuntu.com/ubuntu focal/universe amd64 restic amd64 0.9.6+ds-2 [8524 kB]
Fetched 8524 kB in 10s (846 kB/s)
Selecting previously unselected package restic.
(Reading database ... 108642 files and directories currently installed.)
Preparing to unpack .../restic_0.9.6+ds-2_amd64.deb ...
Unpacking restic (0.9.6+ds-2) ...
Setting up restic (0.9.6+ds-2) ...
Processing triggers for man-db (2.9.1-1) ...
```

### Restic Commands
The Restic program contains a variety of commands.  This guide will not cover all of the available commands and their respective options/flags.  This guide will cover some of the most commonly used commands needed to make and manage backups.  For a list of all Restic commands, simply type `restic` and press `Enter`.

```
deep@ubuntu-vm:~$ restic

restic is a backup program which allows saving multiple revisions of files and
directories in an encrypted repository stored on different backends.

Usage:
  restic [command]

Available Commands:
  backup        Create a new backup of files and/or directories
  cache         Operate on local cache directories
  cat           Print internal objects to stdout
  check         Check the repository for errors
  diff          Show differences between two snapshots
  dump          Print a backed-up file to stdout
  find          Find a file, a directory or restic IDs
  forget        Remove snapshots from the repository
  generate      Generate manual pages and auto-completion files (bash, zsh)
  help          Help about any command
  init          Initialize a new repository
  key           Manage keys (passwords)
  list          List objects in the repository
  ls            List files in a snapshot
  migrate       Apply migrations
  mount         Mount the repository
  prune         Remove unneeded data from the repository
  rebuild-index Build a new index file
  recover       Recover data from the repository
  restore       Extract the data from a snapshot
  self-update   Update the restic binary
  snapshots     List all snapshots
  stats         Scan the repository and show basic statistics
  tag           Modify tags on snapshots
  unlock        Remove locks other processes created
  version       Print version information

Flags:
      --cacert file               file to load root certificates from (default: use system certificates)
      --cache-dir string          set the cache directory. (default: use system default cache directory)
      --cleanup-cache             auto remove old cache directories
  -h, --help                      help for restic
      --json                      set output mode to JSON for commands that support it
      --key-hint string           key ID of key to try decrypting first (default: $RESTIC_KEY_HINT)
      --limit-download int        limits downloads to a maximum rate in KiB/s. (default: unlimited)
      --limit-upload int          limits uploads to a maximum rate in KiB/s. (default: unlimited)
      --no-cache                  do not use a local cache
      --no-lock                   do not lock the repo, this allows some operations on read-only repos
  -o, --option key=value          set extended option (key=value, can be specified multiple times)
      --password-command string   specify a shell command to obtain a password (default: $RESTIC_PASSWORD_COMMAND)
  -p, --password-file string      read the repository password from a file (default: $RESTIC_PASSWORD_FILE)
  -q, --quiet                     do not output comprehensive progress report
  -r, --repo string               repository to backup to or restore from (default: $RESTIC_REPOSITORY)
      --tls-client-cert string    path to a file containing PEM encoded TLS client certificate and private key
  -v, --verbose n                 be verbose (specify --verbose multiple times or level n)

Use "restic [command] --help" for more information about a command.
```

#### Getting Help
Restic is a well-documented program.  In addition to man pages, you can obtain helpful information on any of the restic commands by simply typing the command followed by `--help`.  Soon, we will be creating (initializing) a new Restic repository for data backups.  Let's have a look at the help information for this command.

```
deep@ubuntu-vm:~$ restic init --help

The "init" command initializes a new repository.

Usage:
  restic init [flags]

Flags:
  -h, --help   help for init

Global Flags:
      --cacert file               file to load root certificates from (default: use system certificates)
      --cache-dir string          set the cache directory. (default: use system default cache directory)
      --cleanup-cache             auto remove old cache directories
      --json                      set output mode to JSON for commands that support it
      --key-hint string           key ID of key to try decrypting first (default: $RESTIC_KEY_HINT)
      --limit-download int        limits downloads to a maximum rate in KiB/s. (default: unlimited)
      --limit-upload int          limits uploads to a maximum rate in KiB/s. (default: unlimited)
      --no-cache                  do not use a local cache
      --no-lock                   do not lock the repo, this allows some operations on read-only repos
  -o, --option key=value          set extended option (key=value, can be specified multiple times)
      --password-command string   specify a shell command to obtain a password (default: $RESTIC_PASSWORD_COMMAND)
  -p, --password-file string      read the repository password from a file (default: $RESTIC_PASSWORD_FILE)
  -q, --quiet                     do not output comprehensive progress report
  -r, --repo string               repository to backup to or restore from (default: $RESTIC_REPOSITORY)
      --tls-client-cert string    path to a file containing PEM encoded TLS client certificate and private key
  -v, --verbose n                 be verbose (specify --verbose multiple times or level n)
```
___
## Taking the Initial Snapshot
Restic takes and stores backups (snapshots) of data into a backup location known as a repository.  Each repository is an encrypted space (directory) which securely stores the data for management and future retrieval.  The initial snapshot for a repository takes slightly longer to create and is larger in size than subsequent snapshots.  The initial snapshot is a full backup whereas subsequent backups contain only the objects that have changed (incremental).

### Initialize a Restic Repository
Creating or initializing a repository (also known as a "repo") is the first step in performing backups.  We have to create a location where the backed up data will reside. In our example, we will simply create a repository in the home directory alongside the `files` and `notes` directories created earlier.  This repository will be named "repo" and will be initialized using `restic init -r repo`.  Where the `-r` parameter specifies that the text that will come immediately after it is the path to the repository.  Each repository requires a password for encryption purposes.

```
deep@ubuntu-vm:~$ restic init -r repo
enter password for new repository:
enter password again:
created restic repository 7bfcc5b45d at repo

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

<br/>

>***Note:** It is best practice to create a repository on a separate physical medium such as another disk drive, removable/external drive, NAS, server, or cloud location.*

<br/>

### Analyze the Repository
A repository is simply a directory which is initialized by Restic to store backed up data.  The repository appears as any other directory.  An existing directory can be used as a repository or Restic can be used to create the repository (as a new directory).  When Restic creates the repository as a new directory, by default, only the owner is given full permissions to the directory (700).  

```
deep@ubuntu-vm:~$ ls -l
total 12
drwxrwxr-x 2 deep deep 4096 Dec  7 19:01 files
drwxrwxr-x 2 deep deep 4096 Dec  7 19:01 notes
drwx------ 7 deep deep 4096 Dec  7 21:07 repo
```

Once initialized, the repository contains the subdirectories and files needed to create, store, and manage backups.

```
deep@ubuntu-vm:~$ ls -l repo
total 24
-rw-------   1 deep deep  155 Dec  7 21:07 config
drwx------ 258 deep deep 4096 Dec  7 21:07 data
drwx------   2 deep deep 4096 Dec  7 21:07 index
drwx------   2 deep deep 4096 Dec  7 21:07 keys
drwx------   2 deep deep 4096 Dec  7 21:07 locks
drwx------   2 deep deep 4096 Dec  7 21:07 snapshots
```

### Take the Initial Snapshot
Now that we have some "important" data along with a backup location, we have all of the ingredients needed to create the first backup.  In Restic, creating a backup is synonymous with creating a snapshot.  Let's backup the `files` and `notes` directories into the `repo` backup repository using the `restic backup` command.

```
deep@ubuntu-vm:~$ restic backup files notes -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
created new cache in /home/deep/.cache/restic

Files:           6 new,     0 changed,     0 unmodified
Dirs:            0 new,     0 changed,     0 unmodified
Added to the repo: 704 B

processed 6 files, 0 B in 0:00
snapshot 93591a06 saved
```

### Review the Repository and Confirm the Snapshot
From a very high-level, we can view the statistics associated with the repository with the basic `restic stats` command.

```
deep@ubuntu-vm:~$ restic stats -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
scanning...
Stats for all snapshots in restore-size mode:
  Total File Count:   8
        Total Size:   0 B
```

To verify that the snapshot has been created, let's list all snapshots that exist in the repository.  The handy `restic snapshots` command is very useful in displaying the snapshots contained within a repository.  It will be used throughout this guide.  Note that the each snapshot has its own unique ID (which is required when working with snapshots).  

```
deep@ubuntu-vm:~$ restic snapshots -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
ID        Time                 Host        Tags        Paths
-----------------------------------------------------------------------
93591a06  2020-12-07 21:40:01  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes
-----------------------------------------------------------------------
1 snapshots
```

We can take it one step further with the `restic ls` command to list the files within the snapshot (ID = 93591a06), itself.  

```
deep@ubuntu-vm:~$ restic ls 93591a06 -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
snapshot 93591a06 of [/home/deep/files /home/deep/notes] filtered by [] at 2020-12-07 21:40:01.82686587 +0000 UTC):
/files
/files/file1.txt
/files/file2.txt
/files/file3.txt
/notes
/notes/note1.txt
/notes/note2.txt
/notes/note3.txt
```
___
## Taking the Second Snapshot
Before we take the second snapshot, let's make some changes to the objects that are being backed up.  We will remove all of the files in the "notes" directory.

```
deep@ubuntu-vm:~$ rm notes/*
```

Also, let's add another file named "file4.txt" to the files directory.  Unlike the other files, this file will actually contain some text.  Actually, it will contain a LOT of text.  We can use the `curl` command to bring in the entire GPL v3 into this file.

```
deep@ubuntu-vm:~$ curl https://www.gnu.org/licenses/gpl-3.0.txt > files/file4.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 35149  100 35149    0     0  68650      0 --:--:-- --:--:-- --:--:-- 68650

deep@ubuntu-vm:~$ ls -l files
total 36
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file1.txt
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file2.txt
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file3.txt
-rw-rw-r-- 1 deep deep 35149 Dec  7 22:39 file4.txt
```

### Take the Second Snapshot
Now then, let's create that second snapshot, shall we?  Once again, we will use the `restic backup` command to make yet another snapshot.  Notice that in the output of the command, additional data was added to the repo.  The size of this additional data is approximately 35 KB.  (This can be attributed to file4.txt containing the entire GPL v3!)

```
deep@ubuntu-vm:~$ restic backup files notes -r repo
enter password for repository:
\repository 7bfcc5b4 opened successfully, password is correct

Files:           1 new,     0 changed,     3 unmodified
Dirs:            0 new,     0 changed,     0 unmodified
Added to the repo: 35.013 KiB

processed 4 files, 34.325 KiB in 0:00
snapshot 9d5ddbeb saved
```

### Review the Repository and Confirm the Snapshot
Let's have another look at the repository and its snapshots to see where things are using the commands we executed earlier.

From a very high-level, the `restic stats` command outputs the statistics associated with the repository.  Note that the file count and size have both increased.

```
deep@ubuntu-vm:~$ restic stats -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
scanning...
Stats for all snapshots in restore-size mode:
  Total File Count:   14
        Total Size:   34.325 KiB
```

To verify that both of the snapshots have been created, let's list all snapshots that exist in the repository using the ever-so-handy `restic snapshots` command.  Note that each of the two snapshots has its own unique ID.  

```
deep@ubuntu-vm:~$ restic snapshots -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
ID        Time                 Host        Tags        Paths
-----------------------------------------------------------------------
93591a06  2020-12-07 21:40:01  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes

9d5ddbeb  2020-12-07 22:45:31  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes
-----------------------------------------------------------------------
2 snapshots
```

Finally, we can take it one step further with the `restic ls` command and list the files within the second snapshot (ID = 9d5ddbeb).  Listing the contents for this snapshot does indicate that it contains less files than the initial snapshot (since all of the contents of the notes directory were removed).

```
deep@ubuntu-vm:~$ restic ls 9d5ddbeb -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
snapshot 9d5ddbeb of [/home/deep/files /home/deep/notes] filtered by [] at 2020-12-07 22:45:31.34307187 +0000 UTC):
/files
/files/file1.txt
/files/file2.txt
/files/file3.txt
/files/file4.txt
/notes
```
___
## Comparing Snapshots
Once 2 or more snapshots have been created, they can be compared.  The snapshot ID of each snapshot is required in order to perform this comparison.  The results of the comparison will indicate what has changed between the snapshots.  

### Obtain Snapshot IDs
First, let's get the ID for each of the snapshots.  The easiest way to do so is to use the `restic snapshots` command to list the snapshots in the repo.

```
deep@ubuntu-vm:~$ restic snapshots -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
ID        Time                 Host        Tags        Paths
-----------------------------------------------------------------------
93591a06  2020-12-07 21:40:01  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes

9d5ddbeb  2020-12-07 22:45:31  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes
-----------------------------------------------------------------------
2 snapshots
```

### Compare First Snapshot with Second Snapshot
We can use the The `restic diff` command can to compare the first snapshot with the second snapshot (chronological order) and list the IDs for each snapshot within the same respective order.  The output essentially indicates that a new file was added (35.501 KB), while 3 files were removed (1.635 KB). 

```
deep@ubuntu-vm:~$ restic diff 93591a06 9d5ddbeb -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
comparing snapshot 93591a06 to 9d5ddbeb:

+    /files/file4.txt
-    /notes/note1.txt
-    /notes/note2.txt
-    /notes/note3.txt

Files:           1 new,     3 removed,     0 changed
Dirs:            0 new,     0 removed
Others:          0 new,     0 removed
Data Blobs:      1 new,     0 removed
Tree Blobs:      2 new,     2 removed
  Added:   35.501 KiB
  Removed: 1.635 KiB
```

### Compare Second Snapshot with First Snapshot
If we use the `restic diff` command again, but reverse the order of the snapshot IDs, we get a different perspective on the difference between the 2 snapshots.  The output tells us the inverse of what we saw above.  When comparing the second snapshot to the first snapshot (reverse chronological order), we can see that 1 file is removed (35.501 KB) and 3 files are added (1.635 KB).

```
deep@ubuntu-vm:~$ restic diff 9d5ddbeb 93591a06 -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
comparing snapshot 9d5ddbeb to 93591a06:

-    /files/file4.txt
+    /notes/note1.txt
+    /notes/note2.txt
+    /notes/note3.txt

Files:           3 new,     1 removed,     0 changed
Dirs:            0 new,     0 removed
Others:          0 new,     0 removed
Data Blobs:      0 new,     1 removed
Tree Blobs:      2 new,     2 removed
  Added:   1.635 KiB
  Removed: 35.501 KiB
```
___
## Recover Data from Snapshots
There are 2 general methods in which data can be restored from snapshots within a repository.  The first method is a restore of the data from a snapshot to a target directory.  The second method is what makes Restic simply amazing: mounting the respository to a mount-point and directly accessing the data from within the snapshot, itself!

### Restore from a Repository
Let's create a new directory called `restore` which will serve as the target location for the restoration of the data.    

```
deep@ubuntu-vm:~$ mkdir restore
```

The next step is to perform `restic restore` using the snapshot ID of the desired snapshot (ID = 9d5ddbeb) and pointing the output to the newly created directory using `--target restore`.
```
deep@ubuntu-vm:~$ restic restore 9d5ddbeb --target restore -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
restoring <Snapshot 9d5ddbeb of [/home/deep/files /home/deep/notes] at 2020-12-07 22:45:31.34307187 +0000 UTC by deep@ubuntu-vm> to restore
```

The keyword `latest` can also be used in-lieu of the snapshot ID if the restoration from the most recent snapshot is desired.

```
deep@ubuntu-vm:~$ restic restore latest --target restore -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
restoring <Snapshot 9d5ddbeb of [/home/deep/files /home/deep/notes] at 2020-12-07 22:45:31.34307187 +0000 UTC by deep@ubuntu-vm> to restore
```

We can now navigate to the `restore` directory and view the contents.  The output indicates that all files from the most recent snapshot are indeed intact and available.  If needed, we can now copy/restore any of the files from this directory as needed.

```
deep@ubuntu-vm:~$ cd restore

deep@ubuntu-vm:~/restore$ ls -l *
files:
total 36
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file1.txt
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file2.txt
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file3.txt
-rw-rw-r-- 1 deep deep 35149 Dec  7 22:39 file4.txt

notes:
total 0
```

### Mount a Repository
Now let's get to one of my favorite features of Restic: mounting a repository.  In order to do so, we will first need a mount-point where we can mount the repository; the `/mnt/restic` mount-point can be created and used for this purpose.  

```
deep@ubuntu-vm:~$ sudo mkdir /mnt/restic
```

A repository can be mounted to this mount-point using the `restic-mount` command.  The `--allow-other` parameter will be used in this example since it ensures that others can access the repository as well.  Once this command is executed in the foreground, the prompt is left in a hanging state.  For the purposes of this example, verification of the mount will be performed using a second SSH session against the host machine: ubuntu-vm.
```
deep@ubuntu-vm:~$ sudo restic mount --allow-other /mnt/restic -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
Now serving the repository at /mnt/restic
When finished, quit with Ctrl-c or umount the mountpoint.
```

From within a second SSH session, we can list the contents of the `/mnt/restic` directory. 

```
deep@ubuntu-vm:~$ ls -l /mnt/restic
total 0
dr-xr-xr-x 1 root root 0 Dec  7 23:53 hosts
dr-xr-xr-x 1 root root 0 Dec  7 23:53 ids
dr-xr-xr-x 1 root root 0 Dec  7 23:53 snapshots
dr-xr-xr-x 1 root root 0 Dec  7 23:53 tags
```

The `snapshots` directory contains all of the available snapshots.  A symlink has been created with the name "latest" and is pointed to the most recent snapshot.

```
deep@ubuntu-vm:~$ ls -l /mnt/restic/snapshots
total 0
dr-xr-xr-x 4 root root 0 Dec  7 21:40 2020-12-07T21:40:01Z
dr-xr-xr-x 4 root root 0 Dec  7 22:45 2020-12-07T22:45:31Z
lrwxrwxrwx 1 root root 0 Dec  7 22:45 latest -> 2020-12-07T22:45:31Z
```

We can navigate to this directory and list all of the contents.  The output indicates that all files from the most recent snapshot are indeed intact and available.  If needed, we can now copy/restore any of the files from this mounted repository as needed.

```
deep@ubuntu-vm:~$ cd /mnt/restic/snapshots/latest

deep@ubuntu-vm:/mnt/restic/snapshots/latest$ ls -l *
files:
total 36
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file1.txt
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file2.txt
-rw-rw-r-- 1 deep deep     0 Dec  7 19:00 file3.txt
-rw-rw-r-- 1 deep deep 35149 Dec  7 22:39 file4.txt

notes:
total 0
```

Once complete, we can go back to the session where the repository was initially mounted and press `Ctrl + c` to unmount the repository.

```
...
Now serving the repository at /mnt/restic
When finished, quit with Ctrl-c or umount the mountpoint.
  signal interrupt received, cleaning up

deep@ubuntu-vm:~$
```
___
## Delete Snapshots
Although Restic uses deduplication and progressive snapshots which are incremental in nature to keep the size of backed up data minimal, it is still good practice to remove any unneeded snapshots on a routine basis.  The proper deletion of snapshots is a 2 part process: forgetting the snapshot and pruning the repository.

Before we do any sory of deletion, let's have a look at the existing snapshots using the `restic snapshots` command to obtain some information about each one.  It would be advantageous to keep the latest snapshot (ID = 9d5ddbeb) and delete the first snapshot (ID = 93591a06).

```
deep@ubuntu-vm:~$ restic snapshots -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
ID        Time                 Host        Tags        Paths
-----------------------------------------------------------------------
93591a06  2020-12-07 21:40:01  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes

9d5ddbeb  2020-12-07 22:45:31  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes
-----------------------------------------------------------------------
2 snapshots
```

### Forget the Snapshot 
Forgetting a snapshot only deletes the snapshot object from the repository, but does not delete the associated data.  This makes sense.  Think of it this way: when you forget about someone or something, it doesn't mean that that person or thing ceases to exist and is wiped off of the face of the planet.  The data which the snapshot used to reference still exists, but is no longer being referenced by a snapshot.  The `restic forget` command can be used to forget an existing snapshot.

```
deep@ubuntu-vm:~$ restic forget 93591a06 -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
removed snapshot 93591a06
```

### Prune the Repository
Now we can wipe that data off the face of the planet (well, at least off of the storage space).  That unreferenced, orphaned data from the previous command can and should now be pruned usng the `restic prune` command.

```
deep@ubuntu-vm:~$ restic prune -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
counting files in repo
building new index for repo
[0:00] 100.00%  3 / 3 packs
repository contains 3 packs (7 blobs) with 39.088 KiB
processed 7 blobs: 0 duplicate blobs, 0 B duplicate
load all snapshots
find data that is still in use for 1 snapshots
[0:00] 100.00%  1 / 1 snapshots
found 4 of 7 data blobs still in use, removing 3 blobs
will remove 0 invalid files
will delete 1 packs and rewrite 0 packs, this frees 2.416 KiB
counting files in repo
[0:00] 100.00%  2 / 2 packs
finding old index files
saved new indexes as [9794a016]
remove 1 old index files
[0:00] 100.00%  1 / 1 packs deleted
done
```

### Confirm the Forget and Prune
Let's have one last look at the repository with the `restic snapshots` command to see how things look.  We should just see 1 snapshot in existence within our repository.

```
deep@ubuntu-vm:~$ restic snapshots -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
ID        Time                 Host        Tags        Paths
-----------------------------------------------------------------------
9d5ddbeb  2020-12-07 22:45:31  ubuntu-vm               /home/deep/files
                                                       /home/deep/notes
-----------------------------------------------------------------------
1 snapshots
```
___
## Troubleshooting - Unlock a Repository
On rare occassions, you may encounter a message after attempting to execute a Restic command against a repository which indicates that the repository is locked by another process.  If this lock is indeed a remnant of a previous operation and is no longer needed, it can be removed and the repository can be freed up again using the `restic unlock` command.

```
deep@ubuntu-vm:~$ restic forget -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
Fatal: unable to create lock in backend: repository is already locked by PID 2757 on ubuntu-vm by deep (UID 1000, GID 1000)
lock was created at 2020-12-07 23:39:03 (17h8m9.288079923s ago)
storage ID 1bf13395

deep@ubuntu-vm:~$ restic unlock -r repo
enter password for repository:
repository 7bfcc5b4 opened successfully, password is correct
successfully removed locks
```
___
## Summary
Reliable backups can be a misunderstood concept.  Commands and tools such as `cp`, `rsync`, and `unison` are excellent for moving data from one location to the other and even for synchronizing data.  However, they are not ideal for a differential or incremental backup strategy.  They are also not efficient in terms of resource usage.  A good backup solution not only performs a full backup, but also examines the objects to see what has changed.  It accounts for these changes and saves them accordingly with emphasis on security and efficiency.  When it comes to restoration, a viable backup solution should allow for the bulk and granular restoration of files, directories, and objects based on a point-in-time.  Restic checks all of the boxes when it comes to a solid backup solution.  I use Restic at home and at work.  On numerous occassions, it has saved me from what could otherwise have been unfavorable or unfortunate situations.  This guide should now have put you in a good position to implement Restic and enjoy the security and benefits that comes with it.  With some basic scripting and perhaps a proper cron job or two, a powerful backup strategy is in your near future.