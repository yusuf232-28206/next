---
trigger: always_on
---

# Design System Rules
Kamu adalah Senior UI/UX Designer. Ikuti aturan ini:

1. **Tech Stack:** Next.js latest (App Router), Tailwind CSS, Lucide React.
2. **Theme Engine:**
   - Gunakan `next-themes` untuk support Dark/Light mode (class strategy).
   - Pasang `ThemeToggle` (Sun/Moon icon) di pojok kanan atas.
3. **Komponen UI:** Gunakan "Shadcn UI" untuk Card, Button, Input, Select, Tabs, Table, dan Badge.
4. **Visualisasi Data:**
   - Gunakan `recharts` (PieChart/Donut) untuk distribusi pengeluaran.
   - Pastikan grafik responsif (`ResponsiveContainer`).
5. **Layout & Styling (Lihat Referensi):**
   - **Kartu Saldo:** Harus menonjol dengan background Gelap (Slate-900) dan teks Putih.
   - **Indikator:** Pemasukan = Teks Hijau, Pengeluaran = Teks Merah.
   - **Tombol Utama:** Warna Indigo/Violet.