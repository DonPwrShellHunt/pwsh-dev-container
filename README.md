# pwsh-dev-container
Experimenting with vscode devcontainer and pwsh

## VSCode create dev container in volume for PowerShell

Selected x86_64 container type despite running on M1 Mac.

Image was `mcr.microsoft.com/powershell:lts-debian-11`. From my previous research, I do not believe any of the powershell containers are multi-architure aware, so the first changes I make will be to choose an images which will run on the ARM architecture.

## Modifications after initial settings

Use a devcontainer/base:dev-noble image and tweak it to run as the vscode remoteuser.
