### DB architecture

<u>**ENTITY 1:  USER**</u>

Need: Stores common information about everyone who uses AutoQueue, regardless of their role.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| userId | UUID | PK | Unique user identifier |
| name | VARCHAR  |  | Full name |
| email | VARCHAR | UNIQUE | login email |
| passwordHash | VARCHAR |  | Hashed Password |
| phone | VARCHAR |  | Contact number |
| role | ENUM |  | CUSTOMER/DEALER/ADMIN |
| createdAt | TIMESTAMP |  | Account creation date |
| updatedAt | TIMESTAMP |  | Last profile update |

Relationships:

- One User can have zero or one Dealer profile.
- One User can create multiple QueueEntries.
- One User can make multiple Bookings.

<u>**ENTITY 2:  DEALER**</u>

Need: Stores additional information about users registered as vehicle dealers.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| dealerId | UUID | PK | Unique dealer identifier |
| userId | UUID | FK, UNIQUE | References User |
| dealerName | VARCHAR |  | Dealership name |
| city | VARCHAR |  | Dealership city |
| address | TEXT |  | Physical address |
| verified | BOOLEAN |  | Admin verification |
| createdAt | TIMESTAMP |  | Registration date |
| updatedAt | TIMESTAMP |  | Last update |

Relationships:

- Each Dealer belongs to exactly one User.
- One Dealer manages multiple Inventory records.
- One Dealer can create multiple Offers.
- One Dealer can receive multiple Bookings.
- One Dealer can be the preferred dealer for multiple QueueEntries.

<u>**ENTITY 3:  CAR_MODEL**</u>

Need: Stores the master catalogue of available car models.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| carModelId | UUID | PK | Unique dealer identifier |
| brand | VARCHAR |  | Manufacturer |
| model | VARCHAR |  | Car model name |
| imageUrl | VARCHAR |  | Car image URL |
| createdAt | TIMESTAMP |  | Record creation date |

Relationships:

- One CarModel can appear in multiple Inventory records.
- One CarModel can have multiple Queues across cities and variants.

<u>**ENTITY 4:  INVENTORY**</u>

Need: Stores the vehicles, availability and pricing maintained by individual dealerships.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| inventoryId | UUID | PK | Unique inventory record |
| dealerId | UUID | FK | References Dealer |
| carModelId | UUID | FK | References CarModel |
| variant | VARCHAR |  | Vehicle variant |
| color | VARCHAR |  | Vehicle colour |
| totalUnits | INTEGER |  | Total stock |
| availableUnits | INTEGER |  | Currently available stock |
| exShowroom | DECIMAL |  | Ex-showroom price |
| insurance			 | DECIMAL |  | Insurance cost |
| rto			 | DECIMAL |  | Registration cost |
| accessories		 | DECIMAL	 |  | Accessories cost |
| discount | DECIMAL |  | Dealer discount |
| active | BOOLEAN |  | Whether listing is active |
| createdAt | TIMESTAMP |  | Record creation date |
| updatedAt | TIMESTAMP |  | Last update |

Relationships:

- Each Inventory record belongs to one Dealer.
- Each Inventory record belongs to one CarModel.
- One Inventory record can be referenced by multiple Offers.
- One Inventory record can be referenced by multiple Bookings over time.

Important constraint:

`dealerId + carModelId + variant + color` must be unique.

<u>**ENTITY 5:  QUEUE**</u>

Need: Represents the shared waiting list for a specific car model, variant and city.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| queueId | UUID | PK | Unique queue identifier |
| carModelId | UUID | FK | References CarModel |
| variant | VARCHAR |  | Requested vehicle variant |
| city | VARCHAR |  | Queue location |
| status | ENUM |  | OPEN/CLOSED |
| createdAt | TIMESTAMP |  | Queue creation date |

Relationships:

- Each Queue belongs to one CarModel.
- One Queue contains multiple QueueEntries.

Important constraint:

`carModelId + variant + city` must be unique.

<u>**ENTITY 6:  QUEUE_ENTRY**</u>

Need: Represents an individual customer's registration in a shared queue.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| entryId | UUID | PK | Unique queue entry identifier |
| queueId | UUID | FK | References Queue |
| customerId | UUID | FK | References User |
| preferredDealerId | UUID | FK,Nullable | Customer’s preferred dealer |
| colourPreference | VARCHAR | Nullable | Preferred vehicle colour |
| maxBudget | DECIMAL | Nullable | Customer’s maximum budget |
| status | ENUM |  | WAITING/OFFERED/BOOKED/LEFT |
| joinedAt | TIMESTAMP |  | Queue joining time |
| updatedAt | TIMESTAMP |  | Last status update |

Relationships:

- Each QueueEntry belongs to one Queue.
- Each QueueEntry belongs to one Customer.
- A QueueEntry can optionally reference one preferred Dealer.
- One QueueEntry can receive multiple Offers.
- One QueueEntry can produce at most one Booking.

Important constraint:

`queueId + customerId` must be unique.

<u>**ENTITY 7:  OFFER**</u>

Need: Represents a price quotation submitted by a dealer to a customer in a queue.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| offerId | UUID | PK | Unique offer identifier |
| queueEntryId | UUID | FK | References QueueEntry |
| inventoryId | UUID | FK | References Inventory |
| dealerId | UUID | FK | References Dealer |
| exShowroom | DECIMAL |  | Quoted ex-showroom price |
| insurance | DECIMAL |  | Quoted insurance |
| rto | DECIMAL |  | Quoted RTO |
| accessories | DECIMAL |  | Offered discount |
| discount | DECIMAL |  | Offered discount |
| totalPrice | DECIMAL |  | Final calculated price |
| status | ENUM |  | PENDING/ACCEPTED/REJECTED/EXPIRED/CANCELLED |
| expiresAt | TIMESTAMP  |  | Offer expiration |
| createdAt | TIMESTAMP |  | Offer creation date |
| updatedAt | TIMESTAMP |  | Last update |

Relationships:

- Each Offer belongs to one QueueEntry.
- Each Offer belongs to one Dealer.
- Each Offer references one Inventory record.
- One Offer can generate zero or one Booking.

Important constraint:

`queueEntryId + dealerId` must be unique for our MVP.

<u>**ENTITY 8:  BOOKING**</u>

Need: Represents a confirmed reservation made by a customer with a particular dealer.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| bookingId | UUID | PK | Unique booking identifier |
| offerId | UUID | FK,UNIQUE | References accepted Offer |
| entryId | UUID | FK,UNIQUE | References QueueEntry |
| customerId | UUID | FK | References User |
| dealerId | UUID | FK | References Dealer |
| inventoryId | UUID | FK | References Inventory |
| priceSnapshot | JSON |  | Immutable price breakdown |
| totalPrice | DECIMAL |  | Final agreed price |
| status | ENUM |  | CONFIRMED/CANCELLED |
| createdAt | TIMESTAMP |  | Booking creation date |
| updatedAt | TIMESTAMP |  | Last update |

Relationships:

- Each Booking belongs to one Offer.
- Each Booking belongs to one QueueEntry.
- Each Booking belongs to one Customer.
- Each Booking belongs to one Dealer.
- Each Booking references one Inventory record.

Important constraints:

- `offerId` is UNIQUE.
- `entryId` is UNIQUE.

<u>**ENTITY 9: AUDIT_LOG**</u>

Need: Maintains a record of important actions performed by users,dealers and admins.

| **Attribute** | **Data type** | **Key** | **Description** |
| --- | --- | --- | --- |
| logId | UUID | PK | Unique audit record |
| actorId | UUID | FK,Nullable | User who performed the action |
| action | VARCHAR |  | Action performed |
| entity | VARCHAR |  | Affected entity type |
| entityId | VARCHAR |  | Identifier of affected record |
| metadata | JSON | Nullable | Additional action details |
| createdAt | TIMESTAMP |  | Action timestamp |

Relationships:

- An AuditLog can optionally belong to one User.
- One User can generate multiple AuditLogs.