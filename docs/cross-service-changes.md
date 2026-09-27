# Cross-Service Changes

Dokumen ini mengatur perubahan yang memengaruhi lebih dari satu repository.
Perubahan pada satu repository tetap mengikuti `AGENTS.md` dan test workflow
repository tersebut.

## Prinsip Umum

- Tentukan owner sebelum memilih repository yang diubah.
- Identifikasi producer, implementer, dan seluruh consumer.
- Pertahankan backward compatibility kecuali breaking change disetujui.
- Jangan mengubah repository sibling hanya karena tersedia di workspace; scope
  perubahan harus termasuk dalam permintaan user.
- Local workspace verification tidak menggantikan verification terhadap module
  version yang benar-benar dideklarasikan consumer.

## Shared Contract Change

Gunakan workflow ini untuk protobuf, field semantics, shared error codes, locale,
atau protocol response yang dimiliki `api-contracts`.

1. Definisikan behavior dan compatibility impact.
2. Ubah source contract di `api-contracts`.
3. Jalankan generator dan test milik `api-contracts`.
4. Implementasikan atau sesuaikan behavior di producer/implementer, biasanya
   `grpc-auth`.
5. Sesuaikan downstream consumer, biasanya `rest-orc`.
6. Jalankan targeted dan full tests pada setiap repository yang berubah.
7. Verifikasi integration melalui parent `go.work`.
8. Setelah contract dipublikasikan, verifikasi consumer dengan `GOWORK=off` dan
   module version yang dideklarasikan.

Urutan konseptual:

```text
api-contracts
    -> grpc-auth implementation
        -> rest-orc consumer and HTTP mapping
```

## ORM Change

Gunakan workflow ini ketika generic database behavior berubah.

1. Pastikan behavior benar-benar generic dan tidak dimiliki satu service.
2. Ubah dan uji `orm`.
3. Identifikasi public API atau semantic compatibility impact.
4. Uji consumer yang terdampak, saat ini terutama `grpc-auth`.
5. Verifikasi transaction, query arguments, context propagation, dan error
   normalization yang relevan.

Jangan memindahkan auth-specific query atau business rule ke `orm`.

## Error Contract Change

Bedakan dua jenis perubahan:

- Shared error key, code, localized message, protobuf shape, atau mapping
  metadata: dimulai di `api-contracts`.
- Generic loading, typed error construction, localization mechanics, fallback,
  atau output formatting: dimulai di `error-package`.

Jika perubahan catalog memerlukan kemampuan presentation baru:

1. Definisikan semantics dan compatibility di `api-contracts`.
2. Tambahkan generic presentation capability di `error-package` tanpa
   meng-hardcode business message.
3. Update service atau HTTP/gRPC boundary yang memilih catalog dan mapping.
4. Verifikasi error identity, locale, protocol mapping, dan response shape pada
   seluruh consumer yang terdampak.

Jangan membuat registry ganda dengan code/message yang dapat berbeda secara
diam-diam. Jangan memindahkan HTTP/gRPC mapping ke `error-package`.

## HTTP Orchestration Change

Jika perubahan hanya menambah HTTP route atau orchestration tanpa mengubah
shared contract, implementasi dapat tetap di `rest-orc`.

Jika downstream RPC baru dibutuhkan:

1. Ubah `api-contracts`.
2. Implementasikan RPC di `grpc-auth` atau service pemilik.
3. Tambahkan client/adapter dan orchestration di `rest-orc`.

## Verification Matrix

| Perubahan | Verifikasi utama | Consumer verification |
|---|---|---|
| Protobuf atau shared error | `api-contracts` | `grpc-auth`, `rest-orc` |
| Auth business behavior | `grpc-auth` | `rest-orc` bila response berubah |
| HTTP route/orchestration | `rest-orc` | Downstream contract mocks/integration |
| ORM public API/semantics | `orm` | `grpc-auth` |
| Error presentation mechanics | `error-package` | `orm`, `rest-orc`, transitive consumers |

## Completion Report

Untuk perubahan lintas repository, laporan akhir harus menyebutkan:

- Repository dan file yang berubah.
- Owner behavior dan alasan penempatannya.
- Contract serta compatibility impact.
- Urutan perubahan dan publishing yang diperlukan.
- Test per repository dan integration verification.
- Risiko atau keputusan yang masih terbuka.
