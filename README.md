# FlexASIO README
Author: Etienne Dechamps <etienne@edechamps.fr>
Website: https://github.com/dechamps/FlexASIO
License: General Public License, version 3

**If you are looking for an installer, see the
[GitHub releases page](https://github.com/dechamps/FlexASIO/releases).**

## DESCRIPTION

Background: http://en.wikipedia.org/wiki/Audio_Stream_Input/Output

FlexASIO is an universal ASIO driver, meaning that it is not tied to
specific audio hardware. Other examples of universal ASIO drivers
include ASIO4ALL, ASIO2KS, ASIO2WASAPI. Universal ASIO drivers use
hardware-agnostic audio interfaces provided by the operating system to
produce and consume sound. The typical use case for such a driver is
to make ASIO usable with audio hardware that doesn't come with its own
ASIO drivers.

While ASIO4ALL and ASIO2KS use a low-level Windows audio API known as
"WDM-KS" (also called "DirectKS", "Kernel Streaming") to operate, and
ASIO2WASAPI uses WASAPI (in exclusive mode), FlexASIO uses an
intermediate library called PortAudio that itself supports a large
number of operating system sound APIs, including MME, DirectSound,
WDM-KS, as well as the modern WASAPI interface that was released with
Windows Vista (ironically, PortAudio can use ASIO as well, nicely
closing the circle). Thus FlexASIO can be used to interface with any
sound API available with your system.

PortAudio: http://www.portaudio.com/

In particular, FlexASIO makes it possible to open an audio device in so-called
"shared" mode like any other application, which makes it an interesting
alternative to ASIO4ALL/ASIO2KS/ASIO2WASAPI, as these drivers always open
devices in exclusive mode without giving you a choice.

FlexASIO should be able to run on any version of Microsoft Windows,
even very old ones, at least in theory. It is compatible with 32-bit and
64-bit ASIO host applications.

## HOW TO USE

Just install it and FlexASIO should magically appear in the ASIO
driver list in any ASIO-enabled application. There is no graphical configuration
interface (see "caveats" below), but there is a way to change some FlexASIO
settings through a configuration file; see [CONFIGURATION](CONFIGURATION.md)
for details.

To uninstall FlexASIO, just use the Windows "add/remove programs"
control panel.

If you don't want to use the installer, you can install it manually by
simply registering the DLL:

    regsvr32 FlexASIO_x64.dll
    regsvr32 FlexASIO_x86.dll

Use the `/u` switch to unregister.

## LIMITATIONS AND CAVEATS

This an early version of FlexASIO. It has not been tested in any extensive way,
and is certainly not free from bugs and crashes.

FlexASIO doesn't come with a graphical configuration interface ("control
panel" in ASIO terminology). The main reason is because programming GUIs
takes a lot of time that I don't have (especially since I have zero
experience in GUI programming). This means that you are forced to use
FlexASIO defaults when it comes to device selection, and buffer size. These
defaults are as follows:
 - FlexASIO selects the default audio devices as configured in the
   Windows audio control panel.
 - Preferred buffer size is hardcoded to 1024 samples (21.3 ms at
   48000Hz). This is purely arbitrary.

However, FlexASIO does come with a [configuration file](CONFIGURATION.md) that
can be used to customize a few basic settings, such as the audio backend used
(WASAPI, DirectSound, etc.). Like most of FlexASIO, this is very much a work in
progress and more options will be added over time.

If you are using different hardware devices for input and output, each
with its own hardware clock, you are likely to end up with glitches
sooner or later during playback. How soon depends on the amount of
clock drift between the two hardware devices. Note that this is
basically a fact of life and is a problem with all audio APIs and
drivers; the only way around it is to compensate the clock dift on the
fly using sample rate conversion, but that's much more complicated.

WASAPI in "shared" mode (at least on my test system) seems to require that the
sample rate used by the application matches the sample rate configured for the
device (which is configurable in the Windows audio control panel).
Corollary: if you use WASAPI Shared, your input device's sample rate need to
match the output device's, else FlexASIO will not return any usable sample
rates. In some ways this could be considered a feature since it guarantees that
no unwanted sample rate conversions will take place.

FlexASIO has not been designed with latency in mind. That being said,
the current version should not add any latency on top of PortAudio
itself. The thing is, due to the way ASIO works (static buffer sizes),
PortAudio sometimes has no choice but to add additional buffering (which
adds latency) in order to meet the requirements of both FlexASIO and
the system API it's using.

If you are using a backend other than WASAPI, FlexASIO will be unable to display
the channel names (i.e. "Surround Left", etc.) in the channel list. That's
a limitation of PortAudio.

FlexASIO is Windows-only for now. That could change in the future, as
PortAudio itself is cross-platform.

## TROUBLESHOOTING

### Logging

FlexASIO includes a logging system that describes everything that is
happening within the driver in an excruciating amount of detail. It is
especially useful for troubleshooting driver initialization failures and
other issues. It can also be used for verification (e.g. to double-check
that FlexASIO is using the device and audio format that you expect).

To enable logging, simply create an empty file (e.g. with Notepad) named
`FlexASIO.log` directly under your user directory (e.g.
`C:\Users\Your Name Here\FlexASIO.log`). Then restart your ASIO host
application. FlexASIO will notice the presence of the file and start
logging to it.

Note that the contents of the log file are intended for consumption by
developers. That said, grave errors should stick out in an obvious way
(especially if you look towards the end of the log). If you are having
trouble interpreting the contents of the log, feel free to ask for help
by opening an issue (see "Reporting Issues", below).

*Do not forget to remove the logfile once you're done with it* (or move
it elsewhere). Indeed, logging slows down FlexASIO, which can lead to
buffer underruns (audio glitches). The logfile can also grow to a very
large size over time.

### Test program

FlexASIO includes a rudimentary self-test program that can help diagnose
issues in some cases. It attempts to emulate what a basic ASIO host
application would do in a controlled, easily reproducible environment.

The program is called `FlexASIOTest.exe` and can be found in the `x64`
(64-bit) or `x86` (32-bit) subfolder in the FlexASIO installation
folder. It is a console program that should be run from the command
line.

It is a good idea to have logging enabled while running the test (see
above).

Note that a successful test run does not necessarily mean FlexASIO is
not at fault. Indeed it might be that the ASIO host application that
you're using is triggering a pathological case in FlexASIO. If you
suspect that's the case, please feel free to ask for help (see below).

## REPORTING ISSUES

Just use the
[GitHub issue tracker](https://github.com/dechamps/FlexASIO/issues).
When asking for help, it is strongly recommended to produce a log (see
above) while the problem is occuring, and attach it to your report. The
output of `FlexASIOTest` (see above), along with its log output, might
also help.

## DEVELOPER INFORMATION

FlexASIO is designed to use the Microsoft Visual C++ 2017 toolchain.

You will need to provide the PortAudio library dependency. The best way
is to use [vcpkg](https://github.com/Microsoft/vcpkg):

```
vcpkg install portaudio:x64-windows portaudio:x86-windows
```

Note that, at the time of writing, the portaudio port in current vcpkg
master has a number of issues; make sure you have the following vcpkg
patches before running the above command:

 - [Add pa_win_waveformat.h to public includes](https://github.com/Microsoft/vcpkg/pull/4582)
 - [Copy PDB files](https://github.com/Microsoft/vcpkg/pull/4583)
 - [Enable debug output](https://github.com/Microsoft/vcpkg/pull/4592)

There is also a dependency on the
[tinytoml](https://github.com/mayah/tinytoml) library:

```
vcpkg install tinytoml:x64-windows tinytoml:x86-windows
```

You will also need to provide the ASIO SDK.
[Download](http://www.steinberg.net/en/company/developer.html) the SDK
and copy the ASIOSDK2.3.1 folder to the FlexASIO folder.

The installer can be built using
[Inno Setup](http://www.jrsoftware.org/isdl.php).
