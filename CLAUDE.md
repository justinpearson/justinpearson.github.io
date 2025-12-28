# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- **Start development server**: `bundle exec jekyll serve` (runs on http://localhost:4000)
- **Build site**: `bundle exec jekyll build` (outputs to `_site/`)
- **Install dependencies**: `bundle install`

## Important Reminders

- **When updating resume content**: After modifying `resume.md`, you MUST regenerate the PDF resume using Firefox (see `DOC/how-to-update-job-title-or-resume.md` for detailed steps). The PDF generation is very browser-specific and only works properly in Firefox.

## Architecture Overview

This is a Jekyll-based static website for Justin Pearson's professional portfolio. The site showcases academic research, industry experience, teaching, and programming projects.

### Content Structure

**Data-driven content**: Most dynamic content is defined in YAML files under `_data/`:
- `projects.yaml` - Programming and research projects with links, summaries, and metadata
- `teaching.yaml` - Teaching experiences and videos  
- `research.yaml` - Academic publications and research work
- `talks.yaml` - Conference presentations and tech talks
- `nav.yaml` - Site navigation structure
- `fun-videos.yaml` - Educational/entertainment video content

**Page structure**: Main content pages in root directory (`.md` files) use front-matter to specify layout and navigation context. Each page references data from `_data/` files to populate content dynamically.

**Layout system**: Single base layout (`_layouts/base-layout.html`) with modular includes (`_includes/`) for header, navigation, and footer. Navigation is data-driven from `nav.yaml`.

### Asset Organization

**Static assets** in `assets/` directory organized by content type:
- `projects/` - Project thumbnails and supporting files
- `research/` - Papers, presentations, and research-related media
- `teaching/` - Educational materials and documentation
- `industry/` - Professional experience assets

**Generated pages**: Some projects have dedicated sub-pages in `pages/` directory (auto-generated from Mathematica notebooks to HTML/PDF).

### Key Patterns

**Episode-based content**: Teaching and research sections use an episode-list pattern for displaying video content with thumbnails, durations, and metadata.

**External vs internal links**: Content distinguishes between internal site pages and external resources (videos, papers, code repositories).

**Multi-format content**: Many projects offer multiple formats (HTML, PDF, video) for the same content.

The site prioritizes clear content organization and academic presentation while maintaining modern web standards.