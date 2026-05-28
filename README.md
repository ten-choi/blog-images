
이미지 만들었을때 프롬프트
Security 이미지 원본 을 기준으로 security 말고 japanese 를 넣어서 만들어줘. 지금이미지의 큰틀은 바꾸지않을거야. 그런데 전체적인느낌을 조금은 바꿔줘 너무똑같지는않게. 그리고 로봇으로 하지말고 케로로 느낌나는 캐릭터로해줘. 하지만 저작권은 아주살짝 피해갈정도로.



# blog-images

`ten-dev-notes` / `ten-language-notes` 블로그에 사용되는 이미지를 모아 두는 저장소.
글 마크다운(blog repo)과 분리해서 이미지만 따로 관리한다. 마크다운에서는
jsdelivr CDN 경유로 호출.

## 폴더 구조

블로그(`ten-dev-notes` / `ten-language-notes`) 두 곳에 맞춰 **`dev` / `language`
두 폴더만** 둔다. 발행용 WebP는 폴더 바로 아래에, **원본 PNG는 `_src/`** 하위에
보관한다.

```
blog-images/
├── dev/                          # ten-dev-notes 용
│   ├── database_banner.webp      ← 블로그가 호출 (~100 KB)
│   ├── security_banner.webp
│   └── _src/                     ← 원본 (재변환용)
│       ├── database_banner.png   ← 약 1.3 MB
│       └── security_banner.png
└── language/                     # ten-language-notes 용
    ├── english_banner.webp
    ├── japanese_banner.webp
    └── _src/
        ├── english_banner.png
        └── japanese_banner.png
```

### `_src/` 폴더의 역할

- 발행에는 안 쓰임 (jsdelivr URL로 직접 호출하지 않음)
- 미래에 더 좋은 포맷(AVIF 등) 나오거나 품질 재조정이 필요할 때 다시 변환할
  원본 보관용
- **lossy WebP를 또 lossy로 재변환하면 화질이 누적으로 망가지므로** 항상 PNG
  원본에서 변환해야 안전

## 파일명 규칙

기본 패턴:
```
<주제 또는 글-슬러그>_<용도>.<ext>
```

예:
```
database_banner.webp                # 카테고리 배너 (여러 글 공유)
2026-05-dbms-parsing_banner.webp    # 특정 글 전용 배너 (날짜 prefix)
2026-05-dbms-parsing_flow.webp      # 특정 글 본문 다이어그램
```

- 카테고리 단위 재사용 이미지는 짧은 이름(`database_banner.webp`)
- 글 전용은 **날짜 prefix + 글 슬러그 + 용도** (`2026-05-<슬러그>_<용도>`)
- 확장자: 발행본은 `.webp`, 원본은 `.png` (또는 `.svg`)

## CDN 호출 URL

GitHub Raw 직접 호출은 캐싱이 약하므로 **jsdelivr** 사용 권장.

### jsdelivr 패턴
```
https://cdn.jsdelivr.net/gh/<github_user>/blog-images@main/<path>
```

현재 발행본 (ten-choi 기준):
```
https://cdn.jsdelivr.net/gh/ten-choi/blog-images@main/dev/database_banner.webp
https://cdn.jsdelivr.net/gh/ten-choi/blog-images@main/dev/security_banner.webp
https://cdn.jsdelivr.net/gh/ten-choi/blog-images@main/language/english_banner.webp
https://cdn.jsdelivr.net/gh/ten-choi/blog-images@main/language/japanese_banner.webp
```

- `@main` 대신 커밋 해시를 박으면 immutable URL (무한 캐시)
- `@v1.0.0` 같은 git tag도 사용 가능

### Raw 호출 (jsdelivr 안 쓸 때)
```
https://raw.githubusercontent.com/<github_user>/blog-images/main/<path>
```

## 이미지 사이즈 권장

| 용도 | 권장 사이즈 | 비율 | 비고 |
|---|---|---|---|
| 포스트 banner (og:image) | **1600 × 840** | 1.905:1 | Blogger 1200:630 비율과 호환 |
| 본문 다이어그램 | max-width 1200 | 자유 | 큰 원본 → CSS로 100% |
| 썸네일 | 자동 생성됨 | 1:1 | Blogger가 banner에서 crop |

## 이미지 최적화

### 방법 1: Python Pillow (이미 설치돼 있음)
```python
from PIL import Image
img = Image.open("input.png")
img.save("output.webp", "WEBP", quality=85, method=6)
# method=6은 가장 느린/최고 압축
```

### 방법 2: cwebp 명령어
```bash
# 사진/배너 (그라데이션 많음) — q=85
cwebp -q 85 input.png -o output.webp

# 텍스트/UI 스크린샷 — q=95
cwebp -q 95 input.png -o output.webp

# 픽셀 완전 보존 (다이어그램/로고)
cwebp -lossless input.png -o output.webp
```

### 방법 3: 웹에서 [Squoosh](https://squoosh.app)
드래그앤드롭 + 좌우 슬라이더로 시각적 비교 가능. 1~2장 변환할 땐 가장 빠름.

**목표: 200 KB 이하 / 1장.**

## 워크플로

1. 원본 이미지 만든다 (Figma/Canva/스크린샷 등)
2. 1600×840 PNG로 export
3. `<topic>/_src/<name>.png`에 commit
4. WebP 변환 → `<topic>/<name>.webp`에 commit
5. 블로그 마크다운에서 jsdelivr URL로 참조

```markdown
![](https://cdn.jsdelivr.net/gh/ten-choi/blog-images@main/dev/database_banner.webp)
```

## 일괄 변환 스크립트

`_src/` 안의 모든 PNG를 부모 폴더에 WebP로 변환:

```python
# tools/convert_all.py
from pathlib import Path
from PIL import Image

ROOT = Path(__file__).resolve().parent.parent
for src in ROOT.rglob("_src/*.png"):
    dst = src.parent.parent / (src.stem + ".webp")
    Image.open(src).save(dst, "WEBP", quality=85, method=6)
    print(f"{src.relative_to(ROOT)} → {dst.relative_to(ROOT)}")
```
