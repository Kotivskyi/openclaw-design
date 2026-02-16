# A2UI Evaluation Prompts

**Source:** `vendor/a2ui/specification/0.9/eval/src/prompts.ts`

## Purpose

Test prompts for the A2UI (Agent-to-UI) canvas specification evaluation. These are used to test the agent's ability to generate structured UI messages conforming to the A2UI spec.

## Prompt Categories (37 test scenarios)

### Surface Management
- `deleteSurface` — Generate a deleteSurface JSON message

### Form & Input UIs
- `loginForm` — Login form with username, password, remember checkbox
- `surveyForm` — Multi-field survey form

### Data Display UIs
- `productGallery` — Product gallery with data binding
- `weatherForecast` — Weather forecast display
- `animalKingdomExplorer` — Hierarchy with tabs and nested cards
- `userProfileCard` — Profile card with avatar and stats

### Interactive UIs
- `dogBreedGenerator` — Dog breed info with interactive elements
- `todoApp` — Todo application with CRUD operations
- `calculatorApp` — Calculator with button grid

### Chart & Visualization
- `chartDisplay` — Data visualization charts
- `dashboardLayout` — Multi-panel dashboard

### And 25+ additional UI generation test scenarios

## Evaluation Flow

**Source:** `vendor/a2ui/specification/0.9/eval/src/evaluation_flow.ts`

Uses Genkit-based evaluation with pass/fail criteria based on issue severity in the generated A2UI JSON.
