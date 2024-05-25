# - Experimental/variant - QEMU MESA GL/3Dfx Glide Pass-Through
For info on the project, refer to the original project page. This fork is just an experiment to try out some changes

## Content
    qemu-0/hw/3dfx      - Overlay for QEMU source tree to add 3Dfx Glide pass-through device model
    qemu-1/hw/mesa      - Overlay for QEMU source tree to add MESA GL pass-through device model
    scripts/sign_commit - Script for stamping commit id
    wrappers/3dfx       - Glide wrappers for supported guest OS/environment (DOS/Windows/DJGPP/Linux)
    wrappers/mesa       - MESA GL wrapper for supported guest OS/environment (Windows)
## Patch
    00-qemu82x-mesa-glide.patch - Patch for QEMU version 8.2.x (MESA & Glide)
    01-qemu72x-mesa-glide.patch - Patch for QEMU version 7.2.x (MESA & Glide)
    02-qemu620-mesa-glide.patch - Patch for QEMU version 6.2.0 (MESA & Glide)

## Building QEMU
This seems to be the biggest hurdle that many people encounter. I am trying this on a Windows 11 machine as host, and targeting a Windows Xp 32 bit machine. I started from a fresh install of W11 with minimal apps installed, so your mileage may vary, depending from what you tried before and what you already installed.

This is the overall sequence of events to build the project: 

- Install Msys2; the standard location is fine, or use a custom location if you don't like the C:\ default location
- Once done, run **MSYS2 UCRT64** shell and type the following command
  
```pacman -Syu```

- Restart if needed; depending from how many items you need to update the app may just close to update; run again the **UCRT64** app and type

```pacman -Su```

- Now you need to install all the basic tools required to cross build; you have a 64 bit and need to install tools for the 64 and the 32 bit, as your target system is either W98 or XP most likely, so you will need to swap between different versions of the MSYS2 apps. For now you will need to install these tools in the UCRT64 version, so be sure you launch the right version of the tool, or this whole thing will fail. Start installing the tools with these commands

```pacman -S base-devel mingw-w64-x86_64-toolchain git python ninja rsync patch```

```pacman -S mingw-w64-x86_64-glib2 mingw-w64-x86_64-pixman python-setuptools```

```pacman -S mingw-w64-x86_64-gtk3 mingw-w64-x86_64-SDL2 mingw-w64-x86_64-libslirp```

- With this you are done with the tools; now close this version of MSYS2 and from your start menu, find the MSYS2 version called **MINGW64** and you are ready to check out the project repo and start to patch and build Qemu.

- Create a new directory; at this point you should be in your "home" folder (use ```pwd``` command to check where are you if you are lost); so you can use the original info from the main project

```mkdir ~/myqemu && cd ~/myqemu```

```git clone https://github.com/kjliew/qemu-3dfx.git```

```cd qemu-3dfx```

```wget https://download.qemu.org/qemu-8.2.1.tar.xz```

- If you are having issues with wget to retrieve the file, just use your browser, grab the file and copy it in your MSYS2 home folder manually

```tar xf qemu-8.2.1.tar.xz```

```tar xf qemu-8.2.1.tar.xz```

- Run the Tar command twice; the first time it will tell you that some folders are not available; while the second time it will tell you that some files already exist; I had issues when running it just once so I just ran it twice and it was working; no clue why.

```rsync -r ../qemu-0/hw/3dfx ../qemu-1/hw/mesa ./hw/```

```patch -p0 -i ../00-qemu82x-mesa-glide.patch```

```bash ../scripts/sign_commit```

```mkdir ../build && cd ../build```

```../qemu-8.2.1/configure```

- If the output of the previous command is not giving you errors, you should be ready to actually build with the next command. Otherwise try to solve the issue before running the make command, as it is useless to try to do so before completing the configuration successfully

```make```

- Once done, you should have a bunch of EXE files in the build folder; which are the Qemu files patched. Now you need to build the ***Guest Wrappers***. You can close the **MSYS2 APP** now, as you will not need it for the next step.
- If something failed, trace back your steps and repeat the steps again. Usually it is better to just delete the whole folder you checked out and start from scratch


## Building Guest Wrappers
Now you will need to produce the binaries for the target system, which is a 32 bit system, while you have a 64 bit system. From your start menu, find the **MSYS2 MINGW32** app and run it; this is going to open in the same "home" location as you were before, so all your previous files will still be there, but you need to install the specific tools for the 32 bit side of the house. Run these commands

```pacman -S base-devel mingw-w64-i686-toolchain git python ninja rsync patch```

```pacman -S mingw-w64-i686-glib2 mingw-w64-i686-pixman python-setuptools```

```pacman -S mingw-w64-i686-gtk3 mingw-w64-i686-SDL2 mingw-w64-i686-libslirp```

- Now move to the wrappers folder, starting with the 3dfx folder and build

```cd ~/myqemu/qemu-3dfx/wrappers/3dfx```

```mkdir build && cd build```

```bash ../../../scripts/conf_wrapper```

- Check that the output here is not giving you any error; if this fail, you need to solve the problem before you can continue. If something is missing it means you are missing a package or using the wrong executable (like you are using the wrong version of **MSYS2**); trace back your steps and figure that out

```make```

```make clean```

- You should have at this point the DLLs for your W98/Xp system in the build folder. Repeat the same steps for the **Mesa** folder to produce the **OGL** library

```cd ~/myqemu/qemu-3dfx/wrappers/mesa```

```mkdir build && cd build```

```bash ../../../scripts/conf_wrapper```

- Check that the output here is not giving you any error; if this fail, you need to solve the problem before you can continue. If something is missing it means you are missing a package or using the wrong executable (like you are using the wrong version of **MSYS2**); trace back your steps and figure that out

```make```

```make clean```

- And with this, you have all the libraries you need for the installation.

## Installing Guest Wrappers
**For Win9x/ME:**  
 - Copy `FXMEMMAP.VXD` to `C:\WINDOWS\SYSTEM`  
 - Copy `GLIDE.DLL`, `GLIDE2X.DLL` and `GLIDE3X.DLL` to `C:\WINDOWS\SYSTEM`  
 - Copy `GLIDE2X.OVL` to `C:\WINDOWS`  
 - Copy `OPENGL32.DLL` to `Game Installation` folders

**For Win2k/XP:**  
 - Copy `FXPTL.SYS` to `%SystemRoot%\system32\drivers`  
 - Copy `GLIDE.DLL`, `GLIDE2X.DLL` and `GLIDE3X.DLL` to `%SystemRoot%\system32`  
 - Run `INSTDRV.EXE`, require Administrator Priviledge  
 - Copy `OPENGL32.DLL` to `Game Installation` folders

For copying files I just created a folder where I place all the files, then I make a ISO out of that folder with a free tool for Windows, and mount it with Qemu, so I can copy the files on the image. At that point you should be good to go to start experimenting on your own. 

 -------------------
Thanks to KJ Liew <liewkj@yahoo.com> for creating this wrapper. If you want to support the original project, and donate to the original author, you can do that on the main project page from which this project was branched from.


