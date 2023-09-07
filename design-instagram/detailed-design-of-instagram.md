# Detailed Design of Instagram

### Add more components <a href="#add-more-components-0" id="add-more-components-0"></a>

Let’s add a few more components to our design:

* **Load balancer**: To balance the load of the requests from the end users.
* **Application servers**: To host our service to the end users.
* **Relational database**: To store our data.
* **Blob storage**: To store the photos and videos uploaded by the users.

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.45.42 AM.png" alt=""><figcaption></figcaption></figure>

### Upload, view, and search a photo <a href="#upload-view-and-search-a-photo-0" id="upload-view-and-search-a-photo-0"></a>

The client requests to upload the photo, load balancer passes the request to any of the application servers, which adds an entry to the database. An update that the photo is stored successfully is sent to the user. If an error is encountered, the user is communicated about it as well.

The photo viewing process is also similar to the above-mentioned flow. The client requests to view a photo, and an appropriate photo that matches the request is fetched from the database and shown to the user. The client can also provide a keyword to search for a specific image.

The read requests are more than write requests and it takes time to upload the content in the system. It is efficient if we separate the write (uploads) and read services. The multiple services operated by many servers handle the relevant requests. The read service performs the tasks of fetching the required content for the user, while the write service helps upload content to the system.

We also need to cache the data to handle millions of reads. It improves the user experience by making the fetching process fast. We’ll also opt for lazy loading, which minimizes the client’s waiting time. It allows us to load the content when the user scrolls and therefore save the bandwidth and focus on loading the content the user is currently viewing. It improves the latency to view or search a particular photo or video on Instagram.

The updated design is as follows:

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.45.56 AM.png" alt=""><figcaption></figcaption></figure>

### Generate a timeline <a href="#generate-a-timeline-0" id="generate-a-timeline-0"></a>

Now our task is to generate a user-specific timeline. Let’s explore various approaches and the advantages and disadvantages to opt for the appropriate strategy.

#### The pull approach <a href="#the-pull-approach-1" id="the-pull-approach-1"></a>

When a user opens their Instagram, we send a request for timeline generation. First, we fetch the list of people the user follows, get the photos they recently posted, store them in queues, and display them to the user. But this approach is slow to respond as we generate a timeline every time the user opens Instagram.

We can substantially reduce user-perceived latency by generating the timeline offline. For example, we define a service that fetches the relevant data for the user before, and as the person opens Instagram, it displays the timeline. This decreases the latency rate to show the timeline. Let’s take a look at the slides below to understand the problem and its solution.

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.46.56 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.47.55 AM.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.47.21 AM (1).png" alt=""><figcaption></figcaption></figure>

**Question**

What are the shortcomings of the pull approach?

Instagram is a read-heavy system. Many people don’t post any photos and instead just view others’ posts. So, the calls we make to fetch the recent posts from every follower will usually return nothing. Hence, we should keep in mind that it is not a write-heavy system and should develop a possible solution to cater to it.

\-----------------

#### The push approach <a href="#the-push-approach-0" id="the-push-approach-0"></a>

In a **push approach**, every user is responsible for pushing the content they posted to the people’s timelines who are following them. In the previous approach, we pulled the post from each follower, but in the current approach, we push the post to each follower.

Now we only need to fetch the data that is pushed towards that particular user to generate the timeline. The push approach has stopped a lot of requests that return empty results when followed users have no post in a specified time.

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.48.46 AM.png" alt=""><figcaption></figcaption></figure>

**Question**

What are the shortcomings of the push approach?

Consider an account that belongs to a celebrity, like Cristiano Ronaldo, who has over 400 million followers. So if he posts a photo or a video, we will push the links of the photo/video to 400 million+ users, which is inefficient.

\-------------

#### Hybrid approach <a href="#hybrid-approach-0" id="hybrid-approach-0"></a>

Let’s split our users into two categories:

* **Push-based users**: The users who have a followers count of hundreds or thousands.
* **Pull-based users**: The users who are celebrities and have followers count of a hundred thousand or millions.

The timeline service pulls the data from pull-based followers and adds it to the user’s timeline. The push-based users push their posts to the timeline service of their followers so the timeline service can add to the user’s timeline.

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.49.15 AM.png" alt=""><figcaption></figcaption></figure>

We have used the method which generates the timeline, but where do we store the timeline? We store a user’s timeline against a `userID` in a key-value store. Upon request, we fetch the data from the key-value store and show it to the user. The key is `userID`, while the value is timeline content (links to photos and videos). Because the storage size of the value is often limited to a few MegaBytes, we can store the timeline data in a blob and put the link to the blob in the value of the key as we approach the size limit.

We can add a new feature called story to our Instagram. In the story feature, the users can add a photo that stays available for others to view for 24 hours only. We can do this by maintaining an option in the table where we can store a story’s duration. We can set it to 24 hours, and the [task scheduler](../distributed-task-scheduler/system-design-the-distributed-task-scheduler.md) deletes the entries whose time exceeds the 24 hours limit.

### Finalized design <a href="#finalized-design-0" id="finalized-design-0"></a>

We’ll also use **CDN (content delivery network)** in our design. We can keep images and videos of celebrities in CDN which make it easier for the followers to fetch them. The load balancer first routes the read request to the nearest CDN, if the requested content is not available there, then it forwards the request to the particular read application server (see the “[load balancing chapter](../load-balancers/introduction-to-load-balancers.md)” for the details). The CDN helps our system to be available to millions of concurrent users and minimizes latency.

The final design is given below:

<figure><img src="../.gitbook/assets/Screenshot 2023-09-06 at 12.49.56 AM.png" alt=""><figcaption></figcaption></figure>

Point to Ponder

**Question**

How can we count millions of interactions (like or view) on a celebrity post?

We can use [sharded counters](../sharded-counters/high-level-design-of-sharded-counters.md) to count the number of multiple interactions for a particular user. Each counter has a number of shards distributed across various edge servers to reduce the load on the application server and latency. Users nearest to the edge server get the updated count frequently on a specific post compared to those in distant regions.

\--------------

### Ensure non-functional requirements <a href="#ensure-non-functional-requirements-0" id="ensure-non-functional-requirements-0"></a>

We evaluate the Instagram design with respect to its non-functional requirements:

* **Scalability**: We can add more servers to application service layers to make the scalability better and handle numerous requests from the clients. We can also increase the number of databases to store the growing users’ data.
* **Latency**: The use of cache and CDNs have reduced the content fetching time.
* **Availability**: We have made the system available to the users by using the storage and databases that are replicated across the globe.
* **Durability:** We have persistent storage that maintains the backup of the data so any uploaded content (photos and videos) never gets lost.
* **Consistency**: We have used storage like blob stores and databases to keep our data consistent globally.
* **Reliability**: Our databases handle [replication](../databases/data-replication/) and redundancy, so our system stays reliable and data is not lost. The load balancing layer routes requests around failed servers.

### Conclusion <a href="#conclusion-1" id="conclusion-1"></a>

This design problem highlights that we can provide major services by connecting our building blocks appropriately. The scalable and fault-tolerant building blocks enable us to concentrate on use-case-specific issues (such as the efficient formation of timelines).
