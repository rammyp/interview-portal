# Angular Interview Preparation Guide

A comprehensive guide covering Angular concepts from basic to advanced for interview preparation.

---

## Table of Contents

1. [Angular Architecture](#1-angular-architecture)
2. [Components](#2-components)
3. [Component Lifecycle Hooks](#3-component-lifecycle-hooks)
4. [Data Binding](#4-data-binding)
5. [Directives](#5-directives)
6. [Pipes](#6-pipes)
7. [Services and Dependency Injection](#7-services-and-dependency-injection)
8. [RxJS and Observables](#8-rxjs-and-observables)
9. [Signals](#9-signals-angular-16)
10. [Routing](#10-routing)
11. [Forms](#11-forms)
12. [HTTP Client and Interceptors](#12-http-client-and-interceptors)
13. [Change Detection](#13-change-detection)
14. [Standalone Components](#14-standalone-components)
15. [Performance Optimization](#15-performance-optimization)
16. [State Management with NgRx](#16-state-management-with-ngrx)
17. [Testing](#17-testing)
18. [Common Interview Questions](#18-common-interview-questions)

---

## 1. Angular Architecture

Angular follows a component-based architecture with a clear separation of concerns.

```
┌─────────────────────────────────────────────────────────┐
│                      Angular App                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │   Modules   │  │  Components │  │    Services     │  │
│  │  (NgModule) │  │  (UI Logic) │  │ (Business Logic)│  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │  Directives │  │    Pipes    │  │     Guards      │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Key Building Blocks

| Building Block | Purpose |
|----------------|---------|
| **Modules** | Group related components, services, and other code |
| **Components** | Handle UI and user interaction |
| **Services** | Contain reusable business logic |
| **Directives** | Modify DOM behavior |
| **Pipes** | Transform data for display |
| **Guards** | Control route access |

**Interview Question: Explain Angular architecture**

> Angular follows a component-based architecture. Modules group related components, services, and other code. Components handle the UI and user interaction. Services contain reusable business logic and are injected via dependency injection.

---

## 2. Components

Components are the fundamental building blocks of Angular applications.

### Basic Component Structure

```typescript
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div class="user-card">
      <h2>{{ user.name }}</h2>
      <button (click)="onSave()">Save</button>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserComponent implements OnInit, OnDestroy {
  @Input() user!: User;
  @Output() save = new EventEmitter<User>();

  ngOnInit(): void {
    console.log('Component initialized');
  }

  onSave(): void {
    this.save.emit(this.user);
  }

  ngOnDestroy(): void {
    console.log('Component destroyed');
  }
}
```

### Component Communication

#### Parent to Child: @Input

```typescript
// Parent component
@Component({
  template: `<app-child [message]="parentMessage"></app-child>`
})
export class ParentComponent {
  parentMessage = 'Hello from parent';
}

// Child component
@Component({
  selector: 'app-child',
  template: `<p>{{ message }}</p>`
})
export class ChildComponent {
  @Input() message!: string;
}
```

#### Child to Parent: @Output with EventEmitter

```typescript
// Child component
@Component({
  selector: 'app-child',
  template: `<button (click)="sendMessage()">Send</button>`
})
export class ChildComponent {
  @Output() messageEvent = new EventEmitter<string>();

  sendMessage(): void {
    this.messageEvent.emit('Hello from child');
  }
}

// Parent component
@Component({
  template: `<app-child (messageEvent)="receiveMessage($event)"></app-child>`
})
export class ParentComponent {
  receiveMessage(message: string): void {
    console.log(message);
  }
}
```

#### ViewChild and ContentChild

```typescript
@Component({
  template: `
    <app-child #childRef></app-child>
    <input #inputRef>
  `
})
export class ParentComponent implements AfterViewInit {
  @ViewChild('childRef') childComponent!: ChildComponent;
  @ViewChild('inputRef') inputElement!: ElementRef;

  ngAfterViewInit(): void {
    this.inputElement.nativeElement.focus();
  }
}
```

**Interview Question: What is the difference between @Input and @Output?**

> `@Input` allows parent components to pass data down to child components. `@Output` combined with `EventEmitter` allows child components to emit events up to parent components.

---

## 3. Component Lifecycle Hooks

```typescript
export class LifecycleComponent implements OnChanges, OnInit, DoCheck,
  AfterContentInit, AfterContentChecked, AfterViewInit, AfterViewChecked, OnDestroy {

  @Input() data!: string;

  ngOnChanges(changes: SimpleChanges): void {
    console.log('1. ngOnChanges', changes);
  }

  ngOnInit(): void {
    console.log('2. ngOnInit');
  }

  ngDoCheck(): void {
    console.log('3. ngDoCheck');
  }

  ngAfterContentInit(): void {
    console.log('4. ngAfterContentInit');
  }

  ngAfterContentChecked(): void {
    console.log('5. ngAfterContentChecked');
  }

  ngAfterViewInit(): void {
    console.log('6. ngAfterViewInit');
  }

  ngAfterViewChecked(): void {
    console.log('7. ngAfterViewChecked');
  }

  ngOnDestroy(): void {
    console.log('8. ngOnDestroy - Cleanup here');
  }
}
```

**Interview Question: When would you use ngOnInit vs constructor?**

> **Constructor** is for dependency injection and simple variable initialization.
> **ngOnInit** is for initialization logic that depends on input properties being set, making HTTP calls, or setting up subscriptions.

---

## 4. Data Binding

### Types of Data Binding

```typescript
@Component({
  template: `
    <!-- Interpolation: Component → Template -->
    <h1>{{ title }}</h1>

    <!-- Property Binding: Component → Template -->
    <img [src]="imageUrl">
    <button [disabled]="isDisabled">Click</button>

    <!-- Event Binding: Template → Component -->
    <button (click)="onClick()">Click Me</button>
    <input (keyup.enter)="onSubmit()">

    <!-- Two-Way Binding: Component ↔ Template -->
    <input [(ngModel)]="username">

    <!-- Class Binding -->
    <div [class.active]="isActive">Content</div>
    <div [ngClass]="{'active': isActive, 'disabled': isDisabled}">Content</div>

    <!-- Style Binding -->
    <div [style.color]="textColor">Styled</div>
    <div [ngStyle]="{'font-size': fontSize + 'px'}">Styled</div>
  `
})
export class BindingDemoComponent {
  title = 'Hello Angular';
  imageUrl = 'logo.png';
  isDisabled = false;
  username = '';
  isActive = true;
  textColor = 'blue';
  fontSize = 16;

  onClick(): void {
    console.log('Clicked');
  }

  onSubmit(): void {
    console.log('Submitted');
  }
}
```

| Type | Syntax | Direction |
|------|--------|-----------|
| Interpolation | `{{ value }}` | Component → Template |
| Property Binding | `[property]="value"` | Component → Template |
| Event Binding | `(event)="handler()"` | Template → Component |
| Two-Way Binding | `[(ngModel)]="value"` | Component ↔ Template |

---

## 5. Directives

### Structural Directives

```typescript
@Component({
  template: `
    <!-- *ngIf -->
    <div *ngIf="isVisible">Visible content</div>
    <div *ngIf="user; else noUser">Welcome, {{ user.name }}!</div>
    <ng-template #noUser><p>Please log in</p></ng-template>

    <!-- @if (Angular 17+) -->
    @if (isVisible) {
      <div>Visible</div>
    } @else {
      <div>Hidden</div>
    }

    <!-- *ngFor -->
    <li *ngFor="let item of items; let i = index; trackBy: trackById">
      {{ i + 1 }}. {{ item.name }}
    </li>

    <!-- @for (Angular 17+) -->
    @for (item of items; track item.id) {
      <li>{{ item.name }}</li>
    } @empty {
      <li>No items found</li>
    }

    <!-- *ngSwitch -->
    <div [ngSwitch]="status">
      <p *ngSwitchCase="'active'">Active</p>
      <p *ngSwitchCase="'inactive'">Inactive</p>
      <p *ngSwitchDefault>Unknown</p>
    </div>
  `
})
export class DirectivesComponent {
  trackById(index: number, item: { id: number }): number {
    return item.id;
  }
}
```

### Custom Attribute Directive

```typescript
@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  @Input() appHighlight = 'yellow';

  constructor(private el: ElementRef, private renderer: Renderer2) {}

  @HostListener('mouseenter') onMouseEnter(): void {
    this.highlight(this.appHighlight);
  }

  @HostListener('mouseleave') onMouseLeave(): void {
    this.highlight('');
  }

  private highlight(color: string): void {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
  }
}

// Usage: <p appHighlight="lightblue">Hover me</p>
```

**Interview Question: Difference between structural and attribute directives?**

> **Structural directives** change the DOM layout by adding/removing elements (prefixed with `*`).
> **Attribute directives** change the appearance or behavior of existing elements.

---

## 6. Pipes

### Built-in Pipes

```typescript
@Component({
  template: `
    <!-- Date -->
    <p>{{ today | date:'fullDate' }}</p>

    <!-- Currency -->
    <p>{{ price | currency:'USD' }}</p>

    <!-- Decimal -->
    <p>{{ pi | number:'1.2-4' }}</p>

    <!-- Uppercase/Lowercase -->
    <p>{{ name | uppercase }}</p>

    <!-- JSON -->
    <pre>{{ user | json }}</pre>

    <!-- Async -->
    <p>{{ data$ | async }}</p>

    <!-- Chaining -->
    <p>{{ name | uppercase | slice:0:3 }}</p>
  `
})
export class PipesDemoComponent {
  today = new Date();
  price = 1234.56;
  pi = 3.14159;
  name = 'john doe';
  user = { name: 'John', age: 30 };
  data$ = of('Async data');
}
```

### Custom Pipe

```typescript
@Pipe({
  name: 'truncate',
  standalone: true,
  pure: true
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 50): string {
    return value.length > limit ? value.substring(0, limit) + '...' : value;
  }
}

// Usage: {{ longText | truncate:20 }}
```

**Interview Question: Pure vs Impure pipes?**

> **Pure pipes** are only called when input value changes.
> **Impure pipes** are called on every change detection cycle.

---

## 7. Services and Dependency Injection

```typescript
@Injectable({
  providedIn: 'root'  // Singleton, tree-shakable
})
export class UserService {
  private apiUrl = '/api/users';

  constructor(private http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  getUser(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  createUser(user: Omit<User, 'id'>): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  updateUser(id: number, user: Partial<User>): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

### Injection Patterns

```typescript
// Constructor injection (traditional)
@Injectable({ providedIn: 'root' })
export class ExampleService {
  constructor(private http: HttpClient) {}
}

// Inject function (Angular 14+) - preferred
@Injectable({ providedIn: 'root' })
export class ModernService {
  private http = inject(HttpClient);
  private apiUrl = inject(API_URL);
}
```

**Interview Question: Explain providedIn: 'root' vs providing in module**

> **providedIn: 'root'** makes the service a singleton available application-wide and enables tree-shaking.
> **Providing in a module** creates a new instance for that module's injector.

---

## 8. RxJS and Observables

### Common Operators

```typescript
@Injectable({ providedIn: 'root' })
export class DataService {
  constructor(private http: HttpClient) {}

  // Search with debounce
  search(terms$: Observable<string>): Observable<SearchResult[]> {
    return terms$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      filter(term => term.length >= 2),
      switchMap(term => this.http.get<SearchResult[]>(`/api/search?q=${term}`)),
      catchError(err => of([]))
    );
  }
}
```

### Higher-Order Mapping Operators

```typescript
// switchMap: Cancels previous inner observable
searchProducts(term$: Observable<string>): Observable<Product[]> {
  return term$.pipe(
    switchMap(term => this.http.get<Product[]>(`/api/search?q=${term}`))
  );
}

// mergeMap: Runs all concurrently
loadAllUsers(userIds: number[]): Observable<User> {
  return from(userIds).pipe(
    mergeMap(id => this.http.get<User>(`/api/users/${id}`))
  );
}

// concatMap: Sequential execution
processOrders(orders: Order[]): Observable<OrderResult> {
  return from(orders).pipe(
    concatMap(order => this.http.post<OrderResult>('/api/process', order))
  );
}

// exhaustMap: Ignores new values while active
submitForm(clicks$: Observable<void>): Observable<Response> {
  return clicks$.pipe(
    exhaustMap(() => this.http.post<Response>('/api/submit', data))
  );
}
```

### Subjects

```typescript
// Subject: No initial value, no replay
const subject = new Subject<string>();

// BehaviorSubject: Has initial value, replays current
const behaviorSubject = new BehaviorSubject<string>('Initial');

// ReplaySubject: Replays N previous values
const replaySubject = new ReplaySubject<string>(2);
```

### Combining Observables

```typescript
// forkJoin: Wait for all to complete
loadDashboard(): Observable<DashboardData> {
  return forkJoin({
    users: this.http.get<User[]>('/api/users'),
    orders: this.http.get<Order[]>('/api/orders'),
    stats: this.http.get<Stats>('/api/stats')
  });
}

// combineLatest: Emit when any emits
getFilteredData(data$: Observable<Item[]>, filter$: Observable<Filter>): Observable<Item[]> {
  return combineLatest([data$, filter$]).pipe(
    map(([data, filter]) => this.applyFilter(data, filter))
  );
}
```

**Interview Question: switchMap vs mergeMap vs concatMap?**

> **switchMap**: Cancels previous - use for search/autocomplete
> **mergeMap**: Runs all concurrently - use for parallel requests
> **concatMap**: Sequential - use when order matters

---

## 9. Signals (Angular 16+)

```typescript
@Component({
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ doubleCount() }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class SignalsDemoComponent {
  // Writable signal
  count = signal(0);

  // Computed signal (derived)
  doubleCount = computed(() => this.count() * 2);

  constructor() {
    // Effect for side effects
    effect(() => {
      console.log('Count changed:', this.count());
    });
  }

  increment(): void {
    this.count.update(c => c + 1);
  }
}
```

### Converting Between Signals and Observables

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

@Component({...})
export class ConversionComponent {
  // Observable to Signal
  users = toSignal(this.userService.getUsers(), { initialValue: [] });

  // Signal to Observable
  searchTerm = signal('');
  searchResults$ = toObservable(this.searchTerm).pipe(
    debounceTime(300),
    switchMap(term => this.searchService.search(term))
  );
}
```

**Interview Question: Signals vs Observables?**

> **Signals**: Synchronous, simpler for local state, auto-tracked by change detection.
> **Observables**: Asynchronous, better for event streams and HTTP calls, powerful operators.

---

## 10. Routing

### Route Configuration

```typescript
export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent },
  { path: 'users/:id', component: UserDetailComponent },
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [authGuard]
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  },
  { path: '**', component: NotFoundComponent }
];
```

### Route Guards (Functional Style)

```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }
  return router.createUrlTree(['/login'], { queryParams: { returnUrl: state.url } });
};

export const unsavedChangesGuard: CanDeactivateFn<HasUnsavedChanges> = (component) => {
  if (component.hasUnsavedChanges()) {
    return confirm('You have unsaved changes. Leave?');
  }
  return true;
};
```

### Navigation

```typescript
@Component({
  template: `
    <a routerLink="/home">Home</a>
    <a [routerLink]="['/users', userId]">User</a>
    <a routerLink="/search" [queryParams]="{ q: 'angular' }">Search</a>
    <router-outlet></router-outlet>
  `
})
export class NavigationComponent {
  constructor(private router: Router, private route: ActivatedRoute) {}

  navigate(): void {
    this.router.navigate(['/users', 123], { queryParams: { tab: 'profile' } });
  }
}
```

---

## 11. Forms

### Reactive Forms

```typescript
@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <input formControlName="name">
      <div *ngIf="userForm.get('name')?.errors?.['required']">Required</div>

      <input formControlName="email">

      <div formArrayName="phones">
        <div *ngFor="let phone of phones.controls; let i = index">
          <input [formControlName]="i">
          <button type="button" (click)="removePhone(i)">Remove</button>
        </div>
      </div>
      <button type="button" (click)="addPhone()">Add Phone</button>

      <button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class ReactiveFormComponent implements OnInit {
  userForm!: FormGroup;

  constructor(private fb: FormBuilder) {}

  ngOnInit(): void {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(2)]],
      email: ['', [Validators.required, Validators.email]],
      phones: this.fb.array([this.fb.control('', Validators.required)])
    });
  }

  get phones(): FormArray {
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
// Sync validator
export function noWhitespaceValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const isWhitespace = control.value?.trim().length === 0;
    return isWhitespace ? { whitespace: true } : null;
  };
}

// Async validator
export function uniqueEmailValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return userService.checkEmailExists(control.value).pipe(
      map(exists => exists ? { emailTaken: true } : null),
      catchError(() => of(null))
    );
  };
}
```

**Interview Question: Template-driven vs Reactive forms?**

> **Template-driven**: Defined in template with ngModel, good for simple forms.
> **Reactive**: Defined in component class, better for complex forms, easier to test.

---

## 12. HTTP Client and Interceptors

### HTTP Interceptors (Functional Style)

```typescript
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

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        router.navigate(['/login']);
      }
      return throwError(() => error);
    })
  );
};

export const loadingInterceptor: HttpInterceptorFn = (req, next) => {
  const loadingService = inject(LoadingService);
  loadingService.show();

  return next(req).pipe(
    finalize(() => loadingService.hide())
  );
};
```

### Registration

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor, loadingInterceptor])
    )
  ]
};
```

---

## 13. Change Detection

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ data.name }}</p>`
})
export class OnPushComponent {
  @Input() data!: { name: string };

  constructor(private cdr: ChangeDetectorRef) {}

  // Manual trigger
  triggerCheck(): void {
    this.cdr.markForCheck();
    // or
    this.cdr.detectChanges();
  }
}
```

**Interview Question: Default vs OnPush change detection?**

> **Default**: Checks on every cycle.
> **OnPush**: Only checks when @Input reference changes, events occur, or async pipe emits. Requires immutable data patterns.

---

## 14. Standalone Components

```typescript
@Component({
  selector: 'app-standalone',
  standalone: true,
  imports: [CommonModule, RouterModule, FormsModule],
  template: `<p>Standalone component</p>`
})
export class StandaloneComponent {}

// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimations()
  ]
});
```

---

## 15. Performance Optimization

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <!-- TrackBy -->
    <li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>

    <!-- Async pipe -->
    <div *ngIf="user$ | async as user">{{ user.name }}</div>

    <!-- @defer (Angular 17+) -->
    @defer (on viewport) {
      <app-heavy-component />
    } @placeholder {
      <div>Loading...</div>
    }
  `
})
export class OptimizedComponent {
  trackById(index: number, item: Item): number {
    return item.id;
  }
}
```

| Optimization | Description |
|--------------|-------------|
| OnPush | Reduce change detection cycles |
| TrackBy | Minimize DOM manipulation |
| Async Pipe | Auto-unsubscribe |
| Lazy Loading | Load features on demand |
| @defer | Lazy load template sections |

---

## 16. State Management with NgRx

### Actions

```typescript
export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction('[User] Load Users Success', props<{ users: User[] }>());
export const loadUsersFailure = createAction('[User] Load Users Failure', props<{ error: string }>());
```

### Reducer

```typescript
export const userReducer = createReducer(
  initialState,
  on(loadUsers, state => ({ ...state, loading: true })),
  on(loadUsersSuccess, (state, { users }) => ({ ...state, users, loading: false })),
  on(loadUsersFailure, (state, { error }) => ({ ...state, error, loading: false }))
);
```

### Selectors

```typescript
export const selectUserState = createFeatureSelector<UserState>('users');
export const selectAllUsers = createSelector(selectUserState, state => state.users);
export const selectLoading = createSelector(selectUserState, state => state.loading);
```

### Effects

```typescript
@Injectable()
export class UserEffects {
  private actions$ = inject(Actions);
  private userService = inject(UserService);

  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadUsers),
      exhaustMap(() =>
        this.userService.getUsers().pipe(
          map(users => loadUsersSuccess({ users })),
          catchError(error => of(loadUsersFailure({ error: error.message })))
        )
      )
    )
  );
}
```

---

## 17. Testing

### Component Testing

```typescript
describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;
  let userServiceSpy: jasmine.SpyObj<UserService>;

  beforeEach(async () => {
    userServiceSpy = jasmine.createSpyObj('UserService', ['getUsers']);
    userServiceSpy.getUsers.and.returnValue(of([{ id: 1, name: 'John' }]));

    await TestBed.configureTestingModule({
      imports: [UserComponent],
      providers: [{ provide: UserService, useValue: userServiceSpy }]
    }).compileComponents();

    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
  });

  it('should load users on init', () => {
    fixture.detectChanges();
    expect(userServiceSpy.getUsers).toHaveBeenCalled();
  });
});
```

### Service Testing

```typescript
describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  it('should fetch users', () => {
    const mockUsers = [{ id: 1, name: 'John' }];

    service.getUsers().subscribe(users => {
      expect(users).toEqual(mockUsers);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });
});
```

---

## 18. Common Interview Questions

| Question | Key Points |
|----------|------------|
| What is Angular? | TypeScript-based framework, component architecture, DI |
| Constructor vs ngOnInit? | Constructor: DI. ngOnInit: initialization with inputs |
| Observable vs Promise? | Observable: multiple values, cancellable, lazy |
| ViewChild vs ContentChild? | ViewChild: template. ContentChild: projected content |
| Pure vs Impure pipes? | Pure: on input change. Impure: every detection |
| Lazy loading benefits? | Smaller bundle, faster load, on-demand features |
| AOT vs JIT? | AOT: build time. JIT: runtime |
| Default vs OnPush? | Default: every cycle. OnPush: reference change only |

---

## Preparation Checklist

```
□ Components, Templates, Data Binding
□ Directives and Pipes
□ Services and Dependency Injection
□ RxJS Observables and Operators
□ Signals (Angular 16+)
□ Routing and Guards
□ Reactive Forms and Validation
□ HTTP Client and Interceptors
□ Change Detection
□ Standalone Components
□ Performance Optimization
□ NgRx State Management
□ Testing
```

---

Good luck with your interview! 🚀
