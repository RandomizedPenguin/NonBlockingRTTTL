![NonBlockingRTTTL logo](https://raw.githubusercontent.com/end2endzone/NonBlockingRTTTL/master/docs/NonBlickingRtttl-splashscreen.png)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Github Releases](https://img.shields.io/github/release/end2endzone/NonBlockingRTTTL.svg)](https://github.com/end2endzone/NonBlockingRTTTL/releases)
[![Build status](https://ci.appveyor.com/api/projects/status/wrsctc8vu3f3lq5o/branch/master?svg=true)](https://ci.appveyor.com/project/end2endzone/NonBlockingRTTTL/branch/master)
[![Tests status](https://img.shields.io/appveyor/tests/end2endzone/NonBlockingRTTTL/master.svg)](https://ci.appveyor.com/project/end2endzone/NonBlockingRTTTL/branch/master/tests)

AppVeyor build statistics:

[![Build statistics](https://buildstats.info/appveyor/chart/end2endzone/NonBlockingRTTTL)](https://ci.appveyor.com/project/end2endzone/NonBlockingRTTTL/branch/master)


# NonBlockingRTTTL

NonBlockingRTTTL is a non-blocking arduino library for playing RTTTL melodies.

It's main features are:

*  Fully support the RTTTL format standard.
*  Support unofficial frequencies and tempo/beats per minutes
*  Really small increase in memory & code footprint compared to the usual blocking algorithm.
*  Allows your program to read/write IOs pins while playing. Implementing "stop" or "next song" push buttons is really easy!

# Purpose

Most code that can be found on the internet that "play" an RTTTL string is build this way: sequential calls to the `tone()` function followed by a call to the `delay()` function:

```cpp
tone(pin, note1, duration1);
delay(note_delay);
tone(pin, note2, duration2);
delay(note_delay);
tone(pin, note3, duration3);
delay(note_delay);
//etc...
```

This type of implementation might be good for robots but not for realtime application or projects that needs to monitor pins while the song is playing.

This library is non-blocking which make it suitable to be used by more advanced algorithm. The non-blocking RTTTL library is a port of the RTTTL example from the [Tone library](http://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/rogue-code/Arduino-Library-Tone.zip).

# RTTTL format

The Ring Tone Text Transfer Language (RTTTL) audio format is an audio format for storing single tone (monolithic) melodies. Each melody is composed of successive tone frequencies.

The RTTTL format is [human readable](http://stackoverflow.com/questions/568671/why-should-i-use-a-human-readable-file-format) and usually more compressed than note & duration arrays which helps reduce its memory footprint.

The format is really suitable for embedded device that are limited in memory which can’t store PCM (wav) or even MP3 data.

*Note that RTTTL can also be spelled RTTL (Ringtone Text Transfer Language). According to Samsung phones, a ringtone can also be spelled as a single word...*

More information on the RTTTL format is available on its [Wikipedia acticle](https://en.wikipedia.org/wiki/Ring_Tone_Transfer_Language).

# Usage

Call `rtttl::begin()` to setup the non-blocking RTTTL library. Then call `rtttl::play()` to update the library’s state and play notes as required.

Use `rtttl::done()` or `rtttl::isPlaying()` to know if the library is done playing the given song. Anytime playing, one can call `rtttl::stop()` to stop playing the current song.

Define `RTTTL_NONBLOCKING_INFO` to enable the debugging of the library state on the serial port. Use `NONBLOCKINGRTTTL_VERSION` to read the current version of the library.


# Example

### Play melody:
The following example plays 3 melodies. The first melody is played for 5 seconds and stopped. The second and third melodies and played completely and then the program restart again.

```cpp
#include <NonBlockingRtttl.h>
 
//project's contants
#define BUZZER_PIN 8
const char * tetris = "tetris:d=4,o=5,b=160:e6,8b,8c6,8d6,16e6,16d6,8c6,8b,a,8a,8c6,e6,8d6,8c6,b,8b,8c6,d6,e6,c6,a,2a,8p,d6,8f6,a6,8g6,8f6,e6,8e6,8c6,e6,8d6,8c6,b,8b,8c6,d6,e6,c6,a,a";
const char * arkanoid = "Arkanoid:d=4,o=5,b=140:8g6,16p,16g.6,2a#6,32p,8a6,8g6,8f6,8a6,2g6";
const char * mario = "mario:d=4,o=5,b=100:16e6,16e6,32p,8e6,16c6,8e6,8g6,8p,8g,8p,8c6,16p,8g,16p,8e,16p,8a,8b,16a#,8a,16g.,16e6,16g6,8a6,16f6,8g6,8e6,16c6,16d6,8b,16p,8c6,16p,8g,16p,8e,16p,8a,8b,16a#,8a,16g.,16e6,16g6,8a6,16f6,8g6,8e6,16c6,16d6,8b,8p,16g6,16f#6,16f6,16d#6,16p,16e6,16p,16g#,16a,16c6,16p,16a,16c6,16d6,8p,16g6,16f#6,16f6,16d#6,16p,16e6,16p,16c7,16p,16c7,16c7,p,16g6,16f#6,16f6,16d#6,16p,16e6,16p,16g#,16a,16c6,16p,16a,16c6,16d6,8p,16d#6,8p,16d6,8p,16c6";
byte songIndex = 0; //which song to play when the previous one finishes
 
void setup() {
  pinMode(BUZZER_PIN, OUTPUT);
 
  Serial.begin(115200);
  Serial.println();
}
 
void loop() {
  if ( !rtttl::isPlaying() )
  {
    if (songIndex == 0)
    {
      rtttl::begin(BUZZER_PIN, mario);
      songIndex++; //ready for next song
 
      //play for 5 sec then stop.
      //note: this is a blocking code section
      //use to demonstrate the use of stop()
      unsigned long start = millis();
      while( millis() - start < 5000 ) 
      {
        rtttl::play();
      }
      rtttl::stop();
      
    }
    else if (songIndex == 1)
    {
      rtttl::begin(BUZZER_PIN, arkanoid);
      songIndex++; //ready for next song
    }
    else if (songIndex == 2)
    {
      rtttl::begin(BUZZER_PIN, tetris);
      songIndex++; //ready for next song
    }
  }
  else
  {
    rtttl::play();
  }
}
```

# Installing

Please refer to file [INSTALL.md](INSTALL.md) for details on how installing/building the application.

## Testing
NonBlockingRTTTL comes with unit tests which tests for multiple combinations to validate the integrity of the library with multiple operating systems.

Test are build using the Google Test v1.6.0 framework. For more information on how googletest is working, see the [google test documentation primer](https://github.com/google/googletest/blob/release-1.8.0/googletest/docs/V1_6_Primer.md).  

Test are automatically build when building the solution. Please see the '*build step*' section for details on how to build the software.

Test can be executed from the following two locations:

1) From the Visual Studio IDE:
   1) Select the project `NonBlockingRTTTL_unittest` as StartUp project.
   2) Hit CTRL+F5 (Start Without Debugging)
2) From the output binaries folder:
   1) Open a file navigator and browse to the output folder(for example c:\projects\NonBlockingRTTTL\cmake\build\bin\Release)
   2) Run the `NonBlockingRTTTL_unittest.exe` executable.

See also the latest test results at the beginning of the document.

# Versioning

We use [Semantic Versioning 2.0.0](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/end2endzone/NonBlockingRTTTL/tags).

# Authors

* **Antoine Beauchamp** - *Initial work* - [end2endzone](https://github.com/end2endzone)

See also the list of [contributors](https://github.com/end2endzone/NonBlockingRTTTL/blob/master/AUTHORS) who participated in this project.

# License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
