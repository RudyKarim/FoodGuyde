\# FoodGuyde — MongoDB Data Model



\## 1. Database Overview



\*\*Project:\*\* Restaurant and Menu Search with Delivery Orders



\*\*Application Reference:\*\* DoorDash



\*\*Database Name:\*\* `food\_delivery`



MongoDB is the primary NoSQL document database of FoodGuyde.



It stores the operational data required to manage restaurants, menus, customers, orders, and deliveries.



The architecture uses three NoSQL databases:



\- \*\*MongoDB:\*\* Main operational database.

\- \*\*Redis:\*\* Caching frequently accessed information.

\- \*\*Apache Cassandra:\*\* Storing historical order and delivery events.



Apache Kafka is used for event streaming.



MongoDB Search and Geospatial Indexes provide restaurant, menu, and location-based searching.



\## 2. MongoDB Collections



Our database contains eight main collections.



| Collection | Purpose |

|---|---|

| `restaurants` | Restaurant profiles, cuisine, ratings, locations, and opening status |

| `menus` | Menus belonging to restaurants |

| `menu\_items` | Food items, descriptions, prices, availability, and modifiers |

| `service\_zones` | Geographic areas where restaurants deliver |

| `customers` | Customer profiles and delivery addresses |

| `baskets` | Items selected by customers before checkout |

| `orders` | Customer orders, purchased items, totals, and current status |

| `couriers` | Delivery drivers, availability, and current locations |



\## 3. Example MongoDB Documents



\### 3.1 Restaurants Collection



```json

{

&#x20; "\_id": "rest\_001",

&#x20; "name": "Burger House",

&#x20; "cuisine": "American",

&#x20; "rating": 4.7,

&#x20; "isOpen": true,

&#x20; "location": {

&#x20;   "type": "Point",

&#x20;   "coordinates": \[35.5018, 33.8938]

&#x20; },

&#x20; "serviceZoneIds": \["zone\_001"]

}

```



\### 3.2 Menu Items Collection



```json

{

&#x20; "\_id": "item\_001",

&#x20; "restaurantId": "rest\_001",

&#x20; "menuId": "menu\_001",

&#x20; "name": "Chicken Burger",

&#x20; "description": "Grilled chicken burger with cheese",

&#x20; "price": 8.50,

&#x20; "currency": "USD",

&#x20; "available": true,

&#x20; "dietaryTags": \[],

&#x20; "modifiers": \[

&#x20;   {

&#x20;     "name": "Extra Cheese",

&#x20;     "additionalPrice": 1.50

&#x20;   }

&#x20; ]

}

```



\### 3.3 Orders Collection



```json

{

&#x20; "\_id": "order\_001",

&#x20; "customerId": "cust\_001",

&#x20; "restaurantId": "rest\_001",

&#x20; "items": \[

&#x20;   {

&#x20;     "itemId": "item\_001",

&#x20;     "name": "Chicken Burger",

&#x20;     "quantity": 2,

&#x20;     "unitPrice": 8.50

&#x20;   }

&#x20; ],

&#x20; "totalAmount": 17.00,

&#x20; "currency": "USD",

&#x20; "status": "pending",

&#x20; "courierId": null

}

```



\## 4. Relationships Between Collections



\- One restaurant can have multiple menus.

\- One menu can contain multiple menu items.

\- One menu item can contain multiple embedded modifiers.

\- One restaurant can serve multiple delivery zones.

\- One delivery zone can contain multiple restaurants.

\- One customer can place multiple orders.

\- One customer can have an active basket.

\- One order belongs to one restaurant.

\- One order can be assigned to one courier.

\- One courier can deliver multiple orders over time.



\## 5. MongoDB Relationship Diagram



```mermaid

flowchart TD

&#x20;   R\["restaurants"] -->|1 to many| M\["menus"]

&#x20;   M -->|1 to many| I\["menu\_items"]

&#x20;   I -->|embedded| MOD\["modifiers"]

&#x20;   R <-->|many to many| Z\["service\_zones"]

&#x20;   C\["customers"] -->|1 to many| O\["orders"]

&#x20;   C -->|active basket| B\["baskets"]

&#x20;   R -->|1 to many| O

&#x20;   CO\["couriers"] -->|1 to many over time| O

```



\## 6. Database Design Decisions



\### MongoDB as the Primary Database



MongoDB is selected because it supports flexible JSON-like documents, nested data, indexing, and geographic information.



\### Embedded Documents



Small related information, such as menu item modifiers, can be embedded inside their parent documents.



Order items can also be embedded as purchased-item snapshots to preserve their names and prices at checkout.



\### References



References are used between collections that contain independently updated entities.



For example, each menu item references its restaurant using `restaurantId`.



\### Order Consistency



MongoDB is the authoritative source for the current status of orders and menu availability.



Before confirming an order, the system must recheck restaurant status and item availability using appropriate concurrency controls.



\### Other Database Responsibilities



Redis caches frequently requested restaurant and menu information.



Cassandra stores historical order and delivery events.



Kafka distributes events between services.



\## 7. Next Design Steps



The following will be developed in subsequent sections:



1\. Complete fields and data types for every collection.

2\. Detailed embedded-document versus reference decisions.

3\. MongoDB Search and example search queries.

4\. Geospatial indexing and delivery-zone matching.

5\. Redis caching and cache invalidation.

6\. Sharding and database scalability.

7\. Index maintenance costs and database trade-offs.



