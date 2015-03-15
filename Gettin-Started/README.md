RTEMS on Beagle Bone Black
========

###Overview

1. Building toolchain,uboot,qemu and others
2. RTEMS building & Building Beagle BSPs and testsuites using toolchain
3. Running Test suite on PC
4. Writind Image on SD card
5. Booting and getting output using UART



## Step 1
To get started, first make directory as following.

<pre><code>mkdir -p development/rtems/sources</code></pre>
<pre><code>cd development/rtems/sources</code></pre>

Now fetch RSB(rtems source builder). RSB will first verify wether all the required dependencies are installed or not.

<pre><code>git clone -b beagle https://github.com/bengras/rtems-source-builder.git</code></pre>

Verify the basic dependencies.

<pre><code>./rtems-source-builder/source-builder/sb-check</pre></code>
If found any missing dependency then install it using <pre><code>apt-get</pre></code>

After installing it run the same line you will get message "Environment is ok" at end.

Now we are going to build beagle bset. So for that 

<pre><code>cd rtems-source-builder/rtems</pre></code>
<pre><code>../source-builder/sb-set-builder --log=beagle.txt --prefix=$HOME/development/rtems/4.11 4.11/rtems-arm</pre></code>	

Great, This will have built the toolchain with the right target, qemu, uboot, and some supporting utilities needed to prepare the SD card.

## Step 2

In this step we are going to build BBB BSPs.

First setting the path that includes the build tools.

<pre><code>cd $HOME/development/rtems</pre></code>
<pre><code>export PATH=$HOME/development/rtems/4.11/bin:$PATH</pre></code>

Fetching the code

<pre><code>git clone https://git.rtems.org/rtems.git rtems-src </pre></code>
<pre><code>cd rtems-src</pre></code>

Now we will generate configure files.
<pre><code>./bootstrap; ./bootstrap -p
cd ..</pre></code>

Now we are going to buildfor BBB with the full test suite. making CONSOLE_POLLED=1 at configure time for console operating in polled mode.

<pre><code>mkdir b-beagle ; cd b-beagle</pre></code>
<pre><code>CONSOLE_POLLED=1 ../rtems-src/configure --target=arm-rtems4.11 --enable-rtemsbsp="beagleboneblack" --enable-tests</pre></code>

Now compilation 

<pre><code>make -j4</pre></code>

## Step 3

Now we will run the test suite

<pre><code>cd $HOME/development/rtems</pre></code>

Now fetch the rtems tools having all the test suites
<pre><code>git clone -b bbxm-wip https://github.com/bengras/rtems-tools.git rtems-tools</pre></code>

Now go to tester directory

<pre><code>cd rtems-tools/tester</pre></code>
Now its time to test it. (Take approx. 30-40 minutes.)
<pre><code>./rtems-test --log=bbxm.log --report-mode=all --rtems-bsp=beagleboardxm_qemu --rtems-tools=$HOME/development/rtems/4.11 $HOME/development/rtems/b-beagle/arm-rtems4.11/c/beagleboardxm</pre></code>

## Step 4

Now its time to write our image on SD card.

<pre><code>cd $HOME/development/rtems/rtems-src/c/src/lib/libbsp/arm/beagle/simscripts</pre></code>
<pre><code>sh sdcard.sh $HOME/development/rtems/4.11 $HOME/development/rtems/b-beagle/arm-rtems4.11/c/beagleboneblack/testsuites/samples/hello/hello.exe</pre></code>

After the execution you will get "Result is in bone_hello.exe-sdcard.img."

Now write the build image to sd card by inserting sd and finding the mount path.

Be careful. You can wipe your harddrive.
<pre><code>dd if=bone_hello.exe-sdcard.img of=/dev/sdb bs=4096</pre></code>

boot...

## Step 5

Now power on the BBB and connect ftdi cable. You can connect 3 cables. 

1. BBB Rx - FTDI Tx
2. BBB Tx - FTDI Rx
3. BBB GND- FTDI GND 

[Reference](http://inspire.logicsupply.com/p/serial-connection-j1.html) 

Now after connecting usb to pc execute command in terminal.
<pre><code>sudo apt-get install picocom </pre></code>
<pre><code>picocom -b 115200 /dev/ttyUSB0</pre></code>

Here you can see my setup on 

[Imgur](http://i.imgur.com/dPFBqcV.jpg?1)

