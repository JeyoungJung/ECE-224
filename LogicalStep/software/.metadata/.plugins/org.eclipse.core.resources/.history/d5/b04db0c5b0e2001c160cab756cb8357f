/*
 * "Hello World" example.
 *
 * This example prints 'Hello from Nios II' to the STDOUT stream. It runs on
 * the Nios II 'standard', 'full_featured', 'fast', and 'low_cost' example
 * designs. It runs with or without the MicroC/OS-II RTOS and requires a STDOUT
 * device in your system's hardware.
 * The memory footprint of this hosted application is ~69 kbytes by default
 * using the standard reference design.
 *
 * For a reduced footprint version of this template, and an explanation of how
 * to reduce the memory footprint for a given application, see the
 * "small_hello_world" template.
 *
 */

#include <stdio.h>
#include "system.h"
#include "altera_avalon_pio_regs.h"
#include "sys/alt_irq.h"

volatile int flag;
int mode;

int background()
{
	int j;
	int x = 0;
	int grainsize = 4;
	int g_taskProcessed = 0;
	for (j = 0; j < grainsize; j++)
	{
		g_taskProcessed++;
	}
	return x;
}

static void stim_in_isr(void *context, alt_u32 id);
void tight_polling(void);
void interrupt_behaviour(void);

int main()
{
	printf("NIOOOOS II!\n");
	mode = IORD(SWITCH_PIO_BASE, 0);
	// Indicates we are doing tight polling
	if (mode & 1)
	{
		printf("Tight Polling selected\nPlease press PB0 to continue\n");
		while ((IORD(BUTTON_PIO_BASE, 0) & 1))
		{
		}
		tight_polling();
	}
	else
	{
		printf("Interrupt selected\n Please press PB0 to continue\n");
		while ((IORD(BUTTON_PIO_BASE, 0) & 1))
		{
		}
		interrupt_behaviour();
	}

	// query the missed pulses and average latency
	// and set egm enable to low
	return 0;
}

void tight_polling()
{
	// Turn off interrupt mode
	IOWR(STIMULUS_IN_BASE, 2, 0);

	// 16 bit values for period/duty cycle
	unsigned short period;

	// ensures each cycle only processed once
	for (period = 2; period < 1500; period += 2)
	{
		IOWR(LED_PIO_BASE, 0, 2);
		IOWR(LED_PIO_BASE, 0, 0);
		// Setup and enable EGM
		// Wait for EGM to become disabled
		// Disable EGM for config
		IOWR(EGM_BASE, 0, 0);
		// Set period
		IOWR(EGM_BASE, 2, period);
		// Set duty cycle
		IOWR(EGM_BASE, 3, period / 2);
		// Set enable
		unsigned int bt_count = 0;
		unsigned short num_tasks = 0;
		unsigned short bt = 0;

		IOWR(EGM_BASE, 0, 1);
		IOWR(RESPONSE_OUT_BASE, 0, 1);
		IOWR(RESPONSE_OUT_BASE, 0, 0);
		while(IORD(STIMULUS_IN_BASE, 0)&& IORD(EGM_BASE, 1)) { background(); num_tasks++; }
		while(!IORD(STIMULUS_IN_BASE, 0) && IORD(EGM_BASE, 1)) { background(); num_tasks++; }
		IOWR(RESPONSE_OUT_BASE, 0, 1);
		IOWR(RESPONSE_OUT_BASE, 0, 0);
		while(IORD(STIMULUS_IN_BASE, 0) && IORD(EGM_BASE, 1)) {}
		while(!IORD(STIMULUS_IN_BASE, 0) && IORD(EGM_BASE, 1)) {}

		while (IORD(EGM_BASE, 1)) {
			IOWR(RESPONSE_OUT_BASE, 0, 1);
			IOWR(RESPONSE_OUT_BASE, 0, 0);
			for (bt = 0; bt < num_tasks - 1; bt++) {
				background();
				bt_count++;
			}
			while(IORD(STIMULUS_IN_BASE, 0) && IORD(EGM_BASE, 1)) {}
			while(!IORD(STIMULUS_IN_BASE, 0) && IORD(EGM_BASE, 1)) {}
		}

		// Disable RESPONSE_OUT_BASE

//		break;
//
//		while (IORD(EGM_BASE, 1))
//		{
//			if (first < 2) {
//				if (curr && !prev) { // Rising edge
//					IOWR(RESPONSE_OUT_BASE, 0, 1);
//					// Disable RESPONSE_OUT_BASE
//					IOWR(RESPONSE_OUT_BASE, 0, 0);
//					first++;
//					curr = IORD(STIMULUS_IN_BASE, 0);
//					prev = curr;
//				} else {
//					background();
//					num_tasks++;
//				}
//			} else {
//				for (bt = 0; bt < num_tasks - 1; bt++) {
//					background();
//					bt_count++;
//				}
//				IOWR(RESPONSE_OUT_BASE, 0, 1);
//				// Disable RESPONSE_OUT_BASE
//				IOWR(RESPONSE_OUT_BASE, 0, 0);
//			}
//		}

//		int second = 0;
//		while (IORD(EGM_BASE, 1)) {
//			IOWR(LED_PIO_BASE, 0, 2);
//			IOWR(LED_PIO_BASE, 0, 0);
//
//			if (IORD(STIMULUS_IN_BASE, 0))
//			{
//				IOWR(RESPONSE_OUT_BASE, 0, 1);
//				IOWR(RESPONSE_OUT_BASE, 0, 0);
//			}
//			if (first)
//			{
//				// While STIMULUS_IN_BASE is HIGH
//				while (IORD(STIMULUS_IN_BASE, 0) && IORD(EGM_BASE, 1))
//				{
//					background();
//					num_tasks++;
//				}
//				// While STIMULUS_IN_BASE is LOW
//				while (IORD(STIMULUS_IN_BASE, 0) != 0x01 && IORD(EGM_BASE, 1))
//				{
//					background();
//					num_tasks++;
//				}
//				bt_count = num_tasks;
//				first = 0;
//				second = 1;
//			} else if (second) {
//				second = 0; // Wait for second cycle to execute before collecting data
//				while (IORD(STIMULUS_IN_BASE, 0) && IORD(EGM_BASE, 1)) {}
//			}
//			else
//			{
//				for (bt = 0; bt < 1; bt++)
//				{
//
//					background();
//					bt_count++;
//				}
//			}
//		}

		unsigned short avg_lat = IORD(EGM_BASE, 4);
		unsigned short missed = IORD(EGM_BASE, 5);
		unsigned short multi = IORD(EGM_BASE, 6);
		//printf("num tasks: %d\n", num_tasks);
		printf("period %hu, duty %hu, bt %u, avg lat %hu, missed %hu, multi %hu\n", period, period / 2, bt_count, avg_lat, missed, multi);
		bt_count = 0;
		// Get test data and disable egm
	}
	printf("exit loop\n");
}

void interrupt_behaviour()
{
	// Initialize Interrupt for stim in
	alt_irq_register(STIMULUS_IN_IRQ, NULL, stim_in_isr);
	IOWR(STIMULUS_IN_BASE, 2, 0x1);

	// 16 bit values for period/duty cycle
	unsigned short period;

	for (period = 2; period < 5000; period += 2)
	{
		unsigned int bt_count = 0;
		// Setup and enable EGM
		// Wait for EGM to become disabled

		// Set period
		IOWR(EGM_BASE, 2, period);
		// Set duty cycle
		IOWR(EGM_BASE, 3, period / 2);
		// Set enable
		IOWR(EGM_BASE, 0, 1);

		// While EGM_BASE is busy
		while (IORD(EGM_BASE, 1))
		{
			background();
			bt_count++;
		}
		unsigned short avg_lat = IORD(EGM_BASE, 4);
		unsigned short missed = IORD(EGM_BASE, 5);
		unsigned short multi = IORD(EGM_BASE, 6);
		printf("period %hu, duty %hu, bt %u, avg lat %hu, missed %hu, multi %hu\n", period, period / 2, bt_count, avg_lat, missed, multi);
		// Disable EGM for config
		IOWR(EGM_BASE, 0, 0);
	}
}

static void stim_in_isr(void *context, alt_u32 id)
{
	// Enable RESPONSE_OUT_BASE
	IOWR(RESPONSE_OUT_BASE, 0, 1);
	// Disable RESPONSE_OUT_BASE
	IOWR(RESPONSE_OUT_BASE, 0, 0);
	// Clear STIMULUS_IN_BASE
	IOWR(STIMULUS_IN_BASE, 3, 0x0);
}
