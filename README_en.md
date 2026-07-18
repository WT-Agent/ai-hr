# NetTeng Infinite AI Enterprise HR Interview Questions

## Project Introduction

**NetTeng Infinite AI Enterprise HR Interview Questions** is an interactive interview kit and candidate evaluation micro-application for HR professionals, interviewers, and recruiting teams, inspired by the structured data style of nomads.com (Nomad List). Built on Vue 3 + Vite + Vanilla CSS, the application allows HR users to rate key candidate competency metrics (Technical Skills, Problem Solving & Resilience, Communication & Teamwork, Potential & Learning Ability, and Stability) and share position requirements. The AI then calculates its consensus rating, displays a comparison dashboard (HR Expectation vs. AI Benchmark), and generates a complete interview kit including a competency matrix, 5 behavioral question sets (with STAR evaluation standards and anti-bluffing follow-ups), and hiring recommendations. A responsive virtual Offer Pass Stamp is built-in to support mechanical stamp sound synthesis and candidate pass accumulation.

### Key Features
- **Competency Rating Sliders**: Interactive 1-to-5 star sliders for Technical Skills, Logic & Resilience, Teamwork & Communication, Potential & Learning Ability, and Stability.
- **Offer Pass Stamp**: Local sound synthesis using the Web Audio API (mechanical stamp click). Clicking the stamp increments the Offer Pass count and triggers a floating "Offer Pass +1" animation.
- **AI Consensus & Comparison Dashboard**: Automatic extraction of AI ratings from generated text to render side-by-side progress bars.
- **HR Kit Card Style History**: Local persistence of generated interview kits and stamp counts in the browser, displayed as beautiful HR cards with reload, single deletion, and clear all functions.
- **Floating Share Button**: A sleek glassmorphism share button at the top-right corner to invoke WeChat moments sharing guidance.
- **Side-by-Side QR Codes**: Parallel WeChat and Alipay payment codes in the donation section, and WeChat and DingTalk codes in the Contact Us modal.
- **Adaptive Modal Views**: Terms and privacy modals support vertical scrolling with max-height limits, while QR code modals adjust height automatically to prevent nested scrollbars.

## Quick Start

### 1. Clone the Project
```bash
git clone https://github.com/WT-Agent/ai-hr.git
cd ai-hr
```

### 2. Install Dependencies
This project enforces pnpm as the package manager:
```bash
pnpm install
```

### 3. Local Environment Configuration
Copy and configure the environment variables:
```bash
cp .env.example .env
```
Fill in your API key in the `.env` file:
- `DEEPSEEK_API_KEY`: Your DeepSeek developer API key (used for text generation tasks)

### 4. Development & Build
Start the local development server (with reverse proxy support to prevent API key leaks):
```bash
pnpm dev
```
Build static production assets (outputs to the `dist/` directory, suitable for Vercel, GitHub Pages, or CDN bucket hosting):
```bash
pnpm build
```

## Contact Us

If you have any questions, suggestions, or business cooperation proposals during use, feel free to contact us via WeChat or DingTalk.

## Donation Support

If this project helps you, feel free to support the author. Your support is the driving force for continuous maintenance and optimization.

| WeChat Pay | Alipay |
| :---: | :---: |
| <img src="asset/tenpay.png" width="180" alt="WeChat Pay" /> | <img src="asset/alipay.png" width="180" alt="Alipay" /> |

## License

This project is licensed under the MIT License.

Copyright (c) 2026. All rights reserved.
