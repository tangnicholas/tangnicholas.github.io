---
layout: page
title: Heart Rate Monitor
subtitle: Using a EFM8 micrcontroller and photoelectric components to measure heartrate
---

I will only include the main code below; the full C file is [here](https://github.com/tangnicholas/Assembly-C-Projects/tree/master/Labs)

![hrm_1](assets/img/hrm_1.jpg)

```c
void main (void) 
{
	float hr;
	float period;
	float heartrate;
	char sheartrate[10]; //string to send numbers to LCD
	float heartrate_list[NUM_AVE]; //list of heart rates to average
	float sum = 0; //sum of heart rates
	int i; //index for averaging
	
	LCD_4BIT(); //CONFIGURE LCD
	TIMER0_Init();

	waitms(500); // Give PuTTY a chance to start.
	printf("\x1b[2J"); // Clear screen using ANSI escape sequence.
	
	printf ("EFM8 Period measurement at pin P0.1 using Timer 0.\n"
	        "File: %s\n"
	        "Compiled: %s, %s\n\n",
	        __FILE__, __DATE__, __TIME__);

   while (1)
    {
    	// Reset the counter
		TL0=0; 
		TH0=0;
		TF0=0;
		overflow_count=0;
	
		TR0=1; // Start the timer
		//LCDprint("Heart Rate:    ", 1, 13); //print string on LCD 
		while(P0_1!=0) // Wait for the signal to be zero
		{
			if(TF0==1) // Did the 16-bit timer overflow?
			{
				TF0=0;
				overflow_count++;
			}
		}
	//	LCDprint("Heart Rate:   ", 1, 13); //print string on LCD 
		while(P0_1!=1) // Wait for the signal to be one
		{
			if(TF0==1) // Did the 16-bit timer overflow?
			{
				TF0=0;
				overflow_count++;
			}
			
		}
		
		TR0=0; // Stop timer 0, the 24-bit number [overflow_count-TH0-TL0] has the period!
	 	period=(overflow_count*65536.0+TH0*256.0+TL0)*(12.0/SYSCLK);
		heartrate=60/period;  //calculate heart rate
		
		//shift values over in the array
		for( i = 1; i < NUM_AVE; i++ ) {
			heartrate_list[i] = heartrate_list[i-1]; 
		}
		heartrate_list[0] = heartrate; //store current heart rate after shifting
		
		//compute sum for average
		for( i = 0; i < NUM_AVE; i++ ){
			sum += heartrate_list[i];
		}
	
		//if we have enough values to average over, print
		//if( heartrate_list[NUM_AVE - 1] != 0  ) {
			//LCDprint("Heart Rate:", 1, 1); //print string on LCD 
			
			//if heart rate > 300, ignore the value
			//if( sum/NUM_AVE > 300 ) 
			//	LCDprint( "TOO BIG", 2, 1); //print TOO BIG if heartrate unreasonably high
			//else {
				sprintf(sheartrate , "HR %.0f BPM", (sum/NUM_AVE)-10 ); //Convert heart rate to string
				LCDprint(sheartrate, 1, 1); //print string on LCD 
			//}
			
			hr = (sum/NUM_AVE)-10; 
			if (hr <= 30.0)
		    	LCDprint("YOU'RE DEAD LOL" , 2, 1); 
		    else if (hr > 30.0 && hr < 49.0)
		       	LCDprint("NOT RIGHT" , 2, 1);
		    else if (hr >= 49.0  && hr < 54.0)
		    	LCDprint("ATHLETIC" , 2, 1);
		    else if (hr >= 54.0 && hr < 61.0)
		       	LCDprint("EXCELLENT" , 2, 1);
		    else if (hr >= 61.0 && hr < 70.0)
		    	LCDprint("GREAT" , 2, 1);
		    else if (hr >= 70.0 && hr < 81.0)
		    	LCDprint("MEH. AVG" , 2, 1);
		    else if (hr >= 85.0)
		    	LCDprint("TERRRRRRIBLE" , 2, 1);
		    else LCDprint("<3 <3 <3", 2, 1); 

	if ( hr >= 54.0 && hr <85.0){ 
		P1_2=1;
		P1_4=0;
	} 
	else {
		P1_2=0;
		P1_4=1;
		}
	

		
	//}
		// Send the period to the serial port
		printf( "%f\r\n", hr);
		
		sum = 0; //clear the sum
    }
    
}
```

![hrm_2](assets/img/hrm_2.jpg)
