# Insert Title Here
**Authors:** [@shea786](https://github.com/shea786)  
**Type:** Feature

## Overview
This phase of authentication will allow contributors to register and log in with Google, Facebook, Twitter and Github

### Definitions

| Term         | Definition                                            |
| ------------ | ----------------------------------------------------- |
| Moderator    | A privileged user with full edit and approval rights. |
| Contributor  | A standard user that has limited edit rights.         |
| User         | Either a Moderator or a Contributor                   |
| Social Login | Google, Facebook, Twotter or Github                   |


## Why are we making this change?
<!-- Explain the motivation for this change -->
<!-- 
Example: 
To achieve the greater effort of allowing public edit access on Nawhas.com, enabling moderators to log in is a prerequisite. This change will also lay the foundation for the overall authentication system and enable Contributor registration and logins in the future.
-->

## Requirements
- Allow contributors to sign up to nawhas.com through social login's
- If a user has already registered through a social login and tries to do the same again, notify them that they already have an account and ask them to log in
- Allow a user to log into nawhas.com through social login's.
- If a usre tries to log in through social login but they have not registered then ask the user to register

## Detailed Engineering Design

### API
We will be using the following package which will allow us to log in or sign up with socail accounts
- [laravel/socialite](https://laravel.com/docs/7.x/socialite)

### Frontend
<!-- Describe any changes necessary to the Vue app to make this feature possible. -->

### Deployment Strategy
- Need to think of a way to implement this on lower end environments. Some of the issues are outlined below
  - On PR Preview builds, the return URL will be different to what is in the application details for the relevant provider
  - We will need to add the environment variables for each build. We might have to use a different application for each PR Preview
  - For local environments, we can use the same application as the redirect URL will be the same

## Mockups
<!-- Attach relevant mockups here. Links to Figma are also appropriate. -->