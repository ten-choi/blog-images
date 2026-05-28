# blog-images

`ten-dev-notes` / `ten-language-notes` 블로그에 사용되는 이미지를 모아 두는 저장소.
글 마크다운(blog repo)과 분리해서 이미지만 따로 관리한다. 마크다운에서는
jsdelivr CDN 경유로 호출.

## 폴더 구조

블로그 repo(`personal/blog/blogs/`)와 동일한 트리를 따른다 — 이미지가 어느 글의
것인지 즉시 찾을 수 있게 하기 위함.

```
blog-images/
├── _banners/                  # 포스트 대표 이미지(featured / og:image)
├── dev/
│   ├── algorithms/
│   ├── database/
│   ├── infra/
│   ├── network/
│   └── workflow/
└── language/
    ├── english/
    └── japanese/
```

## 파일명 규칙

```
<날짜접두사>-<글-슬러그>-<용도>.<ext>
```

예:
```
2026-05-dbms-parsing-banner.webp
2026-05-dbms-parsing-flow.png
2026-05-toeic-unit-01-cover.webp
```

- 날짜 접두사(`YYYY-MM`)는 정렬 및 일괄 검색용
- 슬러그는 글 제목과 같이 가져가서 어느 글의 이미지인지 명확
- 용도(`banner`, `cover`, `flow`, `diagram` 등)로 한 글에 여러 장 있을 때 구분
- 확장자는 가능하면 `webp`, 다이어그램은 `svg` 또는 `png`

## CDN 호출 URL

GitHub Raw 직접 호출은 캐싱이 약하므로 **jsdelivr** 사용 권장.

### jsdelivr 패턴
```
https://cdn.jsdelivr.net/gh/<github_user>/blog-images@main/<path>
```

예 (GitHub 사용자명 `ten-choi` 기준):
```
https://cdn.jsdelivr.net/gh/ten-choi/blog-images@main/_banners/2026-05-dbms-parsing-banner.webp
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

- PNG → WebP 변환: `cwebp -q 85 input.png -o output.webp`
- PNG 압축: `pngquant --quality 65-85 input.png`
- 또는 [Squoosh](https://squoosh.app) 사용

목표: 200 KB 이하 / 1장.

## 워크플로

1. 이미지 만든다 (Excalidraw, Figma, Canva, 스크린샷 등)
2. 최적화 후 webp/png로 저장
3. 이 repo의 적절한 폴더에 commit + push
4. 블로그 마크다운에서 jsdelivr URL로 참조

```markdown
![SQL parsing flow](https://cdn.jsdelivr.net/gh/ten-choi/blog-images@main/dev/database/2026-05-dbms-parsing-flow.webp)
```
