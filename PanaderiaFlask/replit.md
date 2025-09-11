# Sweet Delights Bakery

## Overview

Sweet Delights Bakery is a web-based e-commerce platform for a bakery business. The application allows customers to browse products organized by categories, add items to their shopping cart, place orders, and manage their accounts. It includes an admin panel for managing products, categories, and orders. The system is built with Flask and provides a complete online bakery shopping experience with user authentication, cart management, and order processing.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Backend Architecture
- **Framework**: Flask web framework with blueprints for modular organization
- **Database ORM**: SQLAlchemy with Flask-SQLAlchemy integration for database operations
- **Authentication**: Flask-Login for session management and user authentication
- **Database Migrations**: Flask-Migrate for schema version control
- **Form Handling**: WTForms with Flask-WTF for form validation and CSRF protection

### Application Structure
- **Modular Design**: Application organized into blueprints (auth, admin, cart, products) for separation of concerns
- **MVC Pattern**: Clear separation between models (database entities), views (templates), and controllers (route handlers)
- **Factory Pattern**: Application factory pattern in app.py for configuration flexibility

### Database Schema
- **User Model**: Stores user credentials, admin status, and relationships to orders and cart items
- **Product Model**: Contains product information including name, description, price, category, and status
- **Category Model**: Organizes products into categories with descriptions
- **Cart Model**: Manages shopping cart items with user-product relationships and quantities
- **Order System**: Supports order processing with order items tracking

### Authentication & Authorization
- **User Authentication**: Email-based login system with password hashing using Werkzeug
- **Role-Based Access**: Admin users have access to management features
- **Session Management**: Flask-Login handles user sessions and login state
- **Form Security**: CSRF protection on all forms

### Frontend Architecture
- **Template Engine**: Jinja2 templating with template inheritance
- **CSS Framework**: Bootstrap 5 with dark theme for responsive design
- **Client-Side Logic**: JavaScript for cart interactions and form validations
- **Icon System**: Font Awesome for consistent iconography

### Security Features
- **Password Security**: Werkzeug password hashing for secure credential storage
- **CSRF Protection**: Built-in protection against cross-site request forgery
- **Environment Variables**: Sensitive configuration stored in environment variables
- **Proxy Security**: ProxyFix middleware for secure proxy handling

### Data Management
- **Pagination**: Built-in pagination for product listings and admin views
- **Search Functionality**: Text-based search across product names and descriptions
- **Category Filtering**: Product filtering by category with dropdown navigation
- **Form Validation**: Server-side validation for all user inputs

## External Dependencies

### Core Framework Dependencies
- **Flask**: Main web framework
- **SQLAlchemy**: Database ORM and management
- **Flask-Login**: User session management
- **Flask-Migrate**: Database migration handling
- **WTForms**: Form processing and validation

### Frontend Dependencies
- **Bootstrap 5**: CSS framework for responsive design
- **Font Awesome**: Icon library for UI elements
- **Custom CSS**: Application-specific styling enhancements

### Database Support
- **SQLite**: Default database for development (fallback)
- **PostgreSQL**: Production database support via DATABASE_URL environment variable
- **Connection Pooling**: Configured for production database optimization

### Development Tools
- **Werkzeug**: Development server and security utilities
- **Proxy Middleware**: Production deployment support for reverse proxy setups

### Environment Configuration
- **DATABASE_URL**: Database connection string
- **SESSION_SECRET**: Session encryption key
- **Development Mode**: Debug configuration for local development