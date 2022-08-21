# Testing an Angular Component

## Learning Goals

- Test Angular components.

## Introduction

So far our Jasmine tests have been generic and haven't had any context from our
Angular application. In order to start testing actual components, we need to
give our test some Angular context, which is what the `TestBed` class is for.

Let's have a look at what our header component test looks like when it's updated
to actually test our component, and then we'll break down the code:

```typescript
import { ComponentFixture, TestBed } from "@angular/core/testing";

import { HeaderComponentComponent } from "./header-component.component";

describe("HeaderComponentComponent", () => {
  let component: HeaderComponentComponent;
  let fixture: ComponentFixture<HeaderComponentComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [HeaderComponentComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(HeaderComponentComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it("should create the component", () => {
    expect(component).toBeTruthy();
  });
});
```

Let's break down this code:

1. We declare a variable to hold an instance of the component we want to test,
   in this case it's `HeaderComponentComponent`
2. We declare a variable to hold an instance of the component fixture associated
   with the component we want to test
   1. `ComponentFixture` is an Angular core class that gives us access to the
      component and to information about the component within the context of
      Angular. For example, we can ask the fixture to `detectChanges()` changes
      in the component in order to simulate the component lifecycle when it's
      running in the application in an actual browser
   2. `ComponentFixture` is a generic, so we can specify the type of the
      component we expect the `fixture` variable to hold
3. Since our tests are now going to test our component, each test will need the
   component initialized, so we will do this initialization work in the
   `beforeEach()` function:
   1. We call the `TestBed.configurationTestingModule()` function to initialize
      the testing module like our root module would be initialized when our
      application is running inside the browser. The main difference is that
      here we can limit the initialization to the actual dependencies we plan to
      test in this suite.
   2. We then call the `compileComponents()` function to execute their
      configuration
   3. We can then get both the `fixture` for the component and the `component`
      itself. Note that getting the component itself is a convenience, as we
      could always use the `fixture` variable to get to the component through
      its `fixture.componentInstance` property
   4. We then ask Jasmine to `detectChanges()` in the component in the same way
      that Angular would when it's running inside the browser
   5. Since `compileComponents()` is an asynchronous function, we have to
      `await` for its completion, which is why its call is wrapped in the
      `async()` and `await` construct

As you can tell from the steps above, much of the setup of the test for a
specific component is aimed at simulating the browser environment that an
Angular application runs in when it's actually loaded for end users.

Now we can write our first actual component test, which you saw in the new
version of `header-component.component.spec.ts` above:

```typescript
it("should create the component", () => {
  expect(component).toBeTruthy();
});
```

Here, we make a simple assertion that our component should have been created.
This is not a very useful functional test other than it does validate that our
test configuration is correct and leads to the actual creation of the component
we want to test.

Let's add another test that actually tests an aspect of the functionality of our
header:

```typescript
it('should contain the text "Welcome"', () => {
  const componentHTML: HTMLElement = fixture.nativeElement;
  expect(componentHTML.textContent).toContain("Welcome");
});
```

Now we've started to test our actual component - in this case, making sure that
it has the "welcome" text somewhere in it. Note that "somewhere in it" is a
pretty flexible way to test that the text is there, but not be so specific as to
need to be updated every time the layout of the component is changed slightly.

Keeping some flexibility in how a component is tested is important because tests
are only useful if they're not excessively brittle. Brittle tests are tests that
fail even though the functionality they intended to test did not change.

Let's look at a more specific test for the presence of the "Welcome" text in the
header to illustrate this concept:

```typescript
it('should contain the text "Welcome" in an <h2> header', () => {
  const componentHTML: HTMLElement = fixture.nativeElement;
  const h2Header = componentHTML.querySelector("h2");
  expect(h2Header.textContent).toContain("Welcome");
});
```

This test passes as well, but will break if we update our view to use an `h3`
instead of an `h2` to display our welcome message. Try it for yourself.

Because the `querySelector()` returns the first HTML element it finds that
matches it, this `h2`-based test also fails if we add another header to our
component.

In some cases, it's possible that you care about the actual HTML element that
your component text is displayed in, in which case your test should definitely
test that. In other words, your test should be as specific as the functionality
that it's testing, but not any more specific than that.

Another limitation of our current test is that it doesn't test that the name of
the user who is being welcomed by the message is the name of the `activeUser` in
the component's controller. Let's add a test that does that:

```typescript
it("should contain the name of the active user from the controller", () => {
  const activeUser = component.activeUser;
  const componentHTML: HTMLElement = fixture.nativeElement;
  expect(componentHTML.textContent).toContain(activeUser.firstName);
});
```

Now you can see that we have 4 tests in our test suite. That may seem excessive,
and you might be tempted to combine some of these assertions in a single test,
but that is not best practice. Each test should stand on its own and test very
specific functionality. Doing so will ensure that when a test fails, it gives
you something very specific to check in your application to validate whether the
test failed because something is broken in your application, or because the test
itself is broken.

## The Karma UI

Let's have a closer look at the UI for our test running utility, Karma:

![Karma Annotated UI](https://curriculum-content.s3.amazonaws.com/java-mod-8/ng-messaging-karma-annotated-ui.png)

Let's look at each element of the UI:

1. When `ng test` is run, Karma is started and a browser window is started to
   connect to the runner. The URL here shows that the runner is listening on
   port 9876
2. This banner indicates that all test suites have completed and the results
   shows are final
3. You can restart the Karma runner in "debug" mode, which allows you to run
   through your tests in Chrome's debugger console and take advantage of all the
   inspection capabilities that debugging gives you
4. This indicates where Karma is running - remember that you can use Karma to
   run your tests in different environments
5. This is a visual representation of your tests and their status. There will be
   one green dot or red x for each test and its corresponding status
6. The tests can be run in a randomized order, which you can control here through
   the options panel (see below)
   1. Randomizing tests can be a useful way to ensure that your tests are indeed
      independent of one another and do not rely on actions taken by a test or a
      series of tests that run before it
7. The `it()` function of each test suite will show here so you can have a high
   level description of each test alongside its results. If a test fails, this
   is where you will see a detailed stack trace that should help you identify the
   underlying issue.
8. This gives you a few options to control your test run without having to
   change your underlying configuration:
   1. Should the entire test suite be stopped when a specific test fails?
   2. Should a given test fail entirely when a single expectation within that
      test fails?
   3. Should the order in which the tests are run be randomized - see above for
      an explanation of why this is a valuable thing to do
   4. Do you want to show tests that have been disabled? Although it can be
      helpful to disable tests temporarily so you can focus on fixing the
      associated issues, we do not recommend hiding disabled tests as that will
      be one less reminder to work on restoring those "temporarily" disabled
      tests
