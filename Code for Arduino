// Pin untuk motor dan encoder
const byte pin_pwm = 6;    // PWM motor
const byte pin_dir = 4;    // Arah motor (HIGH: maju, LOW: berhenti)
const byte pin_encoder = 2; // Sinyal encoder

// Variabel encoder
volatile int encoder_count = 0; // Hitungan encoder
double pv_speed = 0;            // Kecepatan motor (RPM)

// Variabel PID
double set_speed = 100;        // Kecepatan target (RPM)
double kp = 1.0, ki = 0.5, kd = 0.1; // Parameter PID
double e_speed = 0, e_speed_pre = 0, e_speed_sum = 0; // Error PID
double pwm_pulse = 0;          // Nilai PWM

// Timer
unsigned long prev_time = 0;  // Waktu sebelumnya untuk perhitungan PID
const unsigned long interval = 100; // Interval PID (ms)

// Status motor
bool motor_running = true;  // Status untuk motor berjalan atau berhenti

void setup() {
  pinMode(pin_pwm, OUTPUT);
  pinMode(pin_dir, OUTPUT);
  pinMode(pin_encoder, INPUT_PULLUP);

  attachInterrupt(digitalPinToInterrupt(pin_encoder), count_encoder, RISING);
  Serial.begin(9600);

  // Set motor bergerak ke kanan (maju)
  digitalWrite(pin_dir, HIGH); // Motor maju
}

void loop() {
  if (Serial.available()) {
    String input = Serial.readStringUntil('\n'); // Baca data hingga '\n'
    parseInput(input); // Proses data
  }

  if (motor_running) {
    unsigned long current_time = millis();
    if (current_time - prev_time >= interval) {
      prev_time = current_time;

      pv_speed = (encoder_count * 60.0) / 42.0; // Hitung RPM
      encoder_count = 0; // Reset hitungan encoder

      e_speed = set_speed - pv_speed;
      e_speed_sum += e_speed; 
      double d_speed = e_speed - e_speed_pre; 
      e_speed_pre = e_speed;

      pwm_pulse = (kp * e_speed) + (ki * e_speed_sum) + (kd * d_speed);
      pwm_pulse = constrain(pwm_pulse, 0, 255); 

      analogWrite(pin_pwm, pwm_pulse);

      Serial.print(pv_speed); 
      Serial.print(",");
      Serial.print(e_speed);   
      Serial.print(",");
      Serial.println(pwm_pulse);
    }
  } else {
    analogWrite(pin_pwm, 0); // Hentikan motor saat berhenti
  }
}

// Fungsi interrupt untuk menghitung encoder
void count_encoder() {
  encoder_count++;
}

// Fungsi untuk memproses data serial
void parseInput(String input) {
  input.trim();  // Hapus spasi atau newline tambahan

  if (input.equals("STOP")) {
    motor_running = false;
    Serial.println("Motor Stopped");
  } else if (input.equals("START")) {
    motor_running = true;
    Serial.println("Motor Running");
  } else {
    int firstComma = input.indexOf(',');
    int secondComma = input.indexOf(',', firstComma + 1);
    int thirdComma = input.indexOf(',', secondComma + 1);

    if (firstComma > 0 && secondComma > 0 && thirdComma > 0) {
      kp = input.substring(0, firstComma).toFloat();
      ki = input.substring(firstComma + 1, secondComma).toFloat();
      kd = input.substring(secondComma + 1, thirdComma).toFloat();
      set_speed = input.substring(thirdComma + 1).toFloat();

      Serial.print("Kp: "); Serial.print(kp);
      Serial.print(", Ki: "); Serial.print(ki);
      Serial.print(", Kd: "); Serial.print(kd);
      Serial.print(", Set Speed: "); Serial.println(set_speed);
    }
  }
}
