# Testing Standards

Dokumen ini menetapkan aturan testing bersama untuk Practice Architecture
System. Detail runner, scenario type, assertion library, mock framework, dan
nama subdirectory tetap mengikuti convention repository masing-masing.

## Test Root dan Struktur Existing

- Seluruh test project harus berada di salah satu root-level directory:
  `test/`, `tests/`, `test_scenario/`, atau `test_scenarios/`.
- Ikuti test root yang sudah digunakan repository. Jangan membuat root kedua
  dan jangan melakukan migrasi massal sebagai side effect feature lain.
- Current convention adalah `tests/` untuk `rest-orc`, `grpc-auth`, dan
  `api-contracts`; `test_scenarios/` untuk `orm`; serta `tests/` sebagai
  convention yang ditetapkan untuk test baru di `error-package`.
- Kelompokkan test berdasarkan layer, module, atau use case production yang
  diuji. Nama layer dapat berupa `handler`, `services`, `features`,
  `repositories`, `clients`, `vendors`, `integration`, atau nama yang memang
  dipakai repository.
- Test tidak diletakkan berdampingan dengan production file kecuali toolchain,
  framework, atau kebutuhan white-box secara teknis mewajibkannya. Pengecualian
  harus dijelaskan dan tidak boleh digunakan untuk mengekspos production API
  hanya demi test.

Contoh bentuk konseptual; sesuaikan nama singular/plural dan layer dengan
repository:

```text
<test-root>/
├── repositories/
│   └── user/
│       ├── runner_test.go
│       ├── test_scenarios/
│       ├── mocks/
│       └── fixtures/
├── services/
│   └── auth/
│       ├── runner_test.go
│       ├── test_scenarios/
│       ├── mocks/
│       └── fixtures/
├── handler/
│   └── user/
│       ├── runner_test.go
│       ├── test_scenarios/
│       ├── mocks/
│       └── fixtures/
└── integration/
    └── login/
        ├── runner_test.go
        ├── test_scenarios/
        ├── mocks/
        └── fixtures/
```

## Unit Test Terisolasi

- Setiap perubahan behavior harus memiliki unit test yang isolated,
  deterministic, dan order-independent.
- Satu scenario menguji satu behavior utama dengan Arrange, Act, dan Assert yang
  jelas.
- Unit test tidak boleh mengakses real network, real service, real database,
  wall clock, random source, environment lokal, filesystem eksternal, atau
  shared global state yang tidak dikontrol.
- Gunakan dependency injection melalui contract yang memang dibutuhkan code.
  Jangan membuat production API atau branch behavior khusus hanya untuk test.
- Gunakan mock, fake, stub, fixed clock, fixed identifier, fixture, atau
  in-memory implementation sesuai kebutuhan dan pattern repository.
- Verifikasi tidak hanya return value, tetapi juga dependency call, argument,
  jumlah dan urutan call, side effect, state transition, commit/rollback, serta
  context propagation ketika hal tersebut bagian dari behavior.

## Database dan ORM

- Untuk code yang membangun SQL melalui `orm`, gunakan API `DryRun*` yang
  tersedia untuk memverifikasi query, arguments, placeholder/dialect, dan mode
  tanpa membuka koneksi database.
- Gunakan `sqlmock` bila test perlu memverifikasi behavior pada boundary
  `database/sql`, seperti query execution, returned rows, begin, commit,
  rollback, close, atau dependency error. `sqlmock` digunakan bila diperlukan,
  bukan sebagai pengganti `DryRun*` untuk seluruh SQL-building test.
- Jangan menambahkan dependency `orm` atau `sqlmock` pada repository yang tidak
  memiliki database responsibility hanya untuk memenuhi aturan ini.
- Database scenario harus mencakup success, no rows/not found, malformed atau
  scan error bila relevan, query error, dan transaction failure pada tahap yang
  dapat terjadi.
- Tidak ada unit maupun integration test terkontrol yang menyimpan data ke
  database sungguhan.

## Client Service Eksternal

Jika code memanggil HTTP, gRPC, atau client service lain, mock pada contract atau
transport boundary harus menyediakan scenario yang relevan berikut:

- success dengan response yang valid;
- timeout, deadline exceeded, atau cancellation;
- service unreachable, connection failure, atau transport failure;
- non-success response yang tetap membawa response body dan status/error code;
- nil, empty, atau malformed response bila contract memungkinkan; dan
- verifikasi request mapping, metadata/header, context, serta error mapping.

Jangan melakukan call ke service asli. Mock response body dan code harus cukup
realistis untuk menguji mapping, tetapi tidak boleh berisi credential atau data
sensitif.

## Integration Test Terkontrol

Istilah integration test dalam sistem ini berarti menjalankan beberapa komponen
internal secara bersama-sama—misalnya handler, service/feature, repository
adapter, dan error mapper—dengan seluruh external boundary tetap di-mock.

- Integration test tidak boleh mengakses real network, real service, atau real
  database.
- Gunakan production wiring sejauh aman, lalu inject mock/fake pada database,
  message broker, clock, filesystem eksternal, dan downstream client boundary.
- Minimal sediakan success scenario dan error scenario yang membuktikan
  propagation/mapping lintas layer.
- Tambahkan concurrency/race scenario hanya jika code menggunakan goroutine,
  parallel access, shared cache, global/shared mutable state, atau mekanisme
  concurrency lain.
- Untuk perubahan concurrency/shared state, jalankan test yang relevan dengan
  race detector, misalnya `go test -race <target>`.
- Integration test harus tetap deterministic dan dapat dijalankan berulang kali
  tanpa cleanup environment eksternal.

## Mock, Fixture, dan Scenario

- Letakkan mock di dalam subtree test layer/module yang menggunakannya, pada
  `mock/` atau `mocks/` sesuai convention repository. Jangan mengumpulkan mock
  khusus satu layer ke global mock directory.
- Shared mock hanya diletakkan di shared test directory bila benar-benar dipakai
  oleh lebih dari satu layer/module.
- Mock merepresentasikan contract, bukan detail internal concrete
  implementation. Buat expectation yang cukup ketat terhadap method, argument,
  call count, order, dan injected failure yang relevan.
- Fixture berisi input, response body contoh, expected value, dan reusable test
  data. Jangan menyembunyikan expected result di dalam mock implementation.
- Pertahankan runner/scenario convention existing. Repository yang menggunakan
  runner tipis dan source scenario importable harus melanjutkan pola tersebut;
  repository yang menggunakan `*_test.go` langsung di `test_scenarios/` tetap
  mengikuti pola existing.

## Framework dan Dependency Testing

- Mock dan dependency injection boleh dibuat manual atau menggunakan library
  testing yang tersedia.
- Gunakan framework assertion, mocking, runner, builder, dan scenario contract
  yang sudah dipakai repository. Jangan mencampur framework baru tanpa
  kebutuhan yang tidak dapat dipenuhi pola existing.
- Dependency testing baru boleh ditambahkan bila kebutuhan test terukur dan
  pola existing tidak mencukupi. Jangan menambah dependency runtime untuk
  kebutuhan test.
- Jangan mengubah struktur test, mengganti framework, atau memigrasikan mock
  existing kecuali diminta secara eksplisit.

## Verification Minimum

1. Jalankan formatter untuk file test yang berubah.
2. Jalankan targeted unit test untuk module/use case yang berubah.
3. Jalankan targeted integration test bila request path lintas layer berubah.
4. Jalankan full test suite repository setelah targeted test lulus.
5. Jalankan race detector jika perubahan menyentuh concurrency/shared state.
6. Pastikan tidak ada network/database call asli, flaky timing dependency,
   leaked goroutine, unmet mock expectation, atau test-order dependency.
