# UndertalePSP1000
This kit turns YOUR OWN copy of Undertale for PC into a version that runs on a PlayStation Portable (PSP-1000 or later, with custom firmware). It converts the game's files on your computer and builds an EBOOT.PBP that runs them on the PSP.
#######################################################################################################
#                                                                                                     #
#   IMPORTANT: THE GAME IS NOT INCLUDED                                                               #
#                                                                                                     #
#   This kit does NOT contain Undertale or any of its files (data.win, music, graphics or text).      #
#   It only contains conversion tools and the PSP runner (an open-source GameMaker interpreter).      #
#                                                                                                     #
#   You must obtain Undertale LEGALLY on your own, for example by buying it on Steam, GOG or the      #
#   official website (https://undertale.com). The kit only works with the files of a copy you own.   #
#                                                                                                     #
#   Do NOT share the files the kit generates (the "salida" and "trabajo" folders): they contain the   #
#   game's data and are for your personal use only. Undertale is (c) Toby Fox; this project is not    #
#   affiliated with or endorsed by Toby Fox.                                                          #
#                                                                                                     #
#######################################################################################################


CONTENTS
--------
  1. What you need
  2. Getting the game files (legally)
  3. Installing the requirements on your PC
  4. (Optional) WSL: menu music and building the runner yourself
  5. Creating the port
  6. Installing it on the PSP
  7. Controls
  8. What the kit does
  9. Known limitations
 10. Troubleshooting
 11. Credits and licenses


1. WHAT YOU NEED
----------------
  - A PSP (any model, PSP-1000 included) with custom firmware (CFW) able to run homebrew.
  - A Memory Stick with at least 512 MB (the port takes about 230 MB).
  - A Windows 10/11 PC.
  - Undertale for PC, bought legally (see section 2). Tested with version 1.08 (GameMaker WAD 15).
  - Python 3.8 or newer, with the Pillow and numpy modules.
  - ffmpeg.
  - Optional: WSL (Windows Subsystem for Linux) with Ubuntu, to add music to the PSP menu and to
    build the runner from source.


2. GETTING THE GAME FILES (LEGALLY)
-----------------------------------
  Buy Undertale for Windows on Steam, GOG or from the official store, and install it.
  The kit needs the game's installation folder, which contains:
    - data.win              (the game itself; ~55 MB)
    - mus_*.ogg, snd_*.ogg  (music and some sounds; ~90 MB in total)

  Where to find it:
    - Steam: Library > right-click Undertale > Manage > Browse local files.
    - GOG Galaxy: Undertale > Settings (gear icon) > Manage installation > Show folder.

  Always use the ORIGINAL data.win from that folder. The kit checks it and refuses data.win files that
  were already reduced or that come from unsupported versions (it accepts GameMaker WAD 15 and 16).

  The port also works with fan translations that you have applied to your own copy (it was developed
  and tested with a Spanish translation made with TranslaTale).


3. INSTALLING THE REQUIREMENTS ON YOUR PC
-----------------------------------------
  a) Python 3: download it from https://www.python.org/downloads/ and, during installation, tick
     "Add python.exe to PATH". Then open a Command Prompt and run:

        py -m pip install pillow numpy

  b) ffmpeg: download a Windows build from https://ffmpeg.org/download.html, unzip it and either add
     its "bin" folder to the PATH, or copy ffmpeg.exe into this kit's folder. (The kit also looks in
     C:\MPV\ffmpeg.exe. If it can't find ffmpeg, it will ask you where it is.)


4. (OPTIONAL) WSL: MENU MUSIC AND BUILDING THE RUNNER YOURSELF
--------------------------------------------------------------
  Without WSL the kit still works: it uses the prebuilt runner in runner\DATA.PSP, and the game just
  won't have background music in the PSP menu. Set up WSL if you want either of these extras.

  a) Install WSL with Ubuntu (Command Prompt as administrator), then restart the PC:

        wsl --install -d Ubuntu

  b) Menu music (SND0.AT3): build the free ATRAC3 encoder "atracdenc" in Ubuntu:

        sudo apt update
        sudo apt install -y build-essential cmake git libsndfile1-dev
        git clone https://github.com/dcherednik/atracdenc.git ~/atracdenc
        cd ~/atracdenc
        git submodule update --init --recursive
        cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DATRACDENC_PCM_IO_BACKEND=libsndfile
        cmake --build build -j"$(nproc)"

     The kit expects it at ~/atracdenc/build/src/atracdenc.

  c) Building the runner (PSP toolchain "pspdev"): follow https://pspdev.github.io/installation.html
     and install the toolchain into ~/pspdev (the kit expects ~/pspdev/bin/psp-gcc). git is also
     needed (installed in step b).

     If the kit finds pspdev but no Butterscotch source folder, it DOWNLOADS the runner's source
     automatically from https://github.com/ButterscotchRunner/Butterscotch (commit fec280f, the version
     the port is based on) into .\Butterscotch and applies the PSP port to it
     (runner\butterscotch-psp.patch). It doesn't download anything from Undertale.


5. CREATING THE PORT
--------------------
  1. Double-click "Create Port.bat" (English). CREAR_PORT.bat is the same tool in Spanish.
  2. The kit checks the requirements and shows what it found (Python, ffmpeg, WSL, atracdenc, pspdev).
  3. Choose your ORIGINAL data.win (in the game folder, next to the .ogg files). The kit checks it
     and tells you if any music file is missing.
  4. Choose your own images: the wallpaper (PSP menu background, ideally 480x272) and the splash
     (game icon, ideally 144x80). The kit doesn't include any. Both are optional and any size works
     (they are resized automatically). Press Cancel to skip them: there will be no background and
     the PSP shows its generic icon.
  5. If atracdenc is available, choose the menu music: "Fallen Down" by default, or any other track
     (the first 28 seconds are used).
  6. Choose the title shown in the PSP menu (default: Undertale) and the output folder.
  7. If pspdev is available, choose whether to build the runner from source.
  8. Wait 1 to 3 minutes. When it says "Done", the result is in:

        salida\PSP\GAME\UNDERTALE\
            EBOOT.PBP       the runner, with the title, icon, background and menu music
            data.win        the game, reduced to ~12.5 MB
            texturas.utpt   the graphics, converted for the PSP
            SOUNDBNK.BIN    sound index
            SOUNDS.BIN      sound effects and music

  Unattended (no questions), from a Command Prompt in the kit folder:

        "Create Port.bat" --data-win "C:\path\to\Undertale\data.win" [--wallpaper IMAGE] [--splash IMAGE]
                          [--music OGG|none] [--title TEXT] [--output FOLDER] [--compile | --no-compile]

  (The conversion tools themselves print a few short progress lines in Spanish.)


6. INSTALLING IT ON THE PSP
---------------------------
  1. Connect the PSP to the PC (Settings > USB Connection) or put the Memory Stick in a card reader.
  2. Copy the "PSP" folder from "salida" to the ROOT of the Memory Stick, merging it with the existing
     PSP folder. It must end up as:

        <Memory Stick>\PSP\GAME\UNDERTALE\EBOOT.PBP  (and the other 4 files)

  3. On the PSP, go to Game > Memory Stick and start Undertale.

  Your save files (file0, undertale.ini...) are stored in that same folder.


7. CONTROLS
-----------
  D-pad / analog stick   Move
  X (cross)              Confirm               (Z on PC)
  O (circle)             Cancel                (X on PC)
  START or Triangle      Menu                  (C on PC)
  Square                 Cancel / alternative  (Shift on PC)
  Hold SELECT            Quit the game         (Esc on PC)

  The in-game texts that mention keys ([Z], [X], [C] and the instructions screen) are changed to the
  PSP buttons.


8. WHAT THE KIT DOES
--------------------
  - data.win: removes the embedded graphics and sounds (they go into their own files, loaded only when
    needed) and replaces the keyboard keys in the texts with the PSP buttons.
  - Graphics: repacks every texture into 4/8-bit palette pages of up to 512x512 (the PSP's maximum)
    and loads them from the Memory Stick on demand, with a 4 MB cache.
  - Audio: converts sound effects and music to mono 22 kHz 8-bit mu-law, streamed from the Memory
    Stick while it plays.
  - Runner: Butterscotch (an open-source GameMaker interpreter) with a PSP backend: sceGu graphics,
    sceAudio sound, controls, 333 MHz CPU, sleep/resume support.
  - Memory: the whole game fits in about 22 MB, so it runs on a PSP-1000 (32 MB).


9. KNOWN LIMITATIONS
--------------------
  - Music and sound are mono, 22 kHz.
  - Things the game draws into "surfaces" (some special effects) are not shown yet.
  - The screen is 4:3 with black bars on the sides; battles (640x480) are scaled down, so speech
    bubbles and their text are enlarged to stay readable.
  - Moments with many objects on screen (for example when a monster turns to dust) can drop the
    frame rate.
  - The instructions screen is only translated to PSP buttons for the Spanish texts; with the English
    version of the game, that one screen still shows the PC keys (the [Z]/[X]/[C] hints in dialogue
    are changed in any language).
  - Only Undertale for PC with GameMaker WAD 15/16 data.win files is supported (not the console
    versions).


10. TROUBLESHOOTING
-------------------
  "Missing Python modules"       Run: py -m pip install pillow numpy
  ffmpeg not found               Copy ffmpeg.exe into the kit folder or add it to the PATH.
  "has no embedded sounds: use the game's ORIGINAL data.win"
                                 You chose an already reduced data.win (for example the one in "salida").
                                 Choose the one from the game's installation folder.
  "N missing" (music files)      Some .ogg files are not next to data.win. Verify the game files in
                                 Steam/GOG, or continue anyway (those sounds will be silent).
  No music in the PSP menu       atracdenc was not found (section 4b). The game itself still has music.
  The game doesn't appear on the PSP
                                 Check the path PSP\GAME\UNDERTALE\EBOOT.PBP and that the PSP has CFW.
  The game closes or freezes when starting
                                 Make sure all 5 files were copied and that they were generated together
                                 from the same data.win.


11. CREDITS AND LICENSES
------------------------
  - Undertale (c) Toby Fox. NOT included. This project is not affiliated with Toby Fox.
  - Butterscotch, open-source GameMaker runner: https://github.com/ButterscotchRunner/Butterscotch
    License: GNU AGPL-3.0. The PSP runner in runner\DATA.PSP is built from commit
    fec280ff3ba1333f65df1428ba907917860447c4 plus runner\butterscotch-psp.patch, which together are its
    complete source code, under the same license.
  - atracdenc (ATRAC3 encoder, optional): https://github.com/dcherednik/atracdenc - GNU LGPL-2.1.
  - pspdev / PSPSDK (PSP toolchain, optional): https://github.com/pspdev
  - ffmpeg: https://ffmpeg.org
