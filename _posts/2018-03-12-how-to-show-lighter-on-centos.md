---
layout: post
title:  "centos ssh终端下高亮显示git分支名"
categories: Linux
tags: Linux Ssh Git
---

* content
{:toc}

最近在整centos系统，希望能回到以前的那种工作环境，毕竟习惯了吧，也好用。

这个也是在网上找的，ssh终端高亮的方法，make一下。




#### 代码

将下面这些代码，追加 `/etc/profile` 或 `~/.bash_profie` 或 `~/.bashrc` 任何一个的最后，然后`source /etc/profile` (或`source ~/.bash_profile` 或 `source ~/.bashrc`)，让其生效即可。

```
#set git branch
green=$'\e[1;32m'
magenta=$'\e[1;35m'
normal_colours=$'\e[m'

function find_git_branch {
	 local dir=. head
	 until [ "$dir" -ef / ]; do
		if [ -f "$dir/.git/HEAD" ]; then
			head=$(< "$dir/.git/HEAD")
			if [[ $head == ref:\ refs/heads/* ]]; then
				git_branch=" ${head#*/*/}"
			elif [[ $head != '' ]]; then
				git_branch=' (detached)'
			else
				git_branch=' (unknown)'
			fi
				return
		fi
		dir="../$dir"
	done
	git_branch=''
}

PROMPT_COMMAND="find_git_branch; $PROMPT_COMMAND"
PS1="\[$green\]\u@\h:\w\[$magenta\]\$git_branch\[$green\]\\[$normal_colours\] "
```


最终效果：

![最终效果](https://upload.wikimedia.org/wikipedia/commons/7/75/Centos-lighter-git.png)