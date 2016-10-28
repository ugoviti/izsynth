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

#### RedHat based distros like CentOS/Fedora
`yum install -y epel-release`
`yum install -y sox lame mplayer curl espeak wine`

#### Debian based distros like Ubuntu or Raspian on RaspberryPi
`sudo apt-get update`
`sudo apt-get install -y sox lame mplayer curl espeak wget bsdmainutils file mawk coreutils`

For windows sapi5 support, install the `wine` package and download sapi2wav.exe:

`wget https://gitlab.mister-muffin.de/josch/novel2audio/raw/master/sapi2wav.exe -O /usr/local/bin/sapi2wav.exe`

### Install izSynth

**Stable Release**

Download latest stable version of izSynth package from http://www.initzero.it/products/opensource/izsynth and uncrompress it into a system binary PATH, ex. `/usr/local/bin`

**Development Release**

`cd /usr/local/bin`
`wget https://raw.githubusercontent.com/ugoviti/izsynth/master/izsynth -O izsynth && chmod 755 izsynth`

### Update izSynth

You can **update** izSynth using the same command you used to  install it.

## USAGE
### Command line

Before using izsynth, we must configure and try it via command line.

Make a quick test with the following command:

`izsynth -e naturalreaders -v Peter -t "Welcome home, mr Stark"`

**NB.** Some engines, like **voicerss**, need an **APY KEY** before you can use it, follow the onscreen guide to get your APY KEY:

`izsynth -e voicerss -H`

**izSynth is configurable in 3 ways:**

* From command line specifying the options ('izsynth -h' to list available options or 'izsynth -E' for example usage)
* Using an external config file for overriding the izsynth variables (suggested method)
* Modifying the variables on the izsynth script itself (not suggested)

I suggest to create an external config file to avoid changes to your configurations when you update izsynth script.

Create the following config file: `$HOME/.config/izsynth/izsynth.conf` using the command:

`$ izsynth -C`

and edit it using your prefered text editor.

**NB.** if the `$HOME/.config/izsynth` directory doesn't exist, izsynth will create it on the first run

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

----

**Use an external device (like bluetooth speaker) as playback output:**

First configure the bluetooth stack and connect you izsynth box to the bluetooth speaker.

Now test the output from command line:

`izsynth -t "I love domoticz" -d alsa:device=bluetooth`

or to another audio card:

`izsynth -t "I love domoticz" -d alsa:device=hw=2.0`

you can list all local usable alsa devices with:

`izsynth -D`

----

**Create voice announcements with background music**

`izsynth -e google -v en -m /path/music.mp3 -p 3 -P 2 -F 2 -t "Welcome home, mr Stark"`

