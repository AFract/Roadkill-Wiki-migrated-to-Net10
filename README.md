[![CI](https://github.com/AFract/roadkill-fork/actions/workflows/ci.yml/badge.svg)](https://github.com/AFract/roadkill-fork/actions/workflows/ci.yml)

# Roadkill Wiki in .Net 10

This repository is a fork of [Roadkill wiki](https://github.com/roadkillwiki/roadkill), a nice Wiki project in .Net, licensed under Microsoft Public License (MS-PL) (see [LICENSE](LICENSE.md)). However the legacy (now Archived) repository was not maintained since .Net 4 framework.
I'm not related to the original author in any way.

On this repository, you'll find **a version of it converted to .NET 10 with several fixes and new features** (listed below). 
It was also proposed as a [pull request](https://github.com/roadkillwiki/roadkill/pull/128) to the original author, who have merged it in a separate branch before he archived the original repository: https://github.com/roadkillwiki/roadkill/tree/dotnet-10?tab=readme-ov-file

### Current status (this fork)

This fork has been migrated to **.NET 10 / ASP.NET Core MVC** (details in [MIGRATION.md](MIGRATION.md)).

All (eventual) future developments will happen in this repository. Issues and Pull Requests are warmly welcome, however there is no commitment to take them into account due to lack of time or other reasons.

**Upgrading a 2.x installation:** [English guide](docs/upgrade-v2-to-v3.md) · [Guide en français](docs/migration-v2-vers-v3.md)
Please see dropped features before to avoid surprises during update.

#### Changes (migration)

* Runs on .NET 10 / ASP.NET Core MVC (on IIS with the ASP.NET Core Hosting Bundle, or any other ASP.NET Core host; tested on Linux with Kestrel).
* Markdown parser replaced by Markdig. The existing Roadkill Markdown syntax (`[[[code lang=xx|...]]]`, image sizes, `#Title#`...) still works.
* Data access: SQL Server (2022 or later) and Postgres through Dapper. The database schema is unchanged from 2.x.
* Settings are in `appsettings.json` (`Roadkill` section and `ConnectionStrings:Roadkill`) instead of `web.config` / `Roadkill.config`.
  `tools/ConvertWebConfig.cs` (or `.linq` for LINQPad) converts an existing `web.config`. Logging is still configured in `App_Data/NLog.config`.
* The Lucene search index format changed (Lucene.Net 4.8): the index must be rebuilt after upgrading.
* Themes: `@Html.Action(...)` (child actions) no longer exists in ASP.NET Core; custom `Theme.cshtml` files need a few lines changed (see the upgrade guide).
* MathJax 3 is served by Roadkill (it used the MathJax 2 CDN): no CDN is used any more.
* `Assets/Scripts/roadkill.js` is generated on build (it was done by Grunt). The SCSS files are no longer compiled: change the
  committed `roadkill.css` / `roadkill.installer.css` files instead.

#### Features

* **GitHub Flavored Markdown**: pipe tables, strikethrough, task lists, autolinks, footnotes, fenced code blocks
  (highlighted by the Syntax Highlighter plugin).
* **Mermaid** diagrams through a new plugin (disabled by default), with the Mermaid dark theme on dark site themes.
* New **Pages by tag** page (`/pages/alltagswithpages`, menu token `%tagswithpages%`): each tag with the list of its pages,
  foldable with one click, sorted by number of pages or alphabetically, with expand all / collapse all buttons, and the
  pages without tags at the bottom (to help tagging them). A search field (client side, case and accent insensitive, 2
  characters minimum) expands the tags of the pages whose title matches, and each page shows its number of tags (the
  tags in a tooltip).
  It is in the default menu of new sites; on an upgraded site, add `* %tagswithpages%` in Site settings > Menu.
* The markup help (the "i" of the edit page) documents the GFM syntax, all the link and image forms, and the plugins.

#### Fixes (bugs of Roadkill 2.x)

* Internal links to a page whose title contains a "-" or punctuation (e.g. "Pre-release notes", "C# tips") offered to
  create the page: they now also match the title as it is in the page url (`[text](Pre-release-notes)`, `[text](c-tips)`).
* The edit page preview didn't run the MathJax, Mermaid and syntax highlighter plugins.
* Saving the site settings could fail (InvalidOperationException about the "DatabaseName" drop down list).
* The markup help list and link examples didn't work (a space is needed after "-" or "1.", no spaces in link urls).
* MongoDB: `GetUserByEmail` ignored the "activated" filter, and `GetPageByTitle` was case sensitive.
* The attachments export zip name had a wrong date format.
* Searching from the site settings pages went to a wrong url (404): the themes' search form and logo link now set `area = ""`.
* **Security** (private sites, i.e. not public): the attachments were served without a login, `/wiki/help:about` showed a
  wiki page without a login, and the pages and attachments were sent with `Cache-Control: public`. A test now checks that
  every controller requires a login on a private site (or editor/admin rights, or an API key).
* **Security**: an attachment url could point outside the attachments folder (e.g. `../`); only the files of that folder are served.

#### Removed (no longer supported)

* Databases: **MySQL**, **SQLite** and **SQL Server CE** (and the LightSpeed ORM they relied on).
* **Windows / Active Directory authentication** (and the matching installer step). Only the built-in forms (cookie) authentication remains.
* **reCAPTCHA v1** (Google's v1 API is gone): reCAPTCHA v2 keys are required.
* The XML configuration files (`web.config` Roadkill section, `Roadkill.config`, `connectionStrings.config`).
* Binary compatibility of 2.x plugins: custom plugins must be rebuilt for .NET 10.
* The .NET Framework build tooling: AppVeyor / Travis CI, `build/*.ps1` and Mono scripts, Web Platform Installer package (`lib/WebPI`), XML configs in `lib/Configs`.
  CI now runs on **GitHub Actions** (`.github/workflows/ci.yml`: build, unit tests, SQL Server and Postgres integration tests).
* The **MediaWiki** markup type (unsupported since 2.x, choosing it broke the pages): a site still set to it is rendered as Markdown.
* Unused files: the SQL CE / LocalDB databases (`App_Data/roadkill.mdf`, `*.sdf`, `lib/Test-databases/Upgrade`), the Windows
  auth testing notes, the unused `TestUserService`, `PreviewPage` and `LeftMenu` views, and the Grunt files (`gruntfile.js`, `package.json`).

#### Kept but untested

* **MongoDB**: its repositories pass the integration tests, but a wiki running on MongoDB hasn't been tried.
* Azure Blob attachments storage, SMTP sending, reCAPTCHA v2, hosting on IIS.

#### Build and test

* Build and run: `dotnet run --project src/Roadkill.Web` (solution file: `Roadkill.slnx`).
* Tests: `dotnet test src/Roadkill.Tests`. The integration tests start SQL Server, Postgres and MongoDB themselves in throwaway
  containers ([Testcontainers](https://dotnet.testcontainers.org/)): Docker or a Podman machine must be running, nothing else to
  install or configure. The databases to test (and optionally the container engine endpoint) are set in
  `src/Roadkill.Tests/appsettings.json`. If the container engine isn't available, the integration tests fail.

