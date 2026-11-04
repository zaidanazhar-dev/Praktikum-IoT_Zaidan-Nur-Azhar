## Kode 5A - ESP8266 Mengirim Data Suhu ke MQTT

### Tujuan program

Kode 5A digunakan untuk membuat data suhu simulasi pada ESP8266 dan
mengirimkannya ke broker MQTT. Selain mengirim data, program juga
melakukan perhitungan moving average untuk melihat nilai suhu yang sudah
dirata-ratakan, serta mencatat nilai minimum dan maksimum selama program
berjalan.

Program ini ditempatkan dan dijalankan pada Arduino IDE dengan board
ESP8266 yang sesuai.

### Library yang digunakan

1.  ESP8266WiFi.h digunakan untuk menghubungkan ESP8266 ke jaringan
    WiFi.
2.  PubSubClient.h digunakan untuk melakukan komunikasi MQTT dengan
    broker HiveMQ.
3.  math.h digunakan untuk menyediakan fungsi matematika, salah satunya
    fungsi sin() yang dipakai untuk membuat perubahan suhu simulasi.

### Pengaturan WiFi dan MQTT

Nama jaringan WiFi dan password dimasukkan ke dalam variabel ssid dan
password. Perangkat akan menggunakan data tersebut untuk terhubung ke
jaringan.

Broker MQTT yang digunakan adalah broker.hivemq.com dengan port 1883.
Data dikirim ke topik unsoed/tk245004/broker5/data. Topik ini harus sama
dengan topik yang digunakan pada program Python supaya data dapat
diterima.

Objek WiFiClient digunakan sebagai koneksi jaringan, kemudian objek
PubSubClient menggunakan koneksi tersebut untuk berkomunikasi dengan
broker MQTT.

### Variabel pengolahan data

Variabel N diatur dengan nilai 5. Nilai ini menunjukkan jumlah data yang
digunakan dalam perhitungan moving average. Array buffer\[N\] berfungsi
untuk menyimpan beberapa nilai suhu terakhir.

Variabel indeks digunakan untuk menentukan posisi penyimpanan data
berikutnya pada buffer. Setelah satu data dimasukkan, indeks akan
bertambah. Operator modulo digunakan supaya posisi indeks kembali ke
awal ketika sudah mencapai batas array.

Variabel jumlah digunakan untuk menghitung berapa banyak data yang sudah
masuk ke buffer. Pada awal program jumlahnya masih nol, lalu bertambah
sampai maksimal lima data.

Variabel suhuMin dan suhuMax digunakan untuk menyimpan suhu terendah dan
tertinggi yang pernah terbaca selama program berjalan. Nilai awalnya
dibuat sangat besar untuk suhu minimum dan sangat kecil untuk suhu
maksimum supaya pembacaan pertama bisa langsung menggantinya.

Interval diatur sebesar 2000 milidetik atau 2 detik. Variabel
waktuSebelumnya digunakan untuk menyimpan waktu pengiriman sebelumnya
agar program dapat mengatur jadwal pembacaan tanpa menggunakan delay
panjang.

### Fungsi bacaSuhuMock()

Fungsi bacaSuhuMock() digunakan untuk menghasilkan data suhu simulasi.
Jadi pada bagian ini belum ada pembacaan dari sensor DHT atau sensor
suhu lainnya.

Variabel jumlahBaca akan bertambah setiap fungsi dipanggil. Nilai suhu
dasar dibuat menggunakan rumus 28.0 ditambah hasil perhitungan fungsi
sin(). Dengan cara ini suhu akan naik dan turun secara perlahan,
sehingga data terlihat seperti perubahan suhu yang terjadi dari waktu ke
waktu.

Program juga menambahkan noise atau gangguan kecil secara acak. Nilainya
berasal dari random(-5, 6) yang kemudian dikalikan 0.1. Tujuannya supaya
nilai suhu tidak selalu berubah secara terlalu teratur.

Selain itu, setiap 25 kali pembacaan program menambahkan suhu sebesar 8
derajat. Bagian ini dibuat untuk menghasilkan lonjakan suhu sesekali.
Lonjakan tersebut bisa digunakan untuk melihat bagaimana sistem
pengolahan data dan deteksi anomali bekerja.

Hasil akhir dari fungsi ini dikembalikan sebagai nilai suhu simulasi.

### Fungsi hitungMovingAverage()

Fungsi hitungMovingAverage() digunakan untuk menghitung rata-rata dari
beberapa data suhu terakhir. Nilai suhu baru dimasukkan ke dalam array
buffer pada posisi indeks yang sedang aktif.

Setelah data dimasukkan, indeks berpindah ke posisi berikutnya. Jika
sudah sampai pada batas array, posisi akan kembali ke awal. Cara ini
membuat array digunakan berulang seperti tempat penyimpanan melingkar.

Variabel jumlah akan bertambah selama jumlah data yang masuk belum
mencapai lima. Setelah itu program menjumlahkan seluruh data yang
tersedia di buffer, lalu membaginya dengan jumlah data tersebut.

Hasil perhitungan merupakan nilai moving average. Perhitungan ini
berguna untuk membuat perubahan suhu terlihat lebih halus karena
pengaruh perubahan kecil pada satu pembacaan dapat dikurangi.

Perlu diperhatikan bahwa pada kode ini nilai moving average hanya
ditampilkan di Serial Monitor. Data yang dikirim ke MQTT masih
menggunakan nilai suhu mentah dari variabel suhu, bukan nilai
suhuFilter.

### Fungsi koneksi WiFi

Fungsi koneksiWiFi() digunakan untuk menghubungkan ESP8266 ke jaringan
WiFi. Mode perangkat diatur sebagai WIFI_STA, yaitu mode station yang
membuat ESP8266 terhubung ke router sebagai klien.

Setelah WiFi.begin() dijalankan, program akan menunggu sampai status
koneksi menjadi WL_CONNECTED. Selama belum berhasil, Serial Monitor akan
menampilkan titik setiap setengah detik.

Jika sudah terhubung, program menampilkan pesan keberhasilan dan alamat
IP ESP8266. Alamat IP ini bisa digunakan untuk memastikan perangkat
sudah masuk ke jaringan.

### Fungsi koneksi MQTT

Fungsi koneksiMQTT() digunakan untuk menghubungkan ESP8266 ke broker
HiveMQ. Selama client belum terhubung, program akan mencoba membuat
koneksi kembali.

Client ID dibuat menggunakan awalan ESP8266-Unsoed dan nomor chip
ESP8266 dalam format hexadecimal. Penggunaan ID tersebut membuat
identitas klien dapat dibedakan.

Jika koneksi berhasil, Serial Monitor menampilkan pesan Berhasil. Jika
gagal, program menampilkan kode status dari MQTT dan menunggu selama 2
detik sebelum mencoba lagi.

### Bagian setup()

Pada fungsi setup(), komunikasi serial dimulai dengan baud rate 115200
supaya informasi program dapat dilihat melalui Serial Monitor Arduino
IDE.

randomSeed(micros()) digunakan untuk mengatur nilai awal pembangkit
angka acak. Hal ini membuat nilai noise yang dihasilkan tidak selalu
sama setiap program dimulai.

Setelah itu ESP8266 dihubungkan ke WiFi. Alamat broker dan port MQTT
kemudian diatur menggunakan client.setServer(). Ukuran buffer MQTT juga
diatur menjadi 256 byte agar cukup untuk menampung pesan yang dikirim.

Program selanjutnya menampilkan judul percobaan, topik MQTT, dan
keterangan kolom data yang akan ditampilkan.

### Bagian loop()

Pada fungsi loop(), program terlebih dahulu memeriksa koneksi WiFi. Jika
WiFi terputus, fungsi koneksiWiFi() dipanggil kembali. Program juga
memeriksa koneksi MQTT dan menghubungkan ulang jika diperlukan.

Fungsi client.loop() dijalankan agar komunikasi MQTT tetap diproses.

Selanjutnya program membaca waktu sekarang menggunakan millis(). Jika
selisih waktu sekarang dengan waktuSebelumnya sudah mencapai 2000
milidetik, pembacaan suhu dilakukan.

Nilai suhu simulasi diambil dari bacaSuhuMock(), kemudian dihitung
moving average menggunakan hitungMovingAverage(). Program juga
membandingkan suhu terbaru dengan suhu minimum dan maksimum yang
tersimpan. Jika suhu baru lebih rendah, suhuMin diperbarui. Jika lebih
tinggi, suhuMax diperbarui.

Nilai suhu mentah, hasil filter, minimum, dan maksimum ditampilkan pada
Serial Monitor dengan dua angka di belakang koma.

Setelah itu program membuat payload JSON menggunakan snprintf(). Format
data yang dibuat adalah {"suhu":nilai_suhu}. Payload tersebut kemudian
dikirim menggunakan client.publish() ke topik MQTT yang sudah
ditentukan.

Jika publish berhasil, program menampilkan data yang terkirim. Jika
gagal, program menampilkan pesan bahwa pengiriman MQTT gagal.

## Kode 5B - Penerimaan dan Pengolahan Data Menggunakan Python

### Tujuan program

Kode 5B digunakan untuk menerima data suhu yang dikirim oleh ESP8266
melalui MQTT. Setelah data diterima, Python akan mengolahnya menggunakan
beberapa library untuk membuat file CSV, menghitung nilai statistik,
mendeteksi anomali, dan menampilkan grafik suhu.

Program ini dijalankan menggunakan Python melalui Visual Studio Code,
bukan di Arduino IDE. Jadi kode ini dijalankan pada komputer yang sudah
terpasang Python dan library yang dibutuhkan.

### Library yang digunakan

1.  json digunakan untuk membaca pesan JSON yang diterima dari MQTT.
2.  time digunakan untuk mengambil waktu saat data diterima.
3.  paho.mqtt.client digunakan untuk menghubungkan Python ke broker MQTT
    dan menerima pesan.
4.  pandas digunakan untuk menyimpan dan mengolah data dalam bentuk
    tabel.
5.  matplotlib.pyplot digunakan untuk membuat grafik data suhu.

Sebelum menjalankan program, library Python yang dibutuhkan harus sudah
dipasang. Instalasi dapat dilakukan melalui terminal Visual Studio Code
dengan perintah berikut:

pip install paho-mqtt pandas matplotlib

Modul json dan time sudah tersedia pada Python sehingga tidak perlu
dipasang secara terpisah.

### Pengaturan broker dan jumlah sampel

Variabel BROKER berisi alamat broker HiveMQ, yaitu broker.hivemq.com.
Port yang digunakan adalah 1883.

Variabel TOPIC berisi topik unsoed/tk245004/broker5/data. Topik ini
harus sama dengan topik pada kode 5A. Jika penulisan topik berbeda,
program Python tidak akan menerima pesan yang dikirim ESP8266.

JUMLAH_SAMPEL diatur sebanyak 30. Artinya program akan mengumpulkan 30
data suhu sebelum memutus koneksi MQTT dan melanjutkan proses pengolahan
data.

Variabel data dibuat sebagai list kosong. List ini digunakan untuk
menampung seluruh data suhu beserta waktu saat pesan diterima.

### Fungsi on_connect()

Fungsi on_connect() dijalankan ketika program Python berhasil terhubung
ke broker MQTT. Program akan menampilkan kode koneksi pada terminal.

Setelah itu client.subscribe(TOPIC) dijalankan. Perintah ini membuat
program Python berlangganan pada topik yang sama dengan topik pengiriman
dari ESP8266.

Dengan melakukan subscribe, Python dapat menerima setiap pesan baru yang
diterbitkan oleh ESP8266 pada topik tersebut.

### Fungsi on_message()

Fungsi on_message() dijalankan setiap kali ada pesan masuk dari broker
MQTT.

Pesan MQTT awalnya diterima dalam bentuk byte. Program mengubahnya
menjadi teks menggunakan decode(), kemudian json.loads() digunakan untuk
membaca isi JSON.

Nilai suhu diambil dari bagian suhu pada JSON dan diubah menjadi tipe
float. Jika data tidak memiliki field suhu atau formatnya tidak sesuai,
program akan menampilkan pesan Data tidak valid, diabaikan. Dengan
begitu data yang salah tidak ikut dimasukkan ke dalam pengolahan.

Jika data berhasil dibaca, program menambahkan waktu penerimaan dan
nilai suhu ke dalam list data. Waktu dicatat menggunakan time.strftime()
dengan format tahun, bulan, tanggal, jam, menit, dan detik.

Jumlah data yang sudah diterima juga ditampilkan pada terminal. Jika
jumlah data sudah mencapai 30 sampel, program menjalankan
client.disconnect() untuk memutus koneksi MQTT.

### Koneksi MQTT pada Python

Setelah fungsi callback dibuat, program membuat objek MQTT Client dengan
CallbackAPIVersion.VERSION2. Kemudian fungsi on_connect dan on_message
dipasang sebagai callback.

Program melakukan koneksi ke broker menggunakan client.connect(BROKER,
PORT). Setelah itu client.loop_forever() dijalankan supaya program terus
menunggu dan memproses pesan yang masuk.

Bagian ini akan terus berjalan sampai koneksi dihentikan. Dalam kode
ini, koneksi akan diputus setelah jumlah data yang diterima mencapai 30
sampel.

### Menyimpan data ke CSV

Setelah proses penerimaan selesai, list data diubah menjadi DataFrame
menggunakan pandas.DataFrame().

Data kemudian disimpan ke file dengan nama data_suhu.csv menggunakan
df.to_csv(). Parameter index=False digunakan supaya nomor indeks
DataFrame tidak ikut ditulis sebagai kolom tambahan pada file.

File CSV ini berisi waktu penerimaan dan nilai suhu. File akan tersimpan
pada folder kerja tempat program Python dijalankan, kecuali lokasi
penyimpanan diubah.

Sebelum proses analisis dilakukan, program memeriksa apakah DataFrame
kosong. Jika tidak ada data yang diterima, program menampilkan pesan dan
menghentikan proses menggunakan SystemExit.

### Menghitung statistik suhu

Program menghitung rata-rata suhu menggunakan df\["suhu"\].mean(). Nilai
rata-rata ini menunjukkan nilai suhu secara umum dari seluruh sampel
yang diterima.

Selanjutnya standar deviasi dihitung menggunakan df\["suhu"\].std().
Standar deviasi digunakan untuk melihat seberapa jauh penyebaran data
dari nilai rata-ratanya.

Program juga mencari suhu minimum dengan min() dan suhu maksimum dengan
max(). Hasilnya ditampilkan pada terminal dengan dua angka di belakang
koma.

### Menghitung Z-Score dan mendeteksi anomali

Pada bagian ini program menggunakan metode Z-Score untuk mengetahui
apakah ada data suhu yang berbeda cukup jauh dari rata-rata.

Rumus Z-Score pada program adalah nilai suhu dikurangi rata-rata,
kemudian dibagi dengan standar deviasi.

Jika standar deviasi tidak tersedia atau bernilai nol, program mengisi
nilai zscore dengan 0.0 supaya tidak terjadi pembagian dengan nol.

Setelah nilai Z-Score didapatkan, program membuat kolom anomali. Data
dianggap sebagai anomali jika nilai absolut Z-Score lebih besar dari 2.

Artinya, data yang jaraknya lebih dari dua standar deviasi dari
rata-rata akan ditandai sebagai anomali. Hasil deteksi ini kemudian
dihitung jumlahnya dan ditampilkan pada terminal.

Metode ini digunakan untuk menandai nilai yang terlihat tidak biasa
dibandingkan kumpulan data lainnya. Namun, hasilnya tetap perlu dilihat
kembali karena nilai yang ditandai belum tentu merupakan kesalahan
sensor.

### Membuat grafik suhu

Setelah pengolahan data selesai, program membuat grafik menggunakan
matplotlib. Ukuran grafik diatur menjadi 10 x 4 agar data lebih mudah
dilihat.

Perintah plt.plot() digunakan untuk menampilkan perubahan suhu
berdasarkan urutan data yang diterima. Setiap titik diberi marker
berbentuk lingkaran.

Data yang terdeteksi sebagai anomali diambil menggunakan
df\[df\["anomali"\]\]. Nilai tersebut kemudian ditampilkan menggunakan
plt.scatter() dengan warna merah dan ukuran titik yang lebih besar
supaya mudah dibedakan dari data lainnya.

Garis rata-rata ditambahkan menggunakan plt.axhline(). Garis ini menjadi
pembanding untuk melihat posisi data suhu terhadap nilai rata-rata
keseluruhan.

Program juga menambahkan label sumbu X sebagai urutan data, label sumbu
Y sebagai suhu dalam derajat Celsius, judul grafik, dan legenda.

Terakhir, plt.tight_layout() digunakan untuk merapikan jarak antarbagian
grafik. Grafik disimpan dengan nama grafik_suhu.png menggunakan
plt.savefig(), kemudian ditampilkan ke layar menggunakan plt.show().

## Alur Kerja Kode 5A dan 5B

Secara keseluruhan, kedua kode bekerja secara berurutan seperti berikut:

1.  ESP8266 dinyalakan dan dihubungkan ke jaringan WiFi.
2.  ESP8266 membuat koneksi ke broker MQTT HiveMQ.
3.  Kode 5A menghasilkan nilai suhu simulasi setiap 2 detik.
4.  ESP8266 menghitung moving average serta memperbarui nilai suhu
    minimum dan maksimum.
5.  Nilai suhu mentah dikirim ke topik unsoed/tk245004/broker5/data
    dalam bentuk JSON.
6.  Program Python pada VSCode sudah terhubung ke broker dan melakukan
    subscribe pada topik yang sama.
7.  Python menerima pesan, membaca nilai suhu, lalu menyimpan waktu dan
    suhu ke dalam list.
8.  Setelah 30 data diterima, Python memutus koneksi MQTT.
9.  Data yang sudah terkumpul disimpan menjadi file data_suhu.csv.
10. Python menghitung rata-rata, standar deviasi, nilai minimum,
    maksimum, dan Z-Score.
11. Data yang memiliki nilai absolut Z-Score lebih dari 2 ditandai
    sebagai anomali.
12. Hasil pengolahan ditampilkan dalam bentuk grafik dan disimpan
    sebagai grafik_suhu.png.

## Cara Menjalankan Program

### Menjalankan kode 5A di Arduino IDE

1.  Buka Arduino IDE pada komputer.
2.  Pastikan board ESP8266 sudah terpasang dan dipilih sesuai perangkat
    yang digunakan.
3.  Pastikan library ESP8266WiFi dan PubSubClient sudah tersedia.
4.  Masukkan kode 5A ke Arduino IDE.
5.  Sesuaikan nama WiFi dan password dengan jaringan yang akan
    digunakan.
6.  Pastikan alamat broker dan topik MQTT sama dengan yang digunakan
    pada kode Python.
7.  Hubungkan ESP8266 ke komputer, pilih port yang sesuai, lalu upload
    program.
8.  Buka Serial Monitor dengan baud rate 115200 untuk melihat proses
    koneksi dan data yang dikirim.

### Menjalankan kode 5B di Visual Studio Code

1.  Buka Visual Studio Code.
2.  Pastikan Python sudah terpasang dan interpreter Python sudah
    dipilih.
3.  Buat file Python, misalnya praktikum_bab5.py.
4.  Masukkan kode 5B ke dalam file tersebut.
5.  Buka terminal VSCode dan pasang library yang diperlukan jika belum
    tersedia.
6.  Jalankan program Python.
7.  Pastikan program Python sudah menampilkan pesan terhubung ke broker
    dan melakukan subscribe ke topik.
8.  Jalankan ESP8266 dari Arduino IDE agar data mulai dikirim.
9.  Tunggu sampai Python menerima 30 sampel. Setelah itu file CSV dan
    grafik akan dibuat.

Sebaiknya program Python dijalankan terlebih dahulu sebelum ESP8266
mulai mengirim data. Dengan begitu program penerima sudah siap melakukan
subscribe ketika pesan mulai diterbitkan.

## Hasil yang Diharapkan

Pada Serial Monitor Arduino IDE akan terlihat nilai suhu mentah, nilai
moving average, suhu minimum, dan suhu maksimum. Data suhu juga akan
dikirim ke broker MQTT setiap 2 detik.

Pada terminal Visual Studio Code akan terlihat jumlah sampel yang sudah
diterima beserta nilai suhunya. Setelah 30 sampel terkumpul, program
akan menyimpan hasilnya ke data_suhu.csv.

Program juga akan menampilkan hasil perhitungan statistik seperti
rata-rata, standar deviasi, minimum, maksimum, dan jumlah data anomali.
Selain itu, grafik suhu akan ditampilkan dan disimpan dalam file
grafik_suhu.png.

## Hal yang Perlu Diperhatikan

1.  Nama WiFi dan password pada kode 5A harus disesuaikan dengan
    jaringan yang digunakan.
2.  Kode 5A dijalankan pada ESP8266 melalui Arduino IDE, sedangkan kode
    5B dijalankan pada komputer menggunakan Python melalui Visual Studio
    Code.
3.  Pastikan kedua program menggunakan alamat broker, port, dan topik
    MQTT yang sama.
4.  Kode 5A menggunakan data suhu simulasi, bukan pembacaan dari sensor
    fisik.
5.  Perhitungan moving average pada kode 5A hanya ditampilkan di Serial
    Monitor. Data yang dikirim ke MQTT masih berupa suhu mentah.
6.  Pengiriman data dari ESP8266 dilakukan setiap 2 detik, sedangkan
    Python berhenti menerima setelah mendapatkan 30 sampel.
7.  Jika Python tidak menerima data, periksa koneksi internet, alamat
    broker, nama topik, dan pastikan ESP8266 sudah berhasil terhubung ke
    MQTT.
8.  Broker HiveMQ yang digunakan merupakan broker publik. Sebaiknya
    jangan mengirim data pribadi atau informasi penting melalui broker
    tersebut.
9.  File CSV dan gambar grafik akan tersimpan di folder kerja saat
    program Python dijalankan.
10. Hasil deteksi anomali menggunakan batas Z-Score lebih dari 2. Karena
    jumlah sampel hanya 30, hasil deteksi sebaiknya dipahami sebagai
    analisis sederhana, bukan penentuan pasti bahwa suatu data salah.
