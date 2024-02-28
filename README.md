# Welcome

Welcome to the source code of my humble blog [sanjayts.net](https://sanjayts.net)!

### Git submodule commands

The commands we need to run to get the blog up and running:

    cd blog
    git submodule add https://github.com/sanjayts/hugo-theme-even themes/hugo-theme-even
    git submodule init
    git submodule update
    git submodule add -b master https://github.com/sanjayts/sanjayts.github.io public
    git add * && git commit -m "The msg"
    hugo
    cd public
    git add * && git commit -m "The msg"


In case you want to pull the latest code for submodules:

    git submodule update --recursive

Or if you want to do it for all the modules at once (assuming there are no local changes):

    git submodule foreach git pull
