# MEC4126F-PRAC-5-NVDSAA001

// Description----------------------------------------------------------------|
/*
 *
 */
// DEFINES AND INCLUDES-------------------------------------------------------|

#define STM32F051                                                  //COMPULSORY
#include <stdio.h>
#include "stm32f0xx.h"											   //COMPULSORY
#include "lcd_stm32f0.h"

#define SW1 GPIO_IDR_1
#define SW2 GPIO_IDR_2

#define TRUE 1
#define FALSE 0


// GLOBAL VARIABLES ----------------------------------------------------------|
typedef uint8_t;

int x;
int y;

uint8_t count;



// FUNCTION DECLARATIONS -----------------------------------------------------|

void main(void);                                                   //COMPULSORY
void display_on_LCD(uint8_t count);
void init_LEDs(void);
void display_on_LEDs(uint8_t count);
void init_switches(void);


// MAIN FUNCTION -------------------------------------------------------------|

void main(void)
{
	init_switches();                //initialises push buttons
	init_LEDs();                    //initialises LEDs to be used
	init_LCD();                     //initialises LCD to be used
	count = 0;

	while(1)

	{
		while (count>=0 && count<256){


		display_on_LEDs(count);
		display_on_LCD(count);

		if((GPIOA -> IDR &= SW1) == 0){    //if SW1 pressed

			count = count + 1;
			display_on_LEDs(count);
			display_on_LCD(count);

		}

		if((GPIOA -> IDR &= SW2) == 0){
			count = count -1;
			display_on_LEDs(count);
			display_on_LCD(count);
		}

		}
	}
}

// OTHER FUNCTIONS -----------------------------------------------------------|

void display_on_LCD(uint8_t count)
	{
		char buffer[80];
		sprintf(buffer,"%d",count);
		delay(100000);
		lcd_putstring(buffer);
		delay(100000);


}

void init_LEDs(void)
{
	RCC->AHBENR |= RCC_AHBENR_GPIOBEN;        //Enable clock for LEDs
	GPIOB -> MODER |= (GPIO_MODER_MODER0_0|   //Set PB0 output to 0
					   GPIO_MODER_MODER1_0|	  //Set PB1 output to 0
					   GPIO_MODER_MODER2_0|	  //Set PB2 output to 0
					   GPIO_MODER_MODER3_0|	  //Set PB3 output to 0
					   GPIO_MODER_MODER4_0|	  //Set PB4 output to 0
					   GPIO_MODER_MODER5_0|	  //Set PB5 output to 0
					   GPIO_MODER_MODER6_0|   //Set PB6 output to 0
					   GPIO_MODER_MODER7_0);  //Set PB7 output to 0

}

void display_on_LEDs(uint8_t count)
{
	GPIOB -> ODR = count;
}

void init_switches(void)
{
	RCC ->AHBENR |= RCC_AHBENR_GPIOAEN;   //Enable clock for push buttons

	GPIOA -> MODER &= ~ (GPIO_MODER_MODER1|
						 GPIO_MODER_MODER2); //Set pins A1 and A2 to GPIO inputs

	GPIOA -> PUPDR |= (GPIO_PUPDR_PUPDR1_0|
					  GPIO_PUPDR_PUPDR2_0); //Enable pull up resistors

}
