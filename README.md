This script supports accessing the proton bridge running locally and sending ntfy notifications when a new email comes in.

Basic start from a debian virtual machine:

```bash
apt install ./proton-bridge.deb
apt install pass curl netcat-openbsd

gpg --full-generate-key # Any fake name,fake email, leave password blank
gpg --list-keys
pass init # <fake email address you created>

protonmail-bridge -cli
login # <complete login steps>
info # <copy down local imap password>
exit

nohup protonmail-bridge -n &
```

Now we can run the script below to poll for new imap idle messages. It is largely identical to [this script](https://github.com/johan-adriaans/shell-imap-notify/blob/master/imap-notify) however modified to use a non-ssl connection since we are connecting to localhost on the same machine. The bridge takes care of the encryption/decryption.

You **must** change the user, password and the ntfy.sh/<CHANGEME> to your topic. You could also use apprise here and send notifications wherever you like. ntfy can get away with just using curl however.

```bash
#!/bin/bash
user=bob@example.com
password=<password from bridge; NOT your normal login password>

start_idle() {
	echo ". login \"${user}\" \"${password}\""
	echo ". select inbox"
	echo ". idle"
	while true; do
		sleep 600
		echo "done"
		echo ". noop"
		echo ". idle"
	done
}

# Start non-ssl connection
while read -r line; do
	# Debug info, turn this off for silent operation
	echo "$line"
	if echo "${line}" | grep -Eq ". [1-9][0-9]? RECENT"; then
		echo "New mail received, sending notification"
		curl -H ta:email -H "Title: New Mail!" \
			-d "You have a new email message." ntfy.sh/<CHANGEME>
	fi
done < <(nc -C 127.0.0.1 1143 2>/dev/null < <(start_idle))
```
