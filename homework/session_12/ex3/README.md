# Bài 3: Tách biệt luồng CI/CD đa môi trường (Staging & Production) với GitHub Environments

## Mục tiêu

- Quản lý hạ tầng triển khai đa môi trường (Staging vs Production) qua GitHub Environments.
- Cấu hình **Environment Protection Rules** yêu cầu phê duyệt thủ công trước khi deploy lên Production.

---

## Cấu trúc file

```
.github/
└── workflows/
    └── cd-multi-env.yml   ← File workflow chính
homework/
└── session_12/
    └── ex3/
        ├── cd-multi-env.yml   ← Bản nộp bài
        └── README.md          ← Tài liệu này
```

---

## Bước 1: Tạo GitHub Environments

Truy cập **GitHub Repository → Settings → Environments**, tạo 2 môi trường:

### 1.1 Tạo môi trường `staging`

1. Nhấn **New environment** → đặt tên `staging` → **Configure environment**
2. Không cần bật Protection Rules cho staging
3. Nhấn **Save protection rules**

### 1.2 Tạo môi trường `production`

1. Nhấn **New environment** → đặt tên `production` → **Configure environment**
2. Bật **Required reviewers** → thêm tài khoản cá nhân làm người duyệt
3. Nhấn **Save protection rules**

> ⚠️ Khi `production` có Required reviewers, mỗi lần workflow chạy đến job dùng environment này, GitHub sẽ **dừng lại và chờ phê duyệt** trước khi tiếp tục.

---

## Bước 2: Workflow `cd-multi-env.yml`

### Cơ chế hoạt động

| Branch    | Job được chạy    | Environment  | Cần duyệt? |
|-----------|-----------------|--------------|------------|
| `develop` | `deploy-staging` | staging      | ❌ Không   |
| `main`    | `deploy-prod`    | production   | ✅ Có      |

### Giải thích cấu hình chính

```yaml
# Kích hoạt khi push vào main hoặc develop
on:
  push:
    branches:
      - main
      - develop

jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'   # Chỉ chạy khi push vào develop
    environment:
      name: staging
      url: https://api.staging.example.com

  deploy-prod:
    if: github.ref == 'refs/heads/main'      # Chỉ chạy khi push vào main
    environment:
      name: production                        # ← Trigger protection rule
      url: https://api.prod.example.com
```

Từ khóa `environment: name: production` là chìa khóa kích hoạt **Required reviewers** đã cấu hình ở Bước 1.

---

## Bước 3: Kiểm tra luồng hoạt động

### Push vào `develop` → Staging

```bash
git checkout develop
git commit -m "feat: update feature X"
git push origin develop
```

- Job `deploy-staging` chạy ngay, không cần duyệt.
- Job `deploy-prod` bị bỏ qua (`if` condition = false).

### Push vào `main` → Production (có phê duyệt)

```bash
git checkout main
git merge develop
git push origin main
```

1. Job `deploy-prod` bắt đầu → GitHub dừng lại, hiển thị **"Waiting for review"**
2. Reviewer nhận email/notification
3. Vào tab **Actions** → chọn workflow run → nhấn **Review deployments**
4. Chọn environment `production` → nhấn **Approve and deploy**
5. Job tiếp tục chạy và hoàn thành ✅

---

## Sản phẩm nộp bài

- [x] File `cd-multi-env.yml` — kịch bản workflow
- [ ] Ảnh chụp màn hình cửa sổ **"Waiting for review"** (chụp sau khi push lên GitHub)
- [ ] Ảnh chụp màn hình sau khi **Approve and deploy** thành công (job tích xanh)

---

## Tham khảo

- [GitHub Docs - Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [GitHub Docs - Reviewing deployments](https://docs.github.com/en/actions/managing-workflow-runs/reviewing-deployments)
