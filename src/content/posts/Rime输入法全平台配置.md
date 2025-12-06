---
title: Rime输入法全平台配置
pubDate: '2024-02-07'
---

## Rime方案选择
推荐方案：[雾凇拼音rime-ice](https://github.com/iDvel/rime-ice)。各平台的前端使用：Windows小狼毫，Mac鼠须管，Android小企鹅(Fcitx5，挂载Rime插件)，iOS/iPadOS仓输入法。
## 网盘选择
如果主力手机是iPhone，建议选择iCloud，后面会解释。主力机是Android，网盘的选择比较随意，我用的是OneDrive。下面说下各端的配置详情。
## Windows
小狼毫以默认配置安装在`C:\Users\你的用户名\AppData\Roaming\Rime`，将雾凇拼音rime-ice仓库的所有文件放在该文件夹下。
```markdown
C:\Users\你的用户名\AppData\Roaming\Rime
|   .gitignore
|   default.yaml
|   ......
|   installation.yaml
|   ......
+---build
+---cn_dicts
|   ......
\---rime_ice.userdb
```
修改`installation.yaml`。这样设置后，小狼毫点击同步时，会在D盘OneDrive云盘`/dev/Rime-ice`文件夹下自动创建文件夹`win`，里面包含同步的设置和词库。
```yaml
installation_id: win #任意名称，我按照设备的操作系统取，便于区分。
sync_dir: "F:\\onedrive\\dev\\Rime-ice" #根据网盘的位置，设置同步文件夹位置，注意双引号时的反斜杠转义。
```

自定义设置：以patch的方式打补丁，详见[官方文档](https://dvel.me/posts/rime-ice/#%e4%bb%a5-patch-%e7%9a%84%e6%96%b9%e5%bc%8f%e6%89%93%e8%a1%a5%e4%b8%81)。此种配置方式既能自定义输入法设置，如皮肤、词序等，又能在全仓更新rime-ice的同时保留个人设置项。
## Mac
设置方法与Windows端大同小异，修改`installation.yaml`。
```yaml
installation_id: mac
sync_dir: "/Users/pk/Library/CloudStorage/OneDrive-个人/dev/Rime-ice" #根据网盘的位置，设置同步文件夹位置。
```
## Android
Fcitx5小企鹅，不用写同步目录，app会自动生成同步文件夹`sync`。
1. 但仍需修改`installation.yaml`里`installation_id`的值，假设改为`android`。app点击同步后，会在`sync`文件夹下创建名为`android`的文件夹，里面包含该设备同步的设置和词库。
2. 利用第三方应用同步，如FolderSync。将`android`文件夹推送到OneDrive云盘`/dev/Rime-ice`文件夹下。
3. **与此同时，还需将远端`/dev/Rime-ice`文件夹里其余设备的同步文件夹拉取到本机的`sync`文件夹下，比如上面提到的`win`和`mac`。**
## iOS/iPadOS
仓输入法现阶段只能使用iCloud同步和备份，所以将iPhone作为主力机的用户，建议使用iCloud来多端同步。

## 词库同步逻辑
引擎在部署时，会自动索引同步目录下所有文件夹里的`*.userdb.txt`词库文件，并进行合并，使多端设备的词库使用体验得到统一。

## 补充
如果使用双拼输入，个人更推荐自然码。该方案在多平台有着更好的通用性和普适性。