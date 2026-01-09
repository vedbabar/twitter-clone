# X-Clone: Full-Stack Social Media Platform

## 1. Overview

X-Clone is a modern, full-stack social media application designed to replicate core features of platforms like X (formerly Twitter). It provides secure user authentication, real-time content feeds, profile management, and social interactions (liking, commenting, following, and notifications).

The application is built with a decoupled architecture, featuring a robust Node.js/Express API and a fast, responsive React frontend.

## 2. Tech Stack

| Category | Technology | Purpose |
| :--- | :--- | :--- |
| **Backend** | Node.js, Express | Server runtime and REST API framework. |
| **Database** | MongoDB, Mongoose | NoSQL database and ODM for data persistence. |
| **Authentication** | JWT, `bcryptjs` | Secure session management and password hashing. |
| **Media Storage** | Cloudinary | External service for image upload and management. |
| **Frontend** | React, Vite | Component-based UI library and build tool. |
| **State Management** | TanStack Query (React Query) | Server state management, caching, and mutations. |
| **Styling** | Tailwind CSS, DaisyUI | Utility-first CSS framework and component library. |
| **Routing** | React Router DOM | Client-side routing and navigation. |

## 3. Project Structure

| Path | Role |
| :--- | :--- |
| `backend/controllers/` | Business logic for API endpoints (Auth, Posts, Users). |
| `backend/models/` | Mongoose schemas (`User`, `Post`, `Notification`). |
| `backend/routes/` | Express routing definitions and middleware application. |
| `backend/middleware/` | Security middleware, primarily `protectRoute.js`. |
| `backend/server.js` | Application entry point, configuration, and database connection. |
| `frontend/src/pages/` | Top-level components for application views (Home, Profile, Auth). |
| `frontend/src/components/` | Reusable UI components (Sidebar, Posts, RightPanel). |
| `frontend/src/hooks/` | Custom React hooks for centralized API logic (`useFollow`, `useUpdateUserProfile`). |
| `frontend/src/utils/` | Utility functions (date formatting, mock data). |

## 4. Key Components and Modules

### Backend Logic

*   **`authController.js`**: Manages user lifecycle (signup, login, logout). Uses `bcryptjs` for security and `generateTokenAndSetCookie` for session creation (HTTP-only JWT cookie).
*   **`postController.js`**: Handles all content operations (create, delete, like, comment). Integrates with Cloudinary for image management and creates `Notification` records for likes.
*   **`userController.js`**: Manages user relationships (follow/unfollow), profile viewing, and profile updates (including secure password changes and image replacement via Cloudinary).
*   **`protectRoute.js`**: Middleware that secures API endpoints by validating the JWT cookie and attaching the authenticated user object (`req.user`) to the request.

### Frontend State Management

*   **TanStack Query Integration**: The frontend relies heavily on TanStack Query for managing server state. Custom hooks like `useFollow` and `useUpdateUserProfile` centralize complex API interactions, handle loading states, and ensure immediate UI updates via cache invalidation.
*   **`App.jsx`**: The root component that defines global routing, enforces protected routes, and manages the global `authUser` state check via `useQuery`.
*   **`Post.jsx`**: An interactive component that handles client-side mutations for liking, commenting, and deleting posts, utilizing optimistic UI updates for likes.

## 5. Setup

### Prerequisites

1.  Node.js (v18+)
2.  MongoDB instance (local or cloud)
3.  Cloudinary account

### Installation Steps

1.  Clone the repository:
    ```bash
    git clone <repository-url>
    cd x-clone
    ```

2.  Install backend dependencies:
    ```bash
    cd backend
    npm install
    ```

3.  Install frontend dependencies:
    ```bash
    cd ../frontend
    npm install
    ```

4.  Create a `.env` file in the `backend` directory based on the configuration section below.

## 6. Usage

To run the application, you must start both the backend API server and the frontend development server concurrently.

1.  **Start the Backend API:**
    (From the `backend` directory)
    ```bash
    npm start
    ```
    *The backend will connect to MongoDB and start listening on the configured port (default 5000).*

2.  **Start the Frontend Development Server:**
    (From the `frontend` directory)
    ```bash
    npm run dev
    ```
    *The Vite development server will start, typically on port 3000, and automatically proxy `/api` requests to the backend.*

The application will be accessible at `http://localhost:3000`.

### Key API Endpoints (Quick Reference)

| Method | Endpoint | Description | Security |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/signup` | Register a new user. | Public |
| `POST` | `/api/auth/login` | Authenticate and set JWT cookie. | Public |
| `GET` | `/api/posts/all` | Fetch the global content feed. | Protected |
| `POST` | `/api/posts/create` | Create a new post (text/image). | Protected |
| `POST` | `/api/users/follow/:id` | Toggle follow status for a user. | Protected |
| `GET` | `/api/users/suggested` | Get a list of users to follow. | Protected |
| `GET` | `/api/notifications` | Retrieve and mark all notifications as read. | Protected |

## 7. Configuration

Create a `.env` file in the `backend` directory with the following variables:

| Name | Purpose | Required | Default |
| :--- | :--- | :--- | :--- |
| `PORT` | Backend server port. | No | `5000` |
| `MONGO_URI` | MongoDB connection string. | Yes | - |
| `JWT_SECRET` | Secret key for signing JWTs. | Yes | - |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary account name. | Yes | - |
| `CLOUDINARY_API_KEY` | Cloudinary API key. | Yes | - |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret. | Yes | - |

## 8. Data Model

The application uses three primary Mongoose models to structure data:

### User
*   **Core Identity:** `username` (unique), `email` (unique), `fullName`, `password` (hashed).
*   **Relationships:** `followers` (array of User IDs), `following` (array of User IDs).
*   **Content:** `likedPosts` (array of Post IDs).
*   **Profile:** `profileImg`, `coverImg`, `bio`, `link`.

### Post
*   **Content:** `text`, `img` (Cloudinary URL).
*   **Author:** `user` (reference to User ID, required).
*   **Interactions:** `likes` (array of User IDs), `comments` (array of nested sub-documents with `user` and `text`).

### Notification
*   **Recipient:** `to` (reference to User ID, required).
*   **Sender:** `from` (reference to User ID, required).
*   **Status:** `read` (boolean, defaults to `false`).
*   **Type:** `type` (string, restricted to `'follow'` or `'like'`).

## 9. Deployment

The application is structured for easy deployment:

*   **Backend:** The `backend/server.js` file includes logic to serve the static frontend assets from the `../frontend/dist` directory when running in a production environment.
*   **Frontend Build:** Use the Vite build command to generate production assets:
    ```bash
    cd frontend
    npm run build
    ```
*   **Containerization:** The use of environment variables and separate `backend` and `frontend` directories makes the project suitable for Docker and deployment on services like Render, Vercel (for frontend), or any standard cloud VM. The backend requires access to the MongoDB URI and Cloudinary credentials.
