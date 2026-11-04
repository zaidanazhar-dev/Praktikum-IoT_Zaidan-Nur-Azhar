## Kode 3A - Pengiriman Data Menggunakan HTTP

### Tujuan

Kode 3A digunakan untuk mengirim data dalam format JSON ke server
menggunakan metode HTTP POST. Server yang dipakai pada percobaan ini
adalah httpbin.org dengan alamat endpoint https://httpbin.org/post.

### Library yang digunakan

1.  ESP8266WiFi.h digunakan untuk menghubungkan board ESP8266 ke
    jaringan WiFi.
2.  ESP8266HTTPClient.h digunakan untuk melakukan komunikasi HTTP,
    termasuk mengirim request POST.
3.  WiFiClientSecureBearSSL.h digunakan untuk membuat koneksi HTTPS yang
    aman melalui BearSSL.
4.  ArduinoJson.h digunakan untuk menyusun data dalam format JSON.

### Cara kerja program

Pertama program melakukan inisialisasi komunikasi serial dengan
kecepatan 115200 baud. Setelah itu ESP8266 mulai mencoba terhubung ke
WiFi menggunakan nama jaringan dan password yang sudah ditentukan pada
program.

Selama belum terhubung, program akan menampilkan titik pada Serial
Monitor setiap 500 milidetik. Kalau koneksi berhasil, akan muncul pesan
bahwa WiFi berhasil terhubung.

Pada bagian loop, program memeriksa status koneksi WiFi terlebih dahulu.
Jika masih terhubung, program membuat objek WiFiClientSecure untuk
digunakan pada koneksi HTTPS. Pengaturan setInsecure digunakan agar
sertifikat server tidak diperiksa. Cara ini memudahkan percobaan, tetapi
tidak disarankan untuk penggunaan yang membutuhkan keamanan sungguhan.

Selanjutnya program membuat objek HTTPClient dan memulai koneksi ke
alamat server. Header Content-Type diatur menjadi application/json
karena data yang akan dikirim berbentuk JSON.

Data suhu dan kelembaban dimasukkan ke dalam objek JsonDocument. Setelah
itu data diubah menjadi teks JSON menggunakan serializeJson. Hasilnya
ditampilkan pada Serial Monitor sebelum dikirim ke server.

Pengiriman dilakukan dengan metode POST. Jika server memberikan kode
response positif, program menampilkan kode HTTP dan isi response dari
server. Jika pengiriman gagal, program menampilkan kode error. Setelah
proses selesai, koneksi HTTP ditutup menggunakan http.end().

Program menunggu selama 10 detik sebelum mengulangi proses pengiriman
berikutnya.

### Hasil yang diharapkan

Saat program berjalan, Serial Monitor akan menampilkan proses koneksi
WiFi, data JSON yang dikirim, kode response HTTP, dan isi response dari
httpbin.org. Jika koneksi berhasil, biasanya kode response yang diterima
adalah 200.

Contoh isi data yang dikirim adalah JSON dengan nilai suhu 28.5 dan
kelembaban 65.0. Nilai tersebut hanya data tetap yang ditulis langsung
pada program.

## Kode 3B - Pengiriman Data Menggunakan MQTT

### Tujuan

Kode 3B digunakan untuk mengirim data dalam format JSON menggunakan
protokol MQTT. Pengiriman dilakukan melalui broker publik HiveMQ pada
alamat broker.hivemq.com dengan port 1883.

### Library yang digunakan

1.  ESP8266WiFi.h digunakan untuk menghubungkan ESP8266 ke jaringan
    WiFi.
2.  PubSubClient.h digunakan untuk komunikasi MQTT antara ESP8266 dan
    broker.
3.  ArduinoJson.h digunakan untuk membuat data dalam format JSON.

### Pengaturan MQTT

Broker yang digunakan adalah broker.hivemq.com. Port yang digunakan
yaitu 1883, yang merupakan port MQTT tanpa enkripsi TLS.

Data dikirim ke topik unsoed/tk245004/Percobaan3B/sensor. Topik ini
berfungsi sebagai alamat atau jalur tempat pesan MQTT diterbitkan.
Perangkat atau aplikasi lain yang melakukan subscribe pada topik yang
sama dapat menerima data tersebut.

### Cara kerja program

Pada awal program dibuat objek WiFiClient dan PubSubClient. WiFiClient
digunakan sebagai koneksi jaringan, sedangkan PubSubClient mengatur
komunikasi MQTT.

Program memiliki fungsi hubungkanWiFi() untuk menghubungkan ESP8266 ke
jaringan WiFi. Prosesnya hampir sama seperti kode 3A. Program akan terus
mencoba sampai status WiFi menunjukkan bahwa perangkat sudah terhubung.

Fungsi hubungkanMQTT() digunakan untuk menghubungkan perangkat ke broker
MQTT. Program membuat client ID secara acak supaya identitas klien tidak
selalu sama. Jika koneksi berhasil, Serial Monitor menampilkan pesan
berhasil terhubung. Jika gagal, kode status MQTT ditampilkan dan program
mencoba kembali setelah jeda 2 detik.

Pada fungsi setup(), komunikasi serial dimulai, kemudian perangkat
dihubungkan ke WiFi. Setelah itu alamat broker dan port MQTT dimasukkan
ke konfigurasi client.

Di dalam loop(), program memeriksa apakah koneksi MQTT masih aktif. Jika
terputus, program akan mencoba menghubungkan kembali melalui fungsi
hubungkanMQTT(). Fungsi client.loop() juga dipanggil agar komunikasi
MQTT tetap berjalan.

Berikutnya program membuat data JSON yang berisi suhu 28.5 dan
kelembaban 65.0. Data tersebut diubah menjadi teks menggunakan
serializeJson dan disimpan ke dalam buffer.

Data dikirim menggunakan fungsi publish() ke topik MQTT yang sudah
ditentukan. Setelah dikirim, nama topik dan isi data ditampilkan pada
Serial Monitor. Program kemudian menunggu selama 5 detik sebelum
mengirim data lagi.

### Hasil yang diharapkan

Jika WiFi dan koneksi ke broker berhasil, Serial Monitor akan
menampilkan pesan bahwa perangkat sudah terhubung ke broker MQTT.
Setelah itu akan muncul informasi topik dan data JSON yang dikirim
setiap 5 detik.

Data bisa dilihat menggunakan aplikasi MQTT Client dengan cara melakukan
subscribe pada topik yang sama, yaitu
unsoed/tk245004/Percobaan3B/sensor. Jika subscribe berhasil, pesan suhu
dan kelembaban yang diterbitkan ESP8266 akan diterima oleh aplikasi
tersebut.

## Perbedaan Kode 3A dan 3B

Pada kode 3A, data dikirim menggunakan HTTP POST langsung ke endpoint
server. Setelah request dikirim, server memberikan response yang dapat
dibaca oleh ESP8266. Pengiriman dilakukan setiap 10 detik.

Pada kode 3B, data dikirim menggunakan MQTT melalui broker. ESP8266
menerbitkan pesan ke sebuah topik, kemudian perangkat lain yang
subscribe ke topik tersebut dapat menerima pesan. Pengiriman dilakukan
setiap 5 detik.

HTTP menggunakan pola request dan response, sedangkan MQTT menggunakan
pola publish dan subscribe. MQTT biasanya lebih ringan untuk komunikasi
data IoT yang dikirim secara berkala.

## Hal yang perlu diperhatikan

1.  Nama WiFi dan password harus disesuaikan dengan jaringan yang
    digunakan. Jangan membagikan password WiFi ketika mengunggah kode ke
    repositori publik.
2.  Pastikan board ESP8266 sudah dipilih dengan benar pada Arduino IDE.
3.  Pastikan library yang dibutuhkan sudah terpasang, terutama
    PubSubClient dan ArduinoJson.
4.  Pada kode 3A, setInsecure membuat koneksi HTTPS tidak memeriksa
    sertifikat server. Pengaturan ini hanya cocok untuk percobaan
    sederhana.
5.  Pada kode 3B, broker HiveMQ yang digunakan merupakan broker publik.
    Topik sebaiknya dibuat unik karena pesan pada broker publik dapat
    diakses oleh pengguna lain yang mengetahui topiknya.
6.  Nilai suhu dan kelembaban pada kedua program masih berupa nilai
    tetap. Untuk praktikum yang menggunakan sensor sungguhan, bagian
    tersebut dapat diganti dengan pembacaan dari sensor.
7.  Jika koneksi gagal, periksa kembali jaringan WiFi, password, koneksi
    internet, alamat server atau broker, serta port yang digunakan.
