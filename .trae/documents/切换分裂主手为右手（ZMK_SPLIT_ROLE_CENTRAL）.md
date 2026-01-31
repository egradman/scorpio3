## 目标
- 将分裂主手由左切换为右（右半为 Central，左半为 Peripheral）。
- 将外设轨迹板（Azoteq IQS5xx）从左半启用迁移到右半启用，仅右半生效。

## 具体变更
- Central 归属切换：
  - 修改 [Kconfig.defconfig:L6-L14](file:///Users/chenzeng/project/corne_tracpad/boards/arm/eyelash_corne/Kconfig.defconfig#L6-L14) 的条件块：把 `if BOARD_EYELASH_CORNE_LEFT` 改为 `if BOARD_EYELASH_CORNE_RIGHT`，保留块内 `ZMK_KEYBOARD_NAME` 与 `ZMK_SPLIT_ROLE_CENTRAL`，实现右半默认为 Central。
  - 共同启用项（`ZMK_SPLIT` 等）位于左右共享条件块 [Kconfig.defconfig:L16-L36](file:///Users/chenzeng/project/corne_tracpad/boards/arm/eyelash_corne/Kconfig.defconfig#L16-L36)，无需改动。
- 轨迹板迁移到右半：
  - 公共 DTS 中 IQS5xx 节点默认为禁用：[eyelash_corne.dtsi:L260-L289](file:///Users/chenzeng/project/corne_tracpad/boards/arm/eyelash_corne/eyelash_corne.dtsi#L260-L289)。
  - 当前左右半均显式启用 I2C1 与 IQS5xx：
    - 左半启用点：[eyelash_corne_left.dts:L13-L19](file:///Users/chenzeng/project/corne_tracpad/boards/arm/eyelash_corne/eyelash_corne_left.dts#L13-L19)
    - 右半启用点：[eyelash_corne_right.dts:L14-L20](file:///Users/chenzeng/project/corne_tracpad/boards/arm/eyelash_corne/eyelash_corne_right.dts#L14-L20)
  - 变更方案：
    - 在左半 DTS 移除或改回禁用 `&i2c1` 与 `&tps43`（不再设为 "okay"），让其保留公共 dtsi 的默认 "disabled"。
    - 保持右半 DTS 对 `&i2c1` 与 `&tps43` 的启用为 "okay"，轨迹板仅在右半工作。

## 验证步骤
- 构建：分别构建右半（Central）与左半（Peripheral）固件。
- 配置核对：
  - 右半 `.config` 应包含 `CONFIG_ZMK_SPLIT_ROLE_CENTRAL=y`；左半不应包含。
  - 设备树核对：检查构建产物中的 `zephyr.dts` 或 `build/zephyr/zephyr.dts`：
    - 右半应显示 `&i2c1` 与 `iqs5xx@74` 节点 `status = "okay"`。
    - 左半应为 `status = "disabled"`（未被覆盖启用）。
- 上电测试：右半通电后作为 BLE Central，连接左半；在主机侧验证触控/滚动仅来自右半轨迹板。

## 影响范围
- 仅调整 Central 归属与轨迹板启用侧；不改动矩阵、编码器、背光、供电与 I2C 引脚复用等其余配置。
- 如存在自动打包/刷写流程，需确认右半 UF2 刷写到右板、左半 UF2 刷写到左板。

## 后续操作
- 我将按以上方案修改 Kconfig.defconfig 条件块，并更新左半 DTS 以禁用轨迹板、保留右半启用；随后协助你完成两侧构建与验证。