# Percobaan 1A - Membaca Sensor DHT22

## Tujuan

Percobaan ini bertujuan untuk mengetahui cara membaca suhu dan
kelembapan menggunakan sensor DHT22. Hasil pembacaan nantinya
ditampilkan melalui Serial Monitor pada Arduino IDE.

## Alat dan bahan

-   ESP32 atau board mikrokontroler yang digunakan saat praktikum
-   Sensor DHT22
-   Kabel jumper
-   Breadboard jika diperlukan
-   Kabel USB

## Library yang digunakan

-   DHT sensor library by Adafruit
-   Adafruit Unified Sensor

Library DHT digunakan supaya program bisa berkomunikasi dengan sensor.
Jika library belum dipasang, biasanya akan muncul error DHT.h: No such
file or directory saat program dikompilasi.

## Koneksi pin

-   DATA DHT22 ke GPIO 4
-   VCC ke sumber tegangan sesuai modul sensor
-   GND ke GND board

Jika menggunakan sensor DHT22 yang belum berbentuk modul, perhatikan
kebutuhan resistor pull-up pada pin data. Untuk modul sensor,
rangkaiannya bisa berbeda tergantung jenis modul yang digunakan.

## Penjelasan kode

### Memanggil library

#include <DHT.h\>

Bagian ini digunakan untuk memasukkan library DHT ke dalam program.
Library tersebut menyediakan fungsi untuk membaca data dari sensor.

### Menentukan pin dan tipe sensor

#define DHTPIN 4 
#define DHTTYPE DHT22

DHTPIN menunjukkan pin data sensor yang terhubung ke GPIO 4. DHTTYPE
digunakan untuk menentukan jenis sensor yang dipakai, yaitu DHT22.

### Membuat objek sensor

DHT dht(DHTPIN, DHTTYPE);

Baris ini membuat objek sensor dengan nama dht. Objek tersebut nantinya
dipakai untuk menjalankan fungsi pembacaan suhu dan kelembapan.

### Bagian setup

Pada fungsi setup, program menjalankan beberapa perintah:

-   Serial.begin(115200) untuk memulai komunikasi serial dengan baud
    rate 115200.
-   dht.begin() untuk memulai sensor DHT22.
-   Serial.println() untuk menampilkan pesan bahwa proses pembacaan
    sensor dimulai.

Fungsi setup hanya dijalankan satu kali saat board mulai menyala atau
di-reset.

### Membaca kelembapan dan suhu

float kelembaban = dht.
readHumidity(); 
float suhu =dht.
readTemperature();

readHumidity() digunakan untuk mengambil nilai kelembapan udara dalam
persen. Sementara readTemperature() digunakan untuk mengambil suhu dalam
derajat Celsius. Hasilnya disimpan ke variabel bertipe float karena
nilainya bisa memiliki angka di belakang koma.

### Memeriksa hasil pembacaan

Program menggunakan kondisi if dengan fungsi isnan(). Fungsi ini
memeriksa apakah nilai yang diterima bukan angka atau tidak valid.

Jika nilai suhu atau kelembapan tidak berhasil dibaca, program
menampilkan pesan gagal membaca data. Jika keduanya berhasil dibaca,
program masuk ke bagian else dan menampilkan hasilnya.

### Menampilkan data

Nilai suhu dan kelembapan ditampilkan menggunakan Serial.print() dan
Serial.println(). Serial.print() menampilkan teks atau nilai tanpa
langsung pindah ke baris berikutnya, sedangkan Serial.println()
menampilkan data lalu pindah baris.

### Jeda pembacaan

delay(2000) memberikan jeda selama 2000 milidetik atau 2 detik sebelum
program mengulangi pembacaan sensor. Jeda ini digunakan agar pembacaan
tidak dilakukan terlalu cepat.

## Alur kerja

1.  Board dinyalakan.
2.  Serial Monitor dan sensor DHT22 diinisialisasi.
3.  Program membaca kelembapan dan suhu.
4.  Program memeriksa apakah data berhasil dibaca.
5.  Jika gagal, pesan error ditampilkan.
6.  Jika berhasil, nilai suhu dan kelembapan ditampilkan.
7.  Program menunggu 2 detik lalu mengulangi proses.

## Contoh hasil pada Serial Monitor

Memulai akuisisi data sensor DHT22... Suhu: 28.50 °C, Kelembaban: 65.00
% Suhu: 28.60 °C, Kelembaban: 64.80 %

Angka di atas hanya contoh tampilan. Hasil sebenarnya akan mengikuti
kondisi suhu dan kelembapan di sekitar sensor.

------------------------------------------------------------------------

# Percobaan 2A - Mengendalikan Relay Berdasarkan Suhu

## Tujuan

Percobaan ini menggunakan sensor DHT22 untuk membaca suhu, kemudian
mengatur relay berdasarkan nilai suhu tersebut. Pada kode ini, relay
akan aktif jika suhu lebih dari 30 derajat Celsius. Jika suhu sama
dengan atau kurang dari 30 derajat, relay akan mati.

## Alat dan bahan

-   ESP32 atau board mikrokontroler yang digunakan saat praktikum
-   Sensor DHT22
-   Modul relay
-   Aktuator atau beban yang akan dikendalikan
-   Kabel jumper
-   Breadboard jika diperlukan
-   Kabel USB

## Library yang digunakan

-   DHT sensor library by Adafruit
-   Adafruit Unified Sensor

Library DHT digunakan untuk mengambil data suhu dari sensor. Untuk
mengendalikan relay tidak diperlukan library tambahan karena pin relay
dikontrol menggunakan fungsi GPIO bawaan Arduino.

## Koneksi pin

-   DATA DHT22 ke GPIO 4
-   IN relay ke GPIO 26
-   VCC dan GND sensor serta relay disambungkan sesuai kebutuhan modul
    dan board

Pastikan modul relay cocok dengan tegangan logika dari board yang
digunakan. Jangan langsung menghubungkan beban bertegangan tinggi ke pin
GPIO.

## Penjelasan kode

### Memanggil library dan menentukan pin

#include <DHT.h\>

#define DHTPIN 4 #define DHTTYPE DHT22 #define RELAYPIN 26

Library DHT dipanggil agar sensor bisa digunakan. DHTPIN menentukan pin
data sensor, DHTTYPE menentukan jenis sensor, sedangkan RELAYPIN
menentukan pin yang digunakan untuk mengontrol relay yaitu GPIO 26.

### Membuat objek sensor dan batas suhu

DHT dht(DHTPIN, DHTTYPE);

const float suhu
Threshold = 30.0;

Objek dht digunakan untuk membaca data dari sensor. Variabel
suhuThreshold berisi batas suhu yang digunakan sebagai patokan untuk
mengaktifkan relay. Nilainya dibuat 30.0 derajat Celsius dan menggunakan
const supaya nilainya tidak berubah selama program berjalan.

### Bagian setup

Pada bagian setup terdapat beberapa perintah:

-   Serial.begin(115200) untuk memulai komunikasi serial.
-   dht.begin() untuk menginisialisasi sensor.
-   pinMode(RELAYPIN, OUTPUT) untuk mengatur GPIO 26 sebagai output.
-   digitalWrite(RELAYPIN, LOW) untuk mengatur kondisi awal pin relay
    menjadi LOW.

Pada kode ini, LOW dianggap sebagai kondisi relay mati dan HIGH dianggap
sebagai kondisi relay aktif. Hal tersebut berlaku jika modul relay yang
digunakan memiliki logika active-high.

### Membaca suhu

float suhu = dht.readTemperature();

Perintah ini mengambil data suhu dari DHT22 dalam satuan Celsius.
Nilainya disimpan ke variabel suhu.

### Memeriksa data sensor

if (isnan(suhu))

Bagian ini memeriksa apakah pembacaan suhu gagal. Jika nilainya tidak
valid, program menampilkan pesan Gagal membaca data sensor. Jika
pembacaan berhasil, program melanjutkan ke bagian untuk menampilkan suhu
dan mengatur relay.

### Menampilkan suhu

Serial.print() digunakan untuk menampilkan nilai suhu pada Serial
Monitor. Setelah itu program menampilkan tanda panah sebagai pemisah
sebelum menampilkan status aktuator.

### Mengatur relay menggunakan if dan else

Pada bagian ini suhu dibandingkan dengan suhuThreshold.

-   Jika suhu lebih dari 30.0 derajat Celsius, pin relay diatur menjadi
    HIGH dan Serial Monitor menampilkan Aktuator: ON.
-   Jika suhu sama dengan atau kurang dari 30.0 derajat Celsius, pin
    relay diatur menjadi LOW dan Serial Monitor menampilkan Aktuator:
    OFF.

Karena kondisi yang digunakan adalah suhu \> suhuThreshold, relay belum
aktif ketika suhu tepat 30.0 derajat Celsius.

## Alur kerja

1.  Board dinyalakan dan program menjalankan setup.
2.  Sensor DHT22 diinisialisasi.
3.  GPIO 26 diatur sebagai output untuk relay.
4.  Kondisi awal relay diatur LOW.
5.  Program membaca suhu dari sensor.
6.  Program memeriksa apakah data suhu valid.
7.  Jika pembacaan gagal, pesan error ditampilkan.
8.  Jika berhasil, suhu ditampilkan pada Serial Monitor.
9.  Suhu dibandingkan dengan batas 30.0 derajat Celsius.
10. Relay menyala jika suhu lebih dari batas tersebut dan mati jika
    tidak.
11. Program mengulangi proses pembacaan dan pengendalian.

## Contoh hasil pada Serial Monitor

Jika suhu lebih dari 30 derajat:

Suhu: 31.20 °C -\> Aktuator: ON

Jika suhu 30 derajat atau kurang:

Suhu: 29.50 °C -\> Aktuator: OFF

Nilai di atas hanya contoh. Status relay yang sebenarnya juga bergantung
pada jenis modul relay yang digunakan.
