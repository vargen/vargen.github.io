---
layout: default
title: Authentication
parent: CI Fuzz
nav_order: 2
permalink: ci-fuzz-authentication
---
# **Authentication**
{: .no_toc }

This page describes the different ways to configure authentication to `CI Fuzz Web Application`. `CI Fuzz` supports OAuth, OIDC, and password based logins.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## OAuth

To use SSO with GitHub, Bitbucket, or GitLab you need to create an OAuth app. 

###  GitHub
{: .no_toc }

1. Open the [developer settings](https://github.com/settings/developers) 
2. Click **Register a new application**.
3. For the **Authorization callback URL**, use `https://<fuzzing_server_domain>/auth/github/callback`.
4. After registering the application, GitHub will generate a `Client ID` and `Client Secret`. Open `/etc/cifuzz/config.env` and copy these values to`CIFUZZ_GITHUB_CLIENT_ID` and `CIFUZZ_GITHUB_CLIENT_SECRET` respectively.

### Bitbucket.org
{: .no_toc }

1. Go to [bitbucket cloud](https://bitbucket.org) and select the appropriate workspace
2. Click **Settings**
3. Under **Apps and features**, select **OAuth consumers** and click **Add consumer**. 
4. As the **callback url**, use `https://<fuzzing_server_domain>:<port>/auth/github/callback`. **Note:** The port is mandatory, even if it's the default port 443. 
5. Give it the email and read permissions in the **Account** section and then save it.
6. Expand the section for the consumer you just created to show the **Key** and **Secret**. 
7. Open `/etc/cifuzz/config.env` and copy these values to`CIFUZZ_BITBUCKET_CLIENT_ID` and `CIFUZZ_BITBUCKET_CLIENT_SECRET` respectively.


### GitLab
{: .no_toc }

1. Go to **Preferences** and click **Applications**.
2. Choose a name and set the **Redirect URI** to `https://<fuzzing_server_domain>/auth/github/callback`.
3. Enable the read_user scope.
4. Click **Save application** and then select the application from **Your applications**.
5. Open `/etc/cifuzz/config.env` and copy **Application ID** and **Secret** to`CIFUZZ_GITLAB_CLIENT_ID` and `CIFUZZ_GITLAB_CLIENT_SECRET` respectively.


## OIDC

This section describes to setup your own OIDC provider with `CI Fuzz`.

### Create an OIDC-capable application
  
In the OIDC provider, create an OIDC-capable application with:

* Redirect URL: `<baseURL>/auth/<provider>/callback`, where:
    * `<baseURL>` is the URL that the CI Fuzz web app is available at, for example https://cifuzz.example.com.
    * `<provider>` is a name of your choice, that will be used for this OIDC provider in the CI Fuzz web app.
* If configurable at the provider, permissions that allow reading user profile information, like the name and email address, via OIDC.

  
Take note of the application's client ID and client secret, you need those below.  

### Configure CI Fuzz Server

If the OIDC provider implements the [OpenID Connect Discovery spec](https://openid.net/specs/openid-connect-discovery-1_0.html) (i.e. a JSON document exists at `.well-known/openid-configuration`), the setup is simpler. In that case, create the file `/etc/cifuzz/oidc.yaml` as:

```
auth:  
  oidc:  
    <provider>:  
      id: <client ID>  
      secret: <client secret>  
      issuer_url: <issuer URL>
```
  
... where:  

* `<provider>` is the name for the OIDC provider you chose above.
* `<client ID>` and <client secret> are the client ID and secret of the application you created above.
* `<issuer URL>` is the base URL of the OIDC provider, for example https://gitlab.com.

If the OIDC provider does not support OpenID Connect Discovery, add these settings to the configuration file instead:  

```
auth:    
  oidc:  
    <provider>:  
      id: <client ID>  
      secret: <client secret>  
      auth_endpoint: <auth endpoint URL>  
      token_endpoint: <token endpoint URL>  
      userinfo_endpoint: <UserInfo endpoint URL> 
      jwks_url: <JWKS URL> 
```

  
... where:  

*   `<provider>` is the name for the OIDC provider you chose above.
*   `<client ID>` and `<client secret>` are the client ID and secret of the application you created above.
*   `<auth endpoint URL>` is the URL of the authorization endpoint of the OIDC provider, for example https://gitlab.com/oauth/authorize.
*   `<token endpoint URL>` is the URL of the token endpoint of the OIDC provider, for example https://gitlab.com/oauth/token.
*   `<UserInfo endpoint URL>` is the URL of the UserInfo endpoint of the  OIDC provider, for example https://gitlab.com/oauth/userinfo.
    * This setting is optional. If no UserInfo endpoint is specified, only the Claims of the ID Token will be used.
*   `<JWKS URL>` is the URL of the OIDC provider's JSON Web Key Set document, for example https://gitlab.com/oauth/discovery/keys.


## Password

If you are just trying out `CI Fuzz`, it may be more convenient to use a password as the authentication method. In `/etc/cifuzz/cifuzz.env` there are two options you need to set:

* `CIFUZZ_ENABLE_PASSWORD_LOGIN=1`
* `DEMO_ORG_ADMIN_TOKEN=<your_password_here>`

**Note:** This is not the recommended authentication approach for `CI Fuzz` with multiple users. Password authentication is only intended for initial testing and setup until you are ready to implement OAuth.
