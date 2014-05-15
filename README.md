darrenxyli.github.io
====================

darrenxyli.github.io


# copy to ~/.bash_profile
#
# xTerm-256color
export LS_OPTIONS='--color=auto'
export CLICOLOR='Yes'
export LSCOLORS='Exfxcxdxbxegedabagacad'

# Modify Terminal Prompt and Color
case $(id -u) in
    0)
    STARTCOLOUR='\[\e[1;91m\]';
        ;;
    *)
    STARTCOLOUR='\[\e[1;93m\]';
    ;;
esac
ENDCOLOR="\[\e[0m\]"
UNDERLINEBLUE="\[\e[4;34m\]"
MYWORD="ϟ-VeniVidiVici@Darrenxyli:"
PS1="\n$STARTCOLOUR$MYWORD$ENDECOLOR $UNDERLINEBLUE\w$ENDCOLOR\n\$ ";


# set go path
export GOPATH="/Users/xli/Documents/playground/mygo"

# docker ip
export DOCKER_HOST=tcp://127.0.0.1:4243

# set gopath
export GOPATH="/usr/local/go"

# set alias
alias ll = 'ls -al'

# this is for .gitconfig
[user]
	name = darrenxyli-aa
	email = xli@appannie.com

[alias]
    st = status -s
    ci = commit
    l = log --oneline --decorate -12 --color
    ll = log --oneline --decorate --color
    lc = log --graph --color
    co = checkout
    br = branch
    rb = rebase
    dci = dcommit
    sbi = submodule init
    sbu = submodule update
    sbp = submodule foreach git pull
    sbc = submodule foreach git co master
