# Percobaan 2A - Menghubungkan ESP8266 ke WiFi

## Tujuan

Percobaan ini bertujuan untuk menghubungkan ESP8266 ke jaringan WiFi
yang sudah ada. Setelah berhasil terhubung, program akan menampilkan IP
Address, MAC Address, dan RSSI dari koneksi WiFi. LED juga digunakan
sebagai indikator bahwa perangkat berhasil terhubung.

## Alat dan bahan

-   Board ESP8266
-   LED bawaan board atau LED eksternal
-   Kabel USB
-   Komputer dengan Arduino IDE
-   Jaringan WiFi yang tersedia

## Library yang digunakan

Library yang digunakan adalah ESP8266WiFi.h. Library ini digunakan untuk
mengatur koneksi WiFi pada board ESP8266, seperti menghubungkan
perangkat ke router, memeriksa status koneksi, dan mengambil informasi
jaringan.

Pastikan board ESP8266 sudah dipilih pada pengaturan Board Arduino IDE.
Library ini biasanya sudah tersedia ketika paket board ESP8266 sudah
dipasang.

## Konfigurasi program

Pada kode terdapat beberapa bagian yang perlu diperhatikan:

-   SSID berisi nama jaringan WiFi yang akan dihubungkan.
-   Password berisi password dari jaringan WiFi tersebut.
-   ledPin diatur ke pin 2 sebagai pin LED.

Nama jaringan dan password harus disesuaikan dengan jaringan yang
digunakan saat praktikum. Jangan mengunggah password WiFi asli ke
repository GitHub yang bersifat public. Sebaiknya ganti dengan nilai
contoh atau gunakan konfigurasi lokal yang tidak ikut diunggah.

## Penjelasan kode

### Memanggil library

#include <ESP8266WiFi.h\>

Bagian ini digunakan untuk memasukkan library WiFi ESP8266. Dengan
library ini, program bisa menjalankan fungsi yang berhubungan dengan
koneksi WiFi.

### Menentukan SSID dan password

const char\* ssid = "nama_wifi";
const char\* password ="password_wifi";

Variabel ssid digunakan untuk menyimpan nama jaringan WiFi, sedangkan
password digunakan untuk menyimpan kata sandi jaringan. Nilainya harus
sesuai dengan jaringan yang ingin digunakan.

Pada file yang digunakan saat praktikum, bagian ini diisi dengan data
jaringan yang dipakai. Untuk dokumentasi yang diunggah ke GitHub,
sebaiknya jangan menampilkan password yang sebenarnya.

### Menentukan pin LED

const int ledPin = 2;

Variabel ledPin menentukan pin yang digunakan untuk mengendalikan LED.
Pada program ini pin yang dipakai adalah GPIO 2.

### Bagian setup

Pada fungsi setup, program menjalankan beberapa perintah awal:

-   Serial.begin(115200) memulai komunikasi serial dengan baud rate
    115200.
-   pinMode(ledPin, OUTPUT) mengatur pin LED sebagai output.
-   digitalWrite(ledPin, LOW) mengatur kondisi awal LED menjadi LOW.
-   WiFi.mode(WIFI_STA) mengatur ESP8266 ke mode Station.

Mode Station berarti ESP8266 bertindak sebagai perangkat yang terhubung
ke jaringan WiFi lain, seperti laptop atau smartphone yang terhubung ke
router.

### Memulai koneksi WiFi

WiFi.begin(ssid, password);

Perintah ini digunakan untuk memulai proses koneksi ke jaringan WiFi
berdasarkan SSID dan password yang sudah ditentukan.

Setelah itu program menampilkan tulisan menghubungkan ke WiFi. Bagian
while digunakan untuk terus memeriksa status koneksi selama perangkat
belum berhasil terhubung.

Di dalam while terdapat delay(500) yang memberikan jeda setengah detik.
Tanda titik juga ditampilkan setiap kali program masih menunggu koneksi.

### Menampilkan informasi WiFi

Jika koneksi berhasil, program akan menampilkan beberapa informasi:

-   WiFi.localIP() digunakan untuk mengambil IP Address yang diberikan
    oleh jaringan.
-   WiFi.macAddress() digunakan untuk menampilkan MAC Address dari
    ESP8266.
-   WiFi.RSSI() digunakan untuk mengetahui kekuatan sinyal WiFi dalam
    satuan dBm.

Informasi tersebut ditampilkan melalui Serial Monitor. Nilai RSSI
biasanya berupa angka negatif. Semakin mendekati nol, umumnya sinyal
yang diterima semakin kuat.

Setelah informasi ditampilkan, perintah digitalWrite(ledPin, HIGH)
dijalankan untuk menyalakan LED sebagai tanda bahwa koneksi berhasil.

### Bagian loop

Pada fungsi loop, program memeriksa status koneksi menggunakan
WiFi.status().

Jika statusnya WL_CONNECTED, program menampilkan tulisan Status:
Terhubung. Jika tidak, program menampilkan Status: Terputus dan mengatur
LED menjadi LOW.

Perintah delay(5000) memberikan jeda selama 5 detik sebelum status
koneksi diperiksa kembali.

Ada satu hal yang perlu diperhatikan dari kode ini. Jika WiFi terputus
setelah program masuk ke loop, LED akan dimatikan. Namun jika WiFi
tersambung kembali, bagian loop hanya menampilkan status terhubung dan
belum menyalakan LED lagi. Jadi LED bisa tetap mati walaupun koneksi
sudah kembali.

## Alur kerja

1.  ESP8266 dinyalakan.
2.  Serial Monitor dimulai dan pin LED diatur sebagai output.
3.  ESP8266 diatur ke mode Station.
4.  Program mencoba menghubungkan perangkat ke WiFi.
5.  Selama belum terhubung, program menampilkan tanda titik setiap
    setengah detik.
6.  Setelah berhasil terhubung, program menampilkan IP Address, MAC
    Address, dan RSSI.
7.  LED dinyalakan sebagai indikator koneksi berhasil.
8.  Program memeriksa status WiFi setiap 5 detik.
9.  Jika koneksi terputus, pesan terputus ditampilkan dan LED dimatikan.

## Contoh hasil pada Serial Monitor

menghubungkan ke WiFi.... WiFi berhasil terhubung! IP Adress :
192.168.1.10 MAC Address : XX:XX:XX:XX:XX:XX RSSI (dBm) : -55 Status:
Terhubung

Hasil IP Address, MAC Address, dan RSSI akan berbeda tergantung
perangkat dan jaringan yang digunakan. Contoh di atas hanya untuk
menunjukkan bentuk keluarannya.

------------------------------------------------------------------------

# Percobaan 2B - Membuat WiFi Access Point

## Tujuan

Percobaan ini bertujuan untuk membuat ESP8266 menjadi Access Point atau
pemancar jaringan WiFi. Dengan mode ini, ESP8266 menyediakan jaringan
sendiri yang bisa ditemukan dan dihubungkan oleh perangkat lain.

Program juga menampilkan nama jaringan, IP Address Access Point, serta
jumlah perangkat yang sedang terhubung.

## Alat dan bahan

-   Board ESP8266
-   Kabel USB
-   Komputer dengan Arduino IDE
-   Laptop atau smartphone untuk mencoba koneksi ke Access Point

## Library yang digunakan

Library yang digunakan adalah ESP8266WiFi.h. Library ini menyediakan
fungsi untuk mengatur mode Access Point, membuat jaringan WiFi,
mengambil IP Address, dan menghitung jumlah perangkat yang terhubung.

Catatan: Walaupun nama SSID pada kode menggunakan tulisan ESP32_Zaidan,
program tetap menggunakan library ESP8266WiFi.h. Jadi board yang
digunakan harus sesuai dengan library tersebut, yaitu ESP8266. Nama SSID
hanya nama jaringan dan tidak menentukan jenis board.

## Konfigurasi program

Pada kode terdapat dua variabel utama:

-   ap_ssid digunakan untuk menentukan nama jaringan WiFi yang dibuat
    oleh ESP8266.
-   ap_password digunakan untuk menentukan password agar perangkat lain
    bisa terhubung.

Password Access Point harus memenuhi ketentuan keamanan WiFi yang
digunakan. Untuk WPA2 pada fungsi softAP, password umumnya harus terdiri
dari minimal 8 karakter.

## Penjelasan kode

### Memanggil library

#include \<ESP8266WiFi.h\>

Library ini digunakan untuk menjalankan fitur WiFi pada ESP8266. Fungsi
seperti WiFi.mode(), WiFi.softAP(), dan WiFi.softAPIP() berasal dari
library ini.

### Menentukan nama dan password Access Point

const char* ap_ssid = "ESP32_Zaidan";
const char\* ap_password ="password_access_point";

ap_ssid digunakan sebagai nama jaringan yang akan muncul ketika
perangkat lain mencari WiFi. ap_password digunakan sebagai kata sandi
untuk menghubungkan perangkat ke jaringan tersebut.

Nama dan password dapat diganti sesuai kebutuhan praktikum. Jika
repository bersifat public, jangan menggunakan password pribadi atau
password jaringan yang masih digunakan.

### Bagian setup

Serial.begin(115200) digunakan untuk memulai komunikasi serial dengan
baud rate 115200.

WiFi.mode(WIFI_AP) mengatur ESP8266 ke mode Access Point. Dalam mode ini
ESP8266 membuat jaringan WiFi sendiri, bukan bergabung ke jaringan
router seperti pada percobaan 2A.

### Membuat Access Point

WiFi.softAP(ap_ssid, ap_password);

Perintah ini digunakan untuk mengaktifkan Access Point dengan nama dan
password yang sudah ditentukan. Setelah perintah berhasil dijalankan,
jaringan WiFi akan muncul pada daftar jaringan di laptop atau
smartphone.

### Mengambil IP Address

IPAddress apIP = WiFi.softAPIP();

Perintah ini digunakan untuk mengambil IP Address dari Access Point yang
dibuat. Alamat tersebut kemudian disimpan pada variabel apIP dan
ditampilkan melalui Serial Monitor.

IP Address ini dapat digunakan sebagai alamat jaringan ESP8266 ketika
perangkat lain sudah terhubung ke Access Point.

### Menampilkan informasi Access Point

Program menampilkan pesan Access Point aktif, nama SSID, dan IP Address.
Informasi ini membantu pengguna mengetahui bahwa jaringan sudah dibuat
dan mengetahui nama jaringan yang harus dicari pada perangkat lain.

### Bagian loop

Pada fungsi loop terdapat perintah:

int jumlahClient = WiFi.softAPgetStationNum();

Perintah tersebut digunakan untuk menghitung jumlah perangkat yang
sedang terhubung ke Access Point. Hasilnya disimpan ke variabel
jumlahClient.

Setelah itu jumlah perangkat ditampilkan melalui Serial Monitor.
Perintah delay(5000) memberikan jeda 5 detik sebelum program menghitung
jumlah perangkat lagi.

## Alur kerja

1.  ESP8266 dinyalakan.
2.  Komunikasi serial dimulai.
3.  ESP8266 diatur ke mode Access Point.
4.  Program membuat jaringan WiFi berdasarkan SSID dan password.
5.  Program mengambil IP Address Access Point.
6.  Nama jaringan dan IP Address ditampilkan pada Serial Monitor.
7.  Perangkat lain mencari nama WiFi yang dibuat oleh ESP8266.
8.  Setelah perangkat terhubung, program menghitung jumlah client setiap
    5 detik.

## Contoh hasil pada Serial Monitor

Access Point aktif! SSID : ESP32_Zaidan IP Address : 192.168.4.1 Jumlah
perangkat terhubung: 1

Alamat IP dan jumlah perangkat dapat berbeda tergantung pengaturan serta
kondisi saat pengujian. Angka di atas hanya contoh keluaran.

------------------------------------------------------------------------

# Perbedaan Percobaan 2A dan 2B

Percobaan 2A menggunakan mode Station, yaitu ESP8266 terhubung ke
jaringan WiFi yang sudah tersedia. Program mengambil informasi dari
jaringan tersebut dan menggunakan LED sebagai indikator koneksi.

Percobaan 2B menggunakan mode Access Point, yaitu ESP8266 membuat
jaringan WiFi sendiri. Perangkat lain dapat terhubung ke jaringan
tersebut dan jumlah perangkat yang terhubung dapat diperiksa melalui
program.

Jadi, perbedaan utamanya ada pada fungsi ESP8266 dalam jaringan. Pada
percobaan 2A ESP8266 menjadi client, sedangkan pada percobaan 2B ESP8266
menjadi penyedia jaringan atau Access Point.

------------------------------------------------------------------------

# Catatan dan kendala yang mungkin terjadi

## ESP8266 tidak bisa terhubung ke WiFi

Periksa kembali nama SSID dan password. Keduanya harus sesuai dengan
jaringan yang digunakan. Pastikan juga jaringan WiFi berada dalam
jangkauan perangkat dan menggunakan frekuensi yang didukung ESP8266,
yaitu 2.4 GHz.

## Access Point tidak muncul

Periksa apakah program berhasil dikompilasi dan diunggah ke board.
Pastikan mode yang digunakan adalah WIFI_AP dan perintah WiFi.softAP()
dijalankan. Coba cari jaringan dari perangkat lain yang mendukung WiFi
2.4 GHz.

## Serial Monitor tidak menampilkan informasi

Pastikan Serial Monitor menggunakan baud rate 115200, sama dengan nilai
pada Serial.begin(115200). Periksa juga port COM dan board yang dipilih
pada Arduino IDE.

## Jumlah client tidak berubah

Pastikan perangkat lain sudah benar-benar terhubung ke SSID Access Point
yang dibuat oleh ESP8266. Perangkat yang hanya melihat jaringan tetapi
belum tersambung tidak dihitung sebagai client.

## Catatan keamanan

SSID dan password yang ditulis langsung di dalam program dapat terlihat
oleh orang yang memiliki akses ke repository. Karena itu, jangan
mengunggah password WiFi pribadi atau password yang masih digunakan ke
repository public. Gunakan password khusus untuk praktikum dan ganti
kembali jika pernah terlanjur dipublikasikan.
