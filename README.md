Keyboard Controller. [🙋]

[🙋]: https://github.com/qb40/keyboard-controller

<br>

```
The keyboard controller has three 8-bit registers involved in communication with CPU.
INPUT buffer		port 60h or 64h		(WRITE)
OUTPUT buffer		port 60h		(READ)
STATUS register		port 64h		(READ)
If CPU writes to port 64h, it is interpreted as command byte.
If CPU writes to port 60h, it is interpreted as data byte.


Ths STATUS register
-------------------
Bit 7: Parity error
Bit 6: Timeout
Bit 5: Auxiliary output buffer full
Bit 4: Keyboard lock
Bit 3: Command/Data
Bit 2: System flag
Bit 1: Input buffer status
Bit 0: Output buffer status

Bit 7: Parity error
0 - OK
1 - There was parity error with last byte

Bit 6: Timeout
0 - OK
1 - Timeout(General[PS/2];transmission timeout from keyboard)

Bit 5: Auxiliary output buffer full
[PS/2]
0 - Keyboard data
1 - Mouse data

[AT systems]
0 - OK
1 - Timeout on transmission from keyboard controller to keyboard
    (may indicate that keyboard is absent)

Bit 4: Keyboard lock
0 - Locked
1 - Not locked

Bit 3: Command/Data
0 - Last write to input buffer was data(60h)
1 - Last write to input buffer was command(64h)

Bit 2: System flag
0 - Keyboard controller not tested(after power reset)
1 - Keyboard controller self-test complete
    (Basic Assurance Test)(BAT)

Bit 1: Input buffer status
0 - Input buffer empty(can be written)
1 - Input buffer full(can't be written)

Bit 0: Output buffer status
0 - Output buffer empty(don't read)
1 - Output buffer full(can be read)(60h)


The keyboard controller command byte
------------------------------------
The keyboard controller is provided with some RAM, for example 32 bytes, that can be accessed by the CPU.
Most important - byte 0 : Command Controller Byte (CCB)
It can be accessed by writing 20h to port 64h
It can be read/written by reading/writing to or from port 60h

Bit 7: Unused
0 - Always

Bit 6: Translate
0 - No translation
1 - Translation of keyboard scancodes, using the translation table.
    MCA type 2 controllers annot set this bit to 1.
    In that case scan code conversion is set using command 0F0h to port 60h

Bit 5: Mouse enable
[EISA or PS/2]
0 - Enable mouse
1 - Disable mouse

[ISA system "PC mode"]
0 - Use 11-bit codes, parity check and do scan conversion
1 - Use 8086 codes, no parity check, no scan conversion

Bit 4: Keyboard enable
0 - Enable keyboard
1 - Diasble keyboard

Bit 3: Ignore keyboard lock
[PS/2]
0 - Always

[AT]
0 - No action
1 - Force bit 4 of status register to 1, "not locked"
    used for testing keyboard

Bit 2: System flag
This is bit 2 of status register
0 - Keyboard controller not tested(after power reset)
1 - Keyboard controller self-test complete
    (Basic Assurance Test)(BAT)

Bit 1: Mouse interrupt enable
[ISA system]
0 - Always

[EISA or PS/2]
0 - Mouse interrupts diabled
1 - Mouse interrupts enable(IRQ 12)

Bit 0: Keyboard interrupt enable
0 - Keyboard interrupts disabled
1 - Keyboard interrupts enabled(IRQ 1)
    CPU has to poll bits 0 and 5 of status register

Keyboard controller commands
----------------------------
Useful and generally available keyboard commands:
20h - Read keyboard controller command byte
60h - Write keyboard controller command byte
AAh - Self test
ABh - Interface test
ADh - Disable keyboard
AEh - Enable keyboard
C0h - Read input port
D0h - Read output port
D1h - Write output port
E0h - Read test inputs
FEh - System reset

Useful and generally available mouse commands:
A7h - Disable mouse port
A8h - Enable mouse port
A9h - Test mouse port
D4h - Write to mouse

Obscure and obsolete commands:
00h-1Fh		Read keyboard controller RAM
20h-3Fh		Read keyboard controller RAM
40h-5Fh		Write keyboard controller RAM
60h-7Fh		Write keyboard controller RAM
90h-93h		Synaptics multiplexer prefix
90h-9Fh		Write Port13-Port10
A0h		Read copyright
A1h		Read frimware version
A2h		Switch speed
A3h		Switch speed
A4h		Check if password is installed
A5h		Load password
A6h		Check password
ACh		Diagnostic jump
AFh		Read keyboard version
B0h-B5h		Reset keyboard controller line
B8h-BDh		Set keyboard conroller line
C1h		Continuous input port poll, low
C2h		Continuous input port poll, high
C8h		Unblock lines P22 and P23
C9h		Block lines P22 and P23
CAh		Read keyboard controller mode
CBh		Write keyboard controller mode
D2h		Write keyboard output buffer
D3h		Write mouse output buffer
DDh		Disable A20 address line
DFh		Enable A20 address line
F0h-FFh		Pulse output bit

Command 00h-1Fh: Read keyboard controller RAM
Aliases for 20h-3Fh(AMIBIOS only)

Command 20h-3Fh: Read keyboard controller RAM
The last six bits of the command specify the RAM address to read. The read data is placed into the output buffer, and can be read by reading port 0x60. On MCA systems, type 1 controllers can access all 32 locations; type 2 controllers can only access locations 0, 0x13-0x17, 0x1d, 0x1f.
Location 00h - Command byte
Location 13h - nonzero when password enabled(on MCA)
Location 14h - nonzero when password matched(on MCA)
Loaction 16h-17h	two make codes to be discarded during password matching(on MCA)

Command 40h-5Fh: Write keyboard controller RAM

Command 60h-7Fh: Write keyboard controller RAM

Command 90h-93h: Synaptics routing prefixes
Prefix a PS/2 mouse command with one of these to talk to one of at most four multiplexed devices.
Unfortunately, VIA also uses this command.

Command 90h-9Fh: Write Port13-Port10
(VIA VT82C42) Write low nibble to Port13-Port10.

Command A0h: Read copyright
On some keyboard controllers: a single ASCII byte is made available for reading via port 0x60. On other systems: no effect, the command is ignored.

Command A1h: Read controller firmware version
On some keyboard controllers: a single ASCII byte is made available for reading via port 0x60. On other systems: no effect, the command is ignored.

Command A2h: Switch speed
(On ISA/EISA systems with AMI BIOS) Reset keyboard controller lines P22 and P23 low. These lines can be used for speed switching via the keyboard controller. When done, the keyboard controller sends one garbage byte to the system.

Command A3h: Switch speed
(On ISA/EISA systems with AMI BIOS) Set keyboard controller lines P22 and P23 high. These lines can be used for speed switching via the keyboard controller. When done, the keyboard controller sends one garbage byte to the system.
(Compaq BIOS: Enable system speed control.)

Command A4h: Check if password installed
On MCA systems: Return 0xf1 (via port 0x60) when no password is installed, return 0xfa when a password has been installed. Some systems without password facility always return 0xf1.
(On ISA/EISA systems with AMI BIOS) Write Clock = Low.
(Compaq BIOS: toggle speed.)

Command A5h: Load password
On MCA systems: Load a password by writing a NUL-terminated string to port 0x60. The string is in scancode format.
(On ISA/EISA systems with AMI BIOS) Write Clock = High.
(Compaq BIOS: special read of P2, with bits 4 and 5 replaced: Bit 5: 0: 9-bit keyboard, 1: 11-bit keyboard. Bit 4: 0: outp-buff-full interrupt disabled, 1: enabled.)

Command A6h: Check password
On MCA systems: When a password is installed: Check password by matching keystrokes with the stored password. Enable keyboard upon successful match.
(On ISA/EISA systems with AMI BIOS) Read Clock. 0: Low. 1: High.

Command A7h: Disable mouse port
On MCA systems: disable the mouse (auxiliary device) by setting its clock line low, and set bit 5 of the Command byte. Now P23 = 1.(On ISA/EISA systems with AMI BIOS) Write Cache Bad.

Command A8h: Enable mouse port
On MCA systems: enable the mouse (auxiliary device), clear bit 5 of the Command byte. Now P23 = 0.
(On ISA/EISA systems with AMI BIOS) Write Cache Good.

Command A9h: Test mouse port
On MCA and other systems: test the serial link between keyboard controller and mouse. The result can be read from port 0x60. 0: OK. 1: Mouse clock line stuck low. 2: Mouse clock line stuck high. 3: Mouse data line stuck low. 4: Mouse data line stuck high. 0xff: No mouse.(On ISA/EISA systems with AMI BIOS) Read Cache Bad or Good. 0: Bad. 1: Good.

Command AAh: Self test
Perform self-test. Return 0x55 if OK, 0xfc if NOK.

Command ABh: Interface test
Test the serial link between keyboard controller and keyboard. The result can be read from port 0x60. 0: OK. 1: Keyboard clock line stuck low. 2: Keyboard clock line stuck high. 3: Keyboard data line stuck low. 4: Keyboard data line stuck high. 0xff: General error.

Command ACh: Diagnostic dump
(On some systems) Read from port 0x60 sixteen bytes of keyboard controller RAM, and the output and input ports and the controller's program status word.

Command ADh: Disable keyboard
Disable the keyboard clock line and set bit 4 of the Command byte. Any keyboard command enables the keyboard again.

Command AEh:  Enable keyboard
Enable the keyboard clock line and clear bit 4 of the Command byte.

Command AFh: Read keyboard version
(Award BIOS, VIA)

Command B0h-B5h,B8h-BDh: Reset/set keyboard controller line
AMI BIOS: Commands 0xb0-0xb5 reset a keyboard controller line low. Commands 0xb8-0xbd set the corresponding keyboard controller line high. The lines are P10, P11, P12, P13, P22 and P23, respectively. (In the case of the lines P10, P11, P22, P23 this is on ISA/EISA systems only.) When done, the keyboard controller sends one garbage byte to the system.
VIA BIOS: Commands 0xb0-0xb7 write 0 to lines P10, P11, P12, P13, P22, P23, P14, P15. Commands 0xb8-0xbf write 1 to lines P10, P11, P12, P13, P22, P23, P14, P15.

Command C0h: Read input port
Read the input port (P1), and make the resulting byte available to be read from port 0x60.

Command C1h: Continuous input port poll, low
(MCA systems with type 1 controller only) Continuously copy bits 3-0 of the input port to be read from bits 7-4 of port 0x64, until another keyboard controller command is received.

Command C2h: Continuous input port poll, high
(MCA systems with type 1 controller only) Continuously copy bits 7-4 of the input port to be read from bits 7-4 of port 0x64, until another keyboard controller command is received.

Command C8h: Unblock keyboard controller lines P22 and P23
(On ISA/EISA systems with AMI BIOS) After this command, the system can make lines P22 and P23 low/high using command 0xd1.

Command C9h:  Block keyboard controller lines P22 and P23
(On ISA/EISA systems with AMI BIOS) After this command, the system cannot make lines P22 and P23 low/high using command 0xd1.

Command CAh: Read keyboard controller mode
(AMI BIOS, VIA) Read keyboard controller mode to bit 0 of port 0x60. 0: ISA (AT) interface. 1: PS/2 (MCA)interface.

Command CBh: Write keyboard controller mode
(AMI BIOS) Write keyboard controller mode to bit 0 of port 0x60. 0: ISA (AT) interface. 1: PS/2 (MCA)interface. (First read the mode using command 0xca, then modify only the last bit, then write the mode using this command.)

Command D0h: Read output port
Read the output port (P2) and place the result in the output buffer. Use only when output buffer is empty.

Command D1h: Write output port
Write the output port (P2). Note that writing a 0 in bit 0 will cause a hardware reset.
(Compaq: the system speed bits are not set. Use commands 0xa1-0xa6 for that.)

Command D2h: Write keyboard output buffer
(MCA) Write the keyboard controllers output buffer with the byte next written to port 0x60, and act as if this was keyboard data. (In particular, raise IRQ1 when bit 0 of the Command byte says so.)

Command D3h:  Write mouse output buffer
(MCA) Write the keyboard controllers output buffer with the byte next written to port 0x60, and act as if this was mouse data. (In particular, raise IRQ12 when bit 1 of the Command byte says so.)Not all systems support this.Synaptics multiplexing On the other hand, Synaptics (see ps2-mux.PDF) uses this command as a handshake between driver and controller: if the driver gives this command three times, with data bytes 0xf0, 0x56, 0xa4 respectively, and reads 0xf0, 0x56, but not 0xa4 back from the mouse output buffer, then the driver knows that the controller supports Synaptics AUX port multiplexing, and the controller knows that it does not have to do the usual data faking and goes into multiplexed mode. The third byte read is the version of the Synaptics standard.There is a corresponding deactivation sequence, namely 0xf0, 0x56, 0xa5. (And again the last byte is changed to the version number of the standard supported.) This latter sequence works both in multiplexed mode and in legacy mode and can thus be used to determine whether this feature is present without activating it.See also the multiplexer commands 0x90-0x93.For some laptops it has been reported that bit 3 of every third mouse byte is forced to 1 (as it would be with the standard 3-byte mouse packets). This may turn 0xf0, 0x56, 0xa4 into 0xf0, 0x56, 0xac and cause misdetection of Synaptics multiplexing (for version 10.12).

Command D4h: Write to mouse
(MCA) The byte next written to port 0x60 is transmitted to the mouse.

Command DDh: Disable A20 address line
(HP Vectra)

Command DFh: Enable A20 address line
(HP Vectra)

Command E0h: Read test inputs
This command makes the status of the Test inputs T0 and T1 available to be read via port 0x60 in bits 0 and 1, respectively. Use only when the output port is empty.

Command F0h-FFh: Pulse output bit
Bits 3-0 of the output port P2 of the keyboard controller may be pulsed low for approximately 6 µseconds. Bits 3-0 of this command specify the output port bits to be pulsed. 0: Bit should be pulsed. 1: Bit should not be modified. The only useful version of this command is Command 0xfe. (For MCA, replace 3-0 by 1-0 in the above.)

Command FEh: System reset
Pulse bit 0 of the output port P2 of the keyboard controller. This will reset the CPU.

The input port P1
-----------------

This has the following layout.

bit 7		Keyboard lock			0: locked, 1: not locked
bit 6		Display				0: CGA, 1: MDA
bit 5		Manufacturing jumper		0: installed, 1: not installed
		with jumper the BIOS runs an infinite diagnostic loop
bit 4		RAM on motherboard		0: 512 KB, 1: 256 KB
bit 3		Unused in ISA, EISA, PS/2 systems
		Can be configured for clock switching
bit 2		Unused in ISA, EISA, PS/2 systems
		Can be configured for clock switching
		Keyboard power 	PS/2 MCA:	0: keyboard power normal, 1: no power
bit 1		Mouse data in			Unused in ISA
bit 0		Keyboard data in		Unused in ISA

Clearly only bits 1-0 are input bits. Of the above, the original IBM AT used bits 7-4, while PS/2 MCA systems use only bits 2-0.Where in the above lines P10, P11, etc are used, these refer to the pins corresponding to bit 0, bit 1, etc of port P1.

The output port P2
------------------

This has the following layout.

bit 7		Keyboard data, data to keyboard
bit 6		Keyboard clock
bit 5		IRQ12				0: IRQ12 not active, 1: active
bit 4		IRQ1				0: IRQ1 not active, 1: active
bit 3		Mouse clock			Unused in ISA
bit 2		Mouse data			Unused in ISA. Data to mouse
bit 1		A20				0: A20 line is forced 0, 1: A20 enabled
bit 0		Reset				0: reset CPU, 1: normal

Where in the above lines P20, P21, etc are used, these refer to the pins corresponding to bit 0, bit 1, etc of port P2.

The test port T
---------------

bit 0		Keyboard clock (input).
bit 1		(AT) Keyboard data (input). (PS/2) Mouse clock (input).
```

<br>
<br>


[![qb40](https://i.imgur.com/xAWLn0I.jpg)](https://qb40.github.io)
