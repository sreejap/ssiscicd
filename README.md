# ssiscicd
Demo code for SQL Server Integration Services Continuous Integration Continuous Delivery on Microsoft Azure Cloud
Presentation recording is available at https://youtu.be/nuGf6f1Bhj0

## Setup Agent for DevOps Pipeline
1. Create a Dev Test Lab environment and new Dev Test Lab VM "Visual Studio 2019 Community (latest release) on Windows 10 Enterprise N (x64)"
1. Sign in
1. Update to latest version of Visual Studio 2019 (Why isn't this the latest version already, Microsoft?)
1. Open up Visual Studio 2019 Community Edition "Extensions" Menu > Manage Extensions
1. Search for and start install of SQL Server Integration Services Projects. It will require you to close VS2019. Wait like 10 minutes.
1. Follow these instructions to setup the agent on the VM (https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops)
1. Find devenv.exe. On my build box, it is here: "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\Common7\IDE"devenv.exe
1. Install kubectl by following these instructions https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows
1. Install OpenJDK because Java is required by Maven (https://jdk.java.net/13/)
1. Install maven by downloading and following these instructions
   1. https://maven.apache.org/download.cgi
   1. https://maven.apache.org/install.html
1. Install the Azure CLI https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest
1. Install sqlpackage.exe to deploy dacpacas https://docs.microsoft.com/en-us/sql/tools/sqlpackage-download?view=sql-server-ver15
1. Install sqlcmd.exe to run unit tests from self-hosted agent https://docs.microsoft.com/en-us/sql/tools/sqlcmd-utility?view=sql-server-ver15



## Setup VM for compiling container image
- Make sure to clone the master.dacpac and msdb.dacpac to the server, otherwise tsqlt will not deploy.

## Setup Developer Database/AKS
- https://docs.microsoft.com/en-us/azure/aks/windows-container-cli
- https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-2017

### Creates AKS -- note that we removed monitoring
```PASSWORD_WIN="<Y0u4Passwo3dGo3sH!r!>"

az aks create \ 
    -g <yourResourceGroup> \ 
    --name ssiscicdAKS \
    --node-count 1 \
    --kubernetes-version 1.14.6 \
    --generate-ssh-keys \
    --windows-admin-password $PASSWORD_WIN \
    --windows-admin-username azureuser \
    --vm-set-type VirtualMachineScaleSets \
    --network-plugin azure
```

### Add a windows node pool
```az aks nodepool add \
    --resource-group <yourResourceGroup> \
    --cluster-name ssiscicdAKS \
    --os-type Windows \
    --name npwin \
    --node-count 1 \
    --kubernetes-version 1.14.6
```
    
### Configure kubectl to hit our AKS
`az aks get-credentials --resource-group <yourResourceGroup> --name ssiscicdAKS`	
