# Evaluation of a Newsfeed System’s Design

### Fulfill requirements <a href="#fulfill-requirements-0" id="fulfill-requirements-0"></a>

Our non-functional requirements for the proposed newsfeed system design are scalability, fault tolerance, availability, and low latency. Let’s discuss how the proposed system fulfills these requirements:

1. **Scalability:** The proposed system is scalable to handle an ever-increasing number of users. The required resources, including load balancers, web servers, and other relevant servers, are added/removed on demand.
2. **Fault tolerance:** The replication of data consisting of users’ metadata, posts, and newsfeed makes the system fault-tolerant. Moreover, the redundant resources are always there to handle the failure of a server or its component.
3. **Availability:** The system is highly available by providing redundant servers and replicating data on them. When a user gets disconnected due to some fault in the server, the session is re-created via a load balancer with a different server. Moreover, the data (users metadata, posts, and newsfeeds) is stored on different and redundant database clusters, which provides high availability and durability.
4. **Low latency:** We can minimize the system’s latency at various levels by:
   * Geographically distributed servers and the cache associated with them. This way, we bring the service close to users.
   * Using CDNs for frequently accessed newsfeeds and media content.

### Quiz on the newsfeed system’s design <a href="#quiz-on-the-newsfeed-systems-design-0" id="quiz-on-the-newsfeed-systems-design-0"></a>

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.28.07 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.27.35 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.28.40 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.29.30 AM.png" alt=""><figcaption></figcaption></figure>

### Summary <a href="#summary-0" id="summary-0"></a>

In this chapter, we learned to design a newsfeed system at scale. Our design ranked enormous user data to show carefully curated content to the user for better user experience and engagement. Our newsfeed design is general enough that it can be used in many places such as Twitter feeds, Facebook posts, YouTube and Instagram recommendations, News applications, and so on.
