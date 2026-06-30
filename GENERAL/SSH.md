
# IMPLEMENTATIONS

OpenSSH

Paramiko			-	this seems to be a python implementation




# CLIENT

## Configuration files for client:
~/.ssh/config
/etc/ssh/ssh_config



## SSH Options on Client:


ControlPersist		-	allows SSH connections to persist so frequent commands run over SSH don't have to go through the initial handshake over and over again
						until ControlPersist timeout is reached


IdentitiesOnly

ForwardAgent (-A)	-	when set to yes will make the local ssh-agent be available via a Unix socket on the remote host (socket setup by SSHD)
						then ssh-add run on the remote host will be able to talk to ssh-agent on the local host

ProxyJump (-J)		-	the ability to use a bastion host to connect to a remote server
						the ssh client program never runs on the bastion host, 
						instead SSHD on the bastion host connects to the remote host and hands the connection back to the local ssh client,
						the local ssh client then performs a second handshake (so think symmetric key setup for the connection)

ProxyCommand		-	a command to run before making the connection (i think)
(-o ProxyCommand "<command>")



# SERVER


## SSH Options on Server (OpenSSH)

ForceCommand 			internal-sftp
PasswordAuthentication 	yes
ChrootDirectory 		/var/sftp
PermitTunnel 			no
AllowAgentForwarding 	no
AllowTcpForwarding 		no
X11Forwarding 			no



# SSH AGENT

## What it is
On the local host, it is a store of keys and certificates in memory along with passphrase for key


## Agent forwarding
When you connect to a remote host with agent forwarding enabled,
SSHD on the remote host, 
will create a Unix domain socket on the remote host,
linked to the agent forwarding channel of the SSH connection (remember that SSH can have multiple channels on a connection),
and export an environment variable called SSH_AUTH_SOCK pointing to it
Now ssh-add on the remote host can talk to the ssh-agent on the local host


NOTE: the socket that is in the command line of the ssh-agent process (local host) is NOT the same as the socket used by ssh-add on the local host

```shell
# check the agent is running on the local host
ps uxf | grep ssh-agent

# check the socket being used by the agent on the remote host
env | grep SSH_AUTH_SOCK

# add a key to the agent (local or remote)
ssh-add <key>

# list identities (keys and certificates) in the agent
ssh-add -l
```