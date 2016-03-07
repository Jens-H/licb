# Command overview
## licb from command line
### Format
> $ ./licb -h
> usage: licb [-h] [-v] [-d] [-V] file
>
> positional arguments:
>   file           file to process
>
> optional arguments:  
>   -h, --help     show this help message and exit  
>   -v, --verbose  increase output verbosity  
>   -d, --debug    enable debug output  
>   -V, --version  show program's version number and exit  

### Description
-v isn't used much, so better use -d to get more information about what is going on.

The execution will stop if an error (exit status != 0) is returned to python3 subprocess.

### Examples
`./licb -d example-commands`  
Call licb with "example-commands" in the working directory. The debug output is turned on.

`./licb command-files/example-commands`  
Call licb with "example-commands" with relative paths.

`/home/klaus/lxd-stuff/licb /home/klaus/lxd-stuff/command-files/example-commands`
Call licb with "example-commands" with absolute paths.


## FROM
### Format
FROM [origin:]"image name" [AS "container-name"]

### Description
Creates a new 'working container' (named "container-name") from a local or remote image. This container is the basis for all following operations, therefore this command **MUST** be the first one.

If the origin is left or it is "local:", then the local image is used and the container started with `lxc launch`. Otherwise the remote image is copied via `lxc copy` before it is launched.

If no AS "container-name" is given, a random name is generated, that is derived from the image name. However, the names are not checked for collisions with existing containers.

### Examples
**FROM images:ubuntu/wily/amd64 AS ubuntu-wily**  
Copies the "ubuntu/wily/amd64" image from the lxc remote "images" and launches it as "ubuntu-wily".

**FROM local:my/image**  
Launches the local "my/image" with a random name.

**FROM local:my/image AS my-container**  
Launches the local "my/image" as "my-container".

## EXEC
### Format
EXEC command-to-execute

### Description
Executes the command in the container. The complete command including spaces is sent to `lxc exec` directly. There are several short comings that will lead to unexpected or wrong behavior:
- There is no escaping of the command done and therefore it is possible to execute commands directly on the host instead of inside the container. Have a look here for more details: https://en.wikipedia.org/wiki/Code_injection#Shell_injection
- Output/Input redirection (<,>) is not working and could lead to shell injections
- The command is always executed in the home directory. If it is necessary to execute a command inside a certain directory the sh command can be used: `EXEC sh -c '(cd myProgram/programs/server && npm install)'`
- It is obvious that commands normally should not ask for user interaction.

### Examples
**EXEC apt-get upgrade -y**  
Upgrades the container via apt-get without asking.

**EXEC sh -c '(cd myProgram/programs/server && npm install)'**  
A way to combine two commands into one. This should be the exception!

## FILE
### Format
FILE  "file-on-host" "/full/path/to/destination/folder/"

### Description
Copies the "file-on-host" into the container as "/full/path/to/destination/folder/file-on-host". The file on the host is **RELATIVE** to the folder of the licb command file. Inside the container the full path to the destination folder has to be supplied. The file name is copied unchanged. The oid and gid are changed to 0 (=root).  
**!!! No spaces are allowed in neither file nor path, even with surrounding ''!!!**
In case of emergency a workaround would be to copy the file without spaces and rename it afterwards with "EXEC mv ...".

### Examples
**FILE  myprogram.service /lib/systemd/system/**  
Copies the "myprogram.service" into the systemd folder "/lib/systemd/system/".

## SNAPSHOT
### Format
SNAPSHOT [snapshot-name]

### Description
Creates a snapshot of the container. The snapshot is "stateless" and therefore the container is stopped before the snapshot and restarted afterwards. If no "snapshot-name" is provided, the name is generated. The pattern is "s"+ running number, like s1, s2, ... .

### Examples
**SNAPSHOT after-apt-install**  
Creates a snapshot named "after-apt-install".

**SNAPSHOT**  
Creates a snapshot named (depending on number of calls) "s1".

## IMAGE
### Format
IMAGE image-name

### Description
Creates an local image via `lxc publish` from **the last SNAPSHOT**. The image name is set to "image-name".
As the last snapshot is the basis of the image, it is **strongly recommended** to use IMAGE immediately after the SNAPSHOT command. Otherwise all changes after the SNAPSHOT will not be included into the image!
If the image alias already exists, a new one is created and the image alias is only used for the new one.

### Example
**IMAGE wily/my/image**  
Creates a new image "wily/my/image" from the last snapshot.

## Comments
### Format
\# [this is the comment]

### Description
A line starting with "# " (= # + space) is ignored and can be used as a comment line.

### Example
\# Update container via apt-get
