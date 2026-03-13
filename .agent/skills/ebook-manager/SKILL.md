---
name: ebook-manager
description: Mengelola logika penyimpanan data keuangan ke LocalStorage.
---

# Skill: ebook manager

Gunakan skill ini untuk membuat Logic Hook (`useEbookStore`).

## Aturan Logika:

1.  **Struktur Data:**
    - `id`: string (UUID).
    - `type`: 'income' | 'expense'.
    - `amount`: number.
    - `category`: string.
    - `date`: string.
2.  Fitur Logic (Wajib)
    - `addTransaction(data): Tambah ke array.
    - `deleteTransaction(id): Hapus data.
    - `getBalance(): Total Income - Total Expense.
    - `getSummary(): Return object { totalIncome, totalExpense }.
    - `formatRupiah(number): Helper untuk format "Rp 50.000".
3.  Persistensi
    - Simpan ke `localStorage` dengan key `duitku-v1`.
    - Handle `Hydration Mismatch`: Pastikan data di-load via `useEffect`.
