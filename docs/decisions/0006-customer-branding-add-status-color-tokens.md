# ADR-0006 — Semantic status color tokens for Error, Warning, Success and Info


## Status

Proposed

## Story

**As a** plugin author building OpenSCD / CoMPAS web components  
**I want** a documented set of semantic CSS variables for error, warning, success and info  
**so that** status UI (alerts, banners, validation, badges, buttons) looks consistent across plugins in both light and dark mode.

---

## Current problem

The theme contract today exposes only a single status colour:

```css
--oscd-error: var(--oscd-theme-error, #dc322f);
```

That variable is described as covering “errors, destructive actions, and invalid states”. There is **no guidance** on how plugins should use it:

- as text color?
- as background of an error box?
- as border or icon color?
- as fill of a destructive button?

There are also **no tokens** for warning, success or info. Plugins therefore invent their own greens, yellows and blues - or reuse `--oscd-error` / `--oscd-primary` for unrelated states. The result is inconsistent status UI, poor contrast in one of the two themes, and no way for a distribution to brand these states.

---

## Why one variable is not enough

A single color cannot serve every role of a status surface:

| Role | Why a dedicated token is needed |
| --- | --- |
| Accent (`--oscd-error`) | Icon, left border, outline, badge, button fill |
| On-accent (`--oscd-on-error`) | Text / icon **on** that accent (e.g. white label on a red button) |
| Container (`--oscd-error-container`) | Tinted background of an alert / validation box |
| On-container (`--oscd-on-error-container`) | Body text **inside** that box |

Using the same red for background *and* text fails contrast. Using it only as a background is too loud for an alert panel. Warning yellow is especially fragile: in light mode it almost always needs **dark** text; in dark mode it needs a muted surface plus a brighter accent.

Dark mode is not an inversion of light mode. Saturated `#dc322f` on `#fdf6e3` is not the same decision as saturated `#dc322f` on `#002b36`. The **token names** stay stable; only the **values** change per theme.

---

## Proposed solution

Define **four tokens per status**, for Error, Warning, Success and Info (16 tokens).

Naming follows the existing OpenSCD convention:

- distribution / host sets `--oscd-theme-*`
- plugins consume `--oscd-*` with a fallback

### Token set

| Status | Accent | On accent | Container (box background) | On container (box text) |
| --- | --- | --- | --- | --- |
| Error | `--oscd-error` | `--oscd-on-error` | `--oscd-error-container` | `--oscd-on-error-container` |
| Warning | `--oscd-warning` | `--oscd-on-warning` | `--oscd-warning-container` | `--oscd-on-warning-container` |
| Success | `--oscd-success` | `--oscd-on-success` | `--oscd-success-container` | `--oscd-on-success-container` |
| Info | `--oscd-info` | `--oscd-on-info` | `--oscd-info-container` | `--oscd-on-info-container` |

`--oscd-error` stays for backward compatibility and keeps its current role: **accent** (errors, destructive actions, invalid states). The three new error partners make that role explicit.

### Suggested defaults (Solarized-aligned, light / dark)

Values below use `light-dark()`. A distribution can instead set the same names under `[data-theme="light"]` / `[data-theme="dark"]`.

```css
:root {
  color-scheme: light dark;

  /* Error */
  --oscd-theme-error: light-dark(#dc322f, #f38ba0);
  --oscd-theme-on-error: light-dark(#ffffff, #3b1014);
  --oscd-theme-error-container: light-dark(#fdecea, #3b1618);
  --oscd-theme-on-error-container: light-dark(#8b1a1a, #ffcdd2);

  /* Warning */
  --oscd-theme-warning: light-dark(#b58900, #f0c674);
  --oscd-theme-on-warning: light-dark(#ffffff, #1c1403);
  --oscd-theme-warning-container: light-dark(#fff8e1, #3a2a08);
  --oscd-theme-on-warning-container: light-dark(#7c4a03, #ffe082);

  /* Success */
  --oscd-theme-success: light-dark(#859900, #a6e3a1);
  --oscd-theme-on-success: light-dark(#ffffff, #0b1f0d);
  --oscd-theme-success-container: light-dark(#eef6d6, #1b2a14);
  --oscd-theme-on-success-container: light-dark(#4a5c00, #d8f5a2);

  /* Info */
  --oscd-theme-info: light-dark(#268bd2, #89b4fa);
  --oscd-theme-on-info: light-dark(#ffffff, #0b1a2e);
  --oscd-theme-info-container: light-dark(#e3f2fd, #16324d);
  --oscd-theme-on-info-container: light-dark(#0b4a75, #bbdefb);
}

:root[data-theme="light"] { color-scheme: light; }
:root[data-theme="dark"]  { color-scheme: dark; }
```

### Plugin wiring (existing pattern)

```css
:host {
  --oscd-error: var(--oscd-theme-error, #dc322f);
  --oscd-on-error: var(--oscd-theme-on-error, #ffffff);
  --oscd-error-container: var(--oscd-theme-error-container, #fdecea);
  --oscd-on-error-container: var(--oscd-theme-on-error-container, #8b1a1a);

  --oscd-warning: var(--oscd-theme-warning, #b58900);
  --oscd-on-warning: var(--oscd-theme-on-warning, #ffffff);
  --oscd-warning-container: var(--oscd-theme-warning-container, #fff8e1);
  --oscd-on-warning-container: var(--oscd-theme-on-warning-container, #7c4a03);

  --oscd-success: var(--oscd-theme-success, #859900);
  --oscd-on-success: var(--oscd-theme-on-success, #ffffff);
  --oscd-success-container: var(--oscd-theme-success-container, #eef6d6);
  --oscd-on-success-container: var(--oscd-theme-on-success-container, #4a5c00);

  --oscd-info: var(--oscd-theme-info, #268bd2);
  --oscd-on-info: var(--oscd-theme-on-info, #ffffff);
  --oscd-info-container: var(--oscd-theme-info-container, #e3f2fd);
  --oscd-on-info-container: var(--oscd-theme-on-info-container, #0b4a75);
}
```

Plugins must not set `--oscd-theme-*` themselves. Always keep fallbacks: a distro may not define the new tokens yet.

---

## Usage examples

### 1. Status box / alert (container + on-container + accent border)

```css
.box {
  display: flex;
  gap: 0.75rem;
  padding: 0.75rem 1rem;
  border-radius: var(--oscd-shape, 8px);
  border: 1px solid;
  border-left-width: 4px;
}

.box--error {
  background: var(--oscd-error-container);
  color: var(--oscd-on-error-container);
  border-color: var(--oscd-error);
}

.box--warning {
  background: var(--oscd-warning-container);
  color: var(--oscd-on-warning-container);
  border-color: var(--oscd-warning);
}

.box--success {
  background: var(--oscd-success-container);
  color: var(--oscd-on-success-container);
  border-color: var(--oscd-success);
}

.box--info {
  background: var(--oscd-info-container);
  color: var(--oscd-on-info-container);
  border-color: var(--oscd-info);
}

.box__icon { color: var(--oscd-error); }
.box--warning .box__icon { color: var(--oscd-warning); }
.box--success .box__icon { color: var(--oscd-success); }
.box--info .box__icon { color: var(--oscd-info); }
```

```html
<div class="box box--error">
  <span class="box__icon" aria-hidden="true">⚠</span>
  <span>Validation failed: IED name is missing.</span>
</div>

<div class="box box--warning">
  <span class="box__icon" aria-hidden="true">!</span>
  <span>This change cannot be undone.</span>
</div>

<div class="box box--success">
  <span class="box__icon" aria-hidden="true">✓</span>
  <span>SCL file saved.</span>
</div>

<div class="box box--info">
  <span class="box__icon" aria-hidden="true">i</span>
  <span>3 communication addresses will be updated.</span>
</div>
```

### 2. Solid button / chip (accent + on-accent)

```css
.btn-error {
  background: var(--oscd-error);
  color: var(--oscd-on-error);
}

.btn-success {
  background: var(--oscd-success);
  color: var(--oscd-on-success);
}
```

```html
<button class="btn-error">Delete IED</button>
<button class="btn-success">Publish</button>
```

### 3. Inline validation text / icon only (accent)

```css
.field[aria-invalid="true"] {
  border-color: var(--oscd-error);
}

.field__message--error {
  color: var(--oscd-error);
}
```

Never use `--oscd-error` as both the field background *and* the message colour. For a filled error field, use `--oscd-error-container` + `--oscd-on-error-container`.

---

## Acceptance criteria

- [ ] Theme documentation lists the 16 status tokens, their roles, and the mapping `--oscd-theme-*` → `--oscd-*`.
- [ ] `--oscd-error` remains valid and is documented as the **error accent**, not as a generic “use this for everything error-related” colour.
- [ ] Default values are provided for light and dark (or via `light-dark()` + `color-scheme`).
- [ ] Plugin authors are told to consume tokens with fallbacks and not to hard-code hex colours.
- [ ] Examples cover at least: alert box, solid button, inline validation.
- [ ] Guidance states that status must not rely on colour alone (icon + text).
- [ ] Existing plugins that only read `--oscd-error` keep working.

---

## Out of scope

Hover / focus / disabled variants, high-contrast (`prefers-contrast`) palettes, and mapping onto Material `--md-sys-color-error*` tokens can follow in a separate story once the semantic contract is agreed.
