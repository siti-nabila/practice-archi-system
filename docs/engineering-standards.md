# Engineering Standards

Dokumen ini menetapkan aturan engineering bersama untuk seluruh komponen
Practice Architecture System. Repository komponen boleh menambahkan aturan yang
lebih spesifik melalui `AGENTS.md`, tetapi tidak boleh diam-diam melemahkan
compatibility, ownership, atau quality boundary yang ditetapkan di sini.

## Prinsip Dasar

- Terapkan SOLID dan DRY secara pragmatis, bukan sebagai alasan untuk menambah
  layer atau abstraction tanpa kebutuhan terukur.
- Setiap package, type, dan fungsi harus mempunyai tanggung jawab yang jelas.
- Arahkan dependency ke contract yang relevan pada boundary package.
- Gunakan kembali code hanya ketika behavior dan alasan perubahannya memang
  sama. Duplication yang kecil dan eksplisit lebih baik daripada abstraction
  yang menyatukan dua tanggung jawab berbeda.
- Buat perubahan sekecil yang diperlukan dan pertahankan behavior di luar scope.
- Utamakan alur yang dapat dibaca sebagai urutan keputusan bisnis atau langkah
  teknis yang jelas.

## Keterbacaan Alur

- Gunakan guard clause atau early return untuk mengurangi nesting.
- Pisahkan validation, mapping, orchestration, persistence, network call,
  transaction, dan status transition ketika pencampurannya menyulitkan pembaca.
- Nama helper harus menjelaskan intent. Jangan memecah fungsi menjadi banyak
  helper kecil hanya untuk mengurangi jumlah baris.
- Top-level use case sebaiknya memperlihatkan urutan flow tanpa mengharuskan
  pembaca membuka setiap helper untuk memahami keputusan utamanya.
- Hindari boolean flag yang membuat satu fungsi mempunyai beberapa mode behavior
  yang tidak berkaitan.
- Gunakan constant atau typed value untuk status, action, role, protocol key,
  dan business value yang berulang.

## Naming Go

- Gunakan `PascalCase` untuk exported identifier dan `camelCase` untuk
  unexported identifier.
- Jangan menggunakan `snake_case` untuk identifier Go. Nama file tetap mengikuti
  convention repository. `snake_case` hanya boleh digunakan untuk nama file
  yang membutuhkan lebih dari satu kata, bukan untuk function, method, type,
  variable, atau identifier Go lainnya.
- Pertahankan initialism umum seperti `ID`, `HTTP`, `URL`, `API`, `JWT`, `RPC`,
  `SQL`, dan `TLS`, misalnya `userID`, `HTTPClient`, dan `ParseJWT`.
- Nama fungsi dan method harus menjelaskan aksi serta objeknya, misalnya
  `RegisterUser`, `FindProfileByUserID`, atau `MapGRPCError`.
- Hindari nama samar seperti `Do`, `Exec`, `Process`, `Handle`, `Helper`, atau
  `Util` jika intent belum jelas dari receiver dan package.
- Jangan menggunakan prefix `I` untuk interface atau suffix `Impl` untuk
  concrete type. Gunakan `AuthService` untuk contract dan `authService` untuk
  concrete implementation.
- Ikuti istilah domain dan bahasa yang sudah konsisten pada public contract.
  Jangan mengganti istilah hanya karena sinonim terlihat lebih singkat.

## `impl.go` sebagai Entry Package

`impl.go` adalah convention nama file, bukan convention nama identifier. File
ini boleh digunakan ketika feature, service, client, atau adapter memerlukan
satu entry/composition point yang mudah ditemukan.

`impl.go` boleh memuat:

- public contract atau interface package;
- concrete type unexported yang memenuhi contract tersebut;
- exported constructor seperti `NewAuthService`;
- dependency fields dan dependency assembly;
- compile-time interface assertion bila digunakan project; dan
- helper package-private berukuran kecil yang hanya diperlukan untuk setup atau
  initialization.

`impl.go` tidak boleh menjadi tempat menumpuk seluruh business logic. Method
register, login, query, mapping, atau behavior lain ditempatkan di file yang
menjelaskan use case atau intent sesuai convention repository.

Convention ini tidak memaksa satu bentuk constructor. Constructor boleh
menerima dependency dari luar atau merakit dependency yang memang dimiliki
package, sesuai architecture dan testability repository. Convention ini juga
tidak membenarkan penyimpanan `context.Context` berumur request di object
berumur panjang; ownership dan lifetime context harus tetap benar untuk service
tersebut.

Contoh bentuk yang dimaksud:

```go
type AuthService interface {
	Register(req domain.AuthRequest) (token *string, err error)
	Login(req domain.AuthRequest) (token *string, err error)
}

type authService struct {
	authWriter domain.AuthWriter
	authReader domain.AuthReader
}

func NewAuthService(
	authWriter domain.AuthWriter,
	authReader domain.AuthReader,
) AuthService {
	return &authService{
		authWriter: authWriter,
		authReader: authReader,
	}
}
```

Contoh tersebut menunjukkan bentuk file dan naming, bukan signature wajib.
Repository tetap menentukan apakah constructor mengembalikan interface atau
concrete type dan bagaimana dependency dibuat.

## Helper Utility

- Setiap helper utility baru ditempatkan di `pkg/helper/` atau `pkg/helpers/`
  dan dikelompokkan berdasarkan tujuan fungsi, bukan berdasarkan feature
  consumer. Keduanya valid; ikuti convention yang sudah digunakan repository
  dan jangan membuat kedua directory sekaligus dalam satu repository.
- Gunakan satu nama file yang pendek, langsung, dan mudah dicari. Contoh:
  helper formatting string dapat ditempatkan di `pkg/helper/string.go` atau
  `pkg/helpers/string.go`, sedangkan helper document menggunakan
  `document.go` pada directory helper yang dipilih repository.
- Jika nama file membutuhkan dua kata, gunakan lowercase `snake_case`, misalnya
  `string_format.go` atau `document_name.go`.
- Jangan menggunakan nama catch-all seperti `utils.go`, `common.go`,
  `misc.go`, atau `helper.go`. Nama folder sudah menjelaskan bahwa isinya helper;
  nama file harus menjelaskan tujuannya.
- Nama function di dalam file tetap mengikuti Go naming: `FormatString`,
  `BuildDocumentName`, atau bentuk `camelCase` untuk unexported function. Jangan
  menggunakan nama seperti `format_string`.
- Directory helper hanya berisi utility reusable yang bebas dari business rule
  dan tidak bergantung pada feature tertentu. Business-specific logic tetap
  berada di layer atau package pemiliknya.
- Helper setup kecil yang hanya digunakan untuk composition/initialization
  package boleh tetap berada di `impl.go`; helper tersebut tidak perlu dipindah
  ke directory helper jika tidak reusable.
- Sebelum membuat file helper baru, periksa apakah file berdasarkan tujuan yang
  sama sudah tersedia. Tambahkan ke file tersebut selama tanggung jawabnya tetap
  fokus dan mudah dibaca.
- Jangan mengganti `helper` menjadi `helpers`, atau sebaliknya, sebagai side
  effect perubahan lain. Rename harus memiliki scope eksplisit, memperbarui
  seluruh import/consumer, dan diverifikasi dengan test yang relevan.

## Backward Compatibility

- Pertahankan public API, protobuf schema dan field semantics, database schema,
  serialized data, configuration key, error key/code/message, serta protocol
  mapping kecuali breaking change diminta secara eksplisit.
- Untuk protobuf, jangan menggunakan kembali field number yang pernah dipakai.
  Utamakan additive change dan ikuti workflow generation repository contract.
- Jangan mengubah error identity, fallback, default value, side-effect order,
  transaction boundary, atau context propagation secara diam-diam.
- Jika perubahan contract lintas repository diperlukan, identifikasi owner,
  producer, consumer, publishing order, dan migration path sebelum implementasi.
- Compatibility shim harus memiliki tujuan dan batas waktu migrasi yang jelas;
  jangan menjadikannya behavior permanen tanpa keputusan.

## Definition of Done

Sebelum menyelesaikan perubahan code:

1. Pastikan implementation berada di repository dan layer pemilik behavior.
2. Pastikan nama menjelaskan intent dan flow utama tetap mudah dipindai.
3. Pastikan helper atau abstraction baru memiliki kebutuhan nyata.
4. Periksa dampak compatibility pada seluruh public boundary yang tersentuh.
5. Tambahkan atau perbarui test untuk success, validation, dependency failure,
   dan side effect yang relevan.
6. Jalankan formatter, targeted test, full test yang relevan, dan static
   analysis sesuai aturan repository.
7. Review diff agar tidak ada refactor, generated artifact, atau perubahan di
   luar scope.
