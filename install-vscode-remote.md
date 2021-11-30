https://medium.com/@debugger24/installing-vscode-server-on-remote-machine-in-private-network-offline-installation-16e51847e275


OR
Connect to the remote machine and list the directories in the following directory to get the commit ID
$ ls ~/.vscode-server/bin
c9a2f78283b6e5ef708fb8869e2a5adaa476e42f
Step 5: Download the offline installation version of vscode server externally using the following link
For Stable Version
https://update.code.visualstudio.com/commit:{COMMIT_ID}/server-linux-x64/stable
For Insider Version
https://update.code.visualstudio.com/commit:{COMMIT_ID}/server-linux-x64/insider
For example,
https://update.code.visualstudio.com/commit:c9a2f78283b6e5ef708fb8869e2a5adaa476e42/server-linux-x64/insider
Step 6: Transfer this file to the remote machine.
Step 7: Extract the content of this file in the destFolder mentioned in the VSCode logs.
Usually, it is
~/.vscode-server/bin/{COMMIT_ID}
To extract run the following commands
$ cd ~/.vscode-server/bin/c9a2f78283b6e5ef708fb8869e2a5adaa476e42f
$ tar -xvzf ~/vscode-server-linux-x64.tar.gz  --strip-components 1 
vscode-server-linux-x64/LICENSE
vscode-server-linux-x64/node
vscode-server-linux-x64/bin/
vscode-server-linux-x64/bin/code-insiders
vscode-server-linux-x64/server.sh
vscode-server-linux-x64/package.json
vscode-server-linux-x64/out/
vscode-server-linux-x64/out/nls.metadata.json
...
...
...
Now we are done with VSCode installation on the local machine and VSCode Server on the remote machine. Go ahead and try to connect with remote machine as mentioned here https://code.visualstudio.com/docs/remote/ssh#_connect-to-a-remote-host
