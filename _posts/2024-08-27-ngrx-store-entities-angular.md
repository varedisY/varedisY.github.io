---
layout: distill
title: 'Entity Management with NgRx SignalStore: Streamlining State Management in Angular'
date: 2024-08-27 12:00:00
description: Learn how to efficiently manage entity collections in Angular using NgRx SignalStore.
tags: angular, ngrx, state
categories: angular
featured: false
toc:
  - name: Introduction to Entity Management
  - name: Using withEntities Feature
  - name: Leveraging Entity Updaters
  - name: Custom Entity Identifiers
  - name: Named Entity Collections
---

## Introduction to Entity Management

Managing entities efficiently is a crucial aspect of state management in any application. NgRx SignalStore offers an elegant solution for handling entity collections through the `@ngrx/signals/entities` plugin. This plugin simplifies the management of entities, allowing you to focus on the logic and features of your Angular application without getting bogged down by the intricacies of state management. 

If you have never used NgRx SignalStore before, refer to the article [Getting Started with NgRx SignalStore](https://varedisy.github.io/blog/2024/ngrx-store-angular/).

## Using `withEntities` Feature

The `withEntities` feature is the cornerstone of entity management in SignalStore. It automatically generates signals that provide access to entity collections, including:

- **ids**: A signal that returns an array of all entity IDs.
- **entityMap**: A signal that returns a map of entities keyed by their IDs.
- **entities**: A signal that returns an array of all entities, making it easy to work with the entire collection.

Here’s an example of how to define a basic SignalStore with entity management:

```typescript
import { computed } from '@angular/core';
import { signalStore } from '@ngrx/signals';
import { withEntities } from '@ngrx/signals/entities';

type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

export const TodosStore = signalStore(
  withEntities<Todo>()
);
```

This setup gives you access to entity-specific signals, such as `ids`, `entityMap`, and `entities`, which can be used to manage and display entities in your application.

## Leveraging Entity Updaters

Entity updaters are functions provided by the `@ngrx/signals/entities` plugin to manipulate the entity collections within your store. They work seamlessly with the `patchState` function to ensure the store is updated predictably.

Here’s an example of how to add methods to the `TodosStore` for common operations:

```typescript
import { patchState, signalStore, withMethods } from '@ngrx/signals';
import {
  addEntity,
  removeEntities,
  updateAllEntities,
  withEntities,
} from '@ngrx/signals/entities';

type Todo = {
  id: number;
  text: string;
  completed: boolean;
};

export const TodosStore = signalStore(
  withEntities<Todo>(),
  withMethods((store) => ({
    addTodo(todo: Todo): void {
      patchState(store, addEntity(todo));
    },
    removeEmptyTodos(): void {
      patchState(store, removeEntities(({ text }) => !text));
    },
    completeAllTodos(): void {
      patchState(store, updateAllEntities({ completed: true }));
    },
  }))
);
```

## Custom Entity Identifiers

If your entities don’t have an `id` property, you can use a custom identifier by providing a selector function. This selector will be used whenever you add, update, or remove entities from the store.

```typescript
import { patchState, signalStore, withMethods } from '@ngrx/signals';
import {
  addEntities,
  removeEntity,
  SelectEntityId,
  setEntity,
  updateAllEntities,
  withEntities,
} from '@ngrx/signals/entities';

type Todo = {
  key: number;
  text: string;
  completed: boolean;
};

const selectId: SelectEntityId<Todo> = (todo) => todo.key;

export const TodosStore = signalStore(
  withEntities<Todo>(),
  withMethods((store) => ({
    addTodos(todos: Todo[]): void {
      patchState(store, addEntities(todos, { selectId }));
    },
    setTodo(todo: Todo): void {
      patchState(store, setEntity(todo, { selectId }));
    },
    completeAllTodos(): void {
      patchState(
        store,
        updateAllEntities({ completed: true }, { selectId })
      );
    },
    removeTodo(key: number): void {
      patchState(store, removeEntity(key));
    },
  }))
);
```

## Named Entity Collections

The `withEntities` feature allows for managing multiple entity collections within a single store by specifying a custom prefix for each collection. This is particularly useful when dealing with related data types.

```typescript
import { signalStore, type } from '@ngrx/signals';
import { withEntities } from '@ngrx/signals/entities';

type Book = {
  id: number;
  title: string;
  authorId: number;
};

type Author = {
  id: number;
  name: string;
};

export const LibraryStore = signalStore(
  withEntities({ entity: type<Book>(), collection: 'book' }),
  withEntities({ entity: type<Author>(), collection: 'author' }),
  withMethods((store) => ({
    addBook(book: Book): void {
      patchState(store, addEntity(book, { collection: 'book' }));
    },
    addAuthor(author: Author): void {
      patchState(store, addEntity(author, { collection: 'author' }));
    },
  }))
);
```
This approach allows you to maintain multiple collections within a single store while keeping their states isolated and manageable.



