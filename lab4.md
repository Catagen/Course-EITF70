# Computer Organization Lab 4

https://www.eit.lth.se/fileadmin/eit/courses/eitf70/labs/Lab4.pdf

**Home assignment 1 - How do you initialize the used I/O-pins as inputs?**

To initialize port C as an input, we set the DDRC6/7 bits to 0: `DDRC |= (0<<DDRC6) | (0<<DDRC7);`

**Home assignment 2 - Write code that polls the opto interrupters and counts the number of lions that are out in the wild zone.**

```c

#include <avr/io.h>
#include <stdint.h>

uint8_t lions_in_wild = 0;

void init_sensors() {
	DDRC |= (0<<DDRC6) | (0<<DDRC7);
}

void init_leds() {
	DDRB |= 0xff;
}

uint8_t sens1() {
	return (PINC & (1 << PORTC6)) >> PORTC6;
}

uint8_t sens2() {
	return (PINC & (1 << PORTC7)) >> PORTC7;
}

uint8_t state() {
	return sens1() | (sens2() << 1);
}

void lions() {
	static uint8_t last_state = 0;
	static uint8_t enter_state = 0;
	
	if (state() != last_state) {
		if (last_state == 0) enter_state |= state();
		
		if (state() == 0) {
			// End of sequence
			if (enter_state == 1 && last_state == 2) lions_in_wild++;
			if (enter_state == 2 && last_state == 1) lions_in_wild--;
			enter_state = 0;
		}
		
		last_state = state();
	}
}

int main(void)
{
	init_sensors();
	init_leds();

	while (1)
	{
		lions();
		PORTB = lions_in_wild;
	}
}
```

**Lab question 1 - Is it necessary to wait for the whole sequence that is generated when a lion passes through the hallway?**

No, we do not need to wait for the full sequence. We need to know that the lion has entered the passage, meaning the state has gone from (0,0) to either (1,0) or (0,1) - telling us which way it entered. After that, we can save the last state every time it changes. Upon detecting a (0,0) state, we can look at the last saved state (which should be (0,1) or (1,0)) and determine which way it exited. See code above.

**Lab Question 2 - How fast can an obstacle move through the sensor array without it being missed?**

If something passes a sensor faster than the processor tick frequency, it will not be detected.

**Lab Question 3 - OPTIONAL: Suddenly the lions have gained the ability to walk backwards. Do the necessary changes to your code!**

The code should work despite the lions walking backwards.

**High tech lion cage**

```c
int main(void)
{	
	init_sensors();
	init_leds();
	
	security_system_init();

	while (1)
	{
		security_system_run();
		lions();
		send_lions(lions_in_wild);
	}
}
```

**Lab question 4 - If a lion moves fast through the senors is it still registered? If not, why?**

See question 2

**Home assignment 3 - Look on page 13 in the datasheet to see which PCINT (Pin Change Interrupt) the sensors are connected.**

PC6: PCINT22
PC7: PCINT23

**Home assignment 4 - Write code that enables the correct Pin Change Interrupt in the corresponding Pin Change Mask Register, PCMSK**

`PCMSK2 |= (1<<PCINT22) | (1<<PCINT23);`

**Home assignment 5 - How do you enable the pin change interrupt for the corresponding port in the Pin Change Interrupt Control Register, PCICR?**

`PCICR |= 1<<PCIE2`

**Test interrupt service**
```
#include <avr/interrupt.h>

volatile uint8_t counter = 0;

void init_sensors() {
	DDRC |= (0<<DDRC6) | (0<<DDRC7);
}

int main(void)
{	
	init_sensors();
	init_interrupt();
	sei();

	while (1)
	{
	}
}

ISR(PCINT2_vect)
{
	counter++;
}
```

**Lab question 5 - What is the value of the variable?**

**Final Code**
```
#include <avr/interrupt.h>
#include <avr/io.h>
#include <stdint.h>

volatile uint8_t lions_in_wild = 0;

void init_interrupt() {
	PCMSK2 |= (1<<PCINT22) | (1<<PCINT23);
	PCICR |= 1<<PCIE2;
}

void init_sensors() {
	DDRC |= (0<<DDRC6) | (0<<DDRC7);
}

void init_leds() {
	DDRB |= 0xff;
}

uint8_t sens1() {
	return (PINC & (1 << PORTC6)) >> PORTC6;
}

uint8_t sens2() {
	return (PINC & (1 << PORTC7)) >> PORTC7;
}

uint8_t state() {
	return sens1() | (sens2() << 1);
}

void lions() {
	static uint8_t last_state = 0;
	static uint8_t enter_state = 0;
	
	if (state() != last_state) {
		if (last_state == 0) enter_state |= state();
		
		if (state() == 0) {
			// End of sequence
			if (enter_state == 1 && last_state == 2) lions_in_wild++;
			if (enter_state == 2 && last_state == 1) lions_in_wild--;
			enter_state = 0;
		}
		
		last_state = state();
	}
}

int main(void)
{	
	init_sensors();
	init_leds();
	init_interrupt();
	security_system_init();
	sei();

	while (1)
	{
		security_system_run();
	}
}

ISR(PCINT2_vect)
{
	lions();
	send_lions(lions_in_wild);
}
```