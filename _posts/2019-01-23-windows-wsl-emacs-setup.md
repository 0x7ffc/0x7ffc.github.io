---
layout: post
title: "Windows 10 WSL Emacs Setup"
description: "A guide to setup WSL2, Emacs, X410 on Windows."
---

***You may want to scroll down and see all the updates***

I’ve always been using Mac for programming since my first job. I don’t actually own one myself because the company always give me one. Since I bought a thinkpad, I started to setup emacs on windows WSL:

### WSL & Hyper (a fancy terminal)

* Install WSL, You can [watch this youtube video](https://www.youtube.com/watch?v=Cvrqmq9A3tA).
* Install [choco](https://chocolatey.org/install#install-with-cmdexe).
* Install [hyper](https://github.com/zeit/hyper).
  - choco install hyper
    
After this, you may want to use zsh and switch hyper’s default shell to zsh.

### Zsh

* Install zsh & oh-my-zsh
* Set zsh as default shell [in WSL](https://medium.com/@vinhpz/set-and-use-zsh-as-default-shell-in-wsl-on-windows-10-the-right-way-4f30ed9592dc)

You may want to know where the hyper configuration file is: `C:\\Users\\${USER}\\.hyper.js`

### Shadowsocks & Proxychains.

For people like me in China, we all need this one more step…

* Get yourself a shadowsocks server.
* Install shadowsocks client.
* Install and use [proxychains](https://github.com/shadowsocks/shadowsocks/wiki/Using-Shadowsocks-with-Command-Line-Tools)

### Emacs & VcXsrv

* Install emacs on ubuntu.
* Install spacemacs or any configuration you like.
* If you use magit and on ubuntu 14.04, you need to [upgrade git](https://askubuntu.com/questions/579589/upgrade-git-version-on-ubuntu-14-04).
* Install [VcXsrv](https://sourceforge.net/projects/vcxsrv) unless you just use emacs inside the terminal. As for me, I need GUI for Nyan cat.
* I like [Firacode](https://github.com/tonsky/FiraCode/wiki/Linux-instructions) font, so I also installed it.
* Open XLauncher on windows with display number being 0.
* Set this inside WSL: `export DISPLAY=localhost:0.0`.
* And launch emacs.

It’s not perfect. It’s slow af. The text is blurry and Chinese characters are a mess. But I think I can take it after all this trouble.

---

#### Update 1:

I can’t visit my local hexo server through `127.0.0.1:4000`. After searching for a while, It turns out that it’s wegame’s fault ([link](https://github.com/Microsoft/WSL/issues/1554)). To solve this, you need to delete `%systemroot%\system32\drivers\QMTgpNetflow764.sys` and restart your computer. Or just uninstall wegame.

#### Update 2:

Ubuntu 18.04 is out: [link](https://www.microsoft.com/en-us/store/p/ubuntu-1804/9n9tngvndl3q).

#### Update 3:

I got rid of hyper and use [this](https://github.com/goreliu/wsl-terminal) terminal now. Since it supports 24bits color, I only use emacs in terminal mode now. NO MORE X11 SERVER AND SHIT.

#### Update 4:

I just found an X-server on windows 10 that just works out of box: [x410](https://www.microsoft.com/en-us/p/x410/9nlp712zmn9q). It works on my 2k screen (export GDK\_SCALE=2); Copy and paste works; It’s the only X-server that truly supports `fullscreen-frame` with my spacemacs; I can pin linux programes to windows taskbar, though it’s kinda time-consuming. Here are all the files: [gist](https://gist.github.com/ACEMerlin/0db059e6e055e3ef56e8ffde26b01f88), it contains:

* `bat-launcher.vbs`: execute a batch file without any flashing Console window
* `emacs.bat` and a Emacs shortcut
* `open-linux-shell-here.bat` and a terminal shortcut named Tilix
* `add-open-shell-here-menu.reg` and `remove-open-shell-here-menu.reg`: add or remove a `open linux shell here` option to right-click popup menu in windows file explorer
* `start-ubuntu-xfce-desktop.bat`: running xfce desktop

Keep in mind that you must change the scripts according to where you put the scripts. I have them in `E:\wsl`. Also if you don’t have x410, just use VcXsrv or any other X-server instead. All you need to do is comment out the lines with x410 and change it. For example, to use `emacs.bat` with VcXsrv.

```bat
REM here I use another keyboard layout, just remove it if you don't need it.

"C:\\Program Files\\VcXsrv\\vcxsrv.exe" -clipboard -wgl -nohostintitle -silent-dup-error -multiwindow -xkblayout us -xkbvariant dvorak

REM here I use my custom script to start emacs (ec), you can just uses emacs instead.

ubuntu1804.exe run "cd ~; export DISPLAY=127.0.0.1:0.0;  /home/merlin/.spacemacs.d/script/ec"
```

It it now a much more acceptable development workflow on windows.

![WSL emacs workflow screenshot](/assets/images/emacs-workflow.gif)

#### Update 5:

I couldn’t get native Linux docker to work, a work around is to install Windows version of docker and just use `docker.exe` since WSL and Linux share PATHs its more convenient than it sounds. But then I found this [issue](https://github.com/Microsoft/WSL/issues/1854), to fix this:

```
sudo mkdir /e
sudo mount --bind /mnt/e /e
```
From now on just use `/e` instead.

#### Update 6:

Ubuntu 20.04 is out: [link](https://www.microsoft.com/en-us/p/ubuntu-2004-lts/9n6svws3rx71).

#### Update 7:

WSL 2.0 is out: [link](https://docs.microsoft.com/en-us/windows/wsl/install-win10). In order to use it with X410, add this line to your startup script:

```
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2; exit;}'):0.0
```

As you can see, WSL's ip address is no longer `127.0.0.1`. You need to change all scripts with DISPLAY viriable in it.

#### Update 8:

I wrote my own emacs configuration now, I'll talk about it more in later article.