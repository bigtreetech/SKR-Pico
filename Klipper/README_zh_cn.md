# [View English version](./README.md)

# 在 SKR Pico V1.0 主板上使用 Klipper

## Pinout
### 树莓派 pinout
<img src=Images/rpi_pinout.jpg width="800" /><br/>

### SKR Pico V1.0 pinout
<img src=Images/pinout.png width="1600" /><br/>

## 接线图
### 使用额外的5V电源适配器给树莓派供电，并且通过 USB 与 SKR Pico V1.0 通信
<img src=Images/wiring_usb.png width="1600" /><br/>

### 使用主板的+5V给树莓派供电，并且通过 UART 与 SKR Pico V1.0 通信
<img src=Images/wiring_uart.png width="1600" /><br/>

## 编译固件
1. 已编译好直接使用的文件(此二进制文件所使用的源码是 [Commits on Dec 13, 2021](https://github.com/Klipper3d/klipper/commit/323268ea02700b2fd3b6accda310845ba29c894e))
   * [klipper-USB.uf2](./klipper-USB.uf2) 使用 USB 与树莓派通信。直接通过数据线将树莓派的USB-A连接到主板的USB-C接口即可正常通信。
   * [klipper-UART0.uf2](./klipper-UART0.uf2) 使用 UART0 与树莓派通信. 通过杜邦线将树莓派的 UART-TX 接到主板的 UART-RX0 ，将树莓派的 UART-RX 接到主板的 UART-TX0 ，并且将地线连接在一起即可正常通信。

2. 自行编译最新版本的固件<br/>
   1. 参考 [klipper官方的安装说明](https://www.klipper3d.org/Installation.html) 下载klipper源码到树莓派
   2. 使用下面的配置去编译固件
      * [*] Enable extra low-level configuration options
      * Micro-controller Architecture = `Raspberry Pi RP2040`
      * 如果使用 USB 与树莓派通信
         * Communication interface = `USB`
      * 如果使用 USART2 与树莓派通信
         * Communication interface = `Serial (on UART0 GPIO1/GPIO0)`

      <img src=Images/klipper_menuconfig.png width="800" /><br/>
   3. 配置选择完成后, 输入 `q` 退出配置界面，当询问是否保存配置是选择 "Yes" .
   4. 输入 `make` 命令开始编译固件
   5. 当 `make` 命令执行完成后，会在树莓派的`home/pi/kliiper/out`的文件夹中生成我们所需要的`klipper.uf2`固件。你可以在CMD命令行终端中通过`pscp`命令把`klipper.uf2`固件复制到与树莓派在同一个局域网下的电脑上。例如 `pscp -C pi@192.168.0.101:/home/pi/klipper/out/klipper.uf2 c:\klipper.uf2`(命令行会提示 `The server's host key is not cached` 并且询问 `Store key in cache?((y/n)`, 输入 `y` 保存 host key，然后输入树莓派默认的密码：`raspberry`)

## 更新固件
1. 你可以使用 [编译固件 2.5](#编译固件) 中的方法或者使用 `cyberduck`、 `winscp` 工具软件从树莓派中将 `klipper.uf2` 文件复制到电脑上
2. 将`Boot`脚插上跳帽，并且单击`Reset`按钮进入烧录固件模式(注意: 如果你想使用 USB 给主板供电，需要在`USB Power`脚上也插上跳帽，当有外部的 12V / 24V 供电时，最好拔掉`USB Power`的跳帽)
   <img src=Images/boot.png width="800" /><br/>
3. 通过 USB-C 把主板连接到电脑上，然后电脑会识别出一个名为`RPI-PR2`的U盘设备，把我们提供的`klipper-USB.uf2`、 `klipper-UART0.uf2`或者自行编译的 `klipper.uf2`固件复制到此U盘中，然后主板会自动重启并更新固件，当电脑重新设别出U盘设备时，意味着固件更新完成，拔掉`Boot`上的跳帽，并且单机`Reset`按钮进入正常工作模式
   <img src=Images/msc.png width="800" /><br/>
4. 你可以输入 `ls /dev/serial/by-id` 查询主板的串口ID来确认固件是否烧录成功，如果烧录成功了会返回一个klipper的设备ID，如下图所示:

   ![](./Images/rp2040_id.png)

   (注意: 此步骤仅适用于USB通信的工作方式，如果使用USART通信则没有这种ID)

## 配置打印机的参数
### 基础配置
1. 参考 [klipper官方的安装说明](https://www.klipper3d.org/Installation.html) to `Configuring OctoPrint to use Klipper`
2. 参考 [klipper官方的安装说明](https://www.klipper3d.org/Installation.html) to `Configuring Klipper`. 并且使用我们提供的配置文件 [SKR Pico klipper.cfg](./SKR%20Pico%20klipper.cfg) 为基础去修改 `printer.cfg`, 此文件中包含了主板几乎所有的pinout
3. 参考 [klipper官方的配置说明Config_Reference](https://www.klipper3d.org/Config_Reference.html) 去配置你想要的特性和功能
4. 如果你想通过USB与树莓派通信，运行 `ls /dev/serial/by-id/*` 命令去查询主板的设备ID号，在 `printer.cfg` 设置查询到的实际设备ID号，并且接线图请参考[这里](#使用额外的5v电源适配器给树莓派供电并且通过-usb-与-skr-pico-v10-通信)
    ```
    [mcu]
    serial: /dev/serial/by-id/usb-Klipper_rp2040_E66094A027831922-if00
    ```
5. 如果你想通过UART0与树莓派通信, 你需要修改一下的文件(你可以通过SSH终端输入命令修改，也可以直接修改树莓派系统SD卡中的文件)，并且接线图请参考 [这里](#使用主板的5v给树莓派供电并且通过-uart-与-skr-pico-v10-通信)
   * 在 `/boot/cmdline.txt` 文件中删除 `console=serial0,115200` 
   * 在 `/boot/config.txt` 文件的末尾添加 `dtoverlay=pi3-miniuart-bt`
   * 修改配置文件 `printer.cfg` 的 `[mcu]` 部分为 `serial: /dev/ttyAMA0` 并且添加 `restart_method: command`
     ```
     [mcu]
     serial: /dev/ttyAMA0
     restart_method: command
     ```
