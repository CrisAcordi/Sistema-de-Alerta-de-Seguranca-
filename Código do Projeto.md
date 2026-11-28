\#include \<Wire.h\>  
\#include \<LiquidCrystal\_I2C.h\>  
\#include \<Servo.h\>

LiquidCrystal\_I2C lcd(0x20, 16, 2); // padrão do Tinkercad (32 \= 0x20)  
Servo trava;

const int pinoLDR \= A0;  
const int pinoBuzzer \= 5;   // fio roxo  
const int pinoBotao \= 2;    // botão  
const int pinoServo \= 9;    // servo (fio laranja)

// AJUSTE AQUI: Valor para detectar a lanterna do celular  
// Teste no Monitor Serial: a lanterna deve dar valores \> 800  
// Quando você tira a lanterna, o valor cai para \< 300 (dependendo da luz ambiente)  
const int limiarLuz \= 700;  

bool alarmeAtivo \= false;  
bool sistemaArmado \= true; // Controla se o sistema está vigiando ou desativado

void mostrar(const char\* msg) {  
  lcd.clear();  
  lcd.setCursor(0, 0);  
  lcd.print(msg);  
}

void setup() {  
  lcd.init();  
  lcd.backlight();  
  pinMode(pinoBuzzer, OUTPUT);  
  pinMode(pinoBotao, INPUT\_PULLUP);  
  trava.attach(pinoServo);  
  trava.write(0);            // porta aberta (inicial)  
  Serial.begin(9600);  
  mostrar("SISTEMA ARMADO:");  
}

void loop() {  
  int valorLDR \= analogRead(pinoLDR);  
  Serial.print("Luz: ");  
  Serial.println(valorLDR);

  // \--- Lógica do Botão (Desarma/Arma o sistema) \---  
  if (digitalRead(pinoBotao) \== LOW) {  
    delay(50); // Debounce  
    if (digitalRead(pinoBotao) \== LOW) {  
      sistemaArmado \= \!sistemaArmado; // Inverte o estado (Liga/Desliga)  
        
      if (\!sistemaArmado) {  
        // Se desarmou, reseta tudo  
        alarmeAtivo \= false;  
        noTone(pinoBuzzer);  
        trava.write(0); // Destranca  
        mostrar("SYSTEM OFF");  
        Serial.println("Sistema Desativado pelo Botao");  
      } else {  
        // Se armou de novo  
        mostrar("SISTEMA ARMADO:");  
        Serial.println("Sistema Armado");  
      }  
        
      // Espera o botão ser solto para não ficar alternando rápido  
      while(digitalRead(pinoBotao) \== LOW);   
      delay(500);   
    }  
  }

  // \--- Lógica do Alarme (Só roda se estiver ARMADO) \---  
  if (sistemaArmado) {  
      
    // 1\. Dispara o alarme se a luz sumir (abaixo do limiar)  
    if (\!alarmeAtivo && valorLDR \< limiarLuz) {  
      alarmeAtivo \= true;  
      tone(pinoBuzzer, 1000);  
      trava.write(90);         // Tranca a porta  
      mostrar("ALARME DISPARADO\!");  
      Serial.println("ALARME DISPARADO\!");  
    }

    // 2\. Só reseta o alarme se a luz VOLTAR (acima do limiar)  
    // O botão não reseta o alarme, apenas desativa o sistema inteiro.  
    if (alarmeAtivo && valorLDR \>= limiarLuz) {  
      alarmeAtivo \= false;  
      noTone(pinoBuzzer);  
      trava.write(0);          // Destranca  
      mostrar("SISTEMA ARMADO:");  
      Serial.println("Alarme Resetado pela Luz");  
      delay(500);  
    }  
  }

  delay(100);  
}

