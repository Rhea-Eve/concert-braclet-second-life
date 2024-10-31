# CS 60 Final Project: WiFi Powered IR Light (And Taylor Swift bracelets) on the ESP8266

## Idea
There are bracelets that are given out at concerts that use IR to control when the bracelets light up and not, so as to create a synchronized bracelet light show. However, after the concert is over, they are basically just trash since people do not know how to reuse them, and especially since the company that makes them (Pixmob) is very non-open source about what it releases, in part because it makes money by designing the concerts for artists (i.e., it is a closed software company not really a hardware company).

This got me thinking about how to repurpose these in smaller settings (i.e., not having a giant light system that you can blast a hall with).

This project was inspired by this reverse engineering project of the concert bracelets and my obsession with Taylor Swift:

[https://github.com/danielweidman/pixmob-ir-reverse-engineering](https://github.com/danielweidman/pixmob-ir-reverse-engineering)

This is a really cool article that goes over how these bracelets basically recreate a really low-level connection via IR binary to display multiple colors and have fade effects.

### Initial Project Idea
My initial project idea was to make a Raspberry Pi host a WiFi connection that ESP8266 could connect to and have the light states broadcast to them. Then there would be a small IR light controlled by the chip that can adapt with the messages. I also wanted to figure out how the bracelets worked/received messages (depending on certain things about the IR light, they change colors).

### What I Actually Did
I did not do exactly this; rather, I just did a proof of concept and networked in the other direction. I had the ESP8266 set up a WiFi network and server which would then wait for messages and turn on/off/colors the light accordingly.

This would still work since the IR messages themselves are a form of "broadcasting"; it just is not as scalable.

## What I Did

### Board Setup
I set up a Raspberry Pi that connected to the ESP8266 via a serial port. The ESP8266 was on a breadboard which was then hooked up to the IR light (as you can see in the code, this was on port 2).

### Development Environment
I had an SSH connection to the Raspberry Pi after connecting it to WiFi.

I then wrote code in this folder and made the code.

After the code is made, we put the ESP8266 chip in "flash mode" by plugging in the IO0 port and then power cycle the chip. Then run `make flash`.

To run the code, you need to plug back in the IO0 port and power cycle.

Then, to view the output to stdout, you run `make monitor`.

To stop the code, you plug back in IO0 port and power cycle.

### Functionality
Once the chip is running, the program will print out the IP address. For me, this was `192.168.1.216`.

Then you run in a terminal `curl 192.168.1.216/off` or `curl 192.168.1.216/on`.

You can also run `192.168.1.216/off` or `192.168.1.216/on` in a browser, which will then turn the IR light on/off!!!!

The on/off function is just for testing. The bracelets need a broadcasted sequence, which I implemented color for.

`void color(const char* pattern)` flashes the sequence in pattern with a wait of 694 us (compared to the 694.44 in the write-up above - this should be good enough, especially considering the runtime in between).

I implemented this in magenta which is the sequence 101011001100001000110001010110001010001, but it can also be done for other colors.

### Code Run Down
The code works by having the chip set up a server, and then using `httpd_register_uri_handler` to check for queries. (This was based heavily on example code for the ESP8266).

This then calls the relevant handle function. For example, `off_get_handler`

```c
esp_err_t off_get_handler(httpd_req_t *req)
{
    char*  buf;
    size_t buf_len;

    printf("off\n");

    //const char* resp_str = (const char*) req->user_ctx;
    //this is what we send!
    httpd_resp_send(req, "I am off now!", strlen("I am off now!"));
    printf("Turning the light off...!\n");
    gpio_set_level(LED_PIN, 0);

    return ESP_OK;
}
```

This sets the level for the pin to 0 (i.e., off). 

This works so long as we run `gpio_set_direction(IR_PIN, GPIO_MODE_OUTPUT);` in main, to set the direction of the port on our multi-directional circuit board. 

## Limitations and Future Steps
1) There is no command that ends the server. This would be very useful, considering it just keeps running. 
2) As mentioned above, ideally we would have the chip respond to broadcast messages, by connecting to a Raspberry Pi's wifi network and doing that as such. I did some work on this, but it is not done yet. 
3) Ideally, I would add more colors and added fade/header functionality mentioned in the repo listed above.
us
4) I plan to test this better once I have the bracelet (when I go to a concert this summer :). Right now, I am trusting the write up mentioned above!


