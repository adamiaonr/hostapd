# code analysis
in order to better understand the structure of the hostapd code, i decided to 
give [codeviz](https://github.com/petersenna/codeviz) a try. so, all the steps 
below assume you have a copy of codeviz's git repo:

```
$ git clone https://github.com/petersenna/codeviz.git
```

below i summarized the steps i've followed to generate a `.graph` file using the 
codeviz's `genfull` script. all the steps have been tested on Ubuntu 16.04.2 LTS 
(Xenial), 4.4.0-70-generic (x86_64) Linux kernel.

## generating the full graph

### a note on the `cdepn` method
first, i've tried to follow the `cdepn` method proposed by the [codeviz README](https://github.com/petersenna/codeviz), which involves compiling a patched version of `gcc` (version 4.6.2). that 
turned out to be a "huge mistake" (explanation below). so, i ended up using the alternative `nccout` method: not only did it work, it was also quick & easy.

the `cdepn` method proposed in the [codeviz README](https://github.com/petersenna/codeviz) is full of minor 
errors which break the compilation of `gcc`. it also doesn't work well in x86_64 architectures. anyway, 
if you really wanna go down the `cdepn` path, check these sources for 
troubleshooting [1](http://www.jianshu.com/p/b3ed2b3652ac), [2](https://stephanfr.com/2012/10/20/build-a-debug-version-of-gcc-4-7-2-for-ubuntu-12-04/), [3](http://www.yonch.com/tech/code-call-graphs-codeviz)).

### generating `nccout` files

assure you have the `ncc` Ubuntu package installed:

```
$ sudo apt-get install ncc
```

instead of using the `ncc` command (as suggested in the [codeviz README](https://github.com/petersenna/codeviz)), 
you should use `nccgen`:

```
$ cd <hostapd-ccn-dir>/hostapd-1.0/hostapd/
$ cp defconfig-ubuntu-xenial-x68_64 .config
$ make clean; make -i CC='nccgen -ncgcc -ncld -ncfabs'
$ find . -name \*.nccout | xargs cat > code.map.nccout
```

the last step combines all `.nccout` files into a single file: `code.map.nccout`.

### generating the `.graph` file

```
$ cd <codeviz-dir>/bin
$ ./genfull --method cncc -d <hostapd-ccn-dir>/hostapd-1.0/hostapd/
```

the fullgraph should become available as `full.graph`. this is basically a 
`.dot` file, with all the function calls in hostapd. we don't generate a 
graphical representation of the callgraph yet.

## graphical representation of the callgraph

since i was interested in the callers of a given function - a functionality 
which wasn't provided by codeviz's gengraph - i created my own script to do it, 
available in `<hostapd-ccn-dir>/scripts/code-analysis/`.

e.g. to generate a `.pdf` callgraph of the parents of `hostapd_new_assoc_sta()`, 
you can run:

```
$ cd <hostapd-ccn-dir>/scripts/code-analysis/
$ cp <codeviz-dir>/bin/full.graph hostapd.graph
$ python code_analysis.py --graph-file hostapd.graph --print-callers "hostapd_new_assoc_sta" --max-depth 4
```

the `max-depth` option limits the number of parent 'generations' to 4. the 
resulting callgraph will be available as `hostapd_new_assoc_sta-4.pdf`.
