---
name: gsap
description: Install and use GSAP (GreenSock Animation Platform) for high-performance JavaScript animations. Covers core utilities (Tween, Timeline), React integration (@gsap/react/useGSAP), Eases, and popular plugins like ScrollTrigger, ScrollSmoother, and Flip. Use when creating complex animations, scroll-based effects, UI transitions, or SVG animations, or when the user mentions GSAP, GreenSock, or ScrollTrigger.
---

# GSAP (GreenSock Animation Platform)

## Overview

GSAP is a robust, blazingly fast JavaScript animation library that lets you animate anything (CSS properties, SVG, canvas, React components). It's built for modern web performance and handles cross-browser inconsistencies out of the box.

**Core components**:
- **gsap.to(), gsap.from(), gsap.fromTo()**: Basic tweens.
- **Timeline**: `gsap.timeline()` to sequence multiple tweens.
- **Plugins**: Add extra capabilities (e.g., `ScrollTrigger` for scroll animations, `Flip` for FLIP animations).

---

## Installation

### Core Installation
```bash
npm install gsap
# or
pnpm add gsap
```

### React / Next.js Integration (Recommended)
GSAP provides a dedicated React hook to make cleanup and context management effortless.
```bash
npm install gsap @gsap/react
# or
pnpm add gsap @gsap/react
```

---

## Core Usage

### Basic Tweens
```javascript
import gsap from "gsap";

// Animate to a state
gsap.to(".box", { x: 100, duration: 1, ease: "power2.inOut" });

// Animate from a state
gsap.from(".box", { opacity: 0, y: 50, duration: 1, stagger: 0.1 });
```

### Timelines
Timelines are perfect for choreographing sequences.
```javascript
const tl = gsap.timeline({ repeat: -1, yoyo: true });

tl.to(".box1", { x: 100, duration: 1 })
  .to(".box2", { y: 50, duration: 0.5 }, "-=0.5") // Start 0.5s early
  .to(".box3", { rotation: 360 });
```

### Eases
Customize the feel of your animations.
- **Standard**: `"none"`, `"power1"` to `"power4"`, `"back"`, `"bounce"`, `"circ"`, `"elastic"`, `"expo"`, `"sine"`.
- Add `.in`, `.out`, or `.inOut` (e.g., `"power2.inOut"`).

---

## React Usage (`useGSAP`)

When using React (or Next.js), **always** use the `@gsap/react` package and its `useGSAP` hook instead of standard `useEffect`. It automatically handles cleanup, preventing memory leaks and strict-mode double-firings.

```tsx
import { useRef } from "react";
import gsap from "gsap";
import { useGSAP } from "@gsap/react";

gsap.registerPlugin(useGSAP);

export default function App() {
  const container = useRef<HTMLDivElement>(null);

  useGSAP(() => {
    // gsap code here...
    // All animations created here are automatically reverted on cleanup!
    gsap.to(".box", { x: 360, duration: 2 });
  }, { scope: container }); // Scope allows selecting elements only inside this container

  return (
    <div ref={container}>
      <div className="box">Box</div>
    </div>
  );
}
```

---

## Popular Plugins

Plugins extend GSAP's core functionality. They must be registered before use.

### ScrollTrigger
Animate elements on scroll, pin sections, or link animations to the scrollbar.
```javascript
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";

gsap.registerPlugin(ScrollTrigger);

gsap.to(".box", {
  scrollTrigger: {
    trigger: ".box",
    start: "top center", // When top of element hits center of viewport
    end: "bottom top", 
    scrub: true, // Tie animation strictly to scrollbar
    pin: true,
  },
  x: 500,
});
```

### Other Notable Plugins:
- **Flip**: Seamlessly animate elements changing state/layout.
- **Observer**: Normalize events (scroll, touch, pointer) for intent-based interactions.
- **Draggable**: Make elements draggable, spinnable, or tossable.
- **SplitText**: Split HTML text into characters, words, or lines for granular animation (Premium).
- **DrawSVG**: Intuitively animate SVG strokes (Premium).
- **MorphSVG**: Morph any SVG path into another (Premium).

---

## Common Patterns & Best Practices

- **Registration**: Always register plugins up front (e.g., `gsap.registerPlugin(ScrollTrigger, useGSAP)`).
- **React strict mode**: Rely on `useGSAP()` to avoid duplicate animations created by double invocation of `useEffect` in React 18 strict mode.
- **Transforms**: Use GSAP shortcuts (`x`, `y`, `rotation`, `scale`, `autoAlpha`) instead of standard `transform` or `opacity` strings for better performance and accuracy.
- **FOUC (Flash of Unstyled Content)**: For initial load animations, hide elements with CSS (`visibility: hidden`, not `display: none`) and use `gsap.from()` or `gsap.set(..., { autoAlpha: 0 })`.

---

## Additional Resources

- Official Docs: https://gsap.com/docs/v3/
- React Guide: https://gsap.com/resources/React
- ScrollTrigger: https://gsap.com/docs/v3/Plugins/ScrollTrigger
