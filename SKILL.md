# Angular Implementation Skill

## Purpose
This skill guides Claude to implement Angular changes the RIGHT way for THIS specific project —
not generically. Always discover the project's actual patterns before writing any code.

---

## PHASE 0 — Project Discovery (ALWAYS run this first, before any implementation)

Before writing a single line of code, run these commands and study the output:

```bash
# 1. Understand the full project structure
find src/app -type f -name "*.ts" | head -60

# 2. Detect Angular version and config
cat angular.json
cat package.json | grep -E '"@angular|"angular'

# 3. Detect component style (standalone vs NgModules)
grep -r "standalone: true" src/app --include="*.ts" -l | head -5
grep -r "NgModule" src/app --include="*.ts" -l | head -5

# 4. Detect UI libraries in use
grep -r "MatModule\|MaterialModule\|PrimeNG\|primeng\|mat-\|p-" src/app --include="*.ts" -l | head -5

# 5. Detect state management approach
grep -r "BehaviorSubject\|Store\|NgRx\|ngrx" src/app --include="*.ts" -l | head -5

# 6. Find existing feature examples to follow
ls src/app/features/ 2>/dev/null || ls src/app/ 

# 7. Find how services are structured
find src/app -name "*.service.ts" | head -5

# 8. Find routing structure
find src/app -name "*routing*" -o -name "*routes*" | head -5
```

**After discovery, form answers to:**
- Is this project using standalone components or NgModules or both?
- What UI library is being used (Angular Material / PrimeNG / custom)?
- How is state managed (BehaviorSubject / NgRx / simple services)?
- What is the folder structure convention for features?
- What naming conventions are used?

Log your findings:
```
[SKILL] Project uses: NgModules | Angular 15 | Angular Material | BehaviorSubject state
[SKILL] Feature structure: src/app/features/<name>/<name>.module.ts
[SKILL] Following pattern from: src/app/features/orders/
```

---

## PHASE 1 — Understand Scope Before Coding

Based on Jira story size, pick the right approach:

### Small story (1-3 files)
- Identify exact files to change
- Make targeted edits only
- Do not restructure or refactor anything else

### Medium story (4-10 files)
- Map out all files to create/modify before starting
- Tell the user the plan: "I will create X, modify Y, Z"
- Implement feature by feature, not file by file

### Large story (10+ files)
- Break into logical phases: models → services → components → routing → tests
- Implement and verify one phase before moving to next
- Check in with user if scope seems larger than expected:
  "This story touches 15+ files. Should I proceed or break it into sub-tasks?"

---

## PHASE 2 — Implementation Rules

### General
- **Follow existing patterns** — find a similar existing feature and mirror its structure exactly
- **Never introduce new patterns** unless the Jira story explicitly requires it
- **Never change** unrelated files, routing, or modules unless required by the story
- If unsure about a pattern, check 2-3 existing examples before deciding

### Component Style
**If project uses Standalone Components:**
```typescript
@Component({
  selector: 'app-example',
  standalone: true,
  imports: [CommonModule, RouterModule, /* only what's needed */],
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.scss']
})
export class ExampleComponent {}
```

**If project uses NgModules:**
- Create component inside the relevant feature module
- Declare it in the module's `declarations` array
- Export it only if used outside the module

### Services
- One service per domain concept (e.g. `UserService`, `OrderService`)
- Always use `providedIn: 'root'` unless feature-scoped
- HTTP calls only inside services — never in components
- Return `Observable<T>` — never subscribe inside a service

```typescript
@Injectable({ providedIn: 'root' })
export class ExampleService {
  private apiUrl = environment.apiUrl;
  
  constructor(private http: HttpClient) {}

  getItems(): Observable<Item[]> {
    return this.http.get<Item[]>(`${this.apiUrl}/items`);
  }
}
```

### State Management

**BehaviorSubject pattern (if project uses this):**
```typescript
private itemsSubject = new BehaviorSubject<Item[]>([]);
items$ = this.itemsSubject.asObservable();

updateItems(items: Item[]) {
  this.itemsSubject.next(items);
}
```

**In components — always use async pipe, avoid manual subscriptions:**
```html
<div *ngIf="items$ | async as items">...</div>
```

If manual subscription is unavoidable:
```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  this.service.items$
    .pipe(takeUntil(this.destroy$))
    .subscribe(items => this.items = items);
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

### UI Library Usage

**Angular Material — check what's already imported before adding:**
```bash
grep -r "MatModule\|MatButtonModule\|MatInputModule" src/app --include="*.ts" | head -10
```
- Import only specific Material modules needed, not `MatModule` wholesale
- Use existing shared Material module if the project has one

**PrimeNG — check existing usage:**
```bash
grep -r "primeng" src/app --include="*.ts" | head -10
```
- Match the PrimeNG version already in use
- Use `p-` prefixed components consistent with existing code

### Models / Interfaces
- Always define TypeScript interfaces for API responses
- Place in `models/` or `interfaces/` folder — follow project convention
- Never use `any` type

### Routing
- Check existing routing before adding new routes
- Lazy load feature modules if that's the existing pattern
- Never change existing route paths — only add new ones

---

## PHASE 3 — UI Look & Feel (ALWAYS run, even without mockups)

### Step 1 — Discover existing UI patterns first
```bash
# Find existing component HTML to understand layout patterns
find src/app -name "*.html" | head -10

# Check global styles and CSS variables
cat src/styles.scss 2>/dev/null || cat src/styles.css 2>/dev/null
find src -name "_variables*" -o -name "theme*" | grep -v node_modules | head -5

# Find an existing similar screen to mirror
# e.g. if adding a list page, find existing list pages
find src/app -name "*.html" | xargs grep -l "table\|mat-table\|p-table" | head -3
```

### Step 2 — Extract the UI DNA of the project
Before building any template, answer these from existing code:
- **Layout:** Sidebar + content? Tabs? Cards? Panels?
- **Tables:** `mat-table`, `p-table`, or custom? What column patterns exist?
- **Forms:** Reactive or template-driven? What validation display pattern?
- **Buttons:** What classes/colors for primary, secondary, danger actions?
- **Spacing:** What margin/padding classes or variables are used consistently?
- **Colors:** CSS variables? (`var(--primary-color)`) Check `styles.scss`
- **Loading states:** Spinners? Skeleton loaders? What existing pattern?
- **Error/empty states:** How does the app show errors or empty lists?

Log your findings:
```
[UI] Layout: sidebar + router-outlet content area
[UI] Tables: mat-table with mat-paginator + mat-sort
[UI] Forms: Reactive forms with mat-form-field + mat-error
[UI] Buttons: mat-raised-button color="primary"
[UI] Colors: CSS variables --primary, --accent from styles.scss
[UI] Mirroring pattern from: src/app/features/orders/orders-list.component.html
```

### Step 3 — Mirror, don't invent
- Find the **closest existing screen** to what you're building — use it as your template
- Copy its HTML structure, adapt the content — never start from a blank template
- Use the same CSS classes, same spacing, same component hierarchy
- Adding a list page → copy existing list page structure
- Adding a form → copy existing form structure
- **Never introduce new CSS classes** unless absolutely required — reuse existing ones
- If you must add new styles, add them to the component's `.scss` file only, not global styles

### Step 4 — Mockup usage (when MOCKUP_FOLDER is provided)
```bash
ls "$MOCKUP_FOLDER"
```
- Mockup shows **WHAT** to build — existing code shows **HOW** to build it
- Study mockup for: layout intent, form fields, table columns, button placement
- If mockup conflicts with existing UI patterns → follow existing patterns, note the difference to user
- Do not pixel-match the mockup — adapt it to the app's actual design system

---

## PHASE 4 — Quality Checklist (before handing back to main workflow)

Before calling `ng build`, verify:

- [ ] No `any` types introduced
- [ ] No `console.log` left in code (use proper logging service if project has one)
- [ ] All subscriptions are properly unsubscribed
- [ ] New components declared/imported in correct module or marked standalone
- [ ] No hardcoded URLs — use `environment.ts`
- [ ] Naming matches project conventions (check existing files)
- [ ] No unused imports

---

## PHASE 5 — Story Size Adaptation

| Signal | Action |
|--------|--------|
| Story has UI + API + state changes | Plan all three before starting any |
| Story mentions "refactor" | Confirm scope with user before touching anything |
| Story is vague | Ask user ONE clarifying question before implementing |
| Story affects shared/core modules | Extra caution — these affect the whole app |
| Story adds a new feature with routing | Check if lazy loading is needed |

---

## What This Skill Does NOT Do
- Does not replace reading `.claude/jira-context.md` — always read that first
- Does not override user's explicit instructions in DESCRIPTION
- Does not commit or create PRs — that's handled by the main workflow in CLAUDE.md
