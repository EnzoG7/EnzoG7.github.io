---
title: Linux终端常用命令
date: 2025-08-01 00:34:52
tags:
- Linux
- 命令行
categories:
- Linux
comments: true
---
Linux终端是与操作系统交互的重要工具，掌握常用命令可以提高工作效率。以下是一些常用的Linux终端命令。

<!-- more -->
# 基本格式
```bash
command [-options] [parameter]
```
command ：命令名,相应功能的英文单词或单词的缩写
[-options] ：选项,特殊参数标识
parameter ：传给命令的参数,可以是 零个、一个 或者 多个

# 快捷操作
快捷键 | 功能
-- | --
Ctrl + C | 终止当前命令
Ctrl + Z | 暂停当前命令
Ctrl + D | 退出终端
Ctrl + L | 清屏
Ctrl + U | 清空当前行
Ctrl + R | 搜索历史命令
Ctrl + A | 光标移动到行首
Ctrl + E | 光标移动到行尾

**自动补全**
输入文件或命令前几个字母后,按Tab键可以自动补全

# 常用命令
命令 | 说明 | 特殊参数
-- | -- | --
**ls** | 列出目录下的文件和子目录 | -l 详细信息, -a 显示所有包括隐藏文件
**cd** | 切换目录 | find [目录路径] -name "文件名"
**pwd** | 显示当前目录的路径 |
**clear** |  清屏 |
**mkdir** | 创建目录 | -p 创建所有不存在的路径的目录
**touch** | 创建文件 | 文件名
**rm** | 删除文件 | -f 强制删除, -r 递归删除目录
**cp** | 复制文件 | 源文件 目标文件, -i 覆盖提示, -r 递归复制目录
**mv** | 移动文件 | 源文件 目标文件, -i 覆盖提示
**cat** | 显示文件内容 | 文件名, -b 显示行号
**more** | 分页显示文件内容 | 文件名
**grep** | 搜索文件内容 | 搜索内容 文件名
**find** | 查找文件 | 查找路径 -name 文件名
**tar** | 压缩文件 | -zcvf 压缩文件名 源文件
**tar** | 解压文件 | -zxvf 压缩文件名

# vi/vim 编辑器
vi/vim 是一个强大的文本编辑器，常用于编辑配置文件等。

模式 | 操作 | 说明
-- | -- | --
普通模式 | 输入 vi 文件名 | 查看文件内容
插入模式 | 普通模式下输入 i,a,o | 从当前光标位置插入/光标下一个字符/行尾开始编辑文本
命令模式 | 普通模式下输入 : | 输入命令,例如wq(保存并退出),q!(不保存并退出)

# Docker命令
## 常用命令
命令 | 说明 | 特殊参数
-- | -- | --
**docker ps** | 列出运行中的容器 |
**docker ps -a** | 列出所有容器 |
**docker images** | 列出所有镜像 |
**docker run** | 运行容器 | -d 后台运行, -p 端口映射, --name 容器名称
**docker exec** | 在容器内执行命令 | -it 交互式
**docker stop** | 停止容器 | 容器ID或名称
**docker rm** | 删除容器 | 容器ID或名称
**docker rmi** | 删除镜像 | 镜像ID或名称, -f 强制删除
**docker build** | 构建镜像 | -t 镜像名称:标签 Dockerfile路径
**docker pull** | 从仓库拉取镜像 | 镜像名称:标签
**docker push** | 推送镜像到仓库 | 镜像名称:标签
**docker logs** | 查看容器日志 | 容器ID或名称
**docker network ls** | 列出网络

## 数据卷相关命令
命令 | 说明
-- | --
**docker volume create** | 创建数据卷 | 数据卷名称
**docker volume inspect** | 查看数据卷信息 | 数据卷名称
**docker volume rm** | 删除数据卷 | 数据卷名称
**docker volume prune** | 删除未使用的数据卷
**docker volume ls** | 列出所有数据卷