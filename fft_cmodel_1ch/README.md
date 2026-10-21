# 单通道 FFT C Model 仿真

一个可独立运行的本地 Git 仓库：**int16 采样 → 定点加窗 → 真实 AMD FFT C Model → 整数功率谱 → 物理 PSD**。C++ 实现运算和模型调用；Python 生成测试信号、检查结果和绘图。

默认参数为单通道、8192 点、SSR=8、16 位数据、19 位旋转因子、1 GSps。SSR=8 表示每拍处理八个采样点，仍然只有一个通道。默认示例包含三帧双音加噪声数据，使用周期 Hann 窗。

新增 [三组实验与结论](experiments/README.md)：100 MHz 单音、10 MHz 方波、100 MHz 实数中频 LTE 风格 OFDM。每组预生成 100 帧 ADC 输入，经真实 AMD 模型运行，输出一张含 8 个中文子图的大 PNG，以及 M=1、10、50、100 的整数累加平均结果。

## 数学原型与团队对齐

[数学原理与 RTL 数值约定](docs/mathematical_principles.md) 给出乘窗、DFT/FFT 蝶形、定点缩放、功率和 PSD 的完整推导，并定义位宽、负数取整、饱和、SSR8 lane 顺序与逐位验收方法。后续 RTL 实现以其中的精确规则和本仓库 C Model 输出作为共同基准。

## 开始运行

仓库已包含 AMD FFT C Model 的头文件和全部配套动态库，**不需要安装 Vivado，也不需要另外提供模型 ZIP**。新机器仍需 Linux x86-64、Python 3.12+（含 venv/pip）、GNU Make 和支持 C++17 的 g++。首次 Python 依赖安装需要网络或本地包缓存。

中文图表需要系统中文字体，例如 Debian/Ubuntu 的 `fonts-noto-cjk`；也可设置 `CMODEL_CJK_FONT=/路径/中文字体文件`。字体与系统环境由使用者安装。

```bash
make all
```

`make all` 校验仓库内模型文件，安装 Python 依赖，完成 C++ 编译、原三帧示例、三组各 100 帧实验、绘图和自动测试。无需原四通道工程或旁边的 RTL 仓库；已初始化的目录可用 `make demo` 重跑原示例，或 `make experiments` 重跑新实验。

模型固定为 Vivado 2026.1 随附的 Linux x86-64 版本。包含的 3 个头文件、4 个共享库及来源、校验值见 [vendor/xfft](vendor/xfft/README.md)。普通开发环境和 NumPy、Matplotlib、pytest 等 Python 包不纳入 Git。

已在未安装 Vivado、未挂载宿主机目录的 Debian 12 / Python 3.12 容器内完成全新 Git 克隆验证：普通 Python 依赖由外部包缓存安装，`make all` 成功，23 项测试全部通过，默认示例及三组实验的输入、窗后数据、FFT、功率与各组累加平均结果均与原仓库逐字节一致。

## 结果入口

- [单音实验总览](experiments/results/tone/overview.png)
- [方波实验总览](experiments/results/square/overview.png)
- [实数中频 OFDM 实验总览](experiments/results/lte_ofdm/overview.png)
- [实验条件、结论与文件格式](experiments/README.md)
- [输入、窗和定点加窗结果](results/cmodel_pipeline.png)
- [功率谱与物理 PSD](results/cmodel_spectrum.png)
- [AMD 定点 FFT 与浮点 DFT 对比](results/cmodel_vs_numpy.png)
- [实测参数与校验值](results/reference_manifest.json)
- [验证摘要](docs/validation.md)
- [详细中文手册](docs/manual.md)

`src/fft_reference.cpp` 是完整的单通道 C++ 驱动和定点前后处理源码。FFT 内部实现由仓库中的 AMD 共享库提供。本项目源码、脚本、输入样例、实测结果以及模型头文件和运行库均已纳入 Git，也随 ZIP 和 Git bundle 一起交付。
