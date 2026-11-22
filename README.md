# Bordplan – seating planner

A free seating planner for classrooms and events. It runs fully in the browser: no sign-up, no cookies, no tracking. Available in Danish and English.

**Try it:** https://bordplan.kskspace.dk

## Features

**Class mode** (`?class` / `?klasse`)
- Guided setup: students → level → seat needs → relations → rotation → summary
- Pairs students by neighbouring levels (high with middle, middle with low – never high with low), as in the Danish SYKL research on student pairs
- "Near the board" seats for students who need them
- "Works well together" and "keep apart" rules
- Remembers past plans and gives new partners on each rotation (every 1, 2 or 4 weeks)

**Event mode** (`?event` / `?fest`)
- Long and round tables, end seats, and a floor plan where tables can be dragged, rotated and joined (L, T, U and E shapes)
- Guests, with children seated next to or at the same table as their parents
- Seating rules, and seats that can be locked or swapped

**Both modes**
- Automatic seating, with an explanation for each seat
- Print the plan, the list and place cards; export as CSV or JSON; share by link
- Dark mode (default) and light mode; Lucide icons

## Privacy

- No cookies, no analytics, and nothing loaded from other websites.
- Plans are saved only in the browser's local storage on your device and are never sent to a server.
- Share links keep the plan after the `#`, a part of the link that browsers never send to the server.

## Run it yourself

The whole app is a single `index.html` file with no build step and no backend.

- Open `index.html` directly in a browser, or
- host it on any static web host (GitHub Pages, Netlify, nginx, …).

## License

[MIT](LICENSE) © 2026 [KSKIT](https://github.com/ksk-itdk). You are free to use, copy and change it, as long as you keep the copyright notice and credit KSKIT.

Icons: [Lucide](https://lucide.dev) (ISC license).
