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
  - If `Nickname` is provided then we must check to see if the nickname is unique. If nickname is not unique then we should ask the user to pick another nickname.
- Once there are no validation errors, user should be able to submit the registration form.
- Once the user has been created we should trigger a notification to the end user informing them that their account has been created.
- By default, the user role will be contributor

## Detailed Engineering Design

### API

#### New Endpoints
We'll add the following two endpoints

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

GET /v1/auth/nickname
RESPONSES
  - 200: Nickname availability
         { available: true/false }
```

We will be adding the following functions to `App\Http\Controllers\Api\AuthController`
- register
- nicknameCheck

`register` function will handle the creation of the user.
`nicknameCheck` function will be used to determine if the nickname given is already taken or not

### Frontend

#### Vuex Store

##### Actions
- `signUp` - Post to the register API endpoint and commit `LOGIN`


#### SignUpForm Component
A `signUpForm` component will handle the processes of:
- gathering user input for name, email, password, confirm password
- dispatching `checkNickname` request inline to see if the nickname is taken or not
- dispatching the `auth/signUp` action
- handling validation errors

### Deployment Strategy
This code will not be affecting any existing users and therefore should be safe to roll out to 100% of users

## Mockups
<!-- Attach relevant mockups here. Links to Figma are also appropriate. -->
