---
title: WTH is GitHub's GraphQL API even?
author: Zhian N. Kamvar
date: '2022-12-20'
slug: github-graphql
categories: ["github"]
tags: ["git", "GraphQL", "GitHub", "R", "API", "Learning"]
subtitle: 'Using R to understand what even is this nonsense'
excerpt: ''
draft: no
series: ~
layout: single
---

After my previous blog post, I thought I would try to blog a bit more about
small things that I learn as I go along. This is one of those posts.

I ran into a situation recently where [I needed to use the GitHub API to retrieve
the last N commits from a pull request](https://github.com/carpentries/actions/pull/68).

It turned out that the way to do it was to create a GraphQL query that looked
like this and [implement it in JavaScript](https://github.com/carpentries/actions/pull/68/files#diff-5cfda5b5171fcd68534db78d7c1de9561292f14c54889a66484b4e67e52ebcfeR62-R101):

```
query lastCommits($owner: String!, $repo: String!, $pull_number: Int = 1, $n: Int = 1) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $pull_number) {
      commits(last: $n) {
        edges {
          node {
            commit {
              oid
            }
          }
        }
      }
    }
  }
}
```

Of course, when I first saw this, I didn't know what the hell to think. Here's 
how I got here. 


## REST assured, I will find the answer

Of course, I thought the correct answer was to use to use [the REST API to get
the commits from the pull request](https://docs.github.com/en/rest/pulls/pulls?apiVersion=2022-11-28#list-commits-on-a-pull-request):

```r
res <- gh::gh("GET /repos/{owner}/{repo}/pulls/{pull}/commits", 
  owner = "carpentries",
  repo = "actions",
  pull = 65,
  per_page = 5)
purrr::map_chr(res, \(x) x$sha)
```

```
## [1] "ca84cd24382762bdb6f683fbcf85153cad67ff36" "28a30d4aed001a65cf4def1b73e7e5ee067a9072"
## [3] "b642782d5dfe6959a659fd57c6f8e074f6609b9f" "502bda45f74b0a4d4ca6e005d452e17b2cb3ac26"
## [5] "b28092b12c8f57ec8971f6bc690f1b491ad322ed"
```

Of course, if you look at <https://github.com/carpentries/actions/pull/68>, you
will see that those are the _first five_ commits. I needed the _last five_.
:weary:

## GraphQL Content

After some frantic googling, I found that a lot of answers for how to get the
last N commits of a resource on GitHub all seemed to reference using the GraphQL
API. The last time I had thought about the GraphQL API was when I read
<https://frie.codes/posts/analyzing-commits-graphql-r/> two years ago. While
Frie is a great writer, I have been super intimidated by the GraphQL language,
mainly becuse the [GitHub GraphQL API documentation](https://docs.github.com/en/graphql/overview/about-the-graphql-api)
is so full of jargon that I really don't understand. 

What I needed was a good, working example with something I could manipulate and
then I realised that I could look up [The JavaScript implementation](https://github.com/octokit/octokit.js#graphql-api-queries) and lo and behold, a useful query for me to
mess with appeared:

```javascript
const { lastIssues } = await octokit.graphql(
  `
    query lastIssues($owner: String!, $repo: String!, $num: Int = 3) {
      repository(owner: $owner, name: $repo) {
        issues(last: $num) {
          edges {
            node {
              title
            }
          }
        }
      }
    }
  `,
  {
    owner: "octokit",
    repo: "graphql.js",
  }
);
```

Now here, I know the basic framework of a JS request like this is:

```javascript
const { thingToExtract } = await library.function(
  `
  long string argument with $variables
  `,
  {
    variables: "that are replaced in the first argument"
  }
);
```

This gives me the clue as to what exactly the GraphQL query is doing and puts
[the tutorial step into context](https://docs.github.com/en/graphql/guides/forming-calls-with-graphql#working-with-variables).


```
query lastIssues($owner: String!, $repo: String!, $num: Int = 3) {
  repository(owner: $owner, name: $repo) {
    issues(last: $num) {
      edges {
        node {
          title
        }
      }
    }
  }
}
```

Here, I can see that the query is setting up a typed function (I know this!) and
that it's passing the variable names to the function itself. This is a similar
kind of mechanism as the curly braces in the REST API, except it's actually typed.

I started playing around with the [GraphQL explorer](https://docs.github.com/en/graphql/overview/explorer), which _is_ actually helpful for looking up what different
elements of the query are and auto-filling some of the edges information and I
was able to get this query working (oid is the ID for the commit object).

```
query {
  repository(owner: "carpentries", name: "actions") {
    pullRequest(number: 65) {
      commits(last: 5) {
        edges {
          node {
            commit {
              oid
            }
          }
        }
      }
    }
  }
}
```

The only thing was, it gave me a response of and I was _pretty sure_ how to
parse it, but I wanted to confirm, so I fired up {gh} to help me see what
the reponse looked like in a language I understood:

````
{
  "data": {
    "repository": {
      "pullRequest": {
        "commits": {
          "edges": [
            {
              "node": {
                "commit": {
                  "oid": "25f7362f9a200f333429e70085eeb0eb1f8d95e8"
                }
              }
            },
            {
              ... // you get the idea
            }
          ]
        }
      }
    }
  }
}

````

````r
query <- r"{
query {
  repository(owner: "carpentries", name: "actions") {
    pullRequest(number: 65) {
      commits(last: 5) {
        edges {
          node {
            commit {
              oid
            }
          }
        }
      }
    }
  }
}
}"
res <- gh::gh_gql(query)
purrr::map_chr(res$data$repository$pullRequest$commits$edges, \(x) x$node$commit$oid)
````

````
## [1] "25f7362f9a200f333429e70085eeb0eb1f8d95e8" "f1e6ae3309f1a784a6b399974ff476df793316cc"
## [3] "d2b975918b2ac51c97aae2894c34c97885303c1f" "578ec3e4571978e3766fa40ffd63e5bef274b178"
## [5] "a063e9dc57693e90900884fd50454cfc33a11f5e"
## 
````


Now that I had the knowledge of how to form the query and how to parse the
output, I could futz around with jamming it into JavaScript until I got
something working.

## Conclusion

GraphQL has been around for a few years and it's not unique to GitHub, but the
documentation could use some work. I was able to get through it not because of
the documentation, but because of the examples provided by third-party
developers. It often takes a while for a concept to click and it's really nice
to be able to have a working examples with small variations to use as scaffolds,
especially if you have a way to interact with the output in a language you 
understand. 
