# AI System Evaluation, Safety & Optimization — IT Helpdesk Assistant

> Catatan pengisi: bagian bertanda **[ISI]** harus diisi dari hasil eksekusi nyata. Hapus catatan ini sebelum submit.

## 1. Problem Statement

**Objective evaluasi.** Memastikan sebelum masuk produksi bahwa asisten helpdesk (a) menjawab relevan dan tidak mengarang, (b) merespons dalam waktu wajar, (c) hemat token/biaya, dan (d) tahan terhadap input tidak wajar.

**Sistem yang dievaluasi.** Hasil Assignment 4 milik sendiri (`system/`). **[ISI alasan, mis. sistem berjalan end-to-end di Docker.]**

**Asumsi & batasan.**
- Data: 31 FAQ + 15 tiket mock; model Gemini sesuai `.env`.
- Latency diukur dari klien lokal (termasuk jaringan ke Gemini API) sehingga bervariasi antar-run; tiap skenario dijalankan **[ISI N]** kali dan dilaporkan median.
- Token usage berasal dari `usage_metadata` Gemini (tidak termasuk panggilan embedding).
- Penilaian relevansi/hallucination dilakukan manual dengan acuan `expected_behavior_note`.

## 2. Struktur & Cara Menjalankan

```
ai-evaluation-assignment/
├── system/        # sistem Assignment 4 (sudah diperbaiki) — salin data/ dari project Assignment 4
├── evaluation/    # skenario, hasil, framework safety, before/after, run_evaluation.py
├── README.md
└── requirements.txt   # dependensi skrip evaluasi
```

**Before vs After dari codebase yang sama** (saklar `BASELINE_MODE`):

| Mode | Perilaku |
|---|---|
| `BASELINE_MODE=true` | Sebelum optimasi: prompt satu blok tanpa system_instruction, tanpa batas output, tanpa cache embedding, tanpa shortcut sapaan, tanpa pola "tiket saya". |
| `BASELINE_MODE=false` (default) | Setelah optimasi. |

Instrumentasi (`retrieval_ms`, `llm_ms`, token, skor retrieval) aktif di kedua mode dan dikembalikan pada field `meta` di respons `/ask`.

```bash
# 1) Jalankan sistem (BASELINE) — dari folder system/
cd system && vi .env     # isi GEMINI_API_KEY
BASELINE_MODE=true uvicorn app.main:app --port 8000
# atau Docker:
# docker build -t helpdesk-agent . && docker run --rm -p 8000:8000 --env-file .env -e BASELINE_MODE=true helpdesk-agent

# 2) Di terminal lain — dari root repo
pip install -r requirements.txt
cd evaluation
python run_evaluation.py --label before --repeat 3
python run_evaluation.py --label safety_before --safety

# 3) Restart sistem tanpa BASELINE_MODE, lalu
python run_evaluation.py --label after --repeat 3
python run_evaluation.py --label safety_after --safety
```
Output: `evaluation_results.csv` (before), `evaluation_results_after.csv`, `safety_results_*.csv`.

## 3. Ringkasan Hasil Evaluasi (12 Skenario)

**[ISI]** Tabel ringkas dari `evaluation_results.csv`:

| ID | Intent | Ringkasan respons | Relevan? | Catatan |
|---|---|---|---|---|
| EVAL-01 | | | | |
| ... | | | | |
| EVAL-12 | | | | |

### Temuan utama
**[ISI]** daftar isu dengan kutipan respons bermasalah (wajib ada kutipan evidence).

### Hallucination / ketidakrelevanan (EVAL-04, 05, 10)
**[ISI]** kutipan respons, skor retrieval (`top_score`), dan penilaian.

### Temuan kualitatif routing campuran (EVAL-09)
**[ISI]** intent yang dipilih router dan apakah masuk akal.

## 4. Analisis Latency & Bottleneck
**[ISI]** tabel `latency_ms`, `retrieval_ms`, `llm_ms` untuk EVAL-12 vs EVAL-11 dan skenario representatif. Hitung porsi tiap tahap (mis. `llm_ms / server_latency_ms`) dan simpulkan bottleneck.

## 5. Analisis Resource / Token
**[ISI]** tabel `prompt_tokens`, `output_tokens`, `total_tokens` (EVAL-01, 11, 12). Identifikasi pola boros (mis. output tanpa batas, konteks top-k, panggilan LLM untuk sapaan) dan implikasi biaya.

## 6. Safety / Guardrail Testing
Lihat `evaluation/safety_guardrail_testing_framework.md` (4 kategori, format baku). **[ISI ringkasan status tiap kategori.]**

## 7. Root Cause Analysis & Perbaikan
Isu yang ditangani (minimal 2, pilih berdasarkan data): **[ISI]**

| Isu | Evidence (kutipan/angka) | Root cause | Perbaikan di kode (file) |
|---|---|---|---|
| | | | |

Kandidat perbaikan yang sudah tersedia di kode (aktif saat `BASELINE_MODE=false`): system_instruction anti-override/anti-bocor + query sebagai data (`rag.py`), batas output & opsi thinking budget (`rag.py`), cache embedding query (`rag.py`), shortcut sapaan & pola "tiket saya" (`router.py`). **Klaim hanya perbaikan yang evidence-nya mendukung.**

## 8. Perbandingan Before vs After
Detail: `evaluation/before_after_improvement_template.csv`. **[ISI tabel ringkas angka before → after.]**

## 9. Rekomendasi Teknis Lanjutan
**[ISI]** spesifik dan dapat ditindaklanjuti, berbasis temuan. Contoh arah (hapus yang tidak relevan dengan evidence-mu): routing campuran (EVAL-09) dengan memecah query menjadi sub-intent; kalibrasi `MIN_RELEVANCE_SCORE` dengan set evaluasi lebih besar; streaming respons untuk latency persepsi; tracing per-tahap dengan LangSmith; caching respons FAQ; rate limiting dan batas panjang input; otorisasi pada tool tiket.