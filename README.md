# Automated Attendance Management System (AAMS) - Backend

## Overview
The **Automated Attendance Management System (AAMS)** is a backend system designed to streamline and automate student attendance tracking. Students scan their IDs upon entering the class, and the system logs their attendance securely and efficiently.

This backend provides:
- **User authentication and security** (JWT-based authentication)
- **Role-based access control (RBAC)** (Students, Teachers, Admins, System Admins)
- **Attendance tracking and validation**
- **Class and session management**
- **Reporting and analytics**

## Features
- **Secure Authentication**: JWT-based authentication with refresh tokens.
- **Role-Based Access Control (RBAC)**: Different access levels for Students, Teachers, and Admins.
- **Automated Attendance Logging**: Students scan IDs, and attendance is recorded.
- **Real-time Attendance Status**: Teachers can monitor real-time class attendance.
- **Customizable Attendance Rules**: Define grace periods, minimum attendance percentage, etc.
- **Reports & Analytics**: Generate attendance reports and track trends.

## Security

The **AAMS Backend** is designed with **Spring Security** and **JWT-based authentication** to ensure data protection and secure access control. It implements **role-based access control (RBAC)**, token security, and various security best practices.

### **Security Features**
| Feature           | Description |
|------------------|-------------|
| **JWT Authentication** | Stateless authentication with short-lived access tokens and refresh tokens |
| **Role-Based Access Control (RBAC)** | Restricts access based on user roles (Student, Teacher, Admin, System Admin) |
| **Password Security** | Passwords are securely hashed with **BCrypt** |
| **Token Revocation** | Tokens are invalidated on logout or password change |
| **Security Headers** | Protects against XSS, Clickjacking, and Injection attacks |
| **Logout & Session Management** | Clears security context and invalidates all active tokens |
| **Database-Stored Refresh Tokens** | Ensures refresh tokens can be revoked upon logout |

### **Authentication & Token Management**
| Method | Endpoint | Description |
|--------|---------|-------------|
| `POST` | `/api/auth/login` | Authenticate user and return access and refresh tokens |
| `POST` | `/api/auth/refresh-token` | Obtain a new access token using a refresh token |
| `GET`  | `/api/auth/verify-access-token` | Verify if an access token is valid |
| `GET`  | `/api/auth/verify-refresh-token` | Verify if a refresh token is valid |
| `POST` | `/api/auth/logout` | Invalidate the user's current session |
| `POST` | `/api/auth/logout/all` | Log out the user from all active sessions |

### **Access Control & Protection**
| Method | Endpoint | Description |
|--------|---------|-------------|
| `GET`  | `/api/user/details` | Retrieve authenticated user details |
| `POST` | `/api/user/assign-role` | Assign a role to a user (Admin/System Admin only) |
| `POST` | `/api/auth/change-password` | Change the authenticated user's password |
| `DELETE` | `/api/user/delete` | Delete the authenticated user's account |

📌 **[See full security details](docs/security.md)**




## User Management

The **Automated Attendance Management System (AAMS)** provides a robust user management system that supports role-based access control (RBAC), authentication, and user lifecycle management.

### **Roles & Permissions**
| Role             | Permissions |
|-----------------|-------------|
| **Student**      | View personal attendance records |
| **Teacher**      | Manage class attendance, view student attendance |
| **Admin**        | Manage users, classes, attendance records |
| **System Admin** | Full system control, user management, security policies |

### **User Management Endpoints**
| Method | Endpoint | Description |
|--------|---------|-------------|
| `POST` | `/api/auth/register` | Register a new user |
| `POST` | `/api/auth/login` | Authenticate user and return tokens |
| `POST` | `/api/auth/refresh-token` | Refresh access token |
| `GET`  | `/api/auth/verify-token` | Verify if a token is valid |
| `POST` | `/api/auth/change-password` | Change user password |
| `DELETE` | `/api/user/delete` | Delete a user account |
| `PUT` | `/api/user/update` | Update user details |
| `GET` | `/api/user/details` | Retrieve authenticated user details |
| `POST` | `/api/user/assign-role` | Assign a role to a user (Admin/System Admin only) |

📌 **[See full user management details](docs/user-management.md)**

---
## Technologies Used
- **Backend**: Java Spring Boot
- **Authentication**: JWT with Refresh Tokens
- **Database**: PostgreSQL
- **Logging**: SLF4J with Logback
- **Security**: Spring Security, BCrypt password hashing
- **Containerization**: Docker (Planned)
- **Cloud Deployment**: AWS (Planned)
