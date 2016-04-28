izSynth
==================
TTS/Text To Speech synthesizer, background music overlay assembler and audio file converter for PBX and Home Automation Systems

## Description
izSynth is a bash script running under Linux, written to automate the synthesis of voices used into izPBX System or for realtime TTS (Text To Speech) used into Home Automation solutions.
It can use offline synthesis software like Loquendo (using the Wine environment), eSpeak, Festival, VoiceRSS, NaturalReaders, and other popular online web TTS services to synthesize audio voices from ASCII text files and automatically merging the audio with background music (mp3 and wav format are supported), adding silences and fade in and out.


For the official site and latest stable version: http://www.initzero.it/products/opensource/izsynth

Follow and contribute the development on GitHub: https://github.com/ugoviti/izsynth

## Key Features
* Install and use within 60 seconds
* Synth and download the voice the first time, and reuse it every time without downloading it again (time and bandwidth consuming saver)
* izSynth can use local engine or web based engines (list supported engines with `izsynth -L`. NB. some engines need an APYKEY to works)
* izSynth support many languages (list supported languages for the given engine with `izsynth -l`)
* izSynth can detach the process to give responsiveness to the shell prompt (use **-b** option to disable/enable)
* Discover all features with `izsynth -h`

## INSTALL

### Install OS Dependencies

Before installing izsynth, install the dependencies for your distribution:

#### For CentOS:
`yum install -y epel-release`
`yum install -y sox lame mplayer curl espeak wine`

#### For Debian based distros (like Raspian on RaspberryPi)
`sudo apt-get update`
`sudo apt-get install -y sox lame mplayer curl espeak wget bsdmainutils file`

For windows sapi5 support, install the `wine` package and download sapi2wav.exe:

`wget https://gitlab.mister-muffin.de/josch/novel2audio/raw/master/sapi2wav.exe -O /usr/local/bin/sapi2wav.exe`

### Install izSynth

**Stable Release**

Download latest stable version of izSynth package from http://www.initzero.it/products/opensource/izsynth and uncrompress it into a system binary PATH, ex. `/usr/local/bin`

**Development Release**

`wget https://raw.githubusercontent.com/ugoviti/izsynth/master/izsynth -O /usr/local/bin/izsynth && chmod 755 /usr/local/bin/izsynth`

## USAGE
### Command line

Before using izsynth, we must configure and try it via command line.

Make a quick test with the following command:

`izsynth -e voicerss -v en-gb -t "Welcome home, sir Stark"`

**NB.** Some engines, like **voicerss**, need an **APY KEY** before you can use it, follow the onscreen guide to get your APY KEY:

`izsynth -e voicerss -H`

**izSynth is configurable in 3 ways:**

* From command line specifying the options
* Modifying the variables on the izsynth script itself
* Using an external config file for overriding the izsynth variables.


I suggest to create an external config file to avoid changes to your configurations when you update izsynth.

create the following config file: `$HOME/.config/izsynth/izsynth.conf`

**NB.** if the `$HOME/.config/izsynth` doesn't exist, izsynth will make it on the first run

```
# ENGINE SPECIFIC API KEYS
VOICERSS_APIKEY=your_apy_key_hash_go_here

# default tts engine
TTS_ENGINE="voicerss"
# default tts voice (null = let tts engine to set default voice. it will use english)
TTS_VOICE="en-gb"

# realtime audio playback variables
PLAYBACK="yes"
# play the file in background, otherwise foreground
PLAYBACK_BACKGROUND="yes"
# resynth the file if it already exist
PLAYBACK_RESYNTH="no"
# sound card playback volume
#PLAYBACK_VOLUME="30"
# command used to play the synthehtized audio file
PLAYBACK_COMMAND="mplayer -ao alsa -really-quiet -nolirc -noconsolecontrols"

# default base temp directory (comment if you want use the system default directory base, ex. /tmp)
TMP_DIR_BASE="/dev/shm"

# default redirect to tmp dir
OUT_DIR="$TMP_DIR_BASE"

# default tts volume
#TTS_VOLUME="0.4"

# default wait n. seconds before speaking tts
TTS_PAD_BEGIN="5"

# default wait n. seconds after speaked tts
TTS_PAD_END="5"

# default start music after n. seconds
MUSIC_START="0"

# default fade the start and end of music with 5 seconds
MUSIC_FADE="5"

# default music volume
#MUSIC_VOLUME="0.1"
```

**REMEMBER: the command line options win always against the config file**

Now let's try the various izsynth options available.
izsynth allow for example to put a background music in every synthesized file, or you can mix various languages and engines in one (MEGAMIX feature)... use the command `izsynth -E` to discover.

## izSynth Tips&Tricks

**Make izsynth say the current time without writing complex scripts:**

`izsynth -t "it's $(date +%H:%M)"`

----

**If the soundcard output volume is too low:**

Use the **-W** switch to change the output volume of the hardware sound card:

`script://izsynth -W 90 -t "Can you hear me now?"`

If you found the correct audio volume, I suggest to modify the config file, so you don't need to specify the -W switch every time:

`PLAYBACK_VOLUME="90"`

----

