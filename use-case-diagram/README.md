

### 1. Identifying Actors and Use Cases

* The image below details the general overview [here](usecase.png)

#### Actors:

*   **Primary Actors (Humans):**
    *   **User:** A general, unauthenticated person visiting the site.
    *   **Guest:** An authenticated user who books properties. (Inherits from User)
    *   **Host:** An authenticated user who lists properties. (Inherits from User)
    *   **Admin:** A privileged user who manages the platform. (Inherits from User)
*   **Secondary Actors (External Systems):**
    *   **Payment Gateway:** An external service like Stripe or PayPal that processes payments.
    *   **OAuth Provider:** An external service like Google or Facebook for social login.
    *   **Email Service:** An external service like SendGrid that sends notifications.

#### Key Use Cases (Actions):

*   **General/User Management:**
    *   Register Account
    *   Log In with Email
    *   Log In with OAuth
    *   Manage Profile
    *   Search Listings
    *   View Listing Details
*   **Guest Specific:**
    *   Book Property
    *   Make Payment
    *   Cancel Own Booking
    *   Write Review
*   **Host Specific:**
    *   Create Listing
    *   Manage Own Listings
    *   Cancel Guest Booking
    *   Respond to Review
*   **Admin Specific:**
    *   Manage Users
    *   Manage All Listings
    *   Manage All Bookings
*   **System Internal/Triggered:**
    *   Send Notification

------