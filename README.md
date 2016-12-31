# WiFi-Based-Greenhouse-parameter-controller-IoT-


##Developed an Internet of Things (IoT) concept to develop cost-effective automatic greenhouse weather control system using Wi-Fi.


1) WIFI based Greenhouse parameters Monitoring and Control project is used to measure the parameters like Temperature, Humidity, Light and Soil Moisture and transmits these parameters using WIFI to various stations.

2) Temperature, Humidity, Light and Soil Moisture are sensed by their respective sensors.

3) Using a WIFI module, the Microcontroller transmits the sensor data which can be received using a Laptop or an Android application.



## Source Code

*#*include*<reg51.h>*

*#*define LCD_DATA	P1

*#*define ADC_DATA	P0

sbit lights=P2^4;				
sbit soilmtr=P2^5;				//////////// Pins to Control Relays and Motors 
sbit hummtr=P3^3;
sbit fan=P3^2;

sbit ssen=P3^4;					//////////Soil Moisture Sensor Input Pin

sbit ADC_START=P2^0;
sbit ADC_EOC=P2^1;
sbit ADC_ALE=P2^2;					//////////ADC Control Bits
sbit ADC_OE=P2^3;

sbit CS_A=P3^5;	
sbit CS_B=P3^6;				////////////////ADC Selection Bits
sbit CS_C=P3^7;

void adc_read(char ch);
void delay(unsigned int val);
float adc_value;
int Voltage,Temperature,Humidity,Light,Light_temp;
unsigned char Serial_Data,LCD_CLEAR=0x01;	
unsigned char ptr=1;
bit LCD_Cmd=0,LCD_Data=1;
void Lcd_Init();
void Lcd_Data_Chr(bit RS ,unsigned char line,unsigned char position,unsigned char temp1) ;
void Lcd_Data_Str(unsigned char line,unsigned char position,unsigned char *temp);
void Lcd_Wr(unsigned char r);

/////////////////LCD Control  Pins//////////////////////////////////////
sbit RS=P2^5;
sbit RW=P2^6;
sbit EN=P2^7;
/*********************************************************************
                       SERIAL FUNCTIONS
**********************************************************************/
void Init_Serial();
void Serial_TX_Chr(unsigned char);
void Serial_TX_Str(char *temp); 
unsigned char W_rx,rx,rx1;
int Light_disp;
////////////////////////////////Main Function///////////////////////////////////////////
main()
{
 int c;
Lcd_Init(); 
Init_Serial();
while(1)
{ 
		for(c=0;c<4;c++)
		{
			adc_read(c);
			delay(10);
		}
////////////////////////////////////////////////////Soil Moisture Function////////////////////////////////////////////
		if(ssen==0)
		{
			Lcd_Data_Str(2,14,"S:W");
			delay(100);
			 soilmtr=0;
			delay(100);
			 Serial_TX_Str("Soil:WET            WaterPump:OFF\r\n"); 
			 Serial_TX_Str("\n");
			   Serial_TX_Str("AUTOMATIC CONTROL OF GREENHOUSE PARAMETERS\r\n");			
			   Serial_TX_Str("\n");
			}
			else if(ssen==1)
			{ 
			Lcd_Data_Str(2,14,"S:D");
			delay(100);
			 soilmtr=1;
			delay(100);
			Serial_TX_Str("Soil:DRY            WaterPump:ON\r\n"); 
			 Serial_TX_Str("\n");
			 Serial_TX_Str("AUTOMATIC CONTROL OF GREENHOUSE PARAMETERS\r\n"); 
			 Serial_TX_Str("\n");

}
}
 }

/////////////////////////////////////////////To read ADC Channel///////////////////////////////////////////
void adc_read(char ch)
{
	if(ch==0)
		CS_A=CS_B=CS_C=0;
	else if(ch==1)
	{
		CS_A=CS_B=CS_C=0;
		CS_A=1;
	}
	else if(ch==2)
	{
		CS_A=CS_B=CS_C=0;
		CS_B=1;
	}
	else if(ch==3)
	{
		CS_A=CS_B=CS_C=0;
		CS_A=CS_B=1;
	}
	else if(ch==4)
	{
		CS_A=CS_B=0;
		CS_C=1;
	}
	else if(ch==5)
	{
		CS_B=0;
		CS_C=CS_A=1;
	}
	else if(ch==6)
	{
		CS_A=0;
		CS_C=CS_B=1;
	}
	else if(ch==7)
	{
		CS_A=CS_B=CS_C=1;
	}
	ADC_ALE=1;
	delay(5);
	ADC_START=1;
	delay(5);
	ADC_START=0;
	ADC_ALE=0;
	delay(10);
	ADC_OE=1;
	delay(5);
	adc_value=ADC_DATA*0.01953125;
	ADC_OE=0;
///////////////////////////////Temperature Function///////////////////////////////////////////////////////////////
	if(ch==0)                   
	{
		Temperature = adc_value*100;
        Lcd_Data_Str(1,1,"Temp:");
		Temperature=Temperature%100;
		Lcd_Data_Chr( 1,1,6,(Temperature/10)+48);
		Lcd_Data_Chr(1,1,7,(Temperature%10)+48);
		Lcd_Data_Str(1,8,"C");
        Serial_TX_Str("\nTemprature:");
		Serial_TX_Chr((Temperature/100)+48);
		Temperature=Temperature%100;
		Serial_TX_Chr( (Temperature/10)+48);
	    Serial_TX_Chr((Temperature%10)+48);
		Serial_TX_Str("C    ");
		if (Temperature<=45)
	{
		fan=0;
		Serial_TX_Str(" Fan:OFF\r\n");
	}
	else if (Temperature>45)
	{
		fan=1;
		Serial_TX_Str(" Fan:ON\r\n");
	}
	}
///////////////////////////////////////////////////Light  Function////////////////////////////////////////////////////////////
	else if(ch==2)           
	{
	  Light_temp = (2500/adc_value-500);
		Light = Light_temp/2.2;
		Light_disp=Light;
		Lcd_Data_Str(2,1,"L:");
		Serial_TX_Str("Light:");
		Lcd_Data_Chr(1,2,3,(Light/1000)+48);
		Serial_TX_Chr( (Light/1000)+48);
		Light=Light%1000;
		Lcd_Data_Chr(1,2,4,(Light/100)+48);
		Serial_TX_Chr((Light/100)+48);
		Light=Light%100;
		Serial_TX_Chr((Light/10)+48);
		Serial_TX_Chr((Light%10)+48);
		Serial_TX_Str("LUX      ");
		Lcd_Data_Chr(1,2,5,(Light/10)+48);
		Lcd_Data_Chr(1,2,6,(Light%10)+48);
		Lcd_Data_Str(2,7,"LUX");
	if (Light_disp>20)
	{
			lights=0;
		   Serial_TX_Str(" Lights:OFF\r\n");
	}
	else if (Light_disp<=20)
	{
		lights=1;
	Serial_TX_Str(" Lights:ON\r\n");
	
	}
	}
////////////////////////////////////////////////// Humidity Function///////////////////////////////////////////////
else if(ch==3)           
	{
		 {
		Humidity = (adc_value/0.0330);
		Lcd_Data_Str(1,10,"Hum:");
		Lcd_Data_Chr(1,1,14,(Humidity/10)+48);
		Lcd_Data_Chr(1,1,15,(Humidity%10)+48);
		Lcd_Data_Str(1,16,"%");
		Serial_TX_Str("Humidity:");
		Serial_TX_Chr((Humidity/10)+48);
		Serial_TX_Chr((Humidity%10)+48);
		Serial_TX_Str("%       ");
	if (Humidity>35)
	{
		
			hummtr=0;
		   Serial_TX_Str(" Fogger:OFF\r\n");
	}
	else if (Humidity<=35)
	{
		hummtr=1;
	Serial_TX_Str(" Fogger:ON\r\n");
	}
	}
	}
	}
//////////////////////////////////////////To Initialize LCD/////////////////////////////////////////////////////
void Lcd_Init()			
  {     
unsigned char LCD_2_LINE=0x38;
unsigned char LCD_CLEAR=0X01;
unsigned char DISPLAY_ON=0X0E;
unsigned char LCD_CURSOR_OFF=0x0C;						
		Lcd_Data_Chr(0,0,0, LCD_2_LINE);		
		Lcd_Data_Chr(0,0,0, DISPLAY_ON);
		Lcd_Data_Chr(0,0,0, LCD_CURSOR_OFF);
		Lcd_Data_Chr(0,0,0, LCD_CLEAR);
   }						
/////////////////////////////////////To display Char in LCD //////////////////////////////////////
void Lcd_Data_Chr(bit RS ,unsigned char line,unsigned char position,unsigned char temp1)
{
if(RS==0)
{
LCD_DATA=temp1;
Lcd_Wr(LCD_Cmd);
}
if(RS==1)
{
   if(line==1)
    {
    LCD_DATA=0x7f+position;
	Lcd_Wr(LCD_Cmd);
    }
   if(line==2)
    {
    LCD_DATA=0xbf+position;
    Lcd_Wr(LCD_Cmd);
    }
	LCD_DATA=temp1;
	Lcd_Wr(LCD_Data);		
}		
}
///////////////////////////////////////  //To display String in LCD/////////////////////////////////////////////////
void Lcd_Data_Str(unsigned char line,unsigned char position,unsigned char *temp)                                                                           
{
unsigned char p;
if(line==1)
{
p=0x7f+position;
Lcd_Data_Chr(0,0,0,p);
}
if(line==2)
{
p=0xbf+position;
Lcd_Data_Chr(0,0,0,p);
}
while(*temp!='\0')
	{			
	LCD_DATA=*temp;
	Lcd_Wr(LCD_Data); 
    temp++;
}
}
void Lcd_Wr(unsigned char r)           
{
RS=r;
EN=1;
delay(1);
EN=0;
}
/////////////////////////////////////////////Intitialize Serial Communication///////////////////////////
void Init_Serial()                  
{
TMOD=0X20;
TH1=0xfd;			
SCON=0X50;
TR1=1;
}			
////////////////////////////To Transmit Char/////////////////////////////////////////////////////////////////
void Serial_TX_Chr(unsigned char recv)                
{
SBUF=recv;
while(TI==0);
TI=0;
}
////////////////////////////To Transmit String//////////////////////////////////////////////////////////////////////////////
void Serial_TX_Str(char *temp)         
{
while(*temp!='\0')
{
SBUF=*temp;
while(TI==0);
TI=0;
temp++;
}
}
////////////////////////////////////////Delay Function////////////////////////////////////////////////////////
void delay(unsigned int time)                  
{
unsigned int i,j;
for(i=0;i<time;i++)
for(j=0;j<1200;j++);

}
