# AuthService

AuthService is a Java 21 / Spring Boot service that provides user signup and
login, JWT access tokens, refresh tokens, and a protected endpoint for checking
the current user's identity. It uses MySQL for user and refresh-token storage
and Kafka for publishing user information events.

## Requirements

- JDK 21
- MySQL
- Kafka

Gradle is provided by the included wrapper.

## Configure

The application reads its database and Kafka connection settings from
environment variables. Configure the following before starting the service:

| Variable | Purpose |
| --- | --- |
| `MYSQL_HOST` | MySQL server hostname |
| `MYSQL_PORT` | MySQL server port |
| `MYSQL_DB` | Database name |
| `MYSQL_USER` | Database username |
| `MYSQL_PASSWORD` | Database password |
| `KAFKA_HOST` | Kafka broker hostname |
| `KAFKA_PORT` | Kafka broker port |

Defaults for some connection settings are defined in
`app/src/main/resources/application.properties`. Use a dedicated database
account and strong credentials outside local development.

## Build and run

From the repository root, build the application:

```powershell
.\gradlew.bat :app:bootJar
```

Start the service with the configured environment:

```powershell
.\gradlew.bat :app:bootRun
```

The service listens on port `9898` by default. The container image can be built
after building the JAR:

```powershell
docker build -t authservice .
```

`services.yml` contains a Docker Compose setup for local infrastructure and
related services. Review its service images and connection settings for your
environment before using it.

## HTTP endpoints

| Method | Path | Description |
| --- | --- | --- |
| `POST` | `/auth/v1/signup` | Register a user and return access and refresh tokens |
| `POST` | `/auth/v1/login` | Authenticate a user and return tokens |
| `POST` | `/auth/v1/refreshToken` | Exchange a valid refresh token for a new access token |
| `GET` | `/auth/v1/ping` | Return the authenticated user's ID |
| `GET` | `/health` | Basic health check |

Login request:

```json
{
  "username": "your-username",
  "password": "your-password"
}
```

Refresh-token request:

```json
{
  "token": "your-refresh-token"
}
```

Send the access token as a bearer token when calling the protected ping
endpoint:

```text
Authorization: Bearer <access-token>
```

Signup also accepts user profile fields such as `first_name`, `last_name`,
`email`, and `phone_number`.

## Deployment

The GitHub Actions workflow in `.github/workflows/deploy.yml` builds and pushes
the container image to Amazon ECR and deploys the CloudFormation stack when
changes are pushed to `main`. Configure the AWS, network, AMI, and database
GitHub Actions secrets referenced by that workflow before enabling deployment.

## Security note

Before deploying to production, replace the JWT signing key currently defined
in the application code with a securely managed secret, and rotate any
credentials that may have been used in other environments. Never commit
production secrets to the repository.
