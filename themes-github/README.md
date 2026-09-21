# stripe-listener themes

Repo được extension `stripe-listener` fetch qua jsDelivr (fallback
raw.githubusercontent.com) mỗi giờ 1 lần.

**Cả theme visual đều nằm trên đây** — CSS + HTML của scene, extension chỉ
cầm engine (host viewport, timing, particle randomizer). **Thêm/sửa theme
không cần rebuild extension**, chỉ push file lên đây.

## Layout

```
sl/
├── theme.json                    ← trỏ theme active + version toàn cục
├── mid-autumn/                   ← bundled fallback (offline)
│   ├── scene.json
│   ├── scene.css
│   └── scene.html
└── primary/                      ← default live theme
    ├── scene.json                ← layers[] = SVG pool (random pick)
    ├── scene.css
    ├── scene.html                ← .slma-hero + .slma-stars[data-kind=sparkle]
    └── *.svg                     ← fetched only if listed in layers
```

## Cách extension resolve theme

1. Background service-worker chạy `chrome.alarms` mỗi 60 phút.
2. Fetch `sl/theme.json` qua jsDelivr:
   `https://cdn.jsdelivr.net/gh/<user>/<repo>@main/sl/theme.json`
3. Đọc `active` (ví dụ `"primary"`) rồi fetch tiếp:
   - `sl/<active>/scene.json`
   - `sl/<active>/scene.css`
   - `sl/<active>/scene.html`
4. Sanitize toàn bộ, cache vào `chrome.storage.local`.
5. Content script preload cache ngay khi inject → payment success render 0 lag.

Fallback jsDelivr → raw.githubusercontent.com nếu jsDelivr 5xx.

## Thêm theme mới (ví dụ Giáng Sinh)

Không đụng vào extension:

1. Tạo folder mới:
   ```
   sl/christmas/scene.json
   sl/christmas/scene.css
   sl/christmas/scene.html
   ```
2. `scene.json`:
   ```json
   { "id": "christmas", "version": 1, "duration": 5200 }
   ```
3. `scene.html`: bọc centerpiece trong `.slma-stage` để engine handle
   enter/exit. Optional:
   - `<div class="slma-hero"></div>` — engine pick random 1 SVG từ `layers[]`
   - `<div class="slma-stars"></div>` — hạt rơi (mặc định). Thêm
     `data-kind="sparkle"` để hạt nhỏ, nổi lên.
4. `scene.css`: scope dưới `.slma-*`. Particles dùng `--slma-star-op`,
   `--slma-drift`, `--slma-rot`; keyframe `slma-fall` phải đọc các var này.
5. Đổi `sl/theme.json`:
   ```json
   { "active": "christmas", "version": 2 }
   ```
   Bump version để invalidate cache mọi client.
6. Commit + push. jsDelivr auto-purge sau ~5 phút; force ngay tại
   https://www.jsdelivr.com/tools/purge

## Sanitize rules (đọc kỹ trước khi viết scene mới)

Extension chạy sanitizer trên mọi file fetch về. Nếu file có nội dung không
hợp lệ, bị strip hoặc load bundled fallback thay thế.

**scene.json:**
- `id`: `[a-z0-9][a-z0-9_-]{0,63}` (khớp `<active>` trong theme.json)
- `duration`: 100 – 15000 ms
- `version`: số nguyên

**scene.html:**
- Bị **strip toàn bộ** các tag: `<script>`, `<iframe>`, `<object>`, `<embed>`,
  `<link>`, `<meta>`, `<base>`, `<form>`, `<input>`, `<button>`, `<audio>`,
  `<video>`, `<source>`, `<applet>`, ... — chỉ dùng `<div>`, `<span>`, `<i>`,
  `<b>` v.v.
- Bị strip: `on*` attribute, `javascript:` / `vbscript:` / `data:text/html`
  trong `href`/`src`/`action`.
- Max 96KB.

**scene.css:**
- Bị strip: `@import`, `@charset`, `expression()`, `behavior:`,
  `-moz-binding:`, `javascript:`, `vbscript:`, `url(...)` trỏ ngoài
  (chỉ cho `url(data:...)` hoặc `url(#fragment)`).
- Max 96KB.

SVG lớn (>96KB) để trong folder theme + `layers[]`, không inline vào
`scene.html`. Tên file phải khớp `[a-z0-9._-]+.svg` (không dấu cách).
Mỗi file ≤ 300KB — `Stripe Scratch.svg` (514KB) bị bỏ.

## Bundled fallback

`src/theme/bundled/mid-autumn.ts` trong extension chứa bản copy inline của
mid-autumn để offline / fresh install vẫn có scene chạy. Cập nhật CDN đồng
thời với bundled thì rebuild extension; nếu chỉ update CDN thì offline user
vẫn thấy bản cũ tới khi có mạng.
