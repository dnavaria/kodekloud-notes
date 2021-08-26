# pushd & popd
- push directory to directory stack => `pushd <directory>`
- pops last pushed directory and change directory => `popd`
# whatis, man & appropos
- Print one line description of what a command does => `whatis <command>`
- Print detailed description of what a command does => `man <command>`
- search the manual page names and descriptions => `appropos <command>`
# Environment Variables
- To make env var in linux persistent over subsequent logins and reboot add them to `~/.profile or ~/.pam_environment`
- Adding path to path variable => `export PATH=$PATH:<new path>`
- Customizing PS1 variable => `PS1="[\d \t \u@\h:\w ] $"`
- `echo 'PS1="[\d]\u@\h:\w$"' >> ~/.profile`
![[Pasted image 20210826171130.png]]

- Change Shell => `sudo chsh -s /bin/<shell type> <username>`