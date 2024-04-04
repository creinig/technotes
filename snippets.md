## ffmpeg

Only copy first 20 minutes of file: `ffmpeg -t 00:20:00 -i input.opus output.opus`

## git

* Find commits across all branches containing a certain file: `git log --all -- '**/FileName.ext'`
* List branches containing a certain commit: `git branch --containing COMMITHASH`

## X11

* Use the display of the first logged in user (e.g. when logged in via ssh): `export DISPLAY=$(w -hs | awk '{print $3}' | grep -Eo '^:[0-9]+' | head -n 1)`

## ugrep

* Find string recursively in all files, including archives and binary files (e.g. .jar and .class): `ug --decompress --recursive --with-hex "some_string_or_pattern"`

## General / misc

* Get my current public IP address: `curl -4 ifconfig.co` / `curl -6 ifconfig.co` 
* Locally override the login shell of a LDAP user: 'sudo sss_override user-add myuser -s /bin/zsh && sudo systemctl restart sssd` (credits: [Chris Robison](https://serverfault.com/a/1143802/168093))
