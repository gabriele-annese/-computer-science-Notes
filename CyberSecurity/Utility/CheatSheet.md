---
cssclasses:
  - page-grid
  - pen-red
  - recolor-images
tags:
  - python
  - venv
---

This is my RedTeam cheat sheet. 
## Python
### Upgrade Shell
To upgrade reverse shell using python use the follow command
```shell
python -c 'import pty;pty.spawn("/bin/bash")';
```

### Python venv
Create venv folder 
```shell
python -m venv venv
```

Activate virtual env
```shell
source venv/bin/activate
```
