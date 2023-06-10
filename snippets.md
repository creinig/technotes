## ffmpeg

Only copy first 20 minutes of file: `ffmpeg -t 00:20:00 -i input.opus output.opus`

## git

* Find commits across all branches containing a certain file: `git log --all -- '**/FileName.ext'`
* List branches containing a certain commit: `git branch --containing COMMITHASH`

## X11

* Use the display of the first logged in user (e.g. when logged in via ssh): `export DISPLAY=$(w -hs | awk '{print $3}' | grep -Eo '^:[0-9]+' | head -n 1)`
