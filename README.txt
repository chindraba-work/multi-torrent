multi-torrent

Contents
================================================================================

* Description
* Requirements
* Installation
  * Edit the install config file, config.sh
  * Execute the build script, build
  * Execute the install script, install with root access (for systemd control)
  * Execute the control script directly to add instances as needed
  * Execute the control script directly to enable systemd control of each instance
  * Manually launch the instances using the control script, system, or with a reboot
* Usage
  * Using the control script
  * Using systemd
  * Data file
  * Manual systemd changes
* TODO
* Versioning
* Copyright and License
  * The MIT License


Description
================================================================================

Setup and control a configuration for using multiple instances of the rtorrent
client. The clients are launched at boot by the systemd process (as opposed to
waiting for the user to log in). Each client is run in isolation from the rest,
having its own watch, download and session directories, and may be started,
stopped or paused without affecting the others. Each also will have its own
pair of ports, if the process it used correctly when creating new instances to
use. Each instance is run inside of a `screen` instance spawned for and
controlling only the one rtorrent instance. (rtorrent itself works better with
flow-control turned of in the terminal emulator, while the operations of this
controller will not differ either way.)


Requirements
================================================================================

Use of the multiple torrent setup requires, at a minimum:

* Bash, version 4 or better
* systemd operating on the system
* rtorrent, including its dependencies
* screen program to allow multiple screens on a VT100/ANSI Terminal
* Command-line access to the system
* Text editor, minimal use, for creating the original configuration
* Permission level sufficient for starting and stopping instances using systemd
* Root-level access for installing the systemd control files
* Knowledge of your Internet-facing and system-level IP addresses
  (rarely the same)
* Ability to open ports between the system and the Internet (computer firewall,
  router, etc.)
* Internet connection for torrent use


Installation
================================================================================

1. Edit the install config file, config.sh
2. Execute the build script, build
3. Execute the install script, install with root access (for systemd control)
4. Execute the control script directly to add instances as needed
5. Execute the control script directly to enable systemd control of each instance
6. Manually launch the instances using the control script, systemd, or with a
   reboot

* The config file

The config file in the project root is well commented. Additional considerations
include using "shell-safe" names for both the torrent root directory and the
service name. The scripts are built such that most unsafe names ought to be
safe, but there are bound to be oddities I'll never know about. The other parts
of making the system work, including using commands related to this and others
that are not included in my work are just easier to type if shell-safe names are
used.

The service_name is used for all the systemd components and for all associated
file names.

The torrent_root defines the directory where everything else will be built. The
path MUST be an absolute path and MUST end with a '/'. Of course, it also has to
be somewhere which the user has permission to create files. The default is to
create the directory in the user's home directory. That is not required, at all.
I prefer to use a directory off the root structure with a separate partition
mounted there. In my case torrent_root is /shares/.

The internal_ip and external_ip can be IPv4 or IPv6. Whether or not the local
install of rtorrent can handle IPv6 and how the trackers deal with IPv6 is
another issue. Unless you are sure the systems you use can deal with IPv6, or
you have no other choice, using IPv4 is probably the preferred option for now.
IPv6 has been around for a long time, and there is no real reason why IPv4 is
even still in use, but that's not something I can take care of here. This is
ready for IPv6 when, or if, the rest of the Internet is.

The port numbers used by rtorrent fall into two categories; communication with
peers for file sharing, and the DHT system for finding other peers. In the case
of rtorrent, the former can be in a range and/or randomized. This setup is built
to use a single assigned port for sharing, and second assigned port for DHT. Any
port range can be used and by default the system is intended to be in a 5-digit
range, and over the 32K value to avoid interference with other port usage. The
last 2 digits are in the range of 10 to 49 and the 50 to 89, with the range
starting at 10 being for file sharing and the range starting at 50 being for DHT
usage. The first 3 digits are assigned in the config file as port_range, which
should be less than 654, and over 327.

Initial install includes a 'default' instance and is assigned the values 10 and
50 for sharing and DHT respectively. If the port numbers, or name, of the
default instance needs to be changed it should be done in the build script
before execution.

* The build script

From within the project's root directory, executing the build script (./build),
as the user which will be "using" rtorrent will create the torrent root
directory and populate it with the modified template files and customized
torrent config files. The default instance, named 'default', and the control
files will be built. The control files are placed in the bin sub directory of
the torrent root and are ready for installation for systemd control.

As-is, other than systemd and boot-time launch, the system is operational at
this point, using the control file, named as the service name suffixed with ctl.
If the build script detects a bin directory in the user's home directory, a
symlink is placed there, with execute privilege and should be in the user's
PATH. With, or without, a user bin directory the control file is in the bin
sub directory of the torrent root. Using the control script is covered in the
Usage section below.

* The install script

From within the project's root directory, executing the install script
as root, (sudo ./install), will copy the needed files into
/usr/lib/systemd/system and enable the collective in systemd. If the base system
uses some other point for the systemd files, that will need to be adjusted in
the install script. After installing, the collective will be launched at boot,
and is able to be controlled as any other service. Using the default service
name, multi-torrent, the collective can be started with
"sudo systemctl start multi-torrent.target"
and a single instance, default or otherwise, can be controlled with commands
like "sudo systemctl restart mutli-torrent@default".`

As noted above, this step is optional. The system will operate manually just
fine without this.

* Add instances

The build script will create a default instance. Using more than one requires
adding more. If you only need one instance, you don't need this package. Adding
an instance is done with the "new" verb to the control script,
"multi-torrentctl new <instance-name> <peer-port> <dht-port>"
Only one instance can be added per command. Up to 38 additional instances can be
added, in any order. Adding more instances requires making a change to the port
assignment scheme. Instances added can be controlled with the control script,
even before being "enabled" for systemd control below. See Usage below.

* Enable instances

Instances added above are limited to manual control until they are "enabled" for
systemd control. If the "Install script" above is executed, any of the added
instances can be placed under systemd control. The command script can only
enable one instance per execution. The "enable" verb to the control script is
used for adding the instances to systemd control, see Usage below.

* Launch instances

Any single instance can be launched using the "start" verb to the control
script, or using systemd again with the "start" verb to `systemctl`. All
instances can be started manually using the "start" verb of systemctl against
the "target" of the service. All the instances placed under systemd control are
launched automatically at boot time. See Usage below. 


Usage
================================================================================

All the samples are based on using the default service name multi-torrent and
instances named "default" using ports 10 and 50, "linux-distros" using ports 12
and 52, and "music-circle" using ports 36 and 76. The "default" instance, of
course, is already fully operational if all the installation steps are followed.

Using the control script
--------------------------------------------------------------------------------

Show the version and copyright/license information:

    multi-torrent version

Getting help:

    multi-torrent help

Creating a new instance:

    multi-torrent new linux-distros 12 52

    multi-torrent create music-circle 36 76

Adding an instance to systemd control:

    multi-torrent enable linux-distros

Starting, the default instance:

    multi-torrent start default

Stopping the linux-distros instance:

    multi-torrent stop linux-distros

Restarting the linux-distros instance:

    multi-torrent restart linux-distros

Using systemd
--------------------------------------------------------------------------------

Starting all of the instances:

    sudo systemctl start multi-torrent.target

Stopping all of the instances:

    sudo systemctl stop multi-torrent.target

Restarting all of the instance:

    sudo systemctl restart multi-torrent.target

Starting the default instance:

    sudo systemctl start multi-torrent@default

Stopping the linux-distros instance:

    sudo systemctl stop multi-torrent@linux-distros

Restarting the linux-distros instance:

    sudo systemctl restart multi-torrent@linux-distros

Data file
--------------------------------------------------------------------------------

The created data file, `multi-torrent.data`, in the torent root directory is a
convenience file and is not edited or maintained by the scripts. It has the
information needed for manually editing the `.rtorrent.rc` files and helps to
track, or select, the port numbers to assign to new instances.

Manual `systemd` changes
--------------------------------------------------------------------------------

The service target file (/usr/lib/systemd/system/multi-torrent.target by
default) can be edited by hand to add or remove instances from those
automatically controlled by systemd. Especially important for removal, as
there is no mechanism for removing any of the instances created. Neither from
systemd nor from the disk.

Each instance is controlled by a line that begins with "Wants=". The first one
has the default instance, and any others are listed in the order added.
Disabling an instance can be done with a hash mark "#" as the first character of
the line. Deletion of the line removes the instance completely from systemd
control while leaving all the files on disk for it. Instances removed from the
file, or disabled by a comment mark, are still available for manual control with
the control script.


## TODO
================================================================================

Things I'd like to "make happen" with future editions:

*  Create a script to create a symlink chain from the torrent download space to
   other locations for final storage
*  Create a script to handle sorting new torrents from a staging area to a
   specific instance and watch directory
*  Create triggers and/or cron jobs to relocate/copy finished torrents to some
   other location based on why they were downloaded.

Versioning
================================================================================

multi-torrent uses Semantic Versioning v2.0.0.

Semantic Versioning <https://semver.org/spec/v2.0.0.html> was created by Tom
Preston-Werner <http://tom.preston-werner.com/>, inventor of Gravatars and
cofounder of GitHub.

Version numbers take the form X.Y.Z where X is the major version, Y is
the minor version and Z is the patch version. The meaning of the
different levels are:

* Major version increases indicates that there is some kind of change in
  the API (how this program works as seen by the user) or the program
  features which is incompatible with previous version

* Minor version increases indicates that there is some kind of change in
  the API (how this program works as seen by the user) or the program
  features which might be new, while still being compatible with all other
  versions of the same major version

* Patch version increases indicate that there is some internal change,
  bug fixes, changes in logic, or other internal changes which do not
  create any incompatible changes within the same major version, and which
  do not add any features to the program operations or functionality


Copyright and License
================================================================================

The MIT license applies to all the code within this repository.

Copyright © 2020  Chindraba (Ronald Lamoreaux)
            <projects@chindraba.work>
- All Rights Reserved

The MIT License

Permission is hereby granted, free of charge, to any person obtaining a
copy of this software and associated documentation files (the "Software"),
to deal in the Software without restriction, including without limitation
the rights to use, copy, modify, merge, publish, distribute, sublicense,
and/or sell copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGE MENT. IN NO EVENT SHALL
THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
DEALINGS IN THE SOFTWARE.
