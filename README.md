# uipath-proj-tmpl

UiPath Project Template
该代码库可用作通用UiPath的项目模板，适用于开发常见的RPA流程，本项目集成RPA流程开发过程中常用功能并强制相关规范，主要针对实施RPA无人值守流程，解决常见的开发及后期运维的问题。使用本项目模板可提升PRA流程自动化过程的开发效率及实施质量，降低后期维护成本。

## RPA基础规范
本项目相关规范不涉及行政规范，请根据自身公司的具体行政流程，结合本项目使用。

### 角色定义
* **RPA管理员：** IT部门统筹RPA流程的技术负责人，统筹业务流程自动化的实施工作人员，例如：RPA技术经理/主管；
* **RPA运维人员：** IT部门负责RPA流程日常的运行维护工作的人员，负责流程监控、机器人状态监控、流程运行状态监控等职责，例如：RPA运维工程师；
* **RPA开发人员：** IT部门负责开发RPA流程的相关人员，负责自动化流程的需求分析、流程设计以及流程开发等任务，例如：RPA流程开发工程师或RPA流程设计工程师；
* **业务负责人：** 业务部门负责具体业务流程正常开展的人员，在流程未实施自动化前，需要人工操作业务流程的业务相关人。或从行政职能上，需要对业务执行结果负责审核或确认的人员，例如：负责每日向管理层发送每日销售报告的员工；
* **业务最终用户：** 业务流程执行过程中，最终受业务执行结果影响（受益）的人员，例如：每天接收ROI报告的人员；

### 告警机制
* **通知级别：** 流程运行过程中，达到特定的步骤时，需要向业务负责人通报流程进度时，发送的通知信息。例如：流程成功运行后，需要向**业务负责人**发送“流程运行成功”的通知邮件/短信，以便**业务负责人**确认执行结果；
* **警告级别：** 流程运行过程中，遇到非RPA程序自身问题导致的失败（例如：预设的用户名密码错误、业务数据错误）时，发送的警告信息。例如：预设的用户名密码不匹配，导致登陆步骤提示失败，需要向**业务负责人**及**RPA运维人员**发送警告邮件，以便相关人员确认问题并后续跟进；
* **异常级别：** 流程运行过程中，因RPA程序自身缺陷导致的失败（例如：Element Selector错误导致元素抓取失败）时，发送的错误报告信息。例如：业务系统界面更新导致RPA流程执行失败，需要通知RPA运维人员及RPA开发人员更新流程程序；

### 检查点机制(Checkpoint)
开发无人值守流程时，需要设置检查点。检查点为流程执行的关键步骤结果核验，决定流程是否可以继续往下执行。例如：某个自动化流程需要定时下载报表并通过邮件发送给业务相关人员，则是否成功下载报表决定了后续的发送邮件是否需要继续执行。那么，“报表文件”是否存在就是这个流程的一个检查点，如果报表文件下载失败，中止后续操作并发送警告邮件给“业务负责人”。

本项目通过捕捉CheckpointException达成检查点机制。使用UiPath.Core.Activities.CheckTrue及UiPath.Core.Activities.CheckFalse可以出发检查点。
例子：通过使用UiPath组件“Activity: Check True”，填写Expression: `File.Exists("C:\path\to\download_file.xlsx")`及ErrorMessage: `"文件下载失败！"`，则可以达成文件下载失败后，自动发送失败信息的警告邮件。无需增加大量的流程分支If。

### 全局配置
本项目使用全局配置机制，通过参数传递，将配置传递至子流程文件。

**全局配置参数：** 项目的流程文件，均需设置传入参数dictConfig，全局配置参数（参数类型：Dictionary(Of String, Object))。调用时，将全局配置对象传入，以便全局引用。

**全局配置文件：** 调用主流程文件前，模板会先判断流程启动参数是否指定配置文件路径，如有，则使用启动参数指定的配置文件；如无，则从项目当前目录检索是否存在“config.json”或“config.xlsx”文件，并设置为全局设置文件，读取至全局配置对象（dictConfig）中。
出于安全考虑，本模板支持读取密码保护的Excel文件。使用命令行启动流程，或下发任务时指定相应的参数。如：
```
C:\>SET RPA_CONFIG_PASSWORD=P@ssw0rd
C:\>\path\to\uipath\UiRobot.exe execute --file "\path\to\project\project.json" --input "{'dictArgs': {'config_file': '\path\to\config_file.xlsx'}}"
```
**环境分离：** 根据不同的部署环境，修改配置文件：config_dev.json、config_uat.json、config_prd.json，并交由不同权限的人员进行管理及使用。
建议部署方式：将生产环境的配置文件config_prd.json集中放置于本地硬盘。发布流程时，指定流程的启动参数，并将实际生产环境配置文件路径传入，以达到较好的环境分离效果。

## 如何使用？

### 创建项目
1. Clone项目至本地；
2. 使用UiPath Studio打开本项目；
3. 点击“Save as Template”并完成保存，将本项目保存为项目模板；
4. 使用“New from Template”功能，以刚保存的项目模板创建新项目；
5. 将“EntryPoint.xaml”设置为主要（Set as Main)；
6. 打开“config.json”文件，修改相关配置项；
7. 打开“Main.xaml”文件，开发RPA流程；
8. 点击“Debug”按钮测试流程；

### 配置项
* **DEBUG:** 调试标志，设置为true，则会在运行流程时，输出配置文件内容，部署生产环境时，请务必关闭此选项。默认值：True；
* **MAIN_XAML:** 主流程文件，即启动流程后，会自动调用该配置指定的主要流程文件。默认值：Main.xaml；
* **PRE_MAIN_XAML:** 预执行流程文件，在启动**MAIN_XAML**前，会启动本配置项指定的流程文件。默认值：空字符串；
* **POST_MAIN_XAML:** 后执行流程文件，在启动**MAIN_XAML**后，会启动本配置项指定的流程文件。默认值：空字符串；
* **PROCESS_NAME:** 流程名称，在发送告警邮件时，使用该名称作为邮件标题发送，必填；
* **MAX_RETRIES:** 最大重试次数。如指定重试次数，则在流程执行过程中，如发生异常，会尝试按照**PRE_MAIN_XAML** -> **MAIN_XAML** -> **POST_MAIN_XAML** 的顺序重新执行流程。默认值：0（不重试）
* **PROCESS_TIMEOUT:** 流程执行超时限制，单位为毫秒。限制流程执行的总时间长度不得超过本项设置，包括重试。默认值：3600000（1小时），如无需限制，则修改为0或负数；
* **EMAIL_PROTOCOL:** 告警邮件的发送方式，支持SMTP/EXCHANGE/OUTLOOK；
* **EMAIL_HOST:** 告警邮件服务的服务器地址；
* **EMAIL_PORT:** 告警邮件服务的服务器端口；
* **EMAIL_USER:** 告警邮件服务的登陆用户名；
* **EMAIL_PSWD:** 告警邮件服务的登陆密码；
* **NOTIFICATION_TYPE:** 告警提示方式，暂时只支持EMAIL方式；
* **EMAIL_NOTIFY_INFO:** **通知级别**邮件的收件人列表。一般情况下，指定**业务负责人**及**RPA运维人员**为收件人；
* **EMAIL_NOTIFY_WARN:** **警告级别**邮件的收件人列表。一般情况下，指定**业务负责人**、**RPA运维人员**及**RPA管理员**为收件人；
* **EMAIL_NOTIFY_EXCE:** **异常级别**邮件的收件人列表。一般情况下，指定**RPA运维人员**、**RPA开发人员**及**RPA管理员**为收件人；
* **TAKE_SCREENSHOT:** 发送告警邮件时，是否附加当前屏幕截屏为附件；
* **DISABLE_INFO:** 禁用**通知级别**邮件功能；
* **DISABLE_WARN:** 禁用**警告级别**邮件功能；
