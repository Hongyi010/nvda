# Espeak-ng submodule

The submodule contained in the espeak directory is a cross platform open source speech synthesizer.

## Building
NVDA has a custom build of ESpeak because not all components are required.

### Background
The main authority on build requirements should be `<nvda repo root>/include/espeak/Makefile.am`.
The `*.vcxproj` files in `<nvda repo root>/include/espeak/src/windows/` can also be considered,
however these are not always kept up to date.

We don't use the auto make files or the visual studio files, we maintain our own method of building espeak.
Modifications will need to be made in `<nvda repo root>/nvdaHelper/espeak`
* `sconscript` for the build process.
* `config.h` to set the eSpeak-ng version that NVDA outputs to the log file.

### Updating the version used by NVDA

1. Start from a clean branch off of NVDA `master`
   1. Check out the latest NVDA `origin/master` and create a new branch.
   1. Do a git clean to ensure the working directory is clean.
1. Ensure submodules are up to date
   1. Synchronize submodules with `git submodule sync`
   1. Update submodules with `git submodule update --init --recursive`
1. Checkout the new eSpeak-ng revision to try.
   1. Change to the `include/espeak/` directory
   1. Do `git fetch` to get the latest from the espeak-ng repo
   1. Do `git checkout origin/master` or whichever espeak-ng ref you wish.
1. Look at changes in `Makefile.am` and update our build.
   1. Diff `Makefile.am` with the previously used commit of espeak.
   1. Changes to Dictionary compilation should be reflected in `espeakDictionaryCompileList`
   1. Some modules are intentionally excluded from the build.
      If unsure, err on the side of including it and raise it as a question when submitting a PR.
   1. Modify the `<nvda repo root>/nvdaHelper/espeak/config.h` file as required.
1. Update our record of the version number and build.
   1. Change back to the NVDA repo root
   1. Update the `/DPACKAGE_VERSION` in `espeak/sconscript`
      - The preprocessor definition is used to supply these definitions instead of `<nvda repo root>/nvdaHelper/espeak/config.h`
      - `<nvda repo root>/nvdaHelper/espeak/config.h` must exist (despite being empty) since a "config.h" is included within espeak.
      - Compare to espeak source info: `<nvda repo root>/include/espeak/src/windows/config.h`.
   1. Update NVDA `readme.md` with espeak version and commit.
   1. Build NVDA
1. Run NVDA (set eSpeak-ng as the synthesizer) and test.
1. Ensure that the log file contains the new version number for eSpeak-NG

### Troubleshooting Build failures

If python crashes while building, check the log.
If the last thing is compiling some dictionary try excluding it.
This can be done in `<nvda repo root>/nvdaHelper/espeak/sconscript`.
Remember to report this to the eSpeak-ng project.

If the build fails, take note of the error, compare the diff of the `Makefile.am` file and mirror 
any changes in our `sconscript` file.

### Known issues
Due to problems with emoji support (causing crashes), emoji dictionary files are being excluded
from the build, they are deleted prior to compiling the dictionaries in the
`<nvda repo root>/nvdaHelper/espeak/sconscript` file.

## Manually testing Espeak-ng
If you wish to test eSpeak-ng directly, perhaps to create steps to reproduce when raising an issue for the eSpeak-ng project, consider feeding SSML directly to the executable provided by the project.

The [espeak docs](https://github.com/espeak-ng/espeak-ng/blob/master/docs/index.md) are worth reading.
They describe the various [(espeak-ng) command line arguments](https://github.com/espeak-ng/espeak-ng/blob/master/src/espeak-ng.1.ronn) (note also [speak-ng command line](https://github.com/espeak-ng/espeak-ng/blob/master/src/speak-ng.1.ronn)), and [insturctions to build](https://github.com/espeak-ng/espeak-ng/blob/master/docs/building.md#windows) an `.exe` from a commit of eSpeak, locally on Windows.
However, historically the Windows build for espeak-ng hasn't been well maintained, with periods of build failures.
It is also different from the build approach within NVDA.


1. Install an (x86) release: https://github.com/espeak-ng/espeak-ng/releases
   `espeak.exe` is in `C:/Program Files (x86)/espeak-ng/`
   Adding it to your path temporarily will make the following easier.
1. Write out [some SSML](https://github.com/espeak-ng/espeak-ng/blob/master/docs/markup.md) that demonstrates the problem.
   ```sh
   $ cat /c/work/test-espeak-ssml.txt
   <prosody pitch="+100">I am now speaking at an extra high pitch.</prosody>
   <break time="1s"/>
   <prosody pitch="1">I am now speaking at the default pitch.</prosody>
   ```
1. Feed the SSML into `espeak-ng.exe`
   ```sh
   $ cat /c/work/test-espeak-ssml.txt | ./espeak-ng.exe --stdin -m
   ```
