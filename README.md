# blog
My blog source

### Git submodule commands

The commands we need to run to get the blog up and running:

    cd blog
    git submodule add https://github.com/MunifTanjim/minimo themes/minimo
    git submodule init
    git submodule update
    git submodule add -b master https://github.com/sanjayts/sanjayts.github.io public
    git add * && git commit -m "The msg"
    hugo
    cd public
    git add * && git commit -m "The msg"


In case you want to pull the latest code for submodules:

    git submodule update --recursive