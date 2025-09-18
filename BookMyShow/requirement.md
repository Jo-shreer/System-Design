Designing a BookMyShow or Movie Ticket Booking System involves building a system that allows users to:
Browse movies and shows
Select a theater
Choose seats
Book and pay for tickets
Receive confirmations (email/SMS)
Allow theaters to manage screens and show timings.


# HLD
+----------------+      +-----------------+      +-----------------+
|     Client     | ---> |  API Gateway    | ---> |  Microservices  |
| (Web/Mobile)   |      | (Authentication)|      | (Booking, User, |
+----------------+      +-----------------+      |  Movie, Payment)|
                                                     |
                                                     v
                                         +-----------------------+
                                         |    Databases / Cache  |
                                         +-----------------------+
# Microservices

User Service	
  Signup, login, authentication (JWT/OAuth), profile
  
Movie Service	
  Movies, genres, languages, trailers, metadata
  
Theater Service	
  Theater info, screens, locations
  
Showtime Service	
  Mapping between movies and theaters with timings
  
Seat Service	
  Seat layout per screen; real-time availability
  
Booking Service	
    Booking logic, locks seats, generates tickets
    
Payment Service	
  Integrates with payment gateways (Stripe, Razorpay)
  
Notification Service	
  Email/SMS confirmations

# Real-Time Seat Locking (Concurrency Handling)

To avoid overbooking:
When a user selects seats, lock those seats for X minutes (e.g., 5 minutes).
Use distributed locking (e.g., Redis with RedLock algorithm).
If payment is successful → update status to BOOKED.
If payment fails or timeout → revert to AVAILABLE.


