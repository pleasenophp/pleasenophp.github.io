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

## Prine / clear folder with a strategy

Sometimes there is a typical task: automated builds are copied to a folder every hour. We need to schedule a task to remove extra files from time to time.
But it's still good to do it with custom strategy. It can be simple: if file is older then N days, then keep only one last file per day, and if file is older than M days where M > N, then keep only one last file per month.
I didn't find any similar script in the internet, so made one myself. It can be modified to achieve slightly different strategies when needed.

Read the beginning of the script for the usage example: [prune.sh](https://github.com/pleasenophp/scripts/blob/master/prune.sh)


