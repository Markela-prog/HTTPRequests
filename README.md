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
