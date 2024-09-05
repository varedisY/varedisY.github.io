---
layout: distill
title: Integrating Riverpod with Firebase Authentication in Flutter
date: 2024-09-04 12:00:00
description: Learn how to manage Firebase Authentication in Flutter using Riverpod and StreamNotifier.
tags: flutter, riverpod, firebase-auth, authentication
featured: false
toc:
  - name: Introduction
  - name: Setting up FirebaseAuth Provider
  - name: Managing Authentication State
  - name: Signing In and Out
  - name: Handling Errors
  - name: Conclusion
---

## Introduction

Firebase Authentication is a powerful tool for managing user sign-ins in a Flutter app. This guide demonstrates how to integrate Firebase Authentication with Riverpod, leveraging the `StreamNotifier` API for managing authentication state in a structured and reactive way.

This article assumes that you already have Firebase installed and configured in your Flutter project, and you have a basic understanding of Riverpod and its provider system.

## Setting up FirebaseAuth Provider

First, we need a Riverpod provider to expose the `FirebaseAuth` instance to the app. This instance will be used to access the authentication features provided by Firebase.

```dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'auth_provider.g.dart';

@riverpod
FirebaseAuth firebaseAuth(FirebaseAuthRef ref) {
  return FirebaseAuth.instance;
}
```

This `firebaseAuthProvider` makes the `FirebaseAuth` instance available to the rest of the app.

## Managing Authentication State

We’ll now manage the user’s authentication state. `FirebaseAuth` provides a stream that notifies us when the authentication state changes. We can wrap this stream in a `StreamNotifier` to update the state reactively within our app.

```dart
import 'package:firebase_auth/firebase_auth.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'auth_state_provider.g.dart';

@riverpod
class AuthNotifier extends _$AuthNotifier {

  @override
  Stream<User?> build() {
    final firebaseAuth = ref.watch(firebaseAuthProvider);
    return firebaseAuth.authStateChanges();
  }

  Future<void> signIn(String email, String password) async {
    final firebaseAuth = ref.read(firebaseAuthProvider);
    await firebaseAuth.signInWithEmailAndPassword(email: email, password: password);
  }

  Future<void> signOut() async {
    final firebaseAuth = ref.read(firebaseAuthProvider);
    await firebaseAuth.signOut();
  }
}
```

The `AuthNotifier` here automatically updates the state when Firebase authentication events occur. It also provides `signIn` and `signOut` methods that allow users to authenticate via Firebase.

Now that we have a reactive provider that updates the authentication state automatically, we can use it in the UI.

## Signing In and Out

Let's now use our `AuthNotifier` to sign in and out. We will build a sign-in page where the user can input their credentials and trigger the `signIn` method. Similarly, we can add a button to sign out.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class SignInPage extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final authNotifier = ref.read(authNotifierProvider.notifier);
    final authState = ref.watch(authNotifierProvider);

    return Scaffold(
      appBar: AppBar(title: Text('Sign In')),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            if (authState.value != null) ...[
              Text('Logged in as: ${authState.value?.email}'),
              ElevatedButton(
                onPressed: () async {
                  await authNotifier.signOut();
                  ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Signed out!')));
                },
                child: Text('Sign Out'),
              ),
            ] else ...[
              TextField(
                controller: TextEditingController(),
                decoration: InputDecoration(labelText: 'Email'),
              ),
              TextField(
                controller: TextEditingController(),
                decoration: InputDecoration(labelText: 'Password'),
                obscureText: true,
              ),
              ElevatedButton(
                onPressed: () async {
                  try {
                    await authNotifier.signIn('email@example.com', 'password123');
                    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Signed in!')));
                  } catch (e) {
                    ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Sign in failed: $e')));
                  }
                },
                child: Text('Sign In'),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

### Explanation:

- The `authState.value` checks whether the user is authenticated or not. If the user is signed in, it displays their email and a sign-out button. If not, it shows the sign-in form.
- The `signIn` and `signOut` methods are accessed through the `authNotifier`, which manages the authentication state.
  
## Handling Errors

To provide feedback in case of an error, we can catch exceptions thrown by Firebase's authentication methods, such as incorrect passwords or missing user accounts, and display them to the user.

Here’s an example of how we can handle errors during sign-in:

```dart
ElevatedButton(
  onPressed: () async {
    try {
      await ref.read(authNotifierProvider.notifier).signIn('email@example.com', 'password123');
    } on FirebaseAuthException catch (e) {
      String errorMessage;
      switch (e.code) {
        case 'user-not-found':
          errorMessage = 'No user found for that email.';
          break;
        case 'wrong-password':
          errorMessage = 'Wrong password provided.';
          break;
        default:
          errorMessage = 'Something went wrong. Please try again.';
      }
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(errorMessage)));
    }
  },
  child: Text('Sign In'),
);
```

This pattern allows us to catch Firebase-specific exceptions and provide the user with actionable feedback.

## Conclusion

In this article, we demonstrated how to integrate Firebase Authentication with Riverpod using the `StreamNotifier` API. This approach provides a reactive and efficient way to manage authentication state in a Flutter app. We covered how to set up the FirebaseAuth provider, manage the user's authentication state, handle sign-in and sign-out functionality, and show meaningful error messages to users.

By combining the power of Firebase and Riverpod, you can build secure, scalable, and maintainable authentication flows in your Flutter applications.

