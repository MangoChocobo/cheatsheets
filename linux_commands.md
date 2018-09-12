**Easy SSH login**
Generate RSA keys on client and copy to server for passwordless login:
```
ssh-keygen -t rsa
ssh-copy-id user@server
```
Edit SSH config to alias hostname, user, and other settings:
```
# contents of $HOME/.ssh/config
Host <alias>
	HostName <hostname>
	User <user>
```

**Git**
Set username/email:
```
git config --global user.name <username>
git config --global user.email <email>
```
Store credentials:
```
git config credential.helper store	<-- per repo
```
