# I2C-Protocol-Verification-
# I2C Master UVM Testbench with APB Slave Interface

A comprehensive UVM-based verification environment for an I2C Master controller with APB (Advanced Peripheral Bus) slave access interface.

## 📋 Project Overview

This project implements a complete UVM testbench for verifying an I2C Master IP that interfaces with the system through an APB slave port. The testbench validates I2C protocol compliance according to the NXP I2C-bus specification (UM10204).

### Key Features

- **UVM 1.2 Compliant** - Full UVM methodology implementation
- **APB Interface** - Complete APB slave interface for register access
- **I2C Protocol Support** - Standard-mode I2C protocol verification
- **Multiple Test Scenarios** - Reset, Read/Write, Repeated Start, Interrupt handling
- **Scoreboard Integration** - Transaction-level verification with expected vs. actual comparison
- **Waveform Generation** - VCD dump for waveform analysis

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        UVM Testbench                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────────────────────────────┐ │
│  │   i2c_test  │───▶│           i2c_env                  │  │
│  └─────────────┘    │  ┌─────────────┐  ┌──────────────┐  │ │
│                     │  │  apb_agent  │  │  scoreboard  │  │ │
│                     │  │  ┌───────┐  │  └──────────────┘  │ │
│                     │  │  │driver │  │                    │ │
│                     │  │  │seqr   │  │                    │ │
│                     │  │  │monitor│  │                    │ │
│                     │  │  └───────┘  │                    │ │
│                     │  └─────────────┘                    │ │
│                     └─────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│                          DUT (I2C)                          │
│    ┌──────────────────────┐    ┌──────────────────────┐     │
│    │    APB Interface     │    │    I2C Interface     │     │
│    │  PCLK, PRESET        │    │  SCL_drive/result    │     │
│    │  PADDR, PSEL         │    │  SDA_drive/result    │     │
│    │  PENABLE, PWRITE     │    │  Interrupt           │     │
│    │  PWDATA, PRDATA      │    │                      │     │
│    │  PREADY, PSLVERR     │    │                      │     │
│    └──────────────────────┘    └──────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

## 📁 File Structure

```
├── i2c_uvm_tb.sv          # Main UVM testbench file
├── i2c4.sv                # I2C DUT (Design Under Test)
├── README.md              # This file
└── _group_07_i2c_dut_4.vcd # Generated waveform file
```

## 🔧 Register Map

| Address | Register | Description |
|---------|----------|-------------|
| 0x00    | I2C_IADR | I2C Address Register |
| 0x04    | I2C_IFDR | I2C Frequency Divider Register |
| 0x08    | I2C_I2CR | I2C Control Register |
| 0x0C    | I2C_I2SR | I2C Status Register |
| 0x10    | I2C_I2DR | I2C Data Register |

### Control Register (I2C_I2CR) Bits

| Bit | Name | Description |
|-----|------|-------------|
| 7   | IEN  | I2C Enable |
| 6   | IIEN | I2C Interrupt Enable |
| 5   | MSTA | Master/Slave Mode Select |
| 4   | MTX  | Transmit/Receive Mode Select |
| 3   | TXAK | Transmit Acknowledge Enable |
| 2   | RSTA | Repeated START |

## 🧪 Test Sequences

### 1. Reset Sequence (`i2c_rst_seq`)
Validates software reset functionality by:
- Programming registers with known values
- Asserting then deasserting the IEN bit
- Verifying register values are reset (except IADR and IFDR)

### 2. Read/Write Sequence (`apb_read_write_seq`)
FSM-based sequence that tests basic I2C transactions:
- ENABLE → START → SEND_DATA → STOP → DISABLE

### 3. Repeated Start Sequence (`apb_repeated_start_seq`)
Tests I2C repeated start condition for multi-byte transfers without releasing the bus.

### 4. Interrupt Handler Sequence (`i2c_interrupt_seq`)
Validates interrupt generation and handling during I2C transactions.

### 5. Boundary Address Sequence (`valid_slave_boundary_addr_seq`)
Tests boundary slave addresses (0x00 and 0x7F).

## 🚀 Running the Tests

### Prerequisites

- Simulator with UVM 1.2 support (VCS, Questa, Xcelium, etc.)
- SystemVerilog IEEE 1800-2012 support

### Compilation & Simulation

```bash
# Using VCS
vcs -sverilog -ntb_opts uvm-1.2 +define+READ_WRITE i2c_uvm_tb.sv -o simv
./simv

# Using Questa/ModelSim
vlog -sv +define+READ_WRITE i2c_uvm_tb.sv
vsim -c i2c_tb -do "run -all"

# Using Xcelium
xrun -uvm -define READ_WRITE i2c_uvm_tb.sv
```

### Test Selection

Use compile-time defines to select test sequences:

| Define | Test Sequence |
|--------|---------------|
| `+define+RESET` | Reset sequence |
| `+define+READ_WRITE` | Basic read/write sequence |
| `+define+REPEATED_START` | Repeated start sequence |
| `+define+INTERRUPT` | Interrupt handler sequence |
| `+define+VALID_SLAVE_ADDR` | Boundary address sequence |

## 📊 Waveform Analysis

The testbench generates VCD waveforms:

```bash
# View with GTKWave
gtkwave _group_07_i2c_dut_4.vcd
```

## 🔍 I2C Protocol Reference

This testbench verifies compliance with the I2C-bus specification (NXP UM10204):

- **Standard-mode**: Up to 100 kbit/s
- **7-bit addressing**: Slave address format
- **START/STOP conditions**: Bus control
- **Acknowledge (ACK/NACK)**: Data transfer handshaking
- **Repeated START**: Multi-message transfers

### I2C Timing Diagram

```
      START                                    STOP
        |                                        |
SDA ────┐     ┌───┐   ┌───┐       ┌───┐        ┌────
        └─────┘   └───┘   └───────┘   └────────┘
SCL ──────┐   ┌─┐   ┌─┐   ┌─┐   ┌─┐   ┌─┐   ┌───────
          └───┘ └───┘ └───┘ └───┘ └───┘ └───┘
           1     2     3     4     5     6     7   8  9
          |<-------- 7-bit Address ------->| R/W |ACK|
```
## 📚 References

1. NXP Semiconductors, "I2C-bus specification and user manual," UM10204 Rev. 6, April 2014
2. Accellera, "Universal Verification Methodology (UVM) 1.2 User's Guide"
3. ARM, "AMBA APB Protocol Specification"

## 📄 License

This project is developed for academic purposes as part of the EE273 course.

## 🔄 Version History

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2024 | Initial release with basic test sequences |
