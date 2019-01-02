# Observing the DOM in the browser extension

Create a more robust way of handling DOM interactions.

## Background

I frequently see issues that are filed that just say "the browser
extension doesn't work" and then when I go to any links provided, it
works for me ([example](https://github.com/sourcegraph/sourcegraph/issues/756)). I also somewhat frequently see errors come in through
Sentry that are related to elements not being found when we would
logically expect then to be there (examples:
[1](https://sentry.io/sourcegraph/browser-extension/?query=is%3Aunresolved+Unable+to+find+command+palette+mount),
[2](https://sentry.io/sourcegraph/browser-extension/?query=is%3Aunresolved+unable+to+find+repo+pjax+container),
etc). This makes me think we should remove
any assumptions we make around the DOM. This project will fix bugs we
know exist and hopefully take care of a lot of seemingly flakey and
non-reproducible errors at the same time.

## Plan

Create a way to observe the DOM rather than simply query the DOM. We
already have this implemented for listening for new code views being
added to the DOM. I'll abstract that implementation and use that for all
locations where we query the DOM. The API would look as follows:

```typescript
interface DOMObserver {
  /**
   * Listen to the DOM for a specific selector. It will immediately emit all
   * elements already in the DOM and will asynchronously emit new ones added.
   *
   * @param selector the selector we are wanting to match in the DOM.
   * @param timeout time in ms to wait before throwing an error if no matches are found.
   */
  observeSelector(selector: string, timeout = 500): Observable<HTMLElement>
  /**
   * Unsubscribe to either a specific selector or all selectors being listened to.
   *
   * @param selector if provided, stop listening to that specific selector. If
   * not provided, stop listening to all selectors.
   */
  unsubscribe(selector?: string): void
}
```

### Test plan

This will be hard to test in an automated way. This is simply because its a
hard problem, not because the code is making it harder. After this project,
we will have higher test coverage. It will rely on the browser extension's
e2e tests. The library that this will live in will use Karma/mocha to run
unit tests for this in actual browsers. The reason for using Karma/mocha is
[JSDom doesn't support `MutaionObserver`'s
yet.](https://github.com/jsdom/jsdom/pull/2398)

### Release plan

Nothing special here. Just release the browser extension once merged.
We'll monitor the types of errors and issues mentioned in the background
afterwards.

### Success metrics

We no longer see errors in the browser extension related to not finding elements
when we expect to and less errors reported overall.

### Company goals

This supports our company goals by providing a more stable and usable product
for our users.

## Rationale

After watching issues come and observing the errors being sent to
Sentry, I believe this change is necessary. Whether its the root cause
of *all* these errors is unknown, but it will fix a lot of errors we are
seeing.

This will be it's own library rather than a part of the main Sourcegraph
repository because it is a very isolated functionality we want to provide and
should not and will not have any Sourcegraph specific logic. Once in the stable
state, it won't be updated frequently (the API is small and won't have changes).
In addition to the logical split, we want to have full freedom over the test
environment for this library and don't want to muddy up the main repo's testing
infrastructure. The main repo now uses jest and JSDom for testing which doesn't
support `MutationObserver` yet. This is why we'll likely use Karma to run tests
in real browsers.

## Checklist

- [ ] 1/2/19 Create dom observer library 
  - This uses a `MutationObserver` to listen to the DOM for changes
        and will emit when a selector matches an element in a DOM
        change.
- [ ] 1/2/19 Write unit tests for this library.
- [ ] 1/2/19 Replace usages of `document.querySelector` and the like in the
    browser extension. This will require a change to the
    [MountGetter](https://sourcegraph.com/github.com/sourcegraph/sourcegraph/-/blob/client/browser/src/libs/code_intelligence/code_intelligence.tsx#L109:8)
    type. We will remove this type and only accept an object with the selector
    and [insert
    position](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement#Parameters)
    where we will create a new node and add it to the DOM.

## Done date

1/2/19

## Retrospective

> To be updated.

### Actual checklist

> To be updated.

### Actual done date

> To be updated.