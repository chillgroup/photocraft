# Đồng bộ Sepia với upstream PhotoCraft

**Ngày:** 2026-11-04  
**Áp dụng cho:** fork `chillgroup/photocraft` (Sepia)  
**Spec liên quan:** [Sepia desktop kết nối GODSystem và Podlovers](2026-11-04-photocraft-desktop-integration-design.md) §12

Tài liệu này mô tả cách nhận code cộng đồng từ upstream `storytold/photocraft` mà vẫn giữ nguyên phần Sepia tự phát triển.

## 1. Bối cảnh

| Remote     | URL                                       | Vai trò                                      |
| ---------- | ----------------------------------------- | -------------------------------------------- |
| `origin`   | `https://github.com/chillgroup/photocraft.git` | Fork Sepia; nhánh `main` là nhánh phát triển. |
| `upstream` | `https://github.com/storytold/photocraft.git`  | Repo gốc; chỉ fetch, push đã tắt (`DISABLE`). |

Upstream thay đổi rất nhanh. Số liệu đo ngày 2026-11-04:

- khoảng 970 commit/tuần, 8.685 commit/tháng;
- số commit trong tháng sửa các file Sepia có thể chạm tới:

| File                             | Commit/tháng |
| -------------------------------- | -----------: |
| `crates/ui-egui/src/menus.rs`    |          167 |
| `crates/ui-egui/src/state.rs`    |          145 |
| `crates/io/src/psd_export.rs`    |           97 |
| `crates/io/src/psd_import.rs`    |           82 |
| `crates/doc/src/lib.rs`          |           36 |
| `crates/format/src/convert.rs`   |           28 |
| `AGENTS.md`                      |           26 |
| `crates/engine/src/cmd.rs`       |           17 |
| `crates/format/src/manifest.rs`  |           16 |

Hệ quả: chi phí đồng bộ phụ thuộc chủ yếu vào việc Sepia sửa bao nhiêu dòng trong file upstream, không phải vào lệnh git.

## 2. Nguyên tắc đặt code

### 2.1. Code mới nằm ngoài file upstream

Code của Sepia đặt trong file và crate mới mà upstream không có:

| Thành phần                 | Vị trí                                         |
| -------------------------- | ---------------------------------------------- |
| Command `podlovers.*`      | `crates/engine/src/cmd/podlovers_cmds.rs`      |
| Panel Podlovers            | crate mới `crates/sepia-panel` (L6)            |
| Codec XMP `podl`, UID rule | module mới trong `crates/doc` hoặc crate `sepia-meta` ở layer phù hợp |
| Connector                  | package `sepia-connector`, process riêng       |
| Branding Sepia             | một module cấu hình duy nhất (mục 2.4)         |

Crate mới phải đăng ký trong `xtask/src/layers.rs` và có `[lints] workspace = true`.

### 2.2. File upstream chỉ có điểm nối một dòng

Khi buộc phải sửa file upstream, chỉ thêm một dòng gọi vào code Sepia. Ví dụ đăng ký command trong `crates/engine/src/cmd.rs`:

```rust
v.extend(crate::podlovers_cmds::specs()); // sepia: podlovers commands
```

Không viết logic nghiệp vụ trực tiếp trong file upstream. Một conflict một dòng giải trong vài giây; một khối logic chèn giữa file upstream phải đọc lại mỗi lần merge.

### 2.3. Tránh file đổi nhiều nhất

`menus.rs` và `state.rs` đổi hơn 140 lần mỗi tháng. Panel giữ struct state riêng (serde) trong crate của mình; `state.rs` chỉ thêm một field trỏ tới struct đó. Menu item của Sepia đăng ký qua command id, không sửa danh sách menu của upstream nếu có cách khác.

### 2.4. Branding qua cấu hình build

Tên "Sepia", icon, app id, binary name và thư mục config lấy từ một chỗ duy nhất, bật bằng cargo feature `sepia` (hoặc biến môi trường lúc build). Không sửa chuỗi "PhotoCraft" rải rác trong file upstream. Quy tắc branding đầy đủ ở §12.1 của spec.

### 2.5. Gắn nhãn mọi chỗ sửa

Mỗi dòng hoặc khối Sepia sửa trong file upstream có comment `sepia:` kèm lý do ngắn:

```rust
// sepia: assign fresh podl UIDs on duplicate (spec §5.3)
```

Liệt kê toàn bộ điểm chạm:

```sh
rg -n "sepia:" crates apps xtask
```

Danh sách này là thứ cần kiểm sau mỗi lần merge. Một chỗ sửa upstream không có nhãn được coi là lỗi review.

### 2.6. Quy tắc agent riêng

Quy tắc của fork nằm trong `AGENTS.sepia.md`. `AGENTS.md` của upstream chỉ thêm một dòng ở đầu file trỏ tới `AGENTS.sepia.md`, vì `AGENTS.md` cũng đổi thường xuyên.

## 3. Quy trình đồng bộ

### 3.1. Thiết lập một lần

```sh
git remote add upstream https://github.com/storytold/photocraft.git   # nếu chưa có
git remote set-url --push upstream DISABLE
git config rerere.enabled true
```

`rerere` ghi lại cách giải từng conflict và tự áp lại khi gặp conflict giống hệt ở lần merge sau.

### 3.2. Mỗi lần đồng bộ

1. Lấy code mới từ upstream:
   ```sh
   git fetch upstream
   ```
2. Chọn một commit upstream cụ thể đã qua CI của upstream. Không merge thẳng `upstream/main` đang trôi:
   ```sh
   git log --oneline -5 upstream/main
   ```
3. Tạo nhánh sync từ `main` của Sepia:
   ```sh
   git switch -c sync/upstream-<YYYY-MM-DD> main
   ```
4. Merge commit đã chọn:
   ```sh
   git merge --no-ff <upstream-commit> -m "sync: upstream PhotoCraft <short-sha>"
   ```
5. Giải conflict. Ưu tiên giữ code upstream rồi nối lại điểm nối `sepia:`; không sửa code upstream cho vừa với Sepia nếu có thể chuyển phần đó sang code Sepia.
6. Kiểm lại toàn bộ điểm chạm:
   ```sh
   rg -n "sepia:" crates apps xtask
   ```
7. Chạy gate ở mục 4.
8. Cập nhật commit pin (mục 3.4), mở PR vào `main`, review, merge.

### 3.3. Merge, không rebase

`main` của Sepia có nhiều người cùng làm. Rebase viết lại lịch sử và cần force push lên nhánh chung. Merge giữ lịch sử và cho biết rõ Sepia đã nhận upstream tới commit nào. Không force push `main`.

### 3.4. Nhịp và pin

- Đồng bộ mỗi 1–2 tuần. Để lâu hơn, conflict dồn lại và khó giải.
- Mỗi lần nhận upstream, cập nhật commit pin ở ba chỗ: About của Sepia, §2 của spec và mô tả PR sync.
- Trước release pilot, đóng băng đồng bộ; chỉ cherry-pick bản sửa lỗi cần thiết từ upstream.

## 4. Gate sau mỗi lần merge

Theo §12.2 của spec:

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo xtask layers
cargo xtask wasm
cargo test -p photocraft-engine --test panic_hunt -- --ignored
cargo xtask test-corpus
```

Thêm test riêng của Sepia:

- capability tests của connector (§6.3 của spec);
- PC-03, PC-04, PC-06 (binding, UID, undo qua save/duplicate);
- round trip XMP `podl` trong Sepia; PC-21 (qua Photoshop) chạy thủ công trước release.

`photocraft-format` cố ý không compile khi struct trong `photocraft-doc` có thêm field. Thay đổi upstream đụng tới document model vì vậy lộ ra ngay khi build.

### 4.1. CI

Workflow upstream gọi `storytold/craft-common-actions@main`. Fork pin workflow đó theo commit, hoặc dùng CI riêng, để CI của Sepia không đổi theo upstream ngoài ý muốn. Không nhận các thay đổi `.github/workflows/` của upstream mà chưa review.

## 5. Đóng góp ngược lên upstream

Phần không mang tính nghiệp vụ nên gửi PR lên upstream để thu nhỏ diff của fork:

- hook cấp ID mới khi duplicate/copy cho metadata của layer;
- command đọc/ghi layer XMP theo namespace;
- hook đăng ký panel bên ngoài cho `ui-egui`.

Khi upstream nhận, xoá phần tương ứng trong fork ở lần sync sau. Phần nghiệp vụ Podlovers, connector và branding Sepia giữ trong fork.

PR gửi upstream theo `AGENTS.md` và `docs/contributing.md` của upstream, không chứa tên Sepia, dữ liệu Podlovers hay thông tin nội bộ.

## 6. Khi conflict lớn

Nếu một lần merge phát sinh conflict lớn ở vùng Sepia chạm tới (ví dụ upstream tách lại `psd_export.rs`):

1. Huỷ merge (`git merge --abort`), giữ `main` nguyên vẹn.
2. Đọc lịch sử upstream của vùng đó để hiểu cấu trúc mới.
3. Chuyển điểm nối Sepia sang cấu trúc mới trong một PR riêng, rồi merge lại.

Không giải conflict lớn bằng cách giữ toàn bộ phiên bản Sepia của file upstream; làm vậy là âm thầm bỏ thay đổi cộng đồng.
