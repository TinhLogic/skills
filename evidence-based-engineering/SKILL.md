---
name: evidence-based-engineering
description: >-
  Mandatory skill for evidence-based engineering and technical decision making.
  Strictly bans assumptions, guessing, and unverified expectations.
  Requires verifiable evidence (repo code/tests, official docs, library source code, verified GitHub issues/discussions, reproduction tests)
  before writing, modifying, or reviewing code.
  Tự động kích hoạt khi: đưa ra quyết định kỹ thuật, tra cứu thư viện, tích hợp API bên thứ ba,
  debug, refactor, kiểm chứng hành vi hệ thống, viết code hoặc review code.
  Nguyên tắc cốt lõi: Không suy diễn, không suy đoán, không kỳ vọng; không có căn cứ thì phải tra cứu và kiểm chứng thực nghiệm.
---

# Kỹ Thuật Dựa Trên Bằng Chứng (Evidence-Based Engineering)

> 🛑 **QUY TẮC BẮT BUỘC (ƯU TIÊN CAO NHẤT).** Quy tắc kỹ thuật dựa trên bằng chứng có mức ưu tiên cao nhất, áp dụng xuyên suốt mọi tác vụ kỹ thuật. Nếu có xung đột với các giả định hoặc thói quen code, nguyên tắc dựa trên bằng chứng luôn thắng. AI Agent phải áp dụng trước khi viết hoặc sửa bất kỳ dòng code nào.

Quy tắc về **cách ra quyết định kỹ thuật**: không suy diễn, không suy đoán, không kỳ vọng. Mọi khẳng định về hành vi của code, thư viện, framework, API đối tác phải có căn cứ kiểm chứng được. Không có căn cứ thì phải đi tìm cách thực tế mọi người đã giải quyết (GitHub, tài liệu chính thức, diễn đàn kỹ thuật), không được tự "đoán cho hợp lý".

---

## Nguyên Tắc Cốt Lõi: Không Suy Diễn, Suy Đoán, Kỳ Vọng (Không Có Căn Cứ Phải Tra Cứu)

### Vì sao cần áp dụng

| | |
| --- | --- |
| **Lợi ích** | Mỗi quyết định kỹ thuật truy vết được về nguồn cụ thể (code, tài liệu, issue, câu trả lời đã được kiểm chứng). Reviewer kiểm tra lại được thay vì tin cảm tính. Giảm lỗi do "tưởng là vậy" ở các chỗ dễ sai: hành vi EF Core, ABP, Kafka, chữ ký/IPN của đối tác thanh toán. |
| **Rủi ro nếu vi phạm** | AI Agent sinh ra API không tồn tại (hallucination), option không có ở phiên bản đang dùng, hoặc giả định sai về hành vi đối tác. Lỗi này compile được hoặc chạy được ở môi trường dev nhưng sai ở production, đặc biệt ở luồng tiền và chữ ký. |
| **Rủi ro khi áp dụng quá đà** | Tra cứu mọi thứ làm chậm công việc. Vì vậy chỉ bắt buộc khi **không có căn cứ sẵn** (xem mục 2: Thứ tự tìm căn cứ). Điều đã xác minh được từ code trong repo thì không cần tra web. |

---

## 1. Ba Điều Cấm

1. **Không suy diễn (infer)**: không kết luận hành vi của code/thư viện chỉ từ tên hàm, tên biến, hay "nhìn có vẻ".
2. **Không suy đoán (guess)**: không tự điền tên API, tham số, giá trị mặc định, mã lỗi, format field khi chưa thấy ở nguồn.
3. **Không kỳ vọng (assume)**: không giả định đối tác/thư viện "chắc sẽ trả về thế này", "chắc có retry sẵn", "chắc thread-safe".

Các cụm từ báo hiệu vi phạm khi viết kết luận hoặc comment: "chắc là", "có lẽ", "thường thì", "mặc định sẽ", "probably", "should work", "I assume". Gặp các cụm này thì dừng lại và đi tìm căn cứ.

---

## 2. Thứ Tự Tìm Căn Cứ (Dừng ở bước đầu tiên đủ trả lời)

| Bước | Nguồn | Dùng để |
| :---: | :--- | :--- |
| **1** | **Code và test trong repo** (đọc thật, kể cả caller/callee, migration, config) | Hành vi hiện tại của hệ thống |
| **2** | **Tài liệu chính thức** đúng phiên bản đang dùng (Microsoft Learn, docs ABP, docs thư viện, tài liệu tích hợp của đối tác) | Hợp đồng và hành vi được cam kết |
| **3** | **Mã nguồn của thư viện/framework** (dotnet/runtime, efcore, aspnetcore, abp...) | Khi tài liệu không nói rõ |
| **4** | **GitHub**: Issues, Discussions, Pull Requests, code mẫu trong repo thật | Cách người khác gặp và xử lý đúng vấn đề này; bug đã biết; workaround |
| **5** | **Diễn đàn**: Stack Overflow, GitHub Discussions, ABP Support, Reddit r/dotnet, blog kỹ thuật của maintainer | Kinh nghiệm thực tế khi bước 2 đến 4 chưa đủ |
| **6** | **Hỏi người dùng/owner** | Khi mọi nguồn trên không trả lời được, hoặc là ràng buộc nghiệp vụ nội bộ |

Nếu cả sáu bước không có căn cứ: **nói rõ "chưa có căn cứ"** và hỏi người dùng. Không được lấp chỗ trống bằng phỏng đoán.

---

## 3. Cách Tìm Trên GitHub và Diễn Đàn Kỹ Thuật

1. **Dùng từ khoá cụ thể**: tên exception, thông báo lỗi nguyên văn, tên API, phiên bản (`"DbUpdateConcurrencyException" ExecuteUpdateAsync efcore`), thay vì mô tả chung chung.
2. **Ưu tiên nguồn gốc**: issue/PR của chính repo thư viện (`dotnet/efcore`, `dotnet/aspnetcore`, `abpframework/abp`...) hơn bài blog thứ cấp.
3. **Đối chiếu phiên bản**: một câu trả lời cũ cho .NET Framework / EF6 / ABP bản cũ có thể sai với phiên bản dự án. Kiểm tra ngày đăng và phiên bản được nói tới.
4. **Cần ít nhất hai nguồn độc lập** cho quyết định ảnh hưởng tiền, bảo mật, dữ liệu; hoặc một nguồn chính thức/maintainer xác nhận.
5. **Câu trả lời chưa ai xác nhận** (không có upvote, không có người thử lại, không có maintainer) chỉ là gợi ý, phải tự kiểm chứng bằng cách chạy thử.
6. **Không dán nguyên code từ forum**: đọc hiểu, đối chiếu giấy phép nếu là đoạn dài, rồi viết lại theo coding convention của dự án.
7. **Không gửi code nội bộ, secret, dữ liệu khách hàng, thông tin đối tác** lên công cụ tìm kiếm hoặc diễn đàn. Rút gọn thành ví dụ tối thiểu, ẩn danh.

---

## 4. Kiểm Chứng Bằng Thực Nghiệm (Khi nguồn mâu thuẫn hoặc thiếu)

- Viết **test tái hiện** hoặc chạy thử đoạn code tối thiểu để quan sát hành vi thật, thay vì chọn theo cảm tính.
- Với hành vi DB (isolation, lock, constraint, concurrency): chạy trên Testcontainers đúng loại DB production, không suy ra từ InMemory.
- Với API đối tác: dùng môi trường sandbox hoặc vector mẫu trong tài liệu của đối tác (ví dụ chữ ký mẫu), không tự đoán thứ tự field hay cách encode.
- Kết quả thực nghiệm phải được ghi lại (test, hoặc mô tả trong MR).

---

## 5. Ghi Nguồn Trong Kết Quả (MR / ADR / Task Summary)

Khi một quyết định dựa vào tra cứu bên ngoài, ghi nguồn ở **mô tả MR** hoặc **ADR** (Architectural Decision Records), gồm: **link, phiên bản/ngày, kết luận rút ra**. Chỉ ghi link đã mở và đọc, kết luận phải khớp nội dung nguồn.

**KHÔNG đưa link/URL vào comment trong code** (kể cả `//`, XML doc, `#pragma`, `SuppressMessage.Justification`). Link dễ chết, lệch phiên bản và làm nhiễu code. Trong code chỉ giữ comment giải thích **vì sao** (lý do kỹ thuật không hiển nhiên), kèm mã Jira/Issue và tên test kiểm chứng nếu cần; phần link nằm ở MR/ADR.

```csharp
// PH-xxxx | Lý do: <hành vi không hiển nhiên cần giải thích>; kiểm chứng bằng <tên test>.
// PH-xxxx | Why: <the non-obvious behavior>; verified by <test name>.
```

Trong phần tổng kết task, AI Agent phải tách bạch:
- **Đã xác minh** (kèm nguồn hoặc test đã chạy).
- **Chưa xác minh / còn giả định** (liệt kê rõ, đề xuất cách kiểm chứng).

---

## 6. Anti-Patterns (Tuyệt Đối Không Làm)

- Gọi API/method/option mà chưa thấy nó tồn tại ở đúng phiên bản thư viện đang dùng.
- Viết "chắc đối tác trả field X" rồi code theo mà không đối chiếu tài liệu hay response thật.
- Sửa lỗi bằng cách thử ngẫu nhiên nhiều thay đổi khi chưa hiểu nguyên nhân gốc.
- Trích dẫn link chưa mở hoặc bịa nguồn. Không mở được thì nói "không truy cập được nguồn".
- Chọn câu trả lời Stack Overflow đầu tiên mà không kiểm tra phiên bản, ngày, và có bằng chứng xác nhận hay không.
- Báo "hoàn thành" trong khi còn giả định chưa xác minh mà không nêu ra.
- Gửi code nội bộ hoặc secret lên diễn đàn/công cụ ngoài để hỏi.
- Dán link/URL vào comment trong code (link thuộc về MR/ADR).

---

## 📋 Checklist Nghiệm Thu Kỹ Thuật Dựa Trên Bằng Chứng (Audit Checklist)

- [ ] Mọi giả định về API/thư viện/hành vi hệ thống đã được kiểm chứng từ code repo, tài liệu chính thức hoặc mã nguồn chưa?
- [ ] Đã loại bỏ hoàn toàn các cụm từ phỏng đoán ("chắc là", "có lẽ", "probably", "should work") trong kết luận và code comment chưa?
- [ ] Không có API/method/option nào bị hallucinate (bịa đặt) ở phiên bản hiện tại chưa?
- [ ] Khi tài liệu chưa rõ, đã kiểm chứng bằng test thực nghiệm (hoặc sandbox đối tác) chưa?
- [ ] Nguồn tra cứu (link issue, PR, docs) đã được ghi nhận trong mô tả MR / ADR thay vì dán link vào code comment chưa?
- [ ] Kết quả task đã tách bạch rõ ràng phần "Đã xác minh" và phần "Chưa xác minh / Cần hỏi người dùng" chưa?
