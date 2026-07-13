---
title: "Solution"
description: "$ docker"
draft: false
weight: 3
---
### Solution

```yaml
$ docker run --rm -it -e LANG=fr_CA.UTF-8 -e PS1="[\u \t ]\$ " alpine
```

```yaml
$ nano envfile
LANG=fr_CA.UTF-8
PS1=[\u \t ]\$ 

$ docker run --rm -it --env-file ./envfile alpine
```








