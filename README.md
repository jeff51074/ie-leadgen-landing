# Landing Page · 寄书漏斗

## 🖼 换真书封图

Landing 现在用 SVG fallback (`book-cover.svg`). 想换成真书的照片：

1. 准备一张 `.jpg` (推荐 500×700 px, < 200KB, 用真实书的官方封面)
2. 命名为 **`book-cover.jpg`**
3. 放到 `landing/` 目录下 (跟 `index.html` 同级)
4. Commit + push → 页面自动用 JPG, SVG 自动退场

代码已经写好 `onerror` fallback：
```html
<img src="book-cover.jpg" onerror="this.src='book-cover.svg'"/>
```

所以：JPG 在就用 JPG，不在就用 SVG。零配置切换。

---

## 用途

公开 landing page，访客填地址 → 自动加入 lead 数据库 → 寄书 → 进入 nurture 序列。

**主要流量来源：**
1. Meta retargeting ads (Tier 3 - intent layer, 见 docs/08-meta-retargeting-setup.md)
2. Jeff IG/TikTok bio link (持续流量)
3. Email signature
4. QR code at book launch events / speaking gigs

---

## 部署

### Option 1: Railway static site (推荐)
- 新建 Railway service "ie-landing"
- 链接到 GitHub repo
- Build command: 无 (静态文件)
- Start command: `python -m http.server 8080 --directory landing`
- 自定义域名: `book.influenceengine.my`

### Option 2: Vercel / Netlify
- Import 这个 GitHub repo
- Root: `landing/`
- Deploy

### Option 3: 挂在主域 (最简)
- 复制 `landing/index.html` 到 IE 网站 `/book/` 路径
- 访问: `https://influenceengine.my/book/`

---

## URL 设计

| 用途 | URL |
|---|---|
| 主 landing | `book.influenceengine.my` 或 `influenceengine.my/book` |
| Meta retargeting 用 | `book.influenceengine.my/?utm_source=meta&utm_campaign=retargeting` |
| IG bio | `book.influenceengine.my/?utm_source=ig_bio` |
| 邮件签名 | `book.influenceengine.my/?utm_source=email` |
| 演讲 QR code | `book.influenceengine.my/?utm_source=event&utm_campaign=bookfest2026` |

UTM 会自动记录到 `book_gifts.raw_payload`。

---

## 转化漏斗预估

| Step | 假设转化 | Per 1000 visitors |
|---|---|---|
| Landing 访问 | 100% | 1000 |
| Form 完成 | 8-15% | 80-150 |
| 寄书成功 (地址有效) | 95% | 76-143 |
| 收书后约 call | 25-35% | 19-50 |
| 成交 | 10-20% of 约 call | 2-10 |

**预期客单价 RM 80K → 1000 流量 ROI: RM 160K-800K**

---

## 风控

- 月限 30 本（环境变量 `BOOK_MONTHLY_SLOTS` 可调）
- 单 WhatsApp 号 30 天内只能寄 1 次（DB unique check）
- 同 IP 1 小时内 ≤ 3 次提交（待加 rate limiter）
- 海外地址自动拒绝（只寄 MY 境内）

---

## 集成 Meta Pixel

在 `landing/index.html` `<head>` 加：

```html
<!-- Meta Pixel Code -->
<script>
!function(f,b,e,v,n,t,s){...}(window, document,'script',
'https://connect.facebook.net/en_US/fbevents.js');
fbq('init', 'YOUR_PIXEL_ID');
fbq('track', 'PageView');
</script>
```

成功提交后 (`script` 已经写了)：
```js
if (window.fbq) fbq('track', 'Lead');
```

→ 让 Meta 知道哪个 ad 真的带来 lead，优化广告投放算法。
