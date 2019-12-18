---
typora-copy-images-to: image
---

## 一、Pycham的配置

### 1. 恢复 PyCharm 的初始设置

`PyCharm` 的配置信息是保存在用户家目录下的`.PyCharmxxxx.x` 目录下的，`xxxx.x` 表示当前使用的 `PyCharm` 的版本号。如果要恢复 `PyCharm` 的初始设置，可以按照以下步骤进行：

1. 关闭正在运行的 `PyCharm`

2. 在终端中执行以下终端命令，删除 `PyCharm` 的配置信息目录：

   ```bash
   $ rm -r ~/.PyCharm2016.3
   ```

3. 重新启动 `PyCharm`

