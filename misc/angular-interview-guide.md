# Angular Interview Preparation Guide

A comprehensive guide covering fundamental concepts through advanced topics for Angular interviews.

---

## Part 1: Core Concepts

### What is Angular?

Angular is a TypeScript-based open-source framework developed by Google for building single-page applications (SPAs). It provides a complete solution including routing, forms, HTTP client, and testing utilities.

### Angular vs AngularJS vs React vs Vue

| Feature | Angular | AngularJS | React | Vue |
|---------|---------|-----------|-------|-----|
| Language | TypeScript | JavaScript | JavaScript/JSX | JavaScript |
| Architecture | Component-based | MVC | Component-based | Component-based |
| Data Binding | Two-way | Two-way | One-way | Two-way |
| DOM | Real DOM | Real DOM | Virtual DOM | Virtual DOM |
| Learning Curve | Steep | Moderate | Moderate | Easy |
| Bundle Size | Larger | Smaller | Smaller | Smallest |

### Angular Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      NgModule                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Component  │  │  Component  │  │  Component  │     │
│  │  ┌───────┐  │  │  ┌───────┐  │  │  ┌───────┐  │     │
│  │  │Template│  │  │  │Template│  │  │  │Template│  │     │
│  │  └───────┘  │  │  └───────┘  │  │  └───────┘  │     │
│  │  ┌───────┐  │  │  ┌───────┐  │  │  ┌───────┐  │     │
│  │  │ Class │  │  │  │ Class │  │  │  │ Class │  │     │
│  │  └───────┘  │  │  └───────┘  │  │  └───────┘  │     │
│  │  ┌───────┐  │  │  ┌───────┐  │  │  ┌───────┐  │     │
│  │  │ Styles│  │  │  │ Styles│  │  │  │ Styles│  │     │
│  │  └───────┘  │  │  └───────┘  │  │  └───────┘  │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │  Services   │  │  Directives │  │    Pipes    │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

### Key Building Blocks

| Building Block | Purpose |
|----------------|---------|
| Modules | Organize application into cohesive blocks |
| Components | UI building blocks with template, logic, and styles |
| Templates | HTML with Angular syntax |
| Directives | Add behavior to DOM elements |
| Services | Reusable business logic |
| Dependency Injection | Provide dependencies to classes |
| Pipes | Transform data in templates |
| Guards | Control route access |
| Interceptors | Intercept HTTP requests/responses |

---

## Part 2: Components

### Component Structure

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-user-card',
  templateUrl: './user-card.component.html',
  styleUrls: ['./user-card.component.scss'],
  // Or inline:
  // template: `<div>{{ user.name }}</div>`,
  // styles: [`div { color: blue; }`]
})
export class UserCardComponent {
  @Input() user!: User;
  @Output() selected = new EventEmitter<User>();

  onSelect(): void {
    this.selected.emit(this.user);
  }
}
```

### Standalone Components (Angular 14+)

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-dashboard',
  standalone: true,
  imports: [CommonModule, RouterModule],
  template: `
    <h1>Dashboard</h1>
    <router-outlet></router-outlet>
  `
})
export class DashboardComponent {}
```

### Component Lifecycle Hooks

```typescript
import {
  OnInit, OnDestroy, OnChanges, DoCheck,
  AfterContentInit, AfterContentChecked,
  AfterViewInit, AfterViewChecked,
  SimpleChanges
} from '@angular/core';

@Component({ ... })
export class LifecycleComponent implements OnInit, OnChanges, OnDestroy {
  
  // Called once when input properties change (before ngOnInit)
  ngOnChanges(changes: SimpleChanges): void {
    console.log('Input changed:', changes);
  }

  // Called once after first ngOnChanges
  ngOnInit(): void {
    console.log('Component initialized');
    // Fetch data, setup subscriptions
  }

  // Called during every change detection run
  ngDoCheck(): void {
    console.log('Change detection running');
  }

  // Called after content (ng-content) is projected
  ngAfterContentInit(): void {
    console.log('Content projected');
  }

  // Called after every check of projected content
  ngAfterContentChecked(): void {}

  // Called after component's view is initialized
  ngAfterViewInit(): void {
    console.log('View initialized');
    // Safe to access ViewChild here
  }

  // Called after every check of component's view
  ngAfterViewChecked(): void {}

  // Called once before component is destroyed
  ngOnDestroy(): void {
    console.log('Component destroyed');
    // Unsubscribe, cleanup
  }
}
```

### Lifecycle Hook Order

```
constructor()
    ↓
ngOnChanges()      ← (if inputs exist)
    ↓
ngOnInit()
    ↓
ngDoCheck()
    ↓
ngAfterContentInit()
    ↓
ngAfterContentChecked()
    ↓
ngAfterViewInit()
    ↓
ngAfterViewChecked()
    ↓
ngOnDestroy()
```

### Component Communication

**Parent to Child: @Input()**

```typescript
// Parent template
<app-child [message]="parentMessage" [user]="currentUser"></app-child>

// Child component
@Input() message: string = '';
@Input() user!: User;

// With setter for transformation/validation
@Input()
set name(value: string) {
  this._name = value.trim().toUpperCase();
}
private _name = '';
```

**Child to Parent: @Output()**

```typescript
// Child component
@Output() notify = new EventEmitter<string>();

sendNotification(): void {
  this.notify.emit('Hello from child!');
}

// Parent template
<app-child (notify)="onNotify($event)"></app-child>

// Parent component
onNotify(message: string): void {
  console.log(message);
}
```

**ViewChild & ContentChild**

```typescript
import { ViewChild, ContentChild, ElementRef, AfterViewInit } from '@angular/core';

@Component({
  template: `
    <input #nameInput />
    <app-child></app-child>
    <ng-content></ng-content>
  `
})
export class ParentComponent implements AfterViewInit {
  @ViewChild('nameInput') inputRef!: ElementRef<HTMLInputElement>;
  @ViewChild(ChildComponent) childComponent!: ChildComponent;
  @ContentChild('projectedContent') content!: ElementRef;

  ngAfterViewInit(): void {
    this.inputRef.nativeElement.focus();
    this.childComponent.someMethod();
  }
}
```

---

## Part 3: Templates and Data Binding

### Interpolation

```html
<!-- String interpolation -->
<h1>{{ title }}</h1>
<p>{{ user.name }}</p>
<span>{{ 1 + 1 }}</span>
<div>{{ getFullName() }}</div>
```

### Property Binding

```html
<!-- Property binding -->
<img [src]="imageUrl" />
<button [disabled]="isDisabled">Submit</button>
<div [class.active]="isActive"></div>
<div [style.color]="textColor"></div>
<input [value]="name" />

<!-- Attribute binding (when no DOM property exists) -->
<td [attr.colspan]="colSpan"></td>
<div [attr.aria-label]="label"></div>
```

### Event Binding

```html
<!-- Event binding -->
<button (click)="onClick()">Click</button>
<input (input)="onInput($event)" />
<input (keyup.enter)="onEnter()" />
<form (ngSubmit)="onSubmit()">

<!-- Multiple events -->
<div (mouseenter)="onMouseEnter()" (mouseleave)="onMouseLeave()"></div>
```

### Two-Way Binding

```html
<!-- Two-way binding with ngModel -->
<input [(ngModel)]="username" />

<!-- Equivalent to: -->
<input [ngModel]="username" (ngModelChange)="username = $event" />

<!-- Custom two-way binding -->
<app-counter [(value)]="count"></app-counter>
```

```typescript
// Custom two-way binding in component
@Input() value: number = 0;
@Output() valueChange = new EventEmitter<number>();

increment(): void {
  this.value++;
  this.valueChange.emit(this.value);
}
```

### Template Reference Variables

```html
<input #nameInput type="text" />
<button (click)="greet(nameInput.value)">Greet</button>

<!-- Reference to component -->
<app-timer #timer></app-timer>
<button (click)="timer.start()">Start Timer</button>
```

---

## Part 4: Directives

### Types of Directives

| Type | Purpose | Example |
|------|---------|---------|
| Component | Directive with template | `@Component` |
| Structural | Change DOM structure | `*ngIf`, `*ngFor`, `*ngSwitch` |
| Attribute | Change appearance/behavior | `ngClass`, `ngStyle`, custom |

### Structural Directives

**ngIf**

```html
<div *ngIf="isVisible">Visible content</div>

<div *ngIf="user; else noUser">
  Welcome, {{ user.name }}
</div>
<ng-template #noUser>
  <p>Please log in</p>
</ng-template>

<!-- With as syntax -->
<div *ngIf="user$ | async as user">
  {{ user.name }}
</div>
```

**ngFor**

```html
<ul>
  <li *ngFor="let item of items; let i = index; let first = first; let last = last; let even = even; let odd = odd; trackBy: trackById">
    {{ i + 1 }}. {{ item.name }}
  </li>
</ul>
```

```typescript
trackById(index: number, item: Item): number {
  return item.id;
}
```

**ngSwitch**

```html
<div [ngSwitch]="status">
  <p *ngSwitchCase="'pending'">Pending...</p>
  <p *ngSwitchCase="'approved'">Approved!</p>
  <p *ngSwitchCase="'rejected'">Rejected</p>
  <p *ngSwitchDefault>Unknown status</p>
</div>
```

### Attribute Directives

**ngClass**

```html
<div [ngClass]="'active'"></div>
<div [ngClass]="['active', 'highlight']"></div>
<div [ngClass]="{ 'active': isActive, 'disabled': isDisabled }"></div>
<div [ngClass]="getClasses()"></div>
```

**ngStyle**

```html
<div [ngStyle]="{ 'color': textColor, 'font-size': fontSize + 'px' }"></div>
<div [ngStyle]="getStyles()"></div>
```

### Custom Attribute Directive

```typescript
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() appHighlight = 'yellow';
  @Input() defaultColor = 'transparent';

  constructor(private el: ElementRef) {}

  @HostListener('mouseenter')
  onMouseEnter(): void {
    this.highlight(this.appHighlight || 'yellow');
  }

  @HostListener('mouseleave')
  onMouseLeave(): void {
    this.highlight(this.defaultColor);
  }

  private highlight(color: string): void {
    this.el.nativeElement.style.backgroundColor = color;
  }
}
```

```html
<p appHighlight="lightblue">Hover over me!</p>
<p [appHighlight]="'pink'" [defaultColor]="'lightgray'">Custom colors</p>
```

### Custom Structural Directive

```typescript
import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';

@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  private hasView = false;

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}

  @Input()
  set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}
```

```html
<p *appUnless="isLoggedIn">Please log in to continue</p>
```

---

## Part 5: Services and Dependency Injection

### Creating a Service

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';

@Injectable({
  providedIn: 'root' // Singleton, available app-wide
})
export class UserService {
  private apiUrl = '/api/users';
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  
  currentUser$ = this.currentUserSubject.asObservable();

  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  createUser(user: User): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  setCurrentUser(user: User): void {
    this.currentUserSubject.next(user);
  }
}
```

### Dependency Injection Providers

```typescript
// In module
@NgModule({
  providers: [
    // Class provider (default)
    UserService,
    // Equivalent to:
    { provide: UserService, useClass: UserService },

    // Value provider
    { provide: 'API_URL', useValue: 'https://api.example.com' },

    // Factory provider
    {
      provide: LoggerService,
      useFactory: (config: ConfigService) => {
        return config.debug ? new DebugLogger() : new ProductionLogger();
      },
      deps: [ConfigService]
    },

    // Existing provider (alias)
    { provide: AbstractLogger, useExisting: LoggerService }
  ]
})
```

### Injection Tokens

```typescript
import { InjectionToken } from '@angular/core';

export interface AppConfig {
  apiUrl: string;
  debug: boolean;
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

// Provide in module
@NgModule({
  providers: [
    {
      provide: APP_CONFIG,
      useValue: { apiUrl: 'https://api.example.com', debug: true }
    }
  ]
})

// Inject in component/service
constructor(@Inject(APP_CONFIG) private config: AppConfig) {}
```

### providedIn Options

```typescript
// Application-wide singleton
@Injectable({ providedIn: 'root' })

// Lazy-loaded module scope
@Injectable({ providedIn: 'any' })

// Platform-wide (multiple apps)
@Injectable({ providedIn: 'platform' })

// Specific module
@Injectable({ providedIn: SomeModule })
```

### Hierarchical Injection

```typescript
// Component-level provider (new instance per component)
@Component({
  selector: 'app-example',
  providers: [DataService], // New instance for this component tree
  template: '...'
})
export class ExampleComponent {}

// viewProviders - only available to view, not content children
@Component({
  viewProviders: [DataService]
})
```

---

## Part 6: RxJS and Observables

### Common Operators

```typescript
import { 
  map, filter, tap, switchMap, mergeMap, concatMap, exhaustMap,
  catchError, retry, debounceTime, distinctUntilChanged,
  take, takeUntil, first, shareReplay, combineLatestWith
} from 'rxjs/operators';
import { of, throwError, Subject, BehaviorSubject, forkJoin, combineLatest } from 'rxjs';

// Transformation
this.users$ = this.http.get<User[]>('/api/users').pipe(
  map(users => users.filter(u => u.active)),
  tap(users => console.log('Active users:', users))
);

// Error handling
this.data$ = this.http.get('/api/data').pipe(
  retry(3),
  catchError(error => {
    console.error('Error:', error);
    return of([]); // Return fallback value
  })
);

// Search with debounce
this.searchResults$ = this.searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  filter(term => term.length >= 2),
  switchMap(term => this.searchService.search(term))
);
```

### Higher-Order Mapping Operators

| Operator | Behavior | Use Case |
|----------|----------|----------|
| `switchMap` | Cancels previous, uses latest | Search, autocomplete |
| `mergeMap` | Runs all in parallel | Independent requests |
| `concatMap` | Runs sequentially | Order matters |
| `exhaustMap` | Ignores new until current completes | Prevent duplicate submits |

```typescript
// switchMap - Cancel previous request
search$ = this.searchTerm$.pipe(
  switchMap(term => this.api.search(term))
);

// exhaustMap - Prevent duplicate form submissions
submit$ = this.submitClick$.pipe(
  exhaustMap(() => this.api.submitForm(this.form.value))
);

// concatMap - Sequential processing
queue$ = this.tasks$.pipe(
  concatMap(task => this.api.processTask(task))
);
```

### Combining Observables

```typescript
// forkJoin - Wait for all to complete
forkJoin({
  users: this.userService.getUsers(),
  products: this.productService.getProducts(),
  orders: this.orderService.getOrders()
}).subscribe(({ users, products, orders }) => {
  // All data available
});

// combineLatest - Emit when any source emits
combineLatest([
  this.user$,
  this.settings$
]).subscribe(([user, settings]) => {
  // Latest values from both
});

// withLatestFrom - Main stream + latest from others
this.actions$.pipe(
  withLatestFrom(this.state$),
  map(([action, state]) => ({ action, state }))
);
```

### Subject Types

```typescript
// Subject - Basic multicast
const subject = new Subject<number>();
subject.subscribe(v => console.log('A:', v));
subject.next(1);
subject.subscribe(v => console.log('B:', v));
subject.next(2);
// A: 1, A: 2, B: 2

// BehaviorSubject - Has current value
const behavior = new BehaviorSubject<number>(0);
behavior.subscribe(v => console.log('A:', v)); // A: 0
behavior.next(1); // A: 1
behavior.subscribe(v => console.log('B:', v)); // B: 1
console.log(behavior.getValue()); // 1

// ReplaySubject - Replays n values
const replay = new ReplaySubject<number>(2);
replay.next(1);
replay.next(2);
replay.next(3);
replay.subscribe(v => console.log(v)); // 2, 3

// AsyncSubject - Only emits last value on complete
const async = new AsyncSubject<number>();
async.next(1);
async.next(2);
async.subscribe(v => console.log(v));
async.next(3);
async.complete(); // Only now: 3
```

### Unsubscribing Patterns

```typescript
// Pattern 1: takeUntil with destroy subject
export class MyComponent implements OnDestroy {
  private destroy$ = new Subject<void>();

  ngOnInit(): void {
    this.dataService.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => this.data = data);
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// Pattern 2: Async pipe (auto-unsubscribes)
@Component({
  template: `
    <div *ngIf="user$ | async as user">
      {{ user.name }}
    </div>
  `
})
export class MyComponent {
  user$ = this.userService.getUser();
}

// Pattern 3: takeUntilDestroyed (Angular 16+)
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

export class MyComponent {
  constructor() {
    this.dataService.getData()
      .pipe(takeUntilDestroyed())
      .subscribe(data => this.data = data);
  }
}
```

---

## Part 7: Forms

### Template-Driven Forms

```typescript
import { FormsModule } from '@angular/forms';

@Component({
  template: `
    <form #userForm="ngForm" (ngSubmit)="onSubmit(userForm)">
      <input 
        name="name" 
        [(ngModel)]="user.name" 
        required 
        minlength="2"
        #name="ngModel"
      />
      <div *ngIf="name.invalid && name.touched">
        <span *ngIf="name.errors?.['required']">Name is required</span>
        <span *ngIf="name.errors?.['minlength']">Min 2 characters</span>
      </div>

      <input 
        name="email" 
        [(ngModel)]="user.email" 
        required 
        email
      />

      <button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  user = { name: '', email: '' };

  onSubmit(form: NgForm): void {
    if (form.valid) {
      console.log(this.user);
    }
  }
}
```

### Reactive Forms

```typescript
import { ReactiveFormsModule, FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <input formControlName="name" />
      <div *ngIf="userForm.get('name')?.invalid && userForm.get('name')?.touched">
        <span *ngIf="userForm.get('name')?.errors?.['required']">Required</span>
      </div>

      <input formControlName="email" />
      
      <div formGroupName="address">
        <input formControlName="street" />
        <input formControlName="city" />
      </div>

      <div formArrayName="phones">
        <div *ngFor="let phone of phones.controls; let i = index">
          <input [formControlName]="i" />
          <button (click)="removePhone(i)">Remove</button>
        </div>
        <button type="button" (click)="addPhone()">Add Phone</button>
      </div>

      <button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent implements OnInit {
  userForm!: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit(): void {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(2)]],
      email: ['', [Validators.required, Validators.email]],
      address: this.fb.group({
        street: [''],
        city: ['', Validators.required]
      }),
      phones: this.fb.array([])
    });
  }

  get phones() {
    return this.userForm.get('phones') as FormArray;
  }

  addPhone(): void {
    this.phones.push(this.fb.control('', Validators.required));
  }

  removePhone(index: number): void {
    this.phones.removeAt(index);
  }

  onSubmit(): void {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    }
  }
}
```

### Custom Validators

```typescript
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

// Sync validator function
export function forbiddenNameValidator(forbiddenName: RegExp): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = forbiddenName.test(control.value);
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  };
}

// Async validator
@Injectable({ providedIn: 'root' })
export class UniqueEmailValidator {
  constructor(private userService: UserService) {}

  validate(): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      return this.userService.checkEmailExists(control.value).pipe(
        map(exists => exists ? { emailTaken: true } : null),
        catchError(() => of(null))
      );
    };
  }
}

// Usage
this.userForm = this.fb.group({
  name: ['', [Validators.required, forbiddenNameValidator(/admin/i)]],
  email: ['', 
    [Validators.required, Validators.email], 
    [this.emailValidator.validate()]
  ]
});
```

### Template-Driven vs Reactive Forms

| Feature | Template-Driven | Reactive |
|---------|-----------------|----------|
| Form model | Implicit (directives) | Explicit (in class) |
| Data flow | Async | Sync |
| Validation | Directives | Functions |
| Testing | Harder | Easier |
| Scalability | Small forms | Complex forms |
| Dynamic forms | Difficult | Easy |

---

## Part 8: Routing

### Basic Setup

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'users', component: UsersComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: '**', component: NotFoundComponent }
];

// app.config.ts (standalone)
import { provideRouter } from '@angular/router';

export const appConfig = {
  providers: [provideRouter(routes)]
};
```

### Route Parameters

```typescript
import { ActivatedRoute, Router } from '@angular/router';

@Component({ ... })
export class UserDetailComponent implements OnInit {
  constructor(
    private route: ActivatedRoute,
    private router: Router
  ) {}

  ngOnInit(): void {
    // Snapshot (one-time read)
    const id = this.route.snapshot.paramMap.get('id');

    // Observable (reacts to changes)
    this.route.paramMap.subscribe(params => {
      this.userId = params.get('id');
    });

    // Query parameters
    this.route.queryParamMap.subscribe(params => {
      this.page = params.get('page');
    });
  }

  navigateToUser(id: number): void {
    this.router.navigate(['/users', id], {
      queryParams: { tab: 'profile' },
      fragment: 'details'
    });
  }
}
```

### Lazy Loading

```typescript
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes')
      .then(m => m.ADMIN_ROUTES)
  },
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard/dashboard.component')
      .then(m => m.DashboardComponent)
  }
];
```

### Route Guards

```typescript
// Auth guard (functional - Angular 15+)
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }
  
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Can deactivate guard
export const unsavedChangesGuard: CanDeactivateFn<FormComponent> = 
  (component) => {
    if (component.hasUnsavedChanges()) {
      return confirm('Discard unsaved changes?');
    }
    return true;
  };

// Usage in routes
const routes: Routes = [
  {
    path: 'admin',
    canActivate: [authGuard],
    canDeactivate: [unsavedChangesGuard],
    component: AdminComponent
  }
];
```

### Guard Types

| Guard | Purpose | Interface |
|-------|---------|-----------|
| `canActivate` | Control route access | `CanActivateFn` |
| `canActivateChild` | Control child route access | `CanActivateChildFn` |
| `canDeactivate` | Prevent leaving route | `CanDeactivateFn` |
| `canLoad` | Prevent lazy loading | `CanLoadFn` |
| `canMatch` | Control route matching | `CanMatchFn` |
| `resolve` | Pre-fetch data | `ResolveFn` |

### Resolvers

```typescript
// Resolver function
export const userResolver: ResolveFn<User> = (route) => {
  const userService = inject(UserService);
  const id = route.paramMap.get('id')!;
  return userService.getUser(+id);
};

// Route configuration
const routes: Routes = [
  {
    path: 'users/:id',
    component: UserDetailComponent,
    resolve: { user: userResolver }
  }
];

// Access in component
@Component({ ... })
export class UserDetailComponent {
  user = inject(ActivatedRoute).snapshot.data['user'];
  
  // Or reactive
  user$ = inject(ActivatedRoute).data.pipe(
    map(data => data['user'])
  );
}
```

---

## Part 9: HTTP and Interceptors

### HttpClient Usage

```typescript
import { HttpClient, HttpParams, HttpHeaders } from '@angular/common/http';

@Injectable({ providedIn: 'root' })
export class ApiService {
  private baseUrl = '/api';

  constructor(private http: HttpClient) {}

  // GET
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.baseUrl}/users`);
  }

  // GET with params
  searchUsers(term: string, page: number): Observable<User[]> {
    const params = new HttpParams()
      .set('q', term)
      .set('page', page.toString());
    
    return this.http.get<User[]>(`${this.baseUrl}/users`, { params });
  }

  // POST
  createUser(user: User): Observable<User> {
    return this.http.post<User>(`${this.baseUrl}/users`, user);
  }

  // PUT
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.baseUrl}/users/${id}`, user);
  }

  // DELETE
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/users/${id}`);
  }

  // With headers
  uploadFile(file: File): Observable<any> {
    const formData = new FormData();
    formData.append('file', file);

    const headers = new HttpHeaders({
      'Accept': 'application/json'
    });

    return this.http.post(`${this.baseUrl}/upload`, formData, { headers });
  }
}
```

### HTTP Interceptors

```typescript
// Functional interceptor (Angular 15+)
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(req);
};

// Error interceptor
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        inject(AuthService).logout();
        inject(Router).navigate(['/login']);
      }
      return throwError(() => error);
    })
  );
};

// Loading interceptor
export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  const loadingService = inject(LoadingService);
  loadingService.show();

  return next(req).pipe(
    finalize(() => loadingService.hide())
  );
};

// Register interceptors
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor, loadingInterceptor])
    )
  ]
};
```

### Class-based Interceptor (Legacy)

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = this.authService.getToken();
    
    if (token) {
      req = req.clone({
        setHeaders: { Authorization: `Bearer ${token}` }
      });
    }

    return next.handle(req);
  }
}

// Register in module
@NgModule({
  providers: [
    { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true }
  ]
})
```

---

## Part 10: Signals (Angular 16+)

### Basic Signals

```typescript
import { signal, computed, effect } from '@angular/core';

@Component({
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ doubleCount() }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  // Writable signal
  count = signal(0);

  // Computed signal (derived, read-only)
  doubleCount = computed(() => this.count() * 2);

  constructor() {
    // Effect - runs when dependencies change
    effect(() => {
      console.log('Count changed:', this.count());
    });
  }

  increment(): void {
    // Update methods
    this.count.set(10);           // Set value
    this.count.update(v => v + 1); // Update based on previous
  }
}
```

### Signal-based Inputs (Angular 17.1+)

```typescript
import { input, output } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <div>{{ name() }}</div>
    <div>{{ age() }}</div>
  `
})
export class UserCardComponent {
  // Required input
  name = input.required<string>();
  
  // Optional with default
  age = input(0);
  
  // Transformed input
  id = input(0, { transform: numberAttribute });
  
  // Aliased input
  userName = input('', { alias: 'user-name' });

  // Output
  selected = output<User>();

  onSelect(): void {
    this.selected.emit(this.user);
  }
}
```

### Signal Queries (Angular 17.2+)

```typescript
import { viewChild, viewChildren, contentChild, contentChildren } from '@angular/core';

@Component({ ... })
export class ParentComponent {
  // Single element
  input = viewChild<ElementRef>('nameInput');
  
  // Required
  requiredInput = viewChild.required<ElementRef>('nameInput');
  
  // Multiple elements
  items = viewChildren<ItemComponent>(ItemComponent);
  
  // Content projection
  header = contentChild<ElementRef>('header');
  tabs = contentChildren<TabComponent>(TabComponent);

  ngAfterViewInit(): void {
    console.log(this.input()?.nativeElement);
    this.items().forEach(item => item.activate());
  }
}
```

### toSignal and toObservable

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

@Component({ ... })
export class DataComponent {
  private dataService = inject(DataService);

  // Convert Observable to Signal
  users = toSignal(this.dataService.getUsers(), { 
    initialValue: [] 
  });

  // With error handling
  data = toSignal(this.dataService.getData(), {
    initialValue: null,
    rejectErrors: true // Throws instead of returning error
  });

  // Convert Signal to Observable
  count = signal(0);
  count$ = toObservable(this.count);

  constructor() {
    // Use signal in template
    effect(() => {
      console.log('Users:', this.users());
    });
  }
}
```

---

## Part 11: Change Detection

### Change Detection Strategies

```typescript
import { ChangeDetectionStrategy, ChangeDetectorRef } from '@angular/core';

// Default: Checks component on every cycle
@Component({
  changeDetection: ChangeDetectionStrategy.Default
})

// OnPush: Only checks when:
// 1. Input reference changes
// 2. Event from component or children
// 3. Async pipe emits
// 4. Manual trigger
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent {
  constructor(private cdr: ChangeDetectorRef) {}

  updateManually(): void {
    // Mark for check in next cycle
    this.cdr.markForCheck();
    
    // Run change detection immediately
    this.cdr.detectChanges();
    
    // Detach from change detection tree
    this.cdr.detach();
    
    // Reattach
    this.cdr.reattach();
  }
}
```

### Zone.js and NgZone

```typescript
import { NgZone } from '@angular/core';

@Component({ ... })
export class ExampleComponent {
  constructor(private ngZone: NgZone) {}

  // Run outside Angular zone (no change detection)
  runOutsideAngular(): void {
    this.ngZone.runOutsideAngular(() => {
      // Performance-intensive operations
      setInterval(() => {
        // This won't trigger change detection
      }, 100);
    });
  }

  // Run inside Angular zone (triggers change detection)
  runInsideAngular(): void {
    this.ngZone.run(() => {
      this.data = newData;
    });
  }
}
```

### Zoneless Angular (Experimental)

```typescript
// app.config.ts
import { provideExperimentalZonelessChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
};

// Components must use signals or manual change detection
@Component({
  template: `{{ count() }}`
})
export class ZonelessComponent {
  count = signal(0);
  
  increment(): void {
    this.count.update(c => c + 1); // Auto-updates view
  }
}
```

---

## Part 12: Advanced Interview Questions

### Question 1: Explain Angular's dependency injection hierarchy

**Answer**:

Angular has a hierarchical DI system:

```
Platform Injector (singleton services)
    ↓
Root Injector (providedIn: 'root')
    ↓
Module Injectors (NgModule providers)
    ↓
Element Injectors (Component providers)
```

- Services provided in `root` are singletons app-wide
- Component-level providers create new instances for each component tree
- Child components can override parent providers
- Resolution walks up the tree until found or throws error

```typescript
// Different instances per component
@Component({
  providers: [DataService] // New instance for this subtree
})

// Same instance for view, not projected content
@Component({
  viewProviders: [DataService]
})
```

---

### Question 2: What's the difference between ViewChild and ContentChild?

**Answer**:

```typescript
@Component({
  selector: 'app-parent',
  template: `
    <div #viewElement>View content</div>  <!-- ViewChild -->
    <ng-content></ng-content>               <!-- ContentChild -->
  `
})
export class ParentComponent {
  @ViewChild('viewElement') viewEl!: ElementRef;  // Elements in template
  @ContentChild('projected') contentEl!: ElementRef;  // Projected content
}

// Usage
<app-parent>
  <div #projected>This is projected content</div>
</app-parent>
```

- **ViewChild**: Queries elements in the component's own template
- **ContentChild**: Queries elements projected via `<ng-content>`

---

### Question 3: How does Angular handle XSS prevention?

**Answer**:

Angular sanitizes values by default:

```typescript
// Automatically sanitized (safe)
<div [innerHTML]="userContent"></div>

// Bypass sanitization (use with caution)
import { DomSanitizer, SafeHtml } from '@angular/platform-browser';

@Component({ ... })
export class MyComponent {
  trustedHtml: SafeHtml;

  constructor(private sanitizer: DomSanitizer) {
    this.trustedHtml = this.sanitizer.bypassSecurityTrustHtml(htmlString);
  }
}
```

Angular sanitizes:
- HTML (`innerHTML`)
- Styles (`style` property)
- URLs (`href`, `src`)
- Resource URLs (scripts, etc.)

---

### Question 4: Explain the difference between Promise and Observable

**Answer**:

| Feature | Promise | Observable |
|---------|---------|------------|
| Execution | Eager (immediate) | Lazy (on subscribe) |
| Values | Single value | Multiple values over time |
| Cancellation | Not cancellable | Unsubscribable |
| Operators | Limited (then, catch) | Rich operator library |
| Error handling | .catch() | catchError operator |

```typescript
// Promise - executes immediately
const promise = new Promise(resolve => {
  console.log('Promise executing');
  resolve('done');
});

// Observable - executes on subscribe
const observable = new Observable(subscriber => {
  console.log('Observable executing');
  subscriber.next('value 1');
  subscriber.next('value 2');
  subscriber.complete();
});

// Only executes when subscribed
const sub = observable.subscribe(console.log);
sub.unsubscribe(); // Can cancel
```

---

### Question 5: How would you optimize an Angular application?

**Answer**:

**Build Optimization**:
- Enable production mode: `ng build --configuration production`
- Lazy load routes and components
- Use tree-shaking (remove unused code)
- Enable AOT compilation (default in production)

**Runtime Optimization**:
- Use `OnPush` change detection
- Use `trackBy` with `ngFor`
- Unsubscribe from observables
- Use `async` pipe (auto-unsubscribes)
- Use signals for reactive state

**Bundle Optimization**:
- Analyze bundle: `ng build --stats-json` + webpack-bundle-analyzer
- Code splitting with lazy loading
- Import only needed modules from libraries

**Performance Patterns**:
```typescript
// trackBy
<li *ngFor="let item of items; trackBy: trackById">

// OnPush
@Component({ changeDetection: ChangeDetectionStrategy.OnPush })

// Virtual scrolling for large lists
<cdk-virtual-scroll-viewport itemSize="50">
  <div *cdkVirtualFor="let item of items">{{ item }}</div>
</cdk-virtual-scroll-viewport>
```

---

### Question 6: What are the differences between ngOnInit and constructor?

**Answer**:

| Aspect | Constructor | ngOnInit |
|--------|-------------|----------|
| Purpose | DI, basic initialization | Component initialization |
| Timing | Before Angular initializes | After inputs are set |
| Input values | Not available | Available |
| DOM access | Not ready | Still not ready (use AfterViewInit) |
| Best practice | Only for DI | Data fetching, subscriptions |

```typescript
@Component({ ... })
export class MyComponent implements OnInit {
  @Input() userId!: number;

  constructor(private userService: UserService) {
    // userId is undefined here
    console.log(this.userId); // undefined
  }

  ngOnInit(): void {
    // userId is set
    console.log(this.userId); // actual value
    this.userService.getUser(this.userId).subscribe(...);
  }
}
```

---

### Question 7: Explain Angular's compilation modes (JIT vs AOT)

**Answer**:

| Aspect | JIT (Just-in-Time) | AOT (Ahead-of-Time) |
|--------|--------------------|--------------------|
| Compilation | Browser at runtime | Build time |
| Bundle size | Larger (includes compiler) | Smaller |
| Startup | Slower | Faster |
| Error detection | Runtime | Build time |
| Development | Default in dev | Default in prod |

```bash
# JIT (development)
ng serve

# AOT (production)
ng build --configuration production
```

AOT benefits:
- Faster rendering (pre-compiled templates)
- Smaller bundle (no compiler shipped)
- Earlier template error detection
- Better security (templates already compiled)

---

### Question 8: How do Standalone Components change Angular architecture?

**Answer**:

Standalone components (Angular 14+) remove the need for NgModules:

```typescript
// Traditional (with NgModule)
@NgModule({
  declarations: [MyComponent],
  imports: [CommonModule, RouterModule],
  exports: [MyComponent]
})
export class MyModule {}

// Standalone
@Component({
  selector: 'app-my',
  standalone: true,
  imports: [CommonModule, RouterModule, OtherStandaloneComponent],
  template: '...'
})
export class MyComponent {}
```

Benefits:
- Simpler mental model
- Better tree-shaking
- Easier lazy loading
- Less boilerplate
- Direct imports of dependencies

---

### Question 9: Explain content projection and its variations

**Answer**:

```typescript
// Single slot
@Component({
  template: `
    <div class="card">
      <ng-content></ng-content>
    </div>
  `
})

// Multi-slot with select
@Component({
  template: `
    <header>
      <ng-content select="[header]"></ng-content>
    </header>
    <main>
      <ng-content select="[body]"></ng-content>
    </main>
    <footer>
      <ng-content select="[footer]"></ng-content>
    </footer>
  `
})

// Usage
<app-card>
  <div header>Title</div>
  <div body>Content</div>
  <div footer>Actions</div>
</app-card>

// Conditional projection with ng-template
@Component({
  template: `
    <ng-container *ngTemplateOutlet="headerTemplate || defaultHeader">
    </ng-container>
    <ng-template #defaultHeader>Default</ng-template>
  `
})
export class CardComponent {
  @ContentChild('header') headerTemplate?: TemplateRef<any>;
}
```

---

### Question 10: What's new in recent Angular versions?

**Answer**:

**Angular 17+**:
- New control flow syntax (`@if`, `@for`, `@switch`)
- Deferrable views (`@defer`)
- Built-in SSR improvements
- New branding and documentation

```html
<!-- New control flow -->
@if (user) {
  <p>{{ user.name }}</p>
} @else {
  <p>No user</p>
}

@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No items</li>
}

<!-- Deferrable views -->
@defer (on viewport) {
  <app-heavy-component />
} @placeholder {
  <p>Loading...</p>
}
```

**Angular 16+**:
- Signals for reactive state
- Required inputs
- takeUntilDestroyed
- Server-side rendering improvements

**Angular 15+**:
- Stable standalone APIs
- Functional guards and resolvers
- Directive composition API

---

## Quick Reference: Common Mistakes

| Mistake | Solution |
|---------|----------|
| Memory leaks from subscriptions | Use `takeUntil`, `async` pipe, or `takeUntilDestroyed` |
| Mutating @Input objects | Use immutable patterns or OnPush |
| Not using trackBy with ngFor | Always use trackBy for performance |
| Calling methods in templates | Use pipes or computed values instead |
| Overusing any type | Use proper TypeScript typing |
| Direct DOM manipulation | Use Renderer2 or directives |
| Not lazy loading routes | Implement lazy loading for large apps |
| Ignoring bundle size | Analyze and optimize regularly |

---

## Additional Resources

- Official Angular Documentation: https://angular.dev
- Angular Blog: https://blog.angular.dev
- Angular University: https://angular-university.io
- RxJS Documentation: https://rxjs.dev

---

*Good luck with your interview!*
