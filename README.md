# yiyuanSLocal
智能存储货柜一体化管理系统
项目概述
项目背景
智能存储货柜管理系统原采用分布式架构，由Android客户端、Java服务端、Web管理后台和MySQL数据库组成。为简化部署、降低运维成本并实现离线运行，本项目将原有系统整合为单一Android应用，在RK3568平台上实现全栈本地化部署。

核心特性
✅ 一体化部署：所有服务（MySQL、Spring Boot、Web管理界面）集成在单一APK中

✅ 离线运行：不依赖外部网络和服务器，断网情况下正常工作

✅ 无缝迁移：原有Android应用仅需修改服务地址即可接入

✅ 完整功能：保持原有业务逻辑和接口兼容性

✅ 本地管理：通过内置Web界面进行系统管理和监控

硬件要求
组件	最低要求	推荐配置
CPU	四核Cortex-A55 1.5GHz	RK3568四核2.0GHz
内存	4GB	8GB LPDDR4/X
存储	64GB eMMC	256GB eMMC 5.1
系统	Android 11（API 30）	Android 11/12
网络	以太网或WiFi	千兆以太网+WiFi 6
显示	720p触摸屏	1080p触摸屏
软件架构
整体架构
text
Android APK容器
├── 原生服务层（NDK）
│   ├── MySQL 8.0（ARM64优化版）
│   ├── Nginx 1.22（反向代理）
│   └── 服务管理器（Native）
├── Java应用层
│   ├── Spring Boot 2.7服务
│   ├── 前台服务管理
│   └── 权限与资源管理
├── Web界面层
│   └── Vue 3管理界面（预编译静态资源）
└── 客户端适配层
    └── 原APK应用（仅修改配置）
网络架构
服务地址：所有服务绑定127.0.0.1（本地回环）

服务端口：

MySQL: 3306

Spring Boot API: 8080

Nginx Web管理: 80

通信协议：HTTP/HTTPS（本地）、TCP/IP（数据库）

快速开始
环境准备
开发环境：

Android Studio Arctic Fox（2021.3.1）或更高版本

Android NDK r25c

JDK 11

Node.js 16+（用于构建Web界面）

构建工具：

bash
# 安装必要依赖
npm install -g @vue/cli
mvn --version  # 确保Maven 3.6+
./gradlew --version  # 确保Gradle 7.4+
构建流程
1. 克隆项目
bash
git clone https://github.com/dcjk2010/yiyuanSLocal.git
cd yiyuanSLocal
2. 构建Web管理界面
bash
cd web-admin
npm install
npm run build:arm64
# 构建产物将自动复制到Android项目assets目录
3. 构建Spring Boot服务
bash
cd spring-boot-server
mvn clean package -Parm64 -DskipTests
# 生成的可执行JAR将自动复制到Android项目
4. 构建Android应用
bash
cd android
# 配置环境变量（可选）
export JAVA_HOME=/path/to/jdk11
export ANDROID_HOME=/path/to/android/sdk

# 构建APK
./gradlew assembleArm64Release
安装部署
首次安装
安装APK：

bash
adb install app/build/outputs/apk/arm64/release/app-arm64-release.apk
授予必要权限：

网络访问权限

前台服务权限

存储访问权限（如果使用外部存储）

启动服务：

bash
# 通过ADB启动服务
adb shell am start-foreground-service \
  -n com.smartstorage/.service.StorageForegroundService
配置原APK
修改原APK的服务配置，将API地址指向本地：

java
// 修改前
private static final String API_BASE = "http://remote-server:8080";

// 修改后
private static final String API_BASE = "http://127.0.0.1:8080";
配置说明
应用配置（config.properties）
properties
# 服务器配置
server.port=8080
server.address=127.0.0.1

# 数据库配置
db.path=/data/data/com.smartstorage/databases/storage.db
db.max_connections=20

# Web管理配置
web.admin.port=80
web.admin.path=/admin

# 性能配置
memory.limit.mb=2048
cpu.cores=4

# 备份配置
backup.enabled=true
backup.interval.hours=6
backup.retention.days=7
网络配置（network_security_config.xml）
xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="true">localhost</domain>
        <domain includeSubdomains="true">127.0.0.1</domain>
    </domain-config>
</network-security-config>
Android权限配置
xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />
服务管理
服务启动顺序
MySQL数据库服务（端口3306）

Spring Boot应用服务（端口8080）

Nginx Web服务器（端口80）

健康检查服务（端口8081）

服务监控
bash
# 查看服务状态
adb shell dumpsys activity services com.smartstorage/.service.StorageForegroundService

# 查看服务日志
adb logcat -s StorageService:D MySQLService:D SpringBootService:D

# 检查端口状态
adb shell netstat -tlnp | grep -E '3306|8080|80'
服务重启
bash
# 重启所有服务
adb shell am broadcast -a com.smartstorage.RESTART_SERVICES

# 重启单个服务
adb shell am broadcast -a com.smartstorage.RESTART_SERVICE --es service_name "mysql"
数据管理
数据库初始化
首次启动时，系统会自动：

创建SQLite/MySQL数据库结构

导入初始数据（如果存在）

创建必要的索引和视图

数据备份
bash
# 手动备份数据库
adb shell am broadcast -a com.smartstorage.BACKUP_DATABASE

# 查看备份列表
adb shell ls /sdcard/Android/data/com.smartstorage/files/backup/

# 恢复数据库
adb shell am broadcast -a com.smartstorage.RESTORE_DATABASE \
  --es backup_file "backup_20240115_120000.sql.gz"
数据迁移（从远程MySQL）
导出远程数据：

bash
mysqldump -h remote-server -u user -p database > backup.sql
转换格式（如果需要）：

bash
python3 tools/mysql_to_sqlite.py backup.sql storage.db
导入本地数据库：

bash
adb push storage.db /sdcard/Android/data/com.smartstorage/files/
adb shell am broadcast -a com.smartstorage.IMPORT_DATABASE
监控与诊断
健康检查
bash
# 检查服务健康状态
curl http://127.0.0.1:8080/actuator/health

# 查看系统指标
curl http://127.0.0.1:8080/actuator/metrics

# 查看应用信息
curl http://127.0.0.1:8080/actuator/info
性能监控
bash
# 查看CPU使用率
adb shell top -n 1 | grep -E 'mysql|nginx|java'

# 查看内存使用
adb shell dumpsys meminfo com.smartstorage

# 查看存储空间
adb shell df -h /data/data/com.smartstorage
日志管理
bash
# 查看实时日志
adb logcat -s SmartStorage:* *:S

# 导出日志文件
adb pull /data/data/com.smartstorage/files/logs/app.log

# 清理日志
adb shell am broadcast -a com.smartstorage.CLEANUP_LOGS
故障排除
常见问题
1. 服务启动失败
症状：应用安装后服务无法启动
解决方案：

bash
# 检查权限
adb shell pm list permissions | grep smartstorage

# 查看错误日志
adb logcat -s AndroidRuntime:E

# 重新授予权限
adb shell pm grant com.smartstorage android.permission.FOREGROUND_SERVICE
2. 端口冲突
症状：服务启动时提示端口被占用
解决方案：

bash
# 查看端口占用
adb shell netstat -tlnp

# 终止占用进程
adb shell kill -9 <pid>

# 或者修改配置使用其他端口
adb shell am broadcast -a com.smartstorage.RECONFIGURE_PORTS
3. 内存不足
症状：应用频繁崩溃或服务自动停止
解决方案：

bash
# 清理内存缓存
adb shell am broadcast -a com.smartstorage.CLEAN_MEMORY

# 调整内存限制
adb shell setprop com.smartstorage.memory.limit 2048

# 重启应用
adb shell am force-stop com.smartstorage
adb shell am start com.smartstorage/.MainActivity
4. 数据库损坏
症状：数据无法读取或写入
解决方案：

bash
# 检查数据库完整性
adb shell sqlite3 /data/data/com.smartstorage/databases/storage.db "PRAGMA integrity_check;"

# 恢复最近备份
adb shell am broadcast -a com.smartstorage.RECOVER_DATABASE

# 重建数据库
adb shell am broadcast -a com.smartstorage.REBUILD_DATABASE
调试模式
启用调试模式获取更详细的日志：

bash
# 启用调试模式
adb shell am broadcast -a com.smartstorage.SET_DEBUG_MODE --ez enabled true

# 查看调试日志
adb logcat -s SmartStorageDebug:* *:S

# 禁用调试模式
adb shell am broadcast -a com.smartstorage.SET_DEBUG_MODE --ez enabled false
开发指南
项目结构
text
yiyuanSLocal/
├── android/                    # Android主项目
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── cpp/           # NDK原生代码
│   │   │   ├── java/         # Java业务代码
│   │   │   ├── assets/       # 静态资源
│   │   │   └── res/          # 资源文件
│   │   └── build.gradle
│   ├── libs/                  # 预编译库
│   └── build.gradle
├── spring-boot-server/        # Spring Boot后端
│   ├── src/main/java/
│   ├── src/main/resources/
│   └── pom.xml
├── web-admin/                 # Vue管理界面
│   ├── src/
│   ├── public/
│   └── package.json
├── mysql-arm64/              # MySQL ARM64构建
├── nginx-arm64/              # Nginx ARM64构建
└── docs/                     # 文档
代码贡献
分支策略：

main：稳定版本

develop：开发分支

feature/*：功能分支

hotfix/*：紧急修复分支

提交规范：

text
type(scope): subject

body

footer
type: feat, fix, docs, style, refactor, test, chore

scope: android, server, web, docs, etc.

测试
bash
# 运行单元测试
cd android && ./gradlew test

# 运行集成测试
cd android && ./gradlew connectedAndroidTest

# 构建测试APK
cd android && ./gradlew assembleDebugAndroidTest
版本发布
版本号规则
采用语义化版本控制：主版本.次版本.修订版本

主版本：重大架构变更，不向下兼容

次版本：功能新增，向下兼容

修订版本：Bug修复，向下兼容

发布流程
更新版本号（gradle.properties和pom.xml）

更新CHANGELOG.md

创建发布分支

执行完整测试

构建发布版本APK

提交到应用商店/OTA服务器

合并到主分支并打Tag

许可证
text
Copyright 2024 RydenQ

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
联系与支持
作者：RydenQ

邮箱：dcjk2010@hotmail.com

项目主页：https://github.com/dcjk2010/yiyuanSLocal

文档：https://dcjk2010.github.io/yiyuanSLocal

问题反馈：https://github.com/dcjk2010/yiyuanSLocal/issues

更新日志
v1.0.0 (2024-01-15)
初始版本发布

集成MySQL、Spring Boot、Vue全栈服务

支持RK3568 Android 11平台

实现本地一体化部署

