#!/bin/sh
[ -f /etc/banner ] && cat /etc/banner
[ -e /tmp/.failsafe ] && cat /etc/banner.failsafe
fgrep -sq '/ overlay ro,' /proc/mounts && {
	echo 'Your JFFS2-partition seems full and overlayfs is mounted read-only.'
	echo 'Please try to remove files from /overlay/upper/... and reboot!'
}

export PATH="/usr/sbin:/usr/bin:/sbin:/bin"
export HOME=$(grep -e "^${USER:-root}:" /etc/passwd | cut -d ":" -f 6)
export HOME=${HOME:-/root}
export PS1='\u@\h:\w\$ '

[ "$TERM" = "xterm" ] && export PS1='\[\e]0;\u@\h: \w\a\]'$PS1

[ -x /bin/more ] || alias more=less
[ -x /usr/bin/vim ] && alias vi=vim || alias vim=vi

alias ll='ls -alF --color=auto'

[ -z "$KSH_VERSION" -o \! -s /etc/mkshrc ] || . /etc/mkshrc

[ -x /usr/bin/arp -o -x /sbin/arp ] || arp() { cat /proc/net/arp; }
[ -x /usr/bin/ldd ] || ldd() { LD_TRACE_LOADED_OBJECTS=1 $*; }

[ -n "$FAILSAFE" ] || {
	for FILE in /etc/profile.d/*.sh; do
		[ -e "$FILE" ] && . "$FILE"
	done
	unset FILE
}

if ( grep -qs '^root::' /etc/shadow && \
     [ -z "$FAILSAFE" ] )
then
cat << EOF
=== WARNING! =====================================
There is no root password defined on this device!
Use the "passwd" command to set up a new password
in order to prevent unauthorized SSH logins.
--------------------------------------------------
EOF
fi

service() {
	[ -f "/etc/init.d/$1" ] || {
		echo "service "'"'"$1"'"'" not found, the following services are available:"
		ls "/etc/init.d"
		return 1
	}
	/etc/init.d/$@
}


### Custom user commands ##########################################################

# Change bash prompt. See the article
export PS1='\[\e[0;37m\][\[\e[1;35m\]\@ \[\e[1;31m\]\u\[\e[0;37m\]@\[\e[1;36m\]\h:\[\e[1;33m\]\w\[\e[1;32m\]\[\e[0;37m\]\]\$ \[\e[0m\]'

# Clear the terminal
alias c='clear'

# RAM free in kb
alias f='free'

# Disk free in human terms
alias d='df -h'

# Monitor logs
alias log='logread -l40'

# Uptime system
alias t="uptime"

# Reload /etc/profile
alias p="source /etc/profile"

# Command in functions
up() {
	# Uptime
	let upSeconds="$(/usr/bin/cut -d. -f1 /proc/uptime)"
	let secs=$((${upSeconds}%60))
	let mins=$((${upSeconds}/60%60))
	let hours=$((${upSeconds}/3600%24))
	let days=$((${upSeconds}/86400))
	UPTIME=`printf "%d days, %02d:%02d:%02d" "$days" "$hours" "$mins" "$secs"`
	#UPTIME=`printf "%d days %02d:%02d" "$days" "$hours" "$mins"`
	printf "Uptime System: %1s\n" "$UPTIME"
}

u() { cd ".."; }
l() { ls -lahF; }
x() { echo "Bye..."; sleep 1; exit; }
re() { echo -ne "*** WARNING! The system will be rebooted...\n\n"; reboot now; sleep 3; exit; }

eb() { service named enable; }
db() { service named disable; }
ub() { service named start; }
rb() { service named reload; }
sb() { service named stop; }
bind() { cd /etc/bind; ls -la; }
master() { cd /etc/bind/master; ls -la; }
slave() { cd /etc/bind/slave; ls -la; }

szone() { 
	[ "$1" = "w" ] && nano /etc/bind/master/db.source.lab || cat /etc/bind/master/db.source.lab | less;
}

mzone() {
	[ "$1" = "w" ] && nano /etc/bind/master/db.makhonin.ru || cat /etc/bind/master/db.makhonin.ru | less;
}
azone() {
	[ "$1" = "w" ] && nano /etc/bind/master/4.168.192.in-addr.arpa || cat /etc/bind/master/4.168.192.in-addr.arpa | less;
}

root() { cd /; ls -la; }
conf() { cd /etc/config; ls -la; }

etc() { cd /etc; ls -la; }
init() { cd /etc/init.d; ls -la; }
usr() { cd /usr; ls -la; }
tmp() { cd /tmp; ls -la; }

# APT utilit for update and upgrade packages
apt() {
	ver="openwrt ver. 0.1b"
	if [ "$1" == install ]; then 
		echo -ne "Install pakage $2\n"
		opkg install
		echo -ne "\nOperation complete.\n"
		return 0
	elif [ "$1" == "-c" ]; then
		echo -ne "Configure package $2. Please wait...\n"
		opkg configure $2
		echo -ne "\n$LIST\n\nOperation complete.\n"
		return 0
	elif [ "$1" == "-l" ]; then
		echo -ne "List upgradable packages. Please wait...\r"
		LIST=`opkg list-upgradable | awk -F" - " 'BEGIN {printf "%22s\t %28s\t %28s\n\n", "PACKAGE NAME", "CURRENT VERSION", "NEW VERSION";} {printf "%22s\t %28s\t %28s\n", $1,$2,$3} END {printf "\n";}'`
		echo -ne "$LIST\n\n"
		return 0
	elif [[ "$1" == "-r" && -n "$2" ]]; then
		echo -ne "Remove pakage(s) $2\n"
		opkg remove $2
		echo -ne "\nOperation complete.\n"
		return 0
	elif [ "$1" == "-u" ]; then 
		echo -ne "Update list of available packages. Please wait...\n"
		opkg update
		echo -ne "\nOperation complete.\n"
		return 0
	elif [[ "$1" == "-g" && -n "$2" ]]; then
		echo -ne "Upgrade pakage(s) $2\n";
		opkg upgrade $2
		echo -ne "\nOperation complete.\n"
		return 0
	elif [[ "$1" == "-v" && -n "$2" ]]; then
		SCAN=`opkg list-installed | grep "$2"`
		VER=`opkg list-installed | grep "$2" | awk -F" - " 'BEGIN {printf "%22s\t %28s\n\n", "PACKAGE NAME", "CURRENT VERSION";} {printf "%22s\t %28s\n", $1,$2} END {printf "\n";}'`
		if [ -z "$SCAN" ]; then
			echo -ne "Package <$2> is not installed.\n"
			return 0
		else
			echo -ne "$VER\n\n"
			return 1
		fi
	else 
		printf "apt $ver\n"
		printf "Usage: apt command [options]\n"
		printf "\n"
		printf "Commands:\n"
		printf "  -l          - List packages based on package names\n"
		printf "  -u          - Update list of available packages\n"
		printf "  -i <pkgs>   - Install package(s)\n"
		printf "  -c <pkgs>   - Configure unpacked package(s)\n"
		printf "  -g <pkgs>   - Upgrade packages\n"
		printf "  -r <pkgs>   - Remove package(s)\n"
		printf "  -v <pkgs>   - Show the installed version of a package\n\n"
	fi
}

### ─ ┬ ├ ≥ ┤ ▒ ° ± ┘ ┐ ┌ ≤ │ ┴ ┼ └ ###

hint() {
[ "$1" == "w" ] && nano /etc/profile;
cat << EOF
┌──────────────────────────────────────────┐
│          QUICK USER COMMANDS             │
│        c - Clear the terminal            │
│        d - Disk free in human terms      │
│        f - RAM free in kb                │
│        l - List directory                │
│        p - Reload Profile file           │
│        u - To the directory above        │
│        x — Exit from the shell           │
│      log - Show logs                     │
├──────────────────────────────────────────┤
│             NAVIGATION                   │
│     root - Go to directory /             │
│     conf - Go to directory config        │
│      etc - Go to directory etc           │
│     init - Go to directory init.d        │
│      usr - Go to directory usr           │
│      tmp - Go to directory tmp           │
├──────────────────────────────────────────┤
│      apt - Short version opkg            │
├──────────────────────────────────────────┤
│            BIND COMMANDS                 │
│       eb - Service named enable          │
│       db - Service named disable         │
│       ub - Service named start           │
│       rb - Service named reload          │
│       sb - Service named stop            │
│     bind - Go to directory bind          │
│   master - Go to directory master        │
│    slave - Go to directory slave         │
│    szone - Read/Write zone source.lab    │
│    mzone - Read/Write zone makhonin.ru   │
│    szone - Read/Write zone 4.168.192     │
├──────────────────────────────────────────┤
│       re - Reboot System                 │
└──────────────────────────────────────────┘
EOF
}

### Custom user commands ##########################################################

