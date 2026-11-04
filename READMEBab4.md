## Kode 4A - Kontrol LED Menggunakan MQTT

### Tujuan program

Program 4A dibuat untuk mengontrol LED dari jarak jauh menggunakan MQTT.
ESP8266 akan terhubung ke WiFi dan broker MQTT, kemudian melakukan
subscribe ke topik perintah. Saat ada pesan masuk berupa JSON yang
berisi perintah ON atau OFF, ESP8266 akan mengubah kondisi LED sesuai
pesan tersebut.

### Library yang digunakan

1.  ESP8266WiFi.h digunakan untuk menghubungkan ESP8266 ke jaringan
    WiFi.
2.  PubSubClient.h digunakan untuk komunikasi MQTT, termasuk melakukan
    koneksi ke broker, subscribe topik, dan menerima pesan.
3.  ArduinoJson.h digunakan untuk membaca isi pesan yang dikirim dalam
    format JSON.

### Pengaturan koneksi dan pin

Jaringan WiFi diatur melalui variabel ssid dan password. Broker MQTT
yang digunakan adalah broker.hivemq.com dengan port 1883.

Topik yang digunakan untuk menerima perintah adalah
unsoed/tk245004/kelompok2/LEDting. Jadi aplikasi MQTT yang digunakan
untuk mengirim perintah harus memakai topik yang sama supaya pesan bisa
diterima oleh ESP8266.

LED dipasang pada GPIO5. Pada board NodeMCU, pin tersebut biasanya
dikenal sebagai D1. Variabel ledPin digunakan supaya pin LED lebih mudah
dipanggil pada bagian program.

### Penjelasan bagian program

Pada bagian awal dibuat objek WiFiClient dan PubSubClient. Objek
WiFiClient digunakan sebagai koneksi jaringan, lalu objek PubSubClient
memakai koneksi tersebut untuk berkomunikasi dengan broker MQTT.

Fungsi callback() merupakan bagian yang dijalankan ketika ada pesan baru
masuk dari topik yang sudah di-subscribe. Data pesan diterima dalam
bentuk array byte, sehingga program mengubahnya menjadi String dengan
melakukan perulangan dari awal sampai akhir payload.

Setelah pesan berhasil dibaca, program menampilkan nama topik dan isi
pesan ke Serial Monitor. Bagian ini berguna untuk melihat apakah
perintah dari aplikasi MQTT sudah sampai ke ESP8266 atau belum.

Pesan yang diterima kemudian diproses menggunakan deserializeJson().
Fungsi ini digunakan untuk mengubah teks JSON menjadi data yang bisa
dibaca oleh program. Jika format JSON salah, program akan menampilkan
pesan gagal parsing dan proses callback dihentikan.

Setelah JSON berhasil dibaca, program mengambil nilai dari bagian
perintah. Jika nilainya ON, pin LED diberi kondisi HIGH sehingga LED
menyala. Jika nilainya OFF, pin LED diberi kondisi LOW sehingga LED
mati. Kalau isi perintah selain dua nilai tersebut, program menampilkan
pesan bahwa perintah tidak dikenal.

Fungsi hubungkanWiFi() digunakan untuk menyambungkan ESP8266 ke jaringan
WiFi. Program akan terus mencoba sampai status koneksi berubah menjadi
WL_CONNECTED. Setelah berhasil, alamat IP perangkat juga ditampilkan
pada Serial Monitor.

Fungsi hubungkanMQTT() bertugas menghubungkan ESP8266 ke broker HiveMQ.
Program membuat client ID secara acak agar identitas klien tidak selalu
sama. Jika koneksi berhasil, ESP8266 langsung melakukan subscribe ke
topik perintah. Jika gagal, kode status MQTT ditampilkan dan program
mencoba kembali setelah jeda 2 detik.

Pada fungsi setup(), komunikasi serial dimulai dengan baud rate 115200.
Pin LED diatur sebagai OUTPUT dan kondisi awal LED dibuat mati
menggunakan digitalWrite(ledPin, LOW). Setelah itu program menghubungkan
WiFi, mengatur alamat broker, dan memasang fungsi callback agar pesan
MQTT bisa diproses.

Pada bagian loop(), program memeriksa koneksi WiFi dan MQTT. Jika WiFi
terputus, program mencoba menghubungkan kembali. Jika koneksi MQTT
terputus, fungsi hubungkanMQTT() dipanggil lagi. Fungsi client.loop()
harus terus dijalankan supaya ESP8266 bisa memproses pesan MQTT yang
masuk.

### Format pesan yang dikirim

Perintah dikirim dalam bentuk JSON. Contoh untuk menyalakan LED:

{ "perintah": "ON" }

Untuk mematikan LED:

{ "perintah": "OFF" }

Pesan tersebut dikirim ke topik unsoed/tk245004/kelompok2/LEDting.
Penulisan nama topik dan format JSON harus sesuai dengan program.

### Hasil yang diharapkan

Ketika ESP8266 berhasil terhubung ke WiFi dan broker MQTT, Serial
Monitor akan menampilkan informasi koneksi serta topik yang sudah
di-subscribe.

Saat pesan ON dikirim, LED pada GPIO5 akan menyala dan Serial Monitor
menampilkan keterangan Aktuator: ON. Saat pesan OFF dikirim, LED akan
mati dan muncul keterangan Aktuator: OFF.

## Kode 4B - Monitoring Suhu DHT11 dan Kontrol LED Menggunakan MQTT

### Tujuan program

Kode 4B merupakan pengembangan dari kode 4A. Pada program ini ESP8266
tidak hanya menerima perintah untuk mengendalikan LED, tetapi juga
membaca suhu dari sensor DHT11 dan mengirimkan hasil pembacaannya ke
broker MQTT.

Dengan begitu, sistem mempunyai dua fungsi. Pertama sebagai monitoring
suhu, dan kedua sebagai kontrol LED dari jarak jauh.

### Library yang digunakan

1.  ESP8266WiFi.h digunakan untuk menghubungkan ESP8266 ke jaringan
    WiFi.
2.  PubSubClient.h digunakan untuk komunikasi dengan broker MQTT.
3.  ArduinoJson.h digunakan untuk menyusun data suhu dan membaca pesan
    perintah dalam format JSON.
4.  DHT.h digunakan untuk membaca data dari sensor DHT11.

### Pengaturan pin dan topik MQTT

Sensor DHT11 menggunakan pin GPIO2. Pada board NodeMCU, pin tersebut
biasanya dikenal sebagai D4. Jenis sensor ditentukan melalui DHTTYPE
dengan nilai DHT11.

LED menggunakan GPIO5 atau biasanya pin D1 pada NodeMCU. Pin ini
digunakan sebagai output untuk mengendalikan LED.

Pada program ini ada dua topik MQTT yang digunakan, yaitu:

1.  unsoed/tk245004/kelompok2/Disc_1 digunakan untuk mengirim data suhu
    dari ESP8266 ke broker.
2.  unsoed/tk245004/kelompok2/Disc_2 digunakan untuk menerima perintah
    ON atau OFF dari aplikasi MQTT.

Pemisahan topik ini membuat data sensor dan perintah kontrol tidak
tercampur. Aplikasi yang ingin melihat suhu dapat melakukan subscribe ke
Disc_1, sedangkan perintah untuk LED dikirim ke Disc_2.

### Penjelasan bagian program

Pada awal program dibuat objek DHT dengan menentukan pin sensor dan
jenis sensor yang digunakan. Objek ini nantinya dipakai untuk membaca
nilai suhu dari DHT11.

Variabel waktuTerakhirPublish digunakan untuk menyimpan waktu terakhir
data dikirim. Sementara itu, intervalPublish diatur sebesar 5000
milidetik atau 5 detik. Penggunaan millis() membuat program bisa
mengatur jeda pengiriman tanpa harus menghentikan seluruh proses
menggunakan delay().

Fungsi callback() masih mempunyai fungsi yang mirip dengan kode 4A,
yaitu menerima pesan dari broker MQTT. Pesan yang masuk diubah menjadi
String, kemudian ditampilkan pada Serial Monitor.

Setelah itu pesan diproses menggunakan deserializeJson(). Jika format
JSON tidak sesuai, program menampilkan pesan gagal parsing. Jika
berhasil, program mengambil nilai perintah dari data JSON.

Apabila perintah yang diterima adalah ON, LED pada GPIO5 dinyalakan.
Jika perintahnya OFF, LED dimatikan. Dengan cara ini LED dapat
dikendalikan melalui aplikasi MQTT selama perangkat masih terhubung ke
broker.

Fungsi hubungkanWiFi() digunakan untuk menghubungkan perangkat ke
jaringan WiFi. Program akan terus mencoba sampai koneksi berhasil.
Setelah terhubung, alamat IP ESP8266 ditampilkan pada Serial Monitor.

Fungsi hubungkanMQTT() digunakan untuk membuat koneksi ke broker HiveMQ.
Jika koneksi berhasil, perangkat langsung melakukan subscribe ke topik
Disc_2. Artinya, ESP8266 akan menerima pesan perintah yang dikirim ke
topik tersebut. Jika koneksi gagal, program menampilkan kode status dan
mencoba kembali setelah 2 detik.

Pada fungsi setup(), Serial Monitor dimulai dengan baud rate 115200. Pin
LED diatur sebagai OUTPUT dan kondisi awalnya dibuat mati. Sensor DHT11
juga diaktifkan menggunakan dht.begin(). Setelah itu perangkat
dihubungkan ke WiFi, alamat broker MQTT diatur, dan callback dipasang
untuk menerima pesan.

Di dalam loop(), program memeriksa apakah koneksi MQTT masih aktif. Jika
terputus, program akan mencoba menghubungkan kembali. Fungsi
client.loop() dijalankan agar pesan dari broker dapat diterima dan
diproses.

Bagian berikutnya adalah proses pengiriman data suhu. Program memeriksa
selisih antara waktu sekarang dengan waktu terakhir pengiriman
menggunakan millis(). Jika sudah mencapai interval 5000 milidetik,
program akan membaca suhu dari sensor melalui dht.readTemperature().

Nilai suhu disimpan dalam variabel suhu. Program kemudian memeriksa
apakah hasil pembacaan bernilai NaN atau bukan angka. Jika pembacaan
gagal, Serial Monitor menampilkan pesan bahwa sensor DHT11 gagal dibaca
dan proses pada putaran loop tersebut dihentikan.

Jika pembacaan berhasil, nilai suhu dimasukkan ke dalam JsonDocument
dengan nama field suhu. Data kemudian diubah menjadi teks JSON
menggunakan serializeJson() dan disimpan ke buffer.

Setelah itu data dikirim menggunakan client.publish() ke topik Disc_1.
Informasi data yang berhasil diproses juga ditampilkan pada Serial
Monitor. Proses pembacaan dan pengiriman ini akan diulang setiap 5
detik.

### Format data dan perintah MQTT

Data suhu yang dikirim ke topik Disc_1 memiliki format JSON seperti
berikut:

{ "suhu": 28.5 }

Angka 28.5 hanya contoh hasil pembacaan. Nilai sebenarnya akan mengikuti
suhu yang terbaca oleh sensor DHT11.

Untuk mengendalikan LED, aplikasi MQTT mengirim pesan ke topik Disc_2.
Format pesan untuk menyalakan LED adalah:

{ "perintah": "ON" }

Sedangkan untuk mematikan LED:

{ "perintah": "OFF" }

Pastikan aplikasi MQTT melakukan subscribe ke topik Disc_1 untuk melihat
suhu. Untuk mengontrol LED, kirim pesan ke topik Disc_2.

### Hasil yang diharapkan

Saat program dijalankan, ESP8266 akan terhubung ke WiFi dan broker
HiveMQ. Setelah koneksi berhasil, perangkat melakukan subscribe ke topik
Disc_2.

Sensor DHT11 akan membaca suhu dan mengirimkannya ke topik Disc_1 setiap
5 detik. Data tersebut dapat dilihat melalui aplikasi MQTT Client yang
sudah melakukan subscribe ke topik tersebut.

Selain mengirim data suhu, ESP8266 juga bisa menerima perintah dari
topik Disc_2. Ketika perintah ON diterima, LED akan menyala. Ketika
perintah OFF diterima, LED akan mati.
