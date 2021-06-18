---
title: "TIL: PostgREST & Postgraphile"
template: "post"
date: "2021-05-03"
draft: false
slug: "til-postgrest-postgraphile"
category: "SQL"
tags:
  - "SQL"
  - "PostgreSQL"
description: "Tools for generating APIs from your database schema."
---

Developers often seek ways to make their codebase as homogenous as possible, in the belief that it's easier to maintain an application when everything looks and behaves similarly. This can be true to an extent - it's why we settle on certain architectural patterns, code styling norms, etc. 

This is partly why backend developers lean heavily on [ORMs](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) - it's easier to think in your target language alone, rather than worry about maintaining SQL with C#, Python, etc. 

But the more I've learned about SQL, the more I wonder: isn't it odd that we build further abstractions on top of an already declarative language like SQL? 

I recently discovered a couple tools that pose a different solution: what if we generate our API from our SQL schema?

[PostgREST](https://postgrest.org/en/stable/) and [Postgraphile](https://www.graphile.org/postgraphile/) do just this - point them at a live Postgres database, and you instantly have a working API (RESTful or GraphQL, depending on which tool you choose).

Need to specify some unique business logic that's not handled by default CRUD operations? Since these tools are based on Postgres, you can use language extensions such as [PLV8](https://plv8.github.io/) to use JavaScript in your stored procedures & functions, allowing you to treat your Postgres database like your personal serverless platform.

Amazingly, you can even use [JWT authentication](https://postgrest.org/en/stable/auth.html#) and can use Postgres' [row level security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html) for authorization. 
