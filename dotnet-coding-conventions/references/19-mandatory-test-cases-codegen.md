# Phần XX. Test Case Bắt Buộc & Sinh Code Test (Mandatory Test Cases & Test Code Generation)

Phần XVI (Rule 37) đã quy định *code mới phải có test*. Phần này bổ sung **quy trình bắt buộc**: với mọi thay đổi ở tầng Repository hoặc các thay đổi quan trọng (luồng tiền, trạng thái, nghiệp vụ lõi), AI Agent phải **viết test case dạng bảng trước**, sau đó **sinh ngay code test tự động tương ứng** với từng test case. Không được dừng lại ở bảng test case.

---

## 47. Thay đổi Repository & nghiệp vụ quan trọng bắt buộc viết test case rồi sinh code test

> 💡 **Ghi chú định danh**: Trong một số tài liệu quy chuẩn nội bộ, quy tắc này được đánh số là **Rule 43** (nối tiếp ngay sau Rule 42 của Phần XVII - Quản Trị Convention). Trong bộ skill .NET Enterprise tổng thể (đã bao gồm Zero-Allocation Rule 43–45 và Chuẩn lỗi RFC 9457 Rule 46), quy tắc này được định danh chính thức là **Rule 47**.

### Vì sao

| | |
| --- | --- |
| **Lợi ích** | Repository và nghiệp vụ lõi (tiền, trạng thái, tồn kho, idempotency) là nơi bug gây hậu quả nặng nhất. Viết test case dạng bảng trước giúp liệt kê đủ happy path / negative / edge / idempotent trước khi code, tránh bỏ sót nhánh. Sinh code test ngay sau đó biến đặc tả thành kiểm thử thực thi được, bắt hồi quy mỗi lần refactor (Rule 24) hoặc nâng cấp .NET. |
| **Rủi ro nếu vi phạm** | Chỉ liệt kê test case trên giấy rồi không code test → đặc tả chết, không ai chạy. Sửa repository mà không có test → thay đổi query, transaction, concurrency token âm thầm làm sai số dư/trạng thái và chỉ lộ trên production. |
| **Rủi ro khi áp dụng** | Viết test case/ code test tốn thời gian hơn khi gấp. Giảm thiểu: ưu tiên cover nhánh nghiệp vụ và error path trước (Rule 37.2 §8), tái dùng builder/fixture dữ liệu, không chạy theo coverage hình thức. |

### 47.1. Khi nào BẮT BUỘC áp dụng quy trình này

"Thay đổi quan trọng" = thay đổi mà **nếu sai sẽ gây hậu quả không hồi phục được (mất tiền, sai trạng thái) hoặc khó phát hiện**. Áp dụng quy trình khi task chạm vào **bất kỳ** nhóm nào sau đây:

1. **Repository & tầng truy cập dữ liệu**: thêm/sửa/xoá method của bất kỳ `Repository` nào; đổi query (`Where`, `Include`, projection, phân trang), transaction/Unit of Work, concurrency token, raw SQL, `ExecuteUpdateAsync`/`ExecuteDeleteAsync` (Rule 25, 30).
2. **Luồng tiền** — sai là mất tiền thật: tạo/hoàn/huỷ thanh toán, refund, trừ/cộng số dư, đối soát, tính phí, làm tròn tiền (`decimal` + `MidpointRounding`) (Rule 25.2, 35).
3. **Chuyển trạng thái (state machine)** — sai gây đơn treo, ship nhầm: chuyển trạng thái đơn hàng/thanh toán/subscription, cập nhật tồn kho, workflow Saga + bù trừ (Rule 25.3, 25.5).
4. **Logic phân nhánh nghiệp vụ lõi**: Strategy/Handler phân loại nghiệp vụ (Rule 24), validator FluentValidation (Rule 17), business rule tính toán điều kiện (giá, giảm giá, điểm thưởng, hạn mức).
5. **Idempotency & chống trùng** — sai là double-charge/double-ship: xử lý idempotency key, dedupe consumer/callback, retry (Rule 39).
6. **Tích hợp đối tác & ranh giới hệ thống**: gọi/nhận API đối tác, parse/map response, verify chữ ký callback (IPN/webhook), Outbox publish event (Rule 19, 25.4, 32).
7. **Bảo mật & kiểm soát truy cập**: authentication, authorization/policy, kiểm tra quyền sở hữu tài nguyên, so sánh chữ ký, sinh token/OTP (Rule 32).
8. **Bug fix**: bắt buộc có test tái hiện bug, fail trước khi sửa (Rule 37.2 §7).

> **Nguyên tắc chốt (tie-breaker)**: nếu không chắc một thay đổi có "quan trọng" hay không, mặc định **CÓ** và áp dụng quy trình.

**KHÔNG bắt buộc** (tránh áp dụng máy móc, trừ khi thay đổi kéo theo logic ở các nhóm trên):

- Đổi tên biến/symbol, format/style, sửa comment hoặc XML doc.
- Đổi nội dung log message (không đổi luồng xử lý).
- DTO thuần chỉ khai báo property (không có logic/validator đi kèm).
- Đổi hằng số cấu hình không ảnh hưởng logic nghiệp vụ.
- Wiring DI đơn thuần (đăng ký service, không đổi hành vi).

### 47.2. Quy trình hai bước (không được bỏ bước nào)

**Bước 1 — Viết test case dạng bảng** theo đúng template và quy tắc ở mục [47.4](#474-template--quy-tắc-viết-test-case-copy-từ-skill-test-case). Kết quả ghi ra file `.md` trong thư mục `docs/` của repo (xem mục 47.4). Bảng phải cover đủ các loại case: happy path, negative, edge, idempotent, và với repository là concurrency.

**Bước 2 — Sinh code test ngay** cho **từng dòng** test case ở Bước 1, dùng đúng bộ công cụ chuẩn (xUnit + Shouldly + NSubstitute + Testcontainers) và quy tắc của Rule 37. Mỗi dòng test case ánh xạ tối thiểu một `[Fact]`/`[Theory]`; tên test theo `MethodName_Scenario_ExpectedResult` phản ánh cột *TC Name* và *Expected*. Code test là file **`.cs`** nằm trong **test project** của solution (xem mục [47.2.1](#4721-code-test-sinh-ra-ở-đâu)).

Không được coi task hoàn thành khi mới xong Bước 1. Bảng test case mà thiếu code test tương ứng là **vi phạm Rule 47 (Rule 43 gốc)**.

### 47.2.1. Code test sinh ra ở đâu

> ⚠️ **TUYỆT ĐỐI KHÔNG** viết code test (C#) vào file `docs/test-case-*.md`. File `.md` ở Bước 1 **chỉ** chứa bảng test case để con người/BA đọc. Code test thực thi được là file **`.cs`** và phải nằm trong **test project** của solution. Viết code `.cs` vào file `.md`, hoặc để lẫn code test trong thư mục `docs/`, là **vi phạm quy tắc**.

Quy ước vị trí file code test (theo Rule 37.2 §1):

| Loại code bị sửa | Test project | Hậu tố | Đặt file test |
| --- | --- | --- | --- |
| Repository, truy vấn EF, transaction, concurrency | `{Project}.IntegrationTests` | `IntegrationTests` | Phản chiếu cây namespace của class bị test |
| Service, Strategy/Handler, Validator, Value Object (unit) | `{Project}.Tests` / `{Project}.UnitTests` | `Tests` | Phản chiếu cây namespace của class bị test |

Nguyên tắc đặt file:

- **Test project là project `.csproj` riêng** nằm trong thư mục test của solution (ví dụ `test/` hoặc `tests/`), **không nằm trong** cùng project với code sản phẩm và **không nằm trong** `docs/`.
- **Phản chiếu cấu trúc source**: class test đặt cùng namespace/đường dẫn tương đối với class đang test. Ví dụ class `Acme.Payment.Infrastructure.Repositories.WalletRepository` → file test `test/Acme.Payment.IntegrationTests/Repositories/WalletRepositoryTests.cs`.
- **Tên file = tên class test = `{ClassUnderTest}Tests`** (Rule 37.2 §1), một class test cho một class bị test.
- Nếu test project tương ứng chưa tồn tại thì **tạo mới** `.csproj` kế thừa `Directory.Build.props` chuẩn (Rule 36) và tham chiếu project sản phẩm.
- Dữ liệu dùng chung (builder, fixture, `WebApplicationFactory`) đặt trong chính test project, không rải trong `docs/`.

Tóm lại:
- File `docs/test-case-*.md` → **bảng test case** (đặc tả, Bước 1).
- File `test/{Project}.Tests|IntegrationTests/.../{Class}Tests.cs` → **code test** (thực thi, Bước 2).

### 47.3. Ánh xạ test case → code test

- Mỗi dòng bảng → ít nhất một test method. Các case cùng nghiệp vụ khác dữ liệu nên gộp bằng `[Theory]` + `[InlineData]`/`[MemberData]`.
- Cột **TC Name** ("Kiểm tra ...") → phần `_Scenario_` của tên test.
- Cột **Expected** → phần `_ExpectedResult` và nội dung `Assert` (Shouldly).
- Cột **Note** (edge/idempotent/cần confirm BA) → comment song ngữ trong test (Rule 7.1) hoặc `TODO(<JIRA>)` nếu còn chờ xác nhận.
- Repository → integration test với **Testcontainers** (đúng loại DB production), **cấm EF InMemory** cho logic transaction/constraint/concurrency (Rule 37.2 §5).
- Case concurrency → chạy song song trên hai `DbContext` riêng, assert đúng một lệnh thắng (mẫu `DebitAsync_TwoConcurrentDebitsExceedBalance_OnlyOneSucceeds`, Rule 37.2).

✅ **ĐÚNG** — Từ 2 dòng test case repository sinh ra code test:

Bảng test case (`docs/test-case-wallet-repository.md`):

| TC Name | Expected | Note |
| --- | --- | --- |
| Kiểm tra trừ số dư khi đủ tiền | Số dư giảm đúng, trả về thành công | Happy path |
| Kiểm tra hai lệnh trừ song song vượt số dư | Chỉ một lệnh thành công, số dư không âm | 🔄 Concurrency |

Code test tương ứng — file `test/Acme.Payment.IntegrationTests/Repositories/WalletRepositoryTests.cs` (KHÔNG phải trong `docs/`):

```csharp
public sealed class WalletRepositoryTests : IAsyncLifetime
{
    private readonly MsSqlContainer _sqlContainer = new MsSqlBuilder().Build();

    public Task InitializeAsync() => _sqlContainer.StartAsync();

    public Task DisposeAsync() => _sqlContainer.DisposeAsync().AsTask();

    // Kiểm tra trừ số dư khi đủ tiền
    // Debit succeeds when the balance is sufficient
    [Fact]
    public async Task DebitAsync_BalanceIsSufficient_DecrementsBalance()
    {
        WalletRepository sut = await CreateSutWithWalletAsync(balance: 100m);

        DebitResult result = await sut.DebitAsync(amount: 80m, CancellationToken.None);

        result.Succeeded.ShouldBeTrue();
        (await sut.GetBalanceAsync(CancellationToken.None)).ShouldBe(20m);
    }

    // Kiểm tra hai lệnh trừ song song vượt số dư
    // Two concurrent debits exceeding the balance: only one wins
    [Fact]
    public async Task DebitAsync_TwoConcurrentDebitsExceedBalance_OnlyOneSucceeds()
    {
        // ... seed ví 100, chạy song song hai lệnh trừ 80 trên hai DbContext riêng
        // ... assert đúng một lệnh thành công và số dư cuối là 20
    }
}
```

❌ **SAI**:
- Chỉ nộp bảng test case `.md` mà không có file test `.cs` nào.
- **Viết code test C# vào trong file `docs/test-case-*.md`** (file `.md` chỉ chứa bảng test case).
- Đặt file `.cs` test trong thư mục `docs/` hoặc chung project với code sản phẩm thay vì test project (`{Project}.Tests`/`{Project}.IntegrationTests`).
- Sinh code test nhưng bỏ sót case concurrency/idempotent đã liệt kê trong bảng.

---

## 47.4. Template & quy tắc viết test case (copy từ skill `test-case`)

> Phần dưới đây sao chép nguyên tắc từ skill `test-case` để skill `dotnet-coding-conventions` tự đủ ngữ cảnh khi áp dụng Bước 1. Khi có xung đột về công cụ sinh *code test*, ưu tiên Rule 37 của skill này.

### Quy tắc bắt buộc về đầu ra (Output)

**Kết quả bảng test case BẮT BUỘC được ghi ra một file `.md` trong thư mục `docs/`.**

- Đường dẫn: `docs/test-case-<ten-tinh-nang-hoac-ma-ticket>.md`
  - Ví dụ: `docs/test-case-wallet-repository.md`, `docs/test-case-PAYMENT-1234.md`
  - Tên file dùng chữ thường, không dấu, nối bằng dấu `-` (kebab-case).
- Nếu thư mục `docs/` chưa tồn tại thì tạo mới.
- Nếu file đã tồn tại: hỏi người dùng muốn ghi đè hay tạo file mới với hậu tố `-v2`, `-v3`...
- KHÔNG chỉ in bảng test case ra chat rồi dừng lại. Việc tạo file `.md` là bước hoàn thành bắt buộc, **và sau đó phải sinh code test (Bước 2 / Rule 47.2)**.
- Sau khi tạo file, báo lại cho người dùng đường dẫn file đã xuất.

#### Cấu trúc file `.md` cần có

```markdown
# Test Case — <Tên tính năng / Mã ticket>

- **Nguồn**: <Confluence / Jira ticket / Summary>
- **Ngày tạo**: <yyyy-mm-dd>
- **Người tạo**: AI Agent

## Bảng Test Case

| TC Name | Expected | Note |
|---------|----------|------|
| ... | ... | ... |
```

Nếu có nhiều nhóm scenario, chia thành nhiều mục `## <Tên nhóm>` riêng, mỗi mục một bảng.

### Template bảng

| TC Name | Expected | Note |
|---------|----------|------|
| | | |

### Quy tắc viết test case

- **TC Name**: Luôn bắt đầu bằng **"Kiểm tra"** + mô tả scenario. Ví dụ: `Kiểm tra KH không thuộc KH Sim FPT truy cập xác thực`.
- **Expected**: Kết quả mong đợi sau khi thực hiện.
- **Note**: Ghi chú thêm nếu cần (edge case, điều kiện đặc biệt, cần confirm BA).

### Nguồn để viết test case

1. **Từ Confluence** — đọc phần Mô tả UI / Acceptance Criteria.
2. **Từ Jira ticket** — đọc Description (Steps, Expected, Actual).
3. **Từ Summary ticket** — suy luận các case happy path + negative.
4. **Từ diff code** — với thay đổi repository/nghiệp vụ không có ticket chi tiết, suy luận case từ chính phần code bị sửa (nhánh điều kiện, transaction, concurrency token).

### Các loại test case cần cover

- ✅ Happy path — luồng thành công.
- ❌ Negative — luồng thất bại, lỗi.
- 🔄 Edge case — giá trị biên, dữ liệu đặc biệt.
- 🔁 Idempotent — thực hiện nhiều lần kết quả không đổi.
- 🔀 Concurrency — (bắt buộc với repository ghi dữ liệu dùng chung) hai thao tác song song, đúng một thắng.
- 📱 UI — hiển thị đúng design, ẩn/hiện đúng điều kiện.
- 🔌 API — payload đúng, response đúng.

### Ví dụ thực tế

#### Từ Confluence — Xác thực thuê bao VNeid

| TC Name | Expected | Note |
|---------|----------|------|
| Kiểm tra KH không thuộc KH Sim FPT truy cập xác thực | Hiển thị popup yêu cầu đăng nhập tài khoản FPT | |
| Kiểm tra KH Sim FPT không thuộc đối tượng cần xác thực | Hiển thị thông báo không thuộc đối tượng cần xác thực | Cần confirm lại flow với BA |
| Kiểm tra KH chọn phương thức VNeid | Hiển thị danh sách phương thức, chọn được VNeid | Phương thức hiển thị theo config CMS |
| Kiểm tra nhập CCCD đúng định dạng | Chuyển sang màn xác nhận chia sẻ thông tin VNeid | |
| Kiểm tra nhập CCCD sai định dạng | Hiển thị thông báo lỗi, không cho tiếp tục | |
| Kiểm tra KH xác nhận chia sẻ thông tin VNeid | Gọi API chia sẻ thành công, chuyển sang bước liveness | |
| Kiểm tra liveness thất bại | Hiển thị thông báo lỗi, cho phép thử lại | ánh sáng kém, che mặt, ảnh giả |

#### Từ Jira ticket — Loyalty điểm khi login app

| TC Name | Expected | Note |
|---------|----------|------|
| Kiểm tra login web chưa login app chưa follow Zalo OA rồi login app | isAppFptShop = TRUE, isZaloFptShop = FALSE | Chỉ truyền isAppFptShop = TRUE |
| Kiểm tra login app thành công rồi dùng điểm Loyalty | Cho phép sử dụng điểm Loyalty | isAppFptShop = TRUE là điều kiện đủ |
| Kiểm tra login app nhiều lần liên tiếp | Trạng thái không thay đổi, vẫn cho phép dùng điểm | 🔁 Idempotent |

