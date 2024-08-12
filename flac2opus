#!/usr/bin/env bash

[[ -x "$(command -v opusenc)" ]] || { echo "opusenc not found. Please install opus-tools."; exit 1; }

# Refer https://wiki.xiph.org/Opus_Recommended_Settings
bitrate=128
flac_dir="$1"
opus_dir="$2"
p=$(nproc --all 2>/dev/null || echo 1)

encode() {
  stub=${1%.*} 
  if [[ ! -f "$4/$stub.opus" ]]; then
    opusenc --bitrate "$2" "$3/$stub.flac" "$4/$stub.opus"
  else
    echo "$stub.opus already exists. Skipping encoding"
  fi
} 
export -f encode

# Recreate file structure
find "$flac_dir" -type d -printf "$opus_dir/%P\0" | xargs -r0 mkdir -p

# Encode
find "$flac_dir" -type f -iname '*.flac' -printf "%P\0" |
  xargs -r0P "$p" -I{} bash -c 'encode "$@"' _ {} "$bitrate" "$flac_dir" "$opus_dir"

# Copy Album Covers & lrc files
find "$flac_dir" -type f \( -iregex '.*\(cover\|folder\)\.\(png\|jpg\|gif\)' -o -iname '*.lrc' -o -iname '*.pdf' \) -printf "$flac_dir/%P\0$opus_dir/%P\0" |
  xargs -r0P "$p" -n2 cp -vu 
