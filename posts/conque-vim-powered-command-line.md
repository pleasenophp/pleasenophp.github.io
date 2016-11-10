<!-- 
.. title: Conque: vim-powered command line
.. slug: conque-vim-powered-command-line
.. date: 2016-11-07 21:06:29 UTC+01:00
.. tags: cli, vim, conque 
.. category: Make it easy
.. link: 
.. description: 
.. type: text
-->

I'm doing most actions in the command line. Recently I enabled vim mode in bash, which helped me to easily navigate in the command I'm typing. However, I was quite frustrated that whenever I need to select something from the console output, I need to use mouse. I would like to be able to use Vim for selecting the previous text in the console and pasting it into the command. And here Conque comes to help! It's a plugin that allows to run shell inside of vim, with full output into the current buffer.


Install it from [here](https://code.google.com/archive/p/conque/downloads).

After it, run vim, and you can test it works with the following command:
```vim
:ConqueTerm bash
```

But of course we can improve it a little with a script. Add the following to the .bash_profile:
```bash
alias vimc='vim -c "ConqueTerm bash" -c "set nu"'
```

This will enable the **vimc** alias that will just start vim in conque mode, and will also enable the line numbers (even if you put __set nu__ into your .vimrc, Conque ignores it for some reason).

As I'm on Mac, I also had to create a symlink to my .bash_profile, as Conque reads .bashrc by default.
Run this from the home folder:
```
$ ln -s .bash_profile .bashrc
```

Also you might notice that Conque doesn't display the system user and folder name in the command line. For this just do the following trick (while still in the regular console):

```bash
$ echo "export PS1='$PS1'" >> ~/.bash_profile
```
This will add your regular settings of the shell prompt to the bash initialization script, and Conque will read it.

Voilà! Now just run:
```bash
$ vimc
```

And you can have a lot of fun with command line and your favorite vim! Just better don't try to run vim inside vim, looks a bit ugly :-P
![OMG](/images/vim-in-vim.jpg "WTF??")
