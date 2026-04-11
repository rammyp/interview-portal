# Angular 17 → 20 Upgrade Guide

> ⚠️ **Golden Rule:** Never skip major versions. Upgrade one major version at a time: **17 → 18 → 19 → 20**

---

## Table of Contents

1. [Pre-Upgrade Audit](#phase-0--pre-upgrade-audit)
2. [Angular 17 → 18](#phase-1--angular-17--18)
3. [Angular 18 → 19](#phase-2--angular-18--19)
4. [Angular 19 → 20](#phase-3--angular-19--20)
5. [Testing & Validation](#phase-4--testing--validation)
6. [Ecosystem Libraries](#phase-5--ecosystem-libraries)
7. [Optional Modernizations](#phase-6--optional-modernizations)
8. [Useful Tools](#useful-tools)
9. [Summary Checklist](#summary-checklist)

---

## Phase 0 — Pre-Upgrade Audit

Before touching any version, assess your current state:

```bash
# Check current versions
ng version
node --version
npm --version

# Audit outdated dependencies
npm outdated

# Check for known vulnerabilities
npm audit
```

**Checklist:**
- [ ] Node.js ≥ 18.19 (required for Angular 18+), ≥ 20.x recommended for Angular 20
- [ ] Commit or stash all pending changes (clean git state)
- [ ] Test suite passes on Angular 17
- [ ] Document all 3rd-party Angular libraries (NgRx, Material, etc.)

---

## Phase 1 — Angular 17 → 18

### Key Changes in v18
- **Zoneless change detection** (experimental) introduced
- `afterRender` / `afterNextRender` APIs stabilized
- `@defer` blocks stable
- `RedirectCommand` added to Router

### Upgrade Commands

```bash
ng update @angular/core@18 @angular/cli@18
ng update @angular/material@18   # if using Angular Material
```

### Manual Checks
- Review any use of `ApplicationConfig` — some options shifted
- Migrate `RouterModule` to `provideRouter()` if not already done
- Replace `HttpClientModule` with `provideHttpClient()`

---

## Phase 2 — Angular 18 → 19

### Key Changes in v19
- **Standalone components are now the default** — `standalone: true` can be removed from decorators
- **Signals API fully stable** — `signal()`, `computed()`, `effect()`
- **Signal-based inputs/outputs** — `input()`, `output()`, `model()` replace `@Input()` / `@Output()` (optional)
- **Route-level render mode** for SSR (Server, Client, Prerender per route)
- `linkedSignal()` introduced

### Upgrade Commands

```bash
ng update @angular/core@19 @angular/cli@19
ng update @angular/material@19
```

### Manual Checks
- If `standalone: false`, add it explicitly — it is no longer the default assumption
- Review SSR configs if using Angular Universal (now built-in)
- Check `TestBed` API changes in unit tests

---

## Phase 3 — Angular 19 → 20

### Key Changes in v20
- **Signals-based forms** (experimental)
- **Zoneless** improvements — closer to production-ready
- **`@let` template syntax** stable
- Continued deprecation of older decorator patterns
- `@angular/build` fully replaces `@angular-devkit/build-angular`

### Upgrade Commands

```bash
ng update @angular/core@20 @angular/cli@20
ng update @angular/material@20
```

### Manual Checks
- Replace `@angular-devkit/build-angular` with `@angular/build` in `angular.json`
- Review deprecated `ChangeDetectorRef.markForCheck()` patterns if adopting Signals
- Verify `esbuild` is set as your builder in `angular.json`

---

## Phase 4 — Testing & Validation

Run this checklist **after every major version bump:**

```bash
# Production build check
ng build --configuration production

# Unit tests
ng test --watch=false

# E2E tests
ng e2e

# Linting
ng lint
```

---

## Phase 5 — Ecosystem Libraries

Each library must be upgraded **in sync** with Angular core:

| Library         | Upgrade Command                          |
|-----------------|------------------------------------------|
| Angular Material | `ng update @angular/material@{version}` |
| NgRx            | `ng update @ngrx/store@{version}`        |
| Angular Fire    | `ng update @angular/fire@{version}`      |
| Transloco       | Check release notes manually             |
| ngx-translate   | Check release notes manually             |
| PrimeNG         | Check Angular compatibility matrix       |
| AG Grid         | Check Angular compatibility matrix       |

---

## Phase 6 — Optional Modernizations

Once on v20, consider adopting these modern patterns:

### Signal Inputs & Outputs

```typescript
// Before (still works, but legacy)
@Input() title: string = '';
@Output() clicked = new EventEmitter<void>();

// After (Signals-based, v19+)
title = input<string>('');
clicked = output<void>();
```

### inject() over Constructor Injection

```typescript
// Before
constructor(private http: HttpClient) {}

// After
private http = inject(HttpClient);
```

### @let in Templates

```html
<!-- Angular 20: declare template variables -->
@let user = userSignal();
<p>Welcome, {{ user.name }}</p>
```

### Standalone by Default

```typescript
// Before (Angular 17)
@Component({
  standalone: true,
  selector: 'app-root',
  ...
})

// After (Angular 19+) — standalone: true is the default, no need to declare it
@Component({
  selector: 'app-root',
  ...
})
```

---

## Useful Tools

| Resource | Link |
|----------|-------|
| Official Angular Update Guide | https://update.angular.io/?v=17.0-20.0 |
| Angular Changelog | https://github.com/angular/angular/blob/main/CHANGELOG.md |
| Angular CLI Changelog | https://github.com/angular/angular-cli/blob/main/CHANGELOG.md |
| Angular Material Changelog | https://github.com/angular/components/blob/main/CHANGELOG.md |

```bash
# See what ng update would change without applying it
ng update --dry-run

# Run the official Angular migrations automatically
ng update @angular/core@{version} @angular/cli@{version}
```

---

## Summary Checklist

| Step | Task                                          | Status |
|------|-----------------------------------------------|--------|
| 0    | Audit current state & verify Node.js version  | ☐      |
| 1    | Upgrade to Angular 18 + run tests             | ☐      |
| 2    | Upgrade to Angular 19 + run tests             | ☐      |
| 3    | Upgrade to Angular 20 + run tests             | ☐      |
| 4    | Upgrade all ecosystem libraries               | ☐      |
| 5    | Run full test suite (unit + e2e + lint)       | ☐      |
| 6    | Adopt modern patterns (Signals, inject, etc.) | ☐      |

---

## Notes

- Always back up your project or work on a dedicated branch before upgrading.
- Run `ng update` with no arguments first to see all available updates.
- The Angular team recommends using `ng update` over manual version bumps, as it applies automated code migrations.
- If a 3rd-party library does not yet support the target Angular version, consider pinning it temporarily and upgrading once support is released.

---

*Generated for business upgrade scenario: Angular 17 → 20*
