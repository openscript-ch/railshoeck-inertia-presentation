---
theme: "@openscript-ch/slidev-theme"
title: Railshöck - Rails & Inertia
colorSchema: light
transition: none
---

# Railshöck - Rails & Inertia
Presented by openscript GmbH
---

## Was ist Inertia?

- SPA
- Serversteitiges Routing
- Keine API
- Client-Side Adapter für React, Vue und Svelte
- Server-Side Adapter für Rails, Laravel und Phoenix

<!-- Das Ziel von Inertia ist es einfach Single Page Applikation zu erstellen ohne die Einfachheit von MVC zu verlassen. Es gibt Serverseitiges Routing. Man muss keine API aufbauen und Instandhalten.
Dazu gibt es drei Offizielle Client-Side Adapter, für React, Vue und Svelte sowie drei Server-Side Adapter für Rails, Laravel und Phoenix. -->
---

## Wie kamen wir zu Inertia?

__Die Optionen__
- Plain old Rails
- Rails, Vite und React
- Rails, Vite SSR und React
- Rails, Vite, Inertia und React
- Rails und Hotwire

<!-- Wir hatten einige Diskussionen wie wir unser Projekt umsetzten wollten. Wir verwenden gerne neue und uns noch unbekannte Technologien. Jedoch für dieses Projekt wollten wir uns schon ein bekannteren Tech-Stack annehmen.
Somit ergaben sich für uns einige Optionen wobei Inertia eigentlich noch nicht zur Auswahl Stand. Über ein Laravel + Vue Projekt, welches wir übernahmen, sind wir auf Inertia gestossen. -->
---
layout: two-cols-header
---

## Inertia.ts Setup

::left::

__Inertia.ts__

```ts {all|2|5-11|13-15|all}
createInertiaApp({
  title: (title) => (title ? `${title} - ICT BLJ DocPortal` : "ICT BLJ DocPortal"),

  resolve: (name) => {
    const pages = import.meta.glob<ResolvedComponent>("../pages/**/*.tsx", {
      eager: true,
    });
    const page = pages[`../pages/${name}.tsx`];
    if (!page) {
      console.error(`Missing Inertia page component: '${name}.tsx'`);
    }

    if (page.default?.layout === undefined) {
      page.default.layout = (page: ReactNode) => AppShell({ children: page });
    }

    return page;
  }
});
```

::right::

<span :class="$slidev.nav.clicks === 2 ? 'highlight' : ''">**Home**</span>**.tsx**

```tsx {all|1,10-12|none|8-15|all}{at: +1}
import { Head, Link } from "@inertiajs/react";
import type { ReactNode } from "react";

const Home = ({ username }: { username: string }) => {
  return <h1>Welcome {username}</h1>;
};

Home.layout = (page: ReactNode) => (
  <>
    <Head>
      <title>Home</title>
    </Head>
    {page}
  </>
);

export default Home;

```

<!-- Das Grundsetup und die Installation überspringen wir da es gut dokumentiert ist und auch Projektspezifisch ist. Stattdessen zeigen wir eine Grundkonfiguration des Inertia.ts files. -->

---
layout: two-cols-header
---

## Controllers

::left::

__schools_controller.rb__

```rb {all|5|6-8|all}
  def index
    @schools = @filter.apply(School.all)
    @pagination, @records = pagy(@schools, page:)

    render inertia: "school/Schools", props: {
      schools: SchoolSerializer.render_as_hash(@records, view: :show_view),
      filter: @filter.attributes,
      pagination: PaginationSerializer.render_as_hash(@pagination)
    }
  end
```

::right::

<span :class="$slidev.nav.clicks === 1 ? 'highlight' : ''">**school/Schools**</span>**.tsx**

```tsx {all|10|1-10|all}{at: +1}
type SchoolsProps = {
  schools: School[];
  filter: {
    sort_key?: "bms" | "title" | "short";
    sort_direction?: "asc" | "desc";
  };
  pagination: Pagination;
};

export default function Schools({ schools, filter, pagination }: SchoolsProps) {
...
}
```
---

## Shorthand-Routing

__routes.rb__

```rb
Rails.application.routes.draw do
  inertia 'dashboard' => 'Dashboard'

  inertia :settings

  namespace :admin do
    inertia 'dashboard' => 'Admin/Dashboard'
  end

  resources :users do
    inertia :activity, on: :member
    inertia :statistics, on: :collection
  end
end
```
---
layout: two-cols-header
---

## Links

::left::

__Als Anchor__

```tsx {all|1,8|all}
import { Head, Link } from "@inertiajs/react";
import type { ReactNode } from "react";

const Home = ({ username }: { username: string }) => {
  return (
    <>
      <nav>
        <Link href="/">Home</Link>
      </nav>
      <h1>Welcome {username}</h1>
    </>
  );
};
```

::right::

<v-clicks>
<div>

**Als ein beliebiger Tag**
```tsx {all|8|all}{at: +4}
import { Head, Link } from "@inertiajs/react";
import type { ReactNode } from "react";

const Home = ({ username }: { username: string }) => {
  return (
    <>
      <nav>
        <Link href="/" as="button">Home</Link>
      </nav>
      <h1>Welcome {username}</h1>
    </>
  );
};
```

</div>
</v-clicks>

---

### Weitere optionen für Links
* method (`GET|POST|PUT|PATCH|DELETE`)
* data (POST, PUT data)
* headers
* replace (Browser History ersetzten statt hinzufügen)
* preserveState (nicht komplett neu rendern)
* preserveScroll
* only (Partial Reloading)
---

## Manuelles Routing

```ts
import { router } from '@inertiajs/react';

router.visit(url, options);
```
oder spezifischer
```ts
router.get(url, data, options);
router.post(url, data, options);
router.put(url, data, options);
router.patch(url, data, options);
router.delete(url, options);
router.reload(options);
```

__Beispiel__
```ts
router.post("/login", {
  email: "max.muster@example.org",
  password: "password"
});
```
---

## Event Callbacks

```ts
router.visit(url, {
  onBefore: (visit) => {},
  onStart: (visit) => {},
  onProgress: (progress) => {},
  onSuccess: (page) => {},
  onError: (errors) => {},
  onCancel: () => {},
  onFinish: (visit) => {}
});
```

__Beispiel__

```ts
router.post("/login",
    {
      email: "max.muster@example.org",
      password: "password",
    },
    {
      onSuccess: (page) => flashManager.push(State.SUCCESS, page.props.message),
    },
);
```
---

## Weitere Spielereien

__Browser History__
```ts
router.push(options);
router.replace(options)
```

__Request Abbrechen__

```ts
router.post('/users', data, {
  onCancelToken: (cancelToken) => this.cancelToken = cancelToken,
});

this.cancelToken.cancel();
```
---

## Restliche Optionen für den Router

```ts
router.visit(url, {
  method: 'get',
  data: {},
  replace: false,
  preserveState: false,
  preserveScroll: false,
  only: [],
  except: [],
  headers: {},
  errorBag: null,
  forceFormData: false,
  queryStringArrayFormat: 'brackets',
  async: false,
  showProgress: true,
  fresh: false,
  reset: [],
  preserveUrl: false,
  prefetch: false,
  onPrefetching: () => {},
  onPrefetched: () => {},
});
```
---

## JS-Routes

```bash
rake js:routes
```

__AdminCreate.tsx__

```ts
import { admins_path } from "../../../routes";
import { type RequestPayload, router } from "@inertiajs/core";

export default function AdminCreate() {
  const onSubmit = (values: RequestPayload) => router.post(admins_path(), values);
}
```
---

## Generierte Routes

__routes.d.ts__
```ts
/**
 * Generates rails route to
 * /admins(.:format)
 * @param {object | undefined} options
 * @returns {string} route path
 */
export const admins_path: ((
  options?: {format?: OptionalRouteParameter} & RouteOptions
) => string) & RouteHelperExtras;
```

---
layout: two-cols-header
---

## Client-Side Partial Loading

::left::

```ts{all|2|all}
router.reload({
  only: ["groups"],
  data: { 
    selected_year_context_id: yearContextId,
    selected_occupation: occupation 
  },
});
```

::right::

```rb{all|3|all}{at: +1}
render inertia: "users/apprentice/ApprenticeCreate", props: {
  ...
  groups: GroupSerializer.render_as_hash(@groups || [])
  ...
}
```
---

### Client-Side Partial Loading

__Exkludieren__
```ts
router.reload({
  except: ["groups"]
});
```

__Als Link__

```tsx
<Link href="/groups" only={['groups']}>Gruppen Anzeigen</Link>
```

__Oder nur bestimmte Properties__
```ts
router.reload({
  only: ["user.firstname", "user.lastname"],
});
```
---
layout: two-cols-header
---

## Server-Side Partial Loading

::left::

```rb{all|4|all}
render inertia: "report/Reports", props: {
  ...
  groups: InertiaRails.optional do
    GroupSerializer.render_as_hash(@groups)
  end,
  ...
}
```

::right::

```ts{all|2|all}{at: +1}
const openReportGenerator = () => {
  router.reload({ only: ["groups"] });
  openGenerator();
};

```
---

## Server-Side Partial Loading

__Alle Optionen__

```rb
# IMMER bei standard besuchen dabei
# OPTIONAL bei partiellen reloads
# IMMER evaluiert
users: User.all,

# IMMER bei standard besuchen dabei
# OPTIONAL bei partiellen reloads
# NUR evaluiert wenn benötig
users: -> { User.all },

# NIE bei standard besuchen dabei
# OPTIONAL bei partiellen reloads
# NUR evaluiert wenn benötig
users: InertiaRails.optional { User.all },

# IMMER bei standard esuchen dabei
# IMMER bei partiellen reloads
# IMMER evaluiert
users: InertiaRails.always { User.all },
```
layout: two-cols-header
---

## Props & Validation Errors

::left::

__application_controller.rb__

```rb{all|6-7|all}
def create
  ...
  if admin.save
    redirect_to admins_path
  else
    redirect_to new_admin_path,
     inertia: { errors: admin.errors }
  end
end
```

::right::

__AdminForm.tsx__

```ts{all|1,3-6,9-10|all}{at: +1}
import { usePage } from "@inertiajs/react";

type FormErrors = {
  first_name?: string[];
  last_name?: string[];
};

export function AdminForm() {
  const { errors } = 
    usePage<{ errors: FormErrors }>().props;
}
``` 

---
layout: two-cols-header
---

## Shared data

::left::

__application_controller.rb__

```rb{all|2-8|2-8|all}
class ApplicationController < ActionController::Base
 inertia_share do
    {
      flash: InertiaRails.always do
       flash&.to_hash.presence || nil
      end
    }
  end
end
```

::right::

__admins_controller.rb__

```rb{all|4|9-10|all}{at: +1}
def create
  ...
  if admin.save
    flash[:notice] = "Admin successfully created"
    redirect_to admins_path

    # oder

    redirect_to admins_path,
     flash: { notice: "Admin successfully created" }
  else
    redirect_to new_admin_path, inertia: { errors: admin.errors }
  end
end
```
---
layout: two-cols-header
---

## Deferred Props

::left::

__items_controller.rb__
```rb{all|2|all}
render inertia: "Items/Items", props: {
      items: InertiaRails.defer { @items },
      params: @params
}
```

::right::

__Items.tsx__
```tsx{all|1,5-9|all}{at: +1}
import { Deferred } from "@inertiajs/react";

export default function Items({ items, params }: ItemsProps) {
  return (
    <Deferred data="items" 
      fallback={<ItemsTable loading />}>
      <ItemsTable items={items}
       onDelete={deleteItem}/>
    </Deferred>
  )
}
```
---

<div class="flex justify-center">
  <SlidevVideo controls class="w-[80%] h-auto">
    <source src="./assets/DeferredPropsDemo.mp4" type="video/mp4" />

     <p>
      Your browser does not support videos. You may download it
      <a href="./assets/DeferredPropsDemo.mp4">here</a>.
    </p>
  </SlidevVideo>
</div>
---
---

## Deferred Props

__Gruppierung für mehrere parallele Requests__

```rb{all|2-4|all}
render inertia: "Items/Items", props: {
      items: InertiaRails.defer { @items },
      params: InertiaRails.defer('attributes') { @params },
      filters: InertiaRails.defer('attributes') { @filters },
}
```
---
layout: two-cols-header
---

## Merging Props

::left::

__items_controller.rb__

```rb{all|2-4,6}
def index2
  Item.offset(params["offset"] || 0)
      .limit(20)
      .order(:id)
  render inertia: "Items/ItemsIndex2", props: {
    items: InertiaRails.merge { @items.as_json() },
  }
end
```

::right::

__Items.tsx__

```tsx{all|3-5|all}{at: +1}
const loadMoreRecords = () => {
  setLoading(true);
  router.reload({
    only: ["items"],
    data: { offset: items.length },
    onFinish: () => setLoading(false)
  });
}
```
---

<div class="flex justify-center">
  <SlidevVideo controls class="w-[80%] h-auto">
    <source src="./assets/MergePropsDemo.mp4" type="video/mp4" />

     <p>
      Your browser does not support videos. You may download it
      <a href="./assets/MergePropsDemo.mp4">here</a>.
    </p>
  </SlidevVideo>
</div>

---

## Merging Props

__Merge zurücksetzten__

```tsx{all|4-5|all}
const onReset = () => {
  setLoading(true)
  router.reload({
    reset: ["items"],
    data: { offset: 0 },
    onFinish: () => setLoading(false)
  });
}
```