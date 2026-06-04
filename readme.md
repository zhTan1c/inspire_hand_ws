
---

# Inspire Hand SDK Usage Guide

## Virtual Environment Management

It is recommended to use `venv` for managing the virtual environment:

```bash
python -m venv venv  # or  Unzip venv_x86.tar.xz, and place the.venv in inspire_hand_ws/.venv

# Then execute the script to modify venv:
python update_venv_path.py.venv
python update_bin_files.py.venv 

source venv/bin/activate  # Activate the virtual environment for Linux/MacOS

```

## Installation

1. When configuring the environment yourself, you need to install project dependencies; if you use Unzip venv_x86.tar.xz to set up the environment, you do not need to run the following command:

    ```bash
    pip install -r requirements.txt
    ```

2. Initialize and update submodules:

    ```bash
    git submodule init  # Initialize submodules
    git submodule update  # Update submodules to the latest version
    ```

3. Install the two SDKs:

    ```bash
    cd unitree_sdk2_python
    pip install -e .

    cd ../inspire_hand_sdk
    pip install -e .
    ```

## Control Modes

The Inspire Hand SDK supports multiple control modes, defined as follows:

- **Mode 0**: `0000` (No operation)
- **Mode 1**: `0001` (Angle)
- **Mode 2**: `0010` (Position)
- **Mode 3**: `0011` (Angle + Position)
- **Mode 4**: `0100` (Force control)
- **Mode 5**: `0101` (Angle + Force control)
- **Mode 6**: `0110` (Position + Force control)
- **Mode 7**: `0111` (Angle + Position + Force control)
- **Mode 8**: `1000` (Velocity)
- **Mode 9**: `1001` (Angle + Velocity)
- **Mode 10**: `1010` (Position + Velocity)
- **Mode 11**: `1011` (Angle + Position + Velocity)
- **Mode 12**: `1100` (Force control + Velocity)
- **Mode 13**: `1101` (Angle + Force control + Velocity)
- **Mode 14**: `1110` (Position + Force control + Velocity)
- **Mode 15**: `1111` (Angle + Position + Force control + Velocity)

## Usage Examples

Below are instructions for using common examples:

1. **DDS Control Command Publisher**:

    Run the following script to publish control commands:
    ```bash
    python inspire_hand_sdk/example/dds_publish.py
    ```

2. **DDS Subscriber for Inspire Hand Status and Tactile Sensor Data with Visualization**:

    Run the following script to subscribe to the Inspire Hand status and sensor data, and visualize the results:
    ```bash
    python inspire_hand_sdk/example/dds_subscribe.py
    ```

3. **Inspire Hand DDS Driver (Headless Mode)**:

    Use the following script for the headless mode driver:
    ```bash
    python inspire_hand_sdk/example/Headless_driver.py
    ```

4. **Inspire Hand Configuration Panel**:

    Run the following script to use the Inspire Hand configuration panel:
    ```bash
    python inspire_hand_sdk/example/init_set_inspire_hand.py
    ```

5. **Inspire Hand DDS Driver (Panel Mode)**:

    Use the following script to enter panel mode for the Inspire Hand DDS driver:
    ```bash
    python inspire_hand_sdk/example/Vision_driver.py
    ```

---

## Fork Modifications (this repo vs upstream)

This fork aligns the SDK's DDS type names with the ROS 2 `inspire_hand_msgs` package so that SDK nodes and ROS 2 nodes can communicate on the same DDS domain without bridging.

### 1. DDS typename alignment

| Message | Upstream typename | This fork typename |
|---------|------------------|--------------------|
| `inspire_hand_ctrl` | `"inspire.inspire_hand_ctrl"` | `"inspire_hand_msgs::msg::dds_::InspireHandCtrl_"` |
| `inspire_hand_state` | `"inspire.inspire_hand_state"` | `"inspire_hand_msgs::msg::dds_::InspireHandState_"` |
| `inspire_hand_touch` | `"inspire.inspire_hand_touch"` | `"inspire_hand_msgs::msg::dds_::InspireHandTouch_"` |

**Why**: The upstream SDK uses its own DDS type names (`inspire.xxx`). ROS 2's `inspire_hand_msgs` package generates different DDS type names (`inspire_hand_msgs::msg::dds_::Xxx_`). When both sides use `rmw_cyclonedds`, messages with mismatched type names are silently dropped. After alignment, SDK nodes (unitree_sdk2py based) and ROS 2 nodes can pub/sub each other's topics directly.

**Files changed**: `inspire_sdkpy/inspire_dds/_inspire_hand_ctrl.py`, `_inspire_hand_state.py`, `_inspire_hand_touch.py`

### 2. Lazy-load Qt dependencies

`inspire_sdkpy/__init__.py` now uses `importlib` + `__getattr__` to lazily import `qt_tabs` (which depends on PyQt5). This prevents import errors in headless environments (NX board, servers, CI) where Qt is not installed.

**File changed**: `inspire_sdkpy/__init__.py`
