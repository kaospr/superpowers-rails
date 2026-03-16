---
name: rails-view-conventions
description: Use when creating or modifying Rails views, partials, or Phlex components in app/views or app/views/components, including Turbo frames and form templates
---

# Rails View Conventions

Views are dumb templates. Presentation logic lives in Phlex components, domain logic lives in models.

## Core Principles

1. **Hotwire/Turbo** - Turbo frames for dynamic updates, never JSON APIs
2. **Phlex components for logic** - All presentation logic in components, NOT helpers
3. **No custom helpers** - `app/helpers/` is prohibited. Use Phlex components instead
4. **Dumb views** - No complex logic in ERB. Delegate to models or components
5. **Stimulus for JS** - All JavaScript through Stimulus controllers
6. **Don't duplicate model logic** - Delegate to model methods, don't reimplement

## Phlex Components

**Why?** Testability. Logic in views cannot be unit tested.

**Location:** `app/views/components/` — single-file Ruby classes inheriting from `Phlex::HTML`.

Use for: formatting, conditional rendering, computed display values — anything that would go in a helper.

**Models vs Phlex components:** Models answer domain questions ("what is the deadline?"). Phlex components answer presentation questions ("how do we display it?" — colors, icons, formatting).

**100% test coverage required** for all Phlex components.

```ruby
# app/views/components/status_badge.rb
class StatusBadge < Phlex::HTML
  def initialize(status:)
    @status = status
  end

  def view_template
    span(class: badge_classes) { @status.humanize }
  end

  private

  def badge_classes
    case @status.to_sym
    when :active
      "inline-flex items-center rounded-full bg-green-100 px-2.5 py-0.5 text-xs font-medium text-green-800"
    when :pending
      "inline-flex items-center rounded-full bg-yellow-100 px-2.5 py-0.5 text-xs font-medium text-yellow-800"
    else
      "inline-flex items-center rounded-full bg-gray-100 px-2.5 py-0.5 text-xs font-medium text-gray-800"
    end
  end
end
```

## Tailwind CSS v4

All styling via Tailwind CSS v4 utility classes:

- **No custom CSS files** for components
- **No inline `style` attributes** — use utility classes instead
- **CSS-first configuration** — use `@theme` directive, no `tailwind.config.js`

## Message Passing

Ask models, don't reach into their internals (see `rails-model-conventions` for the full pattern):

```erb
<%# WRONG - reaching into associations %>
<% if current_user.bookmarks.exists?(academy: academy) %>

<%# RIGHT - ask the model %>
<% if current_user.bookmarked?(academy) %>
```

## Forms

- `form_with` for all forms
- Turbo for submissions (no JSON APIs)
- Highlight errored fields inline

## Quick Reference

| Do | Don't |
|----|-------|
| `current_user.bookmarked?(item)` | `current_user.bookmarks.exists?(...)` |
| `task.requires_review?` | `task.review_criteria.any?` in component |
| Turbo frames for updates | JSON API calls |
| Stimulus for JS behavior | Inline JavaScript |
| Partials for simple markup | Duplicated markup |
| Phlex components for logic | Custom helpers |
| Tailwind utility classes | Custom CSS or inline styles |

## Common Mistakes

1. **Logic in views** - Move to Phlex components for testability
2. **Creating custom helpers** - Use Phlex components instead
3. **Reaching into associations** - Use model methods
4. **Inline JavaScript** - Use Stimulus controllers
5. **JSON API calls** - Use Turbo frames/streams
6. **Duplicating model logic** - Delegate to model methods, don't reimplement

**Remember:** Views render. Phlex components present. Models decide.
