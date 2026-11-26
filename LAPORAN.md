# Laporan praktikum arsitektur Flutter MVC dan MVVM

## Identitas mahasiswa

Nama: Muhammad Daffa Ariby Hidayat

NIM: 362558302081

Kelas: TRPL 2E

## Cara menjalankan aplikasi dan pengujian

Jalankan perintah berikut dari folder proyek `prak_arsitektur_tugas` (yang memuat `pubspec.yaml`):

```bash
flutter pub get
flutter run -d chrome
flutter analyze
flutter test
```

Setelah aplikasi terbuka di browser, beranda menampilkan dua tombol menu, yaitu Buka MVC dan Buka MVVM. Setiap layar memiliki penyimpanan tugas di memori masing-masing.

## Perbandingan arsitektur MVC dan MVVM

| Pertanyaan                                                                  | Versi MVC                                                                                                                                                                                                                                                                                                                                                                                                                   | Versi MVVM                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| :-------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ketika filter diketuk, class mana yang menerima aksi dan menyimpan pilihan? | Aksi dari `PanelTugas` diterima oleh `_LayarMvcState` pada metode `_aturFilter` (`lib/mvc/layar_mvc.dart`), lalu diteruskan ke `PengendaliTugas.aturFilter` (`lib/mvc/pengendali_tugas.dart`). Status filter disimpan pada field `_hanyaBelumSelesai` di dalam class `PengendaliTugas`.                                                                                                                                     | Aksi dari `PanelTugas` diteruskan langsung ke metode `aturFilter` pada class `TugasViewModel` (`lib/mvvm/tugas_view_model.dart`). Status filter disimpan pada field `_hanyaBelumSelesai` di dalam class `TugasViewModel`.                                                                                                                                                                                                                                                                                                              |
| Apa yang memicu rebuild? Apa akibatnya jika pemicu itu dihapus?             | Pemicu rebuild adalah pemanggilan `setState` di dalam metode `_tambah`, `_ubahStatus`, dan `_aturFilter` pada `_LayarMvcState`. Saat `setState` di `_aturFilter` dihapus sementara, `flutter test test/filter_test.dart` gagal di baris 32 dengan hasil `+1 -1` karena tugas `Rancang UI Flutter` tetap tampil ketika filter dinyalakan. Setelah `setState` dipulihkan, seluruh 7 test kembali lulus.                       | Pemicu rebuild adalah pemanggilan `notifyListeners()` di `TugasViewModel` pada metode `tambah`, `ubahStatus`, dan `aturFilter`, yang didengarkan oleh `ListenableBuilder` di `LayarMvvm`. Saat `notifyListeners()` di `aturFilter` dihapus sementara, `flutter test test/filter_test.dart` gagal di baris 32 dengan hasil `+1 -1` karena `ListenableBuilder` tidak membangun ulang tampilan dan tugas selesai tetap terlihat. Setelah `notifyListeners()` dipulihkan, seluruh 7 test kembali lulus.                                    |
| Kapan memilih masing-masing pola?                                           | Pola MVC cocok untuk layar tunggal dengan alur sederhana, misalnya form tambah dan centang tugas sebelum ada filter, karena pemanggilan controller dan `setState` dapat dibaca langsung dalam satu file layar (`layar_mvc.dart`). Risikonya, setiap aksi baru di layar harus memanggil `setState` secara manual; jika terlewat seperti pada percobaan `_aturFilter`, data di controller berubah tanpa memperbarui tampilan. | Pola MVVM cocok ketika satu layar membaca beberapa bagian state yang saling bergantung, seperti daftar tugas terfilter, ringkasan total dari repository, dan pesan galat validasi. Kelebihannya, perubahan state dan notifikasi di `TugasViewModel` dapat diuji langsung melalui unit test (`test/logika_test.dart`) tanpa membangun widget. Biayanya, View harus membungkus panel dengan `ListenableBuilder`, memanggil `dispose()` untuk lifecycle ViewModel, dan menjaga agar setiap perubahan state memanggil `notifyListeners()`. |

## Hasil percobaan pemicu rebuild

### Percobaan 1 pada MVC

Bagian yang diubah sementara: menghapus `setState` pada `_aturFilter` di `lib/mvc/layar_mvc.dart`.

Keluaran terminal saat dijalankan dengan `flutter test test/filter_test.dart`:

```text
00:00 +0: MVC menyaring tugas belum selesai
The following TestFailure was thrown running a test:
Expected: no matching candidates
  Actual: _TextWidgetFinder:<Found 1 widget with text "Rancang UI Flutter": [
            Text("Rancang UI Flutter", dependencies: [DefaultSelectionStyle, DefaultTextStyle, MediaQuery]),
          ]>
   Which: means one was found but none were expected

This was caught by the test expectation on the following line:
  file:///D:/praktikum-pasepnew/test/filter_test.dart line 32
00:01 +0 -1: MVC menyaring tugas belum selesai [E]
00:02 +1 -1: Some tests failed.
```

### Percobaan 2 pada MVVM

Bagian yang diubah sementara: menghapus `notifyListeners();` pada `aturFilter` di `lib/mvvm/tugas_view_model.dart`.

Keluaran terminal saat dijalankan dengan `flutter test test/filter_test.dart`:

```text
00:00 +0: MVC menyaring tugas belum selesai
00:01 +1: MVVM menyaring tugas belum selesai
The following TestFailure was thrown running a test:
Expected: no matching candidates
  Actual: _TextWidgetFinder:<Found 1 widget with text "Rancang UI Flutter": [
            Text("Rancang UI Flutter", dependencies: [DefaultSelectionStyle, DefaultTextStyle, MediaQuery]),
          ]>
   Which: means one was found but none were expected

This was caught by the test expectation on the following line:
  file:///D:/praktikum-pasepnew/test/filter_test.dart line 32
00:01 +1 -1: MVVM menyaring tugas belum selesai [E]
00:01 +1 -1: Some tests failed.
```

## Keluaran terminal setelah kode dipulihkan

Hasil `flutter analyze`:

```text
Analyzing praktikum-pasepnew...
No issues found! (ran in 1.7s)
```

Hasil `flutter test`:

```text
00:00 +0: loading D:/praktikum-pasepnew/test/filter_test.dart
00:00 +0: D:/praktikum-pasepnew/test/filter_test.dart: MVC menyaring tugas belum selesai
00:00 +1: D:/praktikum-pasepnew/test/filter_test.dart: MVC menyaring tugas belum selesai
00:00 +2: D:/praktikum-pasepnew/test/filter_test.dart: MVC menyaring tugas belum selesai
00:01 +3: D:/praktikum-pasepnew/test/filter_test.dart: MVC menyaring tugas belum selesai
00:01 +4: D:/praktikum-pasepnew/test/filter_test.dart: MVC menyaring tugas belum selesai
00:01 +5: D:/praktikum-pasepnew/test/layar_test.dart: MVVM memperbarui form dan ringkasan
00:01 +6: D:/praktikum-pasepnew/test/filter_test.dart: MVVM menyaring tugas belum selesai
00:02 +7: All tests passed!
```

## Daftar bukti tangkapan layar

Keempat tangkapan layar berikut disimpan di folder `bukti` dengan dua tugas yang sama (`Rancang UI Flutter` dan `Catat hasil pengujian`) pada kondisi ringkasan `1 dari 2 selesai`:

1. `bukti/mvc_filter_mati.png` menampilkan layar MVC dengan filter nonaktif dan kedua tugas terlihat.
2. `bukti/mvc_filter_aktif.png` menampilkan layar MVC dengan filter aktif dan hanya tugas yang belum selesai terlihat.
3. `bukti/mvvm_filter_mati.png` menampilkan layar MVVM dengan filter nonaktif dan kedua tugas terlihat.
4. `bukti/mvvm_filter_aktif.png` menampilkan layar MVVM dengan filter aktif dan hanya tugas yang belum selesai terlihat.
