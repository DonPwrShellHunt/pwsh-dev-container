# pwsh-dev-container
Experimenting with vscode devcontainer and pwsh

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

Ok - dotnet installs are a little bizarre. The following info was found [here](https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools)

The --global flag causes a tool to default install path to $HOME/.dotnet/tools and tool access is user-specific, not machine global. WTF! Really misleading terminology!

The --tool-path <path> option of dotnet tool install will place the tool in that specified directory, but it will be subject to PATH contents to locate executable.

The --local flag contrains access to a subtree of directories and requires a tool manifest file, typically dotnet-tools.json
