# Practice Architecture System

Repository ini adalah sumber konteks lintas repository untuk mini-project yang
terdiri dari beberapa runtime service, shared contract, dan shared library.

Repository ini tidak berisi business implementation atau runtime application.
Implementasi tetap berada di repository komponen yang memiliki behavior
tersebut.

## Tujuan

- Menjelaskan topologi dan aliran request antar-komponen.
- Menetapkan ownership data, behavior, contract, dan error.
- Menjadi sumber kebenaran untuk perubahan lintas repository.
- Menyimpan source system skill yang dapat dipasang ke Codex secara terpisah.

## Komponen Saat Ini

- `rest-orc`: HTTP boundary dan orchestration service.
- `grpc-auth`: authentication, identity, profile, role, JWT, dan persistence.
- `api-contracts`: shared protobuf serta catalog error message yang menjaga
  bentuk error tetap konsisten dan dapat dipetakan oleh service/HTTP boundary.
- `orm`: shared database, query, transaction, dan pagination library.
- `error-package`: shared library untuk memuat, membentuk, melokalkan, dan
  menampilkan error response bagi service atau package consumer.

## Dokumen

- [System map](docs/system-map.md)
- [Component catalog](docs/component-catalog.md)
- [Engineering standards](docs/engineering-standards.md)
- [Testing standards](docs/testing-standards.md)
- [Cross-service changes](docs/cross-service-changes.md)

## Skill

Source system skill berada di
`skills/develop-practice-archi-system/SKILL.md`. Source ini disimpan di Git,
tetapi belum otomatis terpasang ke directory skill Codex. Mekanisme instalasi
atau sinkronisasi ditentukan terpisah agar tidak ada dua source yang diedit
secara independen.

## Batas Repository

- `AGENTS.md` di repository ini hanya berlaku untuk dokumentasi sistem dan
  source system skill.
- Setiap service atau library tetap memiliki aturan repository sendiri.
- Perubahan repository sibling tidak tersirat oleh perubahan dokumentasi di
  repository ini.
