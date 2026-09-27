# Component Catalog

Catalog ini menjelaskan ownership dan boundary tiap komponen. Detail internal
tetap berada di repository pemiliknya.

## `rest-orc`

### Identity

- Type: runtime service
- Go module: `github.com/siti-nabila/rest-orc`
- Inbound protocol: HTTP
- Primary outbound protocol: gRPC

### Purpose

Menyediakan HTTP API dan mengorkestrasikan downstream service untuk client.

### Owns

- Public HTTP routes dan transport DTO mapping.
- Authentication/authorization middleware pada HTTP boundary.
- Use-case orchestration yang dimiliki feature HTTP.
- Downstream adapters dan HTTP response/error mapping.
- Application wiring dan lifecycle orchestrator.

### Does Not Own

- User database dan transaction persistence.
- Password hashing dan verification.
- JWT issuance atau storage.
- Shared protobuf definitions.
- Generic database abstraction.

### Inbound dan Outbound

- Inbound: frontend atau API client melalui HTTP.
- Outbound: `grpc-auth` melalui generated `api-contracts` client.
- Compile-time: `api-contracts` dan `error-package`.

### Critical Invariants

- Tidak menggunakan local database untuk current architecture.
- Request context diteruskan ke downstream.
- Generated protobuf berhenti pada client/adapter boundary.
- Public HTTP response tidak membocorkan internal downstream errors.

## `grpc-auth`

### Identity

- Type: runtime service
- Go module: `github.com/siti-nabila/grpc-auth`
- Inbound protocol: unary gRPC
- Persistence: PostgreSQL

### Purpose

Mengelola authentication, identity, profile, role, JWT, dan user persistence.

### Owns

- Register, login, `Me`, profile update, dan list/search user behavior.
- Password hashing dan verification.
- JWT issuance dan request session identity.
- User/profile/role database access dan transaction boundary.
- Implementasi `UserService` dan `ProfileService`.

### Does Not Own

- Public HTTP routes.
- Shared protobuf source dan field numbering.
- Generic ORM behavior.
- System-wide orchestration documentation.

### Inbound dan Outbound

- Inbound: gRPC requests berdasarkan `api-contracts`.
- Outbound: PostgreSQL melalui `orm`.
- Compile-time: `api-contracts`, `orm`, dan transitive error utilities.

### Critical Invariants

- Password tidak disimpan sebagai plaintext.
- Protected identity berasal dari request session, bukan user ID payload.
- Flow atomic memakai transaction yang sama untuk related writes.
- Internal database errors dipetakan ke shared error contract.

## `api-contracts`

### Identity

- Type: shared contract module
- Go module: `github.com/siti-nabila/api-contracts`
- Current local checkout: `api-contracts`

### Purpose

Menjadi source of truth untuk protobuf, generated Go contracts, shared error
keys/codes/messages, locale, paginator, dan mapping contract yang menjaga bentuk
error konsisten pada service maupun HTTP boundary.

### Owns

- `.proto` definitions dan field semantics.
- Generated Go protobuf code melalui generator.
- Shared error registries: key, code, localized messages, dan mapping metadata.
- Contract untuk representasi dan mapping error pada gRPC atau HTTP boundary.

### Does Not Own

- Runtime implementation RPC.
- User persistence.
- HTTP route behavior.
- Generic SQL execution.
- Generic runtime mechanics untuk memuat dan memformat error catalog.

### Consumers

- `grpc-auth`
- `rest-orc`

### Critical Invariants

- Generated files tidak diedit manual.
- Existing field numbers dan semantics tetap backward compatible.
- Breaking changes memerlukan keputusan dan migration plan eksplisit.
- Consumer diuji dengan local workspace dan published module version bila
  relevan.

## `orm`

### Identity

- Type: shared Go library
- Go module: `github.com/siti-nabila/orm`

### Purpose

Menyediakan reusable database/query/transaction behavior untuk beberapa SQL
dialects.

### Owns

- Query building dan parameter binding.
- Transaction adapters.
- Database pagination.
- Dialect behavior, mapping, lock, dan dry-run support.

### Does Not Own

- Authentication atau user business rules.
- Service-specific database schema ownership.
- Public HTTP atau gRPC contracts.

### Consumers

- `grpc-auth`

### Critical Invariants

- Query input tetap terparameterisasi atau melalui whitelist yang aman.
- Public library API menjaga backward compatibility.
- Context diteruskan ke database operation.
- Transaction selalu memiliki commit atau rollback path.

## `error-package`

### Identity

- Type: shared Go library
- Go module: `github.com/siti-nabila/error-package`

### Purpose

Menyediakan shared runtime library untuk memuat, membentuk, melokalkan, dan
menampilkan error response bagi service atau package dalam sistem.

### Owns

- Loading definisi error yang diberikan consumer atau shared catalog.
- Typed error dan error identity berdasarkan code dari catalog.
- Locale selection dan deterministic fallback behavior.
- Formatting error tunggal, collection, parameterized message, dan nested field
  errors.

### Does Not Own

- Public protobuf error schema.
- Shared business error key, code, dan localized message definitions.
- HTTP status atau gRPC status mapping.
- Service-specific business decisions.

### Consumers

- `rest-orc` secara langsung.
- `orm` secara langsung.
- `grpc-auth` secara transitif.

### Collaboration with `api-contracts`

- `api-contracts` mendefinisikan error apa yang ada dan bagaimana error tersebut
  harus konsisten serta dapat dipetakan.
- `error-package` mendefinisikan mekanisme generic untuk memuat dan menampilkan
  definisi itu pada runtime.
- Consumer atau boundary memilih catalog dan protocol mapping yang relevan.
- Agent tidak boleh menyalin business message ke `error-package` atau
  memindahkan generic presentation mechanics ke catalog tanpa alasan arsitektur
  yang eksplisit.

Current-state `api-contracts/pkg/dictionary` masih mengandung sebagian registry,
loading, dan runtime error types. Itu adalah overlap implementasi yang perlu
dipertahankan kompatibilitasnya sampai migration plan diminta dan disepakati;
boundary di atas tidak memberikan izin untuk refactor lintas repository.

### Critical Invariants

- Public API dan output formatting tetap backward compatible.
- Error identity dan cause tetap dapat diperiksa consumer.
- Output collection deterministik.
- Locale request tidak boleh menciptakan data race atau leakage antar-request.
- Library tetap bebas dari dependency transport dan business feature.
