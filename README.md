# Project: CH32V003F4U6_Interface_with_OLED  

# Objective:  
This project demonstrates a basic interface between the VSD Squadron Mini Board and OLED. Here we are using the CH32V003F4U6 microcontroller to interface with a 128x64 OLED display using the I2C protocol.  

## 1. Components Needed  
1. CH32V003F4U6 Microcontroller  
2. 128x64 OLED Display (I2C)  
3. Power Supply  
4. Resistors, Capacitors, and Wires  
5. Breadboard and Connectors  

## 2. Circuit Design  
1. OLED to CH32V003F4U6: Connect the I2C pins of the OLED display to the corresponding I2C pins of the CH32V003F4U6.  
 - SCL (Serial Clock Line): Connect to PB6 (I2C1_SCL)  
 - SDA (Serial Data Line): Connect to PB7 (I2C1_SDA)  
 - VCC: Connect to 3.3V or 5V depending on your OLED module.  
 - GND: Connect to GND.  

## 3. Software Implementation  
1. Initialize the I2C and OLED Display: Set up the I2C interface to communicate with the OLED display.  
2. Display Data on OLED: Write functions to send data to the OLED display and show text or graphics.  

## 4. Example Code  
Here's an example implementation in C:  

```c
#include <ch32v00x.h>
#include <stdio.h>

#define I2C_SCL_PIN GPIO_Pin_6
#define I2C_SDA_PIN GPIO_Pin_7
#define OLED_ADDRESS 0x3C // Common I2C address for OLED displays

void I2C1_Init(void) {
    GPIO_InitTypeDef GPIO_InitStructure;
    I2C_InitTypeDef I2C_InitStructure;

    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOD, ENABLE);   //to enable clock for Port D
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C1, ENABLE);    

    GPIO_InitStructure.GPIO_Pin = I2C_SCL_PIN | I2C_SDA_PIN;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOD, &GPIO_InitStructure);

    I2C_InitStructure.I2C_ClockSpeed = 100000;
    I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;
    I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;
    I2C_InitStructure.I2C_OwnAddress1 = 0x00;
    I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;
    I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;
    I2C_Init(I2C1, &I2C_InitStructure);

    I2C_Cmd(I2C1, ENABLE);
}

void OLED_WriteCommand(uint8_t command) {
    I2C_GenerateSTART(I2C1, ENABLE);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT));

    I2C_Send7bitAddress(I2C1, OLED_ADDRESS << 1, I2C_Direction_Transmitter);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED));

    I2C_SendData(I2C1, 0x00);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    I2C_SendData(I2C1, command);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    I2C_GenerateSTOP(I2C1, ENABLE);
}

void OLED_WriteData(uint8_t data) {
    I2C_GenerateSTART(I2C1, ENABLE);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_MODE_SELECT));

    I2C_Send7bitAddress(I2C1, OLED_ADDRESS << 1, I2C_Direction_Transmitter);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED));

    I2C_SendData(I2C1, 0x40);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    I2C_SendData(I2C1, data);
    while (!I2C_CheckEvent(I2C1, I2C_EVENT_MASTER_BYTE_TRANSMITTING));

    I2C_GenerateSTOP(I2C1, ENABLE);
}

void OLED_Init(void) {
    OLED_WriteCommand(0xAE); // Display OFF

    OLED_WriteCommand(0x20); // Set Memory Addressing Mode
    OLED_WriteCommand(0x00); // Horizontal addressing mode

    OLED_WriteCommand(0xB0); // Set Page Start Address for Page Addressing Mode

    OLED_WriteCommand(0xC8); // COM Output Scan Direction
    OLED_WriteCommand(0x00); // Low Column Address
    OLED_WriteCommand(0x10); // High Column Address

    OLED_WriteCommand(0x40); // Start Line Address
    OLED_WriteCommand(0x81); // Contrast Control
    OLED_WriteCommand(0xFF);

    OLED_WriteCommand(0xA1); // Segment Re-map
    OLED_WriteCommand(0xA6); // Normal display

    OLED_WriteCommand(0xA8); // Multiplex Ratio
    OLED_WriteCommand(0x3F);

    OLED_WriteCommand(0xA4); // Display Output RAM
    OLED_WriteCommand(0xD3); // Display Offset
    OLED_WriteCommand(0x00);

    OLED_WriteCommand(0xD5); // Display Clock Divide Ratio
    OLED_WriteCommand(0xF0);

    OLED_WriteCommand(0xD9); // Pre-charge Period
    OLED_WriteCommand(0x22);

    OLED_WriteCommand(0xDA); // COM Pins Hardware Configuration
    OLED_WriteCommand(0x12);

    OLED_WriteCommand(0xDB); // VCOMH Deselect Level
    OLED_WriteCommand(0x20);

    OLED_WriteCommand(0x8D); // Charge Pump Setting
    OLED_WriteCommand(0x14);

    OLED_WriteCommand(0xAF); // Display ON
}

void OLED_SetCursor(uint8_t page, uint8_t column) {
    OLED_WriteCommand(0xB0 + page);
    OLED_WriteCommand((column & 0xF0) >> 4 | 0x10);
    OLED_WriteCommand((column & 0x0F) | 0x01);
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
    OLED_Clear();
}

void loop() {

    OLED_SetCursor(0, 0);
    OLED_DisplayString("Hello, OLED!");
    Delay_Ms(1000);
    OLED_Clear();
    Delay_Ms(1000);
}

int main(void) {
    SystemInit();
    setup();

    while (1) {
        loop();
    }
}

```

## 5. Explanation  
### 5.1 I2C Initialization:  

1. Configures GPIO pins PB6 and PB7 for I2C communication.
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

