//# RTC_CLOCK_I2C
//REAL TIME CLOCK 
// PIC16F877A Configuration Bit Settings

// 'C' source line config statements

// CONFIG
#pragma config FOSC = HS        // Oscillator Selection bits (HS oscillator)
#pragma config WDTE = OFF       // Watchdog Timer Enable bit (WDT disabled)
#pragma config PWRTE = OFF      // Power-up Timer Enable bit (PWRT disabled)
#pragma config BOREN = OFF      // Brown-out Reset Enable bit (BOR disabled)
#pragma config LVP = OFF        // Low-Voltage (Single-Supply) In-Circuit Serial Programming Enable bit (RB3 is digital I/O, HV on MCLR must be used for programming)
#pragma config CPD = OFF        // Data EEPROM Memory Code Protection bit (Data EEPROM code protection off)
#pragma config WRT = OFF        // Flash Program Memory Write Enable bits (Write protection off; all program memory may be written to by EECON control)
#pragma config CP = OFF         // Flash Program Memory Code Protection bit (Code protection off)

// #pragma config statements should precede project file includes.
// Use project enums instead of #define for ON and OFF.

#include <xc.h>



#include <htc.h>
#define _XTAL_FREQ 10000000

__CONFIG (0X3F3A);

void I2C_Master_Init(const unsigned long c);
void I2C_Master_Wait();
void I2C_Master_Start();
void I2C_Master_RepeatedStart();
void I2C_Master_Stop();
void I2C_Master_Write(unsigned char d);
unsigned char I2C_Master_Read(unsigned char a);
int dec_bcd(int data);
int bcd_dec(int data);

int main()
{
    unsigned char read_value = 0, sec = 0, value = 0;
    TRISB = 0x12;                 //PORTB as input
    TRISD = 0x00;                 //PORTD as output
    PORTD = 0x00;                 //All LEDs OFF
    I2C_Master_Init(10000);       //Initialize I2C Master with 100KHz clock

    I2C_Master_Start();
    I2C_Master_Write(0xd0);       //slave address rtc=0xd0 +write bit
    I2C_Master_Write(0x01);       //location address sec
    
    value = dec_bcd(12);
    I2C_Master_Write(value);      //dec
    I2C_Master_Stop();

    while(1)
    {
        //particular address sec 0x00 min 0x01
        I2C_Master_Start();
        I2C_Master_Write(0xd0);   //slave address rtc=0xd0 +write bit
        I2C_Master_Write(0x01);   // SECONDS LOCATION
        I2C_Master_RepeatedStart();
        I2C_Master_Write(0xd1);   //slave address rtc=0xd1 +read bit
        read_value = I2C_Master_Read(0);
        sec = bcd_dec(read_value);
        I2C_Master_Stop();
        __delay_ms(500);
    }
}

void I2C_Master_Init(const unsigned long c)
{
    SSPCON = 0b00101000;
    SSPCON2 = 0;
    SSPADD = 0x3f;
    SSPSTAT = 0;
    TRISC3 = 1;  //Setting as input as given in datasheet
    TRISC4 = 1;  //Setting as input as given in datasheet
}

void I2C_Master_Wait()
{
    while ((SSPSTAT & 0x04) || (SSPCON2 & 0x1F));
}

void I2C_Master_Start()
{
    I2C_Master_Wait();
    SEN = 1;
}

void I2C_Master_RepeatedStart()
{
    I2C_Master_Wait();
    RSEN = 1;
}

void I2C_Master_Stop()
{
    I2C_Master_Wait();
    PEN = 1;
}

void I2C_Master_Write(unsigned char d)
{
    I2C_Master_Wait();
    SSPBUF = d;
}

unsigned char I2C_Master_Read(unsigned char a)
{
    unsigned char temp;
    I2C_Master_Wait();
    RCEN = 1;
    I2C_Master_Wait();
    temp = SSPBUF;
    I2C_Master_Wait();
    ACKDT = (a) ? 0 : 1;
    ACKEN = 1;
    RCEN = 0;
    return temp;
}

int dec_bcd(int data)
{
    int result;
    result = (((data / 10) << 4) + (data % 10));// FORMULA FOR WRITE THE DATA 
    return result;
}

int bcd_dec(int data)
{
    int result;
    result = (((data >> 4) * 10) + (data & 0x0F));// FORMULA FOR READ THE DATA 
    return result;
}
