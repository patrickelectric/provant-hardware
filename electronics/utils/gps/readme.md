Manual de montagem dos perifericos do GPS
====================================

Todos os esquematicos nesta pasta foram realizados utilizando o Eagle, boa parte das bibliotecas utilizadas foram extraidas da [bilbioteca da SparkFun](https://github.com/sparkfun/SparkFun-Eagle-Libraries). 

##### Instalacao de bilbiotecas para o Eagle

O procedimento de instalacao das bibliotecas para o Eagle consta [neste link](
https://learn.sparkfun.com/tutorials/how-to-install-and-setup-eagle/using-the-sparkfun-libraries).

##### Connector_cable
Nesta pasta se encontra o conector utilizado para ter acesso aos pinos para leitura de dados basicos do GPS OEMStar, sendo eles: RX,TX,VCC,GND.

##### Connector_protoboard
Nesta pasta se encontra o conector utilizado para adaptar o conector do GPS OEMStar a protoboard.

##### Arduino_mini

Esta pasta em questao consta o conversor das mensagens do GPS para o VANT.
As mensagens que sao enviadas em formato de String sao de dificil tratamento utilizando as bibliotecas basicas do ARM que estao sendo atualmente utizadas no projeto.
Podendo variar o tamanho, formato e tipo.
As informacoes sobre as mensagens e o GPS podem ser encontradas [aqui](http://www.novatel.com/assets/Documents/Bulletins/apn026.pdf).

Alguns exemplos de mensagens recebidas:
```
#BESTPOSA,COM1,0,93.0,UNKNOWN,0,90.000,004c0000,7145,12014;INSUFFICIENT_OBS,NONE,0.00000000000,0.00000000000,0.0000,0.0000,WGS84,0.0000,0.0000,0.0000,"",0.000,0.000,0,0,0,0,0,00,00,00*18de8d75
#BESTPOSA,COM1,0,91.5,COARSESTEERING,1847,221137.000,004c0000,7145,12014;INSUFFICIENT_OBS,NONE,0.00000000000,0.00000000000,0.0000,0.0000,WGS84,0.0000,0.0000,0.0000,"",0.000,0.000,0,0,0,0,0,00,00,00*43baf10b
#BESTPOSA,COM1,0,87.0,FINESTEERING,1847,221347.000,00440000,7145,12014;SOL_COMPUTED,SINGLE,-27.60022826035,-48.51710556420,-2.9004,0.4000,WGS84,22.9611,14.4112,31.3712,"",0.000,0.000,4,4,4,4,0
```

#### Sobre o GPS
Boa parte da documentacao do GPS OEMStar pode ser encontrado dentro do Dropbox do projeto.
> proVANT/birotor/components/electronics/datasheets/sensors/GPS/novatek_oemstar

##### Iniciando comunicacao
Primeiramente deve ser feito um dos conectores, podendo ser tanto o connector_protoboard quanto o connector_cable, pois ambos dao acesso aos pinos basicos ditos anteriormente.

Os pinos que deveram ser conectados sao: 1,2 como VCC; 10,13,16,18 como GND; 11 como TX; 12 como RX;

| Signal   | Behavior       | Descriptions                                                      | Pin |
|----------|----------------|-------------------------------------------------------------------|-----|
| LNA_PWR  | Input DC       | Power supply for external antenna                                 | 1   |
| Vin      | Input DC       | DC power supply for card                                          | 2   |
| USB d-   | Bi-derectional | USB interface data -                                              | 3   |
| USB d+   | Bi-directional | USB interface data +                                              | 4   |
| RESETIN  | See strobes    | Card reset                                                        | 5   |
| VARF     | output         | VARF                                                              | 6   |
| Reserved |                |                                                                   |     |
| Reserved |                |                                                                   |     |
| Event1   | Input          | Strobe, see strobes, default: Event1                              | 9   |
| GND      | Ground         | Digita Ground                                                     | 10  |
| COM1_Tx  | Output         | Transmitted Data for COM 1 output                                 | 11  |
| COM1_Rx  | Input          | Receved Data for COM 1 input                                      | 12  |
| GND      | Ground         | Digital Ground                                                    | 13  |
| COM2_Tx  | Output         | Transmitted Data for COM 2 output                                 | 14  |
| COM2_Rx  | Input          | Received Data for COM 2 input                                     | 15  |
| GND      | Ground         | Digital Ground                                                    | 16  |
| PV       | Output         | Output indicates 'good solution' or vallid FPS position when high | 17  |
| GND      | Ground         | Digital Gorund                                                    | 18  |
| PPS      | See strobes    | Pulse output synchronized to GPS Time                             | 19  |
|          |                |                                                                   |     |

Com estes pinos conectados em seus respectivos lugares, pode ser realizado os testes.

###### Software
Com os pinos conectados em um microcontrolador ou numa interface USB-serial, pode-se iniciar a comunicacao, com o baudrate de 9600, pedindo ao dispostivo a melhor posicao utilizando o comando:
```
LOG COM1 BESTPOSA ONTIME 1
```
Enquanto o GPS nao detectar a posicao, ele retornada a msg:
```
#BESTPOSA,COM1,0,93.0,UNKNOWN,0,90.000,004c0000,7145,12014;INSUFFICIENT_OBS,NONE,0.00000000000,0.00000000000,0.0000,0.0000,WGS84,0.0000,0.0000,0.0000,"",0.000,0.000,0,0,0,0,0,00,00,00*18de8d75
```

Um exemplo de codigo para ter uma comunicacao do GPS pode ser encontrado abaixo, este mesmo codigo pode ser utilizado para realizar a comunicacao do GPS com o VANT.
Este codigo foi utilizado para a placa arduino_mini, utilizando os pinos 10 e 11 para a comunicacao serial via software.

```
// all the information about the data can be obtained from: http://www.novatel.com/assets/Documents/Bulletins/apn026.pdf
// or on Dropbox: Geodetic Principles.pdf

// to start, send the command:
// log com1 bestposa ontime 1


#include <SoftwareSerial.h>

char * floatToString(char * outstr, float value, int places);

const int led = 13;
String started="[COM1]";
String bestposa="#BESTPOSA";
String unknown="UNKNOWN";
String coarsesteering="COARSESTEERING";
String finesteering="FINESTEERING";
String insufficient_obs="INSUFFICIENT_OBS";
String sol_computed="SOL_COMPUTED";

#define PROCESS_BESTPOSA      1
#define PROCESS_BESTPOSA_DATA 1
#define DEBUG                 1

//char  float_pos1[]={2,3}
//float Buffer[32]
bool SOL_COMPUTED_FLAG=0;
SoftwareSerial SoSerial(10, 11); // TX = digital pin 10, RX = digital pin 11

void setup() 
{
  delay(2000);
  // initialize serial:
  Serial.begin(9600);
  SoSerial.begin(9600);
  // make the pins outputs:
  pinMode(led, OUTPUT); 
}

void loop() 
{
  digitalWrite(led,HIGH);
  // if there's any serial available, read it:
  while (SoSerial.available() > 0)
  {
    digitalWrite(led,LOW);      // show that we received something
    char newlinearray[300]={};
    SoSerial.readBytesUntil('\n',newlinearray,300);
    String newline = newlinearray;
    
    if(DEBUG)
    {
      Serial.print("[R] >");
      Serial.print(newline);
      Serial.println("<");    // show what received
    }


    if(PROCESS_BESTPOSA && newline.length()>10)
    {
      // process bestposa
      int beginIdx = 0;
      int endIdx = newline.indexOf(";")-6;      // the last ',' of bestposa is ';' -6 char 
      int idx = newline.indexOf(",",beginIdx);  // take the first ','
      String arg[50];
      for(int i=0;i<50 && idx!=endIdx;i++)      // take all data
      {
        arg[i] = newline.substring(beginIdx, idx);
        beginIdx = idx + 1;
        idx = newline.indexOf(",", beginIdx);
        if(DEBUG)
        {
          Serial.print(i);
          Serial.print(" >");
          Serial.println(arg[i]);
        }
      }
    }

    if(DEBUG)
      Serial.println("###################");

    if(PROCESS_BESTPOSA_DATA && newline.length()>10)
    {
      // process data
      SOL_COMPUTED_FLAG = false;
      
      int beginIdx = newline.indexOf(";")+1;
      int endIdx = -1;
      int idx = newline.indexOf(",",beginIdx);
      String arg[50];
      
      for(int i=0;i<50 && idx!=endIdx;i++)
      {
        arg[i] = newline.substring(beginIdx, idx);
        beginIdx = idx + 1;
        idx = newline.indexOf(",", beginIdx);
        if(DEBUG)
        {
          Serial.print(i);
          Serial.print(" > ");
          Serial.println(arg[i]);
        }
      }
      if(arg[0] == sol_computed)
        SOL_COMPUTED_FLAG = true;

        if(SOL_COMPUTED_FLAG)
        {
          Serial.print("R");
          Serial.write(arg[2].length());
          Serial.write(arg[3].length());
          float heigh = arg[5].toFloat()+arg[4].toFloat();
          char buffer[25];
          String Heigh = floatToString(buffer, heigh, 5);
          Serial.write(Heigh.length());
          
          Serial.print(arg[2]);
          Serial.print(",");
          Serial.print(arg[3]);
          Serial.print(",");
          Serial.print(Heigh);
          
          Serial.print('\n');
        }
        else
        {
          Serial.print("R");
          Serial.write(13);
          Serial.write(13);
          Serial.write(7);
          Serial.print("0.00000000000");
          Serial.print(",");
          Serial.print("0.00000000000");
          Serial.print(",");
          Serial.print("0.00000");
          Serial.print('\n');
        }
    }   
    
  } // while serial avaiable

  // send data VANT->GPS
  while(Serial.available()>0)
  {
      SoSerial.write(Serial.read());
  }
      
}

char * floatToString(char * outstr, float value, int places)
{
   int minwidth=0;bool rightjustify=false;
    // this is used to write a float value to string, outstr.  oustr is also the return value.
    int digit;
    float tens = 0.1;
    int tenscount = 0;
    int i;
    float tempfloat = value;
    int c = 0;
    int charcount = 1;
    int extra = 0;
    // make sure we round properly. this could use pow from <math.h>, but doesn't seem worth the import
    // if this rounding step isn't here, the value  54.321 prints as 54.3209

    // calculate rounding term d:   0.5/pow(10,places)  
    float d = 0.5;
    if (value < 0)
        d *= -1.0;
    // divide by ten for each decimal place
    for (i = 0; i < places; i++)
        d/= 10.0;    
    // this small addition, combined with truncation will round our values properly 
    tempfloat +=  d;

    // first get value tens to be the large power of ten less than value    
    if (value < 0)
        tempfloat *= -1.0;
    while ((tens * 10.0) <= tempfloat) {
        tens *= 10.0;
        tenscount += 1;
    }

    if (tenscount > 0)
        charcount += tenscount;
    else
        charcount += 1;

    if (value < 0)
        charcount += 1;
    charcount += 1 + places;

    minwidth += 1; // both count the null final character
    if (minwidth > charcount){        
        extra = minwidth - charcount;
        charcount = minwidth;
    }

    if (extra > 0 and rightjustify) {
        for (int i = 0; i< extra; i++) {
            outstr[c++] = ' ';
        }
    }

    // write out the negative if needed
    if (value < 0)
        outstr[c++] = '-';

    if (tenscount == 0) 
        outstr[c++] = '0';

    for (i=0; i< tenscount; i++) {
        digit = (int) (tempfloat/tens);
        itoa(digit, &outstr[c++], 10);
        tempfloat = tempfloat - ((float)digit * tens);
        tens /= 10.0;
    }

    // if no places after decimal, stop now and return

    // otherwise, write the point and continue on
    if (places > 0)
    outstr[c++] = '.';


    // now write out each decimal place by shifting digits one by one into the ones place and writing the truncated value
    for (i = 0; i < places; i++) {
        tempfloat *= 10.0; 
        digit = (int) tempfloat;
        itoa(digit, &outstr[c++], 10);
        // once written, subtract off that digit
        tempfloat = tempfloat - (float) digit; 
    }
    if (extra > 0 and not rightjustify) {
        for (int i = 0; i< extra; i++) {
            outstr[c++] = ' ';
        }
    }


    outstr[c++] = '\0';
    return outstr;
}
```
