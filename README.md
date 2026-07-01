# JWT Cross API Auth Demo

This project demonstrates how a single Authentication API can generate separate JWT tokens for multiple validator APIs.

## Project Structure

```text
JwtCrossApiAuthDemo
│
├── AuthApi
│   └── Generates JWT tokens
│
├── ValidatorApi1
│   └── Accepts token for audience https://localhost:44394/
│
├── ValidatorApi2
│   └── Accepts token for audience https://localhost:5001/
│
└── ValidatorApi3
    └── Accepts token for audience https://localhost:7001/
```

## Purpose

The purpose of this demo is to understand:

* JWT issuer validation
* JWT audience validation
* Generating different tokens from one Auth API
* Validating each token in its relevant API
* Preventing one API token from being used in another API

## JWT Concept

### Issuer

The issuer identifies who created the token.

```json
"iss": "https://localhost:7191/"
```

In this project, the Auth API is the issuer.

### Audience

The audience identifies which API the token is intended for.

Example:

```json
"aud": "https://localhost:44394/"
```

This means the token is valid only for Validator API 1.

## Token Flow

```text
Auth API
 ├── Creates token for Validator API 1
 ├── Creates token for Validator API 2
 └── Creates token for Validator API 3
```

Each validator API accepts only its own audience.

```text
Validator API 1 → accepts only audience https://localhost:44394/
Validator API 2 → accepts only audience https://localhost:5001/
Validator API 3 → accepts only audience https://localhost:7001/
```

## Auth API Configuration

```json
"Jwt": {
  "Issuer": "https://localhost:7191/",
  "Key": "asdfghjklzxcvbnmqwertyuioasdfghjklzxcvbnmqwertyuiopasdfghjklzxcvbnm",
  "Audiences": {
    "validator1": "https://localhost:44394/",
    "validator2": "https://localhost:5001/",
    "validator3": "https://localhost:7001/"
  }
}
```

## Validator API 1 Configuration

```json
"Jwt": {
  "Issuer": "https://localhost:7191/",
  "Audience": "https://localhost:44394/",
  "Key": "asdfghjklzxcvbnmqwertyuioasdfghjklzxcvbnmqwertyuiopasdfghjklzxcvbnm"
}
```

## Validator API 2 Configuration

```json
"Jwt": {
  "Issuer": "https://localhost:7191/",
  "Audience": "https://localhost:5001/",
  "Key": "asdfghjklzxcvbnmqwertyuioasdfghjklzxcvbnmqwertyuiopasdfghjklzxcvbnm"
}
```

## Validator API 3 Configuration

```json
"Jwt": {
  "Issuer": "https://localhost:7191/",
  "Audience": "https://localhost:7001/",
  "Key": "asdfghjklzxcvbnmqwertyuioasdfghjklzxcvbnmqwertyuiopasdfghjklzxcvbnm"
}
```

## Generate Token

### Validator API 1 Token

```http
POST https://localhost:7191/api/auth/login/validator1
```

### Validator API 2 Token

```http
POST https://localhost:7191/api/auth/login/validator2
```

### Validator API 3 Token

```http
POST https://localhost:7191/api/auth/login/validator3
```

## Example Response

```json
{
  "service": "validator1",
  "audience": "https://localhost:44394/",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

## Test Secure Endpoint

Use Swagger or Postman.

Add the token in the Authorization header:

```http
Authorization: Bearer YOUR_TOKEN_HERE
```

Then call:

```http
GET /api/test/secure
```

## Expected Result

If the correct token is used:

```json
{
  "message": "Token Valid",
  "userId": "1",
  "username": "admin",
  "role": "Admin"
}
```

If the wrong token is used:

```http
401 Unauthorized
```

## Example Validation

| Token            | Validator API 1 | Validator API 2 | Validator API 3 |
| ---------------- | --------------- | --------------- | --------------- |
| validator1 token | Valid           | Invalid         | Invalid         |
| validator2 token | Invalid         | Valid           | Invalid         |
| validator3 token | Invalid         | Invalid         | Valid           |

## Important Notes

* All APIs must use the same signing key.
* All validator APIs must trust the same issuer.
* Each validator API should have its own audience.
* Do not manually add multiple audience claims if you want separate tokens.
* Use one token per API when strict separation is needed.

## Summary

This demo shows how to issue multiple JWT tokens from a single Auth API and validate each token in its relevant API using issuer, audience, signing key, and lifetime validation.
