# StayLink

StayLink is a responsive hotel and resort reservation web application. Guests
can discover rooms, review availability, calculate a stay total, create an
account, and manage their reservations. Staff members get a protected
dashboard for monitoring bookings, occupancy, revenue, and room status.

The project is designed as a lightweight capstone application: the front end
is a single HTML page, while Firebase provides authentication and live
transactional data.

## What the web app includes

### Guest experience

- **Discover page** with a hotel introduction and booking search form.
- **20-room Philippine catalogue** covering Manila, Cebu, Boracay, Palawan,
  Davao, Baguio, Batangas, Tagaytay, Bohol, Siargao, and Iloilo.
- **Room catalogue** with destination and room-type filtering, availability
  counts, and sorting by rating or price.
- **Room details** including photos, description, amenities, bed type, room
  size, view, guest capacity, rating, and reviews.
- **Reservation flow** with check-in, check-out, guest count, guest details,
  exact check-in time, contact number, optional breakfast and wellness spa
  extras, 12% tax calculation, and a generated Firebase reservation ID.
- **Email/password authentication** with sign-in and account creation.
- **My Stays** page showing only the signed-in guest's reservations.
- **Reservation cancellation** that changes a reservation's status instead of
  deleting its record.
- inventory-safe booking with Firestore transactions and availability restore
  on cancellation.

### Loyalty and voucher rules

Points are awarded only after a confirmed stay's checkout date has passed.
The reservation is then marked `completed` and locked from cancellation.
Points are calculated once from the final total:

```text
floor((final total / 100) × current tier rate)
```

The `loyaltyPointsAwarded` flag and a Firestore transaction make completion
idempotent across refreshes and live listeners. Voucher redemptions also use a
transaction, verify the member tier and balance, deduct points atomically, and
prevent the same voucher from being redeemed twice.
- **Profile page** with the current account information and sign-out action.
- **Profile preferences** for room type, bed, smoking, notifications,
  language, and PHP currency.
- **Rewards page** with Bronze, Silver, Gold, and Platinum/Diamond tiers,
  progress tracking, points, and Firebase-backed voucher redemption.
- **StayLink Promise** cards for Concierge, Fine Dining, Wellness Spa, and
  Valet, with responsive detail modals.
- **Responsive navigation** for desktop and mobile layouts.

### Staff experience

Accounts whose email is on the admin allow-list can open the **Admin
Dashboard**. The dashboard contains:

- A staff-only overview with total upcoming bookings for the week, currently
  occupied rooms, rooms needing cleaning, and registered guests.
- Quick actions for opening all bookings and room inventory management.
- A searchable and filterable reservations table with guest/email, room, dates,
  total price, and `Pending`, `Confirmed`, `Checked-In`, `Checked-Out`, or
  `Cancelled` status.
- Per-booking **Check-In**, **Check-Out**, **Cancel Booking**, and **View
  Details** actions. Cancellation releases the room inventory.
- A room and inventory grid with `Available`, `Occupied`, and `Cleaning`
  status controls, editable nightly rates, and room visibility toggles.
- A guest directory showing account email, join date, booking count, loyalty
  tier, and points.

Admins do not see guest booking navigation or client pages; signing in with an
allow-listed staff account opens the staff dashboard directly. The dashboard
and room availability update through Firestore snapshot listeners, so changes
are reflected without manually refreshing the page.

## Technology

- HTML, CSS, and vanilla JavaScript
- Tailwind CSS via CDN
- Firebase Authentication
- Cloud Firestore
- Firebase Hosting
- Google Fonts: Playfair Display and Outfit
- Unsplash room photography

There is no build step or npm project. The deployable site is the
[StayLink Hotel Reser](./StayLink%20Hotel%20Reser/) directory, whose
`index.html` contains the application UI and logic.

No environment variables are required by the current Firebase Hosting
configuration. Firebase supplies the client configuration through
`/__/firebase/init.js`; do not add API keys or service-account credentials to
the repository.

## Project structure

```text
StayLink/
├── StayLink Hotel Reser/
│   └── index.html          # Main single-page web application
├── firebase.json           # Hosting and Firestore deployment configuration
├── firestore.rules         # Database access rules
├── SETUP_FIREBASE.md       # Firebase console setup instructions
├── README.md               # Project documentation
└── staylink(old file lang).html
                            # Older non-Firebase version kept for reference
```

## How the application works

1. A visitor opens the Discover page and searches for a stay.
2. The visitor browses the room catalogue and opens a room's detail view.
3. Selecting **Reserve** requires authentication. A visitor can sign in or
   create an account and then return to the selected room.
4. The app calculates the subtotal from the nightly rate and number of nights,
   adds 12% tax, and writes the reservation to Firestore.
5. The room is marked `Occupied`, and the reservation appears in **My Stays**.
6. A guest can cancel their own reservation. Cancellation is stored as a
   status update so the reservation history remains available.
7. An allow-listed staff account can review all reservations and update room
   or reservation statuses from the dashboard.

## Firebase data model

### `users/{userId}`

Signed-in users can store their full name, email, address, phone, preferences,
loyalty points, membership tier, and redeemed vouchers. Profile data is
restricted to the owning Firebase Auth UID.

### `rooms/{roomId}`

Room descriptions and content are defined in the `ROOMS` array in
`StayLink Hotel Reser/index.html`. Firestore stores the mutable room status:

```text
status: "Available" | "Occupied" | "Cleaning"
availabilityCount: number
```

Room documents are publicly readable so visitors can see live availability.
Only admin accounts can write room status.

### `reservations/{reservationId}`

Reservation documents contain values including:

```text
reference, roomId, roomName, roomType
reservationId, destination, location, roomImage
guestUid, guestName, guestEmail
leadGuest, contactNumber, checkin, checkout, checkinTime
nights, guests, extras, roomPrice, extrasPrice
subtotal, taxes, totalPrice, total
status, createdAt, updatedAt, createdAtMs
cancellationReason, cancelledAt
loyaltyPointsAwarded, loyaltyPointsEarned, loyaltyLockedAt
```

Guests can create and read their own reservations. Admins can read and update
all reservations. Hard deletion is disabled; cancellation is represented by
`status: "cancelled"`.

## Local setup

Firebase must initialize the app before authentication, Firestore, and
reservation persistence can work. Opening the HTML file directly with
`file://` is not supported.

### Prerequisites

- Node.js and npm, so the Firebase CLI can be installed
- A Firebase project
- A Firebase CLI login

### 1. Install and authenticate the Firebase CLI

```bash
npm install -g firebase-tools
firebase login
```

### 2. Configure Firebase

Follow [SETUP_FIREBASE.md](./SETUP_FIREBASE.md) to:

1. Enable Email/Password sign-in.
2. Create a Firestore database.
3. Add the staff email to both admin allow-lists.

The admin email must be kept synchronized in:

- `StayLink Hotel Reser/index.html` (`ADMIN_EMAILS`)
- `firestore.rules` (`isAdminEmail()`)

### 3. Run locally with Firebase

From the repository root:

```bash
firebase emulators:start
```

Use the Hosting emulator URL shown in the terminal. Do not double-click
`index.html`, because Firebase's `/__/firebase/init.js` endpoint is only
available through Firebase Hosting or the emulator.

### 4. Deploy the site

From the repository root:

```bash
firebase deploy
```

The `firebase.json` configuration publishes `StayLink Hotel Reser` and
rewrites application routes to `index.html`. Firestore rules are deployed
alongside Hosting.

To deploy only one part:

```bash
firebase deploy --only hosting
firebase deploy --only firestore:rules
```

### Development validation

This repository is a static Firebase Hosting project and currently has no
package manifest, type-checker, linter, unit-test runner, or production
bundler. Use the Firebase Hosting emulator for development and validate
changes with:

```bash
firebase emulators:start
git diff --check
```

## Security notes

Firestore rules are the actual access-control boundary:

- Anyone may read room availability.
- A signed-in guest may create a reservation only when `guestUid` matches
  their own Firebase Auth UID.
- Guests may read or update only their own reservations.
- Admins may read and update all reservations and room statuses.
- Reservation deletion is denied.

The client-side `ADMIN_EMAILS` array controls which dashboard controls are
shown, but it is not trusted for authorization. Any change to the admin list
must also be made in `firestore.rules`.

## Current scope and future improvements

This version intentionally keeps the implementation simple for a capstone
demonstration:

- Room names, descriptions, photos, amenities, rates, and ratings are static
  JavaScript data rather than Firestore content.
- Availability is currently an inventory count per room document; date-range
  overlap checks are not yet implemented.
- Admin access uses an email allow-list rather than Firebase custom claims.
- Payments, email confirmations, image uploads, and production-grade
  availability locking are not implemented.

Possible next steps include date-aware inventory, Firebase custom claims,
payment integration, server-side booking transactions, image management, and
automated reservation emails.

## Troubleshooting

| Problem | Solution |
| --- | --- |
| Firebase Auth or Firestore does nothing | Open the app through Firebase Hosting or `firebase emulators:start`, not as a local file. |
| The Dashboard link is missing | Sign in with an email present in both admin allow-lists. |
| A room status does not update | Confirm Firestore exists and that the signed-in account is an allowed admin. |
| Guests cannot create reservations | Enable Email/Password authentication and verify the Firestore rules are deployed. |
| The app shows stale data | Check the browser console and Firebase console for rejected Firestore reads or writes. |

## License

This project is intended for educational and demonstration use.
