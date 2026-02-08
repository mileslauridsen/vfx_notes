# vfx_notes
Notes on various shells, shell commands, programs, and resources used in VFX production.

## Table of Contents
- [Shell](#shell)
  - [Common Tools](#common-tools)
  - [Bash](#bash)
  - [Tcsh](#tcsh)
- [Nuke](#nuke)
  - [Python (Nuke)](#python-nuke)
  - [Tcl (Nuke)](#tcl-nuke)
- [Python](#python)
  - [Shotgun](#shotgun)
- [Formulas](#formulas)
  - [Focal Length](#focal-length)
  - [Canon EXIF data](#canon-exif-data)
- [Resources](#resources)
  - [Color](#color)
  - [Delivery](#delivery)
  - [IT](#it)
  - [Lighting and Lookdev](#lighting-and-lookdev)
  - [Math](#math)
  - [Open Source](#open-source)
  - [Optics and Cameras](#optics-and-cameras)
  - [Pipeline](#pipeline)
  - [Python (Resources)](#python-resources)
  - [Software](#software)
  - [Virtualization](#virtualization)

<a id="shell"></a>
## Shell
<a id="common-tools"></a>
### Common Tools
#### cat
Read and print a file to the shell.

Print the contents of file.txt:
```commandline
cat file.txt
```

For more info:
```commandline
man cat
```


#### chmod
Used to change file permissions. Useful for limiting or sharing access to files.

**Quick safety note:** prefer least-privilege permissions first, then relax only if needed.

Set file to execute/read/write access to user only:
```commandline
chmod 700 /path/to/file
```

Set file to read/write for user and group (safer team-share default):
```commandline
chmod 660 /path/to/file
```

Set file to read/write for everyone (rare; potentially risky):
```commandline
chmod 666 /path/to/file
```

When to use / when not to use:
- Use `660` when files are shared inside a trusted group and other users should have no access.
- Avoid `666` for scripts, configs, or production assets; world-writable files can be modified by any local user.

To change all the directories to 755 (drwxr-xr-x):
```commandline
find /path/to/file -type d -exec chmod 755 {} \;
```

To change all the files to 644 (-rw-r--r--):
```commandline
find /path/to/file -type f -exec chmod 644 {} \;
```

For more info:
```commandline
man chmod
```


#### cp
Copy a file to a new location or a new name.

Make a copy of an existing file in the same directory:
```commandline
cp /path/to/file.txt /path/to/newfile.txt
```

Copy a file to a different directory:
```commandline
cp /path/to/file.txt /new/path/to/newfile.txt
```

Copy an entire directory and it's contents to a different directory:
```commandline
cp -r /path/to/dir /path/to/newdir
```

For more info:
```commandline
man cp
```


#### ffmpeg
ffmpeg is a video and audio encoder commonly used in vfx

A number of useful examples of generating videos for a vfx pipeline are available here:
https://trac.ffmpeg.org/wiki/Encode/VFX


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

**Quick compatibility note:** `grep -r` works widely, but `rg` (ripgrep) is usually faster for large trees and respects ignore files.

Find every line in a file that contains the text "file" in it:
```commandline
grep 'file' /path/to/file.nk
```

Find every line in every log file in a directory that contains the text "render" in it:
```commandline
grep -r --include='*.log' 'render' /path/to/files/
```

Find the path to every file that contains the text "compositing":
```commandline
grep -rl --include='*.log' 'compositing' /path/to/files/
```

Equivalent with ripgrep (recommended for large projects):
```commandline
rg -n 'render' /path/to/files/
rg -l 'compositing' /path/to/files/
```

When to use / when not to use:
- Use recursive search on scoped paths (show/shot/task folders) to avoid accidental full-filesystem scans.
- Avoid running broad recursive searches from `/` or very large mount points unless you intentionally need that scope.

For more info:
```commandline
man grep
```


#### mv
Move files or directories.

Move file to new location:
```commandline
mv /source/path/file.txt /dest/path/file.txt
```

Move folder to new location:
```commandline
mv /source/path/dir /dest/path/dir
```

Rename a file:
```commandline
mv /source/path/file.txt /source/path/new_file.txt
```

For more info:
```commandline
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


<a id="bash"></a>
### Bash
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


<a id="tcsh"></a>
### Tcsh
#### Environment Variables
```
setenv TEST_VAR "/some/path"
```

#### Aliases
```
alias ls_all 'ls -l'
```


<a id="nuke"></a>
## Nuke
<a id="python-nuke"></a>
### Python

Example of iterating through selected nodes
```python
for n in nuke.selectedNodes():
    print(n.name())
```

Same example but only using a specific node class
```python
for n in nuke.selectedNodes("Camera2"):
    print(n.name())
```

Get current script path
```python
print nuke.root().name()
```

Get selected node class
```python
print nuke.selectedNode().Class()
```

Looking through all nodes
```python
for n in nuke.allNodes():
    if n.Class() == "Read":
        print(n.name())
```

Select a node by name
```python
nuke.toNode("Noise1")
```

Running tcl commands in python
```python
nuke.tcl("value label")
```

Setting defaults in menu.py or init.py. http://docs.thefoundry.co.uk/nuke/63/pythondevguide/basics.html
```python
nuke.knobDefault("Blur.size", "20")
nuke.knobDefault("Read.exr.compression", "2")
```

Return an expression on a knob if it eists
```python
if knob.hasExpression():
    origExpression = knob.toScript()
```

List all knobs on a node
```python
for n in nuke.selectedNodes().knobs():
    print(n)
```

When to use / when not to use:
- Use `print(...)` syntax for Python 3 compatibility (current Nuke versions are Python 3-based).
- Avoid Python 2 `print` statements (`print x`) unless maintaining legacy scripts on older Nuke/Python 2 environments.

Set value on all Read nodes within a selected group
```python
for n in nuke.selectedNodes():
    if n.Class() == "Group":
        for s in n.nodes():
            if s.Class() == "Read":
                s['postage_stamp'].setValue(True)
```

<a id="tcl-nuke"></a>
### Tcl
Useful expressions and tcl for Nuke

On/Off in GUI mode
```
$gui
$gui ? 1 : 0
$gui ? 1 : 16 #scanline render
```

Use python wrapped in tcl
```
[python nuke.tcl("value gain")]
```

Check node directly above this one and return some values for a knob:
```
[value [value input[value 0].name].distortion2] + [value [value input[value 0].name].distortion1]  > 0 ? 1:0
```

Nuke Random Range between max and min values
```
(1.1 - .9) * random(t) + .9
```

Random curve expression
```
(random(1,frame*1)*1)+0
```

or just simplified:
```
random(1,frame)
```

This creates a curve containing random values between 0 and 1. The breakdown controls:
```
(random(seed,frame*frequency)*amplitude)+valueOffset
```

Simple html for label knobs
```html
<font color="red">Some red text</font> # change font color
<font color="blue">Some blue text</font> # a different font color
<font align="left">Left aligned text
<b>Bold text</b> # bold text
```

Metadata get exr compression type
```
[metadata exr/compression]
```

Root dir
```
[file dirname [knob [topnode].file]]
```

Filename
```
[file tail [knob [topnode].file]]
```

Value of this node's first knob
```
[value this.first]
```

Value of the first input's first knob
```
[value this.input0.first]
```

Value of the second input's first knob
```
[value this.input1.first]
```

Value of top most node in chain's first knob
```
[value [topnode].first]
```

Retime a Camera with a Timewarp node named TimeWarp1.
Useful when matchmove tracks the plate and then a retime is applied to match editorial.
Place this expression on translate, rotate, and focal knobs
```
curve(TimeWarp1.lookup)
```

Useful for determining if sharpening is needed on retimes
Example with a Timewarp node named Timewarp1
```
abs(rint(TimeWarp1.lookup)-TimeWarp1.lookup) > 0.15 ? 1:0
```

Using TCL lindex and split to get a specific portion of a file path
In this case we split the directory separator "/" and choose the 3rd item in the resulting list
```
[ lindex [split [value [topnode].file] / ] 3 ]
```

Normalize
```
(value - valuesMin)/(ValuesMax - valuesMin)
```


<a id="python"></a>
## Python
<a id="shotgun"></a>
### Shotgun
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


<a id="formulas"></a>
## Formulas
<a id="focal-length"></a>
### Focal Length
```math
​2 * atan(hap / (2 * focal)) * 180 / pi degrees( ​2 * atan(hap / (2 * focal)))
```

<a id="canon-exif-data"></a>
### Canon EXIF data
Shutter
```
1 / (2^value)
```

```
1 / (2^"6.32193") = 0.0125
```

Aperture
```
sqrt(2^value)
```

```
sqrt(2^8) = 16
```

<a id="resources"></a>
## Resources
<a id="color"></a>
### Color
https://nick-shaw.github.io/cinematiccolor/cinematic-color.html#
http://brucelindbloom.com/
https://opencolorio.org/
https://github.com/AcademySoftwareFoundation/OpenColorIO
https://github.com/AcademySoftwareFoundation/OpenColorIO-Config-ACES
https://github.com/AcademySoftwareFoundation/openexr-images
https://www.colour-science.org/
https://github.com/colour-science
https://github.com/colour-science/OpenColorIO-Configs
http://mikeboers.com/blog/2013/11/07/linear-raw-conversions
https://nick-shaw.github.io/cinematiccolor/full-and-legal-ranges.html
https://chrisbrejon.com/cg-cinematography/
https://acescentral.com/
https://www.babelcolor.com/colorchecker.htm

<a id="delivery"></a>
### Delivery
https://opencontent.netflix.com/

<a id="it"></a>
### IT
https://docs.ansible.com/ansible/latest/

<a id="lighting-and-lookdev"></a>
### Lighting and Lookdev
https://lollypopman.com/lighting-club/
https://academy.substance3d.com/courses/the-pbr-guide-part-1

<a id="math"></a>
### Math
https://en.wikipedia.org/wiki/Spherical_coordinate_system
https://en.wikipedia.org/wiki/Rotation_matrix
https://en.wikipedia.org/wiki/Euclidean_distance
https://en.wikipedia.org/wiki/Feature_scaling
https://en.wikipedia.org/wiki/Quaternion
https://en.wikipedia.org/wiki/Transformation_matrix
https://en.wikipedia.org/wiki/Sawtooth_wave
https://ocw.mit.edu/courses/mathematics/18-06-linear-algebra-spring-2010/video-lectures/

<a id="open-source"></a>
### Open Source
https://github.com/FFmpeg/FFmpeg
https://github.com/OpenImageIO/oiio
https://github.com/AcademySoftwareFoundation
https://github.com/AcademySoftwareFoundation/openvdb
https://github.com/AcademySoftwareFoundation/openexr
https://github.com/NatronGitHub/Natron
https://github.com/alembic/alembic
https://www.blender.org/

<a id="optics-and-cameras"></a>
### Optics and Cameras
https://en.wikipedia.org/wiki/Circle_of_confusion
https://jtra.cz/stuff/essays/bokeh/#what_is_bokeh
https://vfxcamdb.com/


<a id="pipeline"></a>
### Pipeline
https://github.com/pyblish/pyblish-base
https://getavalon.github.io/2.0/

<a id="python-resources"></a>
### Python
https://docs.python-guide.org/

<a id="software"></a>
### Software
http://vfxplatform.com/

<a id="virtualization"></a>
### Virtualization
https://github.com/AcademySoftwareFoundation/aswf-docker
https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
https://www.codementor.io/@jquacinella/docker-and-docker-compose-for-local-development-and-small-deployments-ph4p434gb
