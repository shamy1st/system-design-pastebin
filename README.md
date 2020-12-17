# Pastebin Design

Let's design a Pastebin like web service, where users can store plain text. Users of the service will enter a piece of text and get a randomly generated URL to access it.  

Similar Services: pastebin.com, pasted.co, chopapp.com

Difficulty Level: Easy

## What is Pastebin?

* Pastebin like services enable users to store plain text or images over the network (typically the Internet) and generate unique URLs to access the uploaded data.
* Such services are also used to share data over the network quickly, as users would just need to pass the URL to let other users see it.
* 

## 1. Requirements

### Functional Requirements

1. Users should be able to upload or “paste” their data and get a unique URL to access it.
2. Users will only be able to upload text.
3. Data and links will expire after a specific timespan automatically; users should also be able to specify expiration time.
4. Users should optionally be able to pick a custom alias for their paste.

### Non-functional Requirements

1. The system should be highly reliable, any data uploaded should not be lost.
2. The system should be highly available. This is required because if our service is down, users will not be able to access their Pastes.
3. Users should be able to access their Pastes in real-time with minimum latency.
4. Paste links should not be guessable (not predictable).

### Out of Scope Requirements

1. Analytics, e.g., how many times a paste was accessed?
2. Our service should also be accessible through REST APIs by other services.

## 2. Design Considerations

* What should be the limit on the amount of text user can paste at a time?
    * We can limit users not to have Pastes bigger than 10MB to stop the abuse of the service.
* Should we impose size limits on custom URLs?
    * Since our service supports custom URLs, users can pick any URL that they like, but providing a custom URL is not mandatory.
    * However, it is reasonable (and often desirable) to impose a size limit on custom URLs, so that we have a consistent URL database.

## 3. Estimation

* assume here that we get one million new pastes added to our system every day.
* assume we plan for 10 years.
* assume a 5:1 ratio between read and write.

### Traffic Estimates

* New Pastes per second: 1M / (24 hours * 3600 seconds) ~= 12 pastes/sec
* Paste reads per second: 5M / (24 hours * 3600 seconds) ~= 58 reads/sec

### Storage Estimates

* users can upload maximum 10MB of data.
* assume that each paste on average contains 10KB
* storage per day: 1M * 10KB = 10 GB/day
* storage per 10 years: 10GB * 365 * 10 ~= 37TB
* New Pastes per 10 years: 1M * 365 * 10 = 3.65 billion pastes
* we need to generate and store keys to uniquely identify these pastes.
* assume base64 encoding, need six letters strings: 64^6 ~= 68.7 billion unique strings
* keys storage per 10 years: 3.65billion * 6 = 22GB
* 22GB is negligible compared to 37TB
* assume a 70% capacity model (meaning we don’t want to use more than 70% of our total storage capacity at any point)
* this will raise our storage needs to 53TB

### Bandwidth Estimates



### Memory Estimates



### High-level Estimates

Metric         | Estimate
---------------|---------
New Pastes     | 12 pastes/sec
Paste Reads    | 58 reads/sec
Storage        | 53 TB/10 years


## 4. High-level Design

## 5. System APIs

## 6. Database Model

    ### Schema 

    ### Which kind of database should we use?

## 7. Low-level Design

    ### Solution 01

    ### Solution 02

## 8. Bottlenecks

    ### Data Partitioning and Replication

    ### Cache

    ### Load Balancer (LB)

    ### Purging or DB cleanup

## 9. Security and Permissions

