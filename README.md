# licb - Lxd Image & Container Builder
Is a python3 command line script made to easily build new lxc container and images.
It simply takes a text file with special commands, translates them to lxd (lxc) and executes them in a serial way.

## Why should I care?
The main advantages compared to a plain bash file with lxc commands are:
- the commands are stripped down to the necessary minimum
- several lxc commands can be combined to have one command for our use case
- the container and snapshot names can be generated automatically and reused implicitly (aka. keeping the internal state)

## A short Example
Of course an actual version of LXD has to be installed.  
### Prepare a command file  
Create a file "java8-commands" and paste following commands:
> FROM images:ubuntu/wily/amd64 AS ubuntu-wily-java8  
>
> \# Update system  
> EXEC apt-get update  
> EXEC apt-get upgrade -y  
>
> \# Install java via apt  
> EXEC apt-get install -y openjdk-8-jre-headless curl  
>
> \# Create snapshot and image  
> SNAPSHOT
> IMAGE wily/java8

### Call licb with path to command file  
In the easiest case they are both in the same folder:

`./licb java8-commands`  

This should:
- copy the remote image "ubuntu/wily/amd64" to your local repository
- create a container "ubuntu-wily-java8" from this image
- upgrade the container via apt-get update/upgrade
- install openjdk-8 and curl
- make a snapshot
- make a new image "wily/java8"

Now you have a container ready for further actions. You can always use the snapshot to go back or create new containers from the image.

## More details
### Motivation
This program is the result of me trying to get a grip on Python. This works better for me than typing another hello world. However, for you it probably means to not expect anything useful, fancy or even correct.

### Installation
Right now there is no installation method except copying the licb file.

### Manual
The command line usage and the commands in the command file are explained in the commands.md file.

### Dependencies
Licb was tested with

> $ python3 --version  
> Python 3.4.3+

and

> $ lxc --version  
> 2.0.0.rc1

### License
GPL version 2 or (at your option) any later version
