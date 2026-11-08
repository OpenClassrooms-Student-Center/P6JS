# SportSee

This repo contains all the source code to run the micro API for the sports analytics dashboard SportSee.

## 1. General information

To start this project, you are free to use Docker or not. In this documentation, we will see several methods to launch the project easily.

## 2. Project (**without Docker**)

### 2.1 Prerequisites

- [NodeJS (**version 12.18**)](https://nodejs.org/en/) or higher (tested up to Node 20.0)
- [Yarn](https://yarnpkg.com/)

If you are working with several versions of NodeJS, we recommend you install [nvm](https://github.com/nvm-sh/nvm). This tool will allow you to easily manage your NodeJS versions.

### 2.2 Launching the project

- Fork the repository
- Clone it on your computer.
- The `yarn` command will allow you to install the dependencies.
- The `yarn dev` command will allow you to run the micro API.

## 3. Project (**with Docker**)

### 2.1 Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop)

### 2.2 Starting the project

- The `docker image build --no-cache -t micro-api .` command will allow you to build your image.
- The `docker container run --name micro-api -p 8000:8000 -dt micro-api yarn` command will allow you to create your Docker container and run your image on port 8000.
- The `docker container stop micro-api` command will allow you to stop your micro-api.
- The `docker container rm micro-api` command will allow you to delete your micro-api container.

### 2.3 Vscode and container remotes

Finally, if you have VsCode, you can easily launch your project in a docker environment.

You will need the [Remote Development extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack). Once you have this extension installed, just click on the `Reopen in Container` button.

Once in the container, run the `yarn dev` command.

## 4. Authentication

The API uses JWT (JSON Web Token) authentication. To access the endpoints:

1. First obtain a JWT token by logging in:

```bash
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "sophiemartin", "password": "password123"}'
```

2. Use the received token in subsequent requests in the Authorization header:

```bash
curl -H "Authorization: Bearer your-jwt-token" http://localhost:8000/api/user-info
```

### 4.1 Available Users

Currently, the API has three demo users:

- username: `sophiemartin`, password: `password123`
- username: `emmaleroy`, password: `password789`
- username: `marcdubois`, password: `password456`

## 5. Endpoints

### 5.1 Authentication Endpoint

- `POST /api/login` - Authenticates a user and returns a JWT token
  - Required body: `{ "username": "string", "password": "string" }`
  - Returns: `{ "token": "jwt-token", "userId": "string" }`

### 5.2 Data Endpoints

All these endpoints require authentication via a Bearer token in the header:
`Authorization: Bearer <your_token>`

#### Get User Information
```http
GET /api/user-info
```
Returns the user profile and overall statistics:
```json
{
  "profile": {
    "firstName": "string",
    "lastName": "string",
    "createdAt": "string",
    "age": number,
    "gender": "string",
    "weight": number,
    "height": number,
    "profilePicture": "string"
  },
  "weeklyGoal": number,
  "statistics": {
    "totalDistance": number,
    "totalSessions": number,
    "totalDuration": number,
    "totalCalories": number,
    "restDays": number
  }
}
```

- `weeklyGoal`: number of running sessions the user aims to do per week. Some accounts still use the legacy schema and return this value as `goal` instead of `weeklyGoal`.
- `statistics` are computed on past sessions only (future dates excluded), so they match the sessions returned by `/api/user-activity`.
- `totalDistance` is rounded to one decimal.
- `totalCalories` is the sum of `caloriesBurned` over all past sessions.
- `restDays` is the number of days since sign-up (`createdAt`, inclusive) without any running session.

#### Get Activity Sessions
```http
GET /api/user-activity?startWeek=<date>&endWeek=<date>
```
Returns an array of running sessions between two dates (sorted ascending, future dates excluded).

**Parameters:**
- `startWeek`: Start date (ISO format)
- `endWeek`: End date (ISO format)


#### Access Profile Images
```http
GET /images/<filename>
```
Serves the static profile images (e.g. the `profilePicture` URLs returned by `/api/user-info`).


#### Notes
- All dates should be in ISO format (YYYY-MM-DD)
- All distances are in kilometers
- All durations are in minutes
- All calories are in kcal


### 5.3 Examples of queries

```bash
# Login
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "sophiemartin", "password": "password123"}'

# Get user data
curl -H "Authorization: Bearer your-jwt-token" http://localhost:8000/api/user-info

# Get user activity
curl -H "Authorization: Bearer your-jwt-token" \
  "http://localhost:8000/api/user-activity?startWeek=2025-01-01&endWeek=2025-01-31"
```

### 5.4 Error Responses

The API will return appropriate HTTP status codes:

- 400: Bad Request (missing username/password)
- 401: Unauthorized (missing or invalid token)
- 403: Forbidden (invalid token)
- 404: Resource not found
- 500: Server error
