# Job Search Tracker

A simple, self-contained web app for tracking your job search progress. No server required — runs entirely in your browser with localStorage persistence.

## Features

- **Track applications** — Log companies, roles, dates, and application stages
- **Weekly progress** — Monitor your weekly activity against a 3-task goal
- **Stage breakdown** — Visual dashboard showing applications by stage (Applied, Interviewing, Offer, etc.)
- **Company history** — Click any company to see all related applications
- **Search & filter** — Find entries quickly by company, role, or stage
- **Import/Export** — Backup your data as JSON or import from a previous export
- **Responsive** — Works on desktop and mobile

## Application Stages

- **Networking** — Initial outreach or connections
- **Applied** — Application submitted
- **Interviewing** — In the interview process
- **Offer** — Received an offer
- **No hire** — Rejected or withdrawn

## Usage

1. Open `index.html` in any modern browser
2. Click **+ Add Entry** to log a new application
3. Use the filters to search, sort, or filter by stage
4. Click a company row to see its full history
5. Use **Export** to backup your data, **Import** to restore

Your data is saved automatically in the browser's IndexedDB (larger capacity than localStorage).

## Weekly Progress

The app tracks a weekly goal of 3 qualifying activities (applications submitted, networking, interviews, or offers). The progress bar shows your status for the current week.

## Data Storage

Uses IndexedDB with three object stores:
- **companies** — unique company records (name, created date)
- **applications** — individual application events (company, date, stage, role, notes)
- **weeklyTasks** — standalone weekly tasks

Each company appears once in the main list, with all related applications viewable in the history modal.

## Development

No build step required. The app is a single HTML file with embedded CSS and JavaScript.

To modify:
- Edit `index.html`
- Refresh the browser

## License

MIT
