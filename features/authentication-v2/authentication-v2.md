# Authentication v2
**Authors:** [@shea786](https://github.com/shea786)
**Type:** Feature

## Overview
This phase of authentication we will be adding the ability for Contributors to sign up on nawhas.com via email and password.

### Definitions

| Term        | Definition                                            |
| ----------- | ----------------------------------------------------- |
| Contributor | A standard user that has limited edit rights.         |
| User        | A Contributor                                         |


## Why are we making this change?
To allow the Public to edit nawhas.com we had added the foundation of authentication as well as login functionaltiy through [Authentication v1](https://github.com/nawhas/rfcs/blob/master/features/authentication-v1/authentication-v1.md)
We want to allow the users to sign up on nawhas.com and this change will allow for users to register with username/password.

## Requirements
- Public user should be able to sign up for a Contributor account.
- While signing up, user can provide the following information
  - Name *
  - Email *
  - Password *
  - Confirm Password *
  - Nickname
  - If `Password` and `Confirm Password` are not the same then user should not be able to register until issue is resolved as well as be prompted about the passwords not matching.
  - If `Email` is not in the correct format then user cannot submit the form until that is resolved
- Once there are no validation errors, user should be able to submit the registration form.
- After submitting the form
  - If `Email` is already taken then error will be thrown back to the suer
  - If `Nickname` is given and it is already taken then error will be thrown back to the user
- Once all backend validation is complete and user is created we should trigger a notification to the end user informing them that their account has been created.
- By default, the user role will be contributor

## Detailed Engineering Design

### API

#### New Endpoints
We'll add the following endpoints

```
POST /v1/auth/register
{
  "name": string,
  "email": string,
  "password": string,
  "nickname": ?string,
}
RESPONSES
  - 200: Account created and logged in
         { user: { ... } }
ERRORS
  - 422: Invalid request
         { message: "...", "errors": { email: "...", ... } }
```

We will be adding the following functions to `App\Http\Controllers\Api\AuthController`
- register

`register` function will handle the creation of the user.

### Frontend

#### Vuex Store

##### Actions
- `signUp` - Post to the register API endpoint and commit `LOGIN`


#### SignUpForm Component
A `signUpForm` component will handle the processes of:
- gathering user input for name, email, password, confirm password
- dispatching the `auth/signUp` action
- handling validation errors

### Deployment Strategy
This feature needs to be coupled with a few other changes before we can deploy to production
These changes will not be a part of this feature request.
- Make sure that contributor cannot edit/update parts of the system until audit/moderator changes etc are implemented
- Add a feature flag based system so we can turn features on or off. If this is implemented then we do not need to worry about the first point

## Mockups
<!-- Attach relevant mockups here. Links to Figma are also appropriate. -->
