# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- Ruby 3.3.6, Rails 8.1
- SQLite3 (development and test)
- Propshaft (asset pipeline), Importmap (no Node/bundler), Hotwire (Turbo + Stimulus)
- Action Text (rich text via Trix) + Active Storage (file uploads, disk service in dev)
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

### Domain models

- `Product` — `name:string`, `featured_image` (Active Storage attachment), `description` (Action Text rich text). Validates name presence.
- `User` — `email_address:string`, `password_digest:string` (bcrypt via `has_secure_password`). Email is normalized (strip + downcase). Has many sessions.
- `Session` — belongs to User; stores `ip_address` and `user_agent`. Used to track login sessions via a signed cookie.
- `Current` (`ActiveSupport::CurrentAttributes`) — holds `Current.session`; delegates `.user` to it. This is the thread-local accessor for the authenticated user throughout a request.

### Authentication

Authentication lives in `app/controllers/concerns/authentication.rb` and is included in `ApplicationController`. All actions require authentication by default. Controllers opt out with `allow_unauthenticated_access`. Session state is carried in a signed permanent cookie (`session_id`). The `PasswordsController` + `PasswordsMailer` handle password reset via email tokens.

### Localization

`ApplicationController` wraps every action in `I18n.with_locale` via `around_action :switch_locale`. Locale is read from `params[:locale]` with fallback to the default. Locale files live in `config/locales/` (currently `en` and `es`).

### Routes

```
resource  :session                   # login/logout
resources :passwords, param: :token  # password reset
root      "products#home"
resources :products                  # full CRUD
```

### Testing

Integration tests can use `sign_in_as(user)` and `sign_out` helpers from `test/test_helpers/session_test_helper.rb`, which is auto-included into `ActionDispatch::IntegrationTest`.

CI (`.github/workflows/ci.yml`) runs: `scan_ruby` (Brakeman + bundler-audit), `scan_js` (importmap audit), `lint` (RuboCop), `test`, and `system-test`.
