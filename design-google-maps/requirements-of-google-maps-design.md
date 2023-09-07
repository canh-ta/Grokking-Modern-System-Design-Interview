# Requirements of Google Maps' Design

### Requirements <a href="#requirements-0" id="requirements-0"></a>

Before we start requirements, let’s clarify that we will design a system like Google Maps by picking a few key features because actual Google Maps is feature-rich and complex.

Let’s list the functional and non-functional requirements of the system under design.

#### Functional requirements <a href="#functional-requirements-1" id="functional-requirements-1"></a>

The functional requirements of our system are as follows.

* **Identify the current location**: Users should be able to approximate their current location (latitude and longitude in decimal values) on the world map.
* **Recommend the fastest route**: Given the source and destination (place names in text), the system should recommend the optimal route by distance and time, depending on the type of transportation.
* **Give directions**: Once the user has chosen the route, the system should list directions in text format, where each item in the list guides the user to turn or continue in a specific direction to reach the destination.

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.42.29 AM.png" alt=""><figcaption></figcaption></figure>

#### Non-functional requirements <a href="#non-functional-requirements-0" id="non-functional-requirements-0"></a>

The non-functional requirements of our system are as follows.

* **Availability**: The system should be highly available.
* **Scalability**: It should be scalable because both individuals and other enterprise applications like Uber and Lyft use Google Maps to find appropriate routes.
* **Less response time**: It shouldn’t take more than two or three seconds to calculate the ETA and the route, given the source and the destination points.
* **Accuracy**: The ETA we predict should not deviate too much from the actual travel time.

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.42.54 AM.png" alt=""><figcaption></figcaption></figure>

**Note:** We’re not getting into the details of how we get the data on roads and layout. Government agencies provide maps, and in some places, Google itself drives mapping vehicles to find roads and their intersection points. Road networks are modeled with a graph data structure, where intersection points are the **vertices**, and the roads between intersections are the **weighted edges**.

### Challenges <a href="#challenges-0" id="challenges-0"></a>

Some of the challenges that we need to focus on while designing a system like Google Maps are below:

* **Scalability**: Serving millions of queries for different routes in a second, given a graph with billions of nodes and edges spanning over 194 countries, requires robust scalability measures. A simple approach, given the latitude and longitude of the source and destination, would be to apply an algorithm like Dijkstra to find the shortest path between the source and the destination. However, this approach wouldn’t scale well for billions of users sending millions of queries per second. This is because running any path-finding algorithm on a graph with billions of nodes running a million times per second is inefficient in terms of time and cost, ultimately leading to a bad user experience. Therefore, our solution needs to find alternative techniques to scale well.

A graph spanning the whole world network

* **ETA computation**: In an ideal situation with empty roads, it’s straightforward to compute ETA using the distance and the speed of the vehicle we want to ride on. However, we cannot ignore factors like the amount of traffic on the roads and road conditions, which affect the ETA directly. For example, a road under construction, collisions, and rush hours all might slow down traffic. Quantifying the factors above to design our system is not trivial. Therefore, we’ll, categorize the factors above in terms of traffic load to complete our design.

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.43.09 AM.png" alt=""><figcaption></figcaption></figure>

### Resource estimation <a href="#resource-estimation-0" id="resource-estimation-0"></a>

Let’s estimate the total number of servers, storage, and bandwidth required by the system.

#### Number of servers estimation <a href="#number-of-servers-estimation-1" id="number-of-servers-estimation-1"></a>

To estimate the number of servers, we need to know how many daily active users are using Google Maps and how many requests per second a single Google Maps server can handle. We assume the following numbers:

* Daily active users who use Google Maps: 32 million (about 1 billion monthly users).
* Number of requests a single server can handle per second: 8,000.

The number of servers required has been calculated using the below formula:

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.43.42 AM.png" alt=""><figcaption></figcaption></figure>

#### Storage estimation <a href="#storage-estimation-0" id="storage-estimation-0"></a>

Google Maps is essentially a system with a one-time storage requirement. The road data from many countries has already been added, which is over 20 petabytes as of 2022. Since there are minimal changes on the road networks, the daily storage requirement is going to be negligible for Google Maps. Also, short-term changes in the road network is a small amount of data as compared to the full network data. Therefore, our storage needs don’t change rapidly.

#### Bandwidth estimation <a href="#bandwidth-estimation-0" id="bandwidth-estimation-0"></a>

As a standard practice, we have to estimate the bandwidth required for the incoming and outgoing traffic of our system. Most of the bandwidth requirements for Google Maps are due to requests sent by users. Therefore, we’ve devised the following formula to calculate bandwidth:

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.44.16 AM.png" alt=""><figcaption></figcaption></figure>

**Incoming traffic**

To estimate the incoming query traffic bandwidth, we assume the following numbers:

* Maximum number of requests by a single user per day: 50.
* Request size (source and destination): 200 Bytes.

Using the assumptions above, we can estimate the total number of requests per second on Google Maps using the following formula:

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.44.45 AM.png" alt=""><figcaption></figcaption></figure>

We can calculate the incoming query traffic bandwidth required by Google Maps by inserting the request per second calculated above and the size of each request in the aforementioned bandwidth formula.

**Bandwidth Required for Incoming Query Traffic**

| No. of requests per second | Request size (Bytes) | Bandwidth (Mb/s) |
| -------------------------- | -------------------- | ---------------- |
| 18518                      | 200                  | f29.63           |

**Outgoing traffic**

Outgoing application traffic will include the response that the server generates for the user when they make a navigation request. The response consists of visuals and textual content, and typically includes the route shown on the map, estimated time, distance, and more detail about each step in the route. We assume the following numbers to estimate the outgoing traffic bandwidth:

* Total requests per second (calculated above): 18,518.
* Response size: 2 MB + 5 KB = 2005 KB.
  * Size of visual data on average: 2 MB.
  * Size of textual data on average: 5 KB.

We can calculate the bandwidth required for outgoing traffic using the same formula.

**Bandwidth Required for the Outgoing Application Traffic**

| No. of requests per second | Response size (KB) | Bandwidth (Gb/s) |
| -------------------------- | ------------------ | ---------------- |
| 18518                      | 2005               | f297.03          |

Show Detailed CalculationsSummarizing the bandwidth requirements for Google Maps

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.45.36 AM (1).png" alt=""><figcaption></figcaption></figure>

### Building blocks we will use <a href="#building-blocks-we-will-use-0" id="building-blocks-we-will-use-0"></a>

Now that we’ve completed our estimates of resources required, let’s identify the building blocks that will be an integral part of our design for the Google Maps system. Below, we have the key building blocks:

<figure><img src="../.gitbook/assets/Screenshot 2023-09-03 at 11.46.46 AM.png" alt=""><figcaption></figcaption></figure>

* [**Load balancers**](../load-balancers/introduction-to-load-balancers.md) are necessary to distribute user requests among different servers and services.
* [**Databases**](../databases/introduction-to-databases.md) are required to store data in the form of a graph along with metadata information.
* [**Distributed search**](../distributed-search/system-design-the-distributed-search.md) is needed to search different places on the map.
* [**A pub-sub system**](../pub-sub/system-design-the-pub-sub-abstraction.md) is required for generating and responding to important events during navigation and notifying the corresponding services.
* [**A key-value store**](../key-value-store/system-design-the-key-value-store.md) is also used to store some metadata information.

Besides the building blocks mentioned above, other components will also be required for designing our maps system. These components will be discussed in the design lessons. We are now ready to explore the system and API design of Google Maps.
