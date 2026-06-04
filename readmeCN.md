
---

# 灵巧手SDK使用说明

## 环境管理

建议使用 `venv` 进行虚拟环境管理：

```bash
python -m venv venv # 或 解压venv_x86.tar.xz,将其中的.venv放置在inspire_hand_ws/.venv

# 之后执行脚本对venv进行修改：
python update_venv_path.py .venv
python update_bin_files.py .venv 

source venv/bin/activate  # Linux/MacOS 激活虚拟环境
```

## 安装依赖

1. 当自行配置环境时，需要安装项目依赖；如果你使用 Unzip venv_x86.tar.xz 去设置环境，则不需要运行以下命令：

    ```bash
    pip install -r requirements.txt
    ```

2. 更新子模块：

    ```bash
    git submodule init  # 初始化子模块
    git submodule update  # 更新子模块到最新版本
    ```

3. 分别安装两个SDK：

    ```bash
    cd unitree_sdk2_python
    pip install -e .

    cd ../inspire_hand_sdk
    pip install -e .
    ```

## 控制模式

Inspire 灵巧手 SDK 支持多种控制模式，定义如下：

- **Mode 0**: `0000` (无操作)
- **Mode 1**: `0001` (角度控制)
- **Mode 2**: `0010` (位置控制)
- **Mode 3**: `0011` (角度 + 位置)
- **Mode 4**: `0100` (力控)
- **Mode 5**: `0101` (角度 + 力控)
- **Mode 6**: `0110` (位置 + 力控)
- **Mode 7**: `0111` (角度 + 位置 + 力控)
- **Mode 8**: `1000` (速度控制)
- **Mode 9**: `1001` (角度 + 速度)
- **Mode 10**: `1010` (位置 + 速度)
- **Mode 11**: `1011` (角度 + 位置 + 速度)
- **Mode 12**: `1100` (力控 + 速度)
- **Mode 13**: `1101` (角度 + 力控 + 速度)
- **Mode 14**: `1110` (位置 + 力控 + 速度)
- **Mode 15**: `1111` (角度 + 位置 + 力控 + 速度)

## 使用示例

以下为几个常用示例的使用说明：

1. **DDS 发布控制指令**：

    运行以下脚本来发布控制指令：
    ```bash
    python inspire_hand_sdk/example/dds_publish.py
    ```

2. **DDS 订阅灵巧手状态和触觉传感器数据，并可视化**：

    运行以下脚本来订阅灵巧手的状态和传感器数据，并进行数据可视化：
    ```bash
    python inspire_hand_sdk/example/dds_subscribe.py
    ```

3. **灵巧手 DDS 驱动（无图模式）**：

    使用以下脚本进行无图模式的驱动操作：
    ```bash
    python inspire_hand_sdk/example/Headless_driver.py
    ```

4. **灵巧手配置面板**：

    运行以下脚本来使用灵巧手的配置面板：
    ```bash
    python inspire_hand_sdk/example/init_set_inspire_hand.py
    ```

5. **灵巧手 DDS 驱动（面板模式）**：

    通过以下脚本进入面板模式，控制灵巧手的 DDS 驱动：
    ```bash
    python inspire_hand_sdk/example/Vision_driver.py
    ```

---

## 本 Fork 相对于上游的改动

本 fork 对 SDK 做了少量修改，使 SDK 的 DDS 消息类型名与 ROS 2 的 `inspire_hand_msgs` 包一致，从而让 SDK 节点和 ROS 2 节点可以在同一个 DDS 域内直接通信。

### 1. DDS typename 对齐

| 消息 | 上游 typename | 本 fork typename |
|------|--------------|-----------------|
| `inspire_hand_ctrl` | `"inspire.inspire_hand_ctrl"` | `"inspire_hand_msgs::msg::dds_::InspireHandCtrl_"` |
| `inspire_hand_state` | `"inspire.inspire_hand_state"` | `"inspire_hand_msgs::msg::dds_::InspireHandState_"` |
| `inspire_hand_touch` | `"inspire.inspire_hand_touch"` | `"inspire_hand_msgs::msg::dds_::InspireHandTouch_"` |

**为什么改**：上游 SDK 使用自己的 DDS 类型名（`inspire.xxx`），而 ROS 2 的 `inspire_hand_msgs` 包编译后生成的 DDS 类型名是 `inspire_hand_msgs::msg::dds_::Xxx_`。当双方都使用 `rmw_cyclonedds` 时，类型名不匹配的消息会被静默丢弃。改了 typename 后，SDK 节点（基于 unitree_sdk2py）和 ROS 2 节点可以直接互相发布/订阅。

**改动文件**：`inspire_sdkpy/inspire_dds/_inspire_hand_ctrl.py`、`_inspire_hand_state.py`、`_inspire_hand_touch.py`

### 2. Qt 依赖惰性加载

`inspire_sdkpy/__init__.py` 改用 `importlib` + `__getattr__` 惰性导入 `qt_tabs`（依赖 PyQt5）。这样在无头环境（NX 板、服务器、CI）中不会因为缺少 Qt 而导入失败。

**改动文件**：`inspire_sdkpy/__init__.py`
