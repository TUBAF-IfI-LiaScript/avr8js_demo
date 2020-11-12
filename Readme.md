<!--
author:   Sebastian Zug & André Dietrich
email:    sebastian.zug@informatik.tu-freiberg.de & andre.dietrich@informatik.tu-freiberg.de
version:  0.0.6
language: en

narrator: US English Male

import: https://github.com/LiaTemplates/AVR8js/main/README.md#10
  https://raw.githubusercontent.com/liaTemplates/JSCPP/master/README.md
-->

# How to embed plain c and assembly code in your arduino project?

_André Dietrich, Sebastian Zug - TU Bergakademie Freiberg, Germany_

---------------------

This course illustrates the integration and the usage of _avr8js_ in LiaScript. It illustrates combination of explaination parts with interactive simulator sessions.

The interactive version is available via [Link](https://liascript.github.io/course/?https://raw.githubusercontent.com/TUBAF-IfI-LiaScript/avr8js_demo/main/Readme.md#1)

An webbased authoring tool - CodiLIA - can be found on [Link](https://liamd.informatik.tu-freiberg.de/6VF6RN68SkSM9HEx_xj8RA?edit). It bases on CodiMD but integrates all features of LiaScript too. Feel free to log in and to publish your own course materials.

The basics of LiaScript are available

+ on the [project website](https://liascript.github.io)
+ in the [documentation](https://github.com/liaScript/docs)
+ in short [presentation](https://github.com/SebastianZug/WillkommenAufLiaScript) of ideas and motivations

Please be aware, that LiaScript offers three modi - book style, presentation and lecture with text2speech elements. You can switch between these modes by clicking the symbol in the right upper corner.

---------------------

## Hello World from Arduino

A traditional Hello-World example for each enthusastic Arduino developer is a blinking LED.

> Switch to the presentation mode (ear symbol) and follow the examplary dialog by click the left arrow in the control bar.

    --{{1}}--
What are the basic components of the following Arduino program?

    --{{2 UK English Male}}--
First of all, each Arduino program consists of at least two functions `loop` and `setup`. While the first one, contains the configuration part, we define continously executed code in the setup function.

    --{{3}}--
And what will happen?

    --{{4 UK English Male}}--
We toggle one of the pins, it means the current is switched on and off. Make a try and start the simulation by pressing the button.

<div>
<wokwi-led color="red" pin="13" port="B"></wokwi-led>
</div>

```cpp helloWorldinArduino.cpp
void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  delay(1000);
}
```
@AVR8js.sketch


## Avrlibc Code

Ok, stop. A simple blinking LED generates 960 Bytes of Code in program memory? Are we able to realize the same function with a smaller program?

Of course we can you plain avrlibc directly. It provides much smaller programms with less overhead but cannot be transfered to non atmega Arduinos.

| Line | Meaning                                                                            |
| ---- | ---------------------------------------------------------------------------------- |
| 8    | Configuration of an output by writing `1` to corresponding register                |
| 11   | Toggling pegel for our specific pin by appling xor on its state and a mask pattern |
| 12   | ....                                                                               |


<div>
<wokwi-led color="red" pin="13" port="B"></wokwi-led>
<span id="simulation-time"></span>
</div>

```cpp avrc.cpp
#define F_CPU 16000000UL

#include <avr/io.h>
#include <util/delay.h>

int main (void) {

   DDRB |= (1 << PB5);

   while(1) {
       PORTB ^= (1 << PB5);
       _delay_ms(254);
   }

   return 0;
}
```
@AVR8js.sketch

Are you aware of the mechanims when deleting or setting individual bits in a register?

!?[Youtube](https://www.youtube.com/watch?v=BKzB6gdRyIM)


Train the usage of bitwise operators `|` and `&` in the following example.

1. Delete the second `1` in `x`.
2. Set the 7th bit in x to `1`.


```cpp  SetAndReset.cpp
#include <iostream>
using namespace std;

long long convertToBinary(int n){
    long long binaryNumber = 0;
    int remainder, i = 1;
    while (n!=0){
        remainder = n%2;
        n /= 2;
        binaryNumber += remainder*i;
        i *= 10;
    }
    return binaryNumber;
}

int main() {
    int x    = 0b00010000;
    int mask = 0b00000010;
    cout << convertToBinary(x | mask);
    return 0;
}
```
@JSCPP.eval

Most of the program size is now used for the interrupt vector. Add a `-nostdlib`
to your local compiler parameters for for extracting actual code of 30 Bytes.

## Assembly code

Unfortunatly, the current _avr8js_ tool chain does not allow to configure
compiler parameters individually. If it will be integrated, we can compare the
previous codes with pure assembly code.

```as
main:  ; ------- INIT -------------------
       ; set DDRB as output
       ; sbi 0x04, 7
       sbi _SFR_IO_ADDR(DDRB),5
       ; ------- Busy waiting 1 s -------
       ; available on
       ; http://www.bretmulvey.com/avrdelay.html
loop:  ldi  r18, 41
       ldi  r19, 150
       ldi  r20, 128
L1:    dec  r20       ; 128
       brne L1        ; 255  * (1 + 2)
       dec  r19       ;        150
       brne L1        ;        255  * (1 + 2)
       dec  r18       ;                41 * (1 + 2)
       brne L1        ;
       sbi  _SFR_IO_ADDR(PINB),5
       rjmp loop

```


Woa, the code now takes only 24 Bytes! Do you see additional chances to shorten the code again? The usage of timers ... yes, but this is another story.

> This diagram ist automatically generated by the LiaScript interpreter according to the data available in the following table. Take a view to the corresponding LiaScript code!

<!-- data-show data-type="barchart" -->
| Size         | Arduino | avrlibc | avrlibcOpt | Assembly |
| ------------ | -------:| -------:| ----------:| --------:|
| Size in Byte |     928 |     162 |         30 |       24 |

## Quiz

What is the larges disadvantage of the last solution?

    [[ ]] The Arduino code looks much better.
    [[X]] The code cannot be transfered to other microcontroller types
    [[?]] What about the used assembly commands?
    [[?]] ... do you expect to find them for each architecture?
***********************************************************************

You are right! If you compile the Arduino code for another controller board, no
problem. The avrlibc code and the assemble code can only be compiled for atmega
mikrocontrollers.

***********************************************************************
