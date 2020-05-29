# 📝 Koj Handbook

This repository serves as the source of truth for our documentation, developer handbook, and general guidelines.

**⚠️ WARNING:**: This handbook is still in a very early stage. Expect breaking changes.

## Ideas

- We use Conventional Commits (without Gitmoji)
- Pull requests only, always, never push to master
- We have a generator for `nuxt-i18n` using directory structure (e.g., `path/to/file.json` -> `$i18n.t("path.to.file.key")`)
- Guide on how to update `status.joinkoj.com`
- We use a Staart API-based backend with a new frontend (not using Staart UI)
  - Map `organizations` to `homes`? Let users switch in advanced settings?
  - Use `@staart/payments` with Stripe Switzerland?
