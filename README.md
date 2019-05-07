# DigiDev_Spring2019_Project
Arduino novice exploring digital communication in hardware with sensors and strip LEDs.
I'm sharing my process in the hopse thats someone else in sinpired. Maybe they can do what I could not, or something entirely different.


## Idea Origins
When we were given this project I had no idea what I wanted to make. I couldn't wrap my head around the wiring and coding well enough to envision something I would find interesting enough to make. Since i'm new to digitial developtment that may have been a barrier that stunted my exploration.

It wasn't until I unfortunately dropped my favorte pair of headphones on the ground and completely ruined the left ear speaker. I then decided I could do something with this peice of hardware because they meant so much to me. I was the type of individual who was never seen without my headphones. Always huge over-the-ear headphones, the same model for 6 years.
To me they were a sign communitationg to others that "I can't hear you." More importantly, "I don't want to speak (to you/right now/at all)". However, they don't do this job well enough. 

As headphones sets are now it's one-way communication: You listen to them. (not including sets with mics)
-They don't listen to you or anyone else.
-They don't speak to you or anyone else.

I wondered what kind of headphone set I could create that could speak to others for you. What if they could also sense your envirnoment and make changes.

Imagine walking to the train station with youe headphones on, jamming out to your one of your many playlists, while the headphones indicate to everyone else around you that you aren't listening. You're in a "do not disturb"-mode. Now you're enterting the train and there's a rouwudy group enjoying themsleves and making a huge commition. Your headphones sense the audio levels and might turin up your volume (to a safe level) and block them out. (Im using a semi-noise canceling headset for urban enviroments where you'd still want to have some audio awareness of your surroundings.) Once you're off the train and heading down the back alley to the entrance of your usual dive bar, the voulume lowers itslef since it's picking up little to now audio in the surrounding area.




## Wiring
![fritzing setup](/AngleLED_Setup_bb.jpg "fritzing setup")

## Coding
As I've said this was all new to me so i've used alot of example files to help build a high-level prototype of what I orginally planned to create. High-level meaning "in a borad sense" of theorgial goal. This is exploring one possible function of a larger story.

```
#include <Wire.h>
#include <MPU6050_tockn.h>
#include <I2Cdev.h>

#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h> // Required for 16 MHz Adafruit Trinket
#endif
Adafruit_NeoPixel strip = Adafruit_NeoPixel(14, 6, NEO_GRB + NEO_KHZ800);

MPU6050 mpu6050(Wire);

int MPUinputPin = 2;
int accelerometer_x, accelerometer_y, accelerometer_z;
int gyro_x, gyro_y, gyro_z; // variables for gyro raw data

void setup() {


  pinMode(MPUinputPin, INPUT);     // declare sensor as input

  //mpu5060 set up

  Wire.begin();
  mpu6050.begin();
  mpu6050.calcGyroOffsets(true);

  //neopixel setup

  // These lines are specifically to support the Adafruit Trinket 5V 16 MHz.
  // Any other board, you can remove this part (but no harm leaving it):
#if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
  clock_prescale_set(clock_div_1);
#endif
  // END of Trinket-specific code.
  strip.begin();
  strip.show(); // Initalize all pixels to 'off'
  strip.setBrightness(50); // Set BRIGHTNESS to about 1/5 (max = 255)
  Serial.begin(9600);

}

void loop() {
  //code for the mpu6050
  mpu6050.update();

  //need to retrieve Z, and Y paramaters and create algrithim for when Y,
  //and Z  are in given "field" they trigger changes in the neopixel strip
  Serial.print("angleX : ");
  Serial.print(mpu6050.getAngleX());
  Serial.print("\tangleY : ");
  Serial.print(mpu6050.getAngleY());
  Serial.print("\tangleZ : ");
  Serial.println(mpu6050.getAngleZ());
  //end print code for the mpu6050

  //code or the neopixel strip.color=(R,B,G)

  if (mpu6050.getAngleX() > -35)
  {
    for (int i = 0; i < strip.numPixels(); i++)
    {
      strip.setPixelColor(i, strip.Color( 0, 127, 0));
      strip.show();
    }
  }
  else
  {
    for (int i = 0; i < strip.numPixels(); i++)
    {
      colorWipe(strip.Color(127, 0, 0 ), 10);
      strip.show();
    }
  }
}

//rainbow colorWipe
void colorWipe(uint32_t color, int wait) {
  for(int i=0; i<strip.numPixels(); i++) { // For each pixel in strip...
    strip.setPixelColor(i, color);         //  Set pixel's color (in RAM)
    strip.show();                          //  Update strip to match
    delay(wait);                           //  Pause for a moment
  }
}

//rainbow code
void rainbow(int wait) {
  // Hue of first pixel runs 5 complete loops through the color wheel.
  // Color wheel has a range of 65536 but it's OK if we roll over, so
  // just count from 0 to 5*65536. Adding 256 to firstPixelHue each time
  // means we'll make 5*65536/256 = 1280 passes through this outer loop:
  for(long firstPixelHue = 0; firstPixelHue < 5*65536; firstPixelHue += 256) {
    for(int i=0; i<strip.numPixels(); i++) { // For each pixel in strip...
      // Offset pixel hue by an amount to make one full revolution of the
      // color wheel (range of 65536) along the length of the strip
      // (strip.numPixels() steps):
      int pixelHue = firstPixelHue + (i * 65536L / strip.numPixels());
      // strip.ColorHSV() can take 1 or 3 arguments: a hue (0 to 65535) or
      // optionally add saturation and value (brightness) (each 0 to 255).
      // Here we're using just the single-argument hue variant. The result
      // is passed through strip.gamma32() to provide 'truer' colors
      // before assigning to each pixel:
      strip.setPixelColor(i, strip.gamma32(strip.ColorHSV(pixelHue)));
    }
    strip.show(); // Update strip with new contents
    delay(wait);  // Pause for a moment
  }
}
```


