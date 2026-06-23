# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- Ruby 3.3.6, Rails 8.1
- SQLite3 (development and test)
- Propshaft (asset pipeline), Importmap (no Node/bundler), Hotwire (Turbo + Stimulus)
- Solid Cache / Solid Queue / Solid Cable (DB-backed adapters — no Redis required)
- Deployed via Kamal (Docker)

## Commands

```bash
bin/rails server              # start dev server
bin/rails db:migrate          # run pending migrations
bin/rails db:test:prepare     # prepare test DB (run before tests after schema changes)
bin/rails test                # run all unit/integration tests
bin/rails test test/models/product_test.rb  # run a single test file
bin/rails test:system         # run system tests (Capybara + Selenium)
bin/rubocop                   # lint Ruby (omakase style via rubocop-rails-omakase)
bin/rubocop -a                # auto-correct offenses
bin/brakeman --no-pager       # static security scan
bin/bundler-audit             # check gems for known CVEs
bin/importmap audit           # check JS dependencies for vulnerabilities
```

## Architecture

This is a standard Rails MVC app, currently in early development. The only domain model so far is `Product` (`app/models/product.rb`) with a `name` string column.

Routes are empty (no root route yet) — the app boots but serves nothing at `/`.

JavaScript follows the Stimulus controller pattern: controllers live in `app/javascript/controllers/` and are auto-loaded via `app/javascript/controllers/index.js`.

CI (`.github/workflows/ci.yml`) runs four jobs: `scan_ruby` (Brakeman + bundler-audit), `scan_js` (importmap audit), `lint` (RuboCop), and `test` + `system-test`.
