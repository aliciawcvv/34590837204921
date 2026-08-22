# stripe-listener themes

Nội dung để commit lên GitHub repo `reborninapril/3284957638945` (hoặc bất kỳ
repo nào bạn cấu hình trong `src/theme/constants.ts`).

## Layout

```
sl/
├── theme.json                    ← điều phối: theme nào đang active + version
└── mid-autumn/
    ├── scene.json                ← manifest chi tiết
    ├── moon.svg
    ├── hangnga.svg
    ├── tho.svg
    ├── kylan.svg
    ├── banh.svg
    ├── caycua.svg
    ├── lantern-red.svg
    ├── lantern-yellow.svg
    ├── lantern-orange.svg
    └── lantern-cyan.svg
```

Copy nguyên thư mục `sl/` này lên root của repo GitHub.

## Cách extension load theme

1. Background service-worker chạy `chrome.alarms` mỗi 60 phút.
2. Fetch `sl/theme.json` qua jsDelivr:
   `https://cdn.jsdelivr.net/gh/<user>/<repo>@main/sl/theme.json`
3. Nếu `version` trong `theme.json` khác với cache local → fetch tiếp
   `sl/<active>/scene.json` + toàn bộ asset SVG được reference.
4. Sanitize + cache vào `chrome.storage.local`.
5. Content script preload cache ngay khi inject → khi payment thành công,
   scene render với latency 0.

Fallback tự động sang `raw.githubusercontent.com` nếu jsDelivr trả 5xx.

## Đổi theme

- **Đổi visual của theme hiện tại:** sửa SVG hoặc `scene.json` trong
  `sl/mid-autumn/` → bump `version` trong `sl/theme.json` → commit + push.
- **Đổi hẳn theme khác (ví dụ Giáng Sinh):** tạo folder mới `sl/giangsinh/`
  với `scene.json` + SVG, đổi `active` trong `sl/theme.json` thành
  `"giangsinh"`, bump `version`, commit + push.
- **Force refresh ngay:** vào popup extension → Advanced → **Refresh theme cache**.

jsDelivr auto-purge sau ~5 phút khi git push, không cần thao tác thêm.

## Sanitize rules (đọc kỹ trước khi tự viết `scene.json`)

Extension chỉ chấp nhận field trong whitelist. Nếu bạn thêm field ngoài
schema (định nghĩa ở `src/theme/types.ts`), field đó sẽ bị bỏ qua.

**Giới hạn:**
- `layers.length ≤ 24`
- `particles[].count ≤ 240`
- `stamps.length ≤ 2`
- `duration ≤ 15000` ms
- Mỗi asset SVG ≤ 300KB
- Chỉ chấp nhận easing name trong: `linear`, `easeInSine`, `easeOutSine`,
  `easeInOutSine`, `easeInQuad`, `easeOutQuad`, `easeInOutQuad`,
  `easeInCubic`, `easeOutCubic`, `easeInOutCubic`, `easeOutExpo`,
  `easeInOutExpo`, `easeOutBack`, `easeOutElastic`, `easeOutBounce`
- Position units: `px`, `%`, `vw`, `vh`, `em`, `rem`, hoặc `clamp(a, b, c)`
- SVG bị strip: `<script>`, `on*` handler, `<foreignObject>`, remote `href`

**Path motion tokens:**
- `from` / `to`: `"top-left"`, `"top-right"`, `"bottom-left"`, `"bottom-right"`,
  `"off-screen-left/right/top/bottom"`, hoặc `"<layerId>"` (bay tới vị trí của
  layer khác).
- `type`: `"linear"`, `"sine"`, `"hop"`, `"zigzag"`.

## Bundled fallback

`src/theme/bundled/mid-autumn.ts` chứa bản copy inline của theme này, dùng khi
extension chưa fetch được CDN (offline / fresh install). Khi cập nhật SVG
trên GitHub, nếu muốn bản offline cũng đổi thì phải rebuild extension.
