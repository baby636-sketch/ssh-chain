DESCRIPTION

ssh-chain - ssh via a chain of intermediary hosts

NOTE

This functionality is built into OpenSSH via the -J option as of version 7.3

INSTALL

Copy the ssh-chain script to somewhere that's in your path. Append the
following to ~/.ssh/config or /etc/ssh/ssh_config:

# This should be the last entry
Host *^*
ProxyCommand ssh-chain %h %p

and you're done.

USAGE

ssh-chain can act as a wrapper to ssh in order to avoid filling your
known_hosts file with garbage - just run ssh-chain instead of ssh.

The simple use case is this:

ssh final.example^second.example^first.example

The connection is built right to left, so you'll end up with a set of
connections that looks like this:

you -> first.example -> second.example -> final.example 

This will also work with scp/sftp and hopefully any other tool that invokes
ssh as a backend (e.g. rsync, git, svn, etc.) and all the standard features
such as port forwarding should work.

ADVANCED USAGE

Sometimes you'll have need to specify a username or port for an
intermediary host.  Since ssh will normally consume these, different (and
sort of weird) syntax is used. Ports are specified by appending an underscore
(e.g. foo.example_2222) and usernames use a plus instead of an at symbol (e.g.
jdoe+foo.example). The far left host still needs to be specified using an
at symbol since this doesn't get fed to the ProxyCommand.  Example:

jdoe@final.example^johnd+second.example_2222^john+first.example_443

HOST-SPECIFIC OPTIONS

To make host-specific options for hosts other than the first one in the chain
work, you need to change lines like this

Host *.foo.example bar.example
User john
Port 2222

to

Host *.foo.example *.foo.example^* bar.example bar.example^*
User john
Port 2222

NOTES

It's preferable to use OpenSSH 5.4 or newer with ssh-chain. 'netcat mode' (-W)
was added then and this is faster then exec'ing netcat on the remote host.
ssh-chain auto-detects if -W is available and will remote exec netcat
otherwise.
