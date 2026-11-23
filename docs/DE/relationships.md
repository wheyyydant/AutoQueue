Relations

| Entity A | Entity B | Relationship | Explanation |
| --- | --- | --- | --- |
| User | Dealer | 1 : 0..1 | A user may have one dealer profile. |
| User | QueueEntry | 1 : N | A customer can join multiple queues. |
| User | Booking | 1 : N | A customer can make multiple bookings. |
| User | AuditLog | 1 : N | A user can generate multiple audit records. |
| Dealer | Inventory | 1 : N | A dealer can stock multiple configurations. |
| Dealer | Offer | 1 : N | A dealer can create multiple offers. |
| Dealer | Booking | 1 : N | A dealer can receive multiple reservations. |
| Dealer | QueueEntry | 1 : N | A dealer can be preferred by multiple customers. |
| CarModel | Inventory | 1 : N | A model can exist in multiple inventories. |
| CarModel | Queue | 1 : N | A model can have multiple city/variant queues. |
| Queue | QueueEntry | 1 : N | A queue contains multiple customers. |
| QueueEntry | Offer | 1 : N | A customer can receive multiple offers. |
| QueueEntry | Booking | 1 : 0..1 | An entry can generate at most one booking. |
| Inventory | Offer | 1 : N | Inventory can be quoted in multiple offers. |
| Inventory | Booking | 1 : N | Inventory can be reserved repeatedly over time. |
| Offer | Booking | 1 : 0..1 | An offer can generate at most one booking. |