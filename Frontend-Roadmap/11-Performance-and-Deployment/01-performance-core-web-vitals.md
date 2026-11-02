# عملکرد وب و Core Web Vitals

عملکرد = سرعت واقعی + **سرعت احساس‌شده**. روی تبدیل، SEO و رضایت کاربر اثر مستقیم دارد.

## Core Web Vitals
| شاخص | معنی | خوب |
|---|---|---|
| **LCP** (Largest Contentful Paint) | زمان نمایش بزرگ‌ترین محتوای صفحه | ≤ ۲.۵ ثانیه |
| **INP** (Interaction to Next Paint) | تأخیر پاسخ به تعاملات (جایگزین FID) | ≤ ۲۰۰ میلی‌ثانیه |
| **CLS** (Cumulative Layout Shift) | پرش‌های غیرمنتظره‌ی چیدمان | ≤ ۰.۱ |

شاخص‌های کمکی: **TTFB** (زمان اولین بایت)، **FCP**، **TBT** (در آزمایشگاه).

## اندازه‌گیری
- **Lab** (شبیه‌سازی): Lighthouse، DevTools Performance، WebPageTest
- **Field** (کاربران واقعی): CrUX / PageSpeed Insights، کتابخانه‌ی `web-vitals`
```js
import { onLCP, onINP, onCLS } from "web-vitals";
onLCP(console.log); onINP(console.log); onCLS(console.log);
```
- **Performance budget**: سقف حجم/زمان در CI (Lighthouse CI).

## بهبود LCP
- سرور سریع، CDN، کش، فشرده‌سازی (Brotli/gzip)، HTTP/2-3
- تصویر hero: فرمت **AVIF/WebP**، اندازه‌ی مناسب، `fetchpriority="high"`، **بدون lazy**
- preload منابع بحرانی: `<link rel="preload" as="image" href="hero.avif">`
- CSS بحرانی inline و بقیه async ([[../../CSS/03-Advanced/05-performance-and-modern-features|عملکرد CSS]])
- کاهش JS مسدودکننده (`defer`)
- SSR/SSG برای محتوا ([[../05-Frameworks/06-meta-frameworks-ssr-ssg|SSR]])

## بهبود INP
- شکستن Taskهای طولانی (> ۵۰ms)
- کاهش کار روی thread اصلی: Web Worker، `requestIdleCallback`
- debounce/throttle رویدادهای پرتکرار ([[../../JavaScript/04-Advanced/03-functional-patterns|الگوهای تابعی]])
- جلوگیری از رندر مجدد غیرضروری (memo، کلیدهای درست، virtualization لیست‌های بلند)
- `scheduler.yield()` / `setTimeout(0)` برای بازگشت کنترل به مرورگر
```js
for (const chunk of chunks) {
  process(chunk);
  await new Promise(r => setTimeout(r));   // فرصت دادن به UI
}
```

## بهبود CLS
```html
<img src="a.jpg" width="800" height="600" alt="" />    <!-- ابعاد = نسبت ثابت -->
```
```css
.embed { aspect-ratio: 16 / 9; }
.ad { min-height: 250px; }
```
- فضای تبلیغ و iframe را رزرو کن
- فونت: `font-display: swap` + `size-adjust`
- انیمیشن فقط با `transform/opacity`
- محتوا را بالای محتوای موجود وارد نکن.

## تصاویر
```html
<picture>
  <source type="image/avif" srcset="a.avif">
  <source type="image/webp" srcset="a.webp">
  <img src="a.jpg" alt="…" width="800" height="600" loading="lazy" decoding="async"
       srcset="a-400.jpg 400w, a-800.jpg 800w" sizes="(max-width: 600px) 100vw, 800px">
</picture>
```
SVG برای آیکن، CDN تصویر (Cloudinary/Imgix) برای تغییر اندازه ([[../../HTML/03-Advanced/02-seo-meta-performance|HTML performance]]).

## JavaScript
- **Code splitting** و lazy load مسیرها/کامپوننت‌ها ([[../03-Package-Managers-and-Tooling/02-vite-bundlers-transpilers|Vite]])
- Tree shaking، حذف وابستگی‌های سنگین (moment → date-fns / Intl)
- بررسی bundle: `rollup-plugin-visualizer`, `webpack-bundle-analyzer`
- `defer`/`async`, `type="module"`
- شخص ثالث (آنالیتیکس، چت): تأخیر در لود (`requestIdleCallback`)، Partytown

## شبکه و کش
```http
# فایل با hash در نام (app.3f9a.js)
Cache-Control: public, max-age=31536000, immutable
# HTML
Cache-Control: no-cache
```
- `ETag` / `Last-Modified` + 304
- `<link rel="preconnect" href="https://api.example.com">`، `dns-prefetch`
- `rel="prefetch"` برای صفحه‌ی بعدی، `<link rel="modulepreload">`
- Service Worker برای کش آفلاین ([[../07-Web-APIs-and-PWA/02-pwa-service-workers|PWA]])
- CDN و compress Brotli

## فونت
`woff2`، subset (فقط حروف فارسی/لاتین لازم)، حداکثر ۲ خانواده، preload فونت بحرانی، فونت سیستم برای عملکرد حداکثر.

## رندر و حافظه
- `content-visibility: auto`
- لیست‌های بلند ← **virtualization** (`@tanstack/virtual`, `react-window`)
- پاک‌سازی listenerها و timerها (نشت حافظه) ([[../../JavaScript/04-Advanced/06-performance-and-memory|عملکرد JS]])
- DevTools ← Memory ← heap snapshot

## چک‌لیست سریع
- [ ] Lighthouse ≥ ۹۰ روی موبایل
- [ ] تصاویر بهینه با ابعاد و lazy
- [ ] JS بدون استفاده حذف و split شده
- [ ] cache طولانی برای فایل‌های hash‌دار
- [ ] فونت بهینه
- [ ] بدون layout shift
- [ ] third-party کنترل‌شده

## تمرین
۱. یک سایت را با Lighthouse بسنج و سه مورد بهبود اجرا کن.
۲. bundle پروژه‌ات را تحلیل و بزرگ‌ترین وابستگی را کوچک کن.
۳. CLS را با افزودن width/height به تصاویر کاهش بده و تفاوت را اندازه بگیر.
