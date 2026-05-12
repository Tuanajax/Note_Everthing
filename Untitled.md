# 🐍 Cấu Trúc Dự Án Python CLI Tool / Library

## Tổng quan

Tài liệu này mô tả cấu trúc chuẩn cho một dự án Python vừa là **CLI Tool** vừa có thể dùng như **Library** (thư viện tái sử dụng).

---

## 📁 Cây thư mục

```
my-project/
│
├── 📄 README.md                  # Mô tả dự án, hướng dẫn cài đặt & sử dụng
├── 📄 pyproject.toml             # Cấu hình build, dependencies, metadata (PEP 517/518)
├── 📄 setup.cfg                  # Cấu hình bổ sung (tuỳ chọn, thay thế bằng pyproject.toml)
├── 📄 .env.example               # Mẫu biến môi trường
├── 📄 .gitignore                 # File/thư mục bỏ qua khi commit
├── 📄 .pre-commit-config.yaml    # Cấu hình pre-commit hooks (lint, format)
├── 📄 Makefile                   # Các lệnh tắt: make test, make lint, make build
├── 📄 CHANGELOG.md               # Lịch sử thay đổi theo phiên bản
├── 📄 LICENSE                    # Giấy phép sử dụng (MIT, Apache 2.0, ...)
│
├── 📂 src/
│   └── 📂 my_project/            # Package chính (dùng src layout để tránh import nhầm)
│       │
│       ├── 📄 __init__.py        # Export public API của library
│       ├── 📄 __main__.py        # Cho phép chạy: python -m my_project
│       ├── 📄 cli.py             # Entry point CLI (dùng Click hoặc Typer)
│       ├── 📄 config.py          # Quản lý cấu hình & biến môi trường
│       ├── 📄 exceptions.py      # Định nghĩa các custom exception
│       ├── 📄 constants.py       # Hằng số toàn cục
│       │
│       ├── 📂 core/              # Logic nghiệp vụ chính (không phụ thuộc CLI)
│       │   ├── 📄 __init__.py
│       │   ├── 📄 processor.py   # Xử lý dữ liệu cốt lõi
│       │   └── 📄 validator.py   # Kiểm tra & xác thực dữ liệu
│       │
│       ├── 📂 commands/          # Các lệnh CLI (mỗi file = một nhóm lệnh)
│       │   ├── 📄 __init__.py
│       │   ├── 📄 run.py         # Lệnh: my-project run
│       │   ├── 📄 init.py        # Lệnh: my-project init
│       │   └── 📄 export.py      # Lệnh: my-project export
│       │
│       ├── 📂 models/            # Dataclass / Pydantic models
│       │   ├── 📄 __init__.py
│       │   └── 📄 schema.py      # Định nghĩa cấu trúc dữ liệu
│       │
│       ├── 📂 services/          # Tầng dịch vụ (gọi API, DB, file I/O...)
│       │   ├── 📄 __init__.py
│       │   ├── 📄 file_service.py
│       │   └── 📄 api_service.py
│       │
│       └── 📂 utils/             # Tiện ích dùng chung
│           ├── 📄 __init__.py
│           ├── 📄 logger.py      # Cấu hình logging
│           └── 📄 helpers.py     # Hàm tiện ích nhỏ
│
├── 📂 tests/                     # Toàn bộ test
│   ├── 📄 conftest.py            # Fixtures dùng chung (pytest)
│   ├── 📄 __init__.py
│   │
│   ├── 📂 unit/                  # Unit test (test từng hàm/class độc lập)
│   │   ├── 📄 test_processor.py
│   │   └── 📄 test_validator.py
│   │
│   ├── 📂 integration/           # Integration test (test kết hợp nhiều module)
│   │   └── 📄 test_pipeline.py
│   │
│   └── 📂 cli/                   # Test CLI commands (dùng Click's test runner)
│       └── 📄 test_commands.py
│
├── 📂 docs/                      # Tài liệu dự án
│   ├── 📄 index.md               # Trang chủ tài liệu
│   ├── 📄 installation.md        # Hướng dẫn cài đặt
│   ├── 📄 usage.md               # Hướng dẫn sử dụng
│   └── 📄 api-reference.md       # Tài liệu API (library)
│
├── 📂 scripts/                   # Script tiện ích (CI, deploy, seed data...)
│   └── 📄 release.sh
│
└── 📂 .github/                   # GitHub Actions workflows
    └── 📂 workflows/
        ├── 📄 ci.yml             # Chạy test & lint khi push/PR
        └── 📄 publish.yml        # Tự động publish lên PyPI khi release
```

---

## 📦 Các file cấu hình quan trọng

### `pyproject.toml` — Cấu hình trung tâm

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-project"
version = "0.1.0"
description = "Mô tả ngắn gọn về tool / library"
readme = "README.md"
requires-python = ">=3.10"
license = { text = "MIT" }

dependencies = [
    "typer[all]>=0.12",       # CLI framework
    "pydantic>=2.0",          # Data validation & models
    "rich>=13.0",             # Terminal output đẹp
    "python-dotenv>=1.0",     # Load .env files
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov",
    "ruff",                   # Linting + formatting
    "mypy",                   # Type checking
    "pre-commit",
]

[project.scripts]
my-project = "my_project.cli:app"   # Định nghĩa lệnh CLI

[tool.hatch.build.targets.wheel]
packages = ["src/my_project"]
```

### `src/my_project/cli.py` — Entry point CLI

```python
import typer
from my_project.commands import run, init, export

app = typer.Typer(
    name="my-project",
    help="Mô tả CLI tool của bạn",
    pretty_exceptions_enable=False,
)

app.add_typer(run.app, name="run")
app.add_typer(init.app, name="init")
app.add_typer(export.app, name="export")

if __name__ == "__main__":
    app()
```

### `src/my_project/__init__.py` — Public API cho Library

```python
"""my-project: CLI tool & Python library."""

__version__ = "0.1.0"

# Export những gì người dùng library cần
from my_project.core.processor import Processor
from my_project.models.schema import MySchema
from my_project.exceptions import MyProjectError

__all__ = ["Processor", "MySchema", "MyProjectError", "__version__"]
```

---

## 🧩 Phân tầng kiến trúc

```
┌─────────────────────────────────────────┐
│               CLI Layer                 │  ← Giao diện dòng lệnh (Typer/Click)
│         commands/ + cli.py              │     Chuyển input → gọi Services
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│             Service Layer               │  ← Điều phối logic, gọi Core & I/O
│              services/                  │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│              Core Layer                 │  ← Logic nghiệp vụ thuần tuý
│           core/ + models/               │     Không phụ thuộc CLI hay I/O
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│           Infrastructure                │  ← File I/O, HTTP, DB, logging
│            utils/ + config/             │
└─────────────────────────────────────────┘
```

---

## 🚀 Quy trình phát triển

### Cài đặt môi trường

```bash
# 1. Clone dự án
git clone https://github.com/username/my-project.git
cd my-project

# 2. Tạo virtual environment
python -m venv .venv
source .venv/bin/activate        # Linux/macOS
.venv\Scripts\activate           # Windows

# 3. Cài dependencies (bao gồm dev tools)
pip install -e ".[dev]"

# 4. Cài pre-commit hooks
pre-commit install
```

### Các lệnh thường dùng (`Makefile`)

| Lệnh             | Mô tả                         |
| ---------------- | ----------------------------- |
| `make test`      | Chạy toàn bộ test suite       |
| `make lint`      | Kiểm tra code style (ruff)    |
| `make format`    | Tự động format code           |
| `make typecheck` | Kiểm tra kiểu dữ liệu (mypy)  |
| `make build`     | Build package (wheel + sdist) |
| `make publish`   | Publish lên PyPI              |
| `make docs`      | Sinh tài liệu (MkDocs)        |

---

## ✅ Checklist trước khi release

- [ ] Tất cả tests pass (`make test`)
- [ ] Coverage ≥ 80%
- [ ] Không có lỗi type (`make typecheck`)
- [ ] `CHANGELOG.md` đã cập nhật
- [ ] `__version__` trong `__init__.py` đã tăng
- [ ] Tag git version: `git tag v0.2.0`
- [ ] Build & kiểm tra package: `make build`
- [ ] Publish: `make publish`

---

## 📚 Tài liệu tham khảo

- [Typer](https://typer.tiangolo.com/) — CLI framework hiện đại
- [Pydantic v2](https://docs.pydantic.dev/) — Data validation
- [Rich](https://rich.readthedocs.io/) — Terminal output đẹp
- [Ruff](https://docs.astral.sh/ruff/) — Linter & formatter siêu nhanh
- [Hatch](https://hatch.pypa.io/) — Build & project management
- [pytest](https://docs.pytest.org/) — Testing framework