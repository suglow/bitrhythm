+++
title = 'Autohot key修改键位记录'
date = 2023-11-06T06:25:52Z
draft = false
+++

### 键位修改描述
将caplock修改为ctrl，同时caplock的tap修改为esc
将space修改为modifier按键，空格+p为粘贴，空格+y为复制
```
#Requires AutoHotkey v2.0
; Remap Capslock to Ctrl:
Capslock::Ctrl

Capslock Up::{
    Send "{Ctrl Up}"
    If (A_PriorKey = "Capslock") ; if Capslock was pressed alone
        Send "{Esc}"
}


Space::Return
Space Up::
{
    global
    if (A_PriorKey = "Space")
    {
        Send("{Space}")
    }
    return
}
Space & j::Up
Space & h::Left
Space & k::Down
Space & l::Right

Space & p::{
    Send "{Blind}+{INS}"
}
Space & y::{
    Send "{Blind}^{INS}"
}
```