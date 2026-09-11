# mecanicas-
// Medición de velocidad en MRU con 2 sensores HC-SR04
// v = d / t

const int trigPin1 = 9,  echoPin1 = 10;
const int trigPin2 = 11, echoPin2 = 12;

const float distanciaSensores = 0.50; // metros (AJUSTAR a tu montaje)
const float umbralDeteccion = 15.0;   // cm: distancia por debajo de la cual se considera "objeto presente"

unsigned long tiempoSensor1 = 0;
unsigned long tiempoSensor2 = 0;
bool detectado1 = false;
bool detectado2 = false;

float medirDistanciaCM(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  long duracion = pulseIn(echoPin, HIGH, 30000); // timeout 30 ms
  if (duracion == 0) return 999; // sin eco = "vacío"
  return duracion * 0.0343 / 2.0; // cm
}

void setup() {
  Serial.begin(9600);
  pinMode(trigPin1, OUTPUT); pinMode(echoPin1, INPUT);
  pinMode(trigPin2, OUTPUT); pinMode(echoPin2, INPUT);
  Serial.println("Sistema listo. Distancia entre sensores: " + String(distanciaSensores) + " m");
}

void loop() {
  float d1 = medirDistanciaCM(trigPin1, echoPin1);
  float d2 = medirDistanciaCM(trigPin2, echoPin2);

  // Detección en sensor 1
  if (d1 < umbralDeteccion && !detectado1) {
    detectado1 = true;
    tiempoSensor1 = micros();
    Serial.println("Objeto detectado en Sensor 1");
  }

  // Detección en sensor 2 (solo cuenta si ya pasó por el 1)
  if (d2 < umbralDeteccion && detectado1 && !detectado2) {
    detectado2 = true;
    tiempoSensor2 = micros();

    float deltaT = (tiempoSensor2 - tiempoSensor1) / 1000000.0; // segundos
    float velocidad = distanciaSensores / deltaT;

    Serial.println("Objeto detectado en Sensor 2");
    Serial.print("Tiempo transcurrido: "); Serial.print(deltaT, 4); Serial.println(" s");
    Serial.print("Velocidad calculada: "); Serial.print(velocidad, 3); Serial.println(" m/s");
    Serial.println("--------------------------------------");

    // Reiniciar para la siguiente medición
    delay(1500);
    detectado1 = false;
    detectado2 = false;
  }

  delay(20); // pequeña pausa entre lecturas
}