For testing/using Plugins:

you better start minimal_vst2_host and load a plugin dll.
The minimal_vst2_host can be used as midi keyboard that sends noteOn/noteOff commands
to the plugin. 

In standalone mode u will probably dont hear any sound without a midi controller 
connected and activated on your PC.

Close the minimal_vst2_host with CTRL + C press(es). Its a console app, so treat it that way.
Its meant for quick testing not really meant for your convenient user experience.


For building/developing Plugins:

This JUCE lib is compiled with MinGW ( aka Linux GCC on Windows64 ) 7.3.1 x64.
Building needs 3 installed programs: QtSDK + CMake + git4Windows
With these 3 programs you get a complete Linux dev env on Windows64.
Its heaven compared to Visual Studio + cmd.exe.

What u get:

QtSDK = MinGW x64 compiler >= 7.3.1 ( your C/C++ compiler + gdb + windres + make )
      + QtCreator ( your IDE, aka Visual Studio replacement )
      + QWidgets ( developing GUI, if needed )

CMake = Programmable platform independent projects. 
        The entire workspace uses CMakeLists.txt recipe files
        instead of hard to read makefiles or Visual Studio project files.
        Every library and every app has one CMake recipe of similar content 
        ( easy to adapt to your needs )
        
        I dont use fancy CMake commands and i dont pollute global namespaces
        so i made it easy for others to contribute and use my stuff.
        
        You need atleast CMake version 3.14 and c++17 standard.
        
git4windows = Brings git, but more importantly it brings a Linux terminal to Windows.

        I daily use the included Linux terminal emulator instead of the ugly and boring cmd.exe
        
If you have these programs installed, at best altogether in C:/SDK
you have to adapt _dev_with_gitbash.bat to your installation paths.

Then u should adapt the QtSDK directory in the root CMakeLists.txt
because the recipe copies the runtime dlls from there into your output dir 
when building, also copies Qt dlls from there if you enable BENNI_USE_QT 1.

Then adapt QtSDK directory in the build scripts you want to use, e.g.
   - build_win_release_shared.sh
   - build_win_release_static.sh
   - build_win_debug_static.sh
   - etc...
   
This is necessary so QtCreator finds the Qt libraries (programmaticly) when using CMake.

If you did these additional steps u are good to go,
doubleclick on _dev_with_gitbash.bat to start your development terminal session.

in the now colored terminal promt u can call scripts like
   - ./clean_all.sh which deletes bin + build directories.
   - ./build_win_release_shared.sh
   - ./build_win_release_static.sh
   - ./build_win_debug_static.sh
   - etc...
   
You can ofcourse call cmake yourself, e.g.

   PWD="$(pwd)"
   BUILD=$PWD/build/win64_Release_static
   mkdir -p $BUILD
   cd $BUILD
   cmake \
      -G "MinGW Makefiles" \
      -DBUILD_STATIC=1 \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_PREFIX_PATH=C:/SDK/Qt/5.12.3/mingw73_64/lib/cmake \
      ../../
   mingw32-make -j16


Encountered Problems:

- If QtCreator opens the project, but only shows the nasm assembler project, 
  then u have to call ./build_release_shared.sh atleast once from terminal.
  QtCreator opens the project in release shared mode as default i guess so it expects
  a built assembler before building the rest of project, aka libs and apps.

  This "problem", an additional build step due to assembler, is disabled when set(BENNI_USE_NASM 0), 
  but audio libs need the assembler to build fast code and JUCE is an audio lib.

  If QtCreators opens the project in another config <release|debug|minsizrel,relwithdebinfos>_<shared|static>
  and only shows assembler project, then u should call the corresponding build 
  script atleast once to build the assembler and enable the second phase (libs+apps)

- If it wont compile because of sh.exe, just compress it (backup) and delete the sh.exe. 
  My stuff uses bash.exe. sh.exe was encountered in git4windows if i remember correctly.

- Dont use -G "MSYS Makefiles", years ago i used msys's make instead of mingw32-make.
  Its not needed anymore. git4windows uses msys2 if im correct to create the env for terminal.

- Dont install Strawberry Perl in QtSDK setup, because it brings an ancient gcc 4-6 compiler that
  pollutes your PATH variable and destroys your development env. 

- Some MinGW headers still suck and i had to disable webcam recording and playback in juce_video (yet).
  There was a __UUIDOF nested template bla error i couldnt fix yet, because i did not understand.

Pros:

But all in all with MinGW you have a free open source Linux compiler with whom u can develop
full blown Windows64 multimedia apps ( Audio, Video, OpenGL, COM, OLE ) on Windows64 
but without needing any Microsoft tool and having a more consistent/platform independent code base.

Visual Studio is the worst for a graphics and library developer like my self.
Just look at the tiny single line where i shall add 100 external libraries to.
And this shit exists since 1998 without any progress!

Also the /xxxx command line options with numbers instead of readable names. 
Am i a warning database encyplop or a human beeing?

No doubled runtimes /MD, /MDd, /MT, /MTd. Just one for release and one for debug mode.
That alone increases productivity by 100%

QtCreator is not the best IDE in the world but very good enough. It has clang annotations now.
That actually helped me alot while porting JUCE to MinGW.

The terminal is 10000 times better than cmd.exe. 
Windows people are always surprised when they see a colored terminal, 
even though ANSI colors exist since 1970!
But cmd.exe is not an ANSI terminal and we're all damaged because of this shit terminal.

With CMake your projects are also platform independent, not only your source code.

More pros:

No MFC, ATL, UWP, MSVCRT 100 bis 666.

Why this MinGW port is better than original JUCE:
---------------------------------------------------

I really dont know how your JUCE projects ever compiled on Windows before.
JUCE did pollute the global namespace with Windows.h and other headers.

I got 'ambigous' errors when overriding LookAndFeel and using juce::Rectangle<int>
because Windows.h already defines a function called Rectangle().

This error alone made the entire port necessary.

After banning windows.h stuff from public inclusion it also compiles a bit faster.
Doesnt mean that this port compiles faster than JUCE superbuild/unitybuild structure.

That structure is gone so developers can much better navigate classes in project tree.
If JUCE is built than rebuilding your project should be faster than before.

Until you change a define/setting in the libs.

Every JUCE module got a config file. Every unit of the module includes this config
and at the end of the module is the old module header file that includes all units
at once for convenience.

TODOs:
---------------------------------------------------
 - Testing, testing, testing...
 - Check if SSE2 and AVX are working, 
   i guess not since i did not add compile options for them (yet)
 - Make Direct2D work? d.k. yet OpenGL is better, 
   but juce_graphics has no GL context?
 - Add more free VST plugin projects to make the porting time worth it.
 - Add and enable AU,VST3,LV2,etc.. plugin stuff.
 - Support Linux and Mac. This is purely a Windows port yet. 
   But Windows is the audio dll land, foremost VST Plugin land.
 - Make the dev env setup into a convenient installer
   but im not allowed to share QtSDK 2GB installer.
   