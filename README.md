
# Objective 

To retrofit the Kennwood R-SE7 with a modern, more powerful on-chip computer than it's designated NEC uPD78045 CPU. Such as an Arduino but I've decided mine will be a 
Raspberry Pi. The uPD78045 will be removed from the existing board using a hot air rework station (preserving the pads) and the pads will be bodged wired to a 
breakout board using some 0.2 or 0.5mm copper wire which will then be used to connect the new raspberry pi. Most of the compoents housed in the rest of the amplifier 
are managed by the upd78045 using i2c however a fluroescent vacuum display driver will need to be implemented to interface the existing screen with the raspberry pi. 
Among some of the added functionality, I'm hoping to include 

- Networking / WiFi obviously; DLNA, jackd network listener, ffmpeg
- A PCM DAC; hopefully >44.1Khz sampling @ 24-bit playback (and hopefully capture from amp input channels)
- bluetooth pairing
- possibly IRDA
- A more customizable and informative, ａｅｓｔｈｅｔｉｃ－ａｓ－ｆｕｃｋ display

## Motivation 
This amp is one of a kind and really deserves to live on. In terms of it's design is very small and compact but offers very high quality sound and capable of driving 
higher end 6 ohm speakers such as many of the Wharfedale Diamond series speakers. The r-se7 was released sometime during the 90s primarily on the Japanese market but 
on the American and European markets as well. While the Japanese and European models featured FM radio RDS (radio data system) capability the American models did not. 
However, like any good electronic product that you pay a lot of money for should the blank PCB for the RDS was included inside of the housing of the units sold on the 
US market. The overall design of the amp is very straight forward and surprisingly very openly documented. I was able to obtain servicing schematics and I've found 
that nearly all of the parts for this amplifier are through hole and easily servicable. 

I have modified mine by removing the op amps and replacing them with sockets and slightly more modern Ti NE5532 parts. the original op amps were an assortment of 
Mitsubishi and Japan Radio Corp dual power supply types. I took this cue from another individual from Japan who also came to know the joys of owning this amp. He 
employed a Barr-Brown OPA2604s (seen here: https://blogs.yahoo.co.jp/toshi_fj1200/26411649.html) . I'm not certain that there is much benefit to this modification but 
I have not attempted to characterize different op amp configurations in a scientific way as of yet; I mostly just wanted an excuse to work on my soldering 
and it was a success. If there is an advantage it particularly makes a lot of sense in this amp given the "Pure A" feature which when activated gives you a constant 
amount of power from the B+ for a better listening experience (lowest possible distortion) but at the expense of higher overall power dissapation (35W over ~20W on 
Kill-a-watt) and the unit runs much hotter even when idle. I personally endorse that the listening experience with this mode is fantastic and well worth it. I 
typically run the amp at full volume but control the volume input from my Yamaha MG10XU mixer / DAC. The operation manual advises against using Pure A for normal 
listening because it runs hotter however I have ran this amp 24/7 for 2 years without any problems. The MG10XU also gives me some nice pan and low/high pass 
adjustments if I need or want them real quick.  

### It's a really good amp
Just as a sort of listener's assay I would go as far as to say I would struggle to be as happy with anything built on discrete logic other than 
this amp. It's probably one and it's particularly convienent because of its size and quality of the design which just doesn't exist anymore with anything new. It's 
really gonna be costly when I finally do have to buy a set of monoblock amps and a set of magnapans but I'm looking forward to the day, eventually--not right now! 
phew! And, on another note my "friend" recently moved to Las Vegas from Seattle and took my KRK rokit 8's with her which is fine, my amp has a sub channel (2.1) I'm 
thinking about buying a good class D powered sub, like a KRK 10' though the immediate lower-freq response with my 6 ohm sony speakers is more than satisfactory to be 
honest. Overall I use this system primarily with my computer which I use for movies, music, and games. 

And in the meantime I want to come up with a design to commemerate this amp, it really deserves it and it's a damn shame that it can't just be this good, even with 
modern amps. This one is really ahead of it's time and my original plan was to hack the NEC 78K but a full retrofit really does seem more appropriate. I looked at 
several ways. It took forever to track down the specific datasheet as Reneasas (formerly NEC) had all of the datasheets except for the 78045 for some reason. Also, 
I have the compiler toolchain as well as some related compiler profiles but it's all very esoteric and not easy to use without getting support (for EOL products 
support would be out of the question anyway.) This particular 78K cannot be programmed in-circuit and would have to be removed to be flashed anyway, so I figured
may as well come up with a different solution since most of the components are i2c (even the volume knob is an i2c rotary encoder.) 

Another idea I had to hack the 78k was just a guess that one could possibly write code to the eeprom mentioned in the schematic (which I was unable to locate on my 
unit) and potentially use the serial port (the one on the back of the unit for connecting other components like the kick ass matching cd/md/cd burner/tape 
player/recorder) to possibly instruct the CPU to run the program from eeprom. From reading the service manual for testing the serial port it sounds like the serial 
port may just change the instruction pointer and if the contents of the eeprom are addressable that might work? I haven't investigated it much and it doesn't seem
likely to happen at this point but I'd be interested in hearing someone's thoughts on this?

# uPD78045 pin notes
This is a cross reference between schematic and datasheet pin descriptions which will be necesarry for understanding how to interface another device in place of the 78k. The port pin numbers don't correspond to the actual physical pin number and that has to be cross referenced. 

## Abbreviations 
* STB       (strobe)
* FIP       (Fluroescent Indicator Panel)
* VLOAD     (Negative Power Supply)
* SB        (Serial2 Bus)
* SI0,SI1   (Serial Input)
* SO0,SO1   (Serial Output)
* ANI       (Analog Input) 
* AVDD      (Analog Power Supply) 
* AVSS      (Analog Ground)
* AVREF     (Analog Reference Voltage)
* VDD       (Power Supply) 
* VSS       (Ground)
* SCK0,SCK1 (Serial Clock)
* P00-P04   (Port 0)
* P10-P17   (Port 1)
* P20-P27   (Port 2)
* P30-P37   (Port 3)
* P70-P74   (Port 7)
* P80-P81   (Port 8)
* P90-P97   (Port 9)
* P100-P107 (Port 10)
* P110-P117 (Port 11)
* P120-P127 (Port 12)

## I/O Port details
* FIP pins are multiplexed with port pins
* CMOS Input: 2 lines
* CMOS I/O: 27 lines
* N-ch open-drain: 5 lines
* P-ch open-drain I/O: 16 lines
* P-ch open-drain output: 18 lines
* datasheet includes schematics for debouncing and various other pin configurations which should closely resemble those of the stereo schematic where applicable 

## A class amplifier (pure a control)
* pin 13 - controls the A class for constant +/- 12V  (pure A mode) datasheet says pin it is P23/STB 
* pin 49 (4.2v drives pure A on LED indicator)

## Fluro tube display pins (on the 78045)
* pins 1-7 - datasheet says 1-7 are (P94/FIP6, P93/FIP5, P92/FIP4, P91/FIP3, P90/FIP2, P81/FIP1, P80/FIP0) respectively (7)
* pins 65-70 - datasheet says 65-70 are (P113/FIP21, P112/FIP20, P111/FIP19, P110/FIP18, P107/FIP17, P106/FIP16) respectively (6)
* pins 72-80 - datasheet says 72-77 are (P105/FIP15, P104/FIP14, P103/FIP13, P102/FIP12, P101/FIP11, P100/FIP10) and 78-80 are (P97/FIP9, P96/FIP8, P95/FIP7) respectively (9)
* pins 61-64 - datasheet says 61-64 are (P117/FIP25, P116/FIP24, P115/FIP23, P114/FIP22) respectively (4)

### details 
* datasheet for the 7805 specifies two display mode registers verbatim DSPM0 and DSPM1 that can control a FIP of 9 to 24 segments and 2-16 digits; in the datasheet it's illustrated as an x,y plane where x is designated segments and y is designated digits. This is what I imagine will be the hardest part of this project is the fact that it's almost designed exclusively for specific types of fluroescent displays because it offers features like the description says verbatim: automatic output of segment signals (DMA operation) and digit signals by automatically reading display data.
* there's a total of 26 pins tied into the display
* the display appears to be capable of displaying up to 8 alphanumeric characters on it's segment/digit plane
* There are other static indicators which supports my theory that this is very specific to the display (duh) 
* The amplifier schematic illustrates the segments and designates groups 1G-8G and several sub designations as well as the corresponding pin numbers 

### display pins (of the actual display) and their corresponding designation and corresponding 78045 pin
* 4  - 1G - 7
* 5  - 2G - 6 
* 6  - 3G - 5 
* 7  - 4G - 4
* 8  - 5G - 3
* 9  - 6G - 2
* 10 - 7G - 1
* 11 - 8G - 80
* 12 - 9G - 79 ( 9G is not illustrated except for it's connection to the display )
* (13 - 19 are marked NX and have no apparent connection on the schematic)
* 20 - P1 - 77
* 21 - P2 - 76
* 22 - P3 - 75
* 23 - P4 - 74
* 24 - P5 - 73
* 25 - P6 - 72 
* 26 - P7 - 71
* 27 - P8 - 70 
* 28 - P9 - 69
* 29 - P10 - 68 
* 30 - P11 - 67
* 31 - P12 - 66
* 32 - P13 - 65
* 33 - P14 - 64 
* 34 - P15 - 63
* 35 - P16 - 62

I have a pretty decent idea for how these work and I'll be able to confirm this when my new scope arrives in the mail. The schematic also indicates the negative voltages on the port pins p1-p16 though the voltages they describe vary from port to port and I'm not sure if that's intended or if that's just what was observed but I would imagine its intended to be pulsed.

## PLL (input selector)
* pins 15-16 CLK & DATA (i2c) - datasheet says 15 is P21/SO1 and 16 is P20/SI1
* 19 & 21 PLL DO, PLL CE - datasheet says 19 is P73 and 21 is P17/ANI7

## Tuner 
* pin 27-28 signal level - datasheet says 27 is P11/ANI11 28 is P10/ANI10

## rotary encoder (volume control) 
* pins 25-26 - datasheet pin 25 is P13/ANI3 and pin 26 is P12/ANI12

## Buttons

## Others
* 1 & 3 (AC?) - datasheet pin 1 is P94/FIP6 and 3 is P92/FIP4
* 14 SEL STB (Strobe?) out - datasheet P22/SCK1
* 22 T. Mute - datashet P22/ANI6 
* 71 VLOAD (-30v) - datasheet describes this as VLOAD



# References

* All of the documentation I've collected for r-se7, several components, (and compilers for the 78k): [https://drive.google.com/open?id=1AyYiC1ziDI0ZXOdMTM25pujILzwA1Hd8]
* uPD78045 specific datasheet (file is named upd78042, but covers 78045): https://drive.google.com/open?id=17MiFq-sq1-PTsri5Ty9TpFgVPU2OfV-g
