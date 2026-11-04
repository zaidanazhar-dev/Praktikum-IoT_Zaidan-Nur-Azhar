-   [Percobaan 1A --- Pembacaan Sensor
-     DHT22](#percobaan-1a--pembacaan-sensor-dht22)
    -   [Tujuan](#tujuan)
    -   [Komponen dan library](#komponen-dan-library)
    -   [Konfigurasi pin](#konfigurasi-pin)
    -   [Penjelasan program](#penjelasan-program)
    -   [Alur kerja](#alur-kerja)
    -   [Hasil yang diharapkan](#hasil-yang-diharapkan)
-   [Percobaan 2A --- Kendali Relay Berdasarkan
    Suhu](#percobaan-2a--kendali-relay-berdasarkan-suhu)
    -   [Tujuan](#tujuan-1)
    -   [Komponen dan library](#komponen-dan-library-1)
    -   [Konfigurasi pin dan ambang
        batas](#konfigurasi-pin-dan-ambang-batas)
    -   [Penjelasan program](#penjelasan-program-1)
    -   [Alur kerja](#alur-kerja-1)
    -   [Hasil yang diharapkan](#hasil-yang-diharapkan-1)
-   [Catatan implementasi dan
    troubleshooting](#catatan-implementasi-dan-troubleshooting)

------------------------------------------------------------------------

# Percobaan 1A --- Pembacaan Sensor DHT22

## Tujuan

Membuat program untuk membaca nilai suhu dan kelembapan udara
menggunakan sensor DHT22, kemudian menampilkan hasil pembacaannya
melalui Serial Monitor.

## Komponen dan library

**Komponen:** - Board mikrokontroler yang digunakan pada praktikum
(misalnya ESP32). - Sensor DHT22. - Kabel jumper dan breadboard. - Kabel
USB untuk menghubungkan board ke komputer.

**Library:** - `DHT.h` --- library untuk mengakses sensor DHT22. Pada
Arduino IDE, library ini umumnya tersedia melalui **DHT sensor library
by Adafruit**. - `Adafruit Unified Sensor` --- dependency yang
diperlukan oleh versi library DHT Adafruit.

Pastikan library sudah terpasang sebelum melakukan kompilasi program.

## Konfigurasi pin

  Perangkat                             Pin pada program Keterangan
  ------------ ----------------------------------------- -------------------
  DHT22 DATA                                      GPIO 4 Jalur data sensor
  DHT22 VCC      3.3 V atau 5 V sesuai spesifikasi modul Catu daya
  DHT22 GND                                          GND Ground

Jika menggunakan sensor DHT22 versi bare sensor (bukan modul),
perhatikan kebutuhan resistor pull-up pada jalur DATA sesuai datasheet
atau rangkaian praktikum.

## Penjelasan program

### 1. Memanggil library

``` cpp
#include <DHT.h>
```

Memasukkan library DHT agar program dapat menggunakan fungsi untuk
menginisialisasi sensor serta membaca suhu dan kelembapan.

### 2. Menentukan pin dan tipe sensor

``` cpp
#define DHTPIN 4
#define DHTTYPE DHT22
```

-   `DHTPIN` menentukan GPIO yang terhubung ke pin DATA sensor, yaitu
    GPIO 4.
-   `DHTTYPE` menentukan jenis sensor yang digunakan, yaitu DHT22.

### 3. Membuat objek sensor

``` cpp
DHT dht(DHTPIN, DHTTYPE);
```

Membuat objek bernama `dht` dengan konfigurasi pin dan tipe sensor yang
telah ditentukan. Objek ini digunakan untuk memanggil fungsi-fungsi dari
library DHT.

### 4. Fungsi `setup()`

``` cpp
void setup() {
  Serial.begin(115200);
  dht.begin();
  Serial.println("Memulai akuisisi data sensor DHT22...");
}
```

Fungsi `setup()` dijalankan satu kali ketika board mulai bekerja.

-   `Serial.begin(115200)` mengaktifkan komunikasi serial dengan baud
    rate 115200.
-   `dht.begin()` menginisialisasi sensor DHT22.
-   `Serial.println()` menampilkan pesan awal pada Serial Monitor.

Atur baud rate Serial Monitor menjadi **115200** agar teks terbaca
dengan benar.

### 5. Membaca kelembapan dan suhu

``` cpp
float kelembaban = dht.readHumidity();
float suhu = dht.readTemperature();
```

-   `readHumidity()` membaca kelembapan relatif dalam persen (%).
-   `readTemperature()` membaca suhu dalam derajat Celsius (°C).
-   Tipe data `float` digunakan agar nilai dapat menyimpan angka
    desimal.

### 6. Memeriksa kegagalan pembacaan

``` cpp
if (isnan(kelembaban) || isnan(suhu)) {
  Serial.println("Gagal membaca data dari sensor DHT22!");
}
```

`isnan()` memeriksa apakah nilai yang diterima bukan angka (*Not a
Number*). Kondisi ini dapat terjadi ketika pembacaan sensor gagal.
Operator `||` berarti pesan kegagalan ditampilkan jika pembacaan
kelembapan **atau** suhu tidak valid.

### 7. Menampilkan hasil pembacaan

``` cpp
else {
  Serial.print("Suhu: ");
  Serial.print(suhu);
  Serial.print(" °C, Kelembaban: ");
  Serial.print(kelembaban);
  Serial.println(" %");
}
```

Jika kedua nilai berhasil dibaca, program menampilkan suhu dan
kelembapan pada Serial Monitor.

### 8. Jeda pembacaan

``` cpp
delay(2000);
```

Memberikan jeda 2000 milidetik (2 detik) sebelum siklus pembacaan
berikutnya. Jeda ini sesuai dengan interval pembacaan yang umum
digunakan untuk DHT22.

## Alur kerja

1.  Board dinyalakan dan menjalankan `setup()`.
2.  Komunikasi serial dimulai pada baud rate 115200.
3.  Sensor DHT22 diinisialisasi.
4.  Program membaca kelembapan dan suhu.
5.  Program memeriksa apakah hasil pembacaan valid.
6.  Jika pembacaan gagal, pesan kesalahan ditampilkan.
7.  Jika pembacaan berhasil, suhu dan kelembapan ditampilkan.
8.  Program menunggu 2 detik, kemudian mengulangi proses.

## Hasil yang diharapkan

Serial Monitor menampilkan pesan awal, kemudian data suhu dan kelembapan
secara berkala. Contoh format keluaran:

``` text
Memulai akuisisi data sensor DHT22...
Suhu: 28.50 °C, Kelembaban: 65.00 %
Suhu: 28.60 °C, Kelembaban: 64.80 %
```

Angka di atas hanya contoh format keluaran, bukan hasil pengukuran
praktikum.

------------------------------------------------------------------------

# Percobaan 2A --- Kendali Relay Berdasarkan Suhu

## Tujuan

Membuat sistem kendali sederhana yang membaca suhu dari sensor DHT22 dan
mengaktifkan atau menonaktifkan relay berdasarkan ambang batas suhu
(*temperature threshold*) sebesar 30 °C.

Pada program ini, relay digunakan sebagai keluaran digital untuk
mengendalikan aktuator atau beban yang terhubung melalui modul relay.

## Komponen dan library

**Komponen:** - Board mikrokontroler yang digunakan pada praktikum
(misalnya ESP32). - Sensor DHT22. - Modul relay yang kompatibel dengan
tegangan logika board. - Aktuator atau beban yang sesuai dengan
spesifikasi modul relay. - Kabel jumper dan breadboard. - Kabel USB
untuk menghubungkan board ke komputer.

**Library:** - `DHT.h` --- untuk membaca data dari sensor DHT22. -
`Adafruit Unified Sensor` --- dependency untuk library DHT Adafruit.

Program ini tidak memerlukan library relay khusus karena relay
dikendalikan menggunakan fungsi GPIO bawaan Arduino, yaitu `pinMode()`
dan `digitalWrite()`.

## Konfigurasi pin dan ambang batas

  Perangkat/parameter       Nilai Keterangan
  --------------------- --------- --------------------------------
  DHT22 DATA               GPIO 4 Jalur data sensor
  Relay IN                GPIO 26 Sinyal kendali relay
  Ambang batas suhu       30.0 °C Batas untuk mengaktifkan relay
  Baud rate serial         115200 Kecepatan komunikasi serial

**Catatan:** Program mengasumsikan relay aktif ketika pin GPIO bernilai
`HIGH` (*active-high*). Beberapa modul relay justru menggunakan logika
*active-low*, sehingga perilakunya perlu disesuaikan dengan spesifikasi
modul.

## Penjelasan program

### 1. Memanggil library

``` cpp
#include <DHT.h>
```

Memasukkan library DHT agar program dapat membaca suhu dari sensor
DHT22.

### 2. Menentukan pin dan tipe sensor

``` cpp
#define DHTPIN 4
#define DHTTYPE DHT22
#define RELAYPIN 26
```

-   `DHTPIN` menentukan GPIO 4 sebagai jalur data sensor.
-   `DHTTYPE` menentukan jenis sensor, yaitu DHT22.
-   `RELAYPIN` menentukan GPIO 26 sebagai pin keluaran untuk
    mengendalikan relay.

### 3. Membuat objek sensor dan ambang batas

``` cpp
DHT dht(DHTPIN, DHTTYPE);

const float suhuThreshold = 30.0;
```

Objek `dht` digunakan untuk mengakses sensor. Variabel `suhuThreshold`
merupakan konstanta bertipe `float` yang menetapkan ambang batas suhu
sebesar 30.0 °C.

### 4. Fungsi `setup()`

``` cpp
void setup() {
  Serial.begin(115200);
  dht.begin();
  pinMode(RELAYPIN, OUTPUT);
  digitalWrite(RELAYPIN, LOW);
}
```

Fungsi `setup()` dijalankan satu kali ketika board mulai bekerja.

-   `Serial.begin(115200)` mengaktifkan komunikasi serial.
-   `dht.begin()` menginisialisasi sensor DHT22.
-   `pinMode(RELAYPIN, OUTPUT)` mengatur GPIO 26 sebagai pin keluaran.
-   `digitalWrite(RELAYPIN, LOW)` mengatur kondisi awal pin relay
    menjadi LOW. Dengan asumsi relay active-high, kondisi ini berarti
    relay OFF.

### 5. Membaca suhu

``` cpp
float suhu = dht.readTemperature();
```

Membaca suhu dari sensor DHT22 dalam satuan derajat Celsius dan
menyimpannya pada variabel `suhu`.

### 6. Memeriksa kegagalan pembacaan

``` cpp
if (isnan(suhu)) {
  Serial.println("Gagal membaca data sensor!");
}
```

Memeriksa apakah nilai suhu tidak valid. Jika pembacaan gagal, program
menampilkan pesan kesalahan dan tidak menjalankan percabangan kendali
suhu pada siklus tersebut.

### 7. Menampilkan suhu

``` cpp
Serial.print("Suhu: ");
Serial.print(suhu);
Serial.print(" °C -> ");
```

Menampilkan nilai suhu yang berhasil dibaca sebelum menampilkan status
aktuator.

### 8. Mengendalikan relay menggunakan conditional

``` cpp
if (suhu > suhuThreshold) {
  digitalWrite(RELAYPIN, HIGH);
  Serial.println("Aktuator: ON");
} else {
  digitalWrite(RELAYPIN, LOW);
  Serial.println("Aktuator: OFF");
}
```

Program menggunakan percabangan `if-else` untuk membandingkan suhu
dengan ambang batas 30.0 °C.

-   Jika `suhu > 30.0`, pin relay diatur menjadi HIGH dan status
    aktuator ditampilkan sebagai ON.
-   Jika `suhu <= 30.0`, pin relay diatur menjadi LOW dan status
    aktuator ditampilkan sebagai OFF.

Dengan demikian, suhu yang tepat **30.0 °C** belum mengaktifkan relay
karena operator yang digunakan adalah `>` dan bukan `>=`.

## Alur kerja

1.  Board dinyalakan dan menjalankan `setup()`.
2.  Komunikasi serial dan sensor DHT22 diinisialisasi.
3.  GPIO 26 diatur sebagai OUTPUT.
4.  Relay diatur ke kondisi awal LOW.
5.  Program membaca suhu dari DHT22.
6.  Program memeriksa validitas data suhu.
7.  Jika pembacaan gagal, pesan kesalahan ditampilkan.
8.  Jika pembacaan berhasil, suhu dibandingkan dengan ambang batas 30.0
    °C.
9.  Jika suhu lebih dari 30.0 °C, relay diaktifkan.
10. Jika suhu 30.0 °C atau lebih rendah, relay dinonaktifkan.
11. Proses pembacaan dan pengendalian diulangi.

## Hasil yang diharapkan

Serial Monitor menampilkan suhu dan status aktuator berdasarkan hasil
pembacaan sensor.

Contoh ketika suhu di atas ambang batas:

``` text
Suhu: 31.20 °C -> Aktuator: ON
```

Contoh ketika suhu sama dengan atau di bawah ambang batas:

``` text
Suhu: 29.50 °C -> Aktuator: OFF
```

Angka di atas hanya contoh format keluaran, bukan hasil pengukuran
praktikum.

------------------------------------------------------------------------

# Catatan implementasi dan troubleshooting

## 1. Library tidak ditemukan

Jika muncul error seperti:

``` text
fatal error: DHT.h: No such file or directory
```

Pasang **DHT sensor library by Adafruit** melalui Library Manager
Arduino IDE. Pastikan dependency **Adafruit Unified Sensor** juga sudah
terpasang.

## 2. Sensor gagal dibaca

Jika Serial Monitor menampilkan pesan kegagalan pembacaan:

-   Periksa koneksi VCC, GND, dan DATA.
-   Pastikan pin DATA terhubung ke GPIO yang sesuai dengan definisi
    program.
-   Pastikan tipe sensor telah ditentukan sebagai `DHT22`.
-   Periksa kebutuhan resistor pull-up jika menggunakan sensor DHT22
    tanpa modul.
-   Pastikan interval pembacaan tidak terlalu cepat.

## 3. Relay tidak bekerja sesuai kondisi suhu

-   Pastikan kabel sinyal relay terhubung ke GPIO 26.
-   Pastikan modul relay kompatibel dengan level tegangan GPIO board.
-   Periksa apakah modul menggunakan logika active-high atau active-low.
-   Pastikan catu daya relay dan aktuator sesuai spesifikasi perangkat.

**Keselamatan:** Jangan menghubungkan beban tegangan AC atau tegangan
tinggi secara langsung ke GPIO mikrokontroler. Gunakan modul relay yang
sesuai, lakukan isolasi dan pengawatan dengan benar, serta minta
pengawasan praktikan/dosen yang kompeten jika menggunakan tegangan
berbahaya.

## 4. Catatan pada kode Percobaan 2A

Potongan kode Percobaan 2A yang disertakan perlu diperiksa sebelum
dikompilasi. Pada bagian akhir yang dikirim, penutup blok fungsi
`loop()` belum terlihat. Pastikan terdapat kurung kurawal penutup `}`
untuk seluruh blok yang dibuka.

Selain itu, kode Percobaan 2A yang diberikan belum memiliki `delay()`.
Akibatnya, pembacaan sensor dan pengendalian relay akan diulang secepat
siklus program berjalan. Jika ingin menggunakan interval pembacaan 2
detik seperti Percobaan 1A, tambahkan `delay(2000);` di akhir fungsi
`loop()` setelah seluruh percabangan selesai. Penambahan ini merupakan
penyesuaian program, bukan bagian dari potongan kode awal.
