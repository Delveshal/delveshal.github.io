---
title: runc 笔记
date: 2018-06-12 22:31:41
tags: [docker,linux]
categories: docker
---

# main 函数
isRootless 判读是否不能以root权限启动
环境变量：`XDG_RUNTIME_DIR` ：针对每个用户的运行时目录。 通常针对每个用户单独挂载一个 "tmpfs" 文件系统实例。
os.ModeSticky ，filemode，特殊标记位，文件只允许创建者和root移除
因为默认记录runc运行状态是在`/run/runc`目录，如果runc不能以root权限启动则不够权限，需要用到`XDG_RUNTIME_DIR`。

# Create
1. checkArgs 检查参数个数
2. revisePidFile 修改pid-file的路径为绝对路径
3. setupSpec 读入配置文件，例如利用 `runc spec` 自动生成的 `config.json`
4. startContainer 启动容器
    1. newNotifySocket 需要用到环境变量`NOTIFY_SOCKET`，一般为空，暂时先不分析。
    2. createContainer 创建容器**对象**需要传入cli上下文，容器id，从配置文件生成的`spec`
        1. 调用 specconv 包的 CreateLibcontainerConfig  ，`specconv` 的作用是 "Package specconv implements conversion of specifications to libcontainer"
            1. 设置容器根文件系统目录
            2. 设置标签（labels，键值对）
            3. 根据create options（从 cli 中获取或自动生成的） 创建 `configs.Config` 对象
            4. 根据配置文件给`Config`对象配置mount(挂载点)
            5. createDevices 配置容器环境的白名单设备（例如：/dev/null, /dev/random）和用户指定设备
            6. createCgroupConfig 创建cgroup配置。**note：** “In rootless containers, any attempt to make cgroup changes will fail.” 依次处理devices，memory，cpu，pid，blockio，hugepagelimit，network。
            7. 配置 namespace，
                * NEWNET，如果`newnet` 并且没有配置路径，默认加入`loopback`（本地回环）
                * NEWUSER，配置user id 和 group id 的宿主主机和容器内部的映射。
            8. 处理配置文件的 process 字段。
            9. createHooks 配置钩子，`Prestart`，`Poststart`，`Poststop`

未完待续。。。
