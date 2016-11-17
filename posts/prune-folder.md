<!-- 
.. title: Prune folder
.. slug: prune-folder
.. date: 2016-11-17 10:24:38 UTC+01:00
.. tags: scripts 
.. category: Bash
.. link: 
.. description: 
.. type: text
-->

## Prune / clear folder with a strategy

Sometimes there is a typical task: automated builds are copied to a folder every hour. We need to schedule a task to remove extra files from time to time.
But it's still good to do it with custom strategy. It can be simple: if file is older then N days, then keep only one last file per day, and if file is older than M days where M > N, then keep only one last file per month.
I didn't find any similar script in the internet, so made one myself. It can be modified to achieve slightly different strategies when needed.

This scripts works if all the files or folders named following the format: 
```
text1-yyyy-mm-dd-text2
```
where *text1* and *text2* can be anything not starting with digits, and *text1* is the same for all the files in the folder. The *yyyy-mm-dd* is the build date in the file name. The date must be the part of the file name, 
because sorting by date of creation is dangerous - it's easy to change the date by doing something with files on the server. 

Read the beginning of the script for the usage example: [prune.sh](https://github.com/pleasenophp/scripts/blob/master/prune.sh)


