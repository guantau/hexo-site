---
title: Django的中文设置
date: 2017-09-30 12:35:00
categories:
tags:
---

Fuck!!!

中文需要的是 zh_Hans 而不是 zh-hans，在APP的目录下面创建目录`locale`，
然后生成命令为

```shell
$ django-admin makemessages -l zh_Hans
$ django-admin compilemessages -l zh_Hans
```

如果想要某个APP下的locale生效，需要将其加入`settings.py`的`INSTALL_APPS`中。

如果还有额外的locale文件生效，可以放在项目目录下，并在`settings.py`中做如下设置：

```python
LOCALE_PATHS = (
    os.path.join(BASE_DIR, 'locale'),
)
```

