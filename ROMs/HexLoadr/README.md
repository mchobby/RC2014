# RC2014 MS BASIC v4.7 including `HLOAD`

This ROM works with the __Mini__, __Micro__, and __Classic__ versions of the RC2014, with 32k of RAM. This is the ROM to choose if you want fast I/O from a standard RC2014, together with the capability to upload and run C or assembly programs from within MS Basic.

__NOTE__ The `HexLoadr` Intel HEX program loader has been integrated inside MS Basic as the `HLOAD` keyword.

ACIA 6850 interrupt driven serial I/O to run modified NASCOM Basic 4.7. The receive interface has a 255 byte software buffer, together with highly optimised buffer management supporting the 68C50 ACIA receive double buffer. Receive hardware (RTS) flow control is provided. The transmit interface is also buffered, with direct cut-through when the 63 byte software buffer is empty, to ensure that the CPU is not held in wait state during serial transmission. Use 115200 baud with 8n2.

Also, this ROM provides both Intel HEX loading functions and an `RST`, `INT0`, and `NMI` RAM JumP Table, starting at `0x8000`.

This allows you to upload Assembly or compiled C programs, and then run them as described below.

```bash
RC2014 - MS Basic Loader
z88dk - feilipu

Cold | Warm start (C|W) ? C

Memory top?
Z80 BASIC Ver 4.7c
Copyright (C) 1978 by Microsoft
31948 Bytes free

Ok

```

The goal of this extension to standard MS Basic is to load an arbitrary program in Intel HEX format into an arbitrary location in the Z80 address space, and allow you to start and use your program from NASCOM Basic. Your program can be created in assembler, or in C, provided the code is available in Intel HEX format.

There are are several stages to this process.

1. At the Basic interpreter type `HLOAD`, then the command will initiate and look for your program's Intel HEX formatted information on the serial interface.
2. Once the final line of the HEX code is read into memory, `HLOAD` will return to NASCOM Basic with `ok`.
3. Start the new arbitrary program from Basic by entering the`USR(x)` command.

The `HLOAD` program can be exited without uploading a valid file by typing `:` followed by `CR CR CR CR CR CR`, or any other character.

The top of Basic memory can be readjusted by using the `RESET` function, when required. `RESET` is functionally equivalent to a cold start.

## RST locations

For convenience, because we can't easily change the ROM code interrupt routines this ROM provides for the RC2014, the ACIA serial Tx and Rx routines are reachable from your assembly program by calling the `RST` instructions from your program.

* Tx: `RST 08` expects a byte to transmit in the `a` register.
* Rx: `RST 10` returns a received byte in the `a` register, and will block (loop) until it has a byte to return.
* Rx Check: `RST 18` will immediately return the number of bytes in the Rx buffer (0 if buffer empty) in the `a` register.
* Unused: `RST 20`, `RST 28`, `RST 30` are available to the user.
* INT: `RST 38` is used by the ACIA 68B50 Serial Device through the IM1 `INT` location.
* NMI: `NMI` is unused and is available to the user.

All `RST xx` targets can be rewritten in a `JP` table originating at `0x8000` in RAM. This allows the use of debugging tools and reorganising the efficient `RST` instructions as needed. For more information on configuring and using the `RST xx` targets [refer to the example in the Wiki](https://github.com/RC2014Z80/RC2014/wiki/Using-Z88DK#basic-subtype).

## USR Jump Address & Parameter Access

For the RC2014 with 32k Basic the `USR(x)` loaded user program address is located at `0x8204`.

Your assembly program can receive a 16 bit parameter passed in from the function by calling `DEINT` at `0x0AF3`. The parameter is stored in register pair `DE`.

When your assembly program is finished it can return a 16 bit parameter stored in `A` (MSB) and `B` (LSB) by jumping to `ABPASS` which is located at `0x12CC`.

Note that these address of these functions can also be read from `0x024B` for `DEINT` and `0x024D` for `ABPASS`, as noted in the NASCOM Basic Manual.

``` asm
                                ; from Nascom Basic Symbol Tables
DEINT           .EQU    $0AF3   ; Function DEINT to get USR(x) into DE registers
ABPASS          .EQU    $12CC   ; Function ABPASS to put output into AB register for return


                .ORG    9000H   ; your code origin, for example
                CALL    DEINT   ; get the USR(x) argument in DE

                                ; your code here

                JP      ABPASS  ; return the 16 bit value to USR(x). Note JP not CALL
```

# Program Usage

1. Select the preferred origin `.ORG` for your arbitrary program, and assemble a HEX file using your preferred assembler, or compile a C program using z88dk. For the RC2014 32kB, suitable origins commence from `0x8400`, and the default z88dk origin for RC2014 is `0x9000`.

2. Give the `HLOAD` command within Basic.

3. Using a serial terminal, upload the HEX file for your arbitrary program that you prepared in Step 1, using the Linux `cat` utility or similar. If desired the python `slowprint.py` program can also be used for this purpose. `python slowprint.py > /dev/ttyUSB0 < myprogram.hex` or `cat > /dev/ttyUSB0 < myprogram.hex`. The RC2014 interface can absorb full rate uploads, so using `slowprint.py` is an unnecessary precaution.

4. Start your program by typing `PRINT USR(0)`, or `? USR(0)`, or other variant if you have an input parameter to pass to your program.

5. Profit.

## Notes

Note that your C or assembly program and the `USR(x)` jump address setting will remain in place through a RC2014 Warm Reset, provided you prevent Basic from initialising the RAM locations you have used.

Any Basic programs loaded will also remain in place during a Warm Reset.

Issuing the `RESET` keyword will clear the RC2014 RAM, and provide an option to return the original memory size. `RESET` is functionally equivalent to a cold start.

# Credits

Derived from the work of @fbergama and @foxweb at RC2014.

https://github.com/RC2014Z80/RC2014/blob/master/ROMs/hexload/hexload.asm

# Copyright

NASCOM ROM BASIC Ver 4.7, (C) 1978 Microsoft

Scanned from source published in 80-BUS NEWS from Vol 2, Issue 3 (May-June 1983) to Vol 3, Issue 3 (May-June 1984).

Adapted for the freeware Zilog Macro Assembler 2.10 to produce the original ROM code (checksum A934H). PA

http://www.nascomhomepage.com/

---

The updates to the original BASIC within this file are copyright (C) Grant Searle

You have permission to use this for NON COMMERCIAL USE ONLY.
If you wish to use it elsewhere, please include an acknowledgement to myself.

http://searle.wales/

---

The rework to support MS Basic HLOAD, RESET, and the 8085 and Z80 instruction tuning are copyright (C) 2021 Phillip Stevens.

This Source Code Form is subject to the terms of the Mozilla Public License, v. 2.0. If a copy of the MPL was not distributed with this file, You can obtain one at http://mozilla.org/MPL/2.0/.

Further source maintenance at https://github.com/feilipu/NASCOM_BASIC_4.7

@feilipu, October 2021
