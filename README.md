# cvwsoc README (WIP)

Core-V Wally SoC (extended Core-V Wally)

Extended Core-V Wally core/SoC (https://github.com/openhwgroup/cvw) with many additional Open Source IPs targeting some popular FPGA boards in order to run Yocto Linux images. 
Simulation (Verilator) and co-simulation (Renode) supported.

## Features
- 32 / 64 bits CPUs, different configs
- Yocto-based Linux images, u-boot, OpenSBI.
- Open source IPs:
    - Core-V Wally: CPU, GPIO, UART, PLIC, SPI
    - SDHCI (micro SD card boot)
    - DMA (MEM2MEM, MEM2DEV)
    - USB 1.1
    - Ethernet
    - VGA
    - DDR2 (LiteDRAM)
    - DDR3 (LiteDRAM, UberDDR3)
    - AXI
    - AXI Stream (Audio TX)
    - I2S
    - etc (more to come).
- Simulation (Verilator)
    - Full SoC
    - Boot: bootrom, OpenSBI, u-boot and Linux support
    - Core AXI bus infrastructure
    - Peripherals (SDHCI, DMA, I2S, etc). More to come
- Co-simulation (Renode)
    - HiFive FU540 SoC (Renode) with (verilated) CVWSoC AXI component, booting custom CVWSoC Yocto image
    - MMIO working but it's slow. Useful for fast, non-cycle accurate iterations

## Memory map


| Region | Base address | End address | Size | Type | Access | Bus | Interconnect | IRQ | Status | Notes |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| Boot ROM | `0x0000_1000` | `0x0000_1FFF` | 4 KiB | Memory | R/X | AHB-Lite | AHB-Lite | — | — | |
| SPI-mode SD-card controller | `0x0001_3000` | `0x0001_3FFF` | 4 KiB | MMIO | R/W, 32/64-bit | APB | AHB-Lite → APB | PLIC 9 | — | |
| CLINT | `0x0200_0000` | `0x0200_FFFF` | 64 KiB | MMIO | R/W | APB | AHB-Lite → APB | Direct: MSI 3, MTI 7 | — | `MSIP + 0x0000`, `MTIMECMP + 0x4000`, `MTIME + 0xBFF8`. |
| PLIC | `0x0C00_0000` | `0x0FFF_FFFF` | 64 MiB | MMIO | R/W, 32-bit | APB | AHB-Lite → APB | Direct: SEI 9, MEI 11 | — | |
| UART 16550 | `0x1000_0000` | `0x1000_0007` | 8 B | MMIO | R/W, 8-bit | APB | AHB-Lite → APB | PLIC 10 | — | |
| SPI controller | `0x1004_0000` | `0x1004_0FFF` | 4 KiB | MMIO | R/W, 32-bit | APB | AHB-Lite → APB | PLIC 6 | — | |
| GPIO | `0x1006_0000` | `0x1006_00FF` | 256 B | MMIO | R/W, 32-bit | APB | AHB-Lite → APB | PLIC 3 | — | |
| iDMA AXI/AXI-Stream frontend | `0x1008_2000` | `0x1008_2FFF` | 4 KiB | MMIO | R/W, 8/16/32/64-bit | AXI4 | AHB-Lite → AXI4 → AXI-Stream | PLIC 13 | — | |
| AXI SDHCI | `0x1009_0000` | `0x1009_0FFF` | 4 KiB | MMIO | R/W, 8/16/32-bit | AXI4 | AHB-Lite → AXI4 | PLIC 17 | — | |
| AXI VGA controller | `0x100B_0000` | `0x100B_0FFF` | 4 KiB | MMIO | R/W, 8/16/32-bit | AXI4 | AHB-Lite → AXI4 | — | — | AXI4 master for FB scanout. |
| AXI USB OHCI controller | `0x100C_0000` | `0x100C_0FFF` | 4 KiB | MMIO | R/W, 8/16/32-bit | AXI4-Lite | AHB-Lite → AXI4 → AXI4-Lite | PLIC 14 | — | AXI4 master interface for DMA. |
| AXI Ethernet | `0x100D_0000` | `0x100E_FFFF` | 128 KiB | MMIO / buffer | R/W, 8/16/32-bit | AXI4 | AHB-Lite → AXI4 | PLIC 15 | — | |
| LiteDRAM CSR window | `0x100F_0000` | `0x100F_1FFF` | 8 KiB | MMIO | R/W, 8/16/32-bit | AXI4-Lite | AHB-Lite → AXI4 → AXI4-Lite | — | — | |
| Wishbone island | `0x1100_0000` | `0x110F_FFFF` | 1 MiB | MMIO region | R/W, 8/16/32-bit | Wishbone | AHB-Lite → Wishbone | See subregions | — | Contains the Wishbone devices below. |
| Wishbone Ethernet | `0x1100_0000` | `0x1100_3FFF` | 16 KiB | MMIO / buffer | R/W, 8/16/32-bit | Wishbone | AHB-Lite → Wishbone | PLIC 12 | — | CSR space begins at offset `0x2000`. |
| Wishbone UART 16550 | `0x1100_4000` | `0x1100_4FFF` | 4 KiB | MMIO | R/W, 8/16/32-bit | Wishbone | AHB-Lite → Wishbone | PLIC 11 | — | |
| Wishbone stub | `0x1100_5000` | `0x1100_5FFF` | 4 KiB | MMIO | R/W, 8/16/32-bit | Wishbone | AHB-Lite → Wishbone | — | — | |
| AXI dummy/test peripheral | `0x2000_0000` | `0x2003_FFFF` | 256 KiB | MMIO / test | R/W, 8/16/32-bit | AXI4 | AHB-Lite → AXI4 | PLIC 16 | — | |
| Uncore RAM | `0x8000_0000` | `0x87FF_FFFF` | 128 MiB | Memory | R/W/X | AHB-Lite | AHB-Lite | — | — | |
| External memory / DDR | `0x8000_0000` | `0x87FF_FFFF` | 128 MiB | Memory | R/W/X | AXI4 | AHB-Lite → AXI4 | — | — | |
| Uncached-memory PMA overlay | `0x8400_0000` | `0x840F_FFFF` | 1 MiB | Attribute | Inherits backing memory | Backing-memory bus | Same as backing memory | — | — | Marks the overlapping memory range as uncached. |
| iDMA descriptor-64 frontend | `0x1008_0000` | `0x1008_0FFF` | 4 KiB | MMIO | R/W, 8/16/32/64-bit | AXI4 | AHB-Lite → AXI4 | PLIC 13 | **Disabled** | |
| iDMA register-64 frontend | `0x1008_1000` | `0x1008_1FFF` | 4 KiB | MMIO | R/W, 8/16/32-bit | AXI4 | AHB-Lite → AXI4 | PLIC 13 | **Disabled** | |
| DTIM | `0x8000_0000` | `0x807F_FFFF` | 8 MiB | Memory | R/W | Core-local | LSU local interface | — | **Disabled** | Bypasses the system buses. |
| IROM | `0x8000_0000` | `0x807F_FFFF` | 8 MiB | Memory | R/X | Core-local | IFU local interface | — | **Disabled** | Bypasses the system buses. |
| Xilinx AXI CDMA | `0x100A_0000` | `0x100A_FFFF` | 64 KiB | MMIO | R/W, 8/16/32-bit | AXI4-Lite | AHB-Lite → AXI4 → AXI4-Lite | PLIC 13 | **Disabled** | AXI4 master interface for DMA. |





## Gateware (Vivado)

Repo: https://github.com/juanschroeder/cvw/tree/cvwsoc

Build:

```
cd fpga/generator
make TARGET
```

Where TARGET can be:
- genesys2soc:                   Normal 64-bit Genesys 2 target
- genesys2socrv32:               RV32 variant
- genesys2socrv32min:            Smaller RV32 variant
- genesys2socrv32w64:            RV32 variant with 64-bit bus
- nexysa7soc:                    Nexys A7 build
- nexysa7rv32w64soc:             RV32 variant with 64-bit bus
- etc

Remarks:
* Bitstream (.bit) will be generated in a subfolder. You can flash it with Vivado or with openFPGAloader.
* It's better to do a 'make cleanIP' (or cleanAll) between builds
* For the moment debugging can be done with Vivado ILA. JTAG debugging and Manta tracing will come at some point.


### Gateware (openXC7, only for Genesys 2 for now)

```
cd fpga/generatorxc7
make docker-bit
```


## Software

Software images are Yocto based and can be generated using this Kas based project: https://github.com/juanschroeder/kas-cvwsoc. See README for more details.


## boot log [deprecated]

Asciinema of boot process over serial:

[![asciicast](https://asciinema.org/a/xH8lwlHBaXZ3RadW.svg)](https://asciinema.org/a/xH8lwlHBaXZ3RadW)


# Doom Demos in FPGA boards


## Doom on the Genesys 2 (demo, no sound)

[![Watch the video](https://img.youtube.com/vi/BBkvet0Lh4Y/default.jpg)](https://youtu.be/BBkvet0Lh4Y)

### Configuration

It was tested in the fastest known configuration, with open source IPs:

- 17-19 fps at 320x240
- CPU: RV32GC with 64-bits bus (XLEN=32, AHBW=64) @ 55 MHz
- DDR: UberDDR3 controller
- Storage: SD card on SDHCI
- USB: OHCI 1.1
- Ethernet: LiteEthernet
- etc

Console:

    # fbdoom -iwad /usr/share/games/doom/freedm.wad -nosound -timedemo demo4
    Starting D_DoomMain
                                   FDoom 0.1
    Z_Init: Init zone memory allocation daemon. 
    zone memory: 0x95047010, 600000 allocated for zone
    Using /mnt/.fdoom.tar/ for configuration and saves
    V_Init: allocate screens.
    M_LoadDefaults: Load system defaults.
    saving config in /mnt/.fdoom.tar/default.cfg
    W_Init: Init WADfiles.
     adding /usr/share/games/doom/freedm.wad
     adding demo4.lmp
     couldn't open demo4.lmp
    Playing demo demo4.lmp.
    Using /mnt/.fdoom.tar/savegame/ for savegames
    ===========================================================================
                                    FreeDM
    ===========================================================================
     FDoom is free software, covered by the GNU General Public
     License.  There is NO warranty; not even for MERCHANTABILITY or FITNESS
     FOR A PARTICULAR PURPOSE. You are welcome to change and distribute
     copies under certain conditions. See the source for more information.
    ===========================================================================
    I_Init: Setting up machine state.
    M_Init: Init miscellaneous info.
    R_Init: Init DOOM refresh daemon - ...............................................
    P_Init: Init Playloop state.
    S_Init: Setting up sound.
    D_CheckNetGame: Checking network game status.
    startskill 2  deathmatch: 0  startmap: 1  startepisode: 1
    player 1 of 1 (1 nodes)
    Emulating the behavior of the 'Doom 1.9' executable.
    HU_Init: Setting up heads up display.
    ST_Init: Init status bar.
    I_InitGraphics: framebuffer: x_res: 320, y_res: 240, x_virtual: 320, y_virtual: 240, bpp: 16, grayscale: 0
    I_InitGraphics: framebuffer: RGBA: 5650, red_off: 11, green_off: 5, blue_off: 0, transp_off: 0
    I_InitGraphics: DOOM screen size: w x h: 320 x 200
    I_InitGraphics: Auto-scaling factor: 1
    101-key keyboard found.
    Using keyboard on /dev/tty0.
    Ready to read keycodes. Press Backspace to exit.
    timed 3623 gametics in 6945 realtics (18.258459 fps)


**UPDATE**: latest version performance without audio and UberDDR3: about **19.8 fps**.


## Doom on the Genesys 2 (with I2S audio)


    # fbdoom -iwad /usr/share/games/doom/freedm.wad -timedemo demo4
 
Extra hardware used for audio: 

- PCM5102A DAC board (https://www.amazon.de/-/en/DollaTek-PCM5102A-Digital-Converter-Raspberry/dp/B07PMGGMJF).
- No SCL neeeded (derived)
- XMT => 3.3V (unmute)
- Headphones sound loud enough at full volume


Remarks:
- Tested only in RV32W64 config
- Performance:
  - With **UberDDR3**: about **16.5 fps**
  - With **LiteDRAM**: about **14.5 fps**
- To be able to play with the keyboard the easiest is to start 'fbdoom' in the framebuffer console (with the USB keyboard) and not in the serial console


## Doom on the Genesys 2 (live, with USB audio)

Another sound configuration that can be used is with an USB audio card.

Setup tested:
- USB audio card USB 1.1 compatible (Tested: 'C-Media Electronics Inc. USB Audio Device')
- USB keyboard (e.g. standard Logitech wireless USB keyboard)


Command:
```
SDL_AUDIODRIVER=dsp SDL_PATH_DSP=/dev/dsp SDL_AUDIO_BUFFER_SIZE=4096 fbdoom -iwad /usr/share/games/doom/freedm.wad -nomusic -timedemo demo4
```

Remarks:
- Result: about 12 fps
- Using default audio device (e.g.AUDIODEV=hw:0,0) gives worse performance


## Doom on the Nexys A7 (live with audio)

Live playing with audio (no demo):

[![Watch the video](https://img.youtube.com/vi/aWhnACIJtG4/default.jpg)](https://youtu.be/aWhnACIJtG4)


```
# fbdoom -iwad /usr/share/games/doom/freedm.wad
```

Console: 

```
# fbdoom -iwad /usr/share/games/doom/freedm.wad -nomusic -timedemo demo4
Starting D_DoomMain
                               FDoom 0.1
Z_Init: Init zone memory allocation daemon. 
zone memory: 0x94f2f010, 600000 allocated for zone
Using /mnt/.fdoom.tar/ for configuration and saves
V_Init: allocate screens.
M_LoadDefaults: Load system defaults.
saving config in /mnt/.fdoom.tar/default.cfg
W_Init: Init WADfiles.
 adding /usr/share/games/doom/freedm.wad
 adding demo4.lmp
 couldn't open demo4.lmp
Playing demo demo4.lmp.
Using /mnt/.fdoom.tar/savegame/ for savegames
===========================================================================
                                FreeDM
===========================================================================
 FDoom is free software, covered by the GNU General Public
 License.  There is NO warranty; not even for MERCHANTABILITY or FITNESS
 FOR A PARTICULAR PURPOSE. You are welcome to change and distribute
 copies under certain conditions. See the source for more information.
===========================================================================
I_Init: Setting up machine state.
[  365.168501] iis-idma-engine 10082000.dma-controller: cyclic: buf=0x82980000 buf_len=147456 period_len=12288 periods=12 hw_descs=12
[  365.181881] idma issue: pending=1 current=c24d055c running=1 cyclic_active=1 stopping=0
M_Init: Init miscellaneous info.
ALSA lib /usr/src/debug/alsa-lib/1.2.14/src/pcm/pcm.c:8772:(snd_pcm_recover) underrun occurred
[  366.290326] iis-idma-engine 10082000.dma-controller: cyclic: buf=0x82980000 buf_len=147456 period_len=12288 periods=12 hw_descs=12
[  366.303686] idma issue: pending=1 current=c24d055c running=1 cyclic_active=1 stopping=0

R_Init: Init DOOM refresh daemon - ...............................................
P_Init: Init Playloop state.
S_Init: Setting up sound.
D_CheckNetGame: Checking network game status.
startskill 2  deathmatch: 0  startmap: 1  startepisode: 1
player 1 of 1 (1 nodes)
Emulating the behavior of the 'Doom 1.9' executable.
HU_Init: Setting up heads up display.
ST_Init: Init status bar.
I_InitGraphics: framebuffer: x_res: 320, y_res: 240, x_virtual: 320, y_virtual: 240, bpp: 16, grayscale: 0
I_InitGraphics: framebuffer: RGBA: 5650, red_off: 11, green_off: 5, blue_off: 0, transp_off: 0
I_InitGraphics: DOOM screen size: w x h: 320 x 200
I_InitGraphics: Auto-scaling factor: 1
101-key keyboard found.
Using keyboard on /dev/tty0.
Ready to read keycodes. Press Backspace to exit.
```

When running 'demo4' demo (-timedemo demo4):
- With I2S audio: `timed 3623 gametics in 24058 realtics (5.270804 fps)`
- Without audio: `timed 3623 gametics in 15822 realtics (8.014474 fps)`



## Boards supported

Boards supported and/or planned :
- Supported:
  - Digilent Nexys A7-100T (does not fit in an A35T)
  - Digilent Genesys 2
- Coming soon:
  - Qmtech Kintex-7
 
Extra hardware:
- Optional: USB PMOD
  - https://github.com/Dolu1990/pmod_usb_host_x4/tree/main
  - https://github.com/nand2mario/usb_host_pmod
  - This store made it avaiable for purchase in 2025 when I requested it (no stock now?):
      - [https://www.aliexpress.com/store/5940159](https://www.aliexpress.com/store/5940159)
      - Alternative: [https://nl.aliexpress.com/item/1005010786670759.html](https://www.aliexpress.com/item/1005010786670759.html)
- Optional: PCM5102A DAC or similar. E.g. this one was tested [DollaTek-PCM5102A](https://www.amazon.de/-/en/DollaTek-PCM5102A-Digital-Converter-Raspberry/dp/B07PMGGMJF)
- Optional: USB keyboard (tried Logitech wireless Keyboard)
- Optional: USB audio card. Tested: https://www.amazon.de/-/en/SABRENT-External-Headphone-Adapter-AU-MMSA/dp/B00IRVQ0F8/
- Optional: Digilent micro SD card PMOD. Not needed anymore but will be needed for the Qmtech Kintex-7.
- Remark: Qmtech Kintex-7 board has no peripherals and will need more PMODs.

## Features

Features:
- u-boot support
- on-board Ethernet support in u-boot. Network boot not tested but
- framebuffer in Linux at 320x240
- fbdoom and some demos in the image
- USB 1.1 host
- Ethernet
  - On-board RMII Ethernet working
  - Some USB Ethernet devices supported (tested: TP-Link UE300 10/100/1000 LAN (ethernet mode) [Realtek RTL8153])
- on-board VGA (4 bits)
- SDHCI SD card storage

Remarks: 
* Apart from some unnecessary IPs, all used components are open source (Nexys A7 build still needs to be updated).
* USB does not work yet in u-boot in some targets


# Co-simulation (Renode)

The AXI bus infrastructure can be co-simulated with a Renode SoC (HiFive FU540). Useful for quick iterations.

Renode HIFive FU540 <--[AXI]--> CVWSOC_AXI

Remarks:
- simulation speed is limited: AXI clock is set to 2 MHz. Faster is possible but simulation is slower.
- much faster than Verilation and including peripherals. **Shell prompt in 2-3 minutes**.
- Tested:
    - iDMA: dmatest and cyclic dma audio (for speaker-test proper audio clock must be uncommented)
    - SDHCI (test image simulated by default)
    - Ethernet: just Kernel initialization
    - I2S: speaker-test possible when audio clock is enabled/connected (slower simulation), even with 2 MHz AXI bus.
    - USB: initialization works, proper simulation might be possible (and slow) later.
    - VGA: not tested, needs proper initialization (according to bus speed).



# Simulation (QEMU, Verilator)

It allows running different combinations of simulation/emulation, boot stages and boot media.

## Summary
- Simulate base SoC + real bus infrastructure + some AXI peripherals
- XLEN:{32, 64}, AHBW:{32, 64}. Core-V Wally configs
- AXI peripherals
    - AXI_RAM. UberDDR3 simulation would be possible but too slow
    - SDHCI
    - DMA
    - DUMMY peripheral mapped to 0x20000000-0x2003FFFF with IRQ 16 can be connected to the AXI bus without any further Hw changes.
    - More to come: USB, etc.
- Console interaction with simulation possible through pseudo terminal (PTS)
- Boot sequence: Bootrom (optional), OpenSBI, u-boot (optional, Linux
- Boot media: preloaded RAM, simulated SD card. All binaries generated in Yocto.
- Many options configuratble in variables/parameters




## General simulation remarks

- Main Makefile: sim/Makefile.cvwsoc. Much of it is LLM generated so not the most readable.
- All boot stages are possible to run: bootrom, OpenSBI, u-boot, Kernel, userspace.
- Only RAM and a potential 'Dummy' peripheral are added to the Verilator testbench for now. Any extra peripheral to be tested can be connected as the 'dummy' peripheral (this needs DTB override and potentially other changes in the binaries/images).
- Simulation is slow. Skip stages when possible.
- Yocto images: for simulation, images in corresponding Yocto 'deploy' folder are used.
    - Depending on the target you'll need different images.
    - The images must be in the expected deploy folder.
    - You should override CVWSOC_DEPLOY_ROOT for this purpose. $(CVWSOC_DEPLOY_ROOT)/$(CVWSOC_MACHINE) is the 'deploy' folder for the corresponding machine
- RV32 (32-bits) can be selected by setting RV32=1. By default RV32=0
- The 'linux' images skip u-boot by adding a 'stup' that jumps directly to Linux.
- Runs without BOOTROM=1 (default) add a stub at reset address (0x1000) that jumps to OpenSBI. Disabled by default. Bootrom run not possible in QEMU but possible in Verilator.
    - Remark: There is an issue with u-boot and when starting from bootrom. To be fixed.
- Boot from SD card (SDHCI=1). When enabled it would use the 'wic' image generated in Yocto. Without SD card preloaded ramdisk is used.
- Not all combinations have been tested and not all possible combinations work.
- Tracing Verilator simulation is possible. Traced signals must be at the 'top' level of the tetstench
- Serial interaction with the verilated run is possible. There's a /dev/pts/NN device created at runtime for this purpose. There is a one character delay issue that still has to be fixed


Different combinations are possible using Yocto images all of them work for 32 and 64 bits CPUs using variables passed to the Makefile when needed:
- Boot 'virt/virt32' image in QEMU 'virt' RV32/RV64 targets
- Boot running in Verilator testbench.
- Preloaded images:
    - Generate 'preloaded' image to use later: speed up simulation by preloading all RAM contents.
    - This is the default for Verilator runs.
    - Boot preloaded image in QEMU
- Trace verilation run: TRACE=0 (no tracing), TRACE=1 (runtime tracing start/stop trigger with kill -USR1), TRACE=sv (trace from the beginning of simulation until simulation is stopped). .fst files are generated.
- Some things can make sense to override for testing
    - override DTB in Verilator run (overwrites content in preloaded image)
    - Override u-booot DTB in Verilator run
    - override DEPLOY folder
    - bus width (AHBW=64)
    - Trace filename prefix
    - etc
- etc


## Simulation examples

Test Yocto OpenSBI and Kernel generated image in QEMU:
```
$ make -f sim/verilator/Makefile.cvwsoc clean qemu-linux RV32=0
```

The same but including u-boot:
```
$ make -f sim/verilator/Makefile.cvwsoc clean qemu-uboot
```

Run Linux verilation for RV64 with SD card emulation, starting from bootrom and overriding OpenSBI/Linux DTB on 'Verilator' runtime stage and generate a trace since boot. 'fpgagenesys2soc' config is used:
```
$ make -f sim/verilator/Makefile.cvwsoc clean run-cvwsoc-linux RV32=0 SDHCI=1 CVWSOC_VERILATOR_DTB=/tmp/wally-virtsoc-linux.dtb.dts.dtb  BOOTROM=1 CONFIG=fpgagenesys2soc TRACE=sv
```

Run Linux verilation for RV32W64 (fastest) with SD card emulation (using image generated in Yocto), with DMA peripheral connected, skipping bootrom, jumping from OpenSBI to Linux (skipping u-boot), doing parallel build for the testbench. 'fpgagenesys2rv32w64soc' config is used:
```
$ make -f sim/verilator/Makefile.cvwsoc clean sim-fast SDHCI=1 DMA=1 TRACE=1
```

Remarks:
- 'clean' is not needed when testebench rebuild is not necessary. The Makefile should normally track all necessary dependencies (and rebuild when needed)
- Models for other peripherals will be integrated later.

# Future steps

Future plans:
- Lots of cleanup needed
- OpenXC7 build improvements (currently at 12.5 MHz)
- Genesys 2:
    - HDMI support
    - mini display support
- Boards:
    - Qmtech Kintex-7 support
    - GateMate board?
    - Other smaller FPGA boards
- Remove remaining remaining Xilinx dependencies
- JTAG debug interface?
- Renode co-simulation
- Other IPs: watchdog, gigabit Ethernet, USB 3.0
- Other PMODs:
    - display
    - HDMI?
- Refactor top files for better customisation
- CPU frequency speedup
- etc

# Example boot log (Nexys A7 RV32)

```

 █▀█        █▀█        █▀█        █▀▀                 █ █
 █          █ █        █▄▀        █▄▄       ▄▄▄       █ █
 █▄█        █▄█        █ █        █▄▄                 ▀▄▀
 ____          ____  ____      ___      ___   ____    ___
 \   \        /   / /    \    |   |    |   |  \   \  /  /
  \   \  __  /   / /      \   |   |    |   |   \   \/  /
   \   \/  \/   / /   /\   \  |   |    |   |    \     /
    \          / /   ____   \ |   |___ |   |___  |   |
     \___/\___/ /___/    \___\|_______||_______| |___|


          INITIALIZING DDR....
Initializing SDRAM @0x80000000...
Switching SDRAM to software control.
Read leveling:
  m0, b00: |00000000000000000000000000000000| delays: -
  m0, b01: |00000000000000000000000000000000| delays: -
  m0, b02: |11111111111111111111100000000000| delays: 10+-10
  m0, b03: |00000000000000000000000011111111| delays: 27+-03
  m0, b04: |00000000000000000000000000000000| delays: -
  m0, b05: |00000000000000000000000000000000| delays: -
  m0, b06: |00000000000000000000000000000000| delays: -
  m0, b07: |00000000000000000000000000000000| delays: -
  best: m0, b02 delays: 10+-10
  m1, b00: |00000000000000000000000000000000| delays: -
  m1, b01: |00000000000000000000000000000000| delays: -
  m1, b02: |11111111111111111111110000000000| delays: 10+-10
  m1, b03: |00000000000000000000000011111111| delays: 27+-03
  m1, b04: |00000000000000000000000000000000| delays: -
  m1, b05: |00000000000000000000000000000000| delays: -
  m1, b06: |00000000000000000000000000000000| delays: -
  m1, b07: |00000000000000000000000011111111| delays: 27+-03
  best: m1, b02 delays: 10+-10
Switching SDRAM to hardware control.

          DDR INIT DONE!
[363 ms] SDHC: SDHC 1., 50 MHz base clock
[602 ms] Getting GPT information.
          Blocks loaded: 1/1
[612 ms] Getting partition entries.
          Blocks loaded: 1/1
[621 ms] Loading device tree at: 0x87000000
          Blocks loaded: 0/12sdhc_wait_intr error: 1
          Blocks loaded: 12/12
[687 ms] Loading OpenSBI at: 0x80000000
          Blocks loaded: 0/526sdhc_wait_intr error: 1
          Blocks loaded: 512/526sdhc_wait_intr error: 1
          Blocks loaded: 526/526
[2462 ms] Loading Linux Kernel at: 0x80200000
          Blocks loaded: 0/950sdhc_wait_intr error: 1
          Blocks loaded: 512/950sdhc_wait_intr error: 1
          Blocks loaded: 950/950
[5626 ms] Done! Flashing LEDs and jumping to OpenSBI...

OpenSBI v1.7
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name               : wally-virt,qemu
Platform Features           : medeleg
Platform HART Count         : 1
Platform IPI Device         : aclint-mswi
Platform Timer Device       : aclint-mtimer @ 25000000Hz
Platform Console Device     : uart8250
Platform HSM Device         : ---
Platform PMU Device         : ---
Platform Reboot Device      : ---
Platform Shutdown Device    : ---
Platform Suspend Device     : ---
Platform CPPC Device        : ---
Firmware Base               : 0x80000000
Firmware Size               : 313 KB
Firmware RW Offset          : 0x40000
Firmware RW Size            : 57 KB
Firmware Heap Offset        : 0x45000
Firmware Heap Size          : 37 KB (total), 2 KB (reserved), 9 KB (used), 25 KB (free)
Firmware Scratch Size       : 4096 B (total), 1276 B (used), 2820 B (free)
Runtime SBI Version         : 3.0
Standard SBI Extensions     : time,rfnc,ipi,base,hsm,pmu,dbcn,fwft,legacy,sse
Experimental SBI Extensions : none

Domain0 Name                : root
Domain0 Boot HART           : 0
Domain0 HARTs               : 0*
Domain0 Region00            : 0x10000000-0x10000fff M: (I,R,W) S/U: (R,W)
Domain0 Region01            : 0x02000000-0x0200ffff M: (I,R,W) S/U: ()
Domain0 Region02            : 0x0c200000-0x0c20ffff M: (I,R,W) S/U: (R,W)
Domain0 Region03            : 0x80040000-0x8004ffff M: (R,W) S/U: ()
Domain0 Region04            : 0x80000000-0x8003ffff M: (R,X) S/U: ()
Domain0 Region05            : 0x0c000000-0x0c1fffff M: (I,R,W) S/U: (R,W)
Domain0 Region06            : 0x00000000-0xffffffff M: () S/U: (R,W,X)
Domain0 Next Address        : 0x80200000
Domain0 Next Arg1           : 0x82200000
Domain0 Next Mode           : S-mode
Domain0 SysReset            : yes
Domain0 SysSuspend          : yes

Boot HART ID                : 0
Boot HART Domain            : root
Boot HART Priv Version      : v1.12
Boot HART Base ISA          : rv32imafdcb
Boot HART ISA Extensions    : sstc,zicntr,zihpm,zicboz,zicbom
Boot HART PMP Count         : 16
Boot HART PMP Granularity   : 6 bits
Boot HART PMP Address Bits  : 32
Boot HART MHPM Info         : 20 (0xfff6d230)
Boot HART Debug Triggers    : 0 triggers
Boot HART MIDELEG           : 0x00000222
Boot HART MEDELEG           : 0x0000b108


U-Boot 2026.01 (Jul 22 2026 - 13:52:08 +0000)

CPU:   riscv
DRAM:  128 MiB
board_init: Initializing VGA at 0x100b0000 with framebuffer at 0x85040000
Core:  20 devices, 14 uclasses, devicetree: embed
MMC:   mmc@10090000: 0
Loading Environment from nowhere... OK
In:    uart@10000000
Out:   uart@10000000
Err:   uart@10000000
Net:   
Warning: ethernet@11003000 (eth0) using random MAC address - 3e:01:07:0a:b2:80
eth0: ethernet@11003000
Hit any key to stop autoboot: 0
CMD_SEND:0
                ARG                      0x00000000
                MMC_RSP_NONE
CMD_SEND:8
                ARG                      0x000001aa
                MMC_RSP_R1,5,6,7         0x000001aa
CMD_SEND:55
                ARG                      0x00000000
                MMC_RSP_R1,5,6,7         0x00000120
CMD_SEND:41
                ARG                      0x40300000
                MMC_RSP_R3,4             0x40ff8000
CMD_SEND:55
                ARG                      0x00000000
                MMC_RSP_R1,5,6,7         0x00000120
CMD_SEND:41
                ARG                      0x40300000
                MMC_RSP_R3,4             0xc0ff8000
CMD_SEND:2
                ARG                      0x00000000
                MMC_RSP_R2               0x03534453
                                         0x44333247
                                         0x855824a9
                                         0x98019500

                                        DUMPING DATA
                                        000 - 03 53 44 53 
                                        004 - 44 33 32 47 
                                        008 - 85 58 24 a9 
                                        012 - 98 01 95 00 
CMD_SEND:3
                ARG                      0x00000000
                MMC_RSP_R1,5,6,7         0xaaaa0520
CMD_SEND:9
                ARG                      0xaaaa0000
                MMC_RSP_R2               0x400e0032
                                         0x5b590000
                                         0xedc87f80
                                         0x0a404000

                                        DUMPING DATA
                                        000 - 40 0e 00 32 
                                        004 - 5b 59 00 00 
                                        008 - ed c8 7f 80 
                                        012 - 0a 40 40 00 
CMD_SEND:7
                ARG                      0xaaaa0000
                MMC_RSP_R1,5,6,7         0x00000700
CMD_SEND:55
                ARG                      0xaaaa0000
                MMC_RSP_R1,5,6,7         0x00000920
CMD_SEND:51
                ARG                      0x00000000
                MMC_RSP_R1,5,6,7         0x00000920
CMD_SEND:6
                ARG                      0x00fffff1
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:55
                ARG                      0xaaaa0000
                MMC_RSP_R1,5,6,7         0x00000920
CMD_SEND:6
                ARG                      0x00000002
                MMC_RSP_R1,5,6,7         0x00000920
CMD_SEND:6
                ARG                      0x80fffff0
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:55
                ARG                      0xaaaa0000
                MMC_RSP_R1,5,6,7         0x00000920
CMD_SEND:13
                ARG                      0x00000000
                MMC_RSP_R1,5,6,7         0x00000920
MMC read: dev # 0, block # 2, count 4 ... CMD_SEND:16
                ARG                      0x00000200
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:18
                ARG                      0x00000002
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:12
                ARG                      0x00000000
                MMC_RSP_R1b              0x00000b00
4 blocks read: OK
MMC read: dev # 0, block # 34, count 12 ... CMD_SEND:16
                ARG                      0x00000200
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:18
                ARG                      0x00000022
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:12
                ARG                      0x00000000
                MMC_RSP_R1b              0x00000b00
12 blocks read: OK
MMC read: dev # 0, block # 1528, count 9884 ... CMD_SEND:16
                ARG                      0x00000200
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:18
                ARG                      0x000005f8
                MMC_RSP_R1,5,6,7         0x00000900
CMD_SEND:12
                ARG                      0x00000000
                MMC_RSP_R1b              0x00000b00
9884 blocks read: OK
   Uncompressing Kernel Image to 0
## Flattened Device Tree blob at 87000000
   Booting using the fdt blob at 0x87000000
Working FDT set to 87000000
   Loading Device Tree to 86757000, end 8675b53a ... OK
Working FDT set to 86757000

Starting kernel ...

[    0.000000] Linux version 6.12.93 (oe-user@oe-host) (riscv32-poky-linux-gcc (GCC) 15.2.0, GNU ld (GNU Binutils) 2.45.0.20250908) #1 PREEMPT Tue Jun  9 10:26:06 UTC 2026
[    0.000000] OF: fdt: Ignoring memory range 0x80000000 - 0x80400000
[    0.000000] Machine model: wally-virt,qemu
[    0.000000] SBI specification v3.0 detected
[    0.000000] SBI implementation ID=0x1 Version=0x10007
[    0.000000] SBI TIME extension detected
[    0.000000] SBI IPI extension detected
[    0.000000] SBI RFENCE extension detected
[    0.000000] SBI DBCN extension detected
[    0.000000] earlycon: uart8250 at MMIO 0x10000000 (options '')
[    0.000000] printk: legacy bootconsole [uart8250] enabled
[    0.000000] efi: UEFI not found.
[    0.000000] OF: reserved mem: 0x84000000..0x8403ffff (256 KiB) nomap non-reusable uncached-memory@84000000
[    0.000000] OF: reserved mem: 0x84040000..0x84043fff (16 KiB) nomap non-reusable idma-desc@84040000
[    0.000000] OF: reserved mem: 0x84044000..0x840fffff (752 KiB) nomap non-reusable uncached_rest@84044000
[    0.000000] OF: reserved mem: 0x85040000..0x850fffff (768 KiB) nomap non-reusable framebuffer@85040000
[    0.000000] OF: reserved mem: 0x80000000..0x8007ffff (512 KiB) nomap non-reusable opensbi@80000000
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000080400000-0x00000000875fffff]
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000080400000-0x0000000083ffffff]
[    0.000000]   node   0: [mem 0x0000000084000000-0x00000000840fffff]
[    0.000000]   node   0: [mem 0x0000000084100000-0x000000008503ffff]
[    0.000000]   node   0: [mem 0x0000000085040000-0x00000000850fffff]
[    0.000000]   node   0: [mem 0x0000000085100000-0x00000000875fffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000080400000-0x00000000875fffff]
[    0.000000] riscv: base ISA extensions acdfim
[    0.000000] riscv: ELF capabilities acdfim
[    0.000000] Kernel command line: root=/dev/mmcblk0p5 rw earlycon=uart8250,mmio,0x10000000 console=ttyS0,115200 loglevel=7 init=/sbin/init rootwait
[    0.000000] Dentry cache hash table entries: 16384 (order: 4, 65536 bytes, linear)
[    0.000000] Inode-cache hash table entries: 8192 (order: 3, 32768 bytes, linear)
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 29184
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Virtual kernel memory layout:
[    0.000000]       fixmap : 0x9d800000 - 0x9e000000   (8192 kB)
[    0.000000]       pci io : 0x9e000000 - 0x9f000000   (  16 MB)
[    0.000000]      vmemmap : 0x9f000000 - 0xa0000000   (  16 MB)
[    0.000000]      vmalloc : 0xa0000000 - 0xc0000000   ( 512 MB)
[    0.000000]       lowmem : 0xc0000000 - 0xc7200000   ( 114 MB)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] rcu: Preemptible hierarchical RCU implementation.
[    0.000000] rcu:     RCU event tracing is enabled.
[    0.000000] rcu:     RCU debug extended QS entry/exit.
[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
[    0.000000] NR_IRQS: 64, nr_irqs: 64, preallocated irqs: 0
[    0.000000] riscv-intc: 32 local interrupts mapped
[    0.000000] rcu: srcu_init: Setting srcu_struct sizes based on contention.
[    0.000000] clocksource: riscv_clocksource: mask: 0xffffffffffffffff max_cycles: 0x5c40939b5, max_idle_ns: 440795202646 ns
[    0.000053] sched_clock: 64 bits at 25MHz, resolution 40ns, wraps every 4398046511100ns
[    0.014856] Console: colour dummy device 80x25
[    0.022266] Calibrating delay loop (skipped), value calculated using timer frequency.. 50.00 BogoMIPS (lpj=100000)
[    0.035017] pid_max: default: 32768 minimum: 301
[    0.046485] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.056098] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.155998] ASID allocator using 9 bits (512 entries)
[    0.170321] rcu: Hierarchical SRCU implementation.
[    0.177415] rcu:     Max phase no-delay instances is 1000.
[    0.200696] EFI services will not be available.
[    0.208638] Memory: 87884K/116736K available (5293K kernel code, 8770K rwdata, 4096K rodata, 4159K init, 232K bss, 28448K reserved, 0K cma-reserved)
[    0.239601] devtmpfs: initialized
[    0.338908] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.352168] futex hash table entries: 256 (order: 1, 7168 bytes, linear)
[    0.372102] DMI not present or invalid.
[    0.417571] NET: Registered PF_NETLINK/PF_ROUTE protocol family
[    0.434624] DMA: preallocated 128 KiB GFP_KERNEL pool for atomic allocations
[    0.692392] cpu0: Ratio of byte access time to unaligned word access is 0.00, unaligned accesses are slow
[    0.960089] SCSI subsystem initialized
[    0.979723] usbcore: registered new interface driver usbfs
[    0.990265] usbcore: registered new interface driver hub
[    0.999378] usbcore: registered new device driver usb
[    1.011116] pps_core: LinuxPPS API ver. 1 registered
[    1.017885] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    1.030238] PTP clock support registered
[    1.043406] Advanced Linux Sound Architecture Driver Initialized.
[    1.087152] vgaarb: loaded
[    1.095985] clocksource: Switched to clocksource riscv_clocksource
[    1.745805] NET: Registered PF_INET protocol family
[    1.762647] IP idents hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    1.810813] tcp_listen_portaddr_hash hash table entries: 256 (order: 1, 5120 bytes, linear)
[    1.826701] Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    1.837488] TCP established hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    1.849429] TCP bind hash table entries: 1024 (order: 4, 40960 bytes, linear)
[    1.863784] TCP: Hash tables configured (established 1024 bind 1024)
[    1.875556] UDP hash table entries: 256 (order: 2, 12288 bytes, linear)
[    1.886967] UDP-Lite hash table entries: 256 (order: 2, 12288 bytes, linear)
[    1.906280] NET: Registered PF_UNIX/PF_LOCAL protocol family
[    1.917238] PCI: CLS 0 bytes, default 64
[    1.974333] workingset: timestamp_bits=30 max_order=15 bucket_order=0
[    2.006401] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 251)
[    2.017711] io scheduler mq-deadline registered
[    2.024457] io scheduler kyber registered
[    2.031286] io scheduler bfq registered
[    2.426786] riscv-plic: plic@c000000: mapped 53 interrupts with 1 handlers for 2 contexts.
[    2.492904] simple-framebuffer 85040000.framebuffer: framebuffer at 0x85040000, 0x25800 bytes
[    2.503764] simple-framebuffer 85040000.framebuffer: format=r5g6b5, mode=320x240x16, linelength=640
[    2.564709] Console: switching to colour frame buffer device 40x30
[    2.601064] simple-framebuffer 85040000.framebuffer: fb0: simplefb registered!
[    6.403337] Serial: 8250/16550 driver, 1 ports, IRQ sharing disabled
[    6.470580] printk: legacy console [ttyS0] disabled
[    6.511452] 10000000.uart: ttyS0 at MMIO 0x10000000 (irq = 2, base_baud = 1562500) is a 16550A
[    6.526676] printk: legacy console [ttyS0] enabled
[    6.526676] printk: legacy console [ttyS0] enabled
[    6.539480] printk: legacy bootconsole [uart8250] disabled
[    6.539480] printk: legacy bootconsole [uart8250] disabled
[    6.583038] of_serial 11004000.uart: probe with driver of_serial failed with error -28
[    6.638835] sifive_spi 13000.spi: mapped; irq=4, cs=4
[    6.730150] liteeth 11003000.ethernet eth0: irq 6 slots: tx 2 rx 2 size 2048
[    6.747480] usbcore: registered new device driver r8152-cfgselector
[    6.759566] usbcore: registered new interface driver r8152
[    6.770987] usbcore: registered new interface driver asix
[    6.781941] usbcore: registered new interface driver ax88179_178a
[    6.793481] usbcore: registered new interface driver cdc_ether
[    6.804827] usbcore: registered new interface driver net1080
[    6.817048] usbcore: registered new interface driver cdc_subset
[    6.827480] usbcore: registered new interface driver zaurus
[    6.841519] usbcore: registered new interface driver cdc_ncm
[    6.851742] usbcore: registered new interface driver r8153_ecm
[    6.887786] usbcore: registered new interface driver usb-storage
[    6.893941] ohci-platform 100c0000.usb: Generic Platform OHCI controller
[    6.927090] sdhci: Secure Digital Host Controller Interface driver
[    6.936965] ohci-platform 100c0000.usb: new USB bus registered, assigned bus number 1
[    6.946894] sdhci: Copyright(c) Pierre Ossman
[    6.955712] ohci-platform 100c0000.usb: irq 7, io mem 0x100c0000
[    6.958904] sdhci-pltfm: SDHCI platform and OF driver helper
[    7.052840] mmc0: SDHCI controller on 10090000.mmc [10090000.mmc] using PIO
[    7.107374] usbcore: registered new interface driver usbhid
[    7.128901] usbhid: USB HID core driver
[    7.153991] hub 1-0:1.0: USB hub found
[    7.170740] hub 1-0:1.0: 2 ports detected
[    7.178007] usbcore: registered new interface driver snd-usb-audio
[    7.335112] NET: Registered PF_INET6 protocol family
[    7.365957] mmc0: new SDHC card at address aaaa
[    7.450265] mmcblk0: mmc0:aaaa SD32G 29.7 GiB
[    7.485324] Segment Routing with IPv6
[    7.505437] In-situ OAM (IOAM) with IPv6
[    7.519794] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    7.603384] NET: Registered PF_PACKET protocol family
[    7.783633] GPT:Primary header thinks Alt. header is not at the end of the disk.
[    7.808525] GPT:251791 != 62333951
[    7.816644] GPT:Alternate GPT header not at the end of the disk.
[    7.836532] GPT:251791 != 62333951
[    7.844809] GPT: Use GNU Parted to correct GPT errors.
[    7.879316]  mmcblk0: p1 p2 p3 p4 p5
[    9.123044] debug_vm_pgtable: [debug_vm_pgtable         ]: Validating architecture page table helpers
[    9.195344] clk: Disabling unused clocks
[    9.202081] ALSA device list:
[    9.206534]   No soundcards found.
[    9.591308] EXT4-fs (mmcblk0p5): mounted filesystem 4d723087-d408-4ff9-a9a5-0261cce26711 r/w with ordered data mode. Quota mode: disabled.
[    9.608742] VFS: Mounted root (ext4 filesystem) on device 179:5.
[    9.627516] devtmpfs: mounted
[    9.914018] Freeing unused kernel image (initmem) memory: 4156K
[    9.926153] Run /sbin/init as init process
INIT: version 3.14 booting
Starting udev
[   19.451499] platform soc:sound: deferred probe pending: asoc-simple-card: parse error
[   22.905234] udevd[75]: starting version 3.2.14
[   24.379639] udevd[76]: starting eudev-3.2.14
[   31.766621] iis-idma-engine 10082000.dma-controller: using local descriptor memory at 0x84040000, size 0x4000
[   87.377562] EXT4-fs (mmcblk0p5): re-mounted 4d723087-d408-4ff9-a9a5-0261cce26711.
Fri Mar  9 13:20:01 UTC 2018
[  103.557760] random: crng init done
INIT: Entering runlevel: 5
Configuring network interfaces... udhcpc: started, v1.37.0
udhcpc: broadcasting discover
udhcpc: broadcasting discover
udhcpc: broadcasting discover
udhcpc: no lease, failing
ifup: failed to bring up eth0
Starting Dropbear SSH server: dropbear.
Starting syslogd/klogd: done

CVWSoC basic distro 5.3.1 cvwsoc-nexysa7rv32 /dev/ttyS0

Type 'root' to login with superuser privileges (no password will be asked).cvwsoc-nexysa7rv32
cvwsoc-nexysa7rv32 login: root


root@cvwsoc-nexysa7rv32:~# 
root@cvwsoc-nexysa7rv32:~# cat /proc/cpuinfo 
processor       : 0
hart            : 0
isa             : rv32imafdc_zicbom_zicboz_zicsr_zifencei_zca_zcd_zcf
mmu             : sv32
mvendorid       : 0x602
marchid         : 0x24
mimpid          : 0x100
hart isa        : rv32imafdc_zicbom_zicboz_zicsr_zifencei_zca_zcd_zcf

root@cvwsoc-nexysa7rv32:~# cat /proc/iomem 
00013000-00013fff : 13000.spi control
10000000-100000ff : serial
10060000-10060fff : 10060000.gpio control
10082000-10082fff : 10082000.dma-controller dma-controller@10082000
10090000-10091fff : 10090000.mmc mmc@10090000
100c0000-100c0fff : 100c0000.usb usb@100C0000
11000000-11001fff : 11003000.ethernet buffer
11003000-110030ff : 11003000.ethernet mac
80400000-83ffffff : System RAM
  80400000-80401fff : Reserved
  80402000-81cdb047 : Kernel image
    80402000-8092d7ab : Kernel code
    81400000-817fffff : Kernel rodata
    81c00000-81ca09ff : Kernel data
    81ca1000-81cdb047 : Kernel bss
  81cdb048-81cdbfff : Reserved
84000000-840fffff : Reserved
84100000-8503ffff : System RAM
85040000-850fffff : Reserved
  85040000-850657ff : simplefb
85100000-875fffff : System RAM
  86757000-86758fff : Reserved
  874d5000-874f4fff : Reserved
  874f6000-874f6fff : Reserved
  874f7000-874f7fff : Reserved
  874f8000-875dffff : Reserved
  875e0000-875fffff : Reserved
c03ff000-ffffffff : Reserved
root@cvwsoc-nexysa7rv32:~# free
              total        used        free      shared  buff/cache   available
Mem:          92444        7320       72312         112       12812       80408
Swap:             0           0           0
root@cvwsoc-nexysa7rv32:~# uname -a
Linux cvwsoc-nexysa7rv32 6.12.93 #1 PREEMPT Tue Jun  9 10:26:06 UTC 2026 riscv32 GNU/Linux
root@cvwsoc-nexysa7rv32:~# 

```
  

# Credits (IPs and tools used)
- Core-V Wally: https://github.com/openhwgroup/cvw
- LiteEth: https://github.com/enjoy-digital/liteeth
- SpinalHDL USB OHCI host: https://spinalhdl.github.io/SpinalDoc-RTD/master/SpinalHDL/Libraries/Com/usb_ohci.html
- PULP project: https://github.com/pulp (VGA, CDC, XBAR, adapters, etc)
- Opencores: opencores.org, https://github.com/klyone/opencores-ip (UART, SD card model for simulation, etc)
- SDHCI: https://github.com/Freakness109/sdhci/
- AHBl-to-AXI4 bridge: https://github.com/juanschroeder/ahbl-to-axi4 (custom)
- LiteDRAM: https://github.com/enjoy-digital/litedram, https://github.com/enjoy-digital/litex
- UberDDR3: https://github.com/AngeloJacobo/UberDDR3
- iDMA: https://github.com/pulp-platform/iDMA
- Verilator: https://github.com/verilator/verilator
- QEMU: https://github.com/qemu/QEMU
- Manta: https://github.com/fischermoseley/manta
- OpenXC7: https://github.com/openxc7
- Renode: https://github.com/renode/renode
- etc
