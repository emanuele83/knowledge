# Git Aliases

## List of git aliases
```
[alias]
    co = checkout
    br = branch
    brv = branch -vvv
    ci = commit
    st = status
    unstage = reset HEAD --
    last = log -1 HEAD
    ls = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate
    ll = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --numstat
    aa = add -A .
    cp = cherry-pick
    rema = remote -add
    remv = remote -v
    fetchall = fetch --all
    fetchallp = fetch --all --prune
    lll = log --graph --topo-order --date=iso8601-strict --no-abbrev-commit --abbrev=40 --decorate --all --boundary --pretty=format:'%Cgreen%ad %Cred%h%Creset -%C(yellow)%d%Creset %s %Cblue[%cn <%ce>]%Creset %Cblue%G?%Creset'
```

## List of Linuc-git aliases

```
alias gst='git status'
alias ga='git add'
alias gaa='git add -A .'
alias gd='git diff'
alias gco='git checkout'
alias gcom='git checkout master'
alias gremv='git remote -v'
alias gbrv='git branch -vvv'
alias gfall='git fetch --all'
alias glog='git ls'
alias glogf='git lll'
alias gk='gitk --all&'
```

### How to create permanent aliases in linux:

- Open the Terminal app

- Edit ~/.bash_aliases or ~/.bashrc file using: vi ~/.bash_aliases
- Append your bash alias. For example append: ```alias update='sudo yum update'```
- Save and close the file.
- Activate alias by typing: source ~/.bash_aliases

Please note that ~/.bash_aliases file only works if the following line presents in the ~/.bashrc file:

```
if [ -f ~/.bash_aliases ]; then
. ~/.bash_aliases
fi
```

[Source] (https://www.cyberciti.biz/faq/create-permanent-bash-alias-linux-unix/)