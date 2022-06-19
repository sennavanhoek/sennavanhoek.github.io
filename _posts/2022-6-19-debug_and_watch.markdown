---
layout: post
title:  "Teaching Link how to teleport"
categories: OpenOCD SWD
permalink: "/debug_and_watch/"
---

A Zelda themed introduction to manipulating memory using OpenOCD.



# Intro

Games really like to restrict where you can go, so I have many childhood memories of that one guard or rock getting in my way. So that seems like a fun setting to try some memory manipulation. I chose Link’s Awakening running on a Game&Watch. It is a handheld emulator based around an ARM stm32h7b0, and most importantly, the debug port isn't disabled!

![Image could not be loaded](/assets/GW_out.webp)

Although some protections (RDP level 1) have been enabled, we can still inspect and change the memory. To disable the remaining protections and make a backup of the device (which I would recommend if you want to mess around with it) you can take a look at [this](https://github.com/ghidraninja/game-and-watch-backup) repository. That is also where you can find the SWD pinout and OpenOCD config files. Although it isn’t necessary, I soldered a header to the PCB to make things easier.

![Image could not be loaded](/assets/GW_in.webp)  

# What to manipulate

Our goal is to make Link teleport by manipulating the player's position.
To do this we need to know at what address his location is stored and in what format.

By first running the same game, 'Link's Awakening' in [mGBA](https://mgba.io/) we can take a look at the game’s RAM before touching the device.
Link's Awakening is a GameBoy game and it has access to different types of RAM. My initial assumption was that the player position would be stored in high RAM since it is updated so often. By running around for a bit, we can see that a lot of values change but the values at address FF98 and FF99 change depending on the direction we run.

<video src="/assets/link_walking.mp4" loop autoplay muted width=600> Unable to load video. </video>

# Memory map

We now know at what location in the GameBoy's memory Link's position is stored, but we still don’t know where the GameBoy’s memory is stored on the Game&Watch… But we now have a lead; we know a value in memory we can freely control, and by working ourselves into a corner we can be sure that the same value also has to be somewhere in the Game&Watch’s memory.

![Image could not be loaded](/assets/mgba_mem.webp)

By taking a snapshot of the RAM while standing in the same spot and looking for `141A` we can find Link's coordinates in the Game&Watch's memory. But first we need to dump the RAM from our target device. When googling the chip, the first hit is a product page, where we can download the [datasheet](https://www.st.com/resource/en/datasheet/stm32h7b0ab.pdf), and go to chapter 4: Memory mapping, which is an empty page! So we need to search for it in the mentioned [reference manual](https://www.st.com/resource/en/reference_manual/rm0455-stm32h7a37b3-and-stm32h7b0-value-line-advanced-armbased-32bit-mcus-stmicroelectronics.pdf).

![Image could not be loaded](/assets/STM_docs.webp)

# Dumping the RAM

Something of note here is that the memory isn't continuous and you shouldn't try to read from reserved addresses, so we will have to dump the different RAM types one by one.


| Name | Full Name | Address | Size |
| :--- | :- | :--- | :--- |
|DTCM | (Data) Tightly-Coupled Memory| 0x20000000 | 0x020000 |
|AXI SRAM | Advanced eXtensible Interface SRAM | 0x24000000 | 0x100000 |
|AHB SRAM | AMBA High-performance Bus SRAM | 0x30000000 | 0x020000 |

After attaching the debugging probe to the Game&Watch, we can start OpenOCD and connect to it using telnet.

```
$ openocd -f "openocd/interface_stlink.cfg"
```

```
$ telnet localhost 4444
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
Open On-Chip Debugger
> dump_image DTCM_RAM.bin 0x20000000 0x20000
dumped 131072 bytes in 1.234625s (103.675 KiB/s)

> dump_image AXI_RAM.bin 0x24000000 0x100000
dumped 1048576 bytes in 9.884282s (103.599 KiB/s)

> dump_image AHB_RAM.bin 0x30000000 0x20000
dumped 131072 bytes in 1.242401s (103.026 KiB/s)

>
```

We can now search the RAM using a hex editor or by piping xxd output to grep.

```
$ xxd AXI_RAM.bin | grep "141a 0000"
```
```
$ xxd DTCM_RAM.bin | grep "1420 0000"
```
```
$ xxd AHB_RAM.bin | grep "141a 0000"
00007f90: 0000 0000 0800 0000 141a 0000 0006 0114  ................
```

It looks like we found something in AHB_RAM.bin at 0x7f90 + 8. Remember that this is a dump taken from 0x30000000, so the real address will be 0x30007f98.

# Reading and writing to memory

The full list of memory access commands is available [here](https://openocd.org/doc/html/General-Commands.html#Memory-access-commands).

There are multiple commands to read memory, dump_image writes the gathered data directly to a file but OpenOCD has a group of commands that is better suited for poking around.

| Command | Description |
| :-- | :-- |
| mdb [address] [count] | Read [count] bytes from [address] |
| mdh [address] [count] | Read [count] half-words from [address] |
| mdw [address] [count] | Read [count] words from [address] |
| mdd [address] [count] | Read [count] double-words from [address] |

Let’s try to read two bytes starting from that address to test our calculation. `mdb` and `mdh` should be the most suitable for this.

```
> mdb 0x30007f98 0x2
0x30007f98: 14 1a

> mdh 0x30007f98
0x30007f98: 1a14
```
Okay, that seems correct!

OpenOCD also has a group of very similar commands to write values.

| Command | Description |
| :-- | :-- |
| mwb [address] [byte] [count] | Write [byte] [count] times, starting from [address] |
| mwh [address] [half-word] [count] | Write [half-word] [count] times, starting from [address] |
| mww [address] [word] [count] | Write [word] [count] times, starting from [address] |
| mwd [address] [double-word] [count] | Write [double-word] [count] times, starting from [address] |

Now we have everything we need to manipulate Link's position on the Game&Watch.

# Demo

While the door is blocked, we can escape the house by teleporting to the edge of the screen!  
(This abuses a mechanic that switches maps when you walk out of bounds)

<video src="/assets/zelda_demo.mp4" controls width=600> Unable to load video. </video>
