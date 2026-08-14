# godot-agent-harness

> Harness (bộ khung quy trình) cho phát triển game **Godot 4** bằng
> **agent AI + dev + tester tay**. Mọi trạng thái nằm trong Markdown đã commit
> → bất kỳ agent / dev / tester nào cũng resume được, không phụ thuộc session AI.

> **Agent AI — đọc trước:** [`AGENTS.md`](AGENTS.md) (protocol vào/kết thúc session, schema frontmatter, kỷ luật).

Tạo ra từ quá trình mượn pattern của các agent harness production-grade:
**model-visible ⟺ logged** (state luôn nằm trong file, không trong hội thoại),
**handoff có cấu trúc**, và **phân tách durable / in-flight / policy**.

---

## Dùng cho project Godot mới

**Cách 1 — GitHub "Use this template"** (bật *Template repository* trong Settings
của repo này): bấm nút → đặt tên repo game mới → clone về máy.

**Cách 2 — clone thủ công:**
```sh
git clone --depth 1 https://github.com/hieubqdsm/godot-agent-harness.git ten-game
cd ten-game
rm -rf .git
git init
```
_(đổi URL theo repo của bạn)_

**Đặt tên project** (chạy 1 lần; `init.sh` tự thay tên + ngày trong mọi file docs):
```sh
bash scripts/init.sh "Tên Game Của Bạn"
```

**Khởi động dev:**
1. `git add -A && git commit -m "init project"`
2. Tạo `project.godot`: mở Godot → New Project tại thư mục này (hoặc `godot --path . -e`).
3. Đọc `docs/SESSION.md` → làm theo `next_action`.

> Xong `init.sh` có thể xoá đi nếu muốn (chỉ dùng 1 lần).

## Bạn nhận được gì

| File | Vai trò |
|---|---|
| `docs/WORKFLOW.md` | Quy trình đầy đủ: vai trò, state machine, protocol đầu/cuối session, kỷ luật state |
| `docs/ROADMAP.md` | Plan tổng thể: milestone + bảng feature |
| `docs/SESSION.md` | **Entry point resume** — việc hiện tại + kế tiếp (handoff có cấu trúc) |
| `docs/PLAYTEST_QUEUE.md` | Worklist hằng ngày của tester |
| `docs/features/F-xxx.md` | 1 feature / file. Frontmatter = nguồn sự thật |
| `docs/features/_TEMPLATE.md` | Mẫu tạo feature mới (copy → `F-00X.md`) |
| `scripts/status.sh` | Bảng trạng thái live từ frontmatter mọi feature |
| `scripts/init.sh` | Đặt tên project (chạy 1 lần) |
| `.gitignore` | Godot 4 ignores |

## Workflow tóm tắt

Mỗi feature đi qua:
```
planned → in_dev → dev_done → playtesting → pass → shipped
                                              ↘ fail → in_dev
```
Hai kênh test **độc lập**: `auto_test` (agent chạy headless, vd GUT) và
`playtest` (tester tay). Feature chỉ merge vào `main` khi
`auto_test != fail` **VÀ** `playtest.result == pass`.

Mở đầu session nào cũng đọc theo thứ tự:
`docs/SESSION.md` → `docs/ROADMAP.md` → `bash scripts/status.sh`.

Chi tiết: `docs/WORKFLOW.md`.

## Kỷ luật state (3 loại, không trộn)

| Loại | Đặt ở đâu |
|---|---|
| **Durable** (fact sống qua reload: status, kết quả test) | frontmatter `docs/features/F-xxx.md` |
| **In-flight** (đang làm gì, kế tiếp) | frontmatter `docs/SESSION.md` |
| **Policy** (luật cố định: điều kiện merge) | `docs/WORKFLOW.md` |

## Yêu cầu
- Godot 4.x
- Bash (Git Bash trên Windows đi kèm Git for Windows)
- (tuỳ chọn) [GUT](https://github.com/bitwes/gut) để auto-test headless

## Giấy phép
MIT — xem `LICENSE`.
