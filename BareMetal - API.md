# BareMetal x86-64 API #

Version 1 - January 9, 2015

### Notes

BareMetal on x86-64 abides by the [System V AMD64 ABI](www.x86-64.org/documentation/abi.pdf). Therefore, register parameters are passed via RDI, RSI, RDX, RCX, R8, R9.

### Contents

1. Output
	- b\_output
	- b\_output\_chars
2. Input
	- b\_input
	- b\_input\_key
3. SMP
	- b\_smp\_enqueue
	- b\_smp\_dequeue
	- b\_smp\_run
	- b\_smp\_wait
4. Memory
	- b\_mem\_allocate
	- b\_mem\_release
5. Network
	- b\_net\_tx
	- b\_net\_rx
6. Disk
	- b\_disk\_read
	- b\_disk\_write
7. Misc
	- b\_system\_config
	- b\_system\_misc

## Output


### b\_output

Output text to the screen or via serial depending on configuration (The string must be null-terminated - also known as ASCIIZ).

Assembly Registers:

	 IN:	RSI = message location (zero-terminated string)
	OUT:	All registers preserved

Assembly Example:

	mov rsi, Message
	call b_output
	...
	Message: db 'This is a test', 0

C Example:

	char Message[] = "This is a test";
	b_output(Message);
	
	b_output("This is a another test");


### b\_output\_chars

Output a number of characters to the screen or via serial depending on configuration.

Assembly Registers:

	 IN:	RSI = message location
			RCX = number of characters to output
	OUT:	All registers preserved

Assembly Example:

	mov rsi, Message
	mov rcx, 4
	call os_output_chars					; Only output the word 'This'
	...
	Message: db 'This is a test', 0

C Example:

	b_output_chars("This is a test", 4);	// Output 'This'
	
	char Message[] = "Hello, world!";
	b_output_chars(Message, 5);				// Output 'Hello'


## Input


### b\_input

Accept a number of keys from the keyboard or via serial depending on configuration. The resulting string will automatically be null-terminated.

Assembly Registers:

	 IN:	RDI = location where string will be stored
			RCX = number of characters to accept
	OUT:	RCX = length of string that were input (NULL not counted)
			All other registers preserved

Assembly Example:

	mov rdi, Input
	mov rcx, 20
	call b_input
	...
	Input: db 0 times 21

C Example:

	char Input[21];
	b_input(Input, 20);


### b\_input\_key

Scans for input from keyboard or serial depending on configuration.

Assembly Registers:

	 IN:	Nothing
	OUT:	AL = 0 if no key pressed, otherwise ASCII code, other regs preserved
			All other registers preserved

Assembly Example:

	call b_input_key
	mov byte [KeyChar], al
	...
	KeyChar: db 0

C Example:

	char KeyChar;
	KeyChar = b_input_key();
	if (KeyChar == 'a')
	...


## SMP


### b\_smp\_enqueue

Add a workload to the processing queue


### b\_smp\_dequeue

Dequeue a workload from the processing queue


### b\_smp\_run

Call the code address stored in RAX


### b\_smp\_wait

Wait until all other CPU Cores are finished processing


## Memory


### b\_mem\_allocate

Allocate pages of memory

Assembly Registers:

	 IN:	RCX = Number of pages to allocate
	OUT:	RAX = Starting address (Set to 0 on failure)
			All other registers preserved

Assembly Example:

	mov rcx, 2		; Allocate 2 2MiB pages (4MiB in total)
	call b_mem_allocate
	jz mem_fail
	mov rsi, rax	; Copy memory address to RSI


### b\_mem\_release

Release pages of memory

Assembly Registers:

	 IN:	RAX = Starting address
			RCX = Number of pages to free
	OUT:	RCX = Number of pages freed
			All other registers preserved

Assembly Example:

	mov rax, rsi	; Copy memory address to RAX
	mov rcx, 2		; Free 2 2MiB pages (4MiB in total)
	call b_mem_release


## Network


### b\_net\_tx

Transmit data via Ethernet

Assembly Registers:

	 IN:	RSI = memory location of packet
			RCX = length of packet
	OUT:	All registers preserved

Assembly Example:

	mov rsi, Packet
	mov rcx, 1500
	call b_net_tx
	...
	Packet:
	Packet_Dest: db 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF ; Broadcast
	Packet_Src: db 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
	Packet_Type: dw 0xABBA
	Packet_Data: db 'This is a test', 0

The packet must contain a proper 14-byte header.


### b\_net\_rx

Receive data via Ethernet

Assembly Registers:

	 IN:	RDI = memory location to store packet
	OUT:	RCX = length of packet, 0 if nothing to receive

Assembly Example:

	mov rdi, Packet
	call b_net_rx
	...
	Packet: times 1518 db 0

Notes: BareMetal OS does not keep a buffer of received packets. This means that the OS will overwrite the last packet received as soon as it receives a new one. You can continuously poll the network by checking `b_net_rx` often, but this is not ideal. BareMetal OS allows for a network interrupt callback handler to be run whenever a packet is received. With a callback, your program will always be aware of when a packet was received. Check `programs/ethtool.asm` for an example of using a callback.


## Disk

BareMetal uses 4096 byte sectors for all disk access.

### b\_disk\_read

Read a number of bytes from disk to memory

Assembly Registers:

	 IN:	RCX = Number of bytes to read
			RSI = Starting sector
			RDI = Destination memory address
	OUT:	RCX = Number of bytes read
			All other registers preserved


### b\_disk\_write

Write a number of bytes from memory to disk

Assembly Registers:

	 IN:	RCX = Number of bytes to write
			RSI = Source memory address
			RDI = Starting sector
	OUT:	RCX = Number of bytes written
			All other registers preserved


## Misc


### b\_system\_config

View or modify system configuration options

Assembly Registers:

	 IN:	RDX = Function #
			RAX = Variable 1
	OUT:	RAX = Result 1

Function numbers come in pairs (one for reading a parameter, and one for writing a parameter). `b_system_config` should be called with a function alias and not a direct function number.


### b\_system\_misc

Call miscellaneous OS sub-functions

Assembly Registers:

	 IN:	RDX = Function #
			RAX = Variable 1
			RCX = Variable 2 
	OUT:	RAX = Result 1
			RCX = Result 2

