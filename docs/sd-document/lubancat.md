# LubanCat 机载电脑二次开发

## 软件概述

- 系统构建基于 ROS + Docker。
- 程序自启动基于 `crontab` 与 shell 脚本。

## 目录与脚本结构

- `/home/cat/uav_ws/docker_launch_px4.sh`：手动启动 Docker 并进入容器。
- `/home/cat/uav_ws`：工作空间，映射到容器内 `/workspace`。
- `/home/cat/uav_ws/launch/*`：启动脚本目录。

启动脚本说明：

- `launch_all.sh`：一键启动定位相关程序（Docker 外部执行，已配置开机自启动）。
- `launch_px4.sh`：飞控程序启动（Docker 内执行）。
- `launch_livox.sh`：雷达驱动启动（Docker 内执行）。
- `launch_fastlio.sh`：SLAM 定位启动（Docker 内执行）。
- `launch_uart.sh`：LubanCat 与 MCU 串口通信启动（Docker 内执行）。

源码目录说明：

- `/home/cat/uav_ws/src/livox_ros_driver2`：MID360 雷达驱动。
- `/home/cat/uav_ws/src/FAST_LIO_forUAV`：SLAM 定位源码。
- `/home/cat/uav_ws/src/uartforstm`：LubanCat 与 MCU 串口通信源码。

## 通信链路

- 雷达输入：`eth0:192.168.1.5` 接收雷达数据，发布到 `/livox/*` 话题。
- SLAM 处理：从 `/livox/*` 获取数据，经 fastlio 解析后发布 `/mavros/odometry/out`。
- 飞控对接：`/dev/ttyS1:921600`，LubanCat UART1 与飞控通信；将里程计发送给飞控用于定位。

## 开发注意事项

- 使用 `sudo crontab -e` 查看与修改开机自启动配置。
- 程序控制推荐发布到 `/mavros/setpoint_raw/local`（FLU 系），并遵循 MAVROS 标准。
- 推荐发布目标速度做闭环控制。
- 世界坐标系采用 `ENU`，机体坐标系采用 `FLU`。
