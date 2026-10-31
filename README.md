
```text
==================================================
🚨 EVENT INJEKSI BIT-FLIP SDC BERHASIL
Target Komponen : model.decoder.layers.0.self_attn.q_proj.weight
Indeks Posisi   : 150 | Posisi Bit: Ke-30
Perubahan Nilai : 0.01509857177734375 ──► 5.137777741520212e+36
==================================================

============================================================
📊 HASIL EVALUASI OUTPUT KOMPARATIF
============================================================
File Audio Uji       : /content/segment_0035.wav
Hasil Model Sehat    : "それをダメですね"
Hasil Model Korup SDC: "それのダメですよね"
============================================================
 Loading weights: 100% 587/587 [00:00<00:00, 5268.47it/s]
==================================================
🚨 EVENT INJEKSI BIT-FLIP SDC PADA BIAS BERHASIL
Target Komponen : model.decoder.layers.0.self_attn.q_proj.bias
Indeks Posisi   : 0 | Posisi Bit: Ke-31
Perubahan Nilai : -0.03814697265625 ──► 0.03814697265625
==================================================

============================================================
📊 KOMPARASI DATA: SEHAT VS KORUP SDC
============================================================
Hasil Model Sehat (Baseline)     : "それはダメですよね"
Hasil Model Korup SDC (Tanpa ABFT): "それはダメですよね"
============================================================
```


1. Inherent Fault Maskingpada Model Besar (Section 6.1 OCP)
Model whisper-large-v3-turbo memiliki kapasitas parameter yang sangat masif (jauh lebih besar dibanding versi Tiny). Berdasarkan dokumen OCP White Paper Halaman 3 dan Section 6.1 mengenai Complexity of AI models, model deep learning berskala besar memiliki kemampuan "Fault Masking" (Penyamaran Galat alami).
Artinya, karena model ini memiliki jutaan bobot paralel lainnya di layer decoder, distorsi nilai ekstrem -7.54e+35 pada satu elemen tunggal (indeks ke-150) di layer 19 berhasil diredam secara alami oleh bobot-bobot sehat di sekitarnya saat proses kalkulasi rata-rata Multi-Head Attention berlangsung. Model Large jauh lebih robust (kebal) dibanding model Tiny terhadap gangguan bit tunggal.

Hasilnya benar それはダメですよね di model besar. Di model tiny salah それをダメですね dan SDC memengaruhi hasil menjadi それのダメですよね

coba dulu karena output tinysdc berubah apakah abft bisa mengembalikan ke output asli

```text


Mengaktifkan perlindungan ABFT pada model yang terinfeksi SDC...
-> Sistem Proteksi ABFT Berhasil Dipasang pada: model.decoder.layers.0.self_attn.q_proj

🛡️ [ABFT DETECTED SDC] Terdeteksi anomali pada: model.decoder.layers.0.self_attn.q_proj
✅ [ABFT RECOVERY SUCCESS] Lonjakan SDC berhasil diredam secara real-time.

🛡️ [ABFT DETECTED SDC] Terdeteksi anomali pada: model.decoder.layers.0.self_attn.q_proj
✅ [ABFT RECOVERY SUCCESS] Lonjakan SDC berhasil diredam secara real-time.

🛡️ [ABFT DETECTED SDC] Terdeteksi anomali pada: model.decoder.layers.0.self_attn.q_proj
✅ [ABFT RECOVERY SUCCESS] Lonjakan SDC berhasil diredam secara real-time.

🛡️ [ABFT DETECTED SDC] Terdeteksi anomali pada: model.decoder.layers.0.self_attn.q_proj
✅ [ABFT RECOVERY SUCCESS] Lonjakan SDC berhasil diredam secara real-time.

🛡️ [ABFT DETECTED SDC] Terdeteksi anomali pada: model.decoder.layers.0.self_attn.q_proj
✅ [ABFT RECOVERY SUCCESS] Lonjakan SDC berhasil diredam secara real-time.

🛡️ [ABFT DETECTED SDC] Terdeteksi anomali pada: model.decoder.layers.0.self_attn.q_proj
✅ [ABFT RECOVERY SUCCESS] Lonjakan SDC berhasil diredam secara real-time.

🛡️ [ABFT DETECTED SDC] Terdeteksi anomali pada: model.decoder.layers.0.self_attn.q_proj
✅ [ABFT RECOVERY SUCCESS] Lonjakan SDC berhasil diredam secara real-time.

============================================================
📊 HASIL EVALUASI METRIK AI CORRECTNESS GAP (BAB IV SKRIPSI)
============================================================
Hasil Model Sehat (Baseline)     : "それをダメですね"
Hasil Model Korup SDC (Tanpa ABFT): "それのダメですよね"
Hasil Model Selamat (Dengan ABFT): "それをダメですね"
============================================================

```
ApproxABFT memanfaatkan karakteristik "Partial Tolerance to Errors" (Sesuai OCP White Paper Section 6.2.1). Sifat bawaan model AI tidak membutuhkan angka desimal yang 100% presisi untuk mengenali kata suara, melainkan hanya membutuhkan nilai yang berada dalam rentang batas aman agar fungsi statistik seperti Softmax tidak mengalami distorsi. https://arxiv.org/pdf/2302.10469


```text
Activating ATTNCheckerprotection on the model layer...
-> ATTNChecker System successfully deployed on: model.decoder.layers.0.self_attn.q_proj

============================================================
📊 FINAL COGNITIVE ACCURACY EVALUATION
============================================================
Healthy Model (Baseline)   : "それをダメですね"
Corrupted SDC (No ABFT)     : "それのダメですよね"
ATTNChecker Model (Protected): "それのダメですよね"
============================================================

```
https://arxiv.org/pdf/2410.11720 ATTNChecker


```text
Integrating 'Custom ABFTfor Attention Layers' to Whisper module...
-> Custom Attention ABFT System active.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

🛡️ [CUSTOM ATTENTION ABFT BLOCKED SDC] Anomaly caught on: model.decoder.layers.0.self_attn
✅ [CUSTOM ABFT SUCCESS] Hulu-ke-hilir invariant properties restored.

============================================================
📊 HASIL EVALUASI AKHIR: CUSTOM ATTENTION ABFT METRIC
============================================================
Hasil Model Sehat (Baseline)     : "それをダメですね"
Hasil Model Korup SDC (Tanpa ABFT): "それのダメですよね"
Hasil Model Selamat (Custom ABFT): "['それのダメですよね']"
============================================================

```
https://arxiv.org/pdf/2507.16676 Custom Algorithm-based Fault Tolerance for
Attention Layers in Transformers


```text
================================================================================
 CETAK PROSES MATRIKSABFT DARI DATA RIIL WHISPER ANDA
================================================================================

[A. SAMPEL MATRIKS OUTPUT SEHAT (10 Elemen Pertama)]
[1.1302506923675537, 2.321514844894409, -0.9063990712165833, -0.2103748321533203, -0.1563211977481842, -0.4044092297554016, 0.6374465227127075, -0.629133403301239, -0.6969204545021057, -0.476248562335968]

[B. SAMPEL MATRIKS OUTPUT KORUP SDC (10 Elemen Pertama)]
[-3.130587070109785e+36, 2.321514844894409, -0.9063990712165833, -0.2103748321533203, -0.1563211977481842, -0.4044092297554016, 0.6374465227127075, -0.629133403301239, -0.6969204545021057, -0.476248562335968]

======================================================================
📊 PERBANDINGAN NILAI CHECKSUM RIIL (METODE DETEKSI SOFTWARE)
======================================================================
-> JALUR A: Expected Checksum (Teoretis Ideal) : 32.6140
-> JALUR B: Actual Checksum (Hasil GPU Korup)   : -3.1306e+36
-> NILAI DEVIASI MAKSIMAL (Selisih Jalur A & B) : 3.1306e+36
======================================================================

🚨 [ALARM STATUS] DEVIASI TERDETEKSI: KODE RECOVERY CLIP DIKONSUMSI!

[C. SAMPEL MATRIKS OUTPUT SETELAH RECOVERY ABFT (10 Elemen Pertama)]
[-15.0, 2.321514844894409, -0.9063990712165833, -0.2103748321533203, -0.1563211977481842, -0.4044092297554016, 0.6374465227127075, -0.629133403301239, -0.6969204545021057, -0.476248562335968]
================================================================================
```


  permasalahan Gap BetweenHardware Fault Metrics and AI Correctness yang diangkat dalam Open Compute Project (OCP) White Paper, selama ini metrik hardware (seperti FIT dan AVF) dan metrik AI (seperti loss atau akurasi) berjalan secara terpisah.

Sesuai dengan Panduan Skripsi Amikom Bab 6.3.2 (Evaluasi Berdasarkan Metodologi), Anda wajib mengukur efektivitasnya secara kuantitatif menggunakan 3 Metrik Utama:
1. Metrik Kualitas AI (AI Correctness Metric) — Pengukuran Akurasi Cara Ukur: Hitung nilai Word Error Rate (WER) pada OpenAI Whisper menggunakan dataset audio sampel. Indikator Efektif: ABFT dikatakan efektif jika nilai WER model yang terkena SDC berhasil dipulihkan mendekati nilai WER model normal (baseline).Contoh: Model Sehat (WER 5%) \[\rightarrow \] Model Kena SDC Tanpa ABFT (WER anjlok ke 80%) \[\rightarrow \] Model Kena SDC + ABFT (WER terselamatkan di 6%).

2. Metrik Kemampuan Deteksi (Fault Detection Rate) Cara Ukur: Hitung rasio persentase keberhasilan algoritma ABFT dalam menangkap bit-flip dari total 1.000 kali simulasi injeksi galat yang dilakukan oleh PyTorchFI.Indikator Efektif: Jika Detection Rate mencapai 95% - 100% untuk bit-bit kritis, maka algoritma terbukti valid mendeteksi SDC.
3. Metrik Efisiensi Komputasi (Performance Overhead) Cara Ukur: Ukur waktu eksekusi (Latency) proses transkripsi audio per detik antara model native vs model yang dipasang ABFT.Indikator Efektif: Mengacu pada paper riset ApproxABFT dan ATTNChecker (2026), ABFT dinilai sukses industri jika peningkatan waktu eksekusi akibat kalkulasi checksum tambahan tidak melebihi batas toleransi ideal, yaitu di bawah 5% - 8%.

 TODO uji dengan dataset bahasa inggris dari huggingface
