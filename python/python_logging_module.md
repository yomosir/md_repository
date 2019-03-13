# logging module

## Introduction

python日志模块，用于打印日志

## code demo

```python
# -*-coding:utf-8-*-
import logging
from logging import handlers


class Logger(object):
    level_relations = {
        'debug': logging.DEBUG,
        'info': logging.INFO,
        'warning': logging.WARNING,
        'error': logging.ERROR,
        'critical': logging.CRITICAL
    }

    def __init__(self, s_name_log, log_level='debug'):
        self.logger = logging.getLogger(s_name_log)
        fmt = logging.Formatter('%(asctime)s - %(pathname)s[line:%(lineno)d] - %(levelname)s: %(message)s')
        self.logger.setLevel(self.level_relations[log_level])
        sh = logging.StreamHandler()
        sh.setFormatter(fmt)
        th = handlers.TimedRotatingFileHandler(filename=s_name_log, when='D', backupCount=3, encoding='utf-8')
        th.setFormatter(fmt)
        if not self.logger.handlers:
            self.logger.addHandler(sh)
            self.logger.addHandler(th)

    def debug(self, message):
        self.logger.debug(message)

    def info(self, message):
        self.logger.info(message)

    def warning(self, message):
        self.logger.warning(message)

    def error(self, message):
        self.logger.error(message)

    def critical(self, message):
        self.logger.critical(message)
```

### problem

1. logger封装会打印重复日志的问题


> 问题描述： 当声明另个日志文件相同的logger，那么就会在日志文件中将该条日志打印多次，每多声明一次就增加一次打印！！！


- 解决方法： 在logger封装类中添加对logger 的 handlers的判断：

```python
        if not self.logger.handlers:
            self.logger.addHandler(sh)
            self.logger.addHandler(th)
```
- 原因：如果每次声明一次就会往logger对象中添加一个handler，这样多次声明就会造成添加多个handler，导致多次打印