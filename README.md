# Job Search Tracker

A simple, self-contained web app for tracking your job search progress. No server required — runs entirely in your browser with IndexedDB persistence.

## Features

- **Track applications** — Log companies, roles, dates, and application stages
- **Company notes** — Add standalone notes to companies without tying them to activities
- **Weekly progress** — Monitor your weekly activity against a 3-task goal
- **Stage breakdown** — Visual dashboard showing applications by stage
- **Company history** — Click any company to see all related applications
- **Search & filter** — Find entries quickly by company, role, or stage
- **Dark mode** — Automatic or manual theme switching
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
2. Click **+ Log Activity** to log a new application or weekly task
3. Use the filters to search, sort, or filter by stage
4. Click a company row to see its full history and add notes
5. Use **Export** to backup your data, **Import** to restore

## Data Storage

Uses IndexedDB with three object stores:

- **companies** — Unique company records with optional notes
- **applications** — Individual application events (company, date, stage, role, notes)
- **weeklyTasks** — Standalone weekly tasks

Each company appears once in the main list, with all related applications viewable in the history modal.

## Weekly Progress

The app tracks a weekly goal of 3 qualifying activities (applications submitted, networking, interviews, or offers). The progress bar shows your status for the current week.

## Tech Stack

- Vanilla HTML/CSS/JavaScript — no build step required
- IndexedDB for browser storage
- [Phosphor Icons](https://phosphoricons.com/) for UI icons
- [Inter](https://rsms.me/inter/) font via Google Fonts

## Development

To modify:

1. Edit `index.html`
2. Refresh the browser

## License

MIT
