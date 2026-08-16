# AGENTS.md — Hướng dẫn vận hành cho agent

> **Đọc file này ĐẦU TIÊN** khi vào repo. Đây là operational manual cho agent AI
> phụ trách phát triển/test feature cho một game Godot trong harness này.
> Nếu đây là lần đầu pull về → chạy `bash scripts/init.sh "<Tên project>"` trước.

Bạn làm việc trong một harness **markdown-first**: mọi trạng thái nằm trong file
đã commit. Bạn **KHÔNG được giữ state trong hội thoại** — hết context = mất state.
Nguyên tắc cốt lõi: **model-visible ⟺ logged** (xem §5).

---

## 1. Protocol VÀO session (làm ngay theo thứ tự)

0. **Kiểm tra template hay game repo:** nếu file `TEMPLATE` còn ở gốc repo →
   đây là BẢN TEMPLATE. **KHÔNG dev game / không tạo project.godot tại đây.**
   Việc của bạn (nếu được yêu cầu làm game): copy repo này sang thư mục mới,
   chạy `bash scripts/init.sh "<Tên game>"` bên đó rồi làm tiếp. File `TEMPLATE`
   biến mất sau init = repo game hợp lệ, làm tiếp các bước dưới.
1. Đọc `docs/SESSION.md` → biết `current_focus` + `next_action` + `blockers`.
2. Đọc `docs/ROADMAP.md` → bức tranh lớn (milestone + bảng feature).
3. Chạy `bash scripts/status.sh` → bảng `STATUS / AUTO_TEST / PLAYTEST / BRANCH` mọi feature.
4. Tester → mở thêm `docs/PLAYTEST_QUEUE.md`.
5. Làm theo `next_action`.
   - `handoff_kind: planned-next` → làm tiếp luôn.
   - `handoff_kind: pause` → trình bày `decisions_pending`, **chờ người quyết** trước khi làm tiếp.

## 2. Tạo feature mới

1. Lấy số tiếp theo (xem `docs/features/F-*.md` hoặc ROADMAP). Copy mẫu:
   ```sh
   cp docs/features/_TEMPLATE.md docs/features/F-001.md
   ```
2. Điền frontmatter (schema ở §4): `id`, `name`, `status: planned`, `branch`,
   `priority`, `assigned`, `depends_on`, `auto_test: none`, `playtest{...}`.
3. Tạo nhánh: `git switch -c feat/F-001-<slug>`.
4. Thêm 1 hàng vào `docs/ROADMAP.md` (feature register).
5. Commit.

## 3. Vòng đời feature (state machine)

```
planned → in_dev → dev_done → playtesting → pass → shipped
                                      ↘ fail → in_dev
```

| Chuyển khi nào | Đặt `status:` | Việc kèm theo |
|---|---|---|
| Bắt đầu code | `in_dev` | — |
| Code xong + `auto_test` pass (hoặc không có auto test) | `dev_done` | đẩy vào `docs/PLAYTEST_QUEUE.md` |
| Tester chơi xong, `playtest.result: pass` | `pass` | sẵn sàng merge `main` |
| Tester chơi xong, `playtest.result: fail` | quay `in_dev` | đọc `playtest.notes` rồi sửa |
| Đã merge main | `shipped` | — |

Hai kênh test **độc lập**: `auto_test` (bạn chạy) và `playtest.result` (**tester ghi, KHÔNG phải bạn**).

## 4. Schema frontmatter (source of truth)

**File feature `docs/features/F-xxx.md`:**
```yaml
id: F-xxx
name: <tên ngắn>
status: planned            # planned | in_dev | dev_done | playtesting | pass | fail | shipped
branch: feat/F-xxx-<slug>
priority: P2               # P0 (blocker) | P1 | P2 | P3
assigned: <agent | tên dev>
depends_on: []             # [F-003, ...]
auto_test: none            # none | pass | fail   ← BẠN ghi
playtest:
  assigned: <tester>
  result: pending          # pending | pass | fail ← TESTER ghi
  checklist: []            # ["Player di chuyển WASD", ...]
  notes: ""                # tester ghi lỗi/nhận xét
```

**File `docs/SESSION.md`:**
```yaml
last_updated: "YYYY-MM-DD"
phase: planned             # planned | dev | paused | done
branch: main
handoff_kind: planned-next # planned-next | pause
current_focus: none        # id feature đang làm, hoặc "none"
next_action: "<ĐÚNG 1 lệnh khả thi, resumable>"
done: []                   # evidence: "<việc> (commit/file:...)" — KHÔNG có = chưa làm
blockers: []               # mỗi dòng: "<cái gì> (→ <điều kiện gỡ>)"
decisions_pending: []      # chỉ bắt buộc khi handoff_kind: pause
```

## 5. Kỷ luật BẮT BUỘC

- **model-visible ⟺ logged:** mọi quyết định/trạng thái PHẢI nằm trong file đã commit.
  Nếu bạn "nhớ" điều gì mà không có trong `docs/` → coi như nó không tồn tại.
- **Evidence bắt buộc:** `SESSION.done` phải kèm bằng chứng (commit hash / file).
  Việc không có evidence = coi như chưa làm.
- **3 domain, không trộn:**
  - *Durable* (sống qua reload: status, kết quả test) → frontmatter `docs/features/F-xxx.md`.
  - *In-flight* (đang làm gì, kế tiếp) → frontmatter `docs/SESSION.md`.
  - *Policy* (luật cố định) → `docs/WORKFLOW.md` (KHÔNG bao giờ lẫn vào state).
- **Merge `main` chỉ khi** `auto_test != fail` **VÀ** `playtest.result == pass`.
- **Bạn KHÔNG tự ghi `playtest.result`** thay tester.

## 6. Auto-test (Godot headless)

Khi feature có thể test tự động, chạy headless rồi đặt `auto_test`:

```sh
# Cả project (scene chính):
godot --path . --quit            # khởi + quit để check lỗi khởi tạo

# GUT (nếu cài https://github.com/bitwes/gut):
godot --path . -s res://addons/gut/gut_cmdln.gd -gdir=res://tests -gexit
```

- Pass → `auto_test: pass` + ghi lệnh đã chạy vào §"Cách test tự động" của feature.
- Fail → `auto_test: fail`, xem output, sửa, chạy lại.
- Không có auto test → để `auto_test: none` (vẫn được, playtest là bắt buộc chính).

> **Godot MCP (tuỳ chọn):** nếu client có lắp `@coding-solo/godot-mcp` (xem
> `docs/MCP.md`), bạn có thể gọi `get_project_info` / `run_project` /
> `get_debug_output` để có vòng lặp feedback giàu hơn. Nhưng `auto_test` headless
> vẫn chạy bằng `godot` CLI ở trên — MCP `run_project` chạy debug có cửa sổ,
> không thay thế headless test.

## 7. Protocol KẾT THÚC session (BẮT BUỘC trước khi thoát)

1. Commit mọi thay đổi code (1 commit / bước logic).
2. Cập nhật `docs/features/F-xxx.md`: `status`, `auto_test`, §Lịch sử thay đổi.
3. Cập nhật hàng tương ứng trong `docs/ROADMAP.md`.
4. Feature tới giai đoạn test → đẩy vào `docs/PLAYTEST_QUEUE.md`.
5. Ghi `docs/SESSION.md` (handoff có cấu trúc): `current_focus`, `phase`,
   `next_action`, `handoff_kind`, `done` (**evidence kèm commit/file**),
   `blockers`, và `decisions_pending` nếu `handoff_kind: pause`.
6. Commit các file docs này. **Đừng thoát khi `docs/` còn thay đổi chưa commit.**

## 8. Lệnh hữu ích

| Lệnh | Khi nào |
|---|---|
| `bash scripts/status.sh` | xem trạng thái live mọi feature (đầu session) |
| `bash scripts/init.sh "<Tên>"` | **chỉ 1 lần** khi mới pull harness về (đặt tên project) |
| `git switch -c feat/F-xxx-<slug>` | tạo nhánh feature |
| `godot --path . -e` | mở editor |
| `godot --path . -s <script>` | chạy scene/test headless |

## 9. Đừng

- ❌ Giữ state trong hội thoại thay vì file.
- ❌ Ghi `playtest.result` thay tester.
- ❌ Merge `main` khi feature chưa `playtest.result == pass`.
- ❌ Đổi `status`/`auto_test` mà không commit + cập nhật `ROADMAP` + `SESSION` đi kèm.
- ❌ Sửa `docs/WORKFLOW.md` (policy) để "lách" một feature fail — policy cố định.
