EiffelStudio Cross-compiler for Android
=================================

Installation
------------

This installation work on Linux Ubuntu 19.05. It should work on other ditribution with very little modification.

### Installation of EiffelStudio

Before compiling the spec, you must install the EiffelStudio ".tar.bz2" file. Do not use the ".deb" file or the PPA (uninstall it if you already install one of those). If you don't have it install yet, follow this steps:

* Download the latest version (I use the version 19.05) of EiffelStudio (".tar.bz2" file) on the website: http://sourceforge.net/projects/eiffelstudio/files/
* Extract the program and put it in a system directory. To do this, open a terminal and use these command lines:

```bash
cd /path/of/the/directory/containing/the/tar.bz2/file
tar xfj Eiffel_??.??_gpl_*_linux-x86*.tar.bz2
sudo mv Eiffel_??.?? /usr/local/
```

* Now that you have the program installed, you can make a launcher. To do this, create an executable file in a system path directory and put the execution lines in it:

```bash
sudo editor /usr/local/bin/eiffelstudio
```

Put this content in the file:

***

	#!/bin/sh
	export LANG=C
	export ISE_EIFFEL=/usr/local/Eiffel_19.05
	#export ISE_PLATFORM=linux-x86    # Uncomment if you use a 32 bits EiffelStudio
	export ISE_PLATFORM=linux-x86-64
	export PATH=$PATH:$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
	estudio

***

Put the file executable:

```bash
sudo chmod +x /usr/local/bin/eiffelstudio
```

### Installation of the Android NDK

Now that you have EiffelStudio on your system, you need to install the Android NDK. This NDK *must* be install in the /opt/ directory. To do this, follow this steps:

* Download and install the NDK from https://developer.android.com/ndk/downloads/index.html . I use the version r20.
* Extract the NDK and put it in the /opt/ directory. To do this, open a terminal and use these command lines:

```bash
cd /path/of/the/directory/containing/the/zip/file
sudo mv android-ndk-*-linux-x86*.zip /opt/
cd /opt/
sudo unzip android-ndk-*-linux-x86*.zip
sudo rm android-ndk-*-linux-x86*.zip
sudo ln -s android-ndk-r20 android-ndk # change the r20 value to your android ndk version.
```

### Compilation of the EiffelStudio cross-compiler for Android

And now, compile the Eiffel run-time library for the new spec. For this exemple, I will the architecture ARM 32 bits (armeabi) with the Android version 22. If you want to change the Android version, change the $android_version variable in the target file. To compile the run-time, follow this steps:

* Download and extract the zipped repository at https://github.com/tioui/Eiffel_Spec .
* In the terminal, go to the android-spec directory in the repository (the directory containing the target (android-arm) file).

```bash
cd /path/to/the/repository/
```

* Open a terminal (and keep it for each one of the following steps) and set the environment variables:

```bash
export ISE_EIFFEL=/usr/local/Eiffel_19.05 # Change that for your EiffelStudio directory
export ISE_PLATFORM=android-arm # change to follow your spec
```

* Install the dependancies:

```bash
sudo apt-get install libgtk2.0-dev libxtst-dev build-essential subversion
```

* Download the Eiffel run-time source code:

```bash
svn co https://svn.eiffel.com/eiffelstudio/branches/Eiffel_19.05/Src/C/ eif_C
```

* Apply the patch:

```bash
patch -p0 -i eif_C.patch
```

* Copy the config file in the CONFIG directory of the eif_C directory:

```bash
cp $ISE_PLATFORM eif_C/CONFIGS
```

* Go to the eif_C directory and compile the run-time:

```bash
cd eif_C
./quick_configure
```

* Note: If the compilation fail, try changing the ~/.subversion/config file by adding these lines after the [miscellany] section and return to the Run-time download step:

***

	[miscellany]
	global-ignores = *.o *.lo *.la #*# .*.rej *.rej .*~ *~ .#* *.swp ~*
	enable-auto-props = yes
	use-commit-times = yes

***

* Some run-time file must be compile for your PC and not for Android:

```bash
cd run-time
gcc -c -I.. -I. -I../ipc/app -I./include -I../idrs -O3 -Wall -pedantic -std=gnu99 -pipe -fPIC -D_GNU_SOURCE  x2c.c
gcc -c -I.. -I. -I../ipc/app -I./include -I../idrs -O3 -Wall -pedantic -std=gnu99 -pipe -fPIC -D_GNU_SOURCE offset.c -o offset.o
gcc -o x2c x2c.o offset.o   -lm
```

### Install the Android Spec in the EiffelStudio directory

* Copy the linux PC spec in EiffelStudio to create the Android spec:

```bash
cp -rp $ISE_EIFFEL/studio/spec/linux-x86* $ISE_EIFFEL/studio/spec/$ISE_PLATFORM
cp -rp $ISE_EIFFEL/studio/config/linux-x86* $ISE_EIFFEL/studio/config/$ISE_PLATFORM
```

* Replace the linux PC runtime by the Android runtime in the new spec:

```bash
cd ..
cp run-time/*.h $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/include
cp config.sh $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/include
cp run-time/lib* $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/lib
cp run-time/x2c $ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
```

* Create the launcher:

```bash
sudo editor /usr/local/bin/eiffelstudio-${ISE_PLATFORM}
```

Put the following content:

***

	#!/bin/sh
	export LANG=C
	export ISE_EIFFEL=/usr/local/Eiffel_19.05
	export ISE_PLATFORM=android-arm		# Adapt with the good architecture
	export PATH=$PATH:$ISE_EIFFEL/studio/spec/$ISE_PLATFORM/bin
	estudio

***

Put the file executable:

```bash
sudo chmod +x /usr/local/bin/eiffelstudio-${ISE_PLATFORM}
```

* To start the Android EiffelStudio cross compiler:

```bash
eiffelstudio-android-arm
```

Note: By default, if you have a precompiled library in your project, the compilation will fail. You can just remove the precompiled library from your project (you can use another target) or you can create the precompiled directory for Android (only usefull to remove EiffelStudio message error) this way:

```bash
mkdir -p ~/.es/eiffel_user_files/19.05/precomp/spec/${ISE_PLATFORM}
cp  ~/.es/eiffel_user_files/19.05/precomp/spec/linux-x86*/*.ecf ~/.es/eiffel_user_files/19.05/precomp/spec/${ISE_PLATFORM}/
```
