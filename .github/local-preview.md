# Local Preview Notes

This repo can now be previewed locally without relying on the broken `rbenv` Ruby install.

## What changed

- Added a root Jekyll config at `/_config.yml`.
- Added active Jekyll source directories at `/_posts` and `/_layouts`.
- Kept the existing `posts/` folder untouched, but Jekyll now builds from the root directories above.
- Added `bin/dev` to start local preview.
- Added `docker-compose.yml` for a Docker-based workflow once Docker is installed.
- Added `.gitignore` entries for local build artifacts and vendored gems.
- Set `.ruby-version` to `system` so local commands stop selecting the broken Ruby 3.1.6 install.
- Normalized `Gemfile.lock` so Bundler 2.3.27 works with the local preview path.

## How to run

From the repo root:

```sh
./bin/dev
```

That script does this:

- Uses `docker compose up` if Docker is installed.
- Otherwise uses macOS system Ruby plus Bundler 2.3.27.
- Installs gems into `vendor/bundle`.
- Starts Jekyll at `http://127.0.0.1:4000`.

## Active edit locations

Edit these paths for the actual Jekyll site:

- `/_posts`
- `/_layouts`
- `/_config.yml`
- `/index.html`

The older files under `/posts` are no longer the active Jekyll source.
