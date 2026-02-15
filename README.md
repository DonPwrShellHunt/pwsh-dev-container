# pwsh-dev-container

Experimenting with vscode devcontainer and pwsh

## Dev Containers

### Updated devcontainer to dotnet:9.0-noble

```zsh
W: GPG error: https://dl.yarnpkg.com/debian stable InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 62D54FD4003F6525

E: The repository 'https://dl.yarnpkg.com/debian stable InRelease' is not signed.

ERROR: Feature "Common Utilities" (ghcr.io/devcontainers/features/common-utils) failed to install! Look at the documentation at https://github.com/devcontainers/features/tree/main/src/common-utils for help troubleshooting this error.
```

```zsh
To fix the yarnpkg error above, I had to find a more recently created image, which ended up
being dotnet:9.0-noble

There are an entire set of images with the following sha256.
sha256:faabbd9a48dae5b9d2b907ca251391c4810272ffcb128a1153a3a2849df3bac7
```

### Explore devcontainer dotnet:1-9.0-noble

Layer 18 is install of powershell (not using devcontainer feature)

```zsh
RUN /bin/sh -c powershell_version=7.5.2 
 && curl --fail --show-error --location --output PowerShell.Linux.arm64.$powershell_version.nupkg https://powershellinfraartifacts-gkhedzdeaghdezhr.z01.azurefd.net/tool/$powershell_version/PowerShell.Linux.arm64.$powershell_version.nupkg 
 && powershell_sha512='93cd89c9a8cf5705fed968453815a76a28c54a8dbf363fbee1d4fc131125b68b2e1c1424c9cc66729503f2caa4cc2934be47dd775970bdb74c5a3d26ee88363c' 
 && echo "$powershell_sha512  PowerShell.Linux.arm64.$powershell_version.nupkg" | sha512sum -c - 
 && mkdir --parents /usr/share/powershell 
 && dotnet tool install --add-source / --tool-path /usr/share/powershell --version $powershell_version PowerShell.Linux.arm64 
 && dotnet nuget locals all --clear 
 && rm PowerShell.Linux.arm64.$powershell_version.nupkg 
 && ln -s /usr/share/powershell/pwsh /usr/bin/pwsh 
 && chmod 755 /usr/share/powershell/pwsh 
 && find /usr/share/powershell -print | grep -i '.*[.]nupkg$' | xargs rm # buildkit
```

Notice the output within the container shows

* $PSHOME/pwsh is not executable within container
* Given execute permission, $PSHOME/pwsh still fails in what appears to be non-arm64 references
* $PSHOME/Modules is where the 7.5.2 powershell modules are located
* Actual pwsh executable is /usr/share/powershell/pwsh (consistent with --tool-path of dotnet tool install)
* Links point to this executable from /bin/pwsh and /usr/bin/pwsh
* Without "terminal.integrated.shellIntegration.enabled": false, shell integration fails causing terminal launch issues

```zsh
> find /usr/share/powershell/ -name "pwsh" -ls
  2516434  76 -rwxr--r--   1 root  root  75208 Jun 18 21:54 /usr/share/powershell/.store/powershell.linux.arm64/7.5.2/powershell.linux.arm64/7.5.2/tools/net9.0/any/pwsh
  2516622  76 -rwxr-xr-x   1 root  root  74808 Jul  8 18:21 /usr/share/powershell/pwsh

> $PSHOME
/usr/share/powershell/.store/powershell.linux.arm64/7.5.2/powershell.linux.arm64/7.5.2/tools/net9.0/any

PS /workspaces/pwsh-dev-container> (gps -Id $pid).Path
/usr/share/powershell/pwsh

PS /workspaces/pwsh-dev-container> which -a pwsh
/usr/bin/pwsh
/bin/pwsh

PS /workspaces/pwsh-dev-container> ls -l /bin/pwsh /usr/bin/pwsh /usr/share/powershell/pwsh
lrwxrwxrwx 1 root root    26 Jul  8 18:21 /bin/pwsh -> /usr/share/powershell/pwsh
lrwxrwxrwx 1 root root    26 Jul  8 18:21 /usr/bin/pwsh -> /usr/share/powershell/pwsh
-rwxr-xr-x 1 root root 74808 Jul  8 18:21 /usr/share/powershell/pwsh

> sudo /usr/share/powershell/.store/powershell.linux.arm64/7.5.2/powershell.linux.arm64/7.5.2/tools/net9.0/any/pwsh --version
rosetta error: failed to open elf at /lib64/ld-linux-x86-64.so.2
```

## VSCode create dev container in volume for PowerShell

Selected x86_64 container type despite running on M1 Mac.

Image was `mcr.microsoft.com/powershell:lts-debian-11`. From my previous research, I do not believe any of the powershell containers are multi-architure aware, so the first changes I make will be to choose an images which will run on the ARM architecture.

## Modifications after initial settings

Use a devcontainer/base:dev-noble image and tweak it to run as the vscode remoteuser.

The base:dev-noble images did change the architecture to ARM (expected), but it also installed  7.5.0-preview.3 (unexpected). The vscode user was also used as the remoteuser and it had a UID/GID of 1001 (also expected from looking at history in devcontainer repo).

Next I'll change back to just noble rather than dev-noble and see what happens.

The unexpected preview version of PowerShell came from devcontainers-contrib. Once I changed it the standard version the stable version of pwsh 7.4.4 was installed.

## Permission Denied with pwsh terminal type

If I create a new zsh terminal, and then type pwsh, I get into pwsh ok.

## examine base:noble image

How is dotnet tool pwsh installed? Appears to be 'local', which may mean it is installed independent of a particular user (vscode). When I installed powershell as vscode user and --global flag, the software was put under /home/vscode/.dotnet/tools if I remember correctly.

Ok - dotnet installs are a little bizarre. The following info was found on [learn site](https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools)

The --global flag causes a tool to default install path to $HOME/.dotnet/tools and tool access is user-specific, not machine global. WTF! Really misleading terminology!

The --tool-path PATH option of dotnet tool install will place the tool in that specified directory, but it will be subject to PATH contents to locate executable.

The --local flag contrains access to a subtree of directories and requires a tool manifest file, typically dotnet-tools.json

## UserUID Choice and pwsh as vscode login shell

Default vscode userUID is 1000 from mcr.microsoft.com/devcontainers/dotnet:9.0-noble, but it does have an app user defined as 1654. The description of this app user would seem to indicate that 1655 would be a better userUID for vscode, so that is what I decided to use for now (July 20,2025). I have already encountered losing access to a volume that was created with 1655, but subsequently was run with vscode set to 1000. Messy scenario that I do not totally understand - still!

For now, I will try to maintain consistant use of 1655 and see what happens.

```zsh
$(which pwsh) | sudo tee -a /etc/shells
```

would add following string at end of this file
`/usr/share/powershell/pwsh`

Need to figure out how to add pwsh to shells AND set the vscode user shell to pwsh. Syntax is tricky for multiple commands in string format.

"postCreateCommand": "command -v pwsh | sudo tee -a /etc/shells && sudo chsh vscode -s \"$(command -v pwsh)\""

## execvp permission denied Error

Notice below that the dotnet instance of pwsh is not executable by everyone. This instance is being used to inject the shell integration into the terminal, but it fails with above error. To avoid this failure I disabled the shell integration in the settings.jason.

```zsh
$ find /usr/share/powershell -name "pwsh" -ls
  2516434     76 -rwxr--r--   1 root     root        75208 Jun 18 21:54 /usr/share/powershell/.store/powershell.linux.arm64/7.5.2/powershell.linux.arm64/7.5.2/tools/net9.0/any/pwsh
  2516622     76 -rwxr-xr-x   1 root     root        74808 Jul  8 18:21 /usr/share/powershell/pwsh
```
