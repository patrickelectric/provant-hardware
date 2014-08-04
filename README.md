provant-hardware
================

#### Descrição ####

Repositório para projeto de Hardware (mecânica e eletrônica) do proVANT. 

#### Estrutura do repositório ####

A pasta _mechanics_ está estruturada em três subpastas:

- __drawings__ : Desenhos técnicos das partes modeladas.
- __images__ :  Renderizações e _prints_ de modelos completos e peças. 
- __models__ : Arquivos dos modelos 3D de partes e montagens.

Os modelos 3D foram desenvolvidos em sua maioria com o uso do CAD [_Solid Works_](www.solidworks.com).

Os esquemáticos e layouts das placas deste projeto foram desenvolvidas com software de EDA [_Eagle_](www.cadsoftusa.com/), e estão contidos na pasta _electronics_. Esta está estruturada em:

- __lib__ :  Contem bibliotecas de componentes exclusivos do projeto, bem como bibliotecas de terceiros (não padrões no Eagle) que foram utilizadas.
- __stm32-e407-shield__ : Contem esquemático/layout do shield desenvolvido para a placa __antiga__ utilizada no projeto, a [STM32-H407](https://www.olimex.com/Products/ARM/ST/STM32-H407/open-source-hardware). Esta pasta e seus conteúdos podem ser considerados __obsoletos__ .
- __stm32f4discovery-shield__ : Contem o esquemático/layout da placa breakout sendo utilizada atualmente no projeto, projetada para a [STM32F4-Discovery](http://www.st.com/web/catalog/tools/FM116/SC959/SS1532/PF252419?sc=internet/evalboard/product/252419.jsp). 