# 上下文速览（Context Quickview）

本项目为 Sputnik44 键盘开发基于 ZMK 的自定义模块 `zmk-device-detector`，实现通过 ADC 电压检测自动识别并管理多种可热插拔外设（如摇杆、轨迹球、旋钮），并集成到左右手固件中。

**当前进展：**
- 已完成基础固件编译、模块集成、Kconfig/CMake/设备树配置打通。
- `zmk-device-detector` 已支持摇杆检测，左右手固件均可正常编译。
- 轨迹球(PMW3610)驱动已完成并成功编译固件，待硬件测试。
- 旋钮(EC11)驱动已完成并成功编译固件，待硬件测试。

**主要技术点：**
- ZMK 模块开发与集成（Kconfig、CMake、module.yml、DTS overlay）
- Zephyr ADC 驱动与电压判别
- 外设 handler 动态初始化/反初始化（热插拔支持）
- Docker 容器内一键编译与本地挂载

**后续关键任务：**
- 进行硬件测试，重点验证摇杆、轨迹球和旋钮功能
- 根据测试反馈优化热插拔检测和设备切换机制

---

# 工作记录板 (Scratchpad)

## 1. 当前用户需求 (Current User Request)
构建一个带有本地module `zmk-device-detector` 的键盘 `sputnik44` 固件（包括左手和右手）。

分目标：
1. 完成 `sputnik-zmk-config` 的配置，并可以正常编译固件。
2. 完成 `zmk-device-detector` driver 的开发。功能包括：从adc通道读电压信息来识别不同的设备（轨迹球/旋钮/摇杆），然后根据不同的设备进行设备的初始化等操作。支持热插拔。用户已有部分驱动代码，会在开发时提供。
3. 把 `zmk-device-detector` 模块配置到 `zmk-config` 中，并成功编译出固件。

## 2. 背景和动机 (Background and Motivation)
用户希望为其 `sputnik44` 键盘构建 ZMK 固件。此固件需要包含一个自定义的本地模块 `zmk-device-detector`，该模块用于通过 ADC 读取电压来自动检测和初始化可热插拔的外围设备（如轨迹球、旋钮、摇杆）。最终目标是为左手和右手键盘生成可用的固件。

## 3. 关键挑战和分析 (Key Challenges and Analysis)
*   **子目标1 (`sputnik-zmk-config` 配置与编译):**
    *   确保 `sputnik-zmk-config` 指向正确的键盘定义文件。
    *   解决编译过程中可能出现的依赖问题或配置错误。
    *   验证基础固件（不含自定义模块）的编译通过性。
*   **子目标2 (`zmk-device-detector` 开发):**
    *   首先需要学习和理解 ZMK 模块开发框架。
    *   ADC 读取与电压分析逻辑的准确性。
    *   各种设备（轨迹球、旋钮、摇杆）的驱动代码整合到 ZMK 模块中。
    *   实现可靠的热插拔检测和重新初始化机制，并符合 ZMK/Zephyr 的事件处理方式。
    *   与 ZMK 固件的事件系统和电源管理正确集成。
*   **子目标3 (模块集成与最终编译):**
    *   正确地将 `zmk-device-detector` 模块添加到 ZMK 的构建系统中，并确保 `sputnik-zmk-config` 正确引用。
    *   处理模块集成可能引入的新的编译或运行时问题。
    *   确保左右手固件均能包含此模块并成功编译。
    *   确保最终固件在实际硬件上经过用户充分验证，能够稳定运行。

## 4. 高层任务拆分 (High-level Task Breakdown)

### 第一阶段：基础固件编译与 `sputnik-zmk-config` 配置

- [X] **任务 1.1:** 确认 `sputnik-zmk-config` 结构和关键文件。
- [X] **任务 1.2:** (规划者辅助) 检查 `sputnik-zmk-config/config/` 目录中是否存在 `sputnik44_left.conf`, `sputnik44_right.conf`, `sputnik44.keymap` (或类似命名) 并初步评估其内容。
- [X] **任务 1.3:** 尝试编译基础固件 (不含 `zmk-device-detector`) - 左手。
- [X] **任务 1.4:** 编译右手基础固件。

### 第二阶段：`zmk-device-detector` 驱动开发
- [X] **任务 2.0:** (执行者学习) 学习 ZMK 模块开发框架。
- [X] **任务 2.1:** (用户提供) 获取用户现有的各外设驱动代码参考。
- [X] **任务 2.2:** 分析现有驱动代码，并基于 ZMK 模块框架规划 `zmk-device-detector` 整体整合方案。

- **任务 2.3: 实现 `zmk-device-detector` 核心逻辑。**
    - [C] **任务 2.3.1:** 在 `device_detector.c` 中实现核心 ADC 读取、电压分析和设备类型判断逻辑。 (初步完成，待硬件测试和DTS完善)
    - [P] **任务 2.3.2:** 在 `device_detector.c` 中实现设备热插拔检测机制 (当前为轮询，待持续优化和测试)。
    - [X] **任务 2.3.3:** 定义各设备处理程序的函数接口 (例如 `_handler_init`, `_handler_deinit`)。

- **任务 2.4: 摇杆 (Joystick) 驱动集成与功能实现。**
    - [X] **任务 2.4.1:** 获取/确认用户提供的摇杆驱动代码 (`zmk-analog-input-driver`)。
    - [X] **任务 2.4.2:** 将摇杆驱动核心代码 (`analog_input.c` 和相关头文件) 集成到 `zmk-device-detector` 模块的 `src/analog_input_subdriver` 和 `include/zmk/drivers`。
    - [ ] **任务 2.4.3:** (待定) 若原始提供的代码为单独副本，集成稳定后可考虑清理。
    - [C] **任务 2.4.4:** 实现 `joystick_handler.c`，使其能调用摇杆子驱动的初始化/反初始化 (通过 `sensor_attr_set`)。 (初步完成)
    - [X] **任务 2.4.5:** 在模块 `Kconfig` 中定义摇杆相关选项 (`ZMK_DEVICE_DETECTOR_HAS_JOYSTICK`, `ANALOG_INPUT`) 并在 `sputnik44.conf` 中启用。
    - [X] **任务 2.4.6:** 在 `sputnik44.overlay` 中配置摇杆设备节点 (`joystick_device` compatible "zmk,analog-input") 及 `zmk_device_detector_node` 对其的引用，并根据实际硬件更新 ADC 通道等参数。
    - [X] **任务 2.4.7:** 成功编译包含摇杆支持的左右手固件。
    - [ ] **任务 2.4.8:** 用户进行摇杆功能硬件测试和调试。

- **任务 2.5: 轨迹球 (Trackball) 驱动集成与功能实现。**
    - [X] **任务 2.5.1:** 获取/确认用户提供的轨迹球驱动代码 (`zmk-pmw3610-driver` 参考)。
    - [X] **任务 2.5.2:** 将轨迹球驱动核心代码 (SPI 通信、传感器数据处理等) 集成到 `zmk-device-detector` 模块的 `src/trackball_handler.c` (或新的子驱动目录)。
    - [X] **任务 2.5.3:** (待定) 若原始提供的代码为单独副本，集成稳定后可考虑清理。
    - [X] **任务 2.5.4:** 实现 `trackball_handler.c` 的初始化/反初始化逻辑。
    - [X] **任务 2.5.5:** 在模块 `Kconfig` 中定义轨迹球相关选项 (`ZMK_DEVICE_DETECTOR_HAS_TRACKBALL` 等) 并在 `sputnik44.conf` 中启用。
    - [X] **任务 2.5.6:** 在 `sputnik44.overlay` 中配置轨迹球设备节点 (例如 compatible "zmk,pmw3610") 及 `zmk_device_detector_node` 对其的引用，配置 SPI 总线、引脚等。
    - [X] **任务 2.5.7:** 成功编译包含轨迹球支持的左右手固件。
    - [ ] **任务 2.5.8:** 用户进行轨迹球功能硬件测试和调试。

- **任务 2.6: 旋钮 (Encoder) 驱动集成与功能实现。**
    - [X] **任务 2.6.1:** 确认旋钮类型 (通常为 EC11，ZMK 内核已支持)。
    - [X] **任务 2.6.2:** 对于 EC11，主要工作不是"拷贝集成代码"，而是确保设备树中有标准定义，并由 `encoder_handler.c` 控制其激活状态。
    - [X] **任务 2.6.3:** (不适用)
    - [X] **任务 2.6.4:** 实现 `encoder_handler.c`，使其能激活/停用设备树中预定义的 EC11 实例。
    - [X] **任务 2.6.5:** 在模块 `Kconfig` 中定义旋钮相关选项 (`ZMK_DEVICE_DETECTOR_HAS_ENCODER`) 并在 `sputnik44.conf` 中启用 (同时确保 `CONFIG_EC11=y` 等基础旋钮支持已开启)。
    - [X] **任务 2.6.6:** 在 `sputnik44.overlay` 中配置标准 EC11 设备节点 (`compatible = "alps,ec11"`)，并确保 `zmk_device_detector_node` 能通过 `controlled-encoder` phandle 指向它。
    - [X] **任务 2.6.7:** 成功编译包含旋钮支持的左右手固件。
    - [ ] **任务 2.6.8:** 用户进行旋钮功能硬件测试和调试。

- **任务 2.7:** (可选) 为 `zmk-device-detector` 编写单元测试或集成测试。 (原任务 2.6)

### 第三阶段：模块集成与最终固件编译
- [X] **任务 3.1:** 将 `zmk-device-detector` 模块配置到 `sputnik-zmk-config` 中 (通过 `Kconfig` 在 `sputnik44.conf` 中启用 `CONFIG_ZMK_DEVICE_DETECTOR=y`，并通过 `sputnik44.overlay` 定义 `zmk_device_detector_node` 节点)。
- [X] **任务 3.2:** 编译包含 `zmk-device-detector` 的左手固件。
- [X] **任务 3.3:** 编译包含 `zmk-device-detector` 的右手固件。
- [ ] **任务 3.4:** 用户在设备上进行初步验证 (验证已集成的第一个外设，例如摇杆)。
- [ ] **任务 3.5:** 根据用户反馈进行固件迭代与最终确认。

## 5. 项目最新进展 (Progress Update)
- 2024-06-09：
    - 已成功将 `zmk-device-detector` 模块集成到 ZMK 固件，并分别为左右手编译通过，生成了可刷写的 .uf2 固件文件。
    - 目前已实现摇杆（joystick）检测与切换的基础功能，核心 ADC 读取与设备类型判断逻辑已可用。
    - 轨迹球（trackball）和旋钮（encoder）相关的 handler 框架已搭建，后续将逐步完善其驱动集成与功能实现。
    - Kconfig、设备树 overlay、CMake 配置等已全部打通，编译流程顺畅。
    - 下一步将进行硬件测试，重点验证摇杆功能，并逐步推进轨迹球和旋钮的集成。

- 2024-06-11:
    - 成功完成PMW3610轨迹球驱动的集成和编译，左右手固件都已通过编译测试
    - 解决了ADC设备引用和设备树配置的问题，正确配置了SPI总线和PMW3610节点
    - 简化了device_detector.c的实现，使用硬编码方式处理ADC设备访问
    - 优化了trackball_handler.c的实现，移除了对电源管理API的依赖
    - 下一步将开始旋钮(encoder)支持的开发，并等待轨迹球的硬件测试结果

- 2024-06-12:
    - 成功完成EC11旋钮驱动的集成和编译，左右手固件编译通过
    - 实现了encoder_handler.c处理逻辑，与设备探测器正确集成
    - 修复了Kconfig中对choice symbols的不正确使用问题
    - 更新了sputnik44.overlay和sputnik44.conf配置文件，启用旋钮支持
    - 项目现在已支持三种外设（摇杆、轨迹球、旋钮）的自动检测和切换

## 6. 执行者反馈或请求帮助 (Executor Feedback or Help Requests)
- 编译命令和 Docker 路径映射已全部理顺，左右手固件均可一键编译。
- 目前主要待办为：
    1. 进行硬件测试，收集实际运行数据。
    2. 持续完善 trackball/encoder handler 及其驱动集成。
    3. 根据测试反馈优化热插拔检测和设备切换机制。

## 7. 备注与讨论点 (Notes and Discussion Points)
*   编译命令中的 `-DZMK_CONFIG` 路径根据 `build-rules.mdc` 确定：
    *   基础编译 (任务 1.3, 1.4): `"/workspaces/zmk-config/config"` (无末尾斜杠)
    *   带额外模块编译 (任务 3.2, 3.3): `"/workspaces/zmk-config/config/"` (有末尾斜杠)
*   编译命令中的 `-DZMK_EXTRA_MODULES` 路径根据 `build-rules.mdc` 确定为 `"/workspaces/zmk-modules/zmk-device-detector"` (指向具体的模块) 而不是 `"/workspaces/zmk-modules/"` (指向父目录)。
*   请确认 Docker volume 映射与这些路径一致：
    *   本地 `sputnik-zmk-config` 子模块 -> 容器内 `/workspaces/zmk-config`
    *   本地 `modules` 目录 -> 容器内 `/workspaces/zmk-modules`
*   `build-rules.mdc` 中右手基础编译命令示例的 `DSHIELD` 仍为 `sputnik44_left`，执行时会修正为 `sputnik44_right`。
*   `build-rules.mdc` 中带额外模块的右手编译命令示例的 `DSHIELD` 仍为 `sputnik44_left`，执行时会修正为 `sputnik44_right`。
*   编译命令中的 `-DZMK_CONFIG="/workspaces/zmk-config/config/"` 路径假定在 Docker 容器内，您的 `sputnik-zmk-config` 子模块被映射到了 `/workspaces/zmk-config`。
*   编译命令中的 `-DZMK_EXTRA_MODULES="/workspaces/zmk-modules/"` 路径假定在 Docker 容器内，您的 `modules` 目录 (包含 `zmk-device-detector`) 被映射到了 `/workspaces/zmk-modules`。
*   请确认 Docker volume 映射与这些路径一致。 