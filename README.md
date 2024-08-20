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
