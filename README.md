JavaScript XHR
---

## Objectives

1. Explain how JavaScript fetches data from remote resources
2. Explain how XHR helps us write dynamic programs
3. Practice initializing an XHR request
4. Practice handling an XHR response

## Introduction

Talk about difficulty of loading all of the information that we need at once (perhaps there's an analogy to a library to be made?)

Then introduce them to fetching data only as it's needed and how we might do this with JavaScript

## Implementation

We can use the [GitHub API](https://developer.github.com/v3/repos/#list-user-repositories) unauthenticated to try out a few requests

It's probably worth spending a bit of time on attaching the event listeners and handling response data (good chance to introduce `JSON.parse()`) -- these things can be confusing for beginners and experienced engineers alike

## Testing?

This could be a tested code-along. For testing, stub out the calls to the API and use mock data.

## Resources

- [MDN: XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- [GitHub API](https://developer.github.com/v3/repos/#list-user-repositories)
