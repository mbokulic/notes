# servers

## nginx with uwsgi
I did not manage to run these two together. It's very confusing and there is a lot of material on the internet. I did manage to run wsgi alone

https://www.digitalocean.com/community/tutorials/how-to-deploy-python-wsgi-applications-using-uwsgi-web-server-with-nginx
http://code-maven.com/deploying-python-flask-using-uwsgi-on-ubuntu-14-04

## ssh
You put your public key into the username:.ssh/authorized_keys file of the remote server. If so, you can log in the server using username@server_ip. You should set up permissions on the server to
```bash
chmod 700 .ssh
chmod 700 .ssh/autorized_keys
```

https://help.ubuntu.com/community/SSH/OpenSSH/Keys

# build tools

## npm
https://medium.com/@dabit3/introduction-to-using-npm-as-a-build-tool-b41076f488b0#.mzd1ghae5

## handlebars
here is how you iterate, use `{{../attr}}` to get to the attribute that is in the parent of the current context
http://handlebarsjs.com/builtin_helpers.html

## browserify
Use require('module_name')  to load modules like in Node.js. Browserify works by preloading modules at build and bundling everything (code + dependencies) in one file.

https://scotch.io/tutorials/getting-started-with-browserify

## learn watch!!!
This tutorial on browserify has a part about watchify

https://scotch.io/tutorials/getting-started-with-browserify

# venv
https://www.sitepoint.com/virtual-environments-python-made-easy
http://docs.python-guide.org/en/latest/dev/virtualenvs

# linux

## tmux
http://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/
https://pragprog.com/book/bhtmux/tmux
http://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/

## i3
short intro and other useful videos: https://www.youtube.com/watch?v=_kjbj-Ez1vU

short intro to config: http://blog.tunnelshade.in/2014/05/making-i3-beautiful.html

## dotfiles
use dothbot

http://www.anishathalye.com/2014/08/03/managing-your-dotfiles/

for sublime
https://chrisarcand.com/sublime-text-settings-and-dotfiles/

## installing beamer themes
https://tex.stackexchange.com/questions/199476/beamer-themes-cannot-be-found-ubuntu-ive-tried-every-folder-one-can-imagine

## tig
useful tool for Git

## pass
encrypts passwords using gpg

https://www.passwordstore.org/

## dmenu
comes with i3, can be scripted3

## irssi
IRC chat client

## jaromail
https://github.com/dyne/JaroMail

# UI

## touch events: 300ms delay
Vanilla: use touchend, but this will mean that event will fire even when you move out of the target

I don't like that it's only on tap
 - https://www.npmjs.com/package/fastclick
 - http://stackoverflow.com/questions/27173272/300ms-delay-removal-using-fastclick-js-vs-using-ontouchstart
 - https://github.com/filamentgroup/tappy

## to jquery or not
// https://toddmotto.com/is-it-time-to-drop-jquery-essentials-to-learning-javascript-from-a-jquery-background/
// http://youmightnotneedjquery.com
// use an SVG library or vanilla JS to manipulate SVG style, not CSS + jQuery

# text editor
[emmet](http://www.hongkiat.com/blog/html-css-faster-emmet/)

# visualization

## word cloud on the net
http://www.wordclouds.com/

# misc
facebook chat client for Python (not official, take care!)
https://github.com/Schmavery/facebook-chat-api#projects-using-this-api



