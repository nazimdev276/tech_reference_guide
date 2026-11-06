# 10 Guided Projects: HTML, CSS & JavaScript (CSS-Focused)

A hands-on path from beginner to confident front-end builder. **About 70% of the work is CSS**, with HTML for structure and JavaScript only where it adds real value.

## How to use this guide

1. Create one folder per project: `project-01`, `project-02`, etc.
2. Each folder has `index.html`, `style.css`, and (when needed) `script.js`.
3. **Don't copy-paste the CSS challenges.** Type them out, break things, and fix them. That is how CSS sticks.
4. Use your browser's DevTools (`F12`) constantly: inspect elements, toggle properties, and watch the box model.
5. Finish the **Checklist** of each project before moving on.

**Basic starter file** (reused in every project):

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Project</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>

  <script src="script.js"></script>
</body>
</html>
```

**Basic CSS reset** (paste at the top of every `style.css`):

```css
*, *::before, *::after { box-sizing: border-box; }
* { margin: 0; }
img { max-width: 100%; display: block; }
body { font-family: system-ui, sans-serif; line-height: 1.5; }
```

## Roadmap

| # | Project | Main CSS focus | JS level |
|---|---------|----------------|----------|
| 1 | Profile Card | Box model, typography, colors, variables | None |
| 2 | Responsive Navbar | Flexbox, media queries, positioning | Light |
| 3 | Pricing Cards | CSS Grid, hover, transitions | None |
| 4 | Landing Page Hero | Gradients, `clamp()`, layering, `z-index` | None |
| 5 | Image Gallery | Grid, `aspect-ratio`, `object-fit`, filters | Medium |
| 6 | Dark Mode Dashboard | Custom properties, theming | Medium |
| 7 | Animated Buttons & Loaders | `@keyframes`, transforms, animation | None |
| 8 | Glassmorphism Login Form | Form styling, pseudo-classes, `backdrop-filter` | Light |
| 9 | 3D Flip Cards | `perspective`, `transform-style`, 3D transforms | Light |
| 10 | Capstone Portfolio Site | Everything combined | Medium |

---

## Project 1: Profile Card

**Goal:** Build a centered card with an avatar, name, bio, and social buttons.

**You will learn:** the box model, `padding` vs `margin`, `border-radius`, `box-shadow`, font styling, CSS variables, centering with Flexbox.

### HTML structure

```html
<main class="wrapper">
  <article class="card">
    <img class="avatar" src="https://i.pravatar.cc/200" alt="Profile photo">
    <h1 class="name">Your Name</h1>
    <p class="role">Front-End Developer</p>
    <p class="bio">I turn coffee into clean, responsive interfaces.</p>
    <div class="actions">
      <a href="#" class="btn btn-primary">Follow</a>
      <a href="#" class="btn btn-ghost">Message</a>
    </div>
  </article>
</main>
```

### Steps

1. **Define variables** in `:root`:
   ```css
   :root {
     --bg: #f1f5f9;
     --card: #ffffff;
     --text: #0f172a;
     --muted: #64748b;
     --accent: #6366f1;
     --radius: 16px;
   }
   ```
2. **Center the card** on the page: make `.wrapper` `min-height: 100vh` and use `display: grid; place-items: center;`.
3. **Style the card:** `max-width: 340px`, `padding: 2rem`, `text-align: center`, `border-radius: var(--radius)`, and a soft `box-shadow`.
4. **Make the avatar circular:** `width: 110px; height: 110px; border-radius: 50%; object-fit: cover; margin: 0 auto 1rem;` and add a `border: 4px solid var(--accent)`.
5. **Typography:** name at `1.5rem` bold, role in `--accent` with `text-transform: uppercase; letter-spacing: .08em; font-size: .8rem`, bio in `--muted`.
6. **Buttons:** build a shared `.btn` class (padding, radius, font-weight, `text-decoration: none`), then `.btn-primary` (filled) and `.btn-ghost` (outlined).
7. **Interaction:** add `transition: transform .2s, box-shadow .2s` and a `:hover` that lifts the card or button with `translateY(-3px)`.

### Stretch goals
- Add a gradient banner behind the avatar using `linear-gradient`.
- Add a row of 3 stats (Posts / Followers / Following) using Flexbox with `justify-content: space-around`.

### Checklist
- [ ] Card is perfectly centered
- [ ] Every color comes from a variable
- [ ] Hover states feel smooth, not jumpy
- [ ] You can explain `content`, `padding`, `border`, and `margin` in DevTools

---

## Project 2: Responsive Navbar

**Goal:** A navigation bar that is horizontal on desktop and becomes a hamburger menu on mobile.

**You will learn:** Flexbox in depth (`justify-content`, `align-items`, `gap`), `position: sticky`, media queries, mobile-first design, pseudo-elements for hover effects.

### HTML structure

```html
<header class="site-header">
  <nav class="nav">
    <a href="#" class="logo">Brand<span>.</span></a>

    <button class="nav-toggle" aria-label="Toggle menu" aria-expanded="false">
      <span></span><span></span><span></span>
    </button>

    <ul class="nav-links">
      <li><a href="#">Home</a></li>
      <li><a href="#">About</a></li>
      <li><a href="#">Services</a></li>
      <li><a href="#">Contact</a></li>
    </ul>
  </nav>
</header>
<main style="height:200vh; padding:2rem">Scroll to test the sticky header.</main>
```

### Steps

1. **Mobile first:** style the small-screen version first. The `.nav-links` list is hidden (`display: none`) and the toggle button is visible.
2. **Header:** `position: sticky; top: 0; background: white; box-shadow: 0 2px 10px rgba(0,0,0,.08); z-index: 100;`
3. **Nav layout:** `.nav { display: flex; justify-content: space-between; align-items: center; padding: 1rem 1.5rem; }`
4. **Hamburger bars:** style the three `<span>` elements as `display:block; width:24px; height:3px; background:#111; margin:5px 0;`
5. **Mobile dropdown:** when `.nav-links.open` is present, show it as a full-width column positioned under the header (`position: absolute; top: 100%; left: 0; right: 0;`).
6. **Desktop breakpoint:**
   ```css
   @media (min-width: 768px) {
     .nav-toggle { display: none; }
     .nav-links { display: flex; gap: 2rem; position: static; }
   }
   ```
7. **Animated underline on links:** use `a::after` with `width: 0` that grows to `100%` on `:hover`, using `transition: width .3s`.
8. **JS (small):**
   ```js
   const toggle = document.querySelector('.nav-toggle');
   const links = document.querySelector('.nav-links');

   toggle.addEventListener('click', () => {
     const open = links.classList.toggle('open');
     toggle.setAttribute('aria-expanded', open);
   });
   ```

### Stretch goals
- Animate the hamburger into an "X" when open (rotate the first and last bars, fade the middle).
- Highlight the active link with a different color.

### Checklist
- [ ] Menu works on a 375px-wide viewport
- [ ] Header stays visible while scrolling
- [ ] You wrote CSS for mobile first, then added `min-width` queries

---

## Project 3: Pricing Cards

**Goal:** Three pricing tiers side by side, with the middle one highlighted.

**You will learn:** CSS Grid (`grid-template-columns`, `repeat`, `minmax`, `auto-fit`), `gap`, pseudo-elements for badges, hover and transform effects, `list-style` customization.

### HTML structure (repeat for 3 cards)

```html
<section class="pricing">
  <article class="plan">
    <h2>Starter</h2>
    <p class="price">$9<span>/mo</span></p>
    <ul>
      <li>1 Project</li>
      <li>Basic Support</li>
      <li>1GB Storage</li>
    </ul>
    <a href="#" class="plan-btn">Choose Plan</a>
  </article>

  <article class="plan featured">...</article>
  <article class="plan">...</article>
</section>
```

### Steps

1. **Responsive grid with no media queries:**
   ```css
   .pricing {
     display: grid;
     grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
     gap: 1.5rem;
     max-width: 1000px;
     margin: 4rem auto;
     padding: 0 1rem;
   }
   ```
   Resize the window and watch it reflow. This is the power of `auto-fit` + `minmax`.
2. **Card base:** white background, `border: 1px solid #e2e8f0`, `border-radius: 16px`, `padding: 2rem`, `display: flex; flex-direction: column;`.
3. **Push buttons to the bottom:** give the `ul` `flex: 1` so all buttons align even if lists differ in length.
4. **Price styling:** large number (`3rem`), small `/mo` using `font-size: 1rem; color: gray`.
5. **Custom list icons:** remove default bullets and add a checkmark:
   ```css
   .plan li { list-style: none; padding: .4rem 0 .4rem 1.6rem; position: relative; }
   .plan li::before { content: "✓"; position: absolute; left: 0; color: var(--accent); }
   ```
6. **Featured plan:** `transform: scale(1.05)`, accent border, and a "Most Popular" badge using `.featured::before` positioned at the top center.
7. **Hover:** all cards lift with `translateY(-6px)` and a deeper shadow; use `transition`.

### Stretch goals
- Add a monthly/yearly toggle (a few lines of JS swapping the price text).
- Use `:nth-child()` to give each card a different accent color.

### Checklist
- [ ] Layout adapts from 3 columns to 1 without a single media query
- [ ] Buttons are aligned on the same horizontal line
- [ ] You understand the difference between `auto-fit` and `auto-fill`

---

## Project 4: Landing Page Hero Section

**Goal:** A modern hero section with a gradient background, large responsive heading, call-to-action buttons, and decorative shapes.

**You will learn:** gradients, `clamp()` for fluid typography, `position: absolute/relative`, `z-index` layering, `::before`/`::after` decoration, `min-height: 100vh`, `background-clip: text`.

### HTML structure

```html
<section class="hero">
  <div class="blob blob-1"></div>
  <div class="blob blob-2"></div>

  <div class="hero-content">
    <span class="badge">New in 2026</span>
    <h1>Build <em>beautiful</em> websites faster</h1>
    <p>A short, friendly sentence explaining what you do and why it matters.</p>
    <div class="cta">
      <a href="#" class="btn btn-primary">Get Started</a>
      <a href="#" class="btn btn-ghost">Watch Demo</a>
    </div>
  </div>
</section>
```

### Steps

1. **Hero container:** `position: relative; min-height: 100vh; display: grid; place-items: center; overflow: hidden; background: linear-gradient(135deg, #0f172a, #312e81);` plus `color: white`.
2. **Fluid heading:** `font-size: clamp(2.2rem, 6vw, 4.5rem); line-height: 1.1;` and a `max-width: 18ch` for nice line breaks.
3. **Gradient text:** style the `<em>`:
   ```css
   h1 em {
     font-style: normal;
     background: linear-gradient(90deg, #22d3ee, #a78bfa);
     -webkit-background-clip: text;
     background-clip: text;
     color: transparent;
   }
   ```
4. **Blurred blobs:** absolutely position two large circles with `filter: blur(80px); opacity: .5;` in the corners.
5. **Layering:** set `.hero-content { position: relative; z-index: 1; }` so text sits above the blobs.
6. **Badge:** pill shape (`border-radius: 999px`), semi-transparent background `rgba(255,255,255,.1)`, thin border.
7. **CTA buttons:** use `display: flex; gap: 1rem; flex-wrap: wrap; justify-content: center;`.

### Stretch goals
- Add a subtle dotted grid background with `radial-gradient` and `background-size`.
- Add a "scroll down" arrow at the bottom, animated with a simple bounce.

### Checklist
- [ ] Heading scales smoothly with zero media queries
- [ ] Text is readable above the decorations
- [ ] You can explain how `z-index` works with `position`

---

## Project 5: Image Gallery with Lightbox

**Goal:** A responsive photo grid with hover overlays, category filters, and a click-to-enlarge lightbox.

**You will learn:** CSS Grid layouts, `aspect-ratio`, `object-fit`, overlays with `inset`, CSS `filter`, DOM manipulation, event delegation.

### HTML structure

```html
<div class="filters">
  <button class="filter active" data-filter="all">All</button>
  <button class="filter" data-filter="nature">Nature</button>
  <button class="filter" data-filter="city">City</button>
</div>

<section class="gallery">
  <figure class="item" data-category="nature">
    <img src="https://picsum.photos/id/1018/600/600" alt="Mountains">
    <figcaption>Mountains</figcaption>
  </figure>
  <!-- add 8-12 more with different categories -->
</section>

<div class="lightbox" id="lightbox">
  <button class="close" aria-label="Close">&times;</button>
  <img src="" alt="">
</div>
```

### Steps

1. **Grid:** `grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 1rem;`
2. **Uniform tiles:** `.item { aspect-ratio: 1; overflow: hidden; position: relative; border-radius: 12px; }` and `.item img { width: 100%; height: 100%; object-fit: cover; }`
3. **Zoom on hover:** `.item img { transition: transform .4s; }` then `.item:hover img { transform: scale(1.1); }`
4. **Overlay caption:** make `figcaption` `position: absolute; inset: 0;` with a dark gradient background, `opacity: 0`, and show on hover. Use `display: grid; align-items: end;` to anchor the text at the bottom.
5. **Grayscale effect:** `filter: grayscale(100%)` by default, `grayscale(0)` on hover.
6. **Lightbox CSS:** `position: fixed; inset: 0; background: rgba(0,0,0,.9); display: none; place-items: center;` and `.lightbox.active { display: grid; }`. Limit the image with `max-width: 90vw; max-height: 90vh;`.
7. **JS: filtering**
   ```js
   const buttons = document.querySelectorAll('.filter');
   const items = document.querySelectorAll('.item');

   buttons.forEach(btn => {
     btn.addEventListener('click', () => {
       buttons.forEach(b => b.classList.remove('active'));
       btn.classList.add('active');

       const filter = btn.dataset.filter;
       items.forEach(item => {
         const show = filter === 'all' || item.dataset.category === filter;
         item.style.display = show ? '' : 'none';
       });
     });
   });
   ```
8. **JS: lightbox**
   ```js
   const lightbox = document.getElementById('lightbox');
   const lightboxImg = lightbox.querySelector('img');

   document.querySelector('.gallery').addEventListener('click', e => {
     const img = e.target.closest('.item')?.querySelector('img');
     if (!img) return;
     lightboxImg.src = img.src;
     lightbox.classList.add('active');
   });

   lightbox.addEventListener('click', () => lightbox.classList.remove('active'));
   document.addEventListener('keydown', e => {
     if (e.key === 'Escape') lightbox.classList.remove('active');
   });
   ```

### Stretch goals
- Make some tiles span 2 columns or rows (`grid-column: span 2`) for a masonry feel.
- Add fade-in animation when filtering.

### Checklist
- [ ] All tiles are the same size regardless of the source image
- [ ] Lightbox closes with click and `Esc`
- [ ] You understand why `object-fit: cover` is needed

---

## Project 6: Dark Mode Dashboard

**Goal:** A small dashboard layout (sidebar, header, stat cards) with a working light/dark theme toggle that remembers the user's choice.

**You will learn:** CSS custom properties for theming, `[data-theme]` attribute selectors, `prefers-color-scheme`, grid layouts for app shells, `localStorage`.

### HTML structure

```html
<div class="app">
  <aside class="sidebar">
    <h2 class="brand">Dash</h2>
    <nav>
      <a href="#" class="active">Overview</a>
      <a href="#">Analytics</a>
      <a href="#">Settings</a>
    </nav>
  </aside>

  <div class="main">
    <header class="topbar">
      <h1>Overview</h1>
      <button id="theme-toggle" aria-label="Toggle theme">🌙</button>
    </header>

    <section class="stats">
      <div class="stat"><span class="label">Users</span><strong>12,480</strong></div>
      <div class="stat"><span class="label">Revenue</span><strong>$48.2k</strong></div>
      <div class="stat"><span class="label">Orders</span><strong>1,284</strong></div>
      <div class="stat"><span class="label">Growth</span><strong>+12%</strong></div>
    </section>
  </div>
</div>
```

### Steps

1. **Theme variables:**
   ```css
   :root {
     --bg: #f8fafc;
     --surface: #ffffff;
     --text: #0f172a;
     --muted: #64748b;
     --border: #e2e8f0;
     --accent: #6366f1;
   }

   [data-theme="dark"] {
     --bg: #0b1120;
     --surface: #111827;
     --text: #e5e7eb;
     --muted: #94a3b8;
     --border: #1f2937;
     --accent: #818cf8;
   }
   ```
   Never hard-code a color again after this. Always use `var(--...)`.
2. **App shell:** `.app { display: grid; grid-template-columns: 240px 1fr; min-height: 100vh; }`
3. **Sidebar:** `background: var(--surface); border-right: 1px solid var(--border);` with links as block-level items with padding and rounded corners. The `.active` link uses a tinted accent background.
4. **Stat cards:** `.stats { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem; }`
5. **Smooth theme change:** add `transition: background-color .3s, color .3s, border-color .3s` to `body`, `.sidebar`, `.stat`.
6. **Mobile:** below `768px`, change `.app` to a single column and hide the sidebar or move it to the top.
7. **JS:**
   ```js
   const root = document.documentElement;
   const btn = document.getElementById('theme-toggle');

   const saved = localStorage.getItem('theme');
   const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
   const initial = saved || (prefersDark ? 'dark' : 'light');
   root.dataset.theme = initial;
   btn.textContent = initial === 'dark' ? '☀️' : '🌙';

   btn.addEventListener('click', () => {
     const next = root.dataset.theme === 'dark' ? 'light' : 'dark';
     root.dataset.theme = next;
     localStorage.setItem('theme', next);
     btn.textContent = next === 'dark' ? '☀️' : '🌙';
   });
   ```

### Stretch goals
- Add a simple bar chart using only `div`s with heights set via CSS variables (`style="--h: 70%"`).
- Add `color-scheme: dark` in dark mode so scrollbars and form controls match.

### Checklist
- [ ] Switching themes changes every color in the UI
- [ ] Choice persists after refresh
- [ ] First visit respects the OS preference

---

## Project 7: Animated Buttons & Loaders

**Goal:** A showcase page with 6 buttons and 4 loaders, all animated with pure CSS.

**You will learn:** `transition` vs `animation`, `@keyframes`, `transform` (translate, rotate, scale), `animation-delay`, `animation-iteration-count`, easing functions, `::before`/`::after` tricks.

### Part A: Buttons

Create a grid of buttons, each with a different effect:

1. **Slide-fill:** a `::before` pseudo-element sits at `left: -100%` and slides to `left: 0` on hover, filling the button with color.
   ```css
   .btn-fill { position: relative; overflow: hidden; z-index: 0; }
   .btn-fill::before {
     content: ""; position: absolute; inset: 0; background: var(--accent);
     transform: translateX(-100%); transition: transform .3s; z-index: -1;
   }
   .btn-fill:hover::before { transform: translateX(0); }
   ```
2. **Glow pulse:** an infinite `box-shadow` animation using `@keyframes pulse`.
3. **Press effect:** `:active { transform: scale(.95); }`
4. **Border draw:** use two pseudo-elements to animate border lines around the button.
5. **Gradient shift:** a large `background-size: 200%` gradient that moves with `background-position` on hover.
6. **Arrow reveal:** an arrow icon that slides in from the left as the text moves right.

### Part B: Loaders

1. **Spinner:**
   ```css
   .spinner {
     width: 48px; height: 48px;
     border: 5px solid #e2e8f0;
     border-top-color: var(--accent);
     border-radius: 50%;
     animation: spin 1s linear infinite;
   }
   @keyframes spin { to { transform: rotate(360deg); } }
   ```
2. **Bouncing dots:** three `<span>`s with the same animation, each with a different `animation-delay` (`0s`, `.15s`, `.3s`).
3. **Progress bar:** a bar whose inner element animates `width` from `0%` to `100%` (try also with `transform: scaleX()` and notice the performance difference).
4. **Pulsing skeleton:** a gray block with a shimmering gradient sliding across using `background-position` animation, like the loading placeholders on modern sites.

### Key rules to learn
- Prefer animating `transform` and `opacity`. They are cheap for the browser.
- Use `animation-fill-mode: forwards` to keep the end state.
- Respect users who prefer less motion:
  ```css
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { animation-duration: .01ms !important; transition-duration: .01ms !important; }
  }
  ```

### Stretch goals
- Create a "typing" text effect using `steps()` and `width` animation.
- Build a pure-CSS hamburger-to-X animation.

### Checklist
- [ ] You can write `@keyframes` from memory
- [ ] You know when to use `transition` and when to use `animation`
- [ ] Reduced-motion media query is included

---

## Project 8: Glassmorphism Login Form

**Goal:** A frosted-glass login/signup card over a colorful background, with polished inputs and live validation feedback.

**You will learn:** `backdrop-filter`, translucent colors, styling form controls, `:focus`, `:focus-within`, `:valid`/`:invalid`, `:placeholder-shown`, floating labels, `accent-color`.

### HTML structure

```html
<main class="scene">
  <form class="glass" id="login-form" novalidate>
    <h1>Welcome back</h1>

    <div class="field">
      <input type="email" id="email" placeholder=" " required>
      <label for="email">Email</label>
    </div>

    <div class="field">
      <input type="password" id="password" placeholder=" " minlength="8" required>
      <label for="password">Password</label>
    </div>

    <label class="check"><input type="checkbox"> Remember me</label>

    <button type="submit">Sign in</button>
    <p class="message" id="message" aria-live="polite"></p>
  </form>
</main>
```

### Steps

1. **Background:** `.scene` fills the screen with a vivid gradient or an image, plus a couple of blurred colored circles for depth (reuse the blob trick from Project 4).
2. **Glass card:**
   ```css
   .glass {
     width: min(92%, 400px);
     padding: 2.5rem 2rem;
     background: rgba(255, 255, 255, .12);
     backdrop-filter: blur(16px);
     -webkit-backdrop-filter: blur(16px);
     border: 1px solid rgba(255, 255, 255, .25);
     border-radius: 20px;
     box-shadow: 0 10px 40px rgba(0, 0, 0, .25);
     color: white;
   }
   ```
3. **Floating labels:** position the label absolutely inside the input. When the input is focused or not showing its placeholder, move the label up and shrink it:
   ```css
   .field { position: relative; margin-bottom: 1.5rem; }
   .field input {
     width: 100%; padding: 1rem .9rem .4rem;
     background: rgba(255,255,255,.1);
     border: 1px solid rgba(255,255,255,.3);
     border-radius: 10px; color: white; outline: none;
   }
   .field label {
     position: absolute; left: .9rem; top: 50%;
     transform: translateY(-50%);
     transition: all .2s; pointer-events: none; opacity: .8;
   }
   .field input:focus + label,
   .field input:not(:placeholder-shown) + label {
     top: .6rem; font-size: .7rem; transform: none;
   }
   ```
4. **Focus ring:** replace the default outline with a glowing `box-shadow` on `:focus`.
5. **Validation styling:** after the user has typed something (`:not(:placeholder-shown)`), show a green border for `:valid` and a red border for `:invalid`.
6. **Custom checkbox:** set `accent-color`, or go further and build one from scratch with `appearance: none`.
7. **Button:** full-width gradient button with hover lift and a disabled style.
8. **JS (small):** on submit, prevent the default behavior, check validity, and show a message.
   ```js
   const form = document.getElementById('login-form');
   const message = document.getElementById('message');

   form.addEventListener('submit', e => {
     e.preventDefault();
     if (!form.checkValidity()) {
       message.textContent = 'Please enter a valid email and a password of at least 8 characters.';
       return;
     }
     message.textContent = 'Signed in (demo only).';
   });
   ```

### Stretch goals
- Add a "show password" eye toggle.
- Add a shake animation to the card on invalid submit.

### Checklist
- [ ] Labels float smoothly with no JavaScript
- [ ] Focus state is clearly visible (accessibility matters)
- [ ] You know why `placeholder=" "` is part of the trick

---

## Project 9: 3D Flip Cards

**Goal:** A set of cards that flip in 3D to reveal their back side on hover (and on tap for mobile).

**You will learn:** `perspective`, `transform-style: preserve-3d`, `backface-visibility`, `rotateY`, 3D thinking, combining CSS with a tiny JS fallback for touch.

### HTML structure

```html
<section class="cards">
  <div class="flip">
    <div class="flip-inner">
      <div class="face front">
        <h2>HTML</h2>
        <p>Structure</p>
      </div>
      <div class="face back">
        <p>The skeleton of every web page. Tags, semantics, accessibility.</p>
      </div>
    </div>
  </div>
  <!-- repeat for CSS and JavaScript -->
</section>
```

### Steps

1. **Container:** `.flip { perspective: 1000px; width: 260px; height: 340px; }` The perspective lives on the parent, not on the rotating element.
2. **Inner wrapper:**
   ```css
   .flip-inner {
     position: relative; width: 100%; height: 100%;
     transform-style: preserve-3d;
     transition: transform .7s cubic-bezier(.4, .2, .2, 1);
   }
   .flip:hover .flip-inner { transform: rotateY(180deg); }
   ```
3. **Faces:**
   ```css
   .face {
     position: absolute; inset: 0;
     backface-visibility: hidden;
     -webkit-backface-visibility: hidden;
     display: grid; place-content: center; text-align: center;
     padding: 1.5rem; border-radius: 18px;
   }
   .back { transform: rotateY(180deg); }
   ```
4. **Color each card differently** using custom properties (`style="--c1:#f97316; --c2:#ef4444"`) and a gradient on `.front` built from them.
5. **Depth:** add a `box-shadow` and a very slight `translateZ` to inner text for a parallax feel.
6. **Touch support (JS):**
   ```js
   document.querySelectorAll('.flip').forEach(card => {
     card.addEventListener('click', () => card.classList.toggle('flipped'));
   });
   ```
   Then add `.flip.flipped .flip-inner { transform: rotateY(180deg); }` to the CSS.
7. **Accessibility:** make cards focusable with `tabindex="0"` and add `.flip:focus-visible .flip-inner { transform: rotateY(180deg); }`.

### Stretch goals
- Build a 3D rotating cube with six faces.
- Tilt a card toward the mouse using `mousemove` and CSS variables for `rotateX`/`rotateY`.

### Checklist
- [ ] Back face is never visible mirrored
- [ ] Works with mouse, touch, and keyboard
- [ ] You can explain why `perspective` and `preserve-3d` are needed

---

## Project 10: Capstone Portfolio Website

**Goal:** A complete, responsive, single-page personal portfolio that combines every skill so far. This one is yours: make it look like *you*.

**Sections to build**

1. **Sticky navbar** (Project 2) with smooth-scroll anchor links.
2. **Hero** (Project 4) with your name, role, and CTA.
3. **About** with a two-column layout (image + text) that stacks on mobile.
4. **Skills** displayed as animated progress bars or tag chips.
5. **Projects** as a card grid (Project 3 + 5) with hover overlays linking to your previous 9 projects.
6. **Contact** form styled like Project 8.
7. **Footer** with social icons.
8. **Theme toggle** (Project 6).

### Required CSS techniques (check each one off)

- [ ] CSS variables for colors, spacing, radius, and shadows
- [ ] A fluid type scale with `clamp()`
- [ ] Flexbox for components, Grid for page layout
- [ ] At least 2 media queries (or intrinsic layouts) for mobile, tablet, and desktop
- [ ] `scroll-behavior: smooth` and `scroll-margin-top` so sticky headers don't cover section titles
- [ ] At least 3 different hover/focus effects
- [ ] At least 2 `@keyframes` animations
- [ ] Dark mode
- [ ] `prefers-reduced-motion` support
- [ ] A visible `:focus-visible` style on every interactive element

### Key JavaScript features

1. **Scroll-reveal with IntersectionObserver:** elements fade and slide up as they enter the viewport.
   ```css
   .reveal { opacity: 0; transform: translateY(30px); transition: opacity .6s, transform .6s; }
   .reveal.visible { opacity: 1; transform: none; }
   ```
   ```js
   const observer = new IntersectionObserver((entries) => {
     entries.forEach(entry => {
       if (entry.isIntersecting) {
         entry.target.classList.add('visible');
         observer.unobserve(entry.target);
       }
     });
   }, { threshold: 0.15 });

   document.querySelectorAll('.reveal').forEach(el => observer.observe(el));
   ```
2. **Active nav link on scroll:** highlight the link whose section is currently in view.
3. **Back-to-top button** that appears after scrolling 400px.
4. **Skill bars** that animate their width when scrolled into view (set a `--level` variable in HTML and read it in CSS).

### Project structure suggestion

```
portfolio/
├── index.html
├── css/
│   ├── base.css        # reset, variables, typography
│   ├── layout.css      # grid, containers, sections
│   ├── components.css  # buttons, cards, forms, navbar
│   └── themes.css      # dark mode
├── js/
│   └── main.js
└── images/
```

Practice splitting CSS into files, or use `@import` / multiple `<link>` tags.

### Final polish

- Run Lighthouse in Chrome DevTools and aim for 90+ in Accessibility and Best Practices.
- Test at widths of 320px, 768px, and 1440px.
- Tab through the whole page with your keyboard only.
- Deploy it free on GitHub Pages, Netlify, or Cloudflare Pages.

---

## CSS Concepts Cheat Sheet

| Concept | Where you practiced it |
|---------|------------------------|
| Box model, `box-sizing` | 1 |
| Variables (custom properties) | 1, 6, 10 |
| Flexbox | 1, 2, 3 |
| Grid | 3, 5, 6 |
| Positioning and `z-index` | 2, 4, 7 |
| Media queries and responsive design | 2, 3, 6, 10 |
| Fluid type with `clamp()` | 4, 10 |
| Gradients | 4, 7, 8 |
| Transitions | 1, 2, 3, 5 |
| `@keyframes` animation | 7, 10 |
| 2D and 3D transforms | 3, 7, 9 |
| Pseudo-classes and pseudo-elements | 2, 3, 7, 8 |
| Forms styling | 8 |
| `filter` and `backdrop-filter` | 4, 5, 8 |
| Theming and dark mode | 6, 10 |
| Accessibility (focus, reduced motion) | 7, 8, 9, 10 |

## JavaScript Concepts You Touched

| Concept | Project |
|---------|---------|
| `querySelector`, `classList.toggle` | 2 |
| Event listeners and event delegation | 5 |
| `dataset` and filtering | 5 |
| `localStorage` | 6 |
| `matchMedia` | 6 |
| Form validation API | 8 |
| `IntersectionObserver` | 10 |

## Tips for getting the most out of this

- **Rebuild from memory.** After finishing a project, delete the CSS and recreate it without looking. This is the fastest way to learn.
- **Break it on purpose.** Remove `position: relative` from a parent or delete `preserve-3d` and observe what happens.
- **Study real sites.** Open DevTools on sites you admire and see how they solve the same problems.
- **Keep a snippets file** of tricks you want to reuse.
- **Reference docs:** [MDN Web Docs](https://developer.mozilla.org), [CSS-Tricks Flexbox guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/), [CSS-Tricks Grid guide](https://css-tricks.com/snippets/css/complete-guide-grid/).

## Where to go next

- Learn **Sass** or **PostCSS**, and modern CSS features like **container queries**, `:has()`, **cascade layers** (`@layer`), and **nesting**.
- Try **Tailwind CSS** once you understand plain CSS well, so you know what it generates for you.
- Move on to a JS framework (React, Vue, Svelte) with a strong CSS foundation behind you.

Happy building! 🚀
