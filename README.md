#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <RTClib.h>

// Adresses I2C des deux écrans LCD
LiquidCrystal_I2C lcd1(0x27, 16, 2); // Premier écran LCD (adresse 0x27)
LiquidCrystal_I2C lcd2(0x3F, 16, 2); // Deuxième écran LCD (adresse 0x3F)

// Pins
const int tempPin = A0;      // Pin du capteur de température LM35
const int potPin = A1;       // Pin du potentiomètre
const int motorPin = 3;      // Pin PWM pour le contrôle du moteur/ventilateur

RTC_DS3231 rtc;  // Initialisation de l'horloge temps réel (RTC)

void setup() {
  Serial.begin(9600);
  lcd1.begin(16, 2);
  lcd2.begin(16, 2);

  lcd1.backlight();
  lcd2.backlight();

  lcd2.setCursor(0, 0);
  lcd2.print("Bienvenue sur");
  lcd2.setCursor(0, 1);
  lcd2.print("   ANTAMIX   ");

  delay(2000); // Pause pour afficher le message de bienvenue

  if (!rtc.begin()) {
    Serial.println("RTC non trouvé !");
    while (1);
  }

  if (rtc.lostPower()) {
    rtc.adjust(DateTime(F(__DATE__), F(__TIME__))); // Initialisation si l'horloge perd sa synchronisation
  }

  pinMode(motorPin, OUTPUT);  // Configure la broche de moteur en sortie
}

void loop() {
  // Lecture de la température
  int tempReading = analogRead(tempPin);            // Lecture analogique du capteur de température
  float voltage = tempReading * (5.0 / 1023.0);     // Conversion en tension (en volts)
  float temperatureC = voltage * 100.0;             // Conversion en °C (LM35 : 10 mV/°C)

  // Lecture du potentiomètre pour régler la vitesse
  int potValue = analogRead(potPin);
  int motorSpeed = map(potValue, 0, 1023, 0, 255); // Conversion de 0-1023 en 0-255 pour PWM
  analogWrite(motorPin, motorSpeed);               // Ajuste la vitesse du moteur

  // Calcul de la vitesse en pourcentage
  float speedPercent = (potValue / 1023.0) * 100;

  // Lecture de l'heure depuis le module RTC
  DateTime now = rtc.now();
  int hours = now.hour();
  int minutes = now.minute();

  // Affichage sur le premier écran LCD
  lcd1.clear();
  lcd1.setCursor(0, 0);
  lcd1.print("Temp: ");
  lcd1.print(temperatureC);
  lcd1.print((char)223); // Symbole de degré
  lcd1.print("C");

  lcd1.setCursor(0, 1);
  lcd1.print("Heure: ");
  if (hours < 10) lcd1.print("0");  // Format heure 0-9
  lcd1.print(hours);
  lcd1.print(":");
  if (minutes < 10) lcd1.print("0");  // Format minute 0-9
  lcd1.print(minutes);

  lcd1.setCursor(10, 1);
  lcd1.print("Vit: ");
  lcd1.print((int)speedPercent);
  lcd1.print("%");

  delay(500); // Pause pour rafraîchir l'affichage
}
Explications
Initialisation des LCD : Deux instances de LiquidCrystal_I2C sont créées pour chaque écran avec des adresses I2C uniques (0x27 et 0x3F). Vérifiez les adresses exactes de vos écrans si nécessaire.

Affichage de bienvenue : Le texte "Bienvenue sur ANTAMIX" est affiché pendant 2 secondes sur le deuxième écran lors de l’initialisation.

Lecture de la température et de la vitesse : Le capteur LM35 mesure la température, et le potentiomètre ajuste la vitesse du moteur. La vitesse est convertie en pourcentage.

Affichage de l'heure, de la température et de la vitesse : Le premier écran affiche la température en °C, l'heure (formatée) et la vitesse du moteur en pourcentage.

Module RTC : Le module RTC DS3231 est utilisé pour obtenir l’heure actuelle.

Remarques
