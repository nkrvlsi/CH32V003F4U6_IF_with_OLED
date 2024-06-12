# Project: CH32V003F4U6_Interface_with_OLED  

# Objective:  
This project demonstrates a basic interface between the VSD Squadron Mini Board and OLED. Here we are using the CH32V003F4U6 microcontroller to interface with a 128x64 OLED display using the I2C protocol.  

## 1. Components Needed  

1. CH32V003F4U6 Microcontroller  
2. 128x64 OLED Display (I2C)
      - OLED stands for **Organic Light Emitting Diodes**. Unlike traditional LED displays which require a backlight, OLED displays consist of individual **organic molecules** that light up when an electric current is applied. This leads to displays that are thinner, lighter, and offer better contrast ratios compared to LCDs.
      - An OLED is a type of diode that consists of an organic compound that emits light when current flows through it.
4. Power Supply  
5. Resistors, Capacitors, and Wires  
6. Breadboard and Connectors  

## 2. Circuit Design  

1. **OLED to CH32V003F4U6:** Connect the I2C pins of the OLED display to the corresponding I2C pins of the CH32V003F4U6.  
   - **SCL** (Serial Clock Line): Connect to PD6 (I2C1_SCL)
   - **SDA** (Serial Data Line): Connect to PD7 (I2C1_SDA)
   - **VCC**: Connect to 3.3V or 5V depending on your OLED module.
   - **GND**: Connect to GND.  

## 3. Software Implementation  

1. **Initialize the I2C and OLED Display:** Set up the I2C interface to communicate with the OLED display.  
2. **Display Data on OLED:** Write functions to send data to the OLED display and show text or graphics.  

## 4. C Source Code  

Here's an implementation in C:  

```c
#include <ch32v00x.h>
#include <stdio.h>
//#include <oled96.h>
#include <debug.h>

#define I2C_SCL_PIN GPIO_Pin_2        //PC2 pin 
#define I2C_SDA_PIN GPIO_Pin_1        //PC1 pin
#define OLED_ADDRESS 0x3C //0x7A //0x78 //0x3C         // Common I2C address for OLED displays

void I2C1_Init(void) {
    GPIO_InitTypeDef GPIO_InitStructure;        //structure variable GPIO_InitStructure of type GPIO_InitTypeDef, which is used for GPIO configuration.
    I2C_InitTypeDef I2C_InitStructure;          //structure variable I2C_InitStructure of type I2C_InitTypeDef, which is used for I2C configuration.

    //enable clocks 
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);   //to enable clock for Port C
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1, ENABLE);    //to enable clock for I2C1

    //GPIO Configuration
    GPIO_InitStructure.GPIO_Pin = I2C_SCL_PIN | I2C_SDA_PIN;    //GPIO_Pin: this is to define which Pin, Here we define multiple pins of single port GPIOC
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;             //GPIO_Mode: configure output type. 1. open drain (GPIO_Mode_Out_OD) or 1. push-pull (GPIO_Mode_Out_PP)  AF- Alternate Function OD- Open Drain  PP-Push Pull drive
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;           //GPIO_Speed: 3speeds: GPIO_Speed_10MHz, GPIO_Speed_2MHz, GPIO_Speed_50MHz. This basically configures the drive strength of the GPIO internally.
    GPIO_Init(GPIOC, &GPIO_InitStructure);

    //kk Step 2: Initialize the I2C1 peripheral
    I2C_DeInit(I2C1); // Reset I2C1 to default state

    //I2C Configuration
    I2C_InitStructure.I2C_ClockSpeed = 50000; //100000; //100KHz =100 * 100 , since 1Khz=100; 100kHz is standard mode
    I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;
    I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;  // 50% duty cycle
    I2C_InitStructure.I2C_OwnAddress1 = 0x00; //0x78;   //0x00;     // Own address (not used in master mode)
    I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;         // Enable acknowledge
    I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;   // 7-bit addressing mode
    
     // Initialize I2C1
    I2C_Init(I2C1, &I2C_InitStructure);
    // Enable I2C1
    I2C_Cmd(I2C1, ENABLE);
    // I2C_AcknowledgeConfig(I2C1, ENABLE);  //kk copied
}

void OLED_WriteCommand(uint8_t command) {
    // Generate I2C start condition
    I2C_GenerateSTART(I2C1, ENABLE);    //start bit
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT));

    // Send OLED address with write instruction //0-write bit;  1-RD bit;
    I2C_Send7bitAddress(I2C1, OLED_ADDRESS, I2C_Direction_Transmitter);
    //I2C_Send7bitAddress(I2C1, OLED_ADDRESS << 1, I2C_Direction_Transmitter);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED));

    // Send the Control byte (Co = 0, D/C# = 0 for command)
    I2C_SendData(I2C1, 0x00);       
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    // Send the command byte
    I2C_SendData(I2C1, command);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    // Generate I2C stop condition
    I2C_GenerateSTOP(I2C1, ENABLE);     //stop bit
}

void OLED_WriteData(uint8_t data) {
    I2C_GenerateSTART(I2C1, ENABLE);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT));

    I2C_Send7bitAddress(I2C1, OLED_ADDRESS, I2C_Direction_Transmitter);
    //I2C_Send7bitAddress(I2C1, OLED_ADDRESS << 1, I2C_Direction_Transmitter);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED));

    I2C_SendData(I2C1, 0x40);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    I2C_SendData(I2C1, data);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    I2C_GenerateSTOP(I2C1, ENABLE);
}

//kk modified below values by checking specs
void OLED_Init(void) {
    OLED_WriteCommand(0xAE); // Displady OFF         //start + 0xA0 (Device Address + RW Bit) + 8/16bit Data + Start + 0xA1 (Device Address + RW Bit) + Read Data + Stop.
    
    OLED_WriteCommand(0x81); // set Contrast Control
        OLED_WriteCommand(0xFF);    //contrast value

    OLED_WriteCommand(0xA5); // A5-  Entire Display ON, A4- Output follows RAM content.
    OLED_WriteCommand(0xA6); // A6- Normal display, A7- Inverse Display

    OLED_WriteCommand(0x21);    // Set Column Address
        OLED_WriteCommand(0x00); // set Low Column start Address 00~0F
        OLED_WriteCommand(0x10); // set High Column start Address 10~1F
    
    OLED_WriteCommand(0x20); // Set Memory Addressing Mode
        OLED_WriteCommand(0x00); // 0x00- Horizontal addressing mode 01- verticle addressing mode 10- page addressing mode
    
    OLED_WriteCommand(0x22);    //set page adress
        OLED_WriteCommand(0xB0); // Set Page Start Address. - for Page Addressing Mode  //Set GDDRAM Page Start Address(PAGE0~PAGE7) for Page Addressing Mode

    OLED_WriteCommand(0x40); // Start Line Address      // Set Display Start Line (40h~7Fh) 
        OLED_WriteCommand(0x00);    //set start address of display ram to 0. and map row-0 to com-0.

    OLED_WriteCommand(0xA0); // 0xA0-column address 0 is mapped to SEG0, 0xA1- column address 127 is mapped to SEG0

    OLED_WriteCommand(0xA8); // Multiplex Ratio
        OLED_WriteCommand(0x3F);    // 64Mux value - setting to max (reset value)

    OLED_WriteCommand(0xC0); // COM Output Scan Direction   // C0- normal mode (RESET_Value) Scan from COM0 to COM[N –1] , C8-remapped mode. Scan from COM[N-1] to COM0 

    OLED_WriteCommand(0xD3); // Display Offset      //Set vertical shift by COM from 0d~63d 
        OLED_WriteCommand(0x00);    //Set vertical shift by COM from 0d~63d. reset_value=0

    OLED_WriteCommand(0xDA); // COM Pins Hardware Configuration
        OLED_WriteCommand(0x12);    //A[4]=1b - Alternative COM pin configuration //A[4]=0b-Sequential COM pin configuration, A[5]=1b, Enable COM Left/Right remap //setting to reset_value

    OLED_WriteCommand(0xD5); // Display Clock Divide Ratio //lower 4 bits = A[3:0] = Define the divide ratio (D) of the display clocks (DCLK) Divide_ratio=A[3:0]+1, A[7:4] = Set the Oscillator Frequency Range:0000b~1111b note: Frequency increases as setting value increases.
        OLED_WriteCommand(0xF0);    

    OLED_WriteCommand(0xD9); // Pre-charge Period
        OLED_WriteCommand(0x22);        // reset value of above precharge period

    OLED_WriteCommand(0xDB); // VCOMH Deselect Level
        OLED_WriteCommand(0x20);    //0x20 - ~0.77xVcc

    OLED_WriteCommand(0x8D); // Charge Pump Setting //Set DC-DC enable
        OLED_WriteCommand(0x14);        //bit[2] - eanble charge pump

    OLED_WriteCommand(0xAF); // Display ON                      //0xAE: Set display OFF , 0xAF:  Set display ON
}


/* void OLED_Init(void) {
   
    OLED_WriteCommand(0xAE); // Display off

    OLED_WriteCommand(0x20); // Set Memory Addressing Mode
    OLED_WriteCommand(0x00); // 00,Horizontal Addressing Mode; 01,Vertical Addressing Mode; 10,Page Addressing Mode (RESET); 11,Invalid

    OLED_WriteCommand(0x40); // Set start line address
    OLED_WriteCommand(0x81); // Set contrast control register
    OLED_WriteCommand(0xFF); // Max contrast

    OLED_WriteCommand(0xA0); // Set segment re-map 0 to 127
    OLED_WriteCommand(0xA7); // Set normal display
    OLED_WriteCommand(0xA8); // Set multiplex ratio(1 to 64)
    OLED_WriteCommand(0x3F); // 1/64 duty

    
    OLED_WriteCommand(0xD3); // Set display offset
    OLED_WriteCommand(0x00); // No offset

    OLED_WriteCommand(0xD5); // Set display clock divide ratio/oscillator frequency
    OLED_WriteCommand(0x10); // Set divide ratio

    OLED_WriteCommand(0xD9); // Set pre-charge period
    OLED_WriteCommand(0x22);
    //0xDA, 0x12, 0xDB, 0x20, 0x8D, 0x14, 0x2E, 0xAF
    OLED_WriteCommand(0xDA); // Set com pins hardware configuration
    OLED_WriteCommand(0x12);

    OLED_WriteCommand(0xDB); // Set vcomh
    OLED_WriteCommand(0x20); // 0x20,0.77xVcc

    OLED_WriteCommand(0x8D); // Set DC-DC enable
    OLED_WriteCommand(0x14);
    OLED_WriteCommand(0x2E);
    OLED_WriteCommand(0xAF); // Turn on OLED panel
	}
    */

void OLED_SetCursor(uint8_t page, uint8_t column) {
    OLED_WriteCommand(0xB0 + page);                         // Set page start address (0xB0 - 0xB7), //OLED_WriteCommand(0xB0 | (page & 0x07)); //select page_no
    OLED_WriteCommand(0x00 | (column & 0x0F));              // set lower column start column address between 0x00 to 0x0f // Set column address lower nibble (0x00 - 0x0F)
    OLED_WriteCommand(0x10 | ((column >> 4) & 0x0F));       // set upper column start column address between 0x10 to 0x1f  // Set column address higher nibble (0x10 - 0x1F)
}

void OLED_Clear(void) {
    for (uint8_t page = 0; page < 8; page++) {
        OLED_SetCursor(page, 0);
        for (uint8_t col = 0; col < 128; col++) {
            OLED_WriteData(0x00);
        }
    }
}

void OLED_DisplayString(char* str) {
    while (*str) {
        OLED_WriteData(*str++);
    }
}

void setup() {
    I2C1_Init();
    OLED_Init();
    //Delay_Ms(1000);
    //OLED_Clear();
}

void loop() {

    OLED_SetCursor(0, 0);       // Set cursor to the top-left corner
        //OLED display message
    OLED_DisplayString("HELLO");
    Delay_Ms(1000);                         // creates no of milliseconds delay
    OLED_Clear();
    Delay_Ms(1000);
}

int main(void) {
    SystemInit();
    SystemCoreClockUpdate();
    setup();

    while (1) {
        loop();
    }
}
```

## 5. Explanation  
### 5.1 I2C Initialization:  

1. Configures GPIO pins PD6 and PD7 for I2C communication.
2. Initializes I2C1 with a clock speed of 100kHz.

### 5.2 OLED Initialization:  

3. Sends a sequence of commands to initialize the OLED display (based on the SSD1306 controller).

### 5.3 OLED_WriteCommand:  

4. Sends a command byte to the OLED display over I2C.

### 5.4 OLED_WriteData:

5. Sends a data byte to the OLED display over I2C.

### 5.5 OLED_SetCursor:  

6. Sets the cursor position on the OLED display.

### 5.6 OLED_Clear:  

7. Clears the OLED display by writing 0x00 to all pixels.

### 5.7 OLED_DisplayString:  

8. Displays a string on the OLED screen by sending each character as data bytes.

### 5.8 setup:  

9. Initializes the I2C interface and the OLED display.
10. Clears the OLED display.
 
### 5.9 loop:

11. Displays a string "Hello, OLED!" on the OLED screen, waits for a second, clears the screen, and repeats.

## Notes
- **Font Data:** In this example, the `OLED_DisplayString function` directly sends character data, which assumes the OLED can interpret ASCII values. For a more complete implementation, include font data to convert characters to bitmaps.
- **Delays:** Ensure you have appropriate delay functions (`delay` and `delayMicroseconds`). If not available, implement them using timer interrupts.
- **OLED Address:** The `OLED_ADDRESS` might differ based on your OLED module. Check your module’s datasheet for the correct address.
- **Optimizations:** For more efficient display updates, consider implementing a display buffer in RAM and sending the entire buffer to the OLED at once.

