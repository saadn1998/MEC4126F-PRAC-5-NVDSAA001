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
#define SW3 GPIO_IDR_3

#define TRUE 1
#define FALSE 0


// GLOBAL VARIABLES ----------------------------------------------------------|
typedef uint8_t;

int x;
int y;
int num;

uint8_t count;



// FUNCTION DECLARATIONS -----------------------------------------------------|

void main(void);                                                   //COMPULSORY
void display_on_LCD(uint8_t count);
void init_LEDs(void);
void display_on_LEDs(uint8_t count);
void init_switches(void);
void init_external_interrupts(void);
int EXTI2_3_IRQHandler(void);

// MAIN FUNCTION -------------------------------------------------------------|

void main(void)
{
	init_switches();                //initialises push buttons
		init_LEDs();                    //initialises LEDs to be used
		init_LCD();                     //initialises LCD to be used
		init_external_interrupts();
		count = 0;
		num = 0;
		x = 0;

		while(1)

		{
			while (count>=0 && count<256){

				if (EXTI-> PR &= EXTI_PR_PR3){    //if button pushed was A3 (SW3)

					init_external_interrupts();
					x = x + 1;           //count for number of times SW3 pressed

					if (x%2 == 1 || x == 1){

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
		}
	}
// OTHER FUNCTIONS -----------------------------------------------------------|

void display_on_LCD(uint8_t count)
	{
		char buffer[80];
		sprintf(buffer,"%d",count);
		lcd_command(CLEAR);
		delay(100000);
		lcd_putstring(buffer);
		delay(100000);
		lcd_command(CLEAR);


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
						 GPIO_MODER_MODER2|          //Set pins A1 and A2 to GPIO inputs
	                     GPIO_MODER_MODER3);

	GPIOA -> PUPDR |= (GPIO_PUPDR_PUPDR1_0|
					  GPIO_PUPDR_PUPDR2_0|          //Enable pull up resistors
	                  GPIO_PUPDR_PUPDR3_0);

}

void init_external_interrupts(void){

	RCC -> APB2ENR |= RCC_APB2ENR_SYSCFGCOMPEN;        //enable clock for sysconfig
	SYSCFG -> EXTICR[0] |= SYSCFG_EXTICR1_EXTI3_PA;    //route PA to EXTI3
	SYSCFG -> EXTICR[0] |= SYSCFG_EXTICR1_EXTI3_PA;
	EXTI -> IMR |= EXTI_IMR_MR3;                       //unmask the interrupt on PA3
	EXTI -> FTSR |= EXTI_FTSR_TR3;                     //falling edge trigger
	NVIC_EnableIRQ(EXTI2_3_IRQn);                      //enable the EXTI0_3 interrupt on NVIC

}

int EXTI2_3_IRQHandler(void){

	if (EXTI-> PR &= EXTI_PR_PR3){    //if button pushed was A3 (SW3)

		x = x + 1;           //count for number of times SW3 pressed
		num = x%2;


		EXTI -> PR |= EXTI_PR_PR3;
		display_on_LCD(x);
		return num;
	}
}
