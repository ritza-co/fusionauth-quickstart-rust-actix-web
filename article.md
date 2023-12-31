---
title: Rust with Actix
description: Quickstart integration of a Rust Actix web application with FusionAuth
navcategory: getting-started
prerequisites: Docker version 20
section: web
cta: EmailListCTA
language: Rust
framework: Actix
icon: /img/icons/rust.svg
faIcon: fa-rust
url: https://github.com/FusionAuth/fusionauth-quickstart-rust-actix-web
codeRoot: https://raw.githubusercontent.com/ritza-co/fusionauth-quickstart-rust-actix-web/main
---

import Aside from '/src/components/Aside.astro';
import LoginAfter from '../../diagrams/quickstarts/login-after.astro';
import LoginBefore from '../../diagrams/quickstarts/login-before.astro';
import {RemoteCode} from '@fusionauth/astro-components';


In this quickstart, you will build an application with {frontmatter.language} and {frontmatter.framework} and integrate it with FusionAuth. The application is for [ChangeBank](https://www.youtube.com/watch?v=pkH-kD73QUM), a global leader in converting dollars to coins. The app will have areas reserved for logged in users and public-facing sections.

While this article uses {frontmatter.framework}, the {frontmatter.language} OAuth2 library can also be used in your preferred framework, such as Rocket or Axum.

Find the Docker Compose file and source code for the complete application at <a href={frontmatter.url}>{frontmatter.url}</a>.

## Prerequisites

For this quickstart, you'll need:

* [Docker](https://www.docker.com) version 20 or later, which is the quickest way to start FusionAuth. (There are [other ways](/docs/v1/tech/installation-guide/).)
* [{frontmatter.language}](https://rustup.rs/#) and Cargo version 1.7 or later.

## General Architecture

This sample application doesn't have login functionality without FusionAuth, but a more typical integration will replace an existing login system with FusionAuth.

In that case, the system might look like this before FusionAuth is introduced.

<LoginBefore alt={"Request flow during login before FusionAuth"}/>

The login flow will look like this after FusionAuth is introduced.

<LoginAfter alt={"Request flow during login after FusionAuth"}/>

In general, you would introduce FusionAuth to normalize and consolidate user data to make it consistent and up to date, and offload your login security and functionality to FusionAuth.

## Getting Started

Start with getting FusionAuth up and running and creating a new {frontmatter.framework} application.

### Clone The Code

First, grab the code from the repository and change to that folder.

```shell
git clone https://github.com/FusionAuth/fusionauth-quickstart-rust-actix-web.git
cd fusionauth-quickstart-rust-actix-web
mkdir your-application
```

All shell commands in this guide can be entered in a terminal in this folder. On Windows, you need to replace forward slashes with backslashes in paths.

All the files you'll create in this guide already exist in the `complete-application` subfolder, if you prefer to copy them to `your-application`.

### Run FusionAuth Via Docker

You'll find a Docker Compose file `docker-compose.yml` and an environment variables configuration file `.env` in the root folder of the repository.

Assuming you have Docker installed, you can start FusionAuth on your machine with the following.

```shell
docker compose up
```

This will start three containers, one each for FusionAuth, Postgres, and OpenSearch.

Here you are using a bootstrapping feature of FusionAuth called [Kickstart](/docs/v1/tech/installation-guide/kickstart). When FusionAuth starts for the first time, it will look at the `kickstart/kickstart.json` file and configure FusionAuth to your specified state.

<Aside type="note">
  If you ever want to reset the FusionAuth system, delete the volumes created by `docker-compose` by executing `docker
  compose down -v`, then rerun `docker compose up -d`.
</Aside>

FusionAuth will be configured with these settings:

* Your client Id is `e9fdb985-9173-4e01-9d73-ac2d60d1dc8e`.
* Your client secret is `super-secret-secret-that-should-be-regenerated-for-production`.
* Your example username is `richard@example.com` and the password is `password`.
* Your admin username is `admin@example.com` and the password is `password`.
* The base URL of FusionAuth is `http://localhost:9011/`.

You can log in to the [FusionAuth admin UI](http://localhost:9011/admin) and look around if you want to, but with Docker and Kickstart, everything will already be configured correctly.

<Aside type="caution">
  The `.env` and `kickstart.json` files contain passwords. In a real application, always add these files to your
  `.gitignore` file and never commit secrets to version control.
</Aside>

### The Basic {frontmatter.framework} Application

While this guide builds a new {frontmatter.framework} project, you can use the same method to integrate your existing project with FusionAuth.

<!-- In the `docker-compose` configuration, the `--volume ./your-application:/app` parameter shares your current folder with the container so both your physical machine and Docker can read and write the same files. -->

<Aside type="note">
  If you simply want to run the application and not create your own, there is a completed version in the `complete-application` directory. You can use the following commands to get it up and running.

  ```shell
  cd complete-application
  cargo install
  cargo run
  ```

  View the application at http://localhost:9012.
</Aside>

From here on, you'll work in the `your-application` directory. Install the dependencies for the web server with the code below.

```bash
cd your-application
cargo init
cargo add actix-web@4 # actix web server
cargo add actix-files@0.6.2 # static file server
cargo add actix-session@0.8.0 --features cookie-session # session manager
cargo add dotenv@0.15.0 # .env file handler
cargo add handlebars@4.5 --features dir_source # html templates
cargo add oauth2@4.4.2 # oauth provider to fusionauth
cargo add reqwest@0.11 --features json # http request helper
cargo add serde@1.0 --features derive # json helper
```

Add the following code to your `src/main.rs` file.

<RemoteCode url={frontmatter.codeRoot + "/complete-application/src/main.rs"}/>

The `main` function configures the web server, adds routes to it, and starts it. The function also:

- Uses `dotenv` to allow you to call any values from the `.env` file later in the application.
-  Uses private same-site cookies to store the user's email when logged in. {frontmatter.framework} does not provide an anti-CSRF token as they [believe same-site cookies render it unnecessary](https://github.com/actix/actix-web/issues/147).
-  Adds routes from the main file to the server, and some authentication routes, which you'll add in the next section.
-  Enables the Handlebars library to provide HTML templates.

The remainder of this main file is three routes: index, account, and change. They each check if the user's email is in the HTTP request cookie (meaning she is logged in), and display an HTML template. The account and change routes send the user's email to the template.

The change route is more complicated. It has two versions: one for GET and one for POST. The POST version checks the request's form to extract a dollar amount, then calls `calculate_change` to convert the dollars to nickels and pennies, and returns them in the state to the Handlebars template.

## Authentication

Authentication in {frontmatter.language} is managed by [OAuth2](https://docs.rs/oauth2/latest/oauth2/).

Make the file `your-application/.env` and insert the following lines.

<RemoteCode url={frontmatter.codeRoot + "/complete-application/.env"}/>

This tells {frontmatter.framework} where to find and connect to FusionAuth.

Authentication is handled by `src/auth.rs`. Make that file now and insert the following code.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/src/auth.rs"}
  lang="rust" />

This code has three routes: `login`, `logout`, and `callback`.

- `logout` clears the user's session and returns them to the home page.
- `login` uses `get_oauth_client` to read your variables from the `.env` file and get a new client that constructs a URL to call FusionAuth. `login` then redirects the user to that URL.
- `callback` does the majority of the work. The user is returned there after logging in with FusionAuth. The function checks that PKCE challenge is correct, retrieves the access token, and makes a final call to FusionAuth to get the user's email, which it stores in the session. Now the application considers the user logged in.

{frontmatter.technology} automatically links the user's session to their browser by returning a cookie for the site, which is then included in every subsequent request.

Now that `callback` has set a cookie, you can see how authentication on the other pages is tested. The application:

- Sends the user to the account page if they already have a login cookie on the home page.
- Sends the user to the home page if they are not logged in on the account page.

## Customization

With authentication done, the last task is to create example pages that a user can browse.

### CSS And HTML

Create a `static` directory in the `your-application` directory.

```shell
mkdir your-application/static
```

Copy images from the example app.

```shell
cp complete-application/static/money.jpg your-application/static/money.jpg
cp complete-application/static/changebank.svg your-application/static/changebank.svg
```

Create a stylesheet file `your-application/static/changebank.css` and add the following code to it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/static/changebank.css"}
  lang="css"/>

Next, create the `your-application/templates` directory. Create three pages inside it. First create the home page, `index.html`, and paste the following code into it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/templates/index.html"}
  lang="html"/>

The index page contains nothing to note except a link to the login page `<a href="/login">`.

Next, create an `account.html` file and paste the following code into it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/templates/account.html"}
  lang="html"/>

The account page displays the user's email from FusionAuth with `          <p class="header-email">{{email}}</p>`.

The account page is only visible to logged in users. If a session email is not found, the user is redirected to login.

Finally, create a `change.html` file and paste the following code into it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/templates/change.html"}
  lang="html"/>

The HTML at the bottom of the file displays a blank form when the page first loads (GET) or the result of the calculation when returning (POST).

## Run The Application

Run your application.

```bash
cargo run
```

Browse to the app at http://localhost:9012. Log in using `richard@example.com` and `password`. The change page allows you to enter a number. If you don't log in, you won't be able to access the change or account pages.

## Next Steps

This quickstart is a great way to get a proof of concept up and running quickly, but to run your application in production, there are some things you're going to want to do.

### FusionAuth Customization

FusionAuth gives you the ability to customize just about everything to do with the user's experience and the integration of your application. This includes:

* [Hosted pages](/docs/v1/tech/themes/) such as login, registration, email verification, and many more.
* [Email templates](/docs/v1/tech/email-templates/email-templates).
* [User data and custom claims in access token JWTs](/articles/tokens/jwt-components-explained).

### Security

* You may want to customize the [token expiration times and policies](/docs/v1/tech/oauth/#configure-application-oauth-settings) in FusionAuth.
* Choose [password rules](/docs/v1/tech/core-concepts/tenants#password) and a [hashing algorithm](/docs/v1/tech/reference/password-hashes) that meet your security needs.

### Tenant And Application Management

* Model your application topology using [Applications](/docs/v1/tech/core-concepts/applications), [Roles](/docs/v1/tech/core-concepts/roles), [Groups](/docs/v1/tech/core-concepts/groups), [Entities](/docs/v1/tech/core-concepts/groups), and more.
* Set up [MFA](/docs/v1/tech/guides/multi-factor-authentication), [Social login](/docs/v1/tech/identity-providers/), or [SAML](/docs/v1/tech/identity-providers/samlv2/) integrations.
* Integrate with external systems using [Webhooks](/docs/v1/tech/events-webhooks/), [SCIM](/docs/v1/tech/core-concepts/scim), and [Lambdas](/docs/v1/tech/lambdas/).

### {frontmatter.language} Authentication

- [Rust OAuth provider](https://docs.rs/oauth2/latest/oauth2/)
- [Actix](https://actix.rs/)

## Troubleshooting

- I get "This site can’t be reached localhost refused to connect" when I click the login button.

Ensure FusionAuth is running in the Docker container. You should be able to log in as the admin user `admin@example.com` with the password `password` at [http://localhost:9011/admin](http://localhost:9011/admin).

- It still doesn't work.

You can always pull down a complete running application and compare what's different.

```shell
git clone https://github.com/FusionAuth/fusionauth-quickstart-rust-actix-web.git
cd fusionauth-quickstart-rust-actix-web
docker compose up
cd complete-application
cargo install
cargo run
```

Browse to the app at http://localhost:9012.
