Installation procedure for Ubuntu SDK (click) cross-compiler for EiffelStudio

This installation work on Linux Ubuntu 14.04. It should work on other ditribution with very little modification.

Before compiling the spec, you must install the EiffelStudio ".tar.bz2" file. Do not use the ".deb" file. If you don't have it install yet, follow this steps:

	- Download the latest version (I use the version 7.1) of EiffelStudio (".tar.bz2" file) on the website: http://sourceforge.net/projects/eiffelstudio/files/
	- Extract the program and put it in a system directory. To do this, open a terminal and use these command lines:
		# cd /path/of/the/directory/containing/the/tar.bz2/file
		# tar xfj Eiffel_??.??_gpl_*_linux-x86*.tar.bz2
		# sudo mv Eiffel_??.?? /usr/local/
	- Now that you have the program installed, you can make a launcher. To do this, create an executable file in a system path directory and put the execution lines in it:
		# sudo nano /usr/local/bin/eiffelstudio
			#!/bin/sh
			export LANG=C
			export ISE_EIFFEL=/usr/local/Eiffel_15.01
			export ISE_PLATFORM=linux-x86
			#export ISE_PLATFORM=linux-x86-64    # Uncomment if you use a 64 bits EiffelStudio
			export PATH=$PATH:$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
			estudio	
		# sudo chmod +x /usr/local/bin/eiffelstudio

Now that you have EiffelStudio on your system, you need to install the Ubuntu SDK. To do this, follow this steps:

	- Add the PPA
		# sudo add-apt-repository ppa:ubuntu-sdk-team/ppa
	- Be sure that everything is up to date on your system (Optionnal):
		# sudo apt-get update && sudo apt-get dist-upgrade
	- Install the Ubuntu SDK
		# sudo apt-get install ubuntu-sdk
	- In the Ubuntu SDK, add kits for the architecture and framework wou want to develop.

And now, compile the Eiffel run-time library for the new spec. To do this, follow this steps:

	- Download and extract the zipped repository at https://github.com/tioui/Eiffel_Spec .
	- In the terminal, go to the ubuntu-click-spec directory in the repository (the directory containing the ubuntu-click-14.10-armhf *text* file).
		# cd /path/to/the/repository/ubuntu-click-spec
	- Modify the spec file (named ubuntu-click-14.10-armhf on this repo) to adapt your desired architecture and framework. To do that, open the file (and rename it) and modify the variable:
		# click_arch=<Disired architecture like armhf or i386)
		# click_framework=<Disired architecture like the ubuntu-sdk-14.10>
	- You also need to modify the click_toolchain variable to the toolchain prefix. You can find it with the folowing command:
		# click chroot -a armhf -f ubuntu-sdk-14.10 maint find /usr/bin/ -name "*-ld"
 	- Open a terminal (and keep it for each one of the following steps) and set the environment variables:
		# export ISE_EIFFEL=/usr/local/Eiffel_15.01 # Change that for your EiffelStudio directory
		# export ISE_PLATFORM=ubuntu-click-14.10-armhf
	- Install the dependancies:
		# sudo apt-get install libgtk2.0-dev libxtst-dev build-essential subversion
	- Download the Eiffel run-time source code:
		# svn co https://svn.eiffel.com/eiffelstudio/branches/Eiffel_15.01/Src/C/ eif_C
	- Copy the config file in the CONFIG directory of the eif_C directory:
		# cp $ISE_PLATFORM eif_C/CONFIGS
	- Go to the eif_C directory and compile the run-time:
		# cd eif_C
		# ./quick_configure 
		-Note: If the compilation fail, try changing the ~/.subversion/config file by adding these lines after the [miscellany] section and return to the Run-time download step:
			# nano ~/.subversion/config
				[miscellany]
				global-ignores = *.o *.lo *.la #*# .*.rej *.rej .*~ *~ .#* *.swp ~*
				enable-auto-props = yes
				use-commit-times = yes
	- Some run-time file must be compile for your PC and not the Ubuntu SDK:
		# cd run-time 
		# gcc -c -I.. -I. -I../ipc/app -I./include -I../idrs -O3 -Wall -pedantic -std=gnu99 -pipe -fPIC -D_GNU_SOURCE  x2c.c
		# gcc -c -I.. -I. -I../ipc/app -I./include -I../idrs -O3 -Wall -pedantic -std=gnu99 -pipe -fPIC -D_GNU_SOURCE offset.c -o offset.o
		# gcc -o x2c x2c.o offset.o   -lm

Create the new spec for the click environment :

	- Copy the linux PC spec in EiffelStudio to create the ubuntu-click-* spec (you probably want to run those with sudo):
		# cp -rp $ISE_EIFFEL/studio/spec/linux-x86* $ISE_EIFFEL/studio/spec/$ISE_PLATFORM
		# cp -rp $ISE_EIFFEL/studio/config/linux-x86* $ISE_EIFFEL/studio/config/$ISE_PLATFORM
	- Replace the linux PC runtime by the ubuntu click runtime in the new spec (you probably want to run those with sudo):
		# cd ..
		# cp run-time/*.h $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/include
		# cp config.sh $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/include
		# cp run-time/lib* $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/lib
		# cp run-time/x2c $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
	- Update the click environment (don't forget to change the armhf for your click architecture and ubuntu-sdk-14.10 for your click framework)
		# mkdir -p /tmp/$ISE_PLATFORM
		# cp -rp $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/lib /tmp/$ISE_PLATFORM/
		# cp -rp $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/include /tmp/$ISE_PLATFORM/
		# click chroot -a armhf -f ubuntu-sdk-14.10 maint mkdir -p $ISE_EIFFEL/studio/spec/
		# click chroot -a armhf -f ubuntu-sdk-14.10 maint mv /tmp/$ISE_PLATFORM $ISE_EIFFEL/studio/spec/
	- Create the launcher:
		# sudo nano /usr/local/bin/eiffelstudio-ubuntu-click-14.10-armhf
			#!/bin/sh
			export LANG=C
			export ISE_EIFFEL=/usr/local/Eiffel_15.01
			export ISE_PLATFORM=ubuntu-click-14.10-armhf
			export PATH=$PATH:$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
			estudio	
		# sudo chmod +x /usr/local/bin/eiffelstudio-ubuntu-click-14.10-armhf

Now, you can use the command line "eiffelstudio-ubuntu-click-14.10-armhf" to start the Ubuntu SDK click EiffelStudio cross compiler. Please note that to be able to compile a project, you must remove any precompile library from the project.

