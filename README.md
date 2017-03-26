This project contains a sample Juce project which should compile on Bela and call Juce functions from within the Bela API.


## Setup
* Update your board with the Bela code from the branch `dev-libbela` (see [here](https://github.com/BelaPlatform/Bela/wiki/Updating-Bela#getting-the-experimental-release) how to update the board).
* You then have to do the following in order to build the Bela library and make your dynamic linker aware of that:
```
make coreclean
make lib
ldconfig /root/Bela/lib
```
* Then you need to connect the board to the internet and install `libcurl`, which is - as of today - the only dependency for the core Juce module:
```
  apt-get install libcurl4-openssl-dev
```
if your board is not connected to the internet, then you can try to download the `.deb` package at the following address:
ftp.us.debian.org/debian/pool/main/c/curl/libcurl4-openssl-dev_7.38.0-4+deb8u5_armhf.deb 
then copy it to the board and install it with:
```
dpkg -i libcurl4-openssl-dev_7.38.0-4+deb8u5_armhf.deb
```
* *Note*: if you add more JUCE modules you may need to install more dependencies.
* *Note2*: in this project I disabled JUCE_USE_CURL, yet it is still a dependency. See [here](https://forum.juce.com/t/sigill-when-running-blocksdrawing-on-beaglebone-black/19779/3)
* Get yourself a copy of [JUCE](https://www.juce.com/get-juce).

## Prepare the project

* Open the `.jucer` file from this project in the ProJucer
* Check that the Modules paths match your computer's configuration and JUCE can find the modules.
* tick the box "Copy the module into the project folder" for each module, save the project and double check that Juce created the appropriate folders in `JuceLibraryCode/modules`

### Build the Project

On Bela:
```
make -C Builds/LinuxMakefile/
```

Run:
```
./Builds/LinuxMakefile/build/belaJuce
```

## Pro-tip

The Juce header file is huge and if your project takes too long to compile you may have two ways around it :
* use precompiled headers (easier to setup. See, e.g.:  [here](https://www.google.co.uk/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=gcc+precompiled+headers&*)
* use `distcc` to off-load compiling back to your host computer (harder to setup, better performance):
** run this on Bela:
```
apt-get install distcc
mkdir -p ~/.distcc
echo 192.168.7.1 >> ~/.distcc/hosts
export CXX="distcc arm-linux-gnueabihf-g++"
export CC="distcc arm-linux-gnueabihf-gcc"
```
note that the last two lines will need to be run in every new shell you start (or after every reboot). You may consider adding them to `~/.bashrc` if you find them useful.
** On your host computer, install the gcc 4.9 cross-compiler `arm-linux-gnueabihf-g++-4.9` (e.g.: for Mac you would get it  [here](http://www.welzels.de/blog/en/arm-cross-compiling-with-mac-os-x/)
** On your host computer install `distccd`
** On the host computer, run the following in a terminal to start the `distcc` daemon
```
distccd --verbose --daemon --allow 192.168.7.2
```
If everything works fine, when you build the Juce project, it should now be much faster.

