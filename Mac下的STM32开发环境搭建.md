
## 1.背景和目的

因为 Keil5 MDK 的编辑界面不太友好，我过去一直在寻找替代方法。  
之前使用 VSCode 搭配 koroFileHeader 插件进行代码编辑，然后在 MDK 中调试。  
这种开发方式部署简单，但调试时需要来回切换编辑器和调试器，体验并不理想。

既然要更换环境，我就寻找一个一步到位的全Free方案，最终选择了以下工具链：
VSCode + Arm GNU Toolchain + OpenOCD + ST-Link

## 2. 开发环境

- macOS 版本：Ventura 13.x (Apple M2)  
- VSCode 版本：1.103.0 
- Arm GNU Toolchain 版本：14.3 Rel1 (官方包安装)  
- OpenOCD 版本：0.12.0 (Homebrew安装)  
- 硬件：STM32F103ZET6，ST-Link V2  

## 3.安装工具

### 3.1.安装Homebrew（MAC包管理器）

打开终端执行:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装完成后，执行：

```bash
brew --version
```

确认显示版本号表示安装成功。

### 3.2. 安装 OpenOCD

```bash
brew install openocd
```

安装完成后，确认一下版本：

```bash
openocd --version
```

### 3.3.安装Arm GNU Toolchain for macOS M1/M2

下载安装包：[链接](https://developer.arm.com/-/media/Files/downloads/gnu/14.3.rel1/binrel/arm-gnu-toolchain-14.3.rel1-darwin-arm64-arm-none-eabi.pkg)

具备图形化界面，下载完成后正常安装即可
安装完成后配置环境变量（路径修改为自己的路径）

编辑你的 shell 配置文件（比如 `~/.zshrc` 或 `~/.bash_profile`），加入：

```bash
export PATH=/opt/arm-gcc/bin:$PATH
```

保存后执行：

```bash
source ~/.zshrc
```

安装完成后，确认一下版本：

```bash
arm-none-eabi-gcc --version
```


### 3.4.安装VSCode插件

**此处默认已经安装好了VSCode 没有安装的自行官网下载**

打开安装如下插件：

1. **C/C++** （微软官方插件）  
    提供代码智能提示。
2. **Cortex-Debug**  
	用于GDB调试STM32，支持变量、寄存器、断点等。
3. **Makefile Tools**（可选）  
	如果你用Makefile管理项目，建议安装。



## 4.创建工程

### 4.1.使用STM32CubeMX创建代码

- 打开 STM32CubeMX（你应该已经有了，如果没有，可以去 ST 官网下载 Mac 版本）
- 新建工程，选择你的 STM32 芯片型号。
- 在 “Project Manager” 里，配置：
    - Toolchain / IDE 选择 **Makefile**（这样生成的工程可以用命令行编译，不绑定 Keil 或其他IDE）
    - Project Name 和路径自定义。
- 配置完后，点击 **Generate Code**。


### 4.2.编译初始工程

1. 打开 VSCode，选择你CubeMX生成的工程文件夹。
2. 打开终端（VSCode内置或者Mac终端），执行编译命令：

```bash
make
```
如果CubeMX生成的Makefile配置正确，应该能成功编译）


## 5.配置调试

### 5.1.创建调试配置文件

工程目录下创建一个 .vscode文件夹后将下面的json文件都放在目录下即可

文件位置：`_attachments/launch.json`  
用途：用于 VSCode 调试 STM32 的配置，包括调试器类型、程序路径和调试选项。  
点击打开：![[_attachments/launch.json]]

文件位置：`_attachments/tasks.json`  
用途：定义 VSCode 的编译任务，例如调用 Makefile 编译项目。  
点击打开：![[_attachments/tasks.json]]

文件位置：`_attachments/c_cpp_properties.json`  
用途：为 VSCode 的 C/C++ 插件提供头文件路径、宏定义和编译器信息，解决预览报错。  
点击打开：![[_attachments/c_cpp_properties.json]]


### 5.3.下载SVD文件

SVD（System View Description）文件是芯片寄存器的描述文件，可让调试器在调试时直观显示寄存器信息。

- **CMSIS Pack 官方仓库**  
  [https://www.keil.arm.com/packs/](https://www.keil.arm.com/packs/)  

1. 打开链接，在搜索栏输入目标芯片型号。  
2. 点击对应的芯片系列进入下载页面，例如 **STM32F1**：  
   [https://www.keil.com/pack/Keil.STM32F1xx_DFP.2.4.1.pack](https://www.keil.com/pack/Keil.STM32F1xx_DFP.2.4.1.pack)  

> `.pack` 文件是 Keil CMSIS Pack 的专用格式，本质上是一个压缩包，其中包含芯片支持文件（含 `.svd` 文件）。

3. 将下载的 `.pack` 文件重命名为 `.zip`。  
4. 使用系统自带解压工具或第三方解压软件解压。  
5. 在解压后的 `SVD` 文件夹中找到 **STM32F103xx.svd** 文件，并将其复制到工程根目录。  
6. 重新打开 VSCode，调试器即可识别寄存器信息。  


## 6.总结

本教程介绍了如何在 macOS 平台上，基于 **VSCode + Arm GNU Toolchain + OpenOCD + ST-Link** 搭建一套完全免费的 STM32 开发与调试环境。  
整个过程包括安装必要的工具链、生成可编译的 Makefile 工程、在 VSCode 中配置调试环境，以及通过 SVD 文件实现寄存器可视化调试。  

相比传统的 Keil5 + MDK 工作流，该方案具有以下优势：  
- 跨平台支持（Windows / macOS / Linux）  
- 完全免费、可自由定制  
- 编辑体验与现代代码工具链结合更紧密  

至此，你已经能够在 VSCode 中完成从编写代码、编译，到下载调试的全流程，享受更高效、开放的嵌入式开发体验。



