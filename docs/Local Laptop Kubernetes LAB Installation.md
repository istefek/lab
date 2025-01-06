
[[Kubernetes]] (also known as K8s), is an open source system for automating deployment, scaling, and management of containerised applications.

The following are steps to install Rancher Desktop and a Kubernetes cluster on a Ubuntu Linux destribution.

For Windows WSL installations, refer to the [online WSL installation guide](https://docs.rancherdesktop.io/1.7/ui/preferences/wsl/).
## Install Rancher Desktop

Install Rancher Desktop from: <https://rancherdesktop.io/>

### 1. Ensuring You Have Access to `/dev/kvm`

On some distributions (Ubuntu 18.04 for example) the user has insufficient privileges to use `/dev/kvm`, which is required for Rancher Desktop. To check whether you have the required privileges, do:

```shell
[ -r /dev/kvm ] && [ -w /dev/kvm ] || echo 'insufficient privileges'
```

If it outputs `insufficient privileges`, you need to add your user to the `kvm` group. You can do this with:

```
sudo usermod -a -G kvm "$USER"
```

Then reboot in order to make these changes take effect.

### 2. Installation via .deb Package

Add the Rancher Desktop repository and install Rancher Desktop with:

```shell
curl -s https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/Release.key | gpg --dearmor | sudo dd status=none of=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/isv-rancher-stable-archive-keyring.gpg] https://download.opensuse.org/repositories/isv:/Rancher:/stable/deb/ ./' | sudo dd status=none of=/etc/apt/sources.list.d/isv-rancher-stable.list
sudo apt update
sudo apt install rancher-desktop
```

## 3. Install Brew for Linux

Source: <https://docs.brew.sh/Homebrew-on-Linux>

Homebrew is the package manager for Mac OS, but I like to also use it on Linux as well - so that the package lists are the same. if I use the same package on both systems, then I know they are from the same source. Also, packages that are not available via `apt`, then, this gives me another source to try.

Run the following to install Brew for Linux.

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Homebrew to your `PATH` and to your _.rcfile_, either `~/.bashrc` for `bash` or `~/.zshrc` for `zsh`. The following is for bash:

```shell
test -d ~/.linuxbrew && eval "$(~/.linuxbrew/bin/brew shellenv)"
test -d /home/linuxbrew/.linuxbrew && eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
echo "eval \"\$($(brew --prefix)/bin/brew shellenv)\"" >> ~/.bashrc
source .
```

## 4. Install Packages

Update Ubuntu package db then Install the following packages:

```shell
sudo apt update
sudo apt install tmux vim
brew install kubectl k9s
```


## 5. Rancher Configuration

1. Start Rancher Desktop application.
2. When the _Welcome to Rancher Desktop by SUSE_ window appears, Accept the defaults and **Click** _OK_.

	![[Rancher-Desktop-accept-default-options.png]]

3. The main window should appear. Rancher starts downloading/configuring vm's in the background which is normal. Wait for this to finish (see progress bar - bottom right of window).

	![[Main-Rancher-desltop.png]]


## 6. Confirm Your Kubernetes Cluster Is Running

Are there any pods in my cluster:

```shell
kubectl get pods
```

Try with `-A`
```shell
kubectl get pods -A
```

You should get a list of pods running in your cluster.

For example:

- local-path-provisioner
- coredns
- helm-install
- svclb-trafik
- helm-install-traefik
- metrics-server

If you take a look at your home directory, you should the addition of a  _.kube_ folder containing a `config` file created by Rancher Desktop app configuration wizard.

The _config_ file contains all of the details for your cluster.

Now try to run `k9s` in your terminal:

```shell
k9s
```

You should see the _k9s_ text console.

Press `0` **to view running pods**.

To exit the _k9s_ console, use the **VIM exit Keybinding**: `:q` [Enter]

You should now be at your prompt again.

## 7. Configure kubectl Bash Completions

1. Open your _.bashrc_ file for editiing:

```shell
vim ~/.bashrc
```

2. Add the following:

```text
# Kubectl
alias k='kubectl'
source /etc/bash_completion
source <(kubectl completion bash) # loads kubectl into the current environment.
complete -o default -F __start_kubectl k # Makes sure kubectl completion also works when I use the 'k' alias.
alias kgp='kubectl get pods'
alias kc='kubectx'
alias kn='kubens'
alias kcs='kubectl config use-context admin@homelab-staging'
alias kcp='kubectl config use-context admin@homelab-production'
```

## 8. VIM Configuration

Set up vim for editing _YAML_ files.

1. Create a _.vimrc_ file in your folder (only if it doesn't exist):

```shell
vim ~/.vimrc
```

2. Add the following lines:

```shell
" Ensure VIM uses filetype plugins
filetype plugin on

" Enable indentation
filetype indent on

" Set the default installation to 2 spaces for all files
set tabstop=2
set softtabstop=2
set shiftwidth=2
set expandtab

" Highlight trailing whitespace in all files
autocmd BufRead,BufNewFile * match Error /\s\+$/

" Enable auto-indentation
set autoindent

" Turn on syntax highlighting
syntax on

" Set baclspace so it acts more intuitively.
set backspace=indent,eol,start
```
 
## Conclusion

That's how easy it is to get a Kubernetes cluster up and running on your Linux laptop for testing and development.

