# TaskMild 公益版操作文档

## 1. 简介

TaskMild 是一个面向日常使用场景的进程压制模块。

公益版当前特点：

- 全局自动识别候选子进程
- 支持白名单包名和白名单进程名
- 支持普通压制与激进压制两种模式
- 支持熄屏处理
- 支持亮屏后台处理
- 默认不处理前台应用
- 主程序、配置、日志、运行文件与模块目录分离
- 通过日志和状态命令可查看运行情况

当前版本为纯内存运行态设计：

- 不使用状态落盘
- `recover` 不再支持按进程恢复
- 修改配置后需要手动重启守护或重启设备

---

## 2. 目录结构

### 工作目录

```text
/data/adb/taskmild/
  taskmild.conf
  run.log
  bin/
    taskmild
    taskmild.pid
```

### 模块目录

```text
/data/adb/modules/taskmild/
  module.prop
  service.sh
  action.sh
  uninstall.sh
```

---

## 3. 重要文件说明

### 主程序

```text
/data/adb/taskmild/bin/taskmild
```

### 配置文件

```text
/data/adb/taskmild/taskmild.conf
```

### 日志文件

```text
/data/adb/taskmild/run.log
```

### 模块信息

```text
/data/adb/modules/taskmild/module.prop
```

---

## 4. 配置文件完整示例

```ini
enable=1
mode=normal
screen_off=1
background=1
screen_on_cleanup=0
white_processes=com.tencent.mm,com.tencent.mobileqq,com.xiaomi.xmsf,com.android.vending,com.google.android.webview,com.miui.securitycenter,com.android.settings,com.xiaomi.finddevice
black_processes=
log_level=warn
scan_idle=30
scan_active=12
scan_screen_off=20
idle_backoff=120
term_grace=8
cooldown=120
foreground_refresh_interval=8
```

---

## 5. 配置项说明

### enable

模块总开关。

- `1` 为开启
- `0` 为关闭

### mode

压制模式。

- `normal`
  - 普通压制
  - 先请求结束进程
  - 不退出再强制结束
- `deep`
  - 激进压制
  - 更快进入强制结束

### screen_off

是否启用熄屏处理。

- `1` 启用
- `0` 关闭

### background

是否启用后台处理。

- `1` 启用
- `0` 关闭

### screen_on_cleanup

亮屏时是否处理后台应用。

- `1` 亮屏后台也处理
- `0` 亮屏时不处理后台

### white_processes

白名单。

支持两种写法：

- 写包名
- 写完整进程名

示例：

```ini
white_processes=com.tencent.mm,com.tencent.mobileqq:MSF
```

含义：

- `com.tencent.mm` 整个包都保护
- `com.tencent.mobileqq:MSF` 只保护该进程

### black_processes

黑名单。

只支持完整进程名。

示例：

```ini
black_processes=com.tencent.mm:appbrand1,com.tencent.mobileqq:tool
```

### log_level

日志级别。

可选值：

- `debug`
- `info`
- `warn`
- `error`

建议：

- 日常使用：`warn`
- 调试问题：`debug`

### scan_idle

亮屏空闲扫描间隔。

### scan_active

亮屏活跃扫描间隔。

### scan_screen_off

熄屏扫描间隔。

### idle_backoff

冷闲时退避间隔。

### term_grace

普通模式下，发送结束信号后等待的秒数。

### cooldown

进程处理后的冷却时间。

### foreground_refresh_interval

前台应用识别缓存时间。

---

## 6. 推荐配置

### 省电平衡版

```ini
enable=1
mode=normal
screen_off=1
background=1
screen_on_cleanup=0
white_processes=com.tencent.mm,com.tencent.mobileqq,com.xiaomi.xmsf,com.android.vending,com.google.android.webview,com.miui.securitycenter,com.android.settings,com.xiaomi.finddevice
black_processes=
log_level=warn
scan_idle=30
scan_active=12
scan_screen_off=20
idle_backoff=120
term_grace=8
cooldown=120
foreground_refresh_interval=8
```

### 效果优先版

```ini
enable=1
mode=deep
screen_off=1
background=1
screen_on_cleanup=1
white_processes=com.tencent.mm,com.tencent.mobileqq
black_processes=
log_level=info
scan_idle=20
scan_active=8
scan_screen_off=15
idle_backoff=90
term_grace=8
cooldown=90
foreground_refresh_interval=5
```

---

## 7. 修改配置的方法

编辑这个文件：

```text
/data/adb/taskmild/taskmild.conf
```

例如：

```sh
nano /data/adb/taskmild/taskmild.conf
```

或者用你自己的编辑器修改。

修改后需要手动重启守护，或者直接重启设备。

---

## 8. 启动、停止、开关

### 手动启动

```sh
/data/adb/taskmild/bin/taskmild run
```

### 单次执行一轮

```sh
/data/adb/taskmild/bin/taskmild once
```

### 查看当前状态

```sh
/data/adb/taskmild/bin/taskmild status
```

### 停止守护

```sh
/data/adb/taskmild/bin/taskmild stop
```

### 关闭并清运行态

```sh
/data/adb/taskmild/bin/taskmild disable
```

### 查看日志

```sh
/data/adb/taskmild/bin/taskmild log
```

查看最近 100 行：

```sh
/data/adb/taskmild/bin/taskmild log 100
```

### 快捷开关

```sh
sh /data/adb/modules/taskmild/action.sh
```

执行一次会自动切换开关：

- 当前开启时，执行后关闭
- 当前关闭时，执行后开启

---

## 9. 开机自启说明

开机后会由：

```text
/data/adb/modules/taskmild/service.sh
```

自动检查：

- 主程序是否存在
- 配置中的 `enable` 是否为 `1`

满足条件才会启动守护。

如果你不想开机自启，可以把配置改成：

```ini
enable=0
```

---

## 10. 如何确认模块是否正常运行

### 方法一：查看状态

```sh
/data/adb/taskmild/bin/taskmild status
```

重点看：

- `daemon_pid`
- `candidate_count`
- `screen_on`
- `foreground`
- `term_count`
- `kill_count`
- `last_action`

### 方法二：查看日志

```sh
/data/adb/taskmild/bin/taskmild log 100
```

如果看到下面这种日志，说明已经真正生效：

```text
[信息] 纳入处理 包=...
[信息] 请求结束 包=...
[信息] 强制结束 包=...
```

### 方法三：查看 `module.prop` 状态提示

```sh
grep '^description=' /data/adb/modules/taskmild/module.prop
```

可能看到：

```text
description=运行中 PID=1234 配置=/data/adb/taskmild/taskmild.conf
```

---

## 11. 日志级别说明

### debug

最详细，适合调试。

会记录：

- 环境
- 判定
- 决策
- 重扫情况
- 冷闲情况

### info

适合看实际动作。

会记录：

- 守护启动
- 单次启动
- 纳入处理
- 请求结束
- 强制结束

### warn

适合日常低打扰运行。

会记录：

- 锁忙
- 旧锁清理
- 退出超时
- 其他警告

### error

只记录错误。

---

## 12. 主进程、白名单、系统应用保护规则

### 主进程

当前只认：

- `进程名 == 包名`

例如：

- `com.tencent.mm` 视为主进程
- `com.tencent.mm:push` 不视为主进程
- `com.tencent.mm:main` 也不默认视为主进程

### 白名单

优先级最高。

命中白名单后不会处理。

### 系统应用

系统应用默认保护，不进入激进处理。

### 前台应用

当前前台包默认保护。

### top-app / foreground

命中该 cgroup 的进程默认保护。

---

## 13. 当前版本限制

当前公益版为纯内存运行态：

- 不使用状态落盘
- `status` 主要反映当前即时判断和日志统计
- `recover` 不支持按进程恢复
- `disable` 主要用于停止守护和清理运行态
- 修改配置后需要手动重启守护或重启设备

---

## 14. 常见问题

### 1）只看到守护启动日志，没有其他日志

这通常不是没运行，而是当前没有真正触发：

- 请求结束
- 强制结束

可以先看状态：

```sh
/data/adb/taskmild/bin/taskmild status
```

### 2）日志太少

把配置改成：

```ini
log_level=debug
```

然后重启守护。

### 3）模块太激进

先改成：

```ini
mode=normal
screen_on_cleanup=0
log_level=warn
```

### 4）感觉太保守，没有效果

可以尝试：

```ini
mode=deep
screen_on_cleanup=1
log_level=info
```

并减少白名单。

### 5）配置改了不生效

当前版本不做热重载。  
修改配置后请执行：

```sh
/data/adb/taskmild/bin/taskmild stop
/data/adb/taskmild/bin/taskmild run
```

或者直接重启设备。

### 6）`recover` 为什么没用

当前版本已经明确收敛为：

- 纯内存运行态
- 不支持按进程恢复

这是当前版本设计结果，不是故障。

---

## 15. 推荐操作流程

### 日常使用

1. 修改配置
2. 执行快捷开关或重启守护
3. 看状态
4. 看日志

### 配置后重启守护

```sh
/data/adb/taskmild/bin/taskmild stop
/data/adb/taskmild/bin/taskmild run
```

### 日志检查

```sh
/data/adb/taskmild/bin/taskmild log 100
```

### 状态检查

```sh
/data/adb/taskmild/bin/taskmild status
```

---

## 16. 卸载

模块卸载后，运行目录仍可能存在。  
执行：

```sh
sh /data/adb/modules/taskmild/uninstall.sh
```

会清理：

```text
/data/adb/taskmild
```

---

## 17. 最后说明

当前公益版的方向是：

- 保留核心处理能力
- 保留全局候选与亮屏后台处理
- 尽量减少无意义 I/O 和重复计算
- 尽量降低发热与功耗
- 尽量避免前台和系统关键进程被误处理

如果你要更激进的效果，优先从：

- 降低白名单数量
- 开启 `screen_on_cleanup`
- 使用 `deep`

开始调。

如果你要更省电，优先从：

- 关闭 `screen_on_cleanup`
- 使用 `normal`
- 提高扫描间隔

开始调。```sh
sh /data/adb/modules/taskmild/action.sh
```

执行一次会自动切换开关：

- 当前开启时，执行后关闭
- 当前关闭时，执行后开启

---

## 9. 开机自启说明

开机后会由：

```text
/data/adb/modules/taskmild/service.sh
```

自动检查：

- 主程序是否存在
- 配置中的 `enable` 是否为 `1`

满足条件才会启动守护。

如果你不想开机自启，可以把配置改成：

```ini
enable=0
```

---

## 10. 如何确认模块是否正常运行

### 方法一：看状态

```sh
/data/adb/taskmild/bin/taskmild status
```

重点看：

- `daemon_pid`
- `candidate_count`
- `screen_on`
- `foreground`
- `term_count`
- `kill_count`
- `last_action`

### 方法二：看日志

```sh
/data/adb/taskmild/bin/taskmild log 100
```

如果看到下面这种日志，说明已经真正生效：

```text
[信息] 纳入处理 包=...
[信息] 请求结束 包=...
[信息] 强制结束 包=...
```

### 方法
grep '^description=' /data/adb/modules/taskmild/module.prop
```

可能看到：

```text
description=运行中 PID=1234 配置=/data/adb/taskmild/taskmild.conf
```

---

## 11. 日志级别说明

### debug
最详细，适合调试。

会记录：

- 环境
- 判定
- 决策
- 重扫情况
- 冷闲情况

### info
适合看实际动作。

会记录：

- 守护启动
- 单次启动
- 纳入处理
- 请求结束
- 强制结束

### warn
适合日常低打扰运行。

会记录：

- 锁忙
- 旧锁清理
- 退出超时
- 其他警告

### error
只记录错误。

---

## 12. 主进程、白名单、系统应用保护规则

### 主进程
当前只认：

- `进程名 == 包名`

例如：

- `com.tencent.mm` 视为主进程
- `com.tencent.mm:push` 不视为主进程
- `com.tencent.mm:main` 也不默认视为主进程

### 白名单
优先级最高。

命中白名单后不会处理。

### 系统应用
系统应用默认保护，不进入激进处理。

### 前台应用
当前前台包默认保护。

### top-app / foreground
命中该 cgroup 的进程默认保护。

---

## 13. 当前版本限制

当前公益版为纯内存运行态：

- 不使用状态落盘
- `status` 主要反映当前即时判断和日志统计
- `recover` 不支持按进程恢复
- `disable` 主要用于停止守护和清理运行态
- 修改配置后需要手动重启守护或重启设备

---

## 14. 常见问题

### 1）只看到守护启动日志，没有其他日志
这通常不是没运行，而是当前没有真正触发：

- 请求结束
- 强制结束

可以先看状态：

```sh
/data/adb/taskmild/bin/taskmild status
```

### 2）日志太少
把配置改成：

```ini
log_level=debug
```

然后重启守护。

### 3）模块太激进
先改成：

```ini
mode=normal
screen_on_cleanup=0
log_level=warn
```

### 4）感觉太保守，没有效果
可以尝试：

```ini
mode=deep
screen_on_cleanup=1
log_level=info
```

并减少白名单。

### 5）配置改了不生效
当前版本不做热重载。  
修改配置后请执行：

```sh
/data/adb/taskmild/bin/taskmild stop
/data/adb/taskmild/bin/taskmild run
```

或者直接重启设备。

### 6）`recover` 为什么没用
当前版本已经明确收敛为：

- 纯内存运行态
- 不支持按进程恢复

这是当前版本设计结果，不是故障。

---

## 15. 推荐操作流程

### 日常使用
1. 修改配置
2. 执行快捷开关或重启守护
3. 看状态
4. 看日志

### 配置后重启守护
```sh
/data/adb/taskmild/bin/taskmild stop
/data/adb/taskmild/bin/taskmild run
```

### 日志检查
```sh
/data/adb/taskmild/bin/taskmild log 100
```

### 状态检查
```sh
/data/adb/taskmild/bin/taskmild status
```

---

## 16. 卸载

模块卸载后，运行目录仍可能存在。  
执行：

```sh
sh /data/adb/modules/taskmild/uninstall.sh
```

会清理：

```text
/data/adb/taskmild
```

---

## 17. 最后说明

当前公益版的方向是：

- 保留核心处理能力
- 保留全局候选与亮屏后台处理
- 尽量减少无意义 I/O 和重复计算
- 尽量降低发热与功耗
- 尽量避免前台和系统关键进程被误处理

如果你要更激进的效果，优先从：

- 降低白名单数量
- 开启 `screen_on_cleanup`
- 使用 `deep`
开始调。

如果你要更省电，优先从：

- 关闭 `screen_on_cleanup`
- 使用 `normal`
- 提高扫描间隔
开始调。
````- 支持熄屏场景
- 支持后台场景
- 支持三档预设模式
- 单主程序结构
- 单配置文件
- 支持基础调试和状态查看

---

## 适用环境

- Ksu及系列
- ap及系列
- 面具及系列

---

## 文件结构

```text
TaskMild/
  module.prop
  customize.sh
  service.sh
  taskmild.sh
  taskmild.conf
  action.sh
