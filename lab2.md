# Computer Organization Lab 2

https://www.eit.lth.se/fileadmin/eit/courses/eitf70/labs/Lab2.pdf

**Home assignment 1 - Using a pointer to the address, how do you turn LED number 3 (only) on?**
```c
#include <avr/io.h>

int main(void)
{
	int* ddrB = 0x24;
	int* porB = 0x25;
	
	// Set data direction to out
	*ddrB |= 0b00001000;
	
	// Set value of port 3 (index 4) to 1
	*porB |= 0b00001000;
}
```

**Home assignment 2 - Using a pointer to the address, how do you read the value of (only) button 4?**
```c
#include <avr/io.h>

int main(void)
{
	int* ddrA = 0x21;
	int* pinA = 0x20;
	
	// Set data direction to in for all buttons
	*ddrA |= 0b00000000;
	
	while (1) {
		if (*pinA >> 5 & 1) // Button 4 pressed
	}
}
```

**Task - implement state machine**
```c
#include <avr/io.h>
#include <stdint.h>

uint8_t* init_leds() {
	// Reference to DDR B
	volatile uint8_t* ddrB = 0x24;

	// Set data direction to out
	*ddrB |= 0b11111110;

	// Return reference to PORT B
	return 0x25;
}

uint8_t* init_buttons() {
	// Reference to DDR A
	volatile uint8_t* ddrA = 0x21;

	// Set data direction to in
	*ddrA |= 0b00000000;

	// Return reference to PIN A
	return 0x20;
}

int main(void)
{
	volatile uint8_t* leds = init_leds();
	volatile uint8_t* buttons = init_buttons();

	while (1) {
		volatile uint8_t button_state = *buttons;
		
		*leds ^= button_state >> 1;

		while (*buttons == button_state);
	}
}
```

**Task - state machine with macros**
```c
#include <avr/io.h>
#include <stdint.h>

uint8_t* init_leds() {
	DDRB |= 0b11111111;
}

uint8_t* init_buttons() {
	DDRD |= 0<<DDRD7;
}

int main(void)
{
	init_leds();
	init_buttons();

	while (1) {
		volatile uint8_t button_state = PIND & 1<<PIND7;
		
		PORTB ^= button_state;
		
		while ((PIND & 1<<PIND7) == button_state);
	}
}
```

**Home assignment 3 - When pressing a mechanical button, do you get a perfectly clean signal? If no, how do we handle such situations?**

The signal is not clean, and the problem could be solved with a delay.

**Home assignment 4 - Create a function, uint8_t button_read_reliably(), that reads the button and excludes the bounces**
```c
#include <avr/io.h>
#include <stdint.h>
#include <util/delay.h>

uint8_t* init_leds() {
	DDRB |= 0b11111111;
}

uint8_t* init_buttons() {
	DDRD |= 0<<DDRD7;
}

uint8_t button_read_reliably() {
	
	volatile uint8_t button_state = (PIND & 1<<PIND7)>>PIND7;
	
	if (button_state & 1) {
		_delay_ms(15);
		
		button_state = (PIND & 1<<PIND7)>>PIND7;
		
		if (button_state & 1) return 1;
	}
	
	return 0;
}

int main(void)
{
	init_leds();
	init_buttons();

	while (1) {
		if (button_read_reliably()) {
			PORTB ^= 1;
			
			while (button_read_reliably());
		}
	}
}
```

**Home assignment 5 - How do you enable the transmitter and receiver for USART0?**

- `RDNx`: Write = write to transmit data buffer
- `RDNx`: Read = read from receive data buffer
- `UCSR0B`: Contains bits that enable the USART receiver and transmitter

By setting bit `TXEN` (3) to 1 in `USART` Control and Status Register n B `UCSR0B`

**Home assignment 6 - How do you set the data length to 8 bits and 1 stop bit for USART0?**

By changing USART Control and Status Register n C, but 8 data bits and 1 parity bit is already the default

**Home assignment 7 - How do you set the baud rate for USART0? You are free to choose any applicable value.**

`UBRR0H`: Most significant bits of baud rate
`UBRR0L`: Least significant bits of baud rate

Setting the baud rate to 103 would yield:
`UBRR0H`: 00000000
`UBRR0L`: 01100111

**Home assignemnt 8 - How do you read the received data from the USART0? Remember that you need to wait until there is data to read.**

By reading `RDN0`

**Home assignment 9 - How do you transmit data via USART0?**

By writing to `RDN0`

**Task - Echoing + lights**
```c
#include <avr/io.h>
#include <stdint.h>

void usart0_init() {
	// Set Baud
	UBRR0 = 103;

	// Enable transmitter and receiver
	UCSR0B = 0b00011000;
	
	// Set config
	UCSR0C = 0b00000110;
}

uint8_t usart0_receive() {
	// Wait until RXC bit becomes 1 (Receive complete)
	while ( !(UCSR0A & 0b10000000) );
	return UDR0;
}

void usart0_transmit(uint8_t data) {
	// Wait until UDRE bit becomes 1 (Ready to transmit)
	while ( !(UCSR0A & 0b00100000) );
	UDR0 = data;
	// Wait until TXC bit becomes 1 (Transmit complete)
	while ( !(UCSR0A & 0b01000000) );
}

uint8_t* led_init() {
	volatile uint8_t* ddrB = 0x24;
	*ddrB |= 0b11111111;
	return 0x25;
}

int main(void)
{
	volatile uint8_t* leds = led_init();

	usart0_init();

	while (1)
	{
		uint8_t msg = usart0_receive();
		*leds = 1<<(msg & 0b00001111);
		usart0_transmit(msg);
	}
}
```

**Lab Question 2 - What happens if the baudrate setting in YAT is changed?**

The echoed characters become distorted, seemingly the microcontroller is sending back the wrong characters. It is in fact sending the right characters, just at the wrong rate.