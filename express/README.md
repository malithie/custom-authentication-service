# Pin-Based Authentication Service

This is a Node.js Express-based authentication service that provides a PIN-based authentication mechanism. The service can run locally and is designed to work in both federated and internal authentication modes, with support for second-factor authentication.

**Note:** This is a sample authentication service designed to demonstrate the **service-based custom authenticator feature** of WSO2 Identity Server. It is intended solely for testing and should not be used in production environments.

## Features

- PIN-based authentication mechanism
- Session persistence using an **in-memory map**
- Supports **federated**, **internal**, and **second-factor** authentication
- Secure handling of authentication requests
- Compatible with **Vercel** and **local environments**

## Prerequisites

Ensure you have the following installed on your system:

- **Node.js** (>=14.x)
- **npm** or **yarn**

## Setup Instructions

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/malithie/custom-authentication-service.git
cd custom-authentication-service/express
```

### 2️⃣ Install Dependencies

```bash
npm install
```

### 3️⃣ Configure Environment Variables

Create a `.env` file in the project root with the following:

```env
AUTH_MODE=federated  # Options: federated, internal, second_factor
BASE_WSO2_IAM_PROVIDER_URL=https://localhost:9443
```

- `AUTH_MODE`: Defines whether the service operates as a **federated authenticator**, an authenticator for **internally managed users**, or a **second-factor authentication service**.
- `BASE_WSO2_IAM_PROVIDER_URL`: Specifies the **host origin** of the running WSO2 Identity Server instance.

### 4️⃣ Add Users for Authentication

Before running the service, define the users that need to be authenticated.

Create a `users.json` file inside the `data/` directory with the following structure:

```json
{
  "federated": [
    {
      "id": "5d009575-8962-4402-94ae-0b14ec0a8d31",
      "username": "emily",
      "pin": "1234",
      "data": {
        "id": "1234",
        "claims": [
          { "uri": "http://wso2.org/claims/emailaddress", "value": "emily@aol.com" },
          { "uri": "http://wso2.org/claims/lastname", "value": "Ellon" },
          { "uri": "http://wso2.org/claims/givenname", "value": "Emily" }
        ]
      }
    }
  ],
  "internal": [
    {
      "id": "8a2b49b5-91fa-4f3b-b2d8-d12f7f77dcaa",
      "username": "peter",
      "pin": "5678",
      "data": {
        "id": "5678",
        "claims": [
          { "uri": "http://wso2.org/claims/emailaddress", "value": "peter@aol.com" },
          { "uri": "http://wso2.org/claims/lastname", "value": "Parker" },
          { "uri": "http://wso2.org/claims/givenname", "value": "Peter" }
        ]
      }
    }
  ]
}
```

- **Internal users**: Ensure the `id` and `username` fields match the user definitions in **Identity Server**.
- **Federated users**: External users that **do not** need to be pre-registered in Identity Server.

### 5️⃣ Run the Service Locally

```bash
npm start
```

The service will be available at: **[http://localhost:3000](http://localhost:3000)**

### 6️⃣ Verify the Service

Run a health check to ensure the service is running:

```bash
curl http://localhost:3000/api/health
```

Expected Response:

```json
{ "status": "ok", "message": "Service is running." }
```

## API Endpoints

### 🔹 **Health Check**

- **GET** `/api/health`
- **Response:** `{ "status": "ok", "message": "Service is running." }`

### 🔹 **Authenticate User**

- **POST** `/api/authenticate`
- **Request Body:**

```json
{
  "flowId": "1234",
  "event": {
    "tenant": { "name": "example.com" },
    "user": { "id": "5678" }
  }
}
```

- **Response:**

```json
{
  "actionStatus": "INCOMPLETE",
  "operations": [{ "op": "redirect", "url": "http://localhost:3000/api/pin-entry?flowId=1234" }]
}
```

### 🔹 **PIN Entry Page**

- **GET** `/api/pin-entry?flowId=1234`
- **Returns:** HTML page for entering PIN.

### 🔹 **Validate PIN**

- **POST** `/api/validate-pin`
- **Request Body:**

```json
{
  "flowId": "1234",
  "userId": "5678",
  "pin": "1234",
  "tenant": "example.com"
}
```

- **Response:**

```json
{
  "redirectingTo": "https://localhost:9443/t/example.com/commonauth?flowId=1234"
}
```

## Deployment

To deploy to **Vercel**, follow these steps:

```bash
vercel --prod
```

### Vercel Configuration Steps

1. **Set Environment Variables**:

   - In Vercel, go to **Project Settings → Environment Variables**.
   - Add `AUTH_MODE` with the preferred authenticator type (`federated`, `internal`, or `second_factor`).
   - Add `USER_CONFIG` by converting the `users.json` file into a compact string:
     ```bash
     cat data/users.json | jq -c
     ```
     Copy and paste the output as the value for `USER_CONFIG` in Vercel.

2. **Disable Vercel Authentication**:

   - Go to **Project Settings → Deployment Protection**.
   - Turn **off** Vercel Authentication to make the service publicly accessible.
   - This step is required so that WSO2 Identity Server can access the authentication service.

3. **Deploy to Vercel**:

    ```bash
    vercel --prod
    ```

## Contributing

Contributions are welcome! Feel free to **fork** this repo and submit pull requests. 🚀








