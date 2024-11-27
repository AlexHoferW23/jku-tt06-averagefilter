## Moving Average Filter Project Documentation

### Table of Contents
1. [Introduction](#introduction)
2. [Features](#features)
3. [Module Overview](#module-overview)
    - [tt_um_moving_average_master](#tt_um_moving_average_master)
    - [tt_um_moving_average](#tt_um_moving_average)
4. [Detailed Description](#detailed-description)
    - [Filter Configuration](#filter-configuration)
    - [Finite State Machine (FSM) Operation](#finite-state-machine-fsm-operation)
5. [Port Definitions](#port-definitions)
6. [Usage Instructions](#usage-instructions)
---

### Introduction

The Moving Average Filter implements a configurable moving average filter using Verilog. This filter smooths out short-term fluctuations in a data stream, highlighting longer-term trends. The design is modular, allowing for different discrete filter window sizes based on user requirements.

### Features

- **Configurable Filter Window:** Supports window sizes of 2, 4, 8, and an extended 8+2 (cascaded) configuration.
- **Finite State Machine (FSM):** Ensures accurate data processing and state transitions.
- **Scalable Design:** Parameterizable for different data widths and filter configurations.

### Module Overview

#### `tt_um_moving_average_master`

This is the top-level module that manages input/output processing and selects the appropriate moving average filter based on control signals.

**Key Responsibilities:**
- Handling dedicated and bidirectional IO ports.
- Instantiating multiple moving average filter modules with different configurations.
- Selecting the active filter based on user input.
- Managing filter outputs and strobe signals.

#### `tt_um_moving_average`

This module implements the moving average filter logic. It processes incoming data, maintains a shift register for the windowed data, and computes the average.

**Key Responsibilities:**
- Storing incoming data in a shift register.
- Calculating the sum of data within the window.
- Computing the average based on the configured window size.
- Managing strobe signals to indicate valid output data.

### Detailed Description

#### Filter Configuration

The filter window size is determined by the `FILTER_POWER` parameter, where the window size is 2^`FILTER_POWER`. The master module instantiates multiple `tt_um_moving_average` modules with different `FILTER_POWER` values to support various window sizes:

- **Window Size 2:** `FILTER_POWER = 1`
- **Window Size 4:** `FILTER_POWER = 2`
- **Window Size 8:** `FILTER_POWER = 3`
- **Extended Window Size (8+2):** Combines an 8-length filter followed by a 2-length filter for enhanced smoothing.

#### Finite State Machine (FSM) Operation

Each `tt_um_moving_average` module operates using an FSM with three states:

1. **WAIT_FOR_STROBE:** The filter waits for a strobe signal indicating new data arrival.
2. **ADD:** Accumulates the sum of the current window's data.
3. **AVERAGE:** Computes the average by shifting the register and outputting the result.

The FSM ensures synchronized data processing aligned with incoming strobe signals, maintaining data integrity and timing.

### Port Definitions

#### `tt_um_moving_average_master`

| Port       | Direction | Width | Description                                                  |
|------------|-----------|-------|--------------------------------------------------------------|
| `ui_in`    | Input     | 8     | Dedicated input for the moving averager                      |
| `uo_out`   | Output    | 8     | Dedicated output from the moving averager                     |
| `uio_in`   | Input     | 8     | Bidirectional input path for control and additional data     |
| `uio_out`  | Output    | 8     | Bidirectional output path for control and additional data    |
| `uio_oe`   | Output    | 8     | Output enable signals for bidirectional IO (active high)      |
| `clk`      | Input     | 1     | Clock signal                                                 |
| `rst_n`    | Input     | 1     | Active low reset signal                                      |
| `ena`      | Input     | 1     | Enable signal                                                |

#### `tt_um_moving_average`

| Port          | Direction | Width        | Description                              |
|---------------|-----------|--------------|------------------------------------------|
| `data_in`     | Input     | `DATA_IN_LEN`| Input data for averaging                 |
| `data_out`    | Output    | `DATA_IN_LEN`| Smoothed output data                     |
| `strobe_in`   | Input     | 1            | Strobe signal indicating new data input  |
| `strobe_out`  | Output    | 1            | Strobe signal indicating valid output    |
| `clk`         | Input     | 1            | Clock signal                             |
| `reset`       | Input     | 1            | Active high reset signal                 |

### Usage Instructions

1. **Integration:**
   - Instantiate the `tt_um_moving_average_master` module in your top-level design.
   - Connect the dedicated and bidirectional IO ports as per your hardware configuration.

2. **Configuration:**
   - Set the `FILTER_POWER` and `DATA_IN_LEN` parameters as needed for your application.
   - Use the `uio_in[7:6]` pins to select the desired filter window size:
     - `00`: Window size 2
     - `01`: Window size 4
     - `10`: Window size 8
     - `11`: Extended window size (8+2)

3. **Operation:**
   - Provide input data through `ui_in` and `uio_out[3:2]`
   - Provide input strobe when data is valid through uio_out[0].
   - Ensure the `clk` and `rst_n` signals are properly connected.
   - Monitor the filtered output on `uo_out` and `uio_out[5:4]`.
   - Use the `uio_out[1]` for strobe signals indicating valid output data.


### License

This project is licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0). You may not use this file except in compliance with the License. A copy of the License is available at the above URL.

---
