# cvwsoc README (WIP)

Core-V Wally SoC (extended Core-V Wally)

I've extended Core-V Wally core/SoC (https://github.com/openhwgroup/cvw) with additional components. on some FPGA boards I have available.

## Features
- 32/64 bits CPUs
- Yocto-based Linux images
- Open source IPs: SDHCI, DMA, USB 1.1, Ethernet, VGA, SPI, DDR2, DDR3 (LiteDRAM, UberDDR3), etc. More to come.

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



## Doom on the Genesys 2 (live, with sound)

For the moment, the only sound configuration tested is with an USB audio card.

Setup tested:
- USB audio card USB 1.1 compatible (Tested: 'C-Media Electronics Inc. USB Audio Device')
- USB keyboard (e.g. standard Logitech wireless USB keyboard)

To be able to play with the keyboard the easiest is to start 'fbdoom' in the framebuffer console (with the USB keyboard) and not in the serial console

Command:
```
SDL_AUDIODRIVER=dsp SDL_PATH_DSP=/dev/dsp SDL_AUDIO_BUFFER_SIZE=4096 fbdoom -iwad /usr/share/games/doom/freedm.wad -nomusic -timedemo demo4
```

Remarks:
- Result: about 12 fps. To be improved.
- Using default audio device (e.g.AUDIODEV=hw:0,0) gives worse performance


## Doom on the Nexys A7 [Deprecated: needs to be updated for Yocto images]

See the video: 

[![Watch the video](https://img.youtube.com/vi/7HBiZ0tc0Uk/default.jpg)](https://youtu.be/7HBiZ0tc0Uk)


Console: (using old Buildroot image => This will be updated soon)

    # /usr/bin/doom-short-demo.sh
    Starting D_DoomMain
                                FDoom 0.1
    Z_Init: Init zone memory allocation daemon. 
    zone memory: 0x7fffbd887010, 600000 allocated for zone
    Using /mnt/.fdoom.tar/ for configuration and saves
    V_Init: allocate screens.
    M_LoadDefaults: Load system defaults.
    saving config in /mnt/.fdoom.tar/default.cfg
    W_Init: Init WADfiles.
    adding /usr/share/games/doom/doom1.wad
    adding /usr/share/games/doom/bounce1.lmp
    Playing demo /usr/share/games/doom/bounce1.lmp.
    Using /mnt/.fdoom.tar/savegame/ for savegames
    ===========================================================================
                                DOOM Shareware
    ===========================================================================
    FDoom is free software, covered by the GNU General Public
    License.  There is NO warranty; not even for MERCHANTABILITY or FITNESS
    FOR A PARTICULAR PURPOSE. You are welcome to change and distribute
    copies under certain conditions. See the source for more information.
    ===========================================================================
    I_Init: Setting up machine state.
    M_Init: Init miscellaneous info.
    R_Init: Init DOOM refresh daemon - ...................
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
    timed 558 gametics in 2845 realtics (6.864675 fps)



*Remark*: for now audio would need to be with a USB audio card (not tested, drivers not in the Kernel build).


## Boards supported

Boards supported and/or planned :
- Supported:
  - Digilent Nexys A7-100T (does not fit in an A35T)
  - Genenesys 2
- Coming soon:
  - Qmtech Kintex-7
 
Necessary extra hardware:
- Digilent micro SD card PMOD (only for Nexys A7; will be removed soon)
- USB PMOD
  - https://github.com/Dolu1990/pmod_usb_host_x4/tree/main
  - https://github.com/nand2mario/usb_host_pmod
  - This store made it avaiable for purchase in 2025 when I requested it (no stock now?):
      - [https://www.aliexpress.com/store/5940159](https://www.aliexpress.com/store/5940159)
      - Alternative: [https://nl.aliexpress.com/item/1005010786670759.html](https://www.aliexpress.com/item/1005010786670759.html)
- Optional: USB keyboard (tried Logitech wireless Keyboard)
- Optional: USB audio card. Tested: https://www.amazon.de/-/en/SABRENT-External-Headphone-Adapter-AU-MMSA/dp/B00IRVQ0F8/
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


## Build steps (Nexys A7) [Deprecated]

 I don't have an Arty A7 board, so the work was done on a Nexys A7 board: https://github.com/openhwgroup/cvw/pull/1613

 Repo/branch for gateware and software: https://github.com/juanschroeder/cvw/tree/cvwsoc
 
 Gateware: 
 
     fpga/generator# make nexysa7
     
 Buildroot: use 'wally_nexysa7_defconfig'



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




## General remarks

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


## Examples

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
- AXI Stream
- I2S
- Nexys A7:
    - Add DAC for Doom audio (with limited on-board option)
    - Add SDHCI support
- Genesys 2:
    - I2S audio support
    - HDMI support
    - mini display support
- Boards:
    - Qmtech Kintex-7 support
    - GateMate board?
    - Other smaller FPGA boards
- Remove all remaining Xilinx dependencies
- JTAG debug interface?
- Renode co-simulation
- Other IPs: watchdog, gigabit Ethernet, USB 3.0
- Other PMODs:
    - display
    - HDMI?
- Refactor top files for better customisation
- CPU frequency speedup
- etc


  

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
