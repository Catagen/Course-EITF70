# Computer Organization Lab 1

https://www.eit.lth.se/fileadmin/eit/courses/eitf70/labs/Lab1.pdf

**Home assignment 1 - Write a program that blinks a LED with the frequency of 1 Hz**
```c
#define F_CPU 16000000UL
#include <avr/io.h>
#include <util/delay.h>


int main(void)
{
    led_init();
	
    while (1) 
    {
		 led_toggle(2);
		 _delay_ms(500);
    }
}
```

**Question 1 - In general, why is a while-loop needed? What would the main function return to? What happens if you remove the loop from your code?**

A while-loop is needed for the program to keep looping, as opposed to running once

**Home assignment  2 - Write a program that toggles a LED when a button is clicked,**
```c
#include <avr/io.h>
#include <stdint.h>

int main(void)
{
    button_init();
	led_init();
	
    while (1) 
    {
		if (button_read(1)) {
			led_toggle(1);
			
			while (1) {
				if (!button_read(1)) break;
			}
		}
	}
}
```

**Question 2 - How many times is it required to press the button until the LED is toggled? Why?**

The program will not "work" since the uint8_t type cannot adopt negative values

```c
#include <avr/io.h>
#include <stdint.h>

uint8_t count = 3;

int main(void)
{
    button_init();
	led_init();
	
    while (1) 
    {
		if (button_read(1)) {
			
			if (count < 0) led_toggle(1);
			
			while (1) {
				if (!button_read(1)) break;
			}
			
			count--;
		}
	}
}
```

**Question 3 - What is the problem with the code? How do you solve it?**

Decrementing the `uint8_t` type variable when = 0 yields 255

**Question 4/5 - How many bytes are required to determine the endianness? What about the values of the variables?**

When storing an integer larger than one byte, e.g 300, we get the bytes `2c 01` stored in the memory. Now, since this order of numbers yields 11265, it means that the proper way to read it is reversed `01 2c` = 300 - which means the memory stores in little endian.

**Question 6 - How is the array stored in memory? Does it conform to the endianness?**

The array `{1, 2}` is stored as `01 02` which, unlike the previous values, is not reversed.

**Question 7 - What is the difference between the size of each variable?**

Normal integers stored in two bytes, `char` in one byte, `int8` in one byte, `int16` in two byes. Unsigned versions of these are stored in the same size, just with a different method of interpretation.
```c
int main()
{
	int a=-2;
	char b='A';
	uint8_t c=1337;
	uint16_t d=1337;
	unsigned int e=1337;
	
	while(1)
	{
		a = 1;
		d = 65535;
	}
}
```

**Question 8 - What is the difference between the contents of num1 and num2 when value is decremented from 0? Explain why?**

As said above, they are stored in the same way but with different methods of interpretation. Decrementing from zero, both variables become `ff`.

**Question 9 - Explain how the value at the address in ptr1 changes?**

```c
int main()
{
	int i1=1;
	int i2=2;
	int* ptr1=&i1; // pointer declaration
	int* ptr2=&i2;
	ptr1=ptr2;
	i1=*ptr1; // accessing value at address in ptr1
	while(1);
}
```

First, we store `i1` at address `40f8` and `i2` at `40fa`. We then declare their pointers, which adopt these values. Changing `ptr1=ptr2` yields `40fa` in `ptr1`, and assigning the value of address `ptr1` (`40fa`) yields `i2=2`.

**Question 10 - Explain when the value of i changes and why?**

```c
#include <stdint.h>
void myfunc1(int p) {
	p++;
}

void myfunc2(int* p) {
	(*p)++;
}

int main(void) {
	int i=4;
	int*ptr=&i;
	while(1) {
		myfunc1(*ptr);
		myfunc2(ptr);
	}
}
```

The value `i` only changes upon calling the second function and passing it's pointer. I'm guessing this is due to the fact that the first function takes a value, and this value becomes local for the function and thus doesn't get incremented outside the function scope. The second function, taking a pointer as argument, explicitly increments the pointer value pointing at `i` in the memory. Also, I can see the value 5 (4 incremented by one) popping up for a brief moment when running the first function, at a different spot in the memory (local variable).