Examples and tasks for the Elektra BLE course.
-------

## TODO

- [ ] Create Segger Embedded Projects for template_project and ble_app_uart 
- [ ] Task 1 should be performed as a walkthrough so that course participants get used to Keil/SES
- [ ] Add SAADC task that shows how to measure VDD.  
- [ ] Add BLE Advanced task (create custom service and characteristic, custom UUID).  


### Course Evaluation

Please use the last 5 minutes of the course to fill out the course evaluation in the link below

[Link to course evaluation](https://docs.google.com/forms/d/e/1FAIpQLScDVuMpX1UtlaiAXDowjTt8rVwuVZcZafOsT5o1SLEj1SeHLg/viewform)

It is important to us that you tell us what you liked about the course and what you did not like about the course so that we can improve the course material and presentations.

The evaluation is of course anonymous. 

### Presentations
The presentations from the course can be downloaded using the links below:

[Nordic Introduction and Course Agenda](https://drive.google.com/open?id=0B21ni_IYbeTXaUdDeXZmbi1xeG8)

[nRF52 Introduction and BLE Protocol Basics ](https://drive.google.com/open?id=0B21ni_IYbeTXSTlqUjh2T2RpZjQ)

[nRF52 & PWM](https://drive.google.com/open?id=0B21ni_IYbeTXbkZrdjNBRGpuNzQ)

[BLE Security](https://drive.google.com/open?id=0B21ni_IYbeTXUHlRWUZrbGRWRUE)

### Requests by course participants

For those of you that asked about the Diffie-Hellman key exchange, I recommend that you take a look at the following video that explains the concept.

https://www.youtube.com/watch?v=YEBfamv-_do

For those of you that asked about alternatives to Keil, I recommend taking a look at Segger Embedded Studio

https://devzone.nordicsemi.com/blogs/1032/segger-embedded-studio-a-cross-platform-ide-w-no-c/

and you should also take a look at our Development with Eclipse and GCC tutorial

https://devzone.nordicsemi.com/tutorials/7

## Course Software

The tasks in this course requires that you download software, e.g. SKD, IDEs, MDKs and command-line tools. As the favoured option

### Nordic nRF5x Software Development Kit

The first thing you have to do is to download the nRF5x Software Development Kit which contains all the source code for the libraries, drivers and examples that we're going to use in this course. 

The nRF5x SDK v12.2.0 can be downloaded by clicking the link below

http://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v12.x.x/nRF5_SDK_12.2.0_f012efa.zip

After the download has finished you should extract the content to a folder of your choosing, but try to keep the path short e.g.

    C:\Nordic_Semiconductor\nRF5_SDK_12.2.0

### Windows

If your using a computer with a Windows OS, then you need to install the software listed below, in the order they are listed.

#### 1. Keil ARM MDK v5.22

https://www.keil.com/demo/eval/arm.htm

#### 2. nRF5x MDK for Keil

http://www.nordicsemi.com/eng/nordic/download_resource/51500/20/99480371

#### 3. nRF Commandline Tools

http://www.nordicsemi.com/eng/nordic/download_resource/51499/38/44235066

### Mac OSX

As Keil uVision is only available for Windows, we recommend that you use Segger Embedded Studio instead. 

#### 1. Segger Embedded Studio

https://www.segger.com/downloads/embeddedstudio

#### 2. J-Link Software and Documentation Pack
https://www.segger.com/downloads/jlink#

#### 2. nRF5x Commandline Tools for OSX
http://www.nordicsemi.com/eng/nordic/download_resource/53408/11/3272144

### Linux 

#### 1. GNU toolchain for ARM Cortex-M

https://launchpad.net/gcc-arm-embedded/+download

Download and install the latest version. Then make sure to add the path to your toolchain to your OS PATH environment variable:

    <path to install directory>/GNU Tools ARM Embedded/4.9 2015q3/bin

Adding the path makes it possible to run the toolchain executables from any directory using the terminal. To verify that the path is set correctly, type the following in your terminal:

    arm-none-eabi-gcc --version

This will return the version of the C compiler if the executable is found in your path.

To build an example in the SDK you first need to set the toolchain path in makefile.windows or makefile.posix depending on platform you are using. That is, the .posix should be edited if your are working on either Linux or OS X. These files are located in

<SDK>/components/toolchain/gcc

Open the file in a text editor, and make sure that the GNU_INSTALL_ROOT variable is pointing to your Gnu tools for ARM embedded Processors install directory. 

#### 2. nRF5x toolset tar for Linux 32-bit (nrfjprog and mergehex) 

http://www.nordicsemi.com/eng/nordic/download_resource/52621/15/41505072 (Linux32)

http://www.nordicsemi.com/eng/nordic/download_resource/51505/20/4271639 (Linux64)

# Hands-on Tasks 

## Hands-on Tasks - Day 1 


The tasks should be performed in the order that they are given and you should use the Template Project found in  nRF5_SDK_12.2.0\examples\peripheral\template_project.

The hands-on tasks for the first course day will cover the use of the application timer library, the button handler library and the UART library. If you have some time left after finsihing the 3 first taske, then you may try to solve the optional Temperature Sensor task.

If you have any compilation problems when using the function calls referred to in the task description then make sure that you enable the the correct modules in the skd_config.c file.

##Task 1: Application Timer
**Scope:** Use an application timer to toggle one LED on the nRF52 DK at a given interval. 

The Application Timer library provides a user friendly way of using the Real Time Counter 1 (RTC1) peripheral to create multiple timer instances. The RTC uses the Low Frequency Clock (LFCLK). Most applications keep the LFCLK active at all times. When using a SoftDevice the LFCLK is always active. Therefor there is normally very little extra power consumption associated with using the application timer. As the clock is 32.768 kHz and the RTC is 24 bit, the time/tick resolution is limited, but it takes a substantial amount of time before the counter wrap around (from 0xFFFFFF to 0). By using the 12 bit (1/x) prescaler the frequency of the RTC can be lowered.

In this part of the tutorial you will configure the library, and use it to create timers that call your timeout event handlers. These can be called repeatedly at a configurable interval or once at a configurable time from now.

**1 -** Include the required header files by adding the following lines below the existing include statements:

```C
#include "app_timer.h"
#include "nrf_drv_clock.h"
```

2 - As a SoftDevice is not enabled in this tutorial, the LFCLK must be requested explicitly. One way of doing this is using the Clock driver. Add the following function somewhere before `main()`

```C
static void lfclk_request(void)
{
    uint32_t err_code = nrf_drv_clock_init();
    APP_ERROR_CHECK(err_code);
    nrf_drv_clock_lfclk_request(NULL);
}
```

Add a line calling lfclk_request() in the start of the main() function:

```C
// Request LF clock.
lfclk_request();
```

3 - The Application Timer should be initialized with the APP_TIMER_INIT() macro. This must be called before any other calls to the Application Timer API. The four parameters must be selected so that they meet the requirements of the application:

* PRESCALER: will be written to the RTC1 PRESCALER register. This determines the time resolution of the timer, and thus the amount of time it can count before it wrap around. On the nRF52 the RTC is a 24-bit counter with a 12 bit prescaler that run on the 32.768 LFCLK. The counter increment frequency (tick rate) fRTC [kHz] = 32.768/(PRESCALER+1). For example, a prescaler value of 15 means that the tick rate or time resolution is 32.768 kHz * 1/(15+1) = 2.048 kHz and the timer will wrap around every (2^24) * 1/2.048 kHz = 8192 s.

* OP_QUEUES_SIZE: determines the maximum number of events that can be queued. Let's say you are calling the API function several times in a row to start a single shot timer, this determines how many times you can have queued before the queue gets full.

* SCHEDULER_FUNC should be set to false when scheduler is not used, as is the case in this tutorial. See the Scheduler tutorial for how to use the Scheduler with the application timer.

In this course you will create several timers. Add the following defines (which represent the parameters described above) close to the top of your code

```C
// General application timer settings.
#define APP_TIMER_PRESCALER             15    // Value of the RTC1 PRESCALER register.
#define APP_TIMER_OP_QUEUE_SIZE         3     // Size of timer operation queues.
```
Then put this line in main() in order to initialize the app timer (after the call to lfclk_request()):

```C
// Initialize the application timer module.
APP_TIMER_INIT(APP_TIMER_PRESCALER, APP_TIMER_OP_QUEUE_SIZE, false);
```

4 - An application timer in the repeated mode will restart each time it expires. Every time it expires the timeout handler will be called. This makes it suitable for performing a task at a regular interval, such as toggling a LED, which is what you will do now. 
Application timers are created using app_timer_create(). This function takes three parameters:

* p_timer_id: pointer to the ID of the timer, which will be populated during the execution of this call.
* mode: either single shot (APP_TIMER_MODE_SINGLE_SHOT) or repeated (APP_TIMER_MODE_REPEATED).
* timeout_handler: pointer to the timeout handler.

First, create a variable that can hold the timer ID to be populated by `app_timer_create()`. Add the following line to your code, close to the top of the file below the include statments

```C
APP_TIMER_DEF(m_led_a_timer_id);
```
Then you will have to create the timeout event handler, which we will use to toggle LED 1 every time it is called. You toggle the pin connected to a led in using  [nrf_gpio_pin_toggle](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__nrf__gpio.html#gac7f7bf539f5bb053b4a313ec51d8157e). Note: The LEDs on the nRF52 are active low.

```C
// Timeout handler for the timer
static void timer_handler(void * p_context)
{
    // Toggle led 1
    nrf_gpio_pin_toggle(LED_1);
}
```

It is a good idea to wrap the creation of the timer in a function, in order to keep a minimal main() function, i.e.

```C
static void create_timer()
{   
    uint32_t err_code;

    // Create timers
    err_code = app_timer_create(&m_led_a_timer_id,
                                APP_TIMER_MODE_REPEATED,
                                timer_handler);
    APP_ERROR_CHECK(err_code);
}
```
5 - Next, we should create the functions `start_timer` and `stop_timer` that will be used to start and stop the application timer. 

```C
static void start_timer()
{
    uint32_t err_code;

    // Start timer
    err_code = app_timer_start(m_led_a_timer_id,
                                APP_TIMER_TICKS(1000, APP_TIMER_PRESCALER),
                                NULL);
    APP_ERROR_CHECK(err_code);
}
```

```C
static void stop_timer()
{
     uint32_t err_code;

    // Stop timer
    err_code = app_timer_stop(m_led_a_timer_id);
    APP_ERROR_CHECK(err_code);
}
```

6 - After performing all the steps above your main should look like this

```C
int main(void)
{
    // Start the LFCLK
    lfclk_request();

    // Set LED_1 pin as output
    nrf_gpio_cfg_output(LED_1);
    nrf_gpio_pin_clear(LED_1);

    // Initialize the application timer module.
    APP_TIMER_INIT(APP_TIMER_PRESCALER, APP_TIMER_OP_QUEUE_SIZE, false);

    create_timer();
    start_timer();
   
    while (true)
    {
        
    }
}
```

##Task 2: Button Handler
**Scope:** Use the buttons on the nRF52 and the button handler library(app_button) to start and stop the application timer from Task 1. The button handler library is documented on [this](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__button.html?resultof=%22%62%75%74%74%6f%6e%22%20%22%68%61%6e%64%6c%65%72%22%20) Infocenter page. 

1 - Create a function called `button_init()`, where you configure the buttons you want to start and stop the application timer and enable the button handler module. Hint: The pins connected to the buttons will be short to ground when the buttons are pressed and should therefore be pulled to VDD when not pressed. A button is configured using a app_button_cfg_t struct as shown below

```C
    static app_button_cfg_t button_config;
    button_config.pin_no        = BUTTON_1;
    button_config.active_state  = APP_BUTTON_ACTIVE_LOW;
    button_config.pull_cfg      = NRF_GPIO_PIN_PULLUP; 
    button_config.button_handler = button_handler;
```
Since there are 4 buttons on the nRF52 DK you have to create an array that contains 4 of these structs, i.e. 

```C
    static app_button_cfg_t button_config[4];
    button_config[0].pin_no        = BUTTON_1 ;
    button_config[0].active_state  = APP_BUTTON_ACTIVE_LOW;
    button_config[0].pull_cfg      = NRF_GPIO_PIN_PULLUP; 
    button_config[0].button_handler = button_handler;
    
    button_config[1].pin_no        = BUTTON_2 ;
    button_config[1].active_state  = APP_BUTTON_ACTIVE_LOW;
    button_config[1].pull_cfg      = NRF_GPIO_PIN_PULLUP; 
    button_config[1].button_handler = button_handler;
   ...
```

After creating the app_button_cfg_t button_config[4] array, you have to call [app_button_init](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__button.html#ga5948c3d8e3e385820dc7d8df0976e8eb) and [app_button_enable](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__button.html#gab498b436a38ebb03393e1129a6daffc9)
in that order.

2 - Create the callback function `button_handler(uint8_t pin_no, uint8_t button_action)` that will be called whenever a button is pressed or released. In this function you can check which button that was pressed and which action caused the callback, i.e. 

```C
   static void button_handler(uint8_t pin_no, uint8_t button_action)
   {
       if(pin_no == BUTTON_4 && button_action == APP_BUTTON_RELEASE)
       {
           \\ Do something if button 4 is released. 
       }
   }
```
3 - Call the `start_timer`function from task 1 when Button 1 is pressed and `stop_timer`when button 2 is pressed. 



##Task 3: UART
**Scope:** Use the nRF52's UART peripheral and the UART library (app_uart) to echo data sent from a terminal. If you do not already have a favorite terminal application, then I recommend using [Termite] (http://www.compuphase.com/software_termite.htm). The UART library is documented on [this](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__uart.html?resultof=%22%41%50%50%5f%55%41%52%54%5f%46%49%46%4f%5f%49%4e%49%54%22%20) Infocenter page. 

1. Create the function uart_init where you use the APP_UART_FIFO_INIT macro to initialize the UART module.
   The baudrate should be set to 115200, Flow Control should be disabled, no parity bits are used and the RX and TX buffers should be      set to 256 in size. The UART pins of the nRF52 DK are listed on the backside of the board. See the UART example in the              \examples\peripheral\uart\pca10040\blank\arm5_no_packs folder
2. Create the function uart_event_handler as shown below and add code that echoes the received data. 

```C
    void uart_event_handler(app_uart_evt_t * p_event)
    {
        /*
        You're not allowed to decleare variables inside switch-cases, 
        so any variables used in the switch-case must be declared here.
        */
        switch (p_event->evt_type)
        {
            case APP_UART_DATA_READY:
                /*  
                The received data is stored in a receive buffer and can be retrieved using app_uart_get.
                Data to be sent can be placed in the transmit buffer using app_uart_put.
                */
                break;

            case APP_UART_COMMUNICATION_ERROR:
                APP_ERROR_HANDLER(p_event->data.error_communication);
                break;

            case APP_UART_FIFO_ERROR:
                APP_ERROR_HANDLER(p_event->data.error_code);
                break;

            default:
                break;
        }
    }
```
The APP_UART_DATA_READY event will be generated for each single byte that is received by the nRF52, which means that [app_uart_get](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__uart.html#gacddb5b7b711ef104f9eb181a13bc4503) must be called everytime the event is received. 

```C
    case APP_UART_DATA_READY:
        app_uart_get(&data_array[index]);
        index++;
        
        break;              
```

Since the [app_uart_get](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__uart.html#gacddb5b7b711ef104f9eb181a13bc4503) function takes the pointer to a uint8_t, we need an array to store the received bytes and and index variable to keep track of how many bytes we have received, i.e.

```C
    static uint8_t data_array[32];
    static uint8_t index = 0;              
```

Most terminals append the `\n` character, also known as the Line Feed character, to the end of the string that is sent. The `\n`  indicates that the next character should be printed on a newline. Therefore it makes sense to receive bytes until we see the `\n` character and then send the entire string back to the terminal using [app_uart_put](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__uart.html#ga2e4c8407274a151e72ed5a226529dc36). 

```C
  if (data_array[index - 1] == '\n') 
  {
    // Call app_uart_put to sent the bytes stored in data_array back to the terminal.
  }
```

The function app_uart_put used to place data in the UART's transmit buffer must be called in a for-loop if more that one byte is to be sent, i.e. 
```C
    for (uint32_t i = 0; i < strlen((const char *)data_array); i++)
    {
        while (app_uart_put(data_array[i]) != NRF_SUCCESS);
    }
```
After adding the array to hold the data and the index to keep track of hom many bytes we have received, adding the if statment and the for loop that calls app_uart_put(), the uart_event_handler function should look something like this:

```C
    void uart_event_handler(app_uart_evt_t * p_event)
    {
        static uint8_t data_array[32];
        static uint8_t index = 0;

        switch (p_event->evt_type)
        {
            case APP_UART_DATA_READY:
                app_uart_get(&data_array[index]);
                index++;

                if (data_array[index - 1] == '\n') 
                {
                    for (uint32_t i = 0; i < strlen((const char *)data_array); i++)
                    {
                        while (app_uart_put(data_array[i]) != NRF_SUCCESS);
                    }
                    memset(data_array,0,sizeof(data_array));
                    index = 0;
                }
                break;

            case APP_UART_COMMUNICATION_ERROR:
                APP_ERROR_HANDLER(p_event->data.error_communication);
                break;

            case APP_UART_FIFO_ERROR:
                APP_ERROR_HANDLER(p_event->data.error_code);
                break;

            default:
                break;
        }
    }
```
The memset function is used to clear the data_array since it is decleared as static, i.e. it will not erase the content in between the calls to `uart_event_handler`. If we do not set data_array to 0 and receive a string that is shorter than the last string we received, then some of the old data will still be stored in the array.

##Task 4: Temperature Sensor (Optional)
**Scope:** Use the die temperature sensor on the nRF52 to measure the temperature in the room. 

1. Create the function read_temperature() that returns the die temperature as a int32_t. *Hint:* Take a look at the temperature example in the SDK before you start modifying your template example.

2. Send the temperature data to your terminal application using the UART. 

Hint 1: The function app_uart_put is used to place data in the UART's transmit buffer and must be called in a for-loop if more that one byte is to be sent, i.e. 
```C
    for (uint32_t i = 0; i < strlen((const char *)data); i++)
    {
        while (app_uart_put(data[i]) != NRF_SUCCESS);
    }
```
Hint 2: Use [sprintf](https://www.tutorialspoint.com/c_standard_library/c_function_sprintf.htm) to copy the content of a string into an array.

## Hands-on Tasks - Day 2

Task 5 and Task 6 are non-BLE tasks and should be done in the template_project example used on Day 1. Task 7 and upwards are BLE task and should be done in the ble_app_uart example.

##Task 5: PWM Sweep
**Scope:** In this task we will use the PWM library in the nRF5x SDK to control a servo. The PWM library uses one of the nRF52s TIMER peripherals in addition to the PPI and GPIOTE peripherals. The app_pwm library is documented on [this](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/lib_pwm.html?resultof=%22%61%70%70%5f%70%77%6d%5f%69%6e%69%74%22%20) Infocenter page

Connecting the Servo to your nRF52 DK:

The three wires coming from the SG92R Servo are:

Brown: 	Ground 				- Should be connected to one of the pins marked GND on your nRF52 DK. 

Red: 	5V 					- Should be connected to the pin marked 5V on your nRF52 DK.

Orange: PWM Control Signal 	- Should be connected to one of the unused GPIO pins of the nRF52 DK (for example P0.4, pin number 4).

1 - The first thing we have to do is to include the header to the PWM library, `app_pwm.h` and create a PWM instance using the TIMER1 peripheral. This is done as shown below 

```C
    #include "app_pwm.h"
    
    APP_PWM_INSTANCE(PWM1,1);                       // Create the instance "PWM1" using TIMER1.
```
2 -	The second thing we have to do is creating the function `pwm_init()` where we configure, initialize and enable the PWM peripheral. You configure the pwm library by creating a app_pwm_config_t struct like shown below

```C
    app_pwm_config_t pwm_config = {
        .pins               = {4, APP_PWM_NOPIN},
        .pin_polarity       = {APP_PWM_POLARITY_ACTIVE_HIGH, APP_PWM_POLARITY_ACTIVE_LOW}, 
        .num_of_channels    = 1,                                                          
        .period_us          = 20000L                                                
    };
```
This struct contains the information about which pins that are used as PWM pins, which polarity the pisn should have, how many channels(the number of PWM outputs, this is limited to 2 per PWM instance) and the period of the PWM signal. The struct is given as an input to the [app_pwm_init](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#gae3b3e1d5404fd776bbf7bf22224b4b0d) function which initializes the PWM library. 

```C
    uint32_t err_code;
    err_code = app_pwm_init(&PWM1,&pwm_config,NULL);
    APP_ERROR_CHECK(err_code);
```

You can initialize the PWM library with a callback function that is called when duty cycle change process is finsihed, but this is not necessary for this example so we'll just pass NULL as an argument. After initializng the PWM library you have to enable the PWM instance by calling [app_pwm_enable](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#ga94f5d824afec86aff163f7cccedaa436).

```C
    app_pwm_enable(&PWM1);
```
The pwm_init() function is now finished and can add it to the `main()` function before the infinite for-loop.

3 - Now that we have initialized the PWM library its time to set the duty cycle of the PWM signal to the servo using the  [app_pwm_channel_duty_set](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#ga071ee86851d8c0845f297df5d23a240d) function. This will set the duty cycle of the PWM signal, i.e. the percentage of the total time the signal is high or low depending on the polarity that has been chosen. If we want to set the PWM signal to be high 50% of the time, then we call `app_pwm_channel_duty_set` with the following parameters.

```C
    while (app_pwm_channel_duty_set(&PWM1, 0, 50) == NRF_ERROR_BUSY);
```
The `app_pwm_channel_duty_set` function should be called until it does not return `NRF_ERROR_BUSY` in order to make sure that the duty cycle is correctly set. 

4 - The goal of this task was to make the servo sweep from its maximum angle to its minimum angle. This can be done by calling app_pwm_channel_duty_set twice with a delay between the two calls in the main while-loop.

```C
    while (true)
    {
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        nrf_delay_ms(1000);
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        nrf_delay_ms(1000);
    }
    
```
The code snippet above sets the duty cycle to 0, you have to figure out the correct duty cycle values for the min and max angle. 

Tips:
* Period should be 20ms (20000us) and duty cycle  for the min and max angle corresponds to 1ms and 2ms respectivly.

##Task 6: PWM & Buttons
**Scope:** Modify the button handler from task 1 so that the servo is placed at its minimum angle angle by pressing button 3 and its maximum angle by pressing button 4.  

# Hands-on Tasks - Day 3



##Flashing the SoftDevice to the nRF52 DK

The following tasks uses the S132 v3.0.0 SoftDevice, which must be flashed to your nRF52 DK. There are two ways to flash the SoftDevice to the nRF52 DK.

###Option 1 - nrfjprog

Navigate to the *\components\softdevice\s132\hex* folder. Click the address line, type in cmd and press enter. This should open a terminal window in the *\components\softdevice\s132\hex* folder.

Now type in 

    nrfjprog --help

this should display all the commands that can be used with nrfjprog and what they do. 

In order to flash the SoftDevice you have to use the following command

    nrfjprog --family nrf52 --program s132_nrf52_3.0.0_softdevice.hex --verify

If you get an error stating that the flash is not erased, then you can erase the flash of the chip with the following command
    
    nrfjprog --family nrf52 --eraseall

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/nrfjprog.PNG" width="1000"> 

###Option 2 - nRFGO Studio

Download nRFGo Studio from the page linked to below

https://www.nordicsemi.com/eng/Products/2.4GHz-RF/nRFgo-Studio/

Open nRFGo Studio, select the Segger xxxxxxxxx device in the Device Manager, then select the Program SoftDevice tab,  navigate to the *\components\softdevice\s132\hex* folder and select the s132_nrf52_3.0.0_softdevice.hex file and press program 

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/nRFGo_studio.PNG" width="1000"> 


##Task 7: Control LEDs using the nRF Toolbox App

**This task is a _BLUETOOTH TASK_!**

**Scope:** Modify the ble_app_uart example to recognise specific commands sent from the nRF Toolbox app and turn on a LED when one of these commands are received.

1 - Open the ble_app_uart example found in the nRF5_SDK_12.2.0\examples\ble_peripheral\ble_app_uart\pca10040\s132\arm5_no_packs folder. Find the `DEVICE_NAME` define and change the device to a unique name that is easily recognisable, for example.

  
    #define DEVICE_NAME                     "Bjoern_UART"       


2 - We need a variable to keep track of the current command that the nRF52 should handle. We can do this by creating an enumeration, which is basically a list of commands that are assigned a number from 0 and upwards. We create an enumeration  like this

```C    
    typedef enum {
        COMMAND_1,
        COMMAND_2,
        COMMAND_3,
        NO_COMMAND
    } uart_command_t;
```
Every variable of the uart_command_t type can be set to one of the commands in the list.  We've  added a `NO_COMMAND` command which is going to be the default state when no command has been received or the last command has been completed. After declaring the enumeration type `uart_command_t` we need to create a variable `m_command` of the `uart_command_t` type and initialize it to `NO_COMMAND`, i.e.

```C  
    uart_command_t m_command = NO_COMMAND;
```

3 - Find the function `nus_data_handler`. This function is called when data is sent to the Nordic UART Service(NUS) from the nRF Toolbox app and this is where we have to look for the specific commands. The data that has been received is stored in a array pointed to by the `p_data` pointer and we need to store it a local array for later use. This can be done by using the [memcpy](https://www.tutorialspoint.com/c_standard_library/c_function_memcpy.htm) function. It will copy the content from cell 0 to `length` in the array pointed to by p_data into the uart_string.      

```C  
    char uart_string[BLE_NUS_MAX_DATA_LEN];
    memset(uart_string,0,BLE_NUS_MAX_DATA_LEN);
    memcpy( uart_string, p_data, length);
```

4 - Now that we have copied the received data into the `uart_string` array we want to compare the content of `uart_string` with a known command. This can be done by using the [strcmp](https://www.tutorialspoint.com/c_standard_library/c_function_strcmp.htm) function, which will return 0 if `uart_string` is equal to the 

```C  
    if(strcmp(uart_string,"COMMAND_1") == 0 )
    {
        m_command = COMMAND_1;    
    }
    else if(strcmp(uart_string,"COMMAND_2") == 0 )
    {
        m_command = COMMAND_2;
    }
    else if(strcmp(uart_string,"COMMAND_3") == 0)
    {
        m_command = COMMAND_3;
    }
    else
    {
        m_command = NO_COMMAND;
    }
```
If the uart_string that we received is equal to the known "COMMAND_1" string, then we set the `m_command` variable to the corresponding command in enumeration we created in step 1.   

5 - Now that the `m_command` variable is set to the correct command if the correct string is received, the last thing we need is a function that checks these commands at a regular interval and runs the code we have assosiated with that command. We'll call this function `uart_command_handler` and it takes the pointer to the m_command variable as an input. Inside the function we have a [switch](https://www.tutorialspoint.com/cprogramming/switch_statement_in_c.htm) statement, which is very useful when comparing a variable against a list of values, like our `uart_command_t` enumeration. The `uart_command_handler` should look something like this

```C  
    void uart_command_handler(uart_command_t * m_command)
    {
        uint32_t err_code = NRF_SUCCESS;

        switch(*m_command)
        {
            case COMMAND_1:
                // Put Action to COMMAND_1 here.
                break;

            case COMMAND_2:
                // Put Action to COMMAND_2 here.
                break;

            case COMMAND_3:
                // Put Action to COMMAND_3 here.
                break;

            case NO_COMMAND:
                // No command has been received -> Do nothing
                break;
                
            default:
                // Invalid command -> Do nothing.
                break;
        }
        /* Reset the command variable to NO_COMMAND after a command has been handled */
        *m_command = NO_COMMAND;

        // Check for errors 
        APP_ERROR_CHECK(err_code);
    }
```
After declaring the `uart_command_handler` we add the uart_command_handler to the infinite for-loop in main as shown below. 
```C 
    for (;;)
        {
            uart_command_handler(&m_command);
            power_manage();
        }
```
6 - Now we want to toggle a led when we receive "COMMAND_1". Since the ble_app_uart example uses LED_1 on the nRF52 DK to indicate if the device is advertising or connected to central, so we have to toggle LED_4 instead. 
```C 
    case COMMAND_1:
        nrf_gpio_pin_toggle(LED_4); 
        break;
```
We also have to configure the pin connected to LED_4 as an output so make sure that you add to main()
```C 
    nrf_gpio_cfg_output(LED_4); 
```

7 - Compile the project and flash it to the nRF52 DK. Make sure that you've also flashed the S132 v3.0 SoftDevice to your board. LED 1 on the nRF52 DK should start blinking, indicating that its advertising.  We've now completed the configuration on the nRF52 side  

8 - Install the nRF Toolbox app on you Android/iOS phone. You can find the app [here](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Toolbox-App) on Google Play Store and [here](https://itunes.apple.com/us/app/nrf-toolbox/id820906058?mt=8) on Apple App Store. Open the nRF Toolbox app and click the UART symbol, which should display the picture under "UART Menu". Press EDIT in the top right corner, the menu should now turn orange and then press the top-left square in the 3x3 matrix. The app should now display the same image as shown under "Edit Button Menu". Enter the command shown in the "Configure Command 1" and select 1 as the icon. After pressing OK you should see return to the orange edit screen shown under "Edit Mode". Press "DONE" in the top-right corner and you should return to the blue UART menu as shown under "Edit Complete".  



nRF Toolbox Menu  | UART Menu     | Edit Button Menu| Configure Command 1 | Edit Mode | Edit Completed |
------------ | ------------- | ------------  | ------------  | ------------  | ------------  |
<img src="https://github.com/bjornspockeli/elektra/blob/master/images/nrf_toolbox.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/default.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/edit.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/command_1.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/edit_done.png" width="200"> | <img src="https://github.com/bjornspockeli/elektra/blob/master/images/done.png" width="200">

9 - Press the Connect button, this should bring up a list of nearby BLE devices. Select the device with the name you assigned in step 1 of this task. It should be the one of the devices with the strongest signal. LED 1 on your nRF52 DK should now stop blinking and stay lit, indicating that its in a connected state. 

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/device_list.png" width="200">

10 - Pressing the button we configured to send "COMMAND_1" to the nRF52 DK should turn on LED 4 on the nRF52 DK. Pressing it again should turn it off. Congratulations, you've just controlled one of the GPIO pins of the nRF52 using Bluetooth Low Energy.

##Task 8: Control the Servo using the nRF Toolbox App
**Scope:** The goal of this task is to include the PWM library in the ble_app_uart example so that we can control the servo from our smartphone instead of using the buttons on the nRF52 DK. 


1 - The files used by the app_pwm library were already included in the template_project, but in the ble_app_uart project you have to add the necessary .c and .h files to the project. The app_pwm library uses the following files

* `app_pwm.h`
* `app_pwm.c`
* `nrf_drv_ppi.c`
* `nrf_drv_timer.c`

##Adding .h files 

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/include_path.PNG" width="1000"> 
Click the "Options for target" button in Keil, then select the C/C++ tab and clik on the "..." on the side of the "Inlude Paths" window. Navigate to the components folder and then find the missing .h file in either nrf_drivers or libraries. 

 ##Adding .c files
 
<img src="https://github.com/bjornspockeli/elektra/blob/master/images/add_c_files.png" width="1000"> 
Right-clik the folder that you want to add the .c file to and select "Add existing files to Group '____'". Navigate to the components folder and then find the missing .c file in either nrf_drivers or libraries. 


You also have to make sure that the correct nRF_Drivers and nRF_Libraries are enabled in the sdk_config.h file. If you're having compilation issues and/or linker errors then select the Configuration Wizard Tab in the bottom of the text window after opening `sdk_config.h` in the ble_app_uart example and compare it to the one in the pwm_library example.

##Modifying sdk_config.c  

<img src="https://github.com/bjornspockeli/elektra/blob/master/images/sdk_config.PNG" width="1000">

Under nRF_Libraries the following boxes must be checked

* APP_PWM_ENABLED

Under nRF_Drivers the following boxes must be checked
* PPI_ENABLED
* TIMER_ENABLED
    * TIMER1_ENABLED

2 - Repeat the steps you did in Task 5: PWM Sweep, i.e. copy the functions you created in the template project to initialize the PWM library into the ble_app_uart project.

3 - Add `SERVO_POS_1` and  `SERVO_POS_2` to the `uart_command_t` enumeration created in Task 7, step 2. Add these commands to the `nus_data_handler` and the `uart_command_handler`. Call `app_pwm_channel_duty_set` when the commands are processed by the `uart_command_handler`, i.e.
```C 
    case SERVO_POS_1:
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        break;
```

4 - Configure two buttons in the nRF Toolbox app to send the `SERVO_POS_1` and  `SERVO_POS_2` commands to your nRF52 DK.

5 - Compile the project, flash it to the nRF52 DK and control the servo using the nRF Toolbox App.

##Task 9: Measure the die temperature of the nRF52 and send it to the nRF Toolbox app.
**Scope:** 

1 - In order to measure the temperature of the nRF52 die you have to read the registers of the TEMP peripheral of the nRF52, see [this](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.nrf52832.ps.v1.1/temp.html?cp=2_2_0_26#concept_fcz_vw4_sr) page on the Nordic Infocenter. However, the SoftDevice uses this peripheral to calibrate the 16 MHz clock of the nRF52 so that its accurate enough to be used for BLE. We can therefore not access the TEMP registers directly, we have to go through the SoftDevice and ask it to check what the temperature is. This is done by calling the [sd_temp_get](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.s132.api.v3.0.0/group___n_r_f___s_o_c___f_u_n_c_t_i_o_n_s.html#gade0ea69f513ff1feab2c4f6e1c393313
) function. 


We can now create a function called `read_temperature()` that in turn calls `sd_temp_get` and returns the temperature in degrees celsius.

```C 
    static double read_temperature()
    {
        int32_t temp;
        sd_temp_get(&temp);

        return ((double)temp*0.25);   
    }
```

3 - Next, we're going to create an application timer that calls `read_temperature()` periodically. First, create the timer ID as shown below

```C 
    APP_TIMER_DEF(m_temp_timer_id);                 // Create the timer ID "m_temp_timer_id" .
```

4 - Next, we're going to create the application timer timeout handler that is going to call `read_temperature()`. We need to use the `ble_nus_string_send()` function to send the measured temperature to the nRF Toolbox app. This function takes the pointer to an `uint8_t` array so we need to format a string and place it in the array. The  [sprintf](https://www.tutorialspoint.com/c_standard_library/c_function_sprintf.htm) function does exactly this, i.e. copies the content of a string into an array.

```C
    void temp_timer_timeout_handler(void * p_context)
    {
        double temp = read_temperature();

        // Place the temperature measurement into the data array
        uint32_t err_code;
        uint8_t data[20];
        sprintf((char *)data, "Temperature: %f", temp);
        
        //Send temperature measurement to nRF Toolbox app
        err_code = ble_nus_string_send(&m_nus, data, sizeof(data));
        APP_ERROR_CHECK(err_code);
    }
```


5 - Next, we need to create the timer and specify that it should use `temp_timer_timeout_handler` as its timeout handler.

```C 
    void create_timers()
    {
        uint32_t err_code;
        // Create  temperature timer
        err_code = app_timer_create(&m_temp_timer_id,
                                    APP_TIMER_MODE_REPEATED,
                                    temp_timer_timeout_handler);
        APP_ERROR_CHECK(err_code);
    }
```
6 - Now, the only thing that remains is to start the application timer. However, it is important that the `ble_nus_string_send` function is not called when we're not connected to the nRF Toolbox app. If we try to send the string containing the temperature without being in a connection then the application will crash.  It should therefore only be possible to start the timer when we issue a command from nRF Toolbox.  

Add `TEMP_TIMER_START` and `TEMP_TIMER_STOP` to the uart_command_t enumeration and modify the `nus_data_handler` so that these commands are recognized.


Lastly, add the following cases to the `uart_command_handler`

```C 
        case TEMP_TIMER_START:
            err_code = app_timer_start(m_temp_timer_id, APP_TIMER_TICKS(1000, APP_TIMER_PRESCALER), NULL);
            APP_ERROR_CHECK(err_code);
            break;
        
        case TEMP_TIMER_STOP:
            err_code = app_timer_stop(m_temp_timer_id);
            APP_ERROR_CHECK(err_code);
            break;
```

7 - Configure two buttons in the nRF Toolbox app to send the `TEMP_TIMER_START`and `TEMP_TIMER_STOP` commands

8 - Compile the project and flash it to you nRF52 DK along with the S132 v3.0.0 SoftDevice if its not already flashed to the DK. After pressing the btton you configured to send the `TEMP_TIMER_START` command you should be able to see the temperature in the nRF Toolbox app log ( you open this by holding your finger above the UART text to the left of the screen and swiping from left to right)  
 
<img src="https://github.com/bjornspockeli/elektra/blob/master/images/temperature.png" width="500">
