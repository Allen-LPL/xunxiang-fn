# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The **uni-app mobile client** for ShopXO, an open-source e-commerce system. One Vue 2 codebase compiles to WeChat / QQ / Baidu / Alipay / Toutiao(Douyin) / Kuaishou mini-programs **+ H5 + native App**. The PHP backend (separate ShopXO project) serves all data; this repo is frontend only.

## Build / run / develop

There is **no npm/CLI build** for this project. It is developed and packaged exclusively with **HBuilderX** (DCloud's IDE):

- Run: HBuilderX toolbar → `运行 (Run) → 运行到小程序模拟器` → pick a platform (e.g. WeChat devtools), or `运行到浏览器` for H5.
- Build/release: HBuilderX toolbar → `发行 (Release)` → pick platform.
- Packaging guide: http://uniapp.shopxo.net/

The root `package.json` / `node_modules` (only `hls.js` + `jweixin-module`) belong to the bundled video-player component, **not** the app's build — do not treat this as a Node project.

### Connecting to a backend (required before anything works)

Edit `App.vue` → `globalData.data`:
- `request_url` — ShopXO backend API base URL (e.g. `https://your-shop.com/`).
- `static_url` — static asset base URL (append `public/` if the site root isn't the `public` dir).
- `default_theme` — one of `red yellow black green orange blue brown purple` (default `red`).

### Formatting

Prettier is configured (`.prettierrc.cjs`): **4-space indent, single quotes, semicolons, `printWidth: 100000` (lines effectively never wrap), `vueIndentScriptAndStyle: true`**. Match this — long single-line statements/templates are intentional, not accidental.

## Architecture

### `App.vue` is the application core (~150KB — read it before changing global behavior)

`globalData.data` is the global config object (URLs, theme, feature flags, cache-key names, tabbar list...). `globalData` also holds **dozens of shared helper methods** used everywhere via `getApp().globalData.xxx(...)`. Key ones:

- `get_request_url(action, controller, plugins, params, group)` — builds the API URL. Produces `{request_url}{group||'api'}.php?s={controller}/{action}&ajax=ajax&...`. When `plugins` is set, the call is routed through the `plugins` controller (`&pluginsname=...&pluginscontrol=...&pluginsaction=...`).
- `request_params_handle(url)` — auto-appends `token`, `uuid`, client type/brand, language, theme, location, referrer to every request URL.
- `get_user_info(obj, method, params)` / `get_user_cache_info()` — user/auth state; triggers platform-specific login (`// #ifdef MP/APP/H5`) when no user, then callbacks into `obj[method](params)`.
- `get_config(key, default)`, `get_theme_color()`, `get_theme_value_view()`, `get_static_url(type)`, `set_tab_bar_badge()`, `app_tabbar_pages()`, `page_share_handle()`.

API calls themselves are plain `uni.request({ url, method:'POST', ... })` inside each page/component, where `url` comes from `get_request_url(...)`. Backend responses use `res.data.code == 0` for success. There is no centralized HTTP client class — follow the existing per-page pattern.

### Global mixins (registered in `main.js`)

- `common/js/common/base.js` — mixed into every component. **Critical: the `setData()` polyfill.** This codebase was converted from a WeChat mini-program, so components call `this.setData({...})` instead of direct assignment. Keep using `this.setData(...)` for reactive updates to stay consistent.
- `common/js/common/share.js` — share/forward behavior mixin.
- `common/js/common/common.js` — exported utility functions (DIY style defaults, condition evaluation, object helpers). Imported explicitly where needed.
- `iconfont` is registered as a global component.

### Platform-conditional code

Conditional-compilation comments are pervasive and meaningful — never strip them:
`// #ifdef MP-WEIXIN || H5 || APP ... // #endif`, `// #ifndef ...`. Same applies inside `pages.json` style blocks. When adding platform-specific logic, branch with these, not runtime checks alone.

### DIY visual page builder

The headline feature is drag-and-drop visual page composition ("DIY 装修"):
- `pages/diy/` renders server-stored DIY config into built-in components (goods, article, image cube/hotspot, video, swiper, custom, etc.).
- `pages/design/` provides layout rendering; `pages/index/index` composes these via `usingComponents` + `componentPlaceholder` declared in `pages.json` (`component-diy`, `component-layout`, `component-form-input`, plus per-plugin list components).
- `pages/diy/init_data.js` is **gitignored** (local DIY seed data).

### Plugins

Each subdirectory of `pages/plugins/` is an optional feature module backed by a corresponding backend plugin — e.g. `distribution`, `seckill`, `coupon`, `wallet`, `points`, `signin`, `realstore` (multi-store), `shop` (multi-merchant), `live`, `video`, `thirdpartylogin`. Frontend calls a plugin by passing its name as the `plugins` arg to `get_request_url(...)`.

### Theming

8 built-in color schemes implemented as CSS class scopes (`.theme-red`, `.theme-blue`, ...) in `common/css/theme.css`. The active theme class is applied at the root and resolved via `globalData.get_theme_value()/get_theme_color()`. New themed styles must be added under each `.theme-*` selector.

### i18n

`locale/index.js` (+ `index-nvue.js` for nvue) initializes vue-i18n with `locale/zh.json` and `locale/en.json`. Use `$t('namespace.key')` in templates/scripts. `i18n_tools.config.js` configures automated key extraction/translation tooling. Active language comes from `globalData.default_language` and is sent with every request.

### Layout conventions

- `pages/` — one folder per route; subfeatures keep their own `components/`. Routes and per-page window/usingComponents config live in `pages.json`.
- `components/` — shared business components (cart, payment, goods-buy, search, share-popup, etc.).
- `uni_modules/` — third-party uni-app plugins (uni-ui, mp-html, sp-editor, ...); generally don't hand-edit.
- `common/css/` — global stylesheets (`theme.css`, `business.css`, `page.css`, `lib.css`, `animation.css`).
- `static/` — bundled images/assets.

## Related projects (separate repos)

Backend: ShopXO (PHP) · DIY editor: shopxo-diy · Form builder: shopxo-form. API docs: https://doc.shopxo.net/article/2.html
