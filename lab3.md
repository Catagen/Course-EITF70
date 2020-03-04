# Computer Organization Lab 3

https://www.eit.lth.se/fileadmin/eit/courses/eitf70/labs/Lab3.pdf

**Home assignment 1 - How many clock cycles are needed if a delay of 0.1 s is desired with a clock frequency of 16 MHz?**

1.6 * 10^6 cycles

**Home assignment 2 - How many bits do we need to count to the value above?**

21 bits

**Home assignment 3 - How many registers do we need for the delay of 0.1 s?**

Assuming the registers are 8-bits, 3

**Home assignment 4 - Write a snippet of assembly code that delays the program for roughly 0.1 s**
```assembly
#define PORTB 0x05
#define DDRB 0x04
#define LED0 0
#define LED3 3
#define decr 81

start:
	ldi r16, 0b11111111
	out DDRB, r16
	ldi r16, (1 << LED0)
	out PORTB, r16
	ldi r20, decr
	l1:
		ldi r21, decr
		l2:
			ldi r22, decr
			l3:
				dec	r22
				brne l3
				dec r21
				brne l2
				dec r20
				brne l1
	ldi r16, (1 << LED3)
	in	r17, PORTB
	or	r16, r17
	out PORTB, r16
end:
	rjmp end
```

Here, a seemingly arbitrary variable `decr` is used. To calculate the relation between this value and the delay time, I wrote this Python script

```python
clock = 16 * (10**6)

def cycles(decr):
  # Calculate the cycles of each loop
  l1 = 3 * decr
  l2 = (l1 + 4) * decr
  l3 = (l2 + 4) * decr
  return l1 + l2 + l3

def time(cycles):
  return cycles / clock

print(time(cycles(80)))

# 0.098
```

**Home assignment 5 - How do you turn on LED 2 on port B in assembly?**
```assembly
#define PORTB 0x05
#define DDRB 0x04
#define LED2 2

start:
	ldi r16, 0b11111111
	out DDRB, r16
	ldi r16, (1 << LED2)
	out PORTB, r16
end:
	rjmp end
```

**Lab question 1 - How can you use your delay code in order to create arbitrary delays? What is the limitation?**

As the maximum registry value is 255 (8 bits), setting `decr` as this value would yield approximately 3.14 seconds delay (PI conincidence? I think not).

**Task - Make LED blink at 1 Hz**

As a `decr` value of 138 yields approx 0.5 sec of delay:
```assembly
#define PORTB 0x05
#define DDRB 0x04
#define LED1 1
#define decr 138

start:
	call led_init
	call led_toggle
	call delay
	call led_toggle
	call delay
	rjmp start
end:
	rjmp end

	led_init:
		ldi r16, 0b11111111
		out DDRB, r16
	ret

	led_toggle:
		ldi r16, (1 << LED1)
		in r17, PORTB
		eor r16, r17
		out PORTB, r16
	ret

	delay:
		l1:
			ldi r21, decr
			l2:
				ldi r22, decr
				l3:
					dec	r22
					brne l3
					dec r21
					brne l2
					dec r20
					brne l1
	ret
```

**Subroutine prologue and epilogue**

The provided examples show a subroutine:
- push Y to stack => pointer - 2
- subtract n_alloc => pointer - 5
- add n_alloc => pointer + 5
- pop Y => pointer + 2

**Home assignment 6 - How do you allocate 10 integers on the stack?**

Prologue:
```assembly
in r28, STACK_L
in r29, STACK_H
sbiw Y, 10
out STACK_L, r28
out STACK_H, r29
```

Epiogue:
```assembly
in r28, STACK_L
in r29, STACK_H
adiw Y, 10
out STACK_L, r28
out STACK_H, r29
```

**Lab question 2 - What is the value of the stack pointer directly after allocation? Use the debugger.**

```assembly
#define STACK_H 0x3E
#define STACK_L 0x3D
#define N_ALLOC 5

start:
	call allocate
end:
    rjmp end

    allocate:
		in r28, STACK_L
		in r29, STACK_H
		sbiw Y, N_ALLOC
		out STACK_L, r28
		out STACK_H, r29 ; end of prologue
		in r28, STACK_L
		in r29, STACK_H
		adiw Y, 10
		out STACK_L, r28
		out STACK_H, r29
    ret
```

The address `0x3D` doesn't seem to be the correct one for the stack pointer, as that value in the memory is always 0 in the debugger. The stack pointer should have the value `0x40F5`, as we're subtracting 10 from `0x40FF` when ignoring the values pushed to stack by calling a subroutine.

**Home assignment 7 - If you allocate an array of bytes on the stack, and the stack pointer after allocation is 0x40E0, what is the address of the value at index 3?**

Should be `0x40E4`, assuming the last value in the array is located at `0x40FF`

**Task - Implement state saver**
```assembly
#define STACK_H 0x3E
#define STACK_L 0x3D
#define N_ALLOC 5
#define DDRA 0x01
#define PINA 0x00

start:
	call button_init

	infinite:
		call subroutine
		rjmp infinite
end:
    rjmp end

	button_init:
		push r16			; Save r16 value in stack
		ldi r16, 0b00000000		; Load 0b00000000 into r16
        out DDRA, r16				; Set DDRA to r16
		pop r16				; Return r16 to initial value
		ret

    subroutine:
		// Prologue
		push r16			; Save r16 value in stack
		push r17			; Save r17 value in stack
		push r18			; Save r18 value in stack
		push r24			; Save r24 value in stack

		in r28, STACK_L			; Load pointer low to r28
		in r29, STACK_H			; Load pointer high to r29
		sbiw Y, N_ALLOC			; Subtract memory allocation value from Y
		out STACK_L, r28		; Set stack low to r28
		out STACK_H, r29		; Set stack high to r29

		// Body
		ldi r24, N_ALLOC		; Set r24 to memory allocation value

		loop:
			in r18, PINA		; Load PINA to r18
			adiw Y, 1		; Add 1 to local pointer (Y)
			st Y, r18		; Write r18 to address Y
			dec r24			; Decrement r24
			brne loop

		// Epilogue
		in r28, STACK_L			; Load stack low into r28
		in r29, STACK_H			; Load stack high into r29
		adiw Y, N_ALLOC			; Add memory allocation value to Y
		out STACK_L, r28		; Set stack low to r28
		out STACK_H, r29		; Set stack high to r29

		pop r24				; Return r24 to initial value
		pop r18				; Return r18 to initial value
		pop r17				; Return r17 to initial value
		pop r16				; Return r16 to initial value
```

**Lab question 3 - What happens if we do not return the allocated memory in the subroutine**

The stack will fill up, and probably result in an error.

**Lab question 4 - Do we need to store the values in the allocated array? What happens if we just store the values randomly on the stack?**

Probably no. If we store randomly but still reset the pointer, we might get away with it although the data saved in the stack might be changed during the course of the subroutine.

**Home assignment 8 - In AVR assembly, how do you declare a subroutine? How do you call it**

Declare it in the assmebly file and make sure to add .global so that it's visible to the C program

**Home assignment 9 - In AVR assembly, how are arguments passed to and returned from a subroutine?**

Arguments are passed by using the registers `r25` to `r8`, and return values are located in `r24` &  `r25` (`r24` being least significant if return value is larger than one byte)

**Task - implement blinking LED on button press**

```assembly
#define STACK_H 0x3E
#define STACK_L 0x3D
#define PORTB 0x05
#define DDRB 0x04
#define DDRA 0x01
#define PINA 0x00
#define decr 138

.global led_init
.global button_init
.global led_on
.global led_off
.global check_button

led_init:
    ldi r18, 0b11111111
    out DDRB, r18
ret

led_on:
    ldi r18, 1				; Load 0b00000001 to r18

    loop_on:
        lsl r18				; Left shift r18
        dec r24				; Decrement r24 (the passed argument)
        brne loop_on

	or r18, PORTB			; r18 = r18 | PORTB
    out PORTB, r18			; Set PORTB to r18
ret

led_off:
    ldi r18, 0				; Load 0b00000000 to r18

    loop_off:
        lsl r18				; Left shift r18
        dec r24				; Decrement r24 (the passed argument)
        brne loop_off

	or r18, PORTB			; r18 = r18 | PORTB
    out PORTB, r18
ret

button_init:
    ldi r18, 0b00000000
    out DDRA, r18
ret

check_button:
	in r18, PINA			; Load PINA value to r18
	ldi r19, 1			; Load a one to r19
	lsr r18				; Right shift r18

	loop_btn:
		lsr r18			; Right shift r18
		dec r24			; Decrement r24 (the passed argument)
		brne loop_btn

	and r18, r19			; r18 = r18 & r19
	cpi r18, 1			; Compare r18 and 1
	breq pressed			; Branch to pressed if equal

	ldi r24, 0			; Return 0
	ret

	pressed:
		ldi r24, 1		; Return 1
		ret
```

```c
#define F_CPU 16000000UL

#include <avr/io.h>
#include <util/delay.h>

extern void led_init();
extern void button_init();
extern void led_on(char);
extern void led_off(char);
extern char check_button(char);

int main(void)
{
	led_init();
	button_init();
	
	int led = 1;
	int button = 1;
	
	while (1)
	{
		if (check_button(button)) {
			led_on(led);
			_delay_ms(500);
			led_off(led);
			_delay_ms(500);
		}
	}
}
```

**Lab Question 5 - When calling the subroutine, led_on, what is being pushed to the stack and why?**

As I am using the free registers (18-27), I am not pushing any previous values to the stack. The only thing might be the previous instruction, which is automatically stored so that we can return to the previous point of execution after calling a subroutine.

**Lab Question 6 - When the ret instruction is executed, what happens with the stack pointer?**

The location of instruction after a subroutine call is popped of the stack, thus the stack is increased by 1.

**Lab Question 7 - In the check_button subroutine, comment out the instruction where you place the return value in r24 and run the code. Does the LED blink? If so, why?**

No it's not, because the loop inside the subroutine decrements r24 (argument) until it's 0, meaning that the return value will always be 0.
