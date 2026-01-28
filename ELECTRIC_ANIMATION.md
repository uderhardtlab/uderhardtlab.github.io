# Electric Signal Animation - Implementation Guide

## Overview

This document describes how to implement the electric signal pulse animation for the hero section. The effect creates glowing particles that travel across the neural network image like electrical signals.

---

## Effect Description

- **Small glowing particles** travel across the screen
- **Very short trails** that fade quickly (6-12 points)
- **Occasional branching** like neural pathways
- **Purple-to-pink color range** (hue 270-310)
- **Mix-blend-mode: screen** so it glows over the image

---

## File Structure

```
uderhardtlab.github.io/
├── index.html
├── images/
│   └── hero.png      # Neural network background image
```

---

## HTML Structure

```html
<section class="hero" id="hero">
    <!-- Background image -->
    <div class="hero-bg">
        <img src="images/hero.png" alt="Neural network">
    </div>
    
    <!-- Electric signals canvas - MUST be above image -->
    <canvas id="electricCanvas"></canvas>
    
    <!-- Glow overlay -->
    <div class="hero-glow"></div>
    
    <!-- Fade edges -->
    <div class="hero-fade"></div>
    
    <!-- Content on top -->
    <div class="hero-content">
        <!-- Title, subtitle, etc -->
    </div>
</section>
```

---

## Required CSS

```css
.hero {
    position: relative;
    min-height: 100vh;
    overflow: hidden;
}

.hero-bg {
    position: absolute;
    inset: 0;
    z-index: 1;
}

.hero-bg img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    filter: saturate(1.1) brightness(0.85);
}

#electricCanvas {
    position: absolute;
    inset: 0;
    z-index: 2;
    pointer-events: none;
    mix-blend-mode: screen;  /* Important: makes glow blend with image */
}

.hero-glow {
    position: absolute;
    inset: 0;
    z-index: 3;
    pointer-events: none;
    background: 
        radial-gradient(ellipse at 30% 40%, rgba(168, 85, 247, 0.15) 0%, transparent 50%),
        radial-gradient(ellipse at 70% 60%, rgba(240, 171, 252, 0.1) 0%, transparent 40%);
    animation: glowPulse 4s ease-in-out infinite;
}

@keyframes glowPulse {
    0%, 100% { opacity: 0.6; }
    50% { opacity: 1; }
}

.hero-fade {
    position: absolute;
    inset: 0;
    z-index: 4;
    pointer-events: none;
    background: 
        linear-gradient(to bottom, #030108 0%, transparent 15%),
        linear-gradient(to top, #030108 0%, transparent 25%),
        radial-gradient(ellipse at center, transparent 40%, #030108 100%);
}

.hero-content {
    position: relative;
    z-index: 10;
    /* ... rest of content styling */
}
```

---

## JavaScript Implementation

```javascript
// ============================================
// ELECTRIC SIGNAL ANIMATION
// ============================================

const electricCanvas = document.getElementById('electricCanvas');
const electricCtx = electricCanvas.getContext('2d');

let electricWidth, electricHeight;
let signals = [];

// Resize canvas to match viewport
function resizeElectricCanvas() {
    electricWidth = electricCanvas.width = window.innerWidth;
    electricHeight = electricCanvas.height = window.innerHeight;
}

window.addEventListener('resize', resizeElectricCanvas);
resizeElectricCanvas();

// Signal Pulse Class
class SignalPulse {
    constructor() {
        this.reset();
    }
    
    reset() {
        // Start from random edge
        const edge = Math.floor(Math.random() * 4);
        switch(edge) {
            case 0: // Top
                this.x = Math.random() * electricWidth;
                this.y = 0;
                this.angle = Math.PI / 2 + (Math.random() - 0.5) * 0.8;
                break;
            case 1: // Right
                this.x = electricWidth;
                this.y = Math.random() * electricHeight;
                this.angle = Math.PI + (Math.random() - 0.5) * 0.8;
                break;
            case 2: // Bottom
                this.x = Math.random() * electricWidth;
                this.y = electricHeight;
                this.angle = -Math.PI / 2 + (Math.random() - 0.5) * 0.8;
                break;
            case 3: // Left
                this.x = 0;
                this.y = Math.random() * electricHeight;
                this.angle = (Math.random() - 0.5) * 0.8;
                break;
        }
        
        this.speed = 1.5 + Math.random() * 2;
        this.trail = [];
        this.maxTrail = 6 + Math.floor(Math.random() * 6); // Very short trail
        this.life = 1;
        this.hue = 270 + Math.random() * 40; // Purple to pink
        this.thickness = 0.5 + Math.random() * 1;
        this.branches = [];
        this.branchChance = 0.015;
    }
    
    update() {
        // Random walk
        this.angle += (Math.random() - 0.5) * 0.12;
        
        // Move
        this.x += Math.cos(this.angle) * this.speed;
        this.y += Math.sin(this.angle) * this.speed;
        
        // Add to trail
        this.trail.push({ x: this.x, y: this.y, alpha: 1 });
        
        // Limit trail length
        if (this.trail.length > this.maxTrail) {
            this.trail.shift();
        }
        
        // Fade trail points
        this.trail.forEach((p, i) => {
            p.alpha = (i / this.trail.length) * this.life;
        });
        
        // Maybe branch
        if (Math.random() < this.branchChance && this.branches.length < 2) {
            this.branches.push({
                x: this.x,
                y: this.y,
                angle: this.angle + (Math.random() - 0.5) * 1.5,
                trail: [],
                life: 0.6,
                speed: this.speed * 0.6
            });
        }
        
        // Update branches
        this.branches.forEach(branch => {
            branch.angle += (Math.random() - 0.5) * 0.15;
            branch.x += Math.cos(branch.angle) * branch.speed;
            branch.y += Math.sin(branch.angle) * branch.speed;
            branch.trail.push({ x: branch.x, y: branch.y, alpha: branch.life });
            if (branch.trail.length > 10) branch.trail.shift();
            branch.life -= 0.025;
        });
        
        this.branches = this.branches.filter(b => b.life > 0);
        
        // Check bounds
        if (this.x < -50 || this.x > electricWidth + 50 || 
            this.y < -50 || this.y > electricHeight + 50) {
            this.life -= 0.05;
        }
        
        return this.life > 0;
    }
    
    draw() {
        // Draw main trail
        if (this.trail.length > 1) {
            electricCtx.beginPath();
            electricCtx.moveTo(this.trail[0].x, this.trail[0].y);
            
            for (let i = 1; i < this.trail.length; i++) {
                electricCtx.lineTo(this.trail[i].x, this.trail[i].y);
            }
            
            electricCtx.strokeStyle = `hsla(${this.hue}, 80%, 70%, ${this.life * 0.7})`;
            electricCtx.lineWidth = this.thickness;
            electricCtx.lineCap = 'round';
            electricCtx.lineJoin = 'round';
            electricCtx.stroke();
            
            // Subtle glow
            electricCtx.strokeStyle = `hsla(${this.hue}, 100%, 80%, ${this.life * 0.2})`;
            electricCtx.lineWidth = this.thickness + 2;
            electricCtx.stroke();
        }
        
        // Draw head glow
        const head = this.trail[this.trail.length - 1];
        if (head) {
            const gradient = electricCtx.createRadialGradient(
                head.x, head.y, 0,
                head.x, head.y, 8
            );
            gradient.addColorStop(0, `hsla(${this.hue}, 100%, 90%, ${this.life * 0.9})`);
            gradient.addColorStop(0.5, `hsla(${this.hue}, 100%, 70%, ${this.life * 0.4})`);
            gradient.addColorStop(1, 'transparent');
            
            electricCtx.beginPath();
            electricCtx.arc(head.x, head.y, 8, 0, Math.PI * 2);
            electricCtx.fillStyle = gradient;
            electricCtx.fill();
        }
        
        // Draw branches
        this.branches.forEach(branch => {
            if (branch.trail.length > 1) {
                electricCtx.beginPath();
                electricCtx.moveTo(branch.trail[0].x, branch.trail[0].y);
                branch.trail.forEach(p => electricCtx.lineTo(p.x, p.y));
                electricCtx.strokeStyle = `hsla(${this.hue}, 80%, 70%, ${branch.life * 0.5})`;
                electricCtx.lineWidth = this.thickness * 0.5;
                electricCtx.stroke();
            }
        });
    }
}

// Animation loop
function animateElectric() {
    // Fast fade for short trails
    electricCtx.fillStyle = 'rgba(3, 1, 8, 0.4)';
    electricCtx.fillRect(0, 0, electricWidth, electricHeight);
    
    // Spawn new signals (keep 4-6 active)
    if (signals.length < 6 && Math.random() < 0.025) {
        signals.push(new SignalPulse());
    }
    
    // Update and draw
    signals = signals.filter(signal => {
        const alive = signal.update();
        if (alive) signal.draw();
        else signals.push(new SignalPulse()); // Respawn
        return alive;
    });
    
    requestAnimationFrame(animateElectric);
}

// Initialize
for (let i = 0; i < 4; i++) {
    signals.push(new SignalPulse());
}
animateElectric();
```

---

## Configurable Parameters

| Parameter | Current Value | Effect |
|-----------|---------------|--------|
| `maxTrail` | 6-12 | Trail length (lower = shorter) |
| `fade opacity` | 0.4 | How fast trails fade (higher = faster) |
| `speed` | 1.5-3.5 | Particle movement speed |
| `thickness` | 0.5-1.5 | Line thickness |
| `hue` | 270-310 | Color range (purple to pink) |
| `head glow radius` | 8px | Size of glowing head |
| `signal count` | 4-6 | Number of simultaneous signals |
| `branchChance` | 0.015 | Probability of branching |

---

## Adjustments

**Shorter trails:** Decrease `maxTrail` and increase fade opacity
```javascript
this.maxTrail = 4 + Math.floor(Math.random() * 4); // 4-8 points
ctx.fillStyle = 'rgba(3, 1, 8, 0.5)'; // faster fade
```

**More signals:** Increase spawn rate and max count
```javascript
if (signals.length < 10 && Math.random() < 0.04) {
```

**Different colors:** Change hue range
```javascript
this.hue = 180 + Math.random() * 40; // Cyan/teal
this.hue = 320 + Math.random() * 40; // Pink/magenta
```

**Faster movement:**
```javascript
this.speed = 2.5 + Math.random() * 3;
```

---

## Integration Checklist

- [ ] Add `<canvas id="electricCanvas"></canvas>` after hero background
- [ ] Add CSS for `#electricCanvas` with `mix-blend-mode: screen`
- [ ] Add SignalPulse class to JavaScript
- [ ] Add animation loop
- [ ] Initialize signals array
- [ ] Add resize handler for canvas

---

*Last updated: January 2025*
