# Synchronous FIFO in Verilog

A parameterizable synchronous FIFO buffer written in Verilog HDL. It uses a circular buffer with separate read and write pointers, and detects full and empty using one extra MSB (wrap bit) on each pointer, so no occupancy counter is needed.

Simulated with Icarus Verilog and inspected in GTKWave.

## Features

- Configurable depth (`FIFO_DEPTH`) and data width (`DATA_WIDTH`)
- Single clock domain, active-low asynchronous reset
- Chip select (`cs`) gating both read and write
- `full` and `empty` flags derived from the pointers
- Writes are ignored when full and reads are ignored when empty

## Parameters

| Parameter    | Default | Description                         |
|--------------|---------|-------------------------------------|
| `FIFO_DEPTH` | 8       | Number of entries in the buffer     |
| `DATA_WIDTH` | 32      | Width of each entry in bits         |

`FIFO_DEPTH` should be a power of two, because the pointers wrap naturally at 2^N.

## Ports

| Port       | Direction | Width        | Description                              |
|------------|-----------|--------------|------------------------------------------|
| `clk`      | input     | 1            | Clock, rising-edge triggered             |
| `rst_n`    | input     | 1            | Active-low asynchronous reset            |
| `cs`       | input     | 1            | Chip select, must be high to read/write  |
| `wr_en`    | input     | 1            | Write enable                             |
| `rd_en`    | input     | 1            | Read enable                              |
| `data_in`  | input     | `DATA_WIDTH` | Data to write                            |
| `data_out` | output    | `DATA_WIDTH` | Data read (registered)                   |
| `empty`    | output    | 1            | High when the FIFO has no data           |
| `full`     | output    | 1            | High when the FIFO cannot accept data    |

## How it works

Memory is an array of `FIFO_DEPTH` words. Each pointer is `log2(FIFO_DEPTH) + 1` bits wide. The lower bits index the memory, and the top bit toggles every time the pointer wraps around the end of the buffer.

- **Empty:** the two pointers are fully equal (same index, same wrap bit).
- **Full:** the index bits are equal but the wrap bits differ, meaning the write pointer has gone around once more than the read pointer.

```verilog
assign empty = (read_pointer == write_pointer);
assign full  = (read_pointer == {~write_pointer[FIFO_DEPTH_LOG],
                                  write_pointer[FIFO_DEPTH_LOG-1:0]});
```

Without the extra bit, "pointers equal" would mean both full and empty, and a counter would be needed to tell them apart.

## Repository structure

```
.
├── fifo_sync.v        # FIFO design
├── tb_fifo_sync.v     # Testbench
├── dump.vcd           # Waveform output from the testbench run
└── README.md
```

## Testbench

`tb_fifo_sync.v` is a directed testbench that drives the FIFO through these scenarios and prints the flags and data to the console:

1. **Basic write/read:** write two values (10, 55), then read both back.
2. **Fill and drain:** write `FIFO_DEPTH + 1` values (powers of two), so the last write is attempted while the FIFO is full, then read `FIFO_DEPTH` values back.

The testbench dumps all signals to `dump.vcd` for waveform inspection.

## Running the simulation

With Icarus Verilog and GTKWave installed:

```bash
iverilog -o sim fifo_sync.v tb_fifo_sync.v
vvp sim
gtkwave dump.vcd
```

The simulation can also be run on [EDA Playground](https://www.edaplayground.com/) by pasting the two files into the design and testbench panes and selecting Icarus Verilog.

## Waveform

<!-- Add a GTKWave screenshot here, e.g. ![waveform](waveform.png) -->

## Possible improvements

- Add automatic checking (a scoreboard or reference queue) so the testbench reports pass/fail on its own
- Reset `data_out`
- Add simultaneous read and write test cases
- Add an asynchronous variant with gray-coded pointers for clock-domain crossing

## Author

Piyush Antil
[LinkedIn](https://linkedin.com/in/antilpiyush) | [GitHub](https://github.com/antilpiyush89)
