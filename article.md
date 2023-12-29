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
url: https://github.com/FusionAuth/fusionauth-quickstart-php-web
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

You will run PHP 8, Composer, and Apache in Docker. Using Docker makes this tutorial suitable for Windows, Linux, and macOS.

If you are experienced in PHP, you can install PHP 8, Composer, and Apache on your host machine instead and convert commands to work on your operating system.

## General Architecture

This sample application doesn't have login functionality without FusionAuth, but a more typical integration will replace an existing login system with FusionAuth.

In that case, the system might look like this before FusionAuth is introduced.

<LoginBefore alt={"Request flow during login before FusionAuth"}/>

The login flow will look like this after FusionAuth is introduced.

<LoginAfter alt={"Request flow during login after FusionAuth"}/>

In general, you would introduce FusionAuth to normalize and consolidate user data to make it consistent and up to date, and offload your login security and functionality to FusionAuth.

## Getting Started

Start with getting FusionAuth up and running and creating a new {frontmatter.technology} application.

### Clone The Code

First, grab the code from the repository and change to that folder.

```shell
git clone https://github.com/FusionAuth/fusionauth-quickstart-php-web.git
cd fusionauth-quickstart-php-web
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

This will start four containers, one each for FusionAuth, Postgres, OpenSearch, and PHP.

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

### The Basic {frontmatter.technology} Application

While this guide builds a new {frontmatter.technology} project, you can use the same method to integrate your existing project with FusionAuth.

<Aside type="note">
  Note that the `phpimage.Dockerfile` in the root directory of the project configures Apache to serve files from the `public` directory over the web, to hide `vendor` and configuration files. If you're not using Docker, you will need to do this yourself.

  ```
  RUN sed -i 's|/var/www/html|/var/www/html/public|g' /etc/apache2/sites-available/000-default.conf
  RUN sed -i 's|/var/www/html|/var/www/html/public|g' /etc/apache2/apache2.conf
  ```
</Aside>

In the `docker-compose` configuration, the `--volume ./your-application:/var/www/html` parameter shares your current folder with the container so both your physical machine and Docker can read and write the same files.

<Aside type="note">
  If you simply want to run the application and not create your own, there is a completed version in the `complete-application` directory. You can use the following commands to get it up and running.

  ```shell
  cd complete-application
  docker compose up
  ```

  View the application at http://localhost:9012.
</Aside>

Before you start coding, you need to install the PHP module for your app to communicate with FusionAuth. Run the following command.

```bash
docker run --rm -v ./your-application:/app composer require vlucas/phpdotenv jerryhopper/oauth2-fusionauth
```

## Authentication

Authentication in {frontmatter.technology} is managed by [Jerry Hopper's FusionAuth Provider](https://github.com/jerryhopper/oauth2-fusionauth) for [The League's OAuth library](https://github.com/thephpleague/oauth2-client).


Make the file `your-application/.env` and insert the following lines.

<RemoteCode url={frontmatter.codeRoot + "/complete-application/your-application/.env"}/>

This tells {frontmatter.technology} where to find and connect to FusionAuth.

<Aside type="note">
  If you're not using Docker, you can change the server URL to match the browser URL, as all network communication will
  be on your physical machine.
</Aside>

Authentication is handled by two files: `login.php` and `logout.php`.

The login file is based almost exactly on Jerry Hopper's code, but is split into neater functions and does two extra things: Starts a session and saves the user details to the session after login.

Create a `public` directory within `your-application`.

```shell
mkdir your-application/public
```

In the directory, make the file `your-application/public/login.php` and insert the following code.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/your-application/public/login.php"}
  lang="php"/>

This setup code:
- Prevents errors from being displayed in the browser, for security.
- Loads all vendor modules into PHP so you can use them.
- Reads your variables from the `.env` file.
- Starts a new session using HTTP (not HTTPS since you're on localhost) and ensures your user Id cookie will not be accessible by JavaScript.

Next comes the FusionAuth handlers. Starting from the `getFusionAuthProvider()` function, the code:
- Creates a Jerry Hopper provider with your environment variables.
- Sends the user to the account page if they already have a login cookie.
- Sends the user to the FusionAuth URL if they are not logged in.
- Checks the user's FusionAuth token if they are not logged in but are returning from FusionAuth.
- Gets the user details from the FusionAuth token and saves them to the server-side session. This also generates a new session or cookie Id for the user for security. The user is then redirected to their account page.

PHP automatically links the user's session to their browser by returning a cookie for the site, which is then included in every subsequent request.

The login page handles both the initial user request to start the login process and the server callback request from FusionAuth to complete the authentication.

Logging out is much simpler. Make the file `your-application/public/logout.php` and insert the following code.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/your-application/public/logout.php"}
  lang="php"/>

This code terminates the session on the server for this user and redirects them back to the home page.

## Customization

Now that authentication is done, the last task is to create example pages that a user can browse.

### CSS And HTML

Create a `static` directory in the `your-application/public` directory.

```shell
mkdir your-application/public/static
```

Copy images from the example app.

```shell
cp complete-application/your-application/public/static/money.jpg your-application/public/static/money.jpg
cp complete-application/your-application/public/static/changebank.svg your-application/public/static/changebank.svg
```

Create a stylesheet file `your-application/public/static/changebank.css` and add the following code to it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/your-application/public/static/changebank.css"}
  lang="css"/>

Next, you'll create three more pages in the `your-application/public` directory. First create the home page, `index.php`, and paste the following code into it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/your-application/public/index.php"}
  lang="php"/>

The index page contains nothing to note except a link to the login page `<a href="login.php">`.

Next, create an `account.php` file and paste the following code into it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/your-application/public/account.php"}
  lang="php"/>

The account page displays the user's email from FusionAuth with `<p class="header-email"><?= $_SESSION['email'] ?></p>`.

The account page is only visible to logged in users. If a session Id is not found, the user is redirected to login.

Finally, create a `change.php` file and paste the following code into it.

<RemoteCode
  url={frontmatter.codeRoot + "/complete-application/your-application/public/change.php"}
  lang="php"/>

In addition to verifying login, the code at the top of the change page checks if the CSRF token is valid on POST requests. The CSRF token is hidden in the form in the HTML, `<input type="hidden" name="csrftoken" value="<?= $_SESSION["csrftoken"] ?>" />`. Although any attacker can make a POST request on behalf of a logged in user, a server will reject a GET request if it's not made from the same origin (URL). Including this CSRF token in your page requires an attacker to make a GET request to obtain it before making the POST request, which isn't possible.

The `calculateChange()` method takes the amount given in the form and returns a `$state` array with the amount in nickels and pennies.

The HTML at the bottom of the file displays a blank form when the page first loads (GET) or the result of the calculation when returning (POST).

## Run The Application

The application is already being served by Apache in Docker.

Browse to the app at http://localhost:9012. Log in using `richard@example.com` and `password`. The change page allows you to enter a number.

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

### {frontmatter.technology} Authentication

- [The League OAuth overview](https://oauth2-client.thephpleague.com/usage/)
- [Jerry Hopper's FusionAuth provider](https://github.com/jerryhopper/oauth2-fusionauth)

## Troubleshooting

- I get "This site can’t be reached localhost refused to connect" when I click the login button.

Ensure FusionAuth is running in the Docker container. You should be able to log in as the admin user `admin@example.com` with the password `password` at [http://localhost:9011/admin](http://localhost:9011/admin).

- {frontmatter.technology} says there is an invalid state exception.

Browse to the home page, log out, and try to log in again. If that still doesn't work, delete and restart all the containers.

- It still doesn't work.

You can always pull down a complete running application and compare what's different.

```shell
git clone https://github.com/FusionAuth/fusionauth-quickstart-php-web.git
cd fusionauth-quickstart-php-web/complete-application
docker compose up
```

Browse to the app at http://localhost:9012.