# Arduino-Car-Registrii
#include <avr\io.h>

float x, y;         // valorile de la axele joystick-ului

void setup() {
  DDRD = 0b11011000; // setare pini 3,4,6,7 ca si OUTPUT, restul INPUT
  DDRB = 0b00011110; // setare pini 9,10,11,12 ca si OUTPUT, restul INPUT

  TCCR1A = 0b10100000; // 
  TCCR1B = 0b00010010;

  ICR1 = 20000;  
}

void loop() {
  MotorSetup();
  Accelerometru();
}

void MotorSetup() {
  InitializareADC();
  x = ReadADC(0)/2 - 256;
  y = ReadADC(1)/2 - 256;
  int RealPowerY = (abs(y)-30)*73 + 3500;
  int RealPowerX = (abs(x)-30)*73 + 3500; 
  
  if(y < -30) {
    PORTD = 0b01001000;               // setarea directiei inapoi
    if(x < -30) {  
      OCR1A = RealPowerY/(-x/127+1);
      OCR1B = RealPowerY;             // inapoi-stanga 
    }
    else if(x > 30) {
      OCR1A = RealPowerY;             // inapoi-dreapta
      OCR1B = RealPowerY/(x/127+1);
    }
    else {
      OCR1A = RealPowerY;             // inapoi
      OCR1B = RealPowerY; 
    }
  }
  else if(y > 30) {
    PORTD = 0b10010000;         // setarea directiei inainte
    if(x < -30) {
      OCR1A = (RealPowerY/(-x/127+1))/2;  // inainte-stanga
      OCR1B = RealPowerY/2;
    }
    else if(x > 30) {
      OCR1A = RealPowerY/2;               // inainte-dreapta
      OCR1B = (RealPowerY/(x/127+1))/2;
    }
    else {
      OCR1A = RealPowerY/2;               // inainte
      OCR1B = RealPowerY/2;
    }
  }
  else if(x < -30) {
    PORTD = 0b01010000;         // setarea directiei in stanga
    OCR1A = RealPowerX;
    OCR1B = RealPowerX;
  }
  else if(x > 30) {
    PORTD = 0b10001000;         // setarea directiei in dreapta
    OCR1A = RealPowerX;
    OCR1B = RealPowerX;
  }
  else {
    PORTD = 0b00000000;         // Starea de repaos
  }
}

void Accelerometru() {
  InitializareADC();
  int x = ReadADC(2); 

  if((((float)x - 331.5)/65*9.8)>2.2&&(y<=0)) { // conditia pentru aprinderea ledurilor
    PORTB |= (1<<PORTB3) | (1<<PORTB4);
  }
  else {
    PORTB &= ~(1<<PORTB3) & ~(1<<PORTB4);
  }
} 

void InitializareADC() {
    // Selecteaza tensiunea de referinta Vref=AVcc
    ADMUX |= (1<<REFS0);
    //prescaleaza cu 128 si porneste ADC 
    ADCSRA |= (1<<ADPS2)|(1<<ADPS1)|(1<<ADPS0)|(1<<ADEN);    
}

uint16_t ReadADC(uint8_t canalADC) {
    //resetam bitii de canal la 0000
    ADMUX &= ADMUX & 0xF0;
    //adunam bitii de la canalul pe care il folosim
    ADMUX |= (canalADC & 0x0F);
    //pornim conversia adc
    ADCSRA |= (1<<ADSC);
    // asteapta pana cand conversia adc e gata
    while( ADCSRA & (1<<ADSC) );
   return ADC;
}
