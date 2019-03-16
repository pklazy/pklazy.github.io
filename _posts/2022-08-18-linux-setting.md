---
layout: post
title: 리눅스 설정 기록
category: 메모
tags: linux
date: 2022-08-18 22:30:00 +0900
---

### gnome

키보드 ctrl capslock 위치 변경

```console
$ gsettings set org.gnome.desktop.input-sources xkb-options "['ctrl:swapcaps']"
```

### i3

마우스 휠 반전
/etc/X11/xorg.conf.d/30-touchpad.conf 
```conf
Section "InputClass"
        Identifier "touchpad"
        MatchIsTouchpad "on"
        Option "NaturalScrolling" "true"
EndSection
```

키보드 ctrl capslock 위치 변경
cat /etc/X11/xorg.conf.d/00-keyboard.conf 
```conf
Section "InputClass"
        Identifier "system-keyboard"
        MatchIsKeyboard "on"
        Option "XkbLayout" "kr"
        Option "XkbOptions" "ctrl:swapcaps"
EndSection
```

### console

키보드 ctrl capslock 위치 변경

```console
$ dumpkeys > backup.kmap
$ showkey
$ dumpkeys | head -1 > ctrl-swapcaps.kmap
$ cat << EOF >> ctrl-swapcaps.kmp
keycode 29 = Caps_Lock
keycode 58 = Control
EOF
# loadkeys ctrl-swapcaps.kmp
```

참고:
- <https://superuser.com/questions/290115/how-to-change-console-keymap-in-linux>

