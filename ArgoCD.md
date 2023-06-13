Absolutely, let's update the tutorial with the Kubeconfig usage:

## Step 1: Install the ArgoCD CLI

Install the ArgoCD CLI on your MacBook using Homebrew. In your terminal, input:

```bash

brew tap argoproj/tap

brew install argoproj/tap/argocd

or follow the new method here: https://argo-cd.readthedocs.io/en/stable/cli_installation/

```

To verify your installation and check the version of ArgoCD, run:

```bash

argocd version

```

## Step 2: Set KUBECONFIG

To use a Kubeconfig file for authentication with `kubectl`, set the `KUBECONFIG` environment variable to point to your Kubeconfig file, which is usually located in `$HOME/.kube/config`:

```bash

export KUBECONFIG=${HOME}/.kube/config

```

To make this setting persistent across terminal sessions, add it to your shell's profile script:

For bash:

```bash

echo 'export KUBECONFIG=${HOME}/.kube/config' >> ${HOME}/.bashrc

source ${HOME}/.bashrc

```

For zsh:

```bash

echo 'export KUBECONFIG=${HOME}/.kube/config' >> ${HOME}/.zshrc

source ${HOME}/.zshrc

```

## Step 3: Authenticate with the ArgoCD API Using Kubeconfig

Now, you can log in to ArgoCD using your Kubeconfig file. Replace `<argo_server>` with your actual ArgoCD server address:

```bash

argocd login <argo_server>

```

## Step 4: Run the ArgoCD Admin Export Command

Once authenticated, you can run ArgoCD admin commands. To export all ArgoCD data, use:

```bash
# https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_admin/

argocd admin export

```

This will export all Argo CD related resources such as applications, projects, repositories, clusters, and settings.

## Step 5: Redirect Output to a File

To save the output of the `argocd admin export` command to a file, use the `>` operator:

```bash

argocd admin export > argocd_export.yaml

```

This will export all Argo CD data and save it to a file named `argocd_export.yaml` in your current directory.

Please note that the `argocd admin` commands, including `argocd admin export`, are available from Argo CD v2.1 onwards, so ensure you have this version or a later one installed. Additionally, you need admin privileges to run these commands. Be cautious with the exported data, as it might contain base64-encoded secrets.
