## JavaScript XHR

The XMLHttpRequest object, or XHR, is a JavaScript API that allows us to
transfer data between a client and a server. This makes it possible to request
and receive web page updates without refreshing, leading to an improved
experience for users. In this lesson, we will be exploring the use of XHR by
using it to access GitHub.

## Objectives

1.  Explain how XHR helps us write dynamic programs
2.  Practice initializing an XHR request
3.  Practice handling an XHR response
4.  Understand how JavaScript fetches data from remote resources

## XMLHttpRequest

XMLHttpRequest was named at a time when XML was all the rage, but it can be used
with any type of data, including JSON, which is the current de facto standard.

> **Definition:** JSON stands for JavaScript Object Notation. Browsers and
> servers are only able pass strings of text to each other. By using JavaScript
> Object Notation, we can structure this text in a way that a browser or server
> can read as a regular JavaScript object.

XHR helps us write dynamic programs by allowing us to fetch data from a server
based on user events, and update parts of pages without requiring a full-page
refresh. This provides users with a smooth, engaging experience that doesn't
require them to stop what they're doing to get new information.

## Using XHR to Get Data from a Server

We're going to be making a simple Github repository browser using the
[Github API][api].

> **Definition:** API stands for [Application Programming Interface][api_wiki].
> In this course, we are working specifically with web APIs, which are sets of
> tools or protocols that allow us to communicate with a resource hosted on a
> remote server. In this lesson, we're communicating with GitHub using the
> protocols they've defined in their documentation.

Code along in the provided `index.html` and `index.js` files. A basic HTML
structure is already in place. Getting data from a server via XHR happens in two
stages. First, we make a _request_, and then we listen for, and handle, the
_response_.

#### Creating the XHR Request

The first thing we want to do is get a list of our public repositories. A
little research on the [Github List Repositories API][user_repos] tells us
we can request a user's public repositories via a `GET` request to `https://api.github.com/users/:username/repos`, so let's try it out.

**Top-tip:** API documentation will often use a colon to precede a dynamic
value in a RESTful URL, like `:username`. That's your hint to supply your own
value.

First, let's add a link to our HTML so we can trigger the request.

```html
<div>
  <h3>Repositories</h3>
  <a href="#" onclick="getRepositories()">Get Repositories</a>
</div>
```

Then let's create our `getRepositories` function and initiate our XHR request.

```js
function getRepositories() {
  const req = new XMLHttpRequest();
  req.open('GET', 'https://api.github.com/users/octocat/repos');
  req.send();
}
```

Here, we're creating a new instance of an `XMLHttpRequest`. We call `open` with
the HTTP verb we want, in this case `GET`, and the URI for the request.

Now that our request is set up and ready to go, we call `send` to make it happen.

Let's open `index.html` in our browser, open the inspector's `Network` tab, and
click the link.

Something happened! If we examine the request in the inspector, we should see a
response that looks something like this:

```js
[
  {
    id: 18221276,
    name: 'git-consortium',
    full_name: 'octocat/git-consortium',
    owner: {
      login: 'octocat',
      id: 583231,
      avatar_url: 'https://avatars.githubusercontent.com/u/583231?v=3',
      gravatar_id: '',
      url: 'https://api.github.com/users/octocat',
      html_url: 'https://github.com/octocat',
      followers_url: 'https://api.github.com/users/octocat/followers',
      following_url:
        'https://api.github.com/users/octocat/following{/other_user}',
      gists_url: 'https://api.github.com/users/octocat/gists{/gist_id}',
      starred_url:
        'https://api.github.com/users/octocat/starred{/owner}{/repo}',
      subscriptions_url: 'https://api.github.com/users/octocat/subscriptions',
      organizations_url: 'https://api.github.com/users/octocat/orgs',
      repos_url: 'https://api.github.com/users/octocat/repos',
      events_url: 'https://api.github.com/users/octocat/events{/privacy}',
      received_events_url:
        'https://api.github.com/users/octocat/received_events',
      type: 'User',
      site_admin: false
    }
  }
  //... more!
];
```

It worked! We successfully fetched data from a remote resource with XHR without
reloading our page!

Now that we have the request part down, we need to figure out how to capture
this response so we can do something with it.

#### Handling the XHR Response

The second part of XHR is handling the response once we've made the request. We
do this by defining an event listener on the request to listen for the `load`
event, which will tell us that the request is complete. We'll give this listener
a _callback function_, which is simply a function that will get called when the
event fires.

```js
function showRepositories() {
  //this is set to the XMLHttpRequest object that fired the event
  console.log(this.responseText);
}

function getRepositories() {
  const req = new XMLHttpRequest();
  req.addEventListener('load', showRepositories);
  req.open('GET', 'https://api.github.com/users/octocat/repos');
  req.send();
}
```

When we add the event listener to our `req` object, we set it up so that `this`
will be our `req` object inside our callback function. So, inside
`showRepositories`, we can access `this.responseText` to see the full body of
the response from our XHR request.

Now that we know how to access the response, let's do something with it.

#### Parsing the XHR Response

Since the Github API deals strictly in JSON, we know that our response will be
well-formed JSON, so it should be easy for us to work with.

Let's parse this response and list out the repositories on the page. We'll
start by giving ourselves a place in the DOM to put the data.

```html
<div>
  <h3>Repositories</h3>
  <a href="#" onclick="getRepositories()">Get Repositories</a>
  <div id="repositories"></div>
</div>
```

Then let's start by simply listing the repository names.

```js
function showRepositories() {
  console.log(this.responseText);
  let repoList = '<ul>';
  for (var i = 0; i < this.responseText.length; i++) {
    repoList += '<li>' + this.responseText[i]['name'] + '</li>';
  }
  repoList += '</ul>';
  document.getElementById('repositories').innerHTML = repoList;
}
```

Now if we reload and click our link...

Okay not quite what we expected. While it might be fun to have a list of a
million `undefined` values on a page, we got repositories to print out. What
happened?

The key lies in the `responseText` property. We can look at it and understand
that it's JSON, but to our JavaScript interpreter, it's just a string of text.
And while we _know_ that all JSON is just a string of text, we have to _tell_
JavaScript that it's working with JSON.

The way we tell the interpreter that we're working with JSON is to parse it with [`JSON.parse`][parse].

```js
function showRepositories() {
  var repos = JSON.parse(this.responseText);
  console.log(repos);
  const repoList = `<ul>${repos
    .map(r => '<li>' + r.name + '</li>')
    .join('')}</ul>`;
  document.getElementById('repositories').innerHTML = repoList;
}
```

Now we're properly parsing the text into an array of objects that we can work
with. If we reload and try it again, we should get our list of repository names.

Okay, let's take this a step further and set ourselves up to make another XHR
request based on our data.

We want to be able to list the commits for any given repository. Again, we
don't want to just re-query the server for each repository as we're processing
that data, we just want to respond to the user asking for a specific repo's commits.

Let's go back into the Github API docs for [commits][commits] and check it out.

We can see that we can make another `GET` request to
`/repos/:owner/:repo/commits` and list the commits. We have the repo name to
fill in for the `:repo` parameter based on our repo list. Let's say "repo" a
few more times for fun then see what we can do.

We know we'll need an element to click for each repository on our page that
will request that repository's commits. So we'll need to add a "Get Commits"
link to our output in `showRepositories`, make a new XHR request when that link
is clicked, and then show the commits in the second column.

We'll start by adding the link to our repository output.

```js
function showRepositories() {
  var repos = JSON.parse(this.responseText);
  console.log(repos);
  const repoList = `<ul>${repos
    .map(
      r =>
        '<li>' +
        r.name +
        ' - <a href="#" data-repo="' +
        r.name +
        '" onclick="getCommits(this)">Get Commits</a></li>'
    )
    .join('')}</ul>`;
  document.getElementById('repositories').innerHTML = repoList;
}
```

Let's look more closely at this line: `r.name + ' - <a href="#" data-repo="' + r.name + '" onclick="getCommits(this)">Get Commits</a></li>'`.

The first interesting thing is that we're using a [data attribute][data]
to hold the repo name. Data attributes make it super easy to pass data around
between DOM elements and JS, so rather than jump through hoops trying to set
and query `id` attributes, we'll do this.

The second thing is our `onclick` is explicitly passing `this` to the
`getCommits` function. We need to do this to make sure that the current
element, that is, the link being clicked, is available to our `getCommits`
function so that we can get at that data attribute later.

Now that that's out of the way, let's set up our `getCommits`. It's going to
look very similar to `getRepositories`, because it's mostly about just making
another XHR request to Github.

```js
function getCommits(el) {
  const name = el.dataset.repo;
  const req = new XMLHttpRequest();
  req.addEventListener('load', showCommits);
  req.open('GET', 'https://api.github.com/repos/octocat/' + name + '/commits');
  req.send();
}
```

Here we grab that `data-repo` value through the `dataset` property, then set up
an XHR request, with an event listener and callback function, just like we did
in `getRepositories`.

Let's create a place in our HTML to put the commits.

```html
<div>
  <h3>Commits</h3>
  <div id="commits"></div>
</div>
```

Finally, let's handle that request with our callback function. We can look at
the docs for this API call to see the JSON structure and know what values we
want to pull out, then display them on the page.

```js
function showCommits() {
  const commits = JSON.parse(this.responseText);
  const commitsList = `<ul>${commits
    .map(
      commit =>
        '<li><strong>' +
        commit.author.login +
        '</strong> - ' +
        commit.commit.message +
        '</li>'
    )
    .join('')}</ul>`;
  document.getElementById('commits').innerHTML = commitsList;
}
```

Reload it and check it out. Now we can load repositories, then see commits for
any repository dynamically without refreshing the page or reloading the
repository list!

## Summary

We learned what the `XMLHttpRequest` object does, how to use it to request data
from a remote resource, and how to handle the response. We also learned how to
parse the `responseText` into JSON and display it on the page.

## Resources

- [MDN: XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)
- [GitHub API](https://developer.github.com/v3/repos/#list-user-repositories)
- [MDN: JSON.Parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)
- [MDN: Using data attributes](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_data_attributes)

[api]: https://developer.github.com/v3/repos/
[user_repos]: https://developer.github.com/v3/repos/#list-user-repositories
[parse]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse
[data]: https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_data_attributes
[commits]: https://developer.github.com/v3/repos/commits/
[api_wiki]: https://en.wikipedia.org/wiki/Application_programming_interface