## TravelTide Mastery Project

This project uses TravelTide customer data to create business-relevant segments and deliver data-driven recommendations based on the insights you find.

## Introduction

TravelTide is a fast-growing e-booking startup and a relatively new entrant in the online travel market. Since launching near the end of the COVID-19 pandemic (2021-04), it has expanded steadily, powered by best-in-class data aggregation and search technology. However, parts of the customer experience remain underdeveloped, leading to low retention. To address this, the next step is to plan and run a marketing campaign with a personalized rewards program designed to bring customers back to the TravelTide platform. Five perks will be assigned to customers:

- Free hotel meal
- Free checked bag
- No cancellation fees
- Exclusive discounts
- 1 free hotel night with flight
## Planning

### Data overview

TravelTide provides four tables:

|            | flights    | hotels    | sessions    | users    |
|------------|------------|-----------|-------------|----------|
| Key        | trip_id    | trip_id   | session_id  | user_id  |
| dim/fact   | fact       | fact      | fact        | dim      |
| # columns  | 1,901,038  | 1,918,617 | 5,408,063   | 1,020,926|
| # rows     | 13         | 7         | 13          | 11       |

- **Users:** user-level information (e.g., demographic attributes).
- **Sessions:** session activity and behavior (e.g., clicks, session ID, booking and cancellation details).
- **Flights:** flight-related details (e.g., departure/arrival dates, home/destination airports, checked bags).
- **Hotels:** hotel-related details (e.g., property name/location, check-in/check-out dates, number of rooms).

### Workflow: assigning a personalized perk

1. **Identify active users:** filter for users with more than 7 sessions since January 4, 2023; reduce the dataset from 5M+ sessions to 5,998 active users.
2. **Explore the data:** analyze patterns and define customer groups that can be matched to the most relevant perk.

Rule Set for User Group Creation

| Segment | Characteristic |
| --- | --- |
| Age Group | 16–24, 25–34, 35–44, 45–54, 55–64, 65+ |
| Travel Frequency | frequent traveler (>= 5 bookings), casual traveler (2–4 bookings), low-frequent traveller (1 booking) |
| Spend | High spender (total cost > 7000 USD) |
| Flight Distance | short-haul-flyer (avg < 500 miles), medium-haul-flyer (avg 500–2500 miles), long-haul-flyer (avg > 2500 miles) |
| Booking Time | Last minute booker (< 7 days), Early booker (7–60 days), Moderate early booker (60–180 days), Super early booker (> 180 days) |
| Cancellation | Same-Day Cancellers, Last-Minute Cancellers (1–7 days), Moderate Cancellers (8–30 days), Early Cancellers (30+ days), No Cancellation |
| Travel Party | Solo traveler, Family/Group traveler, Solo and group traveler |
| Family | married, children, not married, children, married, no children, not married, no children |

### 3. Additional calculations and data preparation

- Recomputed hotel nights while excluding time-based NULL values (these appear only in cancellation sessions where `hotel_booked = false`).
- **Age:** calculated from `birthdate` and `session_end`.
- **Time since signup:** calculated as the difference between `sign_up_date` and `session_end`.
- **Stay duration:** calculated in hours using either `return_time - departure_time` (flights) or `check_out_time - check_in_time` (hotels).
- Linked cancellation information to the correct `session_id`.
- **Average days to cancellation:** calculated from the cancelled session’s `session_end` and the `session_start`.
- **Average session duration:** calculated as `session_end - session_start`.
- **Flight cost:** derived from `base_fare_usd`, adjusted for number of seats and `flight_discount_amount`.
- **Hotel cost:** derived from `hotel_per_room_usd`, adjusted for number of nights and `hotel_discount_amount`.
- **Average days to travel:** calculated as `(flight_departure OR hotel_check_in) - session_end`.
- **Travel party:** recoded `travel_party` (1 = solo, >1 = group/family).
- **Average distance (miles):** computed from home and destination airport latitude/longitude using the Haversine formula.

### 4. Decision tree and customer group assignment


```mermaid
flowchart TB
A[Frequent Traveler] -->|yes| B[Long Distance]
A -->|no| C[Casual Traveler]

B -->|yes| L1[Free Checked Bag - Mile Master]
B -->|no| D[Last Minute Booker]

D -->|yes| L2[No Cancellation Fees - Spontaneous Globetrotter]
D -->|no| E[Early Booker]

E -->|yes| L3[Exclusive Discounts - Jet Sprint Nomad]
E -->|no| F[Last Minute Cancellation]

F -->|yes| L4[No Cancellation Fees - Adaptive Explorer]
F -->|no| L5[Exclusive Discounts - Other]

C -->|yes| G[High Spender]
C -->|no| R1[1 Night free Hotel with Flight - Occasional Traveler]

G -->|yes| R2[1 Night free Hotel with Flight - High Spender]
G -->|no| H[Family, Married, Kids]

H -->|yes| R3[Free Hotel Meal - Family Traveler]
H -->|no| I[Group Traveler]

I -->|yes| R4[Free Hotel Meal - Group Traveler]
I -->|no| J[Age Group 65+]

J -->|yes| R5[No Cancellation Fees - Senior traveller]
J -->|no| K[Age Group 16-24]

K -->|yes| R6[Free Hotel Meal - Next-Gen Wanderer]
K -->|no| R7[No Cancellation Fees - Business Traveler]
