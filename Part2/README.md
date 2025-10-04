
## BabySoc

- **Processor (RVMYTH):** The brain, executes instructions.

- **PLL:** Generates stable, fast clock.

- **DAC:** Converts digital signals to analog.

- **GPIO:** Connects to external devices.

- **Memory:** Stores code and data.

- **Bus/Interconnect:** Connects everything together

## Project Structure

```txt
VSDBabySoC/
├── src/
│   ├── include/      # Header files (*.vh)
│   ├── module/       # Verilog + TLV modules
│   │   ├── vsdbabysoc.v   # Top-level module
│   │   ├── rvmyth.v       # CPU
│   │   ├── avsdpll.v      # PLL
│   │   ├── avsddac.v      # DAC
│   │   └── testbench.v    # Testbench
└── output/           # Simulation outputs
```

---

##  Setup

###  Cloning the Project

```bash
$ cd ~
$ git clone https://github.com/manili/VSDBabySoC.git
```


##  TLV → Verilog Conversion

Since RVMYTH is written in T
L-Verilog, we need to convert it to Verilog for simulating.

```bash
$ sudo apt update
$ sudo apt install python3-venv python3-pip
$ python3 -m venv sp_env
$ source sp_env/bin/activate
$ pip install pyyaml click sandpiper-saas
$ sandpiper-saas -i ./src/module/*.tlv -o rvmyth.v --bestsv --noline -p verilog --outdir ./src/module/
```

---

##  Simulation Flow



###  Pre-Synthesis Simulation

```bash
$ mkdir -p output/pre_synth_sim
$ iverilog -o output/pre_synth_sim/pre_synth_sim.out \
  -DPRE_SYNTH_SIM \
  -I src/include -I src/module \
  src/module/testbench.v
$ cd output/pre_synth_sim
./pre_synth_sim.out
```

### View in GTKWave:

```bash
$ gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```

###  The Instruction Program Driving BabySoC  

1. Increment counters,
2. Accumulate values into `r17`,
3. Oscillate them to generate analog waveforms,
4. Hold in a final loop.

| #  | Instruction         | Action                  |
| -- | ------------------- | ----------------------- |
| 0  | `ADDI r9, r0, 1`    | r9 = 1 (decrement step) |
| 1  | `ADDI r10, r0, 43`  | r10 = 43 (loop limit)   |
| 2  | `ADDI r11, r0, 0`   | r11 = 0 (counter)       |
| 3  | `ADDI r17, r0, 0`   | r17 = 0 (DAC input)     |
| 4  | `ADD r17, r17, r11` | Accumulate into r17     |
| 5  | `ADDI r11, r11, 1`  | Increment counter       |
| 6  | `BNE r11, r10, -4`  | Repeat until r11=43     |
| 7  | `ADD r17, r17, r11` | r17 += r11              |
| 8  | `SUB r17, r17, r11` | r17 -= r11              |
| 9  | `SUB r11, r11, r9`  | r11--                   |
| 10 | `BNE r11, r9, -4`   | Loop until r11=1        |
| 11 | `SUB r17, r17, r11` | Final adjust            |
| 12 | `BEQ r0, r0, ...`   | Infinite loop           |
       |

---

### Data Flow:
**Instruction Memory → CPU Pipeline → Register r17 → DAC → Analog OUT**

---






## Pre_synth_sim Waveform

![Waveform](Images/Task2_Ravi_pre_synth_simualtion_final.png)



## Resources

[BabySoc Manili](https://github.com/manili/VSDBabySoC)
