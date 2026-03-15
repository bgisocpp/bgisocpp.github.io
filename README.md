# bgisocpp.github.io

Official webpage of the Bulgarian C++ Working Group at the Bulgarian Standardization Institute (BDS), participating in ISO/IEC JTC 1/SC 22/WG 21.

Built with [Jekyll](https://jekyllrb.com/) (minima theme) and [Decap CMS](https://decapcms.org/).

## Local Development

### Prerequisites

Install Ruby, the Ruby development headers, and Bundler:

```bash
# Debian/Ubuntu
sudo apt-get install -y ruby ruby-dev build-essential npm
```

### Running the site

```bash
bundle config set --local path 'vendor/bundle'
bundle install
bundle exec jekyll serve
```

The site will be available at `http://localhost:4000`.

## Editing Content

### Locally via Decap CMS

Uncomment `local_backend: true` in `admin/config.yml`, then in a separate terminal:

```bash
npx decap-server
```

Open `http://localhost:4000/admin/` to create and edit meetings and pages using the visual editor. Click **Publish** to save — in local mode this writes the file directly to your filesystem (it does not push to GitHub). Jekyll's live reload picks it up and rebuilds automatically. Remember to re-comment `local_backend: true` before pushing.

### Locally via Markdown files

Edit the Markdown files directly:

- **Meetings** — create a new file in `_meetings/` named `YYYY-MM-DD-title.md` with front matter:

  ```markdown
  ---
  title: "Meeting Title"
  date: 2026-04-15T18:00:00+03:00
  location: "Sofia"
  location_link: "https://maps.google.com/..."
  agenda: "Topics to discuss..."
  speakers: "Speaker name and bio..."
  ---
  ```

- **About page** — edit `about.md`

### Via Decap CMS (production)

1. Go to `https://bgisocpp.github.io/admin/`
2. Click **Login with GitHub** and authorize the OAuth app
3. Use the sidebar to navigate to **Meetings** or **Pages**
4. Create or edit entries using the visual editor
5. Click **Publish** — Decap CMS commits directly to the `master` branch and GitHub Pages rebuilds automatically
