计算机视觉实验三：C++ 与 OpenCV 基础图像处理 (Lab 03)

1. 实验目的与背景
本实验是计算机视觉课程的第三次作业。核心目标在于：
1.环境迁移：从 Python/NumPy 环境过渡到工业界更常用的 C++ 环境。
2. 工具链搭建：在 Linux (WSL Ubuntu) 系统下，从零搭建并配置基于 VS Code 的 C++ 自动化编译与调试环境（整合 `g++`, `gdb`, `tasks.json`, `launch.json`）。
3. OpenCV 基础操作：掌握 C++ 版本 OpenCV 的基本 API，完成图像的读取、信息提取、色彩空间转换、区域裁剪及文件保存。

 2. 开发与运行环境
操作系统: Windows Subsystem for Linux (WSL2) - Ubuntu 24.04.1 LTS
编译器: GCC / g++ (Ubuntu 13.3.0)
调试器 GDB (通过 `apt install gdb` 安装)
核心依赖库: OpenCV 4 (C++ 版本，通过 `libopencv-dev` 安装)
开发工具: Visual Studio Code (结合 C/C++ 扩展插件)

3. 核心任务与技术实现
本实验的所有逻辑均在 `main.cpp` 中实现，包含以下 6 个核心任务：

任务 1 & 2：图像读取与属性解析
使用 `cv::imread("test.jpg")` 读取同级目录下的海浪测试图。
通过 `img.cols`、`img.rows` 和 `img.channels()` 成功提取并打印了图像的宽度 (2043px)、高度 (1149px) 及通道数 (3通道)。

任务 4：色彩空间转换 (灰度化)
使用 `cv::cvtColor(img, gray_img, cv::COLOR_BGR2GRAY)` 将 BGR 彩色图像转换为单通道灰度图像。

任务 6：图像裁剪 (NumPy 切片操作的 C++ 替代方案)
核心思路：在 Python 中通常使用 NumPy 的数组切片 (`img[0:300, 0:300]`) 来裁剪图像。由于 C++ 没有 NumPy，本实验采用了 OpenCV 的 `cv::Rect(x, y, width, height)` 类来定义感兴趣区域 (ROI)。
代码实现：`Mat cropped_img = img(Rect(0, 0, 300, 300));` 成功截取了左上角 300x300 像素的区域。

任务 5 & 3：结果保存与可视化展示
使用 `cv::imwrite()` 将处理后的灰度图和裁剪图统一保存至新建的 `build/` 输出目录中，保持工程目录整洁。
使用 `cv::imshow()` 配合 `cv::waitKey(0)` 弹出窗口，直观对比原图与处理后的结果。
4. 踩坑记录与环境疑难解答 (Troubleshooting)
在配置 C++ OpenCV 环境时，遇到并解决了一些经典的系统级问题，特此记录：

1. GDB 调试器启动卡死 (Downloading separate debug info...)
问题原因：新版 Ubuntu 的 GDB 默认开启了 `debuginfod` 功能，在调试涉及图形底层的 OpenCV 程序时，会尝试连接海外服务器下载大量的系统库调试符号，导致网络阻塞假死。
解决方案：在 `.vscode/launch.json` 的 `setupCommands` 中注入指令 `"set debuginfod enabled off"`，成功关闭该功能，实现程序的秒级启动。
2. WSL调试模式下弹窗显示灰色/加载失败
问题原因：VS Code 的 GDB 调试器拦截了底层的渲染进程，导致 OpenCV 的 `imshow` 窗口虽然弹出，但像素数据无法及时刷入 WSL 的图形显示后台。
解决方案：
方案A (代码级补丁)：在 `waitKey(0)` 前加入 `pollKey()` 强制刷新 UI 事件队列。
	方案B (运行策略，最稳定)：避开调试器的干扰，使用 `Ctrl + F5` (Run Without Debugging) 或直接在终端运行编译好的可执行文件，即可完美显示彩色图像。

5. 项目目录结构
```text
lab03_cpp/
├── .vscode/               # VS Code 自动化配置文件 (放置在工作区顶层)
│   ├── launch.json        # 负责 F5 自动运行与调试的配置 (已修复 GDB 卡死问题)
│   └── tasks.json         # 负责 Ctrl+Shift+B 自动链接 OpenCV 编译的配置
├── build/                 # 编译输出目录
│   ├── main               # g++ 编译生成的 Linux 可执行程序
│   ├── gray_result.jpg    # 输出：任务5生成的灰度图
│   └── cropped_result.jpg # 输出：任务6生成的 300x300 裁剪图
├── main.cpp               # C++ 实验核心业务代码
├── test.jpg               # 输入：实验测试用原始海浪图
└── README.md              # 本实验说明文档
