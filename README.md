# JWT Authentication Demo - Spring Boot

A Spring Boot application demonstrating JWT-based authentication with public and protected API endpoints.

## Features

- ✅ **User Registration** - Register new users with username and password
- ✅ **H2 Database** - File-based persistence for user data
- ✅ **Public API** - Accessible without authentication
- ✅ **Protected API** - Requires valid JWT token
- ✅ **JWT Authentication** - Secure token-based authentication
- ✅ **BCrypt Password Encoding** - Secure password hashing
- ✅ **Stateless Sessions** - No server-side session storage
- ✅ **Data Persistence** - User data survives application restarts

## Tech Stack

- **Java 17**
- **Spring Boot 3.2.1**
- **Spring Security**
- **Spring Data JPA** - Database operations
- **H2 Database** - Embedded database with file persistence
- **JJWT 0.12.3** - JWT library
- **Maven**

## API Endpoints

### Public Endpoints

| Method | Endpoint             | Description             | Authentication  |
| ------ | -------------------- | ----------------------- | --------------- |
| GET    | `/api/public/hello`  | Public greeting         | None            |
| POST   | `/api/auth/register` | Register new user       | None            |
| POST   | `/api/auth/login`    | Login and get JWT token | None            |
| GET    | `/h2-console`        | H2 database console     | None (dev only) |

### Protected Endpoints

| Method | Endpoint             | Description        | Authentication     |
| ------ | -------------------- | ------------------ | ------------------ |
| GET    | `/api/private/hello` | Protected greeting | JWT Token Required |

## Getting Started

### Prerequisites

- Java 17 or higher
- Maven 3.6+

### Build and Run

1. **Clone or navigate to the project directory**

2. **Build the project**

   ```bash
   mvn clean package
   ```

3. **Run the application**

   ```bash
   java -jar target/jwt-demo-1.0.0.jar
   ```

   Or using Maven:

   ```bash
   mvn spring-boot:run
   ```

The application will start on `http://localhost:8080`

## Usage Examples

### 1. Register a New User

```bash
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"myuser","password":"mypassword"}'
```

**PowerShell:**

```powershell
Invoke-WebRequest -Uri http://localhost:8080/api/auth/register `
  -Method POST `
  -Headers @{"Content-Type"="application/json"} `
  -Body '{"username":"myuser","password":"mypassword"}'
```

**Response:**

```json
{
  "message": "User registered successfully",
  "username": "myuser"
}
```

### 2. Access Public API (No Authentication)

```bash
curl http://localhost:8080/api/public/hello
```

**Response:**

```
Hello User
```

### 2. Try to Access Protected API (Without Token)

```bash
curl http://localhost:8080/api/private/hello
```

**Response:**

```
403 Forbidden
```

### 4. Login to Get JWT Token

```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"myuser","password":"mypassword"}'
```

**Response:**

```json
{
  "token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJteXVzZXIiLCJpYXQiOjE3MDY2MA...",
  "username": "myuser"
}
```

### 5. Access Protected API (With Token)

```bash
curl http://localhost:8080/api/private/hello \
  -H "Authorization: Bearer YOUR_JWT_TOKEN_HERE"
```

**Response:**

```
Hello myuser, you are authorized
```

## Database

The application uses **H2 database** with file-based persistence:

- **Database File:** `./data/jwtdemo.mv.db` (created in project root)
- **H2 Console:** http://localhost:8080/h2-console
- **JDBC URL:** `jdbc:h2:file:./data/jwtdemo`
- **Username:** `sa`
- **Password:** `password`

> **Note:** User data persists across application restarts. The H2 console should be disabled in production.

## Configuration

Edit `src/main/resources/application.properties` to customize:

```properties
# Server Port
server.port=8080

# JWT Secret Key (change in production!)
jwt.secret=404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970

# JWT Expiration (24 hours in milliseconds)
jwt.expiration=86400000
```

## Project Structure

```
src/main/java/com/example/jwtdemo/
├── config/
│   ├── SecurityConfig.java           # Spring Security configuration
│   └── PasswordEncoderConfig.java    # Password encoder bean
├── controller/
│   ├── AuthController.java           # Registration & login endpoints
│   ├── PublicController.java         # Public API
│   └── PrivateController.java        # Protected API
├── entity/
│   └── User.java                     # User JPA entity
├── filter/
│   └── JwtAuthenticationFilter.java  # JWT validation filter
├── model/
│   ├── AuthRequest.java              # Login request DTO
│   ├── AuthResponse.java             # Login response DTO
│   ├── RegisterRequest.java          # Registration request DTO
│   └── RegisterResponse.java         # Registration response DTO
├── repository/
│   └── UserRepository.java           # User database repository
├── service/
│   ├── CustomUserDetailsService.java # User authentication service
│   └── UserService.java              # User management service
├── util/
│   └── JwtUtil.java                  # JWT token utilities
└── JwtDemoApplication.java           # Main application class
```

## Security Features

- **JWT Token Validation** - Automatic validation on each request
- **BCrypt Password Hashing** - Secure password storage
- **Stateless Authentication** - No server-side sessions
- **CSRF Protection** - Disabled for stateless JWT auth
- **Token Expiration** - Tokens expire after 24 hours

## How It Works

1. **User registers** with username and password (stored in H2 database)
2. **User logs in** with credentials
3. **Server validates** credentials against database
4. **Server generates** JWT token with user information
5. **Client receives** token and stores it
6. **Client sends** token in `Authorization` header for protected requests
7. **Server validates** token and grants access if valid

## Testing with PowerShell

```powershell
# Register a new user
$regBody = @{username='myuser'; password='mypassword'} | ConvertTo-Json
Invoke-RestMethod -Uri 'http://localhost:8080/api/auth/register' `
  -Method Post -Body $regBody -ContentType 'application/json'

# Login and get token
$loginBody = @{username='myuser'; password='mypassword'} | ConvertTo-Json
$response = Invoke-RestMethod -Uri 'http://localhost:8080/api/auth/login' `
  -Method Post -Body $loginBody -ContentType 'application/json'
$token = $response.token

# Access protected endpoint
$headers = @{Authorization = "Bearer $token"}
Invoke-RestMethod -Uri 'http://localhost:8080/api/private/hello' -Headers $headers
```

## Production Considerations

⚠️ **Before deploying to production:**

1. **Change JWT Secret** - Use a strong, randomly generated secret key
2. **Externalize Configuration** - Move secrets to environment variables or vault
3. **Disable H2 Console** - Set `spring.h2.console.enabled=false`
4. **Use Production Database** - Migrate to PostgreSQL/MySQL
5. **Add Token Refresh** - Implement refresh token mechanism
6. **Enable HTTPS** - Use SSL/TLS for all communications
7. **Add Rate Limiting** - Protect against brute force attacks
8. **Implement Logging** - Add comprehensive security logging
9. **Add Email Verification** - Verify user emails on registration

## License

This is a demonstration project for educational purposes.

## Author

Created as a demonstration of JWT authentication in Spring Boot.
