---
title: "Create tar.gz"
datePublished: Mon Oct 02 2023 07:57:21 GMT+0000 (Coordinated Universal Time)
cuid: cln8llg3j000109icge2i0lsq
slug: create-targz

---

## How to create tar.gz file in Linux using command line

The procedure to create a tar.gz file on Linux is as follows:

1. Open the terminal application in Linux
    
2. Run tar command to create an archived named file.tar.gz for given directory name by running: <kbd>tar -czvf file.tar.gz directory</kbd>
    
3. Verify tar.gz file using the ls command and tar command
    

Let us see all commands and options in details.

## Linux create tar.gz file with tar command

Say you want to create tar.gz file named projects.tar.gz for directory called $HOME/projects/:

`ls -l $HOME/projects/`  
Sample outputs from the ls command:

```bash
total 8
drwxr-xr-x 2 vivek vivek 4096 Oct 30 22:30 helloworld
drwxr-xr-x 4 vivek vivek 4096 Oct 30 13:16 myhellowebapp
```

The syntax for the tar command is as follows:  

`tar -czvf filename.tar.gz /path/to/dir1   tar -czvf filename.tar.gz /path/to/dir1 dir2 file1 file2   # Create a tar.gz file from all pdf (".pdf") files   tar -czvf archive.tgz *.pdf`

`tar -czvf projects.tar.gz $HOME/projects/`

![.cyberciti.biz/media/new/faq/2013/06/How-to-create-tar.gz-file-in-Linux-using-command-line.png](https://www.cyberciti.biz/media/new/faq/2013/06/How-to-create-tar.gz-file-in-Linux-using-command-line.png align="left")

### Verification

Next verify newly created tar.gz file in Linux using the ls command:  

`ls -l projects.tar.gz`  
Example outputs:

```bash
-rw-r--r-- 1 vivek vivek 59262 Nov  3 11:08 projects.tar.gz
```

One can [list the contents of a tar or tar.gz file](https://www.cyberciti.biz/faq/list-the-contents-of-a-tar-or-targz-file/) using the tar command:

`tar -ztvf projects.tar.gz`

![Linux list tar.gz file contents](https://www.cyberciti.biz/media/new/faq/2013/06/Linuxlist-tar.gz-file-contents-300x145.png align="left")

### How to extract a tar.gz compressed archive on Linux

The syntax is follows to extract tar.gz file on Linux operating system:  

`tar -xvf file.tar.gz   tar -xzvf file.tar.gz   tar -xzvf projects.tar.gz`

The syntax is follows to extract tar.gz file on Linux operating system:

`tar -xzvf projects.tar.gz -C /tmp/`  
Sample outputs:

```bash
home/vivek/projects/myhellowebapp/
home/vivek/projects/myhellowebapp/hellowebapp.working/
home/vivek/projects/myhellowebapp/hellowebapp.working/db.sqlite3
home/vivek/projects/myhellowebapp/hellowebapp.working/manage.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/models.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/admin.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/__init__.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/tests.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/static/
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/static/css/
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/static/css/style.css
...
..
...
home/vivek/projects/myhellowebapp/hellowebapp/hellowebapp/__pycache__/urls.cpython-36.pyc
home/vivek/projects/myhellowebapp/hellowebapp/hellowebapp/wsgi.py
home/vivek/projects/myhellowebapp/hellowebapp/hellowebapp/settings.py
home/vivek/projects/helloworld/
```

### Understanding tar command options

| tar command option | Description |
| --- | --- |
| <kbd>-c</kbd> | Create a new archive |
| <kbd>-x</kbd> | Extract files from an archive |
| <kbd>-t</kbd> | List the contents of an archive |
| <kbd>-v</kbd> | Verbose output |
| <kbd>-f file.tar.gz</kbd> | Use archive file |
| <kbd>-C DIR</kbd> | Change to DIR before performing any operations |
| <kbd>-z</kbd> | Filter the archive through gzip i.e. compress or decompress archive |

[https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/](https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/)