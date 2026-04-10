# Priyank Vyas — Portfolio Website

A personal portfolio website for **Priyank Vyas**, Software Engineer 2 at Microsoft Azure.

🌐 **Live:** [priyankvyas31.github.io](https://priyankvyas31.github.io)

---

## What This Website Does

This is a single-page portfolio website that showcases:

- **About Me** — Bio, experience summary, and a profile photo
- **Company Logos** — Scrolling banner of 18+ companies I've interviewed with (Google, Microsoft, Amazon, Nvidia, etc.)
- **Technical Skills** — 6 skill cards covering Systems Programming, Networking, Storage, Embedded, Cloud, and Backend
- **Work Experience** — Timeline of Microsoft, Cisco, and TCS roles with descriptions and skill tags
- **Education** — Academic background with grades
- **Mentorship Sessions** — 6 bookable session cards linked to Topmate (Career Guidance, Mock Interview, Resume Review, etc.)
- **Testimonials** — Scrolling reviews from mentees
- **YouTube Podcast** — Embedded podcast video
- **Contact** — LinkedIn, Email, Topmate, and GitHub links
- **Photo Gallery** — Personal photos from Microsoft office and other locations

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | HTML5, CSS3, Vanilla JavaScript |
| Fonts | Google Fonts (Inter) |
| Hosting | GitHub Pages |
| Version Control | Git + GitHub |
| Booking/Payment | Topmate.io (external) |
| Icons | Inline SVGs |
| Images | Local JPEGs (personal photos + company logos) |

---

## Project Structure

```
portfolio/
├── index.html              # Main (and only) HTML file — contains all CSS and JS
├── README.md               # This file
├── docs/
│   ├── HLD.md              # High-Level Design document
│   └── LLD.md              # Low-Level Design document
└── images/
    ├── personal_photo/     # 7 personal photos (JPEG)
    │   ├── priyank_1.jpeg
    │   ├── priyank_3.jpeg
    │   ├── priyank_5.jpeg
    │   ├── priyank_6.jpeg
    │   ├── priyank_7.jpeg
    │   ├── priyank_8.jpeg
    │   └── priyank_9.jpeg
    └── comanies_logo/      # 18 company logo images (JPG)
        ├── google_logo.jpg
        ├── microsoft_logo.jpg
        ├── amazon_logo.jpg
        └── ... (15 more)
```

---

## How to Run Locally

1. Clone the repo:
   ```bash
   git clone https://github.com/PriyankVyas31/PriyankVyas31.github.io.git
   ```
2. Open `index.html` in any browser — that's it. No build step, no dependencies.

---

## How to Deploy

This repo uses **GitHub Pages** with the `main` branch. Any push to `main` auto-deploys to `https://priyankvyas31.github.io/`.

```bash
git add -A
git commit -m "Your message"
git push origin main
```
Site updates within 1-2 minutes.

---

## Key Features

### Responsive Design
- Works on desktop, tablet, and mobile
- Hamburger menu on small screens
- CSS breakpoints at 768px and 480px

### Animations
- Scroll-triggered fade-in using Intersection Observer API
- Infinite scrolling marquees for company logos and testimonials
- Hover effects on cards with border glow and lift

### Performance
- No external CSS/JS frameworks (no React, no Bootstrap)
- Single HTML file — minimal HTTP requests
- Lazy-loaded images
- Google Fonts preconnect

---

## How to Update Content

### Add a new testimonial
Search for `testimonial-card` in `index.html` and duplicate an existing card block. The testimonials are inside a marquee — make sure to duplicate in both the original and the "seamless loop" copy.

### Add a new session
Search for `session-card` and duplicate an existing `<a>` block. Update the title, subtitle, price, and duration.

### Change photos
Replace files in `images/personal_photo/` and update the `src` attributes in `index.html`.

### Change company logos
Replace/add files in `images/comanies_logo/` and update the marquee HTML.

---

## Author

**Priyank Vyas**
- Microsoft Azure — Software Engineer 2
- [LinkedIn](https://www.linkedin.com/in/priyank-vyas-31march1999/)
- [Topmate](https://topmate.io/priyank_vyas)
- [GitHub](https://github.com/PriyankVyas31)
- Email: priyankvyas001@gmail.com
