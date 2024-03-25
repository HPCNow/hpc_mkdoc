ACC - Advanced Computing Center :VS Code
==================================================

[Microsoft Visual Studio Code](https://code.visualstudio.com/docs) (VS Code) is an extensible text editor which is widely used for editing software source code. It is possible to run VS Code in the Exacloud environment by starting a server component as a job and connecting to it with a web browser. Due to its high resource usage, the use of VS Code on cluster head nodes is discouraged.

### Custom VS Code Web Service


Microsoft provides a Linux command-line interface version of VS Code. Through this CLI version we can use the "serve-web" feature to create a web-based VS Code server.

The steps to getting VS Code running in Exacloud are documented below.

#### Environment Set Up

First choose a filesystem path to act as a home for your installation of VS Code. ACC recommends that you choose gscratch as the location for VS Code as it will have the best performance in the cluster environment. In the examples below we'll set the variable **VSCODE_PATH**. You'll need to customize the location you've chosen.

To create the directories and download VS Code, do the following:


```  sh title="Set-up VS Code"
VSCODE_PATH=/home/exacloud/gscratch/Foolab/johndoe/vscode # Customize this path
mkdir -p $VSCODE_PATH
pushd $VSCODE_PATH
curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz
tar xvzf vscode_cli.tar.gz
rm vscode_cli.tar.gz
```

Now you can proceed to setting up a submission script.

#### Submission Script


In the same directory, create a file `run-vscode.sh`, and copy the contents of the template below into it. You'll then want to customize the template with the appropriate **VSCODE_PATH** variable, along with any job scheduler parameters you want to change.

``` sh title="run-vscode.sh"
#!/bin/bash

#SBATCH --time=08:00:00
#SBATCH --signal=USR2
#SBATCH --partition=interactive
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=8192
#SBATCH --job-name=vscode

# Path to your vscode location
VSCODE_PATH="/home/exacloud/gscratch/Foolab/johndoe/vscode"

readonly PORT=$(/usr/local/bin/get-open-port.py)
readonly NODE=$(/usr/bin/hostname -s)
readonly IP=$(/usr/bin/dig +short "${NODE}.local")
readonly TOKEN=$(/usr/bin/openssl rand -hex 15)

echo "SSH connection string:"
echo "  $ ssh ${USER}@exahead1.ohsu.edu -L ${PORT}:${NODE}:${PORT}"
echo
echo "Once the connection is established, copy the following link"
echo "into your browser:"
echo "  http://127.0.0.1:${PORT}?tkn=${TOKEN}"
echo
echo "Ignore the link which starts with http://${IP} below."

srun $VSCODE_PATH/code serve-web\
	--accept-server-license-terms\
	--host $IP\
	--port $PORT\
	--connection-token $TOKEN\
	--server-data-dir $VSCODE_PATH
```

#### Submitting and Connecting


With the submission script customized, you can submit it by running

``` sh title="Job submission"
sbatch run-vscode.sh
```

Make note of the job ID number. Check the contents of the log file, which will be named like `slurm-<jobID>.out`. To connect to your VS Code job, you'll need to take two actions:

1.  Start a new SSH connection from your workstation with the options given in the log file (under "SSH Connection String").
2.  Navigate to the link starting with http://127.0.0.1 in the log file. VS Code will also output a link later in the log file, but that should be ignored.

VS Code in the browser is functionally identical to the native VS Code application. Keep in mind that any customization you perform will be particular to this location.

### Clean-Up


When you are done with a VS Code session:

1.  Close the browser tab
2.  Close the extra SSH connection
3.  Use `scancel` to cancel the running job

### Known Issues and Troubleshooting


The above guide has the following known issues:

-   User settings (including Theme) and other customizations are not saved between sessions.

If there are any issues getting the job started, please see the [Exacloud Troubleshooting Guide](17-Troubleshooting.md).