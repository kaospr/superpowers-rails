---
name: rails-testing-conventions
description: Use when creating or modifying minitest tests — request, system, model, policy, or component tests
---

# Rails Testing Conventions

Tests verify behavior, not implementation. Real data, real objects, pristine output.

## Core Principles

1. **Never test mocked behavior** - If you mock it, you're not testing it
2. **No mocks in integration tests** - Request/system tests use real data. WebMock for external APIs only
3. **Pristine test output** - Capture and verify expected errors, don't let them pollute output
4. **All failures are your responsibility** - Even pre-existing. Never ignore failing tests
5. **Coverage cannot decrease** - Never delete a failing test, fix the root cause
6. **System tests test through the UI** - Every user action must use Capybara interactions, never direct model/controller calls

## Test Types

| Type | Location | Use For |
|------|----------|---------|
| Request | `test/requests/` | Single action (CRUD, redirects). Never test auth here |
| System | `test/system/` | Multi-step user flows through the UI. Every action via Capybara (clicks, fills, navigates) |
| Model | `test/models/` | Public interface behavior |
| Policy | `test/policies/` | ALL authorization tests belong here |
| Component | `test/views/components/` | Phlex component rendering |

## Factory Rules

- **Explicit attributes** - `create(:company_user, role: :mentor)` not `create(:user)`
- **Use traits** - `:published`, `:draft` for variations
- **`setup` method** - Use `setup` for shared test fixtures, create only what each test needs
- **Create in final state** - No `update!` in setup blocks

## Component Tests

Phlex components require **100% test coverage**. Use `Phlex::Testing::ViewHelper`:

```ruby
# test/views/components/status_badge_test.rb
require "test_helper"

class StatusBadgeTest < ActiveSupport::TestCase
  include Phlex::Testing::ViewHelper

  test "renders active status with green badge" do
    output = render(StatusBadge.new(status: :active))

    assert_includes output, "active"
    assert_includes output, "bg-green-100"
  end

  test "renders pending status with yellow badge" do
    output = render(StatusBadge.new(status: :pending))

    assert_includes output, "pending"
    assert_includes output, "bg-yellow-100"
  end
end
```

## Quick Reference

| Do | Don't |
|----|-------|
| Test real behavior | Test mocked behavior |
| WebMock for external APIs | Mock internal classes |
| Explicit factory attributes | Rely on factory defaults |
| `setup` for shared fixtures | Create objects inline everywhere |
| Capture expected errors | Let errors pollute output |
| Wait for elements | Use `sleep` |
| Assert content in index tests | Only check HTTP status |
| Click/fill/navigate in system tests | Replace user actions with model calls |
| `accept_confirm { click_button }` | Skip confirmation dialogs |

## Common Mistakes

1. **Testing mocks** - You're testing nothing
2. **Mocking policies** - Use real authorized users
3. **Auth tests in request tests** - Move to policy tests
4. **`sleep` in system tests** - Use Capybara's waiting
5. **Deleting failing tests** - Fix the root cause
6. **Bypassing UI in system tests** - `record.do_thing!` + `visit result_path` is not a system test. Click the button

## System Tests (Non-Negotiable)

System tests test the **full user journey through the browser**. If a user would click it, the test clicks it.

**NEVER replace UI interactions with direct model calls.** Setting up state via code (e.g., `record.reserve!(user)`) then visiting the result page is a request test pretending to be a system test. It skips the UI being tested.

| User does this | Test does this | NEVER this |
|----------------|----------------|------------|
| Clicks "Assign to me" | `click_button('Assign to me')` | `record.reserve!(user)` |
| Navigates via menu | `click_link('Reviews')` | `visit reviews_path` directly |
| Confirms a dialog | `accept_confirm { click_button(...) }` | Skip the dialog |
| Fills a form | `fill_in 'Name', with: 'value'` | `Record.create!(name: 'value')` |

**Factory setup** (creating the preconditions: users, records, associations) is fine. An **initial `visit`** to start the flow is fine. **Replacing user actions** (navigation, clicks, form fills that the test actually verifies) is not.

If a UI interaction is hard to automate (confirmation dialogs, JS-heavy flows), use Capybara's tools (`accept_confirm`, `accept_alert`, `execute_script`) — don't bypass the UI.

```ruby
# test/system/task_assignments_test.rb
require "application_system_test_case"

class TaskAssignmentsTest < ApplicationSystemTestCase
  setup do
    @mentor = create(:company_user, role: :mentor)
    @task = create(:task, company: @mentor.company)
    sign_in @mentor
  end

  test "mentor assigns task to themselves" do
    visit tasks_path
    click_link @task.title
    click_button "Assign to me"

    assert_text "Assigned to #{@mentor.name}"
  end
end
```

**Remember:** Tests verify behavior, not implementation.
