# Nextcloud Documentation - AI Agent Guide

## Architecture Overview

This is a **Sphinx-based documentation repository** containing three independent manuals:
- **`admin_manual/`** - Administration guide
- **`user_manual/`** - End-user guide (39 language translations)
- **`developer_manual/`** - Developer guide (includes OpenAPI spec integration)

Each manual is built independently but shares theme/assets via `_shared_assets/` and global configuration from root `conf.py`.

**Critical constraint**: Pages are permanent API contracts - once published, URLs cannot move (external sites link to them). Use redirects (`sphinx_reredirects`) only when absolutely necessary.

## Developer Workflows

### Building Documentation

```bash
# Install dependencies (one-time setup)
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Build all manuals (HTML + PDF)
make all

# Build specific manual
cd admin_manual && make html        # Admin manual
cd user_manual && make html          # All 39 languages (slow)
cd user_manual && make html-lang-en  # English only (fast)
cd developer_manual && make html     # Requires OpenAPI spec

# Live preview with auto-reload
pip install sphinx-autobuild
cd user_manual && make SPHINXBUILD=sphinx-autobuild html
# Open http://127.0.0.1:8000
```

**Build outputs**: `{manual}/_build/html/com/` (use `/com` variant for consistency)

### Developer Manual Special Requirements

The developer manual depends on auto-generated OpenAPI spec:

```bash
make openapi-spec  # Fetches nextcloud/server repo, extracts openapi.json
```

This runs automatically when building developer manual via root `Makefile`, but manual builds inside `developer_manual/` directory will fail without it.

### Testing Changes

CI runs on every PR (`.github/workflows/sphinxbuild.yml`):
- Builds all three manuals independently
- Strict warnings mode (`-W` flag) - warnings = build failures
- Upload artifacts for preview

**Before committing**: Build locally to catch warnings (Sphinx won't show them in live preview).

## RST Conventions (Strict Style Guide)

### Heading Hierarchy (3 levels max)

```rst
==============
Page title, h1
==============

Section heading, h2
-------------------

Subsection heading, h3
^^^^^^^^^^^^^^^^^^^^^^
```

**Rules**:
- Underline must be ≥ title length
- Use sentence case (not Title Case)
- Avoid more than 3 levels (restructure content instead)

### File/Directory Naming

- **Files**: `lowercase_with_underscores.rst`
- **Images**: `lowercase-with-hyphens.png` (PNG only)
- **Image location**: Local `images/` subdirectory within each section

### Common Patterns

**GUI elements** (use exact literal names):
```rst
**Username** field
**Groups** dropdown menu
**Create** button
```

**Code/commands** (double backticks):
```rst
``sudo -E -u www-data php occ files:scan --help``
``config.php``
```

**Example URLs** (always use):
```rst
``https://example.com``
```

**Images with captions**:
```rst
.. figure:: images/screenshot.png
   :alt: Brief descriptive alt text

   *Figure 1: Descriptive caption.*
```

**Cross-references**:
```rst
:doc:`other_document`              # Link to sibling document
:ref:`label-name`                  # Link to labeled section
:download:`files/example.pdf`      # Download link
```

**Common admonitions**:
```rst
.. note::
   Informational message

.. warning::
   Potentially dangerous

.. tip::
   Helpful hint

.. TODO ON RELEASE:
   Update version numbers when releasing (search for this before releases)
```

### Line Length

**Wrap all lines at 80 characters** (`README.rst:20`) - this is enforced by project convention.

## Configuration Inheritance Pattern

Each manual's `conf.py` extends root config:

```python
from conf import *  # Import globals

# Extend (don't replace) lists
extensions += [
    'sphinx.ext.todo',
    'rst2pdf.pdfbuilder',
]

# Override specific settings
project = u'Nextcloud Admin Manual'
root_doc = 'contents'  # or 'index' for developer_manual
html_context['versions'] = generateVersionsDocs('admin_manual')
html_baseurl = "https://docs.nextcloud.com/server/stable/admin_manual/"
```

**Key differences**:

| Setting | Admin | User | Developer |
|---------|-------|------|-----------|
| `root_doc` | contents | contents | index |
| Translations | No | Yes (39 langs) | No |
| Special extensions | sidebar_links | - | phpdomain, reredirects |

## Auto-Generated Content (DO NOT EDIT)

### System Config Documentation

`admin_manual/configuration_server/config_sample_php_parameters.rst` is auto-generated from `nextcloud/server` repository's `config/config.sample.php`.

**Edit process**:
1. Make changes in `nextcloud/server` repo
2. Automated job generates updated RST (weekly or on-demand)
3. Commit message: "Generate system config documentation from config.sample.php"

**Never edit this file directly** - changes will be overwritten.

## Translation Workflow (User Manual Only)

User manual supports 39 languages via Transifex integration:

1. **Source changes** pushed to `master` → triggers `.github/workflows/generate_catalog_templates.yml`
2. Workflow runs `make gettext` → generates `.pot` files in `user_manual/locale/source/`
3. Creates PR → auto-approved and merged
4. Transifex fetches updated `.pot` files automatically
5. Translators work on Transifex platform
6. Translated `.po` files synced back to `user_manual/locale/{lang}/LC_MESSAGES/`

**Build process**:
```bash
cd user_manual
../build/change_file_extension.sh  # Converts .pot → .po
make html-lang-de                  # Build German version
```

**Merged asset handling**: User manual `make html` builds all languages, then consolidates `_static/` and `_images/` directories to avoid duplication (see `merge-folders` target).

## Common Editing Tasks

### Adding a New Page

1. Create `new_page.rst` in appropriate manual subdirectory
2. Add to parent directory's `index.rst` or `contents.rst`:

```rst
.. toctree::
   :maxdepth: 2
   :caption: Section Name

   existing_page
   new_page        # Add here
   another_page
```

3. Build locally to verify
4. **Remember**: This URL is permanent once published

### Updating Screenshots

1. Narrow browser window for compact screenshots (think square, not wide)
2. Save as `.png` in local `images/` directory
3. Use descriptive filename: `sharing-public-link.png`
4. Add with figure + caption:

```rst
.. figure:: images/sharing-public-link.png
   :alt: Public link sharing dialog

   *Figure 1: Creating a public share link.*
```

### Adding Version-Specific Content

Use `TODO ON RELEASE` comments for content needing version updates:

```rst
.. TODO ON RELEASE:
   Update supported PHP versions in table below

Nextcloud |version| supports PHP 8.2, 8.3, and 8.4.
```

Search for `TODO ON RELEASE` before major releases to find update locations.

### Working Across Branches

- `master` → latest development (docs.nextcloud.com/server/latest/)
- `stable31` → current stable release (docs.nextcloud.com/server/stable/)
- `stable30`, `stable29`, etc. → older supported versions

**Version mapping** in `conf.py`:
```python
version_start = 29   # Oldest supported version
version_stable = 31  # Current stable (maps to /stable/ URL)
```

When targeting specific versions, work in appropriate `stableXX` branch and backport changes as needed.

## Key Files Reference

| Task | Location |
|------|----------|
| Build all manuals | `make all` (root) |
| Global config | `conf.py` (root) |
| Manual configs | `{admin,user,developer}_manual/conf.py` |
| Style guide | `style_guide.rst` |
| Build scripts | `build/get-server-sources.sh`, `build/change_file_extension.sh` |
| Auto-generated | `admin_manual/configuration_server/config_sample_php_parameters.rst` |
| CI workflows | `.github/workflows/sphinxbuild.yml` |
| Shared theme | `_shared_assets/static/custom.css` |
| GitHub edit link | `_ext/edit_on_github.py` |

## Debugging Build Issues

**Warnings/errors during build**:
```bash
# See full error output
cd admin_manual && make html

# Common issues:
# - Missing image: Check path relative to .rst file
# - Duplicate label: Search for duplicate `.. _label:` definitions
# - Toctree reference: Ensure referenced file exists (no .rst extension in toctree)
```

**Developer manual won't build**:
```bash
# Missing OpenAPI spec
make openapi-spec  # Run from repository root
```

**Translation build fails**:
```bash
cd user_manual
../build/change_file_extension.sh  # Convert .pot → .po first
rm -rf _build                      # Clean build directory
make html-lang-de                  # Rebuild specific language
```

## Git Workflow

Standard PR workflow with DCO sign-off required:

```bash
git commit -s -m "Fix typo in admin manual installation guide"
```

Commit message must include:
```
Signed-off-by: Your Name <your.email@example.com>
```

Email must match GitHub profile email (or use `github.username@users.noreply.github.com`).
