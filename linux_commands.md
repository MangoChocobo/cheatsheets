**SSH**

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
Make SSH connections share TCP connections to same server (muliplexing):
```
# contents of $HOME/.ssh/config
Host *
   ControlMaster auto
   ControlPath ~/.ssh/master-%r@%h:%p
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
