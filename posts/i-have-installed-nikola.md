<!-- 
.. title: I have installed Nikola
.. slug: i-have-installed-nikola
.. date: 2016-11-06 08:42:39 UTC+01:00
.. tags: nikola, vim, cli
.. category: Make it easy
.. link: 
.. description: Making blogging life easier
.. type: text
-->

## I'm happy

I can now write all my blog posts in vim, using markdown, without distracting from the command line.
[Nikola](https://getnikola.com) rules! It allows you to store blog locally, commit it under git, and generate static html whenever you wish to deploy. You can use your favorite web hosting or github pages to host the blog. 
~~The only thing that frustrates me so far is the default styles of the code blocks, I wanna black background as in the terminal.~~
It appeared Nikola supports many code themes just in the configuration file! So I have set a good black one called "monokai" to start with (might change it later).

```python
CODE_COLOR_SCHEME = 'monokai'
```
in the *conf.py*

And my old blog on Blogger is still [here](https://pleasenophp.blogspot.dk). There is only a few posts.
  
---  
For those who already installed Nikola, there might be helpfull some scripts. I have added this to my .bash_profile:

```bash
alias nstart="cd ~/nikola && source bin/activate && source .rc"
```

This will enable nstart alias that automatically opens your nikola path, and activates the virtual environment. Then it reads my .rc file that I put under nikola folder:

```bash
eval "`nikola tabcompletion`"

alias nnew="nikola new_post -f markdown"
alias nbuild="nikola build"
alias nserve="nikola serve -b"
alias ndeploy="nikola github_deploy"

alias nstop="deactivate && cd ~"
```

It adds some handy short aliases for build, serve and deploy, and the first line also enables the tabcompletion.
When I finish working in the Nikola virtual environment I type nstop.

And ofc you should enable markdown in Nikola, because by default is uses somethin else _(maybe what it uses is better, I didn't check, but I didn't either feel like learning one more not so popular markup standard)._

To enable markdown, just add .md extension in POSTS and PAGES in *conf.py* like this:
```python
POSTS = (
    ("posts/*.rst", "posts", "post.tmpl"),
    ("posts/*.txt", "posts", "post.tmpl"),
    ("posts/*.html", "posts", "post.tmpl"),
    ("posts/*.md", "posts", "post.tmpl")
)
PAGES = (
    ("pages/*.rst", "pages", "story.tmpl"),
    ("pages/*.txt", "pages", "story.tmpl"),
    ("pages/*.html", "pages", "story.tmpl"),
    ("pages/*.md", "pages", "story.tmpl")
)
```

