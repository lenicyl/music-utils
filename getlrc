#!/usr/bin/env bash

usage() {
cat << EOF
Usage: $0 "<Path>"

  Valid entry for <Path> is the path to a directory or an audio file
  If a directory is submitted, all opus,flac and mp3 files in 
  the directory are used to fetch lyrics
  
  This script downloads $(tput bold)synced lrc files$(tput sgr0) for requested music 
  files, saving them in the same directory and with the same 
  filename as the corresponding music file.

  $(tput bold)USES LRCLIB.NET TO FETCH LYRICS 
EOF
}


# Check Pre-requisites
[ ! -x "$(command -v mediainfo)" ] && echo "command not found: mediainfo" && exit 1 
[ ! -x "$(command -v jq)" ] && echo "command not found: jq" && exit 1


get_metadata() {
  inform_pattern="General;get?artist_name=%Performer%&track_name=%Track%&album_name=%Album%&duration="
  metadata=$(mediainfo --Inform="$inform_pattern" "$1")
  metadata=${metadata// /+} # Replaces Spaces with '+'
  metadata=${metadata//’/\'} # Replaces the stupid apostrophe with the un-stupid apostrophe
  get_duration "$1" 
  metadata+="$duration_sec"
}
export -f get_metadata


get_duration() { # TODO: ?Consider replacing with awk in get_metadata itself?
  duration_ms=$(mediainfo --Inform="General;%Duration%" "$1")
  duration_sec=$((duration_ms/1000))
}
export -f get_duration


get_lrc() {
  get_metadata "$1"
  json=$(curl -s "https://lrclib.net/api/$metadata")
  filename=${1##*/} # filename with extension
  fileglob=${filename%.*} # filename, no extension
  filepath=${1%.*} # full path, no file extension

  red=$(tput setaf 1)
  green=$(tput setaf 2)
  gray=$(tput setaf 8)

  if [[ ! -f "$filepath.lrc" ]]; then
    if [[ "$(jq -r ".statusCode" <<< "$json")" = 404 ]]; then
      echo "${red}$filename - Could not fetch lyrics"
    elif [[ "$(jq -r ".syncedLyrics" <<< "$json")" = "null" ]]; then
      echo "${red}$filename - Synced Lyrics unavailable"
    else
      jq -r ".syncedLyrics" <<< "$json" > "$filepath.lrc" &&
      echo "${green}$fileglob.lrc - downloaded"
    fi
  else
    echo "${gray}$fileglob.lrc - already exists"
  fi
}
export -f get_lrc


parse_dir() {
  p=$(nproc --all 2>/dev/null || echo 1)
  find "$1" -type f -iregex '.*\.\(flac\|opus\|mp3\)$' -print0 |
    xargs -r0P "$p" -I{} bash -c 'get_lrc "{}"'
}


{ [ -d "$1" ] && parse_dir "$1"; }  || 
{ [ -f "$1" ] && get_lrc "$1"; }    ||
{ usage; exit 1; }

