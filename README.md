# Building Swift on Windows

## Developer Mode

The Swift Package Manager uses symbolic links, but Microsoft has decided that symbolic links might be harmful (see [there](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-vista/cc766301(v=ws.10)) under "Create symbolic links").

**Check:**  You can test if you can set symbolic links with the command `mklink newfile oldfile`, creating a symbolic link file named `newfile` pointing to `oldfile`. If this command is successful, everything is OK.

If you cannot set symbolic links, you can try to activate the Deverloper Mode in the Windows settings and then restart the computer. You should then be able to create symbolic links (test with the mentioned command again). If not, you need to set the SE_CREATE_SYMBOLIC_LINK privilege using the `gpedit.msc` tool (start it via the context menu of the Windows Explorer as Administrator). (Note that other security policies for your computer might overwrite this setting.) If you have the Home edition of of Windows, you first have to get this tool from Microsoft using the following script (open the command line window as Administrator):

```Batch
@echo off 

pushd "%~dp0" 

dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt 

dir /b %SystemRoot%\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt 

for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i" 

pause
```

You can then set the privilege in the `gpedit.msc` tool under Computer Configuration / Windows Settings / Security Settings / Local Policies / User Rights Assignment (use a right-click for the listed setting and choose Properties / Enhanced / Search to add yourself as user). Then restart your computer.

Note that group policies might forbid setting the above privilege. You might then have to speak with your IT apartment.

## Visual Studio

Keep the visual Studio installer as "vs_community.exe". Execute the follwowing command and remember the Python version then listed in the Visual Studio installer:

```batch
vs_community ^
  --add Component.CPython3.x64 ^
  --add Microsoft.VisualStudio.Component.Git ^
  --add Microsoft.VisualStudio.Component.VC.ATL ^
  --add Microsoft.VisualStudio.Component.VC.CMake.Project ^
  --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 ^
  --add Microsoft.VisualStudio.Component.Windows10SDK ^
  --add Microsoft.VisualStudio.Component.Windows10SDK.17763

```

After installation of Visual Studio, most of the following commands have to be executed in the "64 Native Tools Command Prompt for VSXXXX" ("XXXX" replaced by the named version of Visual Studio, e.g. "2019").

## Python

Change the Python installed by the Visual Studio installer as follows:

1. In the Windows settings, go to Add and Remove Programs.
2. Select the Python (64-bit) installed by Visual Studio.
3. Click Modify, then Yes, then Modify again and then Next.
4. Select a) "Download debug binaries" and b) "py launcher".
5. Click Install.

**Check:** Does the command `python --version`, executed in the "64 Native Tools Command Prompt for VSXXXX", print the correct Python version? If not, add `C:\Program Files (x86)\Microsoft Visual Studio\Shared\Python37_64` or `%ProgramFiles%\Microsoft Visual Studio\Shared\Python37_64` (see what matches) to the PATH environment variable.

**Check:** Create a Python file (extension `.py`) with content `import sys; print(sys.version)`, and call this file from the "64 Native Tools Command Prompt for VSXXXX" with only the path to it. The correct Python version should be then be displayed. If not, associate Python files (`*.py`) with `py.exe` installed by pylauncher from https://bitbucket.org/vinay.sajip/pylauncher/downloads/ (use launcher.amd64.msi: 64-bit launcher, installs to `C:\Program Files\Python Launcher`), by first calling your Python file by its path and then in the Windows dialog choose the Python Launcher with the "always" option. 

## Git

Git should now be installed by the Visual Studio installer.

**Check:** Does the command `git --version`, executed from the "64 Native Tools Command Prompt for VSXXXX", display a version 2 or greater? If not, add `%ProgramFiles%\Git\usr\bin` to the PATH environment variable.

**Check:** Is the `less` command available from the "64 Native Tools Command Prompt for VSXXXX"? If not, add `%ProgramFiles%\Git\usr\bin` to the PATH environment variable.

## The Working Directory

You may work in any directory, but in our commands we will use the drive `S:` as working directory. To set a path `mypath` as a drive:

1. Open Regedit and navigate to key `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` (for all users) or to `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run` (for the current user).
2. Right click on `Run`, the choose `New` and `String Value`. Choose a name (e.g. “Map S”).
3. Double click on the name and for `Value data` enter `subst S: mypath` (`mypath` replaced with your path).
4. Restart Windows.

Check: Type `S:` in the "64 Native Tools Command Prompt for VSXXXX". You should then change to the directory `S:`.

## Clone the sources

**Important:** Even if you would like to later checkout the sources with certain tags (e.g. for a certain release version, see section below), first clone the repositories with exactly the following commands, because some of its settings are crucial.

Go to `S:` in the "64 Native Tools Command Prompt for VSXXXX" and check out all needed sources:

```batch
git clone https://github.com/apple/llvm-project --branch swift/main llvm-project
git clone -c core.autocrlf=input -c core.symlinks=true https://github.com/apple/swift swift
git clone https://github.com/apple/swift-cmark cmark
git clone https://github.com/apple/swift-corelibs-libdispatch swift-corelibs-libdispatch
git clone https://github.com/apple/swift-corelibs-foundation swift-corelibs-foundation
git clone https://github.com/apple/swift-corelibs-xctest swift-corelibs-xctest
git clone https://github.com/apple/swift-tools-support-core swift-tools-support-core
git clone -c core.symlinks=true https://github.com/apple/swift-llbuild swift-llbuild
git clone https://github.com/JPSim/Yams Yams
git clone https://github.com/apple/swift-driver swift-driver
git clone https://github.com/apple/swift-argument-parser swift-argument-parser
git clone -c core.autocrlf=input https://github.com/apple/swift-package-manager swift-package-manager
git clone https://github.com/apple/indexstore-db indexstore-db

```

This will take a while.

## One-time Setup (re-run on Visual Studio upgrades)

Create links inside Visual Studio to some Swift files. Open the "64 Native Tools Command Prompt for VSXXXX" by right clicking whatever shortcut you used to open the first one, choosing **"Run As Administrator",** and pasting the above commands into the resulting window. You can then close the privileged prompt; this is the only step which requires elevation.

**Check:** When executing the following commands, there should be four  `<<===>>` displayed.

```
if exist "%UniversalCRTSdkDir%Include\%UCRTVersion%\ucrt\module.modulemap" del /Q "%UniversalCRTSdkDir%Include\%UCRTVersion%\ucrt\module.modulemap"
mklink "%UniversalCRTSdkDir%Include\%UCRTVersion%\ucrt\module.modulemap" S:\swift\stdlib\public\Platform\ucrt.modulemap
if exist "%UniversalCRTSdkDir%Include\%UCRTVersion%\um\module.modulemap" del /Q "%UniversalCRTSdkDir%Include\%UCRTVersion%\um\module.modulemap"
mklink "%UniversalCRTSdkDir%Include\%UCRTVersion%\um\module.modulemap" S:\swift\stdlib\public\Platform\winsdk.modulemap
if exist "%VCToolsInstallDir%include\module.modulemap" del /Q "%VCToolsInstallDir%include\module.modulemap"
mklink "%VCToolsInstallDir%include\module.modulemap" S:\swift\stdlib\public\Platform\visualc.modulemap
if exist "%VCToolsInstallDir%include\visualc.apinotes" del /Q "%VCToolsInstallDir%include\visualc.apinotes"
mklink "%VCToolsInstallDir%include\visualc.apinotes" S:\swift\stdlib\public\Platform\visualc.apinotes

```

## Dependencies (ICU, SQLite3, curl, libxml2 and zlib)

The instructions assume that the dependencies are in `S:\Library`. The directory structure should resemble:

```text
/Library
  ┝ icu-67
  │   ┕ usr/...
  ├ libcurl-development
  │   ┕ usr/...
  ├ libxml2-development
  │   ┕ usr/...
  ├ sqlite-3.28.0
  │   ┕ usr/...
  ┕ zlib-1.2.11
      ┕ usr/...
```

To build those dependencies, you first need the sources and build them yourself. You can also download all those dependencies, already built, from https://github.com/stefanspringer1/SwiftDependencies.git. In "64 Native Tools Command Prompt for VSXXXX" go to the directory `S:` and execute:

```batch
git clone https://github.com/stefanspringer1/SwiftDependencies.git Library
```

**Important:** Do not forget the `Library` in the above command to checkout with that diretory name.

**Check:** See if the above directory structure matches.

If you want the sources, the links for according repositories are:

- libcurl-development: https://github.com/curl/curl
- libxml2-development: https://github.com/gnome/libxml2
- zlib-1.2.11: https://github.com/madler/zlib

The sources for sqlite-3.28.0 can be obatined from https://sqlite.org/index.html, the ones for `icu-67` from https://icu.unicode.org/download.

## Check out the sources with certain tags

To get the tags in a repository, filtered by an expression, use in `S:` the following command for e.g. alles release versions for the "swift" repository (`C` choose the subdirectory, you do not have to change your current directory, you can drop e.g. `-C swift` when you are in the `swift` subdirectory), press the space bar to read more after an ":" or press "q" to quit:

```batch
git -C swift tag -l *RELEASE*
```

Or more precise: lookup for a certain version:

```batch
git -C swift tag -l *5.4.3-RELEASE*
```

You can then checkout the tag found with e.g.:

```batch
git -C swift checkout tags/swift-5.4.3-RELEASE
```

**Important:** Check the Git documentations to see what consequences the various types of checkouts have, in this case we have a so-called "detached head". You also get a warning, turn the warning permanently off with `git config --global advice.detachedHead false`. If you would like to make changes that you also like to check-in, you need to also create a branch while checking out a tag.

**Important:** Use e.g. `git -C swift switch -` the return from the detached head.

**Check:** See the tags of a current checkout (filtered) with e.g.:

```batch
git -C swift tag --points-at HEAD -l *5.4.3-RELEASE*
```

## Building the toolchain, #1

**Important:** In the following commands, subdirectories of `S:` will be used as build folders. At least at a first try, do not change those paths, as the build directory of one command might be used by a subsequent command.

Note:

- You may leave out out `-D CMAKE_C_FLAGS="/bigobj"` and `-D CMAKE_CXX_FLAGS="/bigobj"` as they are only necessary if you are getting "compile with /bigobj" errors.
- You may leave out `-D LLVM_PARALLEL_LINK_JOBS=1` and `-D DLLVM_PARALLEL_LINK_JOBS=1` or set higher numbers if you have enough RAM (might need 32 GB with debug information via `-D LLVM_ENABLE_PDB=YES`).
- You may leave out `-D LLVM_ENABLE_PDB=YES` if you do not need debug information.

Configuration as follows. The redirection at the end write to file all output (including errors) to the file `onf.log` so you can check e.g. for any "missing:" messages. It may take a minute or two.

```batch
cmake -B "S:\b\1" ^
  -C S:\swift\cmake\caches\Windows-x86_64.cmake ^
  -D CMAKE_BUILD_TYPE=Release ^
  -D CMAKE_INSTALL_PREFIX=C:\Library\Developer\Toolchains\unknown-Asserts-development.xctoolchain\usr ^
  -D LLVM_DEFAULT_TARGET_TRIPLE=x86_64-unknown-windows-msvc ^
  -D LLVM_ENABLE_PDB=YES ^
  -D LLVM_EXTERNAL_CMARK_SOURCE_DIR=S:\cmark ^
  -D LLVM_EXTERNAL_SWIFT_SOURCE_DIR=S:\swift ^
  -D CMAKE_C_FLAGS="/bigobj" ^
  -D CMAKE_CXX_FLAGS="/bigobj" ^
  -D LLVM_PARALLEL_LINK_JOBS=1 ^
  -D DLLVM_PARALLEL_LINK_JOBS=1 ^
  -D SWIFT_PATH_TO_LIBDISPATCH_SOURCE=S:\swift-corelibs-libdispatch ^
  -D SWIFT_WINDOWS_x86_64_ICU_I18N_INCLUDE=S:\Library\icu-67\usr\include ^
  -D SWIFT_WINDOWS_x86_64_ICU_I18N=S:\Library\icu-67\usr\lib\icuin67.lib ^
  -D SWIFT_WINDOWS_x86_64_ICU_UC_INCLUDE=S:\Library\icu-67\usr\include ^
  -D SWIFT_WINDOWS_x86_64_ICU_UC=S:\Library\icu-67\usr\lib\icuuc67.lib ^
  -G Ninja ^
  -S S:\llvm-project\llvm ^
  > conf.log 2>&1

```

Building:

```batch
ninja -C S:\b\1

```