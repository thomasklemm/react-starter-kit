# AGENTS.md

This file provides guidance for agentic coding agents working in this repository.

## Project Overview

Rails 8 + Inertia.js + React starter kit using:

- **Backend**: Ruby on Rails 8.1, SQLite, Puma
- **Frontend**: React 19, TypeScript, Vite, Tailwind CSS v4, shadcn/ui
- **Auth**: authentication-zero based session management
- **Testing**: RSpec with FactoryBot and Capybara

## Build/Lint/Test Commands

### Ruby/Rails

```bash
# Run all tests
bin/rails db:test:prepare spec

# Run specific test file
bin/rspec spec/requests/todos_spec.rb

# Run specific test by line number
bin/rspec spec/requests/todos_spec.rb:42

# Run linting
bin/rubocop

# Auto-fix linting issues
bin/rubocop -A

# Security scans
bin/brakeman --no-pager
bin/bundler-audit
```

### JavaScript/TypeScript

```bash
# Check TypeScript types
npm run check

# Lint JS/TS files
npm run lint

# Auto-fix linting issues
npm run lint:fix

# Check formatting
npm run format

# Auto-fix formatting
npm run format:fix
```

### Development

```bash
# Start dev server
bin/dev

# Build Vite assets (runs automatically in CI/test)
bin/vite build
```

## Code Style Guidelines

### Ruby

- Use RuboCop Omakase style (inherited from rubocop-rails-omakase)
- Always include `# frozen_string_literal: true` at the top
- Use double quotes for strings
- 2-space indentation
- No spaces inside array brackets `[]` or hash braces `{}`
- Private methods indented one level deeper than `private` keyword
- Prefer `fetch` over `[]` for hash access when key might be missing

### TypeScript/React

- Use TypeScript for all new code
- Prefer `type` over `interface` for type definitions
- Use `function` declarations for components, not arrow functions
- Props interfaces defined in the same file as the component
- Import order: React/core libs → third-party → `@/` aliases → local
- Use `@/` path alias for imports from `app/frontend`
- No semicolons (Prettier config: `"semi": false`)
- Trailing commas on multi-line
- 80 character print width
- 2-space indentation

### Naming Conventions

- Ruby: `snake_case` for methods/variables, `PascalCase` for classes
- TypeScript: `camelCase` for variables/functions, `PascalCase` for types/components
- File names match their primary export (e.g., `button.tsx` exports `Button`)
- React components use default exports in page files

### Imports and Path Aliases

```typescript
// Good - using @/ alias
import { Button } from "@/components/ui/button"
import { cn } from "@/lib/utils"

// Bad - relative paths when @/ is available
import { Button } from "../../components/ui/button"
```

### Error Handling

- Rails controllers: Use `redirect_to` with `inertia: {errors: ...}` for form errors
- Always validate user input at the model level
- Use transactions for multi-step database operations
- Controllers should rescue from `ActiveRecord::RecordNotFound` implicitly (returns 404)

### Database/Models

- Use `scope` for query building
- Use `transaction` blocks for atomic operations
- Use `find_by_id` (not `find`) when the record might not exist
- Always validate presence before other validations

### Testing Patterns

```ruby
# Factory usage
let(:user) { create(:user) }

# Authentication in request specs
before { sign_in_as user }

# Common matchers
expect(response).to have_http_status(:success)
expect(response).to redirect_to(path)
expect { action }.to change(Model, :count).by(1)
```

### React/Inertia Patterns

- Pages go in `app/frontend/pages/` matching route structure
- Components go in `app/frontend/components/`
- UI primitives go in `app/frontend/components/ui/`
- Hooks go in `app/frontend/hooks/`
- Layouts go in `app/frontend/layouts/`
- Use `use-flash.tsx` hook for flash messages
- Use `Form` from `@inertiajs/react` for forms with automatic error handling

### Component Structure

```typescript
// UI component pattern (shadcn/ui style)
import { cn } from "@/lib/utils"

interface ButtonProps {
  variant?: "default" | "destructive"
}

function Button({ className, variant = "default" }: ButtonProps) {
  return <button className={cn("base-styles", className)} />
}

export { Button }
```

## Key Architectural Decisions

- **Inertia.js**: No API layer, controllers render Inertia responses directly
- **Current attributes**: Use `Current.user`, `Current.session` for global context
- **Authentication**: Session-based with `sign_in_as` helper in tests
- **Props serialization**: Use `as_json(only: [...])` in Inertia renders
- **Routes**: Access in frontend via auto-generated `window.route` (js-routes)

## CI/CD

GitHub Actions runs:

1. JS linting (prettier, eslint, tsc)
2. Ruby security scans (brakeman, bundler-audit)
3. Ruby linting (rubocop)
4. Full test suite (RSpec with Capybara)

Always run lint commands before pushing to origin.
