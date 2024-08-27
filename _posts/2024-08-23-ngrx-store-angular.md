---
layout: distill
title: Getting Started with NgRx SignalStore
date: 2024-08-22 12:00:00
description: Explore how NgRx SignalStore simplifies state management in Angular with its native support for Signals, and learn how to create, provide, and use SignalStores effectively.
tags: angular, ngrx, state
categories: angular
featured: false
toc:
  - name: Introduction
  - name: Prerequisites
  - name: Creating a Store
  - name: Providing and Injecting the Store
  - name: Reading State
  - name: Defining Computed Signals
  - name: Defining Store Methods
  - name: Putting It All Together
  - name: Managing Entities
---

## Introduction

NgRx SignalStore is a powerful state management solution for Angular applications. It leverages Signals to create stores in a clear, declarative manner, providing flexibility and ease of use.

In this guide, we'll explore how to create and use SignalStores, manage state and define computed signals and methods.


## Prerequisites



   - **Install NgRx Signals:**

     Run the following command to install the core NgRx signals libraries:

     ```bash
     ng add @ngrx/signals@latest
     ```

 


## Creating a Store

A SignalStore is created using the `signalStore` function, which takes a series of store features as arguments. This allows the SignalStore to manage state, computed signals, and methods, making the implementation flexible and extensible. Depending on the features used, `signalStore` returns an injectable service that can be provided and injected wherever needed.

To add state slices to a SignalStore, use the `withState` feature. This feature accepts the initial state, which must be an object literal.

Here’s an example:

```typescript
{% raw %}

import { signalStore, withState } from '@ngrx/signals';
import { Book } from './book.model';

type BooksState = {
  books: Book[];
  isLoading: boolean;
  filter: { query: string; order: 'asc' | 'desc' };
};

const initialState: BooksState = {
  books: [],
  isLoading: false,
  filter: { query: '', order: 'asc' },
};

export const BooksStore = signalStore(
  withState(initialState)
);
{% endraw %}

```

In this example, the `BooksStore` instance will automatically generate signals for each state slice, including nested properties.


## Providing and Injecting the Store

SignalStore can be provided either locally or globally. By default, a SignalStore is not registered with any injector, so it needs to be included in a providers array at the component, route, or root level.

Here’s an example of providing `BooksStore` at the component level:

```typescript
{% raw %}

import { Component, inject } from '@angular/core';
import { BooksStore } from './books.store';

@Component({
  providers: [BooksStore],
})
export class BooksComponent {
  readonly store = inject(BooksStore);
}
{% endraw %}

```

When provided at the component level, the store is tied to the component’s lifecycle, making it ideal for managing component-specific state. Alternatively, you can register a SignalStore globally by setting the `providedIn` property to `'root'`:

```typescript
{% raw %}

export const BooksStore = signalStore(
  { providedIn: 'root' },
  withState(initialState)
);
{% endraw %}

```

## Reading State

Signals generated for state slices can be used to access state values:

```typescript
{% raw %}
import { ChangeDetectionStrategy, Component, inject } from '@angular/core';
import { JsonPipe } from '@angular/common';
import { BooksStore } from './books.store';

@Component({
  standalone: true,
  imports: [JsonPipe],
  template: `
    <p>Books: {{ store.books() | json }}</p>
    <p>Loading: {{ store.isLoading() }}</p>
    <p>Pagination: {{ store.filter() | json }}</p>
    <p>Query: {{ store.filter.query() }}</p>
    <p>Order: {{ store.filter.order() }}</p>
  `,
  providers: [BooksStore],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class BooksComponent {
  readonly store = inject(BooksStore);
}
{% endraw %}
```

## Defining Computed Signals

You can add computed signals to the store using the `withComputed` feature, which takes a factory function that returns a dictionary of computed signals:

```typescript
{% raw %}

import { computed } from '@angular/core';
import { signalStore, withComputed, withState } from '@ngrx/signals';
import { Book } from './book.model';

export const BooksStore = signalStore(
  withState(initialState),
  withComputed(({ books, filter }) => ({
    booksCount: computed(() => books().length),
    sortedBooks: computed(() => {
      const direction = filter.order() === 'asc' ? 1 : -1;

      return books().toSorted((a, b) =>
        direction * a.title.localeCompare(b.title)
      );
    }),
  }))
);
{% endraw %}

```

## Defining Store Methods

Methods can be added to the store using the `withMethods` feature, which takes a factory function that returns a dictionary of methods:

```typescript
{% raw %}

import { patchState } from '@ngrx/signals';

export const BooksStore = signalStore(
  withState(initialState),
  withComputed(/* ... */),
  withMethods((store) => ({
    updateQuery(query: string): void {
      patchState(store, (state) => ({ filter: { ...state.filter, query } }));
    },
    updateOrder(order: 'asc' | 'desc'): void {
      patchState(store, (state) => ({ filter: { ...state.filter, order } }));
    },
  }))
);
{% endraw %}

```

State updates are performed using the `patchState` function. This function ensures that the state is protected from external modifications unless explicitly configured otherwise.


## Putting It All Together

Here’s the complete implementation of `BooksStore`, integrating state, computed signals, and methods:

```typescript
{% raw %}
import { patchState, signalStore, withComputed, withMethods, withState } from '@ngrx/signals';
import { Book } from './book.model';
import { BooksService } from './books.service';

export const BooksStore = signalStore(
  withState(initialState),
  withComputed(({ books, filter }) => ({
    booksCount: computed(() => books().length),
    sortedBooks: computed(() => {
      const direction = filter.order() === 'asc' ? 1 : -1;

      return books().toSorted((a, b) =>
        direction * a.title.localeCompare(b.title)
      );
    }),
  })),
  withMethods((store, booksService = inject(BooksService)) => ({
    updateQuery(query: string): void {
      patchState(store, (state) => ({ filter: { ...state.filter, query } }));
    },
    updateOrder(order: 'asc' | 'desc'): void {
      patchState(store, (state) => ({ filter: { ...state.filter, order } }));
    },

  }))
);
{% endraw %}

```

## Managing Entities
To learn how to manage entities in the ngrx signal store, check out our [Entity Management with NgRx SignalStore: Streamlining State Management in Angular](https://varedisy.github.io/blog/2024/ngrx-store-entities-angular/).

---

This comprehensive setup makes the `BooksStore` a powerful tool for managing application state in Angular, with the added benefit of seamless integration with RxJS for handling more complex, asynchronous tasks.