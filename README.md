# Hasil Eksperimen SDC dan ABFT pada Model Whisper

## 1. Log Inisialisasi Model

```text
preprocessor_config.json: 100% 185k/185k [00:00<00:00, 10.4MB/s]tokenizer_config.json: 100% 283k/283k [00:00<00:00, 14.1MB/s]vocab.json: 100% 836k/836k [00:00<00:00, 19.4MB/s]tokenizer.json: 100% 2.48M/2.48M [00:00<00:00, 45.0MB/s]merges.txt: 100% 494k/494k [00:00<00:00, 13.3MB/s]normalizer.json: 100% 52.7k/52.7k [00:00<00:00, 4.15MB/s]added_tokens.json: 100% 34.6k/34.6k [00:00<00:00, 2.81MB/s]special_tokens_map.json: 100% 2.19k/2.19k [00:00<00:00, 178kB/s]Loading weights: 100% 167/167 [00:00<00:00, 3938.42it/s][transformers] Using custom `forced_decoder_ids` from the (generation) config. This is deprecated in favor of the `task` and `language` flags/config options.
[transformers] Transcription using a multilingual Whisper will default to language detection followed by transcription instead of translation to English. This might be a breaking change for your use case. If you want to instead always translate your audio to English, make sure to pass `language='en'`. See https://github.com/huggingface/transformers/pull/28687 for more details.
[transformers] Passing `generation_config` together with generation-related arguments=({'begin_suppress_tokens', 'suppress_tokens'}) is deprecated and will be removed in future versions. Please pass either a `generation_config` object OR all generation parameters explicitly, but not both.
[transformers] Ignoring clean_up_tokenization_spaces=True for BPE tokenizer WhisperTokenizer. The clean_up_tokenization post-processing step is designed for WordPiece tokenizers and is destructive for BPE (it strips spaces before punctuation). Set clean_up_tokenization_spaces=False to suppress this warning, or set clean_up_tokenization_spaces_for_bpe_even_though_it_will_corrupt_output=True to force cleanup anyway.
```

## 2. Injeksi Bit-Flip pada Model Whisper Tiny

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

## 3. Inherent Fault Masking pada Model Besar

1. Inherent Fault Maskingpada Model Besar (Section 6.1 OCP)
Model whisper-large-v3-turbo memiliki kapasitas parameter yang sangat masif (jauh lebih besar dibanding versi Tiny). Berdasarkan dokumen OCP White Paper Halaman 3 dan Section 6.1 mengenai Complexity of AI models, model deep learning berskala besar memiliki kemampuan "Fault Masking" (Penyamaran Galat alami).
Artinya, karena model ini memiliki jutaan bobot paralel lainnya di layer decoder, distorsi nilai ekstrem -7.54e+35 pada satu elemen tunggal (indeks ke-150) di layer 19 berhasil diredam secara alami oleh bobot-bobot sehat di sekitarnya saat proses kalkulasi rata-rata Multi-Head Attention berlangsung. Model Large jauh lebih robust (kebal) dibanding model Tiny terhadap gangguan bit tunggal.
Hasilnya benar それはダメですよね di model besar. Di model tiny salah それをダメですね dan SDC memengaruhi hasil menjadi それのダメですよね

## 4. Pengujian Pemulihan Menggunakan ABFT

```text
coba dulu karena output tinysdc berubah apakah abft bisa mengembalikan ke output asli
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
ApproxABFT memanfaatkan karakteristik "Partial Tolerance to Errors" (Sesuai OCP White Paper Section 6.2.1). Sifat bawaan model AI tidak membutuhkan angka desimal yang 100% presisi untuk mengenali kata suara, melainkan hanya membutuhkan nilai yang berada dalam rentang batas aman agar fungsi statistik seperti Softmax tidak mengalami distorsi. https://arxiv.org/pdf/2302.10469
```

## 5. Pengujian ATTNChecker

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
https://arxiv.org/pdf/2410.11720 ATTNChecker
```

## 6. Pengujian Custom Attention ABFT

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
https://arxiv.org/pdf/2507.16676 Custom Algorithm-based Fault Tolerance for
Attention Layers in Transformers
```

## 7. Cetak Proses Matriks ABFT

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

## 8. Evaluasi dan Metrik Penelitian

  permasalahan Gap BetweenHardware Fault Metrics and AI Correctness yang diangkat dalam Open Compute Project (OCP) White Paper, selama ini metrik hardware (seperti FIT dan AVF) dan metrik AI (seperti loss atau akurasi) berjalan secara terpisah.

Sesuai dengan Panduan Skripsi Amikom Bab 6.3.2 (Evaluasi Berdasarkan Metodologi), Anda wajib mengukur efektivitasnya secara kuantitatif menggunakan 3 Metrik Utama:
1. Metrik Kualitas AI (AI Correctness Metric) — Pengukuran Akurasi Cara Ukur: Hitung nilai Word Error Rate (WER) pada OpenAI Whisper menggunakan dataset audio sampel. Indikator Efektif: ABFT dikatakan efektif jika nilai WER model yang terkena SDC berhasil dipulihkan mendekati nilai WER model normal (baseline).Contoh: Model Sehat (WER 5%) \[\rightarrow \] Model Kena SDC Tanpa ABFT (WER anjlok ke 80%) \[\rightarrow \] Model Kena SDC + ABFT (WER terselamatkan di 6%).

2. Metrik Kemampuan Deteksi (Fault Detection Rate) Cara Ukur: Hitung rasio persentase keberhasilan algoritma ABFT dalam menangkap bit-flip dari total 1.000 kali simulasi injeksi galat yang dilakukan oleh PyTorchFI.Indikator Efektif: Jika Detection Rate mencapai 95% - 100% untuk bit-bit kritis, maka algoritma terbukti valid mendeteksi SDC.
3. Metrik Efisiensi Komputasi (Performance Overhead) Cara Ukur: Ukur waktu eksekusi (Latency) proses transkripsi audio per detik antara model native vs model yang dipasang ABFT.Indikator Efektif: Mengacu pada paper riset ApproxABFT dan ATTNChecker (2026), ABFT dinilai sukses industri jika peningkatan waktu eksekusi akibat kalkulasi checksum tambahan tidak melebihi batas toleransi ideal, yaitu di bawah 5% - 8%.
4. uji dengan dataset bahasa inggris dari huggingface
