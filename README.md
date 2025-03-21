# Http

## How to connect Angular app to backend

Firstly we have to register httpClient provider in main.ts

```
import { provideHttpClient } from '@angular/common/http';

bootstrapApplication(AppComponent, {
    providers: [provideHttpClient()]
}).catch((err) => console.error(err));
```

Then we can inject httpclientservice in component `private httpClient = inject(HttpClient);`

Now we can call http methods to send a requests

```
export class AvailablePlacesComponent implements OnInit {
  places = signal<Place[] | undefined>(undefined);
  private httpClient = inject(HttpClient);
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    const subscription = this.httpClient
      .get<{ places: Place[] }>('http://localhost:3000/places')
      .subscribe({
        next: (resData) => {
          console.log(resData.places);
        },
      });

    this.destroyRef.onDestroy(() => {
      subscription.unsubscribe();
    });
  }
}
```

**We must subscribe to trigger get request, as get method returns an observable, which will be ignored if no subscription added**

<hr>

You can add config to get methods, for example to get the whole response instead of body:

```
const subscription = this.httpClient
      .get<{ places: Place[] }>('http://localhost:3000/places', {
        observe: 'response'
      })
      .subscribe({
        next: (response) => {
          console.log(response);
          console.log(response.body?.places);
        },
      });
```

<hr>

To use response in template, we can save response data into signal:

```
export class AvailablePlacesComponent implements OnInit {
  places = signal<Place[] | undefined>(undefined);
  private httpClient = inject(HttpClient);
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    const subscription = this.httpClient
      .get<{ places: Place[] }>('http://localhost:3000/places')
      .subscribe({
        next: (resData) => {
          this.places.set(resData.places);
        },
      });

    this.destroyRef.onDestroy(() => {
      subscription.unsubscribe();
    });
  }
}
```

We can also use pipes to transform response data into something

```
export class AvailablePlacesComponent implements OnInit {
  places = signal<Place[] | undefined>(undefined);
  private httpClient = inject(HttpClient);
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    const subscription = this.httpClient
      .get<{ places: Place[] }>('http://localhost:3000/places')
      .pipe(
        map((resData) => resData.places)
      )
      .subscribe({
        next: (places) => {
          this.places.set(places);
        },
      });

    this.destroyRef.onDestroy(() => {
      subscription.unsubscribe();
    });
  }
}
```

Adding a loading fallback:

```
export class AvailablePlacesComponent implements OnInit {
  places = signal<Place[] | undefined>(undefined);
  isFetching = signal(false);
  private httpClient = inject(HttpClient);
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.isFetching.set(true);
    const subscription = this.httpClient
      .get<{ places: Place[] }>('http://localhost:3000/places')
      .subscribe({
        next: (resData) => {
          this.places.set(resData.places);
        },
        complete: () => {
          this.isFetching.set(false);
        }
      });

    this.destroyRef.onDestroy(() => {
      subscription.unsubscribe();
    });
  }
}
```

In template

```
@if (isFetching()) {
    <p class="fallback-text">Fetching available places...</p>
  }
```

## Error handling

```
export class AvailablePlacesComponent implements OnInit {
  places = signal<Place[] | undefined>(undefined);
  isFetching = signal(false);
  error = signal('');
  private httpClient = inject(HttpClient);
  private destroyRef = inject(DestroyRef);

  ngOnInit() {
    this.isFetching.set(true);
    const subscription = this.httpClient
      .get<{ places: Place[] }>('http://localhost:3000/places')
      .pipe(
        catchError((error) => {
          console.log(error);
          return throwError(
            () =>
              new Error(
                'Something went wrong fetching the available places. Please try again later.'
              )
          );
        })
      )
      .subscribe({
        next: (resData) => {
          this.places.set(resData.places);
        },
        error: (error) => {
          console.log(error);
          this.error.set(error.message);
        },
        complete: () => {
          this.isFetching.set(false);
        },
      });

    this.destroyRef.onDestroy(() => {
      subscription.unsubscribe();
    });
  }
}
```

## Sending data to backend

```
onSelectPlace(selectedPlace: Place) {
    this.httpClient.put('http://localhost:3000/user-places', {
      placeId: selectedPlace.id
    }).subscribe({
      next: (resData) => console.log(resData),
    });
  }
```

In the template

```
@if (places()) {
  <app-places [places]="places()! " (selectPlace)="onSelectPlace($event)" />
  } @else if (places()?.length === 0) {
  <p class="fallback-text">Unfortunately, no places could be found.</p>
  }
```
