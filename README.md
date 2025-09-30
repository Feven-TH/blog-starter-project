
# Blog Starter Project

This repository contains a comprehensive, backend for a modern blogging platform, built with Go. It follows a clean architecture pattern and includes features like user authentication, blog and comment management, AI-powered content assistance, and efficient data handling with caching and background workers.

## Architecture

The project is structured using a clean architecture approach to ensure separation of concerns, maintainability, and testability.

-   **`Domain`**: Contains the core business logic, entities (structs), and interfaces that define the contracts for repositories and use cases. This layer is independent of any framework or external technology.
-   **`Usecases`**: Implements the application-specific business rules by orchestrating the flow of data between the `Domain` and `Repository` layers.
-   **`Repository`**: The data access layer. This implementation uses MongoDB to persist data and an in-memory LRU cache to optimize read operations.
-   **`Delivery`**: The presentation layer, responsible for handling HTTP requests and responses. It uses the Gin web framework to define API routes and controllers.
-   **`Infrastructure`**: Holds all the external concerns like database connections, JWT services, OAuth configurations, password hashing, email services, and background workers.

## Key Features

-   **User Management & Authentication**:
    -   Secure user registration with email verification.
    -   Local login using JWT (Access & Refresh Tokens).
    -   Google OAuth2 for social login.
    -   Forgot/Reset password functionality.
    -   User profile creation and updates.
    -   Role-based access control (User vs. Admin).

-   **Blog & Comment System**:
    -   Full CRUD (Create, Read, Update, Delete) for blog posts and comments.
    -   Like/Dislike system for blog posts.
    -   Popularity score calculation based on views, likes, and comments.
    -   Efficient pagination and sorting for blogs (latest, oldest, popular) and comments.
    -   Advanced blog filtering by tags and date ranges.
    -   Full-text search functionality for blog content.

-   **AI-Powered Content Assistance**:
    -   Integrates with Google's Gemini AI to provide content editing and SEO suggestions for blog writers.

-   **Performance & Scalability**:
    -   In-memory LRU caching for frequently accessed blogs and comments to reduce database load.
    -   Asynchronous background worker to refresh blog popularity scores without blocking API requests.
    -   Optimized database queries with indexing.

-   **Configuration**:
    -   Flexible configuration management using Viper, supporting both `config.yaml` and environment variables.

## Getting Started

Follow these instructions to set up and run the project locally.

### Prerequisites

-   Go (version 1.24 or later)
-   MongoDB instance (local or Atlas)
-   A Google Cloud project for OAuth2 credentials.
-   A Google Gemini API Key.
-   An application-specific password for your email account (for sending verification emails).

### Installation & Setup

1.  **Clone the Repository**
    ```sh
    git clone https://github.com/gedyzed/blog-starter-project.git
    cd blog-starter-project
    ```

2.  **Configure Environment Variables**
    Create a `.env` file in the root directory. You can copy the structure from `Infrastructure/config/env.example`, but note that the example file is empty. Your `.env` file should look like this:

    ```env
    # Application
    PORT=8080
    APP_URL="http://localhost:8080"

    # Database
    MONGO_URL="<your_mongodb_connection_string>"

    # Authentication (generate strong random strings)
    AUTH_ACCESS_TOKEN_KEY="<your_super_secret_access_key>"
    AUTH_REFRESH_TOKEN_KEY="<your_super_secret_refresh_key>"

    # Google OAuth2
    OAUTH_CLIENT_ID="<your_google_client_id>"
    OAUTH_CLIENT_SECRET="<your_google_client_secret>"
    OAUTH_REDIRECT_URL="http://localhost:8080/oauth/callback"

    # Email Service (for user verification & password reset)
    EMAIL_SENDER_EMAIL="<your_email@example.com>"
    EMAIL_APP_PASSWORD="<your_email_app_password>"
    EMAIL_SMTP_HOST="smtp.gmail.com"
    EMAIL_SMTP_PORT="587"

    # Generative AI
    GEMINI_API_KEY="<your_google_gemini_api_key>"
    ```

3.  **Install Dependencies**
    ```sh
    go mod tidy
    ```

4.  **Run the Application**
    ```sh
    go run ./Delivery/main.go
    ```
    The server will start on the port specified in your `.env` file (default: `8080`).

## API Endpoints

### User & Authentication Routes

| Method | Endpoint                    | Description                                       | Access     |
| :----- | :-------------------------- | :------------------------------------------------ | :--------- |
| `POST` | `/users/register`           | Register a new user with email verification.      | Public     |
| `POST` | `/users/login`              | Log in a user with username and password.         | Public     |
| `POST` | `/users/forgot-password`    | Send a password reset link to the user's email.   | Public     |
| `POST` | `/users/reset-password`     | Reset password using a token from email.          | Public     |
| `POST` | `/users/token/refresh_token`| Get a new access token using a refresh token.     | Public     |
| `DELETE`| `/users/logout/:username`   | Invalidate the user's session tokens.             | Public     |
| `POST` | `/users/update-profile`     | Update the logged-in user's profile information.  | Protected  |
| `POST` | `/tokens/send-vcode`        | Send a verification code to an email.             | Public     |

### Google OAuth Routes

| Method | Endpoint              | Description                                        | Access |
| :----- | :-------------------- | :------------------------------------------------- | :----- |
| `GET`  | `/oauth/auth/login`   | Redirects to Google's authentication page.         | Public |
| `GET`  | `/oauth/callback`     | Callback URL for Google to redirect to after auth. | Public |
| `POST` | `/oauth/refresh-token`| Refresh an OAuth access token.                     | Public |

### Blog Routes

| Method   | Endpoint           | Description                                    | Access               |
| :------- | :----------------- | :--------------------------------------------- | :------------------- |
| `GET`    | `/blogs`           | Get all blogs with pagination and sorting.     | Public               |
| `GET`    | `/blogs/search`    | Search blogs by a query keyword.               | Public               |
| `GET`    | `/blogs/filter`    | Filter blogs by tags and/or date range.        | Public               |
| `GET`    | `/blogs/:id`       | Get a single blog by its ID.                   | Public               |
| `POST`   | `/blogs`           | Create a new blog post.                        | Protected            |
| `PUT`    | `/blogs/:id`       | Update a blog post.                            | Protected (Author)   |
| `DELETE` | `/blogs/:id`       | Delete a blog post.                            | Protected (Author/Admin) |
| `POST`   | `/blogs/:id/like`  | Like or unlike a blog post.                    | Protected            |
| `POST`   | `/blogs/:id/dislike`| Dislike or remove dislike from a blog post.      | Protected            |

### Comment Routes

| Method   | Endpoint             | Description                         | Access               |
| :------- | :------------------- | :---------------------------------- | :------------------- |
| `GET`    | `/comments/:blogId`  | Get all comments for a blog post.   | Public               |
| `GET`    | `/comments/:blogId/:id`| Get a single comment by its ID.       | Public               |
| `POST`   | `/comments/:blogId`  | Create a new comment.               | Protected            |
| `PUT`    | `/comments/:blogId/:id`| Update a comment.                   | Protected (Author)   |
| `DELETE` | `/comments/:blogId/:id`| Delete a comment.                   | Protected (Author/Admin) |

### AI Routes

| Method | Endpoint        | Description                               | Access    |
| :----- | :-------------- | :---------------------------------------- | :-------- |
| `GET`  | `/ai/generate`  | Get content suggestions from the AI.      | Protected |

### Admin Routes
| Method | Endpoint             | Description                           | Access    |
| :----- | :------------------- | :------------------------------------ | :-------- |
| `POST` | `/admins/promote-demote`| Promote a user to admin or demote an admin to user. | Admin |

## 👩‍💻My Contributions (Feven-TH)

In this project, I contributed to the following features and improvements:

- Developed **CRUD endpoints for blog posts and comments** (create, read, update, delete)
- Implemented **filtering and search functionalities** for blogs
- Added **caching with thread-safe invalidation** to improve performance and consistency
- Implemented **popularity tracking using goroutines**, calculating blog scores based on likes, views, and comments

**Original repository:** [gedyzed/blog-starter-project](https://github.com/gedyzed/blog-starter-project)

