# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

izSynth is a bash script-based TTS (Text-to-Speech) synthesizer designed for PBX and Home Automation systems. It synthesizes voice from text, overlays background music, and converts audio files to various telephony formats.

**Key characteristics:**
- Single-file bash script architecture (~2100 lines)
- Plugin-based TTS and playback engine system
- Supports both online and offline TTS engines
- Audio processing pipeline using sox, lame, and other CLI tools

## Architecture

### Plugin System

The script uses a function-based plugin architecture with two main types:

**TTS Engines** (lines 115-784):
- Each engine is a bash function: `tts_engine_<name>()`
- Supported actions: `description`, `help`, `env`, `commands`, `voices`, `voice`, `synthesize`
- Built-in engines: ispeech, google-translate, gemini-tts, voicerss, naturalreaders, fromtexttospeech, festival, espeak, sapi5
- External engines can be added via `$HOME/.config/izsynth/engines/*.conf`

**Playback Engines** (lines 788-868):
- Each engine is a bash function: `playback_engine_<name>()`
- Supported actions: `description`, `help`, `env`, `commands`, `play`
- Built-in engines: generic, mplayer

### Core Processing Pipeline

1. **Input Processing**: Text string, file, or directory
2. **Synthesis** (`synthesize_file()`): Convert text to audio using selected TTS engine
3. **Assembly** (`assemble_file()`): Add padding, background music, fade effects
4. **Output**: Either playback or export to PBX formats

### Key Functions

- `main()` (line 1595): Main entry point and orchestration
- `assemble_file()` (line 1293): Core audio processing - combines TTS, music, padding
- `synthesize_file()` (line 1105): TTS engine interface and caching
- `export_file()` (line 1200): Convert to telephony formats (gsm, ulaw, alaw, slin, etc.)
- `megamix_file()` (line 1287): Combine multiple TTS files with different engines/languages
- `split_tts_file()` (line 1373): Auto-split text exceeding engine character limits

## Configuration

**Config file location**: `$HOME/.config/izsynth/izsynth.conf`

Configuration priority (highest to lowest):
1. Command-line options
2. Config file variables
3. Script default variables (lines 14-62)

**Critical variables:**
- `TTS_ENGINE`: Default engine (naturalreaders)
- `TTS_VOICE`: Default voice (Peter)
- `TTS_ENGINE_*_APIKEY`: API keys for cloud services
- `EXPORT_AUDIO_FORMATS`: Target formats for PBX export (default: "44khz.wav wav gsm ulaw alaw slin")
- `PLAYBACK_*`: Playback configuration

## Development Commands

### Testing TTS Synthesis
```bash
# Quick test with default engine
./izsynth -t "Test message"

# Test specific engine and voice
./izsynth -e naturalreaders -v Peter -t "Welcome home"

# Test with background music
./izsynth -e google -v en -m /path/music.mp3 -p 3 -P 2 -F 2 -t "Test"

# Export to PBX formats instead of playback
./izsynth -i input.txt -o /tmp/output -x
```

### Listing Available Options
```bash
# List all TTS engines
./izsynth -L

# List voices for specific engine
./izsynth -e voicerss -l

# Show usage examples
./izsynth -E

# Generate default config file
./izsynth -C

# List playback devices
./izsynth -D
```

### Development Workflow
```bash
# Make script executable
chmod +x izsynth

# Test changes with verbose output and forced resynth
./izsynth -e naturalreaders -v Peter -t "Test changes" -w

# Test batch processing
./izsynth -i /tmp/test-dir -o /tmp/output -x
```

## Important Implementation Details

### Audio Format Handling
- Intermediate format: 44.1kHz WAV
- sox handles all audio manipulation (padding, mixing, fade, volume)
- PBX formats converted from 44khz.wav using sox's rate/encoding options

### Caching System
- Synthesized files are cached by default (skip re-download)
- Use `-w` flag to force overwrite/resynth
- Cache location: `$OUT_DIR` (default: `/dev/shm`)

### Text Splitting
- Engines have character limits (stored in `TTS_LENGTH` variable)
- `split_tts_file()` automatically splits long text at sentence boundaries
- Split files are synthesized separately then concatenated

### MEGAMIX Feature
- Allows mixing multiple engines/languages in one output
- Input format: `-i engine:voice:file.txt`
- Example: `-i naturalreaders:Luca:it.txt -i voicerss:en-gb:en.txt`

### Gemini TTS Engine (gemini-tts)
**New in 2025**: High-quality cloud TTS with Neural2 and Chirp3 voices.

**Setup:**
1. Create Google Cloud project at console.cloud.google.com
2. Enable Cloud Text-to-Speech API
3. Create API key in: APIs & Services > Credentials
4. Add to config: `TTS_ENGINE_GEMINI_TTS_APIKEY="your-api-key"`

**Features:**
- Output format: MP3 (efficient, high quality)
- Character limit: 5,000 bytes per request
- Dynamic voice listing via API (requires `jq` or falls back to grep/sed)
- Supports Neural2, Chirp3, and Studio voices
- Base64 JSON response processing

**Usage:**
```bash
# Show help and setup instructions
./izsynth -e gemini-tts -H

# List all available voices (requires API key)
./izsynth -e gemini-tts -l

# Synthesize Italian text with Chirp3 voice
./izsynth -e gemini-tts -v it-IT-Chirp3-HD-Leda -t "Benvenuto!"

# English with Neural2 voice
./izsynth -e gemini-tts -v en-US-Neural2-C -t "Hello world"
```

**Implementation notes:**
- Uses Google Cloud TTS API v1 (not beta)
- Requires `base64` command for audio decoding
- Optional `jq` for better JSON parsing (falls back to grep/sed)
- Automatic language code extraction from voice name
- Error handling for API failures with response display

## Common Modification Patterns

### Adding a New TTS Engine
1. Create function `tts_engine_yourengine()` following the pattern in lines 118-783
2. Implement required actions: `description`, `help`, `env`, `commands`, `voices`, `voice`, `synthesize`
3. Set `TTS_EXT`, `TTS_LENGTH`, default `TTS_LANGUAGE` and `TTS_VOICE` in `env` action
4. Implement `synthesize` action to download/generate audio to `$OUT` file

### Adding Audio Format Support
1. Add format to `EXPORT_AUDIO_FORMATS` variable (line 40)
2. Add conversion case in `export_file()` function (around line 1200)
3. Use sox for format conversion with appropriate encoding/rate parameters

### Modifying Audio Processing
- Audio assembly happens in `assemble_file()` (line 1293)
- Uses sox for all operations: padding, volume adjustment, music mixing, fade effects
- Check sox version compatibility in `check_ver()` (line 1368)

## Dependencies

**Required commands**: curl, md5sum, file, iconv, soxi, sox, lame

**Optional (per engine/playback)**:
- espeak: offline TTS engine
- festival: offline TTS engine
- wine + sapi2wav.exe: Windows SAPI5 voices
- mplayer: audio playback

## License

GNU Lesser General Public License v3 (LGPL-3.0)
