# Angular Lazy Loading Demo Guide

A complete guide to build, test, and understand lazy loading in Angular 19.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Build Steps](#3-build-steps)
4. [Testing Lazy Loading](#4-testing-lazy-loading)
5. [Project Structure](#5-project-structure)
6. [Lazy Loading Patterns](#6-lazy-loading-patterns)
7. [Preloading Strategies](#7-preloading-strategies)
8. [Bundle Analysis](#8-bundle-analysis)
9. [Best Practices](#9-best-practices)
10. [Summary](#10-summary)

---

## 1. Introduction

**Lazy loading** is a design pattern that delays the loading of modules or components until they are actually needed. Instead of loading the entire application upfront, Angular splits the app into smaller chunks that load on demand.

### Why Use Lazy Loading?

| Benefit | Description |
|---------|-------------|
| ⚡ **Faster Initial Load** | Only loads code needed for the first view |
| 📦 **Smaller Main Bundle** | Features split into separate chunk files |
| 🚀 **Better Performance** | Users don't download unused code |
| 😊 **Improved UX** | App feels faster and more responsive |
| 📈 **Scalability** | Easy to add features without affecting main bundle |

---

## 2. Prerequisites

Before starting, ensure you have:

| Requirement | Version |
|-------------|---------|
| Node.js | v18+ LTS (avoid odd versions like v25) |
| Angular CLI | v19+ |
| npm | v9+ |

### Verify Installation

```bash
node -v
ng version
```

---

## 3. Build Steps

### Step 1: Create New Angular Project

Create a new Angular project with routing enabled and standalone components.

```bash
ng new lazy-loading-demo --routing --style=css --standalone
cd lazy-loading-demo
```

---

### Step 2: Generate Home Component

```bash
ng g c home
```

---

### Step 3: Generate Feature Components

**Products Feature:**
```bash
ng g c features/products/product-list
ng g c features/products/product-detail
```

**Admin Feature:**
```bash
ng g c features/admin/admin-dashboard
ng g c features/admin/admin-users
ng g c features/admin/admin-reports
```

**Settings Feature:**
```bash
ng g c features/settings
```

---

### Step 4: Create Feature Route Files

Create the following route files manually:

| File Path | Purpose |
|-----------|---------|
| `src/app/features/products/products.routes.ts` | Products child routes |
| `src/app/features/admin/admin.routes.ts` | Admin child routes |

---

### Step 5: Configure Main Routes

Update `src/app/app.routes.ts` with:

| Route | Loading Type | Method |
|-------|--------------|--------|
| `/` | Eager or Lazy | `loadComponent` |
| `/products` | **Lazy** | `loadChildren` |
| `/admin` | **Lazy** | `loadChildren` |
| `/settings` | **Lazy** | `loadComponent` |

---

### Step 6: Update App Component

Add navigation links to `src/app/app.ts`:

- Home link → `/`
- Products link → `/products`
- Admin link → `/admin`
- Settings link → `/settings`

Include `<router-outlet>` for rendering routed components.

---

### Step 7: Update Feature Components

Add content to each component to identify when they load:

- Add console.log statements in `ngOnInit()`
- Add visual indicators showing "LAZY LOADED"

---

### Step 8: Run the Application

```bash
ng serve
```

Open browser at `http://localhost:4200`

---

## 4. Testing Lazy Loading

### Method 1: Browser DevTools (Recommended)

| Step | Action |
|------|--------|
| 1 | Open app in browser (`http://localhost:4200`) |
| 2 | Open DevTools (`F12` or `Cmd+Option+I`) |
| 3 | Go to **Network** tab |
| 4 | Filter by **JS** files |
| 5 | Clear the network log (🚫 icon) |
| 6 | Click on **Products** link |
| 7 | Observe new `chunk-XXXXX.js` file loading |
| 8 | Click on **Admin** link |
| 9 | Observe another `chunk-XXXXX.js` file loading |
| 10 | Click on **Settings** link |
| 11 | Observe another chunk file loading |

### What You Should See

```
Network Tab (JS filter):

Initial Load:
├── main.js
├── polyfills.js
└── styles.css

After clicking "Products":
└── chunk-XXXXXX.js  ← Products module loaded!

After clicking "Admin":
└── chunk-YYYYYY.js  ← Admin module loaded!

After clicking "Settings":
└── chunk-ZZZZZZ.js  ← Settings component loaded!
```

---

### Method 2: Console Logs

| Step | Action |
|------|--------|
| 1 | Open DevTools → **Console** tab |
| 2 | Navigate to different routes |
| 3 | Observe console messages |

### Expected Console Output

```
✅ Home component loaded
✅ Products module LAZY LOADED!
✅ Admin module LAZY LOADED!
✅ Settings component LAZY LOADED!
```

---

### Method 3: Build Analysis

```bash
# Build for production
ng build --stats-json

# Check output
ls -la dist/lazy-loading-demo/browser/
```

### Expected Build Output

```
dist/lazy-loading-demo/browser/
├── main-XXXXX.js           (main bundle)
├── polyfills-XXXXX.js      (polyfills)
├── chunk-AAAAA.js          (products feature)
├── chunk-BBBBB.js          (admin feature)
├── chunk-CCCCC.js          (settings component)
├── index.html
└── styles-XXXXX.css
```

---

## 5. Project Structure

```
src/app/
├── app.ts                           # Root component
├── app.routes.ts                    # Main routes (with lazy loading)
├── app.config.ts                    # App configuration
│
├── home/
│   └── home.ts                      # Home component
│
└── features/
    │
    ├── products/                    # LAZY LOADED FEATURE
    │   ├── products.routes.ts       # Child routes
    │   ├── product-list/
    │   │   └── product-list.ts
    │   └── product-detail/
    │       └── product-detail.ts
    │
    ├── admin/                       # LAZY LOADED FEATURE
    │   ├── admin.routes.ts          # Child routes
    │   ├── admin-dashboard/
    │   │   └── admin-dashboard.ts
    │   ├── admin-users/
    │   │   └── admin-users.ts
    │   └── admin-reports/
    │       └── admin-reports.ts
    │
    └── settings/                    # LAZY LOADED COMPONENT
        └── settings.ts
```

---

## 6. Lazy Loading Patterns

### Pattern 1: loadChildren (Feature Routes)

Use for loading a set of child routes.

| Aspect | Description |
|--------|-------------|
| **Use Case** | Feature modules with multiple routes |
| **File Required** | Separate routes file (e.g., `products.routes.ts`) |
| **Loads** | All routes defined in the child routes file |

**Route Configuration:**
```
path: 'products' → loadChildren → products.routes.ts
```

---

### Pattern 2: loadComponent (Standalone Component)

Use for loading a single standalone component.

| Aspect | Description |
|--------|-------------|
| **Use Case** | Single page/component without child routes |
| **File Required** | Component file only |
| **Loads** | Just the specified component |

**Route Configuration:**
```
path: 'settings' → loadComponent → settings.ts
```

---

### Comparison

| Feature | loadChildren | loadComponent |
|---------|--------------|---------------|
| Child routes | ✅ Yes | ❌ No |
| Multiple components | ✅ Yes | ❌ Single |
| Routes file needed | ✅ Yes | ❌ No |
| Complexity | Higher | Lower |
| Best for | Feature modules | Single pages |

---

## 7. Preloading Strategies

### Available Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **NoPreloading** | Default. Load only on navigation | Slow networks, large apps |
| **PreloadAllModules** | Preload all lazy modules after initial load | Fast networks, better UX |
| **Custom** | Define your own logic | Selective preloading |

### Configuration Location

Update `src/app/app.config.ts` to change preloading strategy.

### Behavior Comparison

| Strategy | Initial Load | After Initial Load | On Navigation |
|----------|--------------|-------------------|---------------|
| NoPreloading | Main bundle only | Nothing | Load chunk |
| PreloadAllModules | Main bundle only | All chunks | Already loaded |

---

## 8. Bundle Analysis

### Step 1: Build with Stats

```bash
ng build --stats-json
```

### Step 2: Install Analyzer

```bash
npm install -g webpack-bundle-analyzer
```

### Step 3: Run Analyzer

```bash
npx webpack-bundle-analyzer dist/lazy-loading-demo/browser/stats.json
```

### What to Look For

| Indicator | Good | Bad |
|-----------|------|-----|
| Main bundle size | < 500KB | > 1MB |
| Feature chunks | Separate files | All in main bundle |
| Unused code | None | Large unused portions |

---

## 9. Best Practices

### ✅ Do's

| Practice | Reason |
|----------|--------|
| Lazy load feature modules | Reduces initial bundle size |
| Use `loadComponent` for single pages | Simpler than creating routes file |
| Group related components in features | Better organization |
| Add preloading for critical features | Improves perceived performance |
| Analyze bundle regularly | Catch size regressions |

### ❌ Don'ts

| Practice | Reason |
|----------|--------|
| Lazy load tiny components | Overhead not worth it |
| Lazy load shared/core modules | Needed everywhere |
| Over-split the application | Too many HTTP requests |
| Forget to test lazy loading | May break silently |

### Recommended Structure

| Module Type | Loading | Reason |
|-------------|---------|--------|
| Core (auth, http) | Eager | Used everywhere |
| Shared (UI components) | Eager | Used across features |
| Feature modules | **Lazy** | Not always needed |
| Admin/Settings | **Lazy** | Rarely accessed |

---

## 10. Summary

### What We Built

| Component | Route | Loading Type |
|-----------|-------|--------------|
| Home | `/` | Eager |
| Product List | `/products` | **Lazy** |
| Product Detail | `/products/:id` | **Lazy** |
| Admin Dashboard | `/admin` | **Lazy** |
| Admin Users | `/admin/users` | **Lazy** |
| Admin Reports | `/admin/reports` | **Lazy** |
| Settings | `/settings` | **Lazy** |

### Key Takeaways

1. **Lazy loading splits your app** into smaller chunks loaded on demand

2. **Two methods to lazy load:**
   - `loadChildren` → For feature routes
   - `loadComponent` → For standalone components

3. **Test lazy loading** using Browser DevTools → Network tab → JS filter

4. **Preloading strategies** can improve UX by loading modules in background

5. **Bundle analysis** helps identify optimization opportunities

### Quick Reference

| Task | Command/Action |
|------|----------------|
| Create project | `ng new app-name --routing --standalone` |
| Generate component | `ng g c component-name` |
| Run dev server | `ng serve` |
| Build for production | `ng build` |
| Build with stats | `ng build --stats-json` |
| Analyze bundle | `npx webpack-bundle-analyzer dist/*/stats.json` |
| Test lazy loading | DevTools → Network → JS filter |

---

## Additional Resources

| Resource | URL |
|----------|-----|
| Angular Routing Docs | https://angular.dev/guide/routing |
| Lazy Loading Guide | https://angular.dev/guide/ngmodules/lazy-loading |
| Preloading Strategies | https://angular.dev/guide/routing/preloading |

---

**Happy Coding! 🚀**
