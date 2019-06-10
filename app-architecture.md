<https://developer.android.com/jetpack/docs/guide>



# Guide to app architecture

This guide encompasses best practices and recommended architecture for building robust, production-quality apps.

This page assumes a basic familiarity with the Android Framework. If you are new to Android app development, check out our [Developer guides](https://developer.android.com/guide) to get started and learn more about the concepts mentioned in this guide.

## Mobile app user experiences

In the majority of cases, **desktop apps have a single entry point from a desktop or program launcher, then run as a single, monolithic process**. Android apps, on the other hand, have a much more complex structure. A typical Android app contains multiple app components, including `activities, fragments, services, content providers, and broadcast receivers`.

You declare most of these app components in your **app manifest**. The Android OS then uses this file to decide how to integrate your app into the device's overall user experience. Given that a properly-written Android app contains multiple components and that users often interact with multiple apps in a short period of time, apps need to adapt to different kinds of user-driven workflows and tasks.

For example, consider what happens when you share a photo in your favorite social networking app:

1. The app triggers a camera intent. The Android OS then launches a camera app to handle the request. At this point, the user has left the social networking app, but their experience is still seamless.
2. The camera app might trigger other intents, like launching the file chooser, which may launch yet another app.
3. Eventually, the user returns to the social networking app and shares the photo.

At any point during the process, the user could be **interrupted** by a phone call or notification. After acting upon this interruption, the user expects to be able to return to, and **resume**, this photo-sharing process. This app-hopping behavior is common on mobile devices, so your app must handle these flows correctly.

Keep in mind that mobile devices are also resource-constrained, so at any time, the operating system might kill some app processes to make room for new ones.

Given the conditions of this environment, it's possible for your app components to be launched individually and out-of-order, and the operating system or user can destroy them at any time. Because these events aren't under your control, **you shouldn't store any app data or state in your app components**, and your app components shouldn't depend on each other.

## Common architectural principles

If you shouldn't use app components to store app data and state, how should you design your app?

### Separation of concerns

The most important principle to follow is **separation of concerns**. It's a common mistake to write all your code in an `Activity` or a `Fragment`. These UI-based classes should only contain logic that handles UI and operating system interactions. By keeping these classes as lean as possible, you can avoid many lifecycle-related problems.

Keep in mind that you don't *own* implementations of `Activity` and `Fragment`; rather, these are just glue classes that represent the contract between the Android OS and your app. The OS can destroy them at any time based on user interactions or because of system conditions like low memory. To provide a satisfactory user experience and a more manageable app maintenance experience, it's best to minimize your dependency on them.

### Drive UI from a model

Another important principle is that you should **drive your UI from a model**, preferably a persistent model. *Models* are components that are responsible for handling the data for an app. They're independent from the `View` objects and app components in your app, so they're unaffected by the app's lifecycle and the associated concerns.

Persistence is ideal for the following reasons:

- Your users don't lose data if the Android OS destroys your app to free up resources.
- Your app continues to work in cases when a network connection is flaky or not available.

By basing your app on model classes with the well-defined responsibility of managing the data, your app is more testable and consistent.

## Recommended app architecture

In this section, we demonstrate how to structure an app using [Architecture Components](https://developer.android.com/jetpack/#architecture-components) by working through an end-to-end use case.

**Note:** It's impossible to have one way of writing apps that works best for every scenario. That being said, this recommended architecture is a good starting point for most situations and workflows. If you already have a good way of writing Android apps that follows the **common architectural principles**, you don't need to change it.

Imagine we're building a UI that shows a user profile. We use a private backend and a REST API to fetch the data for a given profile.

### Overview

To start, consider the following diagram, which shows how all the modules should interact with one another after designing the app:

![app-arch](images/app-final-architecture.png)

Notice that each component depends only on the component one level below it. For example, activities and fragments depend only on a view model. The repository is the only class that depends on multiple other classes; in this example, the repository depends on a persistent data model and a remote backend data source.

This design creates a consistent and pleasant user experience. Regardless of whether the user comes back to the app several minutes after they've last closed it or several days later, they instantly see a user's information that the app persists locally. If this data is stale, the app's repository module starts updating the data in the background.