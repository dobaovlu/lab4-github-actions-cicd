# Lab 4 — GitHub Actions CI/CD (Python/FastAPI) — Hướng dẫn thực hiện

Thư mục này là bản sao của starter `python-fastapi/` đã có sẵn file
`.github/workflows/ci-cd.yml` (Task A đã được chuẩn bị sẵn). Việc còn lại
phải làm trực tiếp trên GitHub.com và Render.com (cần tài khoản của bạn nên
mình không tự động hoá được) — làm theo đúng thứ tự dưới đây.

## 0. Đẩy project này lên GitHub

```bash
cd "Lab4-GitHub-Actions-CICD"
git init
git add .
git commit -m "Lab 4: starter app with CI workflow"
```
Tạo một repo mới trên github.com (New repository), rồi:
```bash
git branch -M main
git remote add origin https://github.com/<owner>/<repo>.git
git push -u origin main
```
Sau đó vào **Settings → Branches** bật **Require a pull request before merging**
cho nhánh `main` (bước này cần làm trước, giống Session 4).

## Task A — Workflow đã có sẵn, chỉ cần mở Pull Request

File `.github/workflows/ci-cd.yml` đã paste đúng nội dung handout (lint +
test). Vì workflow nằm ngay trong commit đầu nên để "chứng minh" PR chạy
check, hãy:
1. Tạo nhánh mới `lab4-ci`, sửa nhẹ gì đó (vd thêm comment) hoặc tạo PR từ
   nhánh hiện có.
2. Mở Pull Request trên GitHub → xem tab **Checks**: `CI/CD / check` chạy
   Install → Lint → Test, kết quả `2 passed`.
3. Nhờ 1 thành viên khác **Approve**, rồi **Merge pull request**.

➡ **CHECKPOINT A**: tab Actions có run `CI/CD` xanh, bước Test hiện `2 passed`.

## Task B — Bắt buộc check phải pass (admin làm)

Repo → **Settings → Branches** → Edit rule `main`:
- Tick **Require status checks to pass before merging**.
- Gõ `check` trong ô tìm kiếm, chọn kết quả **GitHub Actions**, thêm vào danh sách.
- **Save changes**.

➡ **CHECKPOINT B**: mở lại Edit rule, `check` đã nằm trong required status checks.

## Task C — Chứng minh gate hoạt động (đỏ rồi xanh)

1. Trên branch mới (`lab4-red`), sửa `tests/test_health.py`, đổi
   `assert r.status_code == 200` → `assert r.status_code == 201`.
2. Commit, mở Pull Request → đợi khoảng 1 phút → check đỏ, nút Merge bị khoá.
   **Chụp màn hình 1**: PR + dấu ✗ đỏ + vùng "Merging is blocked".
3. Sửa lại `201` → `200`, commit tiếp lên cùng nhánh `lab4-red` → check xanh.
   **Chụp màn hình 2**: PR xanh, "All checks have passed".
4. Một thành viên khác Approve → Merge → Delete branch.

➡ **CHECKPOINT C**: 1 URL pull request + 2 ảnh chụp (đỏ/bị khoá, rồi xanh).
**Không sửa workflow để nó xanh** — chỉ sửa code/test.

## Task D — Deploy tự động khi merge vào main

**D1 (chủ Render service):** Render → service → Settings → Deploy Hook →
copy icon. URL dạng `https://api.render.com/deploy/srv-...?key=...`.
Không dán vào chat/file, chỉ dán vào bước D2.

**D2 (admin repo):** GitHub repo → Settings → Secrets and variables →
Actions → New repository secret:
- Name: `RENDER_DEPLOY_HOOK` (đúng chính tả, viết hoa)
- Secret: dán URL vừa copy → Add secret.

**D3 (chủ Render service):** Render → service → Settings → Auto-Deploy →
Edit → **Off** → Save (để chỉ pipeline của bạn mới deploy).

**D4:** File `ci-cd.yml` trong thư mục này **đã có sẵn job `deploy`**
(`needs: check`, `if: github.event_name == 'push'`), không cần sửa gì thêm —
chỉ cần D2 và D3 đã xong.

**D5:** Trên nhánh mới `lab4-deploy`, mở `app/main.py`, đổi
`APP_VERSION = "1.1.0"` → `"1.2.0"` (hoặc version kế tiếp của bạn), commit,
mở Pull Request.
- PR sẽ hiện `CI/CD / check` ✓ và `CI/CD / deploy` — *Skipped* (đúng, vì PR
  không được deploy).

**D6:** Approve → Merge → Delete branch. Mở tab **Actions**, chọn run mới
nhất: đồ thị `check → deploy`, đợi cả hai xanh (~1 phút). Copy URL của run
này — sẽ nộp ở bước 5.

Qua Render → service → **Events**: thấy 1 deploy mới ứng với merge commit.
Đợi 2–4 phút rồi mở `https://<service>.onrender.com/` → xác nhận
`"version": "1.2.0"`. Ghi lại short commit hash (7 ký tự) trong Render
Events — phải khớp với merge commit trên GitHub.

➡ **CHECKPOINT D**: check xanh + deploy xanh + Render Events có deploy mới
cùng commit hash + URL sống hiện đúng version mới.

## Nộp bài (mục 5 handout)

1. URL của Pull Request Task C (kèm 2 ảnh chụp đỏ/xanh).
2. URL của Actions run mà job `deploy` thành công (Task D6).
3. Commit hash đang chạy trên URL Render.

## Lưu ý quan trọng

- **Không bao giờ** paste deploy hook vào file trong repo — kể cả xoá sau
  đó Git vẫn giữ lịch sử → tính là Fail. Nếu lỡ dán: Render → Regenerate
  Hook, rồi cập nhật lại secret (D2).
- Indent YAML dùng 2 dấu cách, không dùng tab.
- Repo private cần GitHub Pro mới bật được branch protection — nếu không
  có, để repo ở chế độ public.

## Self-check trước khi rời phòng lab

- [ ] `ci-cd.yml` chạy Lint và Test trên mọi pull request.
- [ ] `check` là required status check trên `main`.
- [ ] Có 2 ảnh chụp: cùng 1 PR lúc đỏ/bị khoá, rồi lúc xanh.
- [ ] `deploy` bị Skip trên PR, chỉ chạy sau khi merge vào `main`.
- [ ] Hook chỉ nằm trong Settings → Secrets; Render Auto-Deploy đã Off.
- [ ] Version mới đã lên URL sống, đã điền 3 dòng vào form nộp bài.
