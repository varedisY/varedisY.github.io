---
layout: distill
title: Getting started with Riverpod in Flutter
date: 2024-09-04 10:00:00
description: A beginner's guide to state management with Riverpod in Flutter.
tags: flutter, riverpod, state, dart
featured: false
toc:
  - name: Introduction
  - name: Installing Riverpod
  - name: Creating a Provider
  - name: Consuming Providers
  - name: Network Requests with Riverpod
  - name: Conclusion
---

## Introduction

Riverpod is a powerful state management solution for Flutter, providing a clean, testable, and scalable way to manage state across your app. In this guide, we'll walk you through getting started with Riverpod, from installation to creating and consuming providers, and even making network requests using Riverpod.

## Installing Riverpod

To get started with Riverpod, you first need to add the required dependencies to your `pubspec.yaml`. Here's how you can do that:

### Adding dependencies

Open your terminal and run the following commands to install the necessary packages:

```bash
flutter pub add flutter_riverpod
flutter pub add riverpod_annotation
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner
flutter pub add dev:custom_lint
flutter pub add dev:riverpod_lint
```

Alternatively, you can manually add them to your `pubspec.yaml` file:

```yaml
name: my_app_name
environment:
  sdk: ">=3.0.0 <4.0.0"
  flutter: ">=3.0.0"

dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5

dev_dependencies:
  build_runner:
  custom_lint:
  riverpod_generator: ^2.4.3
  riverpod_lint: ^2.3.13
```

After adding the dependencies, run:

```bash
flutter pub get
```

This will install all required packages.

Next, to generate code with Riverpod, run the following command:

```bash
flutter pub run build_runner watch
```

## Creating a Provider

Now that you have Riverpod installed, let's create your first provider. Providers in Riverpod store a value that can be accessed and updated throughout the app.

Here's an example of a simple provider that returns a string:

```dart
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'main.g.dart';

@riverpod
String helloWorld(HelloWorldRef ref) {
  return 'Hello world';
}
```

The `@riverpod` annotation enables code generation, creating the `helloWorldProvider` for you.

## Consuming Providers

To use the provider in your Flutter UI, you need to wrap your widget tree with a `ProviderScope`. This is where Riverpod stores the state for all your providers:

```dart
void main() {
  runApp(
    ProviderScope(
      child: MyApp(),
    ),
  );
}
```

Inside your widgets, you can consume the provider using the `ref.watch()` method:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final String value = ref.watch(helloWorldProvider);

    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Riverpod Example')),
        body: Center(
          child: Text(value),
        ),
      ),
    );
  }
}
```

Running this app will display "Hello world" on the screen.

## Network Requests with Riverpod

Riverpod also simplifies making network requests. Let's make a network request using a provider to fetch data from an API.

First, define a model for the response data:

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'activity.freezed.dart';
part 'activity.g.dart';

@freezed
class Activity with _$Activity {
  factory Activity({
    required String activity,
    required String type,
    required int participants,
    required double price,
  }) = _Activity;

  factory Activity.fromJson(Map<String, dynamic> json) => _$ActivityFromJson(json);
}
```

Next, create a provider that makes the HTTP request:

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import 'package:riverpod_annotation/riverpod_annotation.dart';
import 'activity.dart';

part 'provider.g.dart';

@riverpod
Future<Activity> activity(ActivityRef ref) async {
  final response = await http.get(Uri.https('www.boredapi.com', '/api/activity'));
  final json = jsonDecode(response.body) as Map<String, dynamic>;
  return Activity.fromJson(json);
}
```

Finally, consume this provider in your UI:

```dart
class Home extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final activity = ref.watch(activityProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Random Activity')),
      body: Center(
        child: activity.when(
          data: (data) => Text(data.activity),
          loading: () => const CircularProgressIndicator(),
          error: (err, stack) => Text('Error: $err'),
        ),
      ),
    );
  }
}
```

With this setup, Riverpod manages network requests and state changes efficiently.

## Conclusion

Riverpod provides a scalable, testable solution for state management in Flutter, making it easy to handle everything from simple values to asynchronous network requests. In this guide, we covered installing Riverpod, creating providers, consuming them, and making network requests. Now you're ready to explore the full power of Riverpod in your Flutter applications!
