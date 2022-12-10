<!--
This README describes the package. If you publish this package to pub.dev,
this README's contents appear on the landing page for your package.

For information about how to write a good package README, see the guide for
[writing package pages](https://dart.dev/guides/libraries/writing-package-pages).

For general information about developing packages, see the Dart guide for
[creating packages](https://dart.dev/guides/libraries/create-library-packages)
and the Flutter guide for
[developing packages and plugins](https://flutter.dev/developing-packages).
-->

# pbp_django_auth

A Flutter package for helping students to implement authentication from Django web service in Flutter.

<!--## Features

List what your package can do. Maybe include images, gifs, or videos.-->

## Getting Started

### Django's Part

To use the package, you need to make asynchronous JavaScript (AJAX) login view in your Django project.

1. Run `python manage.py startapp authentication` to make a new app module for handling the AJAX login.
2. Add `"authentication"` to `INSTALLED_APPS` in `settings.py`.
3. Run `pip install django-cors-headers` to install the required library.
4. Add `"corsheaders"` to `INSTALLED_APPS` in `settings.py`.
5. Add `"corsheaders.middleware.CorsMiddleware"` to `MIDDLEWARE` in `settings.py`.
6. Create a new variable in `settings.py` called `CORS_ALLOW_ALL_ORIGINS` and set the value to `True` (`CORS_ALLOW_ALL_ORIGINS=True`).
7. Create a new variable in `settings.py` called `CORS_ALLOW_CREDENTIALS` and set the value to `True`, (`CORS_ALLOW_CREDENTIALS=True`).
8. Create the following variables in `settings.py`.

    ```python
    CSRF_COOKIE_SECURE = True
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SAMESITE = 'None'
    SESSION_COOKIE_SAMESITE = 'None'
    ```

9. Create a login view method in `authentication/views.py`.
  
    **Example Login View**

    ```python
    from django.shortcuts import render
    from django.contrib.auth import authenticate, login as auth_login
    from django.http import JsonResponse
    from django.views.decorators.csrf import csrf_exempt

    @csrf_exempt
    def login(request):
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(username=username, password=password)
        if user is not None:
            if user.is_active:
                auth_login(request, user)
                # Redirect to a success page.
                return JsonResponse({
                  "status": True,
                  "message": "Successfully Logged In!"
                  # Insert any extra data if you want to pass data to Flutter
                }, status=200)
            else:
                return JsonResponse({
                  "status": False,
                  "message": "Failed to Login, Account Disabled."
                }, status=401)

        else:
            return JsonResponse({
              "status": False,
              "message": "Failed to Login, check your email/password."
            }, status=401)
    ```

This view will set cookies to the user and allow authenticated requests with `@login_required` decorator.

### Flutter's Part

To use the package, modify application root widget to provide the `CookieRequest` library to all child widgets by using `Provider`.

For example, if the previous app initialization was:

```dart
class MyApp extends StatelessWidget {
    const MyApp({Key? key}) : super(key: key);
    
    @override
    Widget build(BuildContext context) {
        return MaterialApp(
            title: 'Flutter App',
            theme: ThemeData(
                primarySwatch: Colors.blue,
            ),
            home: const MyHomePage(title: 'Flutter App'),
            routes: {
                "/login": (BuildContext context) => const LoginPage(),
            },
        );
    }
}
```

Change it to:

```dart
class MyApp extends StatelessWidget {
    const MyApp({Key? key}) : super(key: key);

    @override
    Widget build(BuildContext context) {
        return Provider(
            create: (_) {
                CookieRequest request = CookieRequest();
                return request;
            },
            child: MaterialApp(
                title: 'Flutter App',
                theme: ThemeData(
                    primarySwatch: Colors.blue,
                ),
                home: const MyHomePage(title: 'Flutter App'),
                routes: {
                    "/login": (BuildContext context) => const LoginPage(),
                },
            ),
        );
    }
}
```

This creates a new `Provider` object that will share the `CookieRequest` instance with all components in the application.

## Usage

To use the package in your project, follow these steps below.

1. Import the `Provider` library and this package to the component.

    ```dart
    import 'package:provider/provider.dart';
    import 'package:pbp_django_auth/pbp_django_auth.dart';
    ...
    ```

2. Instantiate the `request` object by calling `context.watch` in the Widget `build(BuildContext context)` function.

    **Example**

    ```dart
    class _LoginPageState extends State<LoginPage> {
      final _loginFormKey = GlobalKey<FormState>();
      bool isPasswordVisible = false;
      void togglePasswordView() {
        setState(() {
          isPasswordVisible = !isPasswordVisible;
        });
      }

      String username = "";
      String password1 = "";
      @override
      Widget build(BuildContext context) {
        final request = context.watch<CookieRequest>();
        // The rest of your widgets are down below
        ...
      }
    }
    ```

3. To log in using the package, use the `request.login(url, data)` method.

    ```dart
      // 'username' and 'password' should be the values of the user login form.
      final response = await request.login("<DJANGO URL>/auth/login", {
        'username': username,
        'password': password1,
      });
      if (request.loggedIn) {
        // Code here will run if the login succeeded.
      } else {
        // Code here will run if the login failed (wrong username/password).
      }
    ```

4. To fetch or insert data using the library, use the `request.get(url)` or `request.post(url, data)` method.

    ```dart
    /* GET request example: */
    final response = await request.get(<URL TO ACCESS>);
    // The returned response will be a Map object with the keys of the JsonResponse
    
    /* POST request example: */
    final response = await request.post(<URL TO ACCESS>, {
      "data1": "THIS IS EXAMPLE DATA",
      "data2": "THIS IS EXAMPLE DATA 2",
    });
    // The data argument should be the keys of the Django form.
    // The returned response will be a Map obejct with the keys of JsonResponse.
    ```

    You can also use request.postJson(url, encodedJsonData) with jsonEncode function from 'dart:convert' library to send the submitted data without manually converting the data into JSON format one by one.

## Contributors

- [Adrian Ardizza](https://github.com/Meta502)
- [Muhammad Athallah](https://github.com/determinedguy)
- [Sabyna Maharani](https://github.com/sabynn)
- [Rendy](https://github.com/rorre)

<!--
https://blog.logrocket.com/how-to-create-dart-packages-for-flutter/
https://gist.github.com/Meta502/1605fdba3b141fbf67dba689e9e55498
-->
