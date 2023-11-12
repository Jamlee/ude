UDE（USB 设备仿真）虚拟 USB 设备示例（无硬件），以及匹配的主机端驱动程序。 用作USB学习测试台。 也可以作为开发其他虚拟 USB 设备的起点。
该项目是一个参考/示例， 问 https://docs.microsoft.com/en-us/windows-hardware/drivers/usbcon/usb-emulated-device--ude--architecture。  
该项目包含两个 Windows 10 内核模式驱动程序和一个测试应用程序（目前）。

## 驱动程序
内核模式驱动程序是：  
UDEFX2.sys ：示例 USB 设备模拟驱动程序（模拟虚拟 USB 控制器和连接到该虚拟控制器的虚拟 USB 设备）  
hostude.sys ：主机端驱动程序，最初从与 OSR FX2 设备通信的 WDK 示例中拷贝的，然后进行修改以匹配 UDEFX2.sys 定义的端点（见上文）  

UDEFX2.sys 定义了这些端点：  
- BULK/IN 端点：读取时生成模式数据。  
- BULK/OUT 端点：跟踪传入数据以进行确认。  
- 中断/输入端点：根据反向通道控制器测试应用程序的请求（通过反向通道 IOCTL），从虚拟设备生成中断。 如果虚拟设备处于低功耗模式，中断还会生成远程唤醒。

## 驱动程序构建：  

## VS2022
1. WDK 以及 Visual Studio 的 WDK 扩展 ，然后编译驱动文件。
2. 安装驱动程序，请将构建生成的所有 sys/inf/cat 文件放在单个目录中。参考
```
installem.bat
```
3. 测试驱动程序
（您需要将系统置于测试模式，因为驱动程序尚未签名）
要观察驱动程序的行为，您可以使用以下脚本：
tr_on.bat ：打开跟踪  
tr_off.bat：转储跟踪并停止  
在驱动程序安装/卸载期间或测试应用程序运行时（见下文）观察跟踪尤其具有指导意义。

## 全面测试
安装驱动程序后，您可以使用测试应用程序对其进行测试，该应用程序也是从 WDK 示例中窃取并进行修改的。 它可以通过几种方式使用：  
```
hostudetest.exe -a（进入循环等待通过 USB 的命令。这些命令可以使用 -c 标志从单独的实例发送）  
hostudetest.exe -c somemission
```
- 通过 BULK/OUT 发送“somemission”  
- 等待 INTERRUPT/IN 上的中断  
- 最后再读取USB/IN来响应任务  

仅中断测试  
```
hostudetest.exe -i abc（生成一个 INTERRUPT/IN 传输，其中包含与提供的十六进制参数匹配的 4 字节小端有效负载）  
hostudetest.exe -p（等待中断，该中断可以使用 -i 命令在测试应用程序的单独实例中生成 - 请参阅上面的 INTERRUPT/IN 端点描述
```
