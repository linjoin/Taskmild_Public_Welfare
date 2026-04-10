# TaskMild 公益版 v0.4.1

- 工作目录与模块目录正式分离
- 运行目录调整为 `/data/adb/taskmild`
- 主程序迁移到 `/data/adb/taskmild/bin/taskm`
包 删除媒体检测相关逻辑
- 删除状态落盘与状态锁
- 运行状态改为纯内存缓存
- `normal` 模式改为先尝试结束，不退出再强制结束
- `deep` 模式保持更快直接强制结束
- 保留亮屏后台处理与熄屏处理逻辑
- 日志输出改为中文分级显示
- 新增 `module.prop` 运作状态提示，显示 PID 与配置路径
- `action.sh`、`service.sh`、`disable` 流程同步适配新目录结构
- 进一步清理冗余逻辑并减少文件 IO
