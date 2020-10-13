# vfx_notes
Notes on various shells, shell commands, and programs used in VFX production.

## shell

### common tools

#### cat
Read and print a file to the shell.

Print the contents of file.txt:
```
cat file.txt
```

For more info:
```
man cat
```


#### chmod
Used to change file permissions. Useful for limiting or sharing access to files.

Set file to execute/read/write access to user only:
```
chmod 700 /path/to/file
```

Set file to read/write access to read/write for everyone:
```
chmod 666 /path/to/file
```

To change all the directories to 755 (drwxr-xr-x):
```
find /path/to/file -type d -exec chmod 755 {} \;
```

To change all the files to 644 (-rw-r--r--):
```
find /path/to/file -type f -exec chmod 644 {} \;
```

For more info:
```
man chmod
```


#### cp
Copy a file to a new location or a new name.

Make a copy of an existing file in the same directory:
```
cp /path/to/file.txt /path/to/newfile.txt
```

Copy a file to a different directory:
```
cp /path/to/file.txt /new/path/to/newfile.txt
```

Copy an entire directory and it's contents to a different directory:
```
cp -r /path/to/dir /path/to/newdir
```

For more info:
```
man cp
```


#### find
Search for files or directories in a filesystem.

Find a file named "demo_reel.mov":
```
find /path/to/search/ -name "demo_reel.mov"
```

Find a file with a .mov extension modified in last 2 days:
```
find /path/to/search/ -mtime -2 -name "*.mov"
```

Load every QT found into a single RV session:
```
find /path/to/search/ -mtime -2 -name "*.mov" -exec rv {} +
```

For more info:
```
man find
```


#### grep
File pattern search tool.

Find every line in a file that contains the text "file" in it:
```
grep 'file' /path/to/file.nk
```

Find every line in every log file in a directory that contains the text "render" in it:
```
grep -r 'render' /path/to/files/*.log
```

Find the path to every file that contains the text "compositing":
```
grep -rl 'compositing' /path/to/files/*.log
```

For more info:
```
man grep
```


#### mv
Move files or directories.

Move file to new location:
```
mv /source/path/file.txt /dest/path/file.txt
```

Move folder to new location:
```
mv /source/path/dir /dest/path/dir
```

Rename a file:
```
mv /source/path/file.txt /source/path/new_file.txt
```

For more info:
```
man mv
```


#### rm
Delete files or directories. Be careful, there is not an "undo".

Delete file:
```
rm /path/to/file.txt
```

Delete directory and contents:
```
rm -r /path/to/dir
```

For more info:
```
man rm
```


#### rsync
Copy files from or to a local or remote system.
Transfers only the differences in files such as a new file or a change in a file. This is very useful for creating backup systems.

Sync a folder and all files and sub-directories from one location to another:
```
rsync --recursive /path/to/dir/ /path/to/backup/dir/
```

A day of work has gone by. Let's sync again but only the new or changed files:
```
rsync --recursive --ignore-existing /path/to/dir/ /path/to/backup/dir/
```

Another day has gone by. Let's run a test to see which files would be transferred:
```
rsync --recursive --ignore-existing --dry-run /path/to/dir/ /path/to/backup/dir/
```

Transfer files to a remote server:
```
rsync --recursive --ignore-existing /path/to/dir/ user@remote:/path/to/backup/dir/
```

For more info:
```
man rsync
```


#### ssh
Used for securely logging into remote machines.

Log into remote machine with user "worker":
```
ssh worker@remote
```

Use a specific identity key to log into a remote machine:
```
ssh -i ~/.ssh/remote_key.pem worker@remote
```

SSH can use a config file to set commonly used configurations.
First make the file:
```
touch ~/.ssh/config
```

To set our configuration in the file, add this to ~/.ssh/config:
```
Host remote
	HostName remote
	Port 22
	User pi
	IdentityFile ~/.ssh/remote_key.pem
```

Now we can login to the remote server with a shorter command:
```
ssh remote
```


#### tar
Used to put data into archive files for security or file transfer.

To compress:
```
tar -zcvf archive_name.tar.gz folder_to_compress
```

To extract:
```
tar -zxvf archive_name.tar.gz
```


### bash
#### Environment Variables
Setting a basic environment variable:
```
export TEST_VAR="/some/path"
```

Running a command to set a variable.
In this case, the current directory:
```
export CURDIR=$(pwd)
```


#### Aliases
```
alias ls_all='ls -l'
```


### tcsh
#### Environment Variables
```
setenv TEST_VAR "/some/path"
```

#### Aliases
```
alias ls_all 'ls -l'
```

## nuke

### python
#### Shotgun
All examples using Shotgun's demo projects

Find all project names
```python
sg.find('Project',[],['name'])
```

Find info on a specific shot
```python
shot = 'bunny_150_0200'
sg.find('Shot', [['project', 'is', {'type': 'Project', 'id': 70}], ['code', 'is', shot]], [ 'code', 'sg_status_list', 'tags', 'sg_sequence'])
```

Get all shots in a specific sequence
```python
sg.find('Shot', [['project', 'is', {'type': 'Project', 'id': 70}], ['sg_sequence', 'is', {'type': 'Sequence', 'id': 37, 'name': 'bunny_150'}]], ['code', 'sg_sequence', 'tags'])
```

### tcl