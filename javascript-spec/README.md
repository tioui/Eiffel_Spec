Eiffel to JavaScript compiler for EiffelStudio
==============================================

Installation
------------

This installation work on Linux Ubuntu 14.04. It should work on other ditribution with very little modification.

### Installation of EiffelStudio

Before compiling the spec, you must install the EiffelStudio ".tar.bz2" file. Do not use the ".deb" file or the PPA (uninstall it if you already install one of those). If you don't have it install yet, follow this steps:

* Download the latest version (I use the version 14.05) of EiffelStudio (".tar.bz2" file) on the website: http://sourceforge.net/projects/eiffelstudio/files/
* Extract the program and put it in a system directory. To do this, open a terminal and use these command lines:

```bash
cd /path/of/the/directory/containing/the/tar.bz2/file
tar xfj Eiffel_??.??_gpl_*_linux-x86*.tar.bz2
sudo mv Eiffel_??.?? /usr/local/
```

* Now that you have the program installed, you can make a launcher. To do this, create an executable file in a system path directory and put the execution lines in it:

```bash
sudo nano /usr/local/bin/eiffelstudio
```

Put this content in the file:

***

	#!/bin/sh
	export LANG=C
	export ISE_EIFFEL=/usr/local/Eiffel_14.05
	#export ISE_PLATFORM=linux-x86    # Uncomment if you use a 32 bits EiffelStudio
	export ISE_PLATFORM=linux-x86-64
	export PATH=$PATH:$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
	estudio

***

Put the file executable:

```bash
sudo chmod +x /usr/local/bin/eiffelstudio
```

### Installation of the Fastcomp LLVM and clang toolchain

Now that you have EiffelStudio on your system, you need to install the Fastcomp toolchain. This toolchain *must* be install in the /opt/ directory. To do this, follow this steps:

* Install the dependancy:

```bash
sudo apt-get update
sudo apt-get install build-essential cmake python2.7 nodejs git-core default-jre
```

* Cloning the Fastcomp toolchain

```bash
mkdir myfastcomp
cd myfastcomp
git clone https://github.com/kripken/emscripten-fastcomp
cd emscripten-fastcomp
git clone https://github.com/kripken/emscripten-fastcomp-clang tools/clang
```

* Build the toolchain

```bash
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD="X86;JSBackend" -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_INCLUDE_TESTS=OFF -DCLANG_INCLUDE_EXAMPLES=OFF -DCLANG_INCLUDE_TESTS=OFF
make -j1
```

* Move the toolchain in /opt/LLVM

```bash
cd ..
sudo mv build /opt/LLVM
```

### Installation of the Emscripten compiler:

Now, you need to install the Emscripten compiler. This compiler take the LLVM bytecode and compile it to JavaScript. The step to installing it:

* Downloading Emscripten

```bash
git clone https://github.com/kripken/emscripten.git
sudo mv emscripten /opt/
```

* Configuring the Emscripten compiler

```bash
/opt/emscripten/emcc --help # This will create the ~/.emscripten file
nano ~/.emscripten
```

In the file, change the line starting with (in my file, it is line 8):

```
 LLVM_ROOT = ...
```

to:

```
LLVM_ROOT = os.path.expanduser(os.getenv('LLVM') or '/opt/LLVM/bin')
```

### Compilation of the EiffelStudio javascript compiler

First, let's compile the Eiffel run-time library for the new spec. To compile the run-time, follow these steps:

* Download and extract the zipped repository at https://github.com/tioui/Eiffel_Spec .
* In the terminal, go to the javascript-spec directory in the repository.

```bash
cd /path/to/the/repository/javascript-spec
```

* Open a terminal (and keep it for each one of the following steps) and set the environment variables:

```bash
export ISE_EIFFEL=/usr/local/Eiffel_14.05 # Change that for your EiffelStudio directory
export ISE_PLATFORM=javascript
```

* Install the dependancies:

```bash
sudo apt-get install libgtk2.0-dev libxtst-dev build-essential subversion
```

* Download the Eiffel run-time source code:

```bash
svn co https://svn.eiffel.com/eiffelstudio/branches/Eiffel_14.05/Src/C/ eif_C
```

* Copy the config file in the CONFIG directory of the eif_C directory:

```bash
cp $ISE_PLATFORM eif_C/CONFIGS
```

* Go to the eif_C directory and compile the run-time:

```bash
cd eif_C
./quick_configure
mv run-time/libfinalized.a run-time/libfinalized.bc
mv run-time/libwkbench.a run-time/libwkbench.bc
```

* Note: If the compilation fail, try changing the ~/.subversion/config file by adding these lines after the [miscellany] section and return to the Run-time download step:

***

	[miscellany]
	global-ignores = *.o *.lo *.la #*# .*.rej *.rej .*~ *~ .#* *.swp ~*
	enable-auto-props = yes
	use-commit-times = yes

***

* Some run-time file must be compile for your PC and not for the JavaScript compiler:

```bash
cd run-time
gcc -c -I.. -I. -I../ipc/app -I./include -I../idrs -O3 -Wall -pedantic -std=gnu99 -pipe -fPIC -D_GNU_SOURCE  x2c.c
gcc -c -I.. -I. -I../ipc/app -I./include -I../idrs -O3 -Wall -pedantic -std=gnu99 -pipe -fPIC -D_GNU_SOURCE offset.c -o offset.o
gcc -o x2c x2c.o offset.o   -lm
```

### Install the JavaScript Spec in the EiffelStudio directory

* Copy the linux PC spec in EiffelStudio to create the JavaScript spec:

```bash
cp -rp $ISE_EIFFEL/studio/spec/linux-x86* $ISE_EIFFEL/studio/spec/$ISE_PLATFORM
cp -rp $ISE_EIFFEL/studio/config/linux-x86* $ISE_EIFFEL/studio/config/$ISE_PLATFORM
```

* Replace the linux PC runtime by the javascript runtime in the new spec:

```bash
cd ..
cp run-time/*.h $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/include
cp config.sh $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/include
cp run-time/lib* $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/lib
cp run-time/x2c $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
```

* Create the launcher:

```bash
sudo nano /usr/local/bin/eiffelstudio-javascript
```

Put the following content:

***

	#!/bin/sh
	export LANG=C
	export ISE_EIFFEL=/usr/local/Eiffel_14.05
	export ISE_PLATFORM=javascript
	export PATH=$PATH:$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
	estudio

***

Put the file executable:

```bash
sudo chmod +x /usr/local/bin/eiffelstudio-javascript
```

* To start the JavaScript EiffelStudio compiler:

```bash
eiffelstudio-javascript
```

### Note about the build

When you use the compiler of eiffelstudio-javascript, it does not create a JavaScript file. It create a LLVM bytecode file. To create the JavaScript file, you have to execute the Emscripten compiler on the LLVM bytecode file. To create just the JavaScript file (adapt the binary_file file to whatever is compile in you F_code directory):

```bash
mv binary_file binary_file.bc
/opt/emscripten/emcc -s TOTAL_MEMORY=33554432 -s AGGRESSIVE_VARIABLE_ELIMINATION=1 -s ASSERTIONS=2 -O3 --closure 0 -o binary_file.js binary_file.bc
```

It is possible that you have to adjust the TOTAL_MEMORY variable. If the program is a console program, you can try it with NodeJS:

```bash
nodejs binary_file.js
```

To generate the JavaScript and an HTML file that you can see in a browser, use:

```bash
mv binary_file binary_file.bc
/opt/emscripten/emcc -s TOTAL_MEMORY=33554432 -s AGGRESSIVE_VARIABLE_ELIMINATION=1 -s ASSERTIONS=2 -O3 --closure 0 -s NO_EXIT_RUNTIME=1 -o binary_file.html binary_file.bc
```

Again here, it is possible that you have to adjust the TOTAL_MEMORY variable.
