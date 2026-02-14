# URL Shortener

A production-ready URL shortening service built with Spring Boot, featuring user authentication, analytics tracking, and comprehensive click event monitoring.

## Features

- **URL Shortening**: Convert long URLs into short, shareable links
- **User Authentication**: Secure JWT-based authentication system
- **Click Analytics**: Track and analyze click events with date-range filtering
- **User Management**: Role-based access control with user registration
- **Click Tracking**: Automatic recording of every URL access with timestamps
- **Analytics Dashboard**: Retrieve detailed statistics per URL or aggregate user data

## Tech Stack

- **Framework**: Spring Boot 3.x
- **Security**: Spring Security with JWT
- **Database**: JPA/Hibernate (configurable with any SQL database)
- **Authentication**: JWT tokens with role-based authorization
- **Build Tool**: Maven/Gradle
- **Java Version**: 17+

## Project Structure

```
com.url.shortener
├── controller/          # REST API endpoints
│   ├── AuthController          # Authentication & registration
│   ├── UrlMappingController    # URL operations & analytics
│   └── RedirectController      # Short URL redirection
├── dto/                # Data Transfer Objects
├── entity/             # JPA entities
├── repository/         # Data access layer
├── security/           # Security configuration & JWT
│   ├── jwt/
│   └── WebSecurityConfig
└── service/            # Business logic
```

## API Endpoints

### Authentication

#### Register User
```http
POST /api/auth/public/register
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "securePassword123"
}
```

**Response**:
```json
"User registered successfully"
```

#### Login
```http
POST /api/auth/public/login
Content-Type: application/json

{
  "username": "john_doe",
  "password": "securePassword123"
}
```

**Response**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### URL Operations (Requires Authentication)

All endpoints below require the JWT token in the Authorization header:
```
Authorization: Bearer <JWT_TOKEN>
```

#### Create Short URL
```http
POST /api/urls/shorten
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "originalUrl": "https://example.com/very/long/url"
}
```

**Response**:
```json
{
  "id": 1,
  "originalUrl": "https://example.com/very/long/url",
  "shortUrl": "aB3dEfGh",
  "clickCount": 0,
  "createdDate": "2024-12-15T10:30:00",
  "username": "john_doe"
}
```

#### Get User's URLs
```http
GET /api/urls/myurls
Authorization: Bearer <JWT_TOKEN>
```

**Response**:
```json
[
  {
    "id": 1,
    "originalUrl": "https://example.com/very/long/url",
    "shortUrl": "aB3dEfGh",
    "clickCount": 156,
    "createdDate": "2024-12-15T10:30:00",
    "username": "john_doe"
  }
]
```

#### Get URL Analytics
```http
GET /api/urls/analytics/{shortUrl}?startDate=2024-12-01T00:00:00&endDate=2024-12-31T23:59:59
Authorization: Bearer <JWT_TOKEN>
```

**Parameters**:
- `shortUrl`: The 8-character short URL code
- `startDate`: Start date in ISO format (yyyy-MM-ddTHH:mm:ss)
- `endDate`: End date in ISO format (yyyy-MM-ddTHH:mm:ss)

**Response**:
```json
[
  {
    "clickDate": "2024-12-15",
    "count": 25
  },
  {
    "clickDate": "2024-12-16",
    "count": 42
  }
]
```

#### Get Total Clicks by Date Range
```http
GET /api/urls/totalClicks?startDate=2024-12-01&endDate=2024-12-31
Authorization: Bearer <JWT_TOKEN>
```

**Parameters**:
- `startDate`: Start date in format (yyyy-MM-dd)
- `endDate`: End date in format (yyyy-MM-dd)

**Response**:
```json
{
  "2024-12-15": 125,
  "2024-12-16": 203,
  "2024-12-17": 89
}
```

### Public Access

#### Redirect to Original URL
```http
GET /{shortUrl}
```

**Example**: `GET /aB3dEfGh`

**Behavior**: 
- Returns HTTP 302 redirect to the original URL
- Automatically increments click count
- Records click event with timestamp
- Returns 404 if short URL not found

### Generating JWT Secret

Generate a secure Base64-encoded secret key:

```bash
# Using OpenSSL
openssl rand -base64 64

# Using Node.js
node -e "console.log(require('crypto').randomBytes(64).toString('base64'))"
```

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(255) UNIQUE NOT NULL,
    email VARCHAR(255) NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) DEFAULT 'ROLE_USER'
);
```

### URL Mapping Table
```sql
CREATE TABLE url_mapping (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    original_url TEXT NOT NULL,
    short_url VARCHAR(8) UNIQUE NOT NULL,
    click_count INT DEFAULT 0,
    created_date TIMESTAMP NOT NULL,
    user_id BIGINT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### Click Event Table
```sql
CREATE TABLE click_event (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    click_date TIMESTAMP NOT NULL,
    url_mapping_id BIGINT,
    FOREIGN KEY (url_mapping_id) REFERENCES url_mapping(id)
);
```

## Security Features

- **Password Encryption**: BCrypt hashing algorithm for secure password storage
- **JWT Authentication**: Stateless token-based authentication with configurable expiration
- **Role-Based Access Control**: Method-level security with `@PreAuthorize` annotations
- **CORS Configuration**: Configurable cross-origin resource sharing for frontend integration
- **Protected Endpoints**: All URL management endpoints require valid authentication
- **Public Access**: Limited to registration, login, and URL redirection

## Getting Started

### Prerequisites

- Java 17 or higher
- Maven 3.6+ or Gradle 7+
- MySQL/PostgreSQL/H2 database
- Git

### Installation Steps

1. **Clone the repository**
```bash
git clone <repository-url>
cd url-shortener
```

2. **Configure the database**
   - Create a new database: `CREATE DATABASE urlshortener;`
   - Update `application.properties` with your database credentials

3. **Generate JWT secret**
   - Generate a secure Base64-encoded key
   - Update `jwt.secret` in `application.properties`

4. **Build the project**

Using Maven:
```bash
./mvnw clean install
```

Using Gradle:
```bash
./gradlew build
```

5. **Run the application**

Using Maven:
```bash
./mvnw spring-boot:run
```

Using Gradle:
```bash
./gradlew bootRun
```

6. **Access the application**
   - API Base URL: `http://localhost:8080`
   - Test with tools like Postman, cURL, or HTTPie

## Usage Example

### Complete Workflow

1. **Register a new user**
```bash
curl -X POST http://localhost:8080/api/auth/public/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "email": "alice@example.com",
    "password": "securePass123"
  }'
```

2. **Login to get JWT token**
```bash
curl -X POST http://localhost:8080/api/auth/public/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "alice",
    "password": "securePass123"
  }'
```

Response: `{"token":"eyJhbGc..."}`

3. **Create a short URL**
```bash
curl -X POST http://localhost:8080/api/urls/shorten \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer eyJhbGc..." \
  -d '{
    "originalUrl": "https://www.example.com/very/long/url/path"
  }'
```

4. **Access the short URL**
```bash
curl -L http://localhost:8080/aB3dEfGh
```

5. **View analytics**
```bash
curl -X GET "http://localhost:8080/api/urls/analytics/aB3dEfGh?startDate=2024-12-01T00:00:00&endDate=2024-12-31T23:59:59" \
  -H "Authorization: Bearer eyJhbGc..."
```

## Analytics Capabilities

### Per-URL Analytics
Track detailed click statistics for individual shortened URLs:
- Daily click counts within specified date ranges
- Historical click patterns
- Time-series data for visualization

### Aggregate User Analytics
Monitor overall performance across all user's URLs:
- Total clicks per day across all URLs
- User activity trends
- Comparative analysis between different time periods

### Click Event Tracking
Every URL access is recorded with:
- Precise timestamp (LocalDateTime)
- Associated URL mapping reference
- Automatic click count increment

## Error Handling

The API returns appropriate HTTP status codes:

- `200 OK`: Successful request
- `302 Found`: Successful redirect
- `401 Unauthorized`: Missing or invalid JWT token
- `404 Not Found`: Short URL not found
- `500 Internal Server Error`: Server-side error

## Development

### Running Tests
```bash
./mvnw test
```

### Building for Production
```bash
./mvnw package
```

### Docker Deployment

Build and run:
```bash
docker build -t url-shortner-be .
docker tag url-shortner-be akash5307/url-shortner-be:latest
docker push akash5307/url-shortner-be:latest
```

## Future Enhancements

- Custom short URL aliases
- QR code generation
- Link expiration dates
- Geographic click tracking
- Browser and device analytics
- Rate limiting and abuse prevention
- Admin dashboard
- Bulk URL import
- API rate limiting

## License

This project is available for educational and commercial use.

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## Support

For issues, questions, or contributions, please open an issue in the repository.

