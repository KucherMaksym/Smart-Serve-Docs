# Restaurant POS System - Technical Documentation

## Overview

A comprehensive Point of Sale (POS) system designed for restaurants, cafes, and bars that streamlines operations for owners, waiters, and customers. The system facilitates table management, order processing, real-time notifications, and integrated payment processing.

**Project Scale**: ~50,000 lines of code (40% backend, 60% frontend)

## Key Features

### For Restaurant Owners

- Multi-restaurant management capabilities
- Staff management and invitation system
- Dynamic menu creation and editing
- Table layout designer with room management
- Comprehensive analytics and statistics
- Subscription-based pricing with different tiers
- Integration with Stripe Connect for payment processing

### For Waiters

- Clock-in/out functionality with shift tracking
- Real-time order management
- Push notifications for customer actions
- Dish status management (accept/cancel/deliver)
- Direct chat with customers
- Break time tracking

### For Customers

- QR code-based table ordering (registration optional)
- Real-time order tracking
- Direct communication with waiters
- Multiple payment options (cash/card/online)
- Order history (for registered users)
- Multi-participant order support

## Technical Architecture

### Backend Architecture

- **Type**: Monolithic architecture
- **Framework**: Spring Boot (Java)
- **Database**: PostgreSQL (~40 tables)
- **Caching**: Redis (sessions and cache)
- **Message Queue**: RabbitMQ (notifications delivery, payment processing)
- **Monitoring**: ELK Stack
- **Web Server**: Nginx
- **CI/CD**: GitHub Actions

### Frontend Architecture

- **Framework**: Next.js with TypeScript
- **Router**: App Router
- **State Management**: React Query
- **Styling**: Tailwind CSS
- **UI Components**: Custom components with minimal HeroUI usage
- **Animations**: Framer Motion / React Spring
- **Table Layout Designer**: PixiJS
- **Maps**: Leaflet
- **Theme Management**: next-theme
- **Folder Structure**:
  - `/app` - Next.js app directory
  - `/components` - Reusable UI components
  - `/hooks` - Custom React hooks
  - `/utils` - Utility functions
  - `/api` - API integration layer
  - `/models` - TypeScript interfaces and types
  - `/constants` - Application constants

### API Architecture

- **REST API**: Primary communication protocol
- **WebSockets**: Real-time updates for orders and notifications
- **Webhooks**: Stripe payment events

## Core Technologies

### Backend Stack

- **Spring Boot Modules**:
  - Spring Security (Authentication & Authorization)
  - Spring Web (REST API)
  - Spring WebSocket (Real-time communication)
  - Spring Validation (Input validation)
  - Spring Data JPA (Database ORM)
- **Additional Libraries**:
  - JWT for authentication tokens
  - Stripe SDK for payment processing
  - SMTP for email notifications
  - QR code generation libraries
  - Cloudinary SDK for image storage
  - Lombok for boilerplate reduction
  - MapStruct for object mapping
  - Testing: JUnit, Mockito, H2

### Design Patterns

- **Dependency Injection & Inversion of Control**: Core Spring patterns
- **Singleton Pattern**: WebSocket connection manager
- **Factory Pattern**: Extensively used in testing
- **Repository Pattern**: Data access layer
- **Service Pattern**: Business logic layer
- **Event-Driven Architecture**: Spring Events for decoupled components

## Security Architecture

### Authentication Methods

1. **Standard Authentication**: Email/password with sessions in Redis
2. **PIN Authentication**:
   - Quick authentication for shared devices
   - Shorter session duration
   - Limited permissions (methods require `@AllowPinAuth` annotation)
   - Available only on connected devices
3. **Order-Level Authorization**: Only order participants and restaurant staff can manage orders

### Security Features

- Email verification with OTP codes
- Role-based access control (RBAC)
- Global roles (Admin/User) and restaurant-specific roles
- Soft delete for data retention compliance
- User data export functionality
- Session management with Redis
- CORS configuration for cross-origin requests

### Connected Device Management

- **Device Registration**: Invitation link-based authentication
- **Token Expiration**: Time-limited access tokens (few hours)
- **Device Tracking**: Active device monitoring and management

## Data Model Architecture

### Core Entities

#### User Management

- **UserEntity**: Core user profile with authentication details
- **RestaurantUser**: Restaurant-specific user roles and permissions
- **RestaurantOwner**: Restaurant ownership relationship
- **OrderParticipant**: User participation in orders

#### Restaurant Structure

- **Restaurant**: Main restaurant entity with profile, location, and settings
- **Menu**: Restaurant menu container
- **MenuCategory**: Menu organization structure
- **Dish**: Individual menu items with allergen tracking
- **Layout**: Restaurant floor plan
- **RoomEntity**: Room definitions within layout
- **TableEntity**: Individual table configurations

#### Order Management

- **OrderEntity**: Main order container with version control
- **OrderDish**: Individual items within an order
- **OrderStatus**: Order lifecycle states
- **OrderNotification**: Order-related notifications
- **OrderDishStatus**: Individual dish states

#### Payment & Billing

- **Payment**: Payment transaction records
- **StripeAccount**: Restaurant's Stripe Connect integration
- **Subscription**: Restaurant subscription management
- **SubscriptionPlan**: Available subscription tiers

#### Operations

- **WorkShift**: Employee shift tracking
- **Break**: Break time management
- **ConnectedDevice**: Registered restaurant devices
- **DeviceConnectionToken**: Device authentication tokens

#### Analytics

- **DailyRevenue**: Daily revenue aggregation
- **DishStatistics**: Dish performance metrics
- **HourlyActivity**: Hourly activity patterns
- **OrderStatistics**: Order completion metrics

## Business Workflows

### Restaurant Owner Flow

1. **Registration** → Email verification
2. **Restaurant Setup**:
   - Basic info (name, description, currency)
   - Cuisine types and operating hours
   - Features and amenities
   - Location and contact details
   - Media upload (logo, cover, gallery)
3. **Subscription Selection**: Choose plan with specific limits and features
4. **Configuration**:
   - Menu creation with categories and dishes
   - Table layout design with rooms
   - Staff invitations
5. **Operations**:
   - QR code generation for tables
   - Analytics dashboard access
   - Staff performance monitoring

### Waiter Flow

1. **Registration** → Accept invitation
2. **Shift Management**:
   - Clock In → Begin shift
   - Break management (pause notifications)
   - Automatic clock out after 12 hours
3. **Order Management**:
   - Accept/create orders
   - Manage dish statuses
   - Communicate with customers
   - Process payments
4. **Clock Out** → End shift

### Customer Flow

1. **Table Access**: Scan QR code (registration optional)
2. **JWT Token**: Receive order access token
3. **Order Creation**:
   - Browse menu
   - Add dishes with comments
   - Invite other participants
4. **Order Tracking**: Real-time status updates
5. **Payment**: Choose payment method (cash/card/online)

## Advanced Features

### Real-time Order Management

#### WebSocket Architecture

- **WebSocketManager**: Singleton pattern implementation managing all connections
- **Automatic Reconnection**: Built-in disconnection handling and recovery
- **Topic Structure**:
  - **Personal Topics**: Direct messages to assigned waiters
  - **Broadcast Topics**: Unassigned orders to all waiters
  - **Order-specific Topics**: Internal order communication channels
- **Message Routing**: Smart routing based on order assignment status

#### Delta Update System

- **Update Types**:
  - `STATUS`: Order status changes
  - `DISHES_ADDED`: New dishes added to order
  - `DISHES_MODIFIED`: Existing dish modifications
  - `DISHES_REMOVED`: Dish cancellations
  - `PARTICIPANTS_ADDED`: New participants joining
  - `PARTICIPANTS_REMOVED`: Participants leaving
  - `WAITER_ASSIGNED`: Waiter assignment changes
  - `PAYMENT_STATUS`: Payment state updates
  - `TABLE_CHANGED`: Table reassignments
- **Version Control**: Sequential version numbers ensure proper order application
- **Conflict Resolution**: Automatic refetch on version mismatch
- **Periodic Sync**: Regular full state synchronization to ensure consistency
- **BasicOrder Model**: Visual table status for waiters
- **Multi-participant Support**: Concurrent order editing

### Notification System

#### Visual Indicators

- **PixiJS Integration**: Interactive table layout with real-time status
- **SVG Icons**: Dynamic notification badges on tables
- **Interaction**: Click-to-dismiss for read status
- **Persistence**: Icons remain until explicitly marked as read

#### Notification Distribution Logic

- **Assigned Orders**: Notifications sent only to the assigned waiter
- **Unassigned Orders**: Broadcast to all active waiters
- **Icon Types**: Different icons for various notification types
- **Real-time Delivery**: Immediate WebSocket push

#### System Components

- **Customer Notifications**: Order status changes
- **Waiter Notifications**: Customer actions with visual indicators
- **WebSocket Broadcasting**: Real-time delivery
- **Notification Recipients**: Targeted delivery system

### Image Management

- **Cloudinary Integration**: Automatic optimization
- **Resolution Hook**: Dynamic image sizing
- **CDN Delivery**: Fast global access

### Reservation System

#### Availability Algorithm

- **Time Slot Generation**: Dynamic slots based on:
  - Existing reservations
  - Restaurant operating hours
  - Table capacity constraints
- **Real-time Availability Check**:
  - Considers restaurant timezone
  - Validates against existing reservations
  - Accounts for reservation duration

#### Reservation Lifecycle

- **Status Flow**: PENDING → CONFIRMED → CHECKED_IN → COMPLETED
- **Alternative Flows**: NO_SHOW, CANCELLED
- **Order Integration**: Direct order creation from reservation
- **Time-based Status**: ReservationTimeStatus for visual indicators
  - Color-coded table display
  - Countdown to reservation start
  - Business rule enforcement (e.g., no-show restrictions)

#### Table Availability

- **Online Booking**: Public restaurant page
- **Real-time Checking**: Instant availability validation
- **Integration**: Seamless link with order system

### Analytics & Reporting

- **Real-time Metrics**: Live business performance tracking
- **Revenue Analytics**: Daily and monthly aggregations
- **Activity Heat Maps**: Hourly order distribution by day of week
- **Performance Metrics**:
  - Average order completion time
  - Average check size
  - Staff performance indicators
- **Data Aggregation**: Efficient statistical processing
- **Historical Trends**: Long-term business insights

## Payment System

### Stripe Connect Integration

- **Onboarding Flow**:
  1. Restaurant owner enters email
  2. Stripe account automatically created
  3. Owner completes Stripe onboarding process
  4. Webhook receives completion status
  5. Restaurant payment processing activated
  
- **Commission Structure**:
  - Platform fee percentage based on subscription tier
  - Automatic commission deduction from order total
  
- **Payment Processing**:
  - Support for card and online payments
  - Cash payment tracking
  - Failed payment handling
  
- **Webhook Integration**:
  - Real-time payment status updates
  - Account status synchronization
  - Automated retry mechanism

## Subscription Tiers

Different subscription plans offer varying limits on:

- Number of tables and rooms
- Number of waiters
- Menu items and categories
- Connected devices
- Advanced analytics access
- Payment processing fees

## Performance & Optimization

### Database Optimization

- **Indexing**: Optimized indexes for critical query paths
- **Query Performance**: Efficient JPA queries with proper fetching strategies

### Caching Strategy

- **Redis Cache**: Frequently accessed and heavy data caching
- **Session Storage**: Fast session retrieval
- **Cache Invalidation**: Smart cache updates on data changes

### API Optimization

- **Rate Limiting**: Nginx-level request throttling
- **Response Compression**: Gzip compression for API responses
- **Load Balancing**: Nginx reverse proxy configuration

### Frontend Optimization

- **Image Optimization**: Cloudinary automatic format selection
- **React Query**: Intelligent data fetching and caching
- **WebSocket Efficiency**: Delta updates minimize data transfer

### Time Zone Management

- **Backend Storage**: UTC timestamps using Instant
- **Restaurant Time Zone**: Stored in location profile
- **Client-side Formatting**: Automatic conversion to local time
- **Reservation Handling**: Proper timezone consideration for bookings

### REST API Structure

- **URL Pattern**: `/api/restaurants/{id}/resource` format
- **Endpoint Organization**: Centralized endpoint definitions on client
- **Response Format**: Standardized JSON responses
- **Authentication**: JWT Bearer tokens

### Error Handling

- **Global Exception Handler**: Centralized error processing
- **Custom Exceptions**: Business-specific error types
- **User-Friendly Messages**: "Unknown error" for unhandled system errors
- **Structured Error Responses**: Consistent error format across API

## External Integrations

- **Unit Tests**: JUnit with Mockito
- **Integration Tests**: H2 in-memory database
- **API Tests**: REST endpoint validation

## Deployment Architecture

- **Containerization**: Docker support
- **Reverse Proxy**: Nginx configuration
- **SSL/TLS**: HTTPS enforcement
- **Monitoring**: ELK Stack integration
- **CI/CD**: Automated GitHub Actions pipelines

## Conclusion

This Restaurant POS System represents a comprehensive solution for modern restaurant management, combining powerful features with intuitive interfaces for all user types. The architecture prioritizes real-time performance, scalability, and security while maintaining flexibility for future enhancements. The subscription-based model ensures sustainable growth while providing value at every tier.

## Contact

For more information you can contact me: [Telegram](https://t.me/Clusteeer), [LinkedIn](https://linkedin.com/in/maksym-kucher-a4b37831a), [Email](mailto:kuchermaksym07@gmail.com)
