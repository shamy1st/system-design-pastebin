# Pastebin Design

Let's design a Pastebin like web service, where users can store plain text. Users of the service will enter a piece of text and get a randomly generated URL to access it.  

Similar Services: pastebin.com, pasted.co, chopapp.com

Difficulty Level: Easy

## What is Pastebin?

* Pastebin like services enable users to store plain text or images over the network (typically the Internet) and generate unique URLs to access the uploaded data.
* Such services are also used to share data over the network quickly, as users would just need to pass the URL to let other users see it.

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

* assume here that we get 1M new pastes added to our system every day.
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

* average paste contains 10KB
* Incoming data: 12 * 10KB = 120 KB/sec
* Outgoing data: 58 * 10KB =~ 0.6 MB/sec 

### Memory Estimates

* We can cache some of the hot pastes that are frequently accessed.
* Following the 80-20 rule, meaning 20% of hot pastes generate 80% of traffic, we would like to cache these 20% pastes
* Since we have 5M read requests per day, to cache 20% of these requests, we would need: 0.2 * 5M * 10KB ~= 10 GB

### High-level Estimates

Metric               | Estimate
---------------------|---------
New Pastes           | 12 pastes/sec
Paste Reads          | 58 reads/sec
Incoming data        | 120 KB/s
Outgoing data        | 0.6 MB/sec
Storage for 10 years | 53 TB
Memory for cache     | 10 GB


## 4. High-level Design

* At a high level, we need an application layer that will serve all the read and write requests.
* Application layer will talk to a storage layer to store and retrieve data.

![](https://github.com/shamy1st/system-design-pastebin/blob/main/hld.png)

## 5. System APIs

* We can have SOAP or REST APIs to expose the functionality of our service.

1. **addPaste**(api_dev_key, paste_data, custom_url=None user_name=None, paste_name=None, expire_date=None)

   **Parameters**:
   * **api_dev_key** (string): The API developer key of a registered account. This will be used to, among other things, throttle users based on their allocated quota.
   * **paste_data** (string): Textual data of the paste.
   * **custom_url** (string): Optional custom URL.
   * **user_name** (string): Optional user name to be used to generate URL.
   * **paste_name** (string): Optional name of the paste.
   * **expire_date** (string): Optional expiration date for the paste.
   
   **Return**: (string)
   * A successful insertion returns the URL through which the paste can be accessed, otherwise, it will return an error code.

2. **getPaste**(api_dev_key, api_paste_key)
   * Where “api_paste_key” is a string representing the Paste Key of the paste to be retrieved.
   * This API will return the textual data of the paste.

3. **deletePaste**(api_dev_key, api_paste_key)
   * A successful deletion returns ‘true’, otherwise returns ‘false’.

## 6. Database Model

A few observations about the nature of the data we are storing:

1. We need to store billions of records.
2. Each metadata object we are storing would be small (less than 1KB).
3. Each paste object we are storing can be of medium size (it can be a few MB).
4. There are no relationships between records, except if we want to store which user created what Paste.
5. Our service is read-heavy.

### Schema 

![](https://github.com/shamy1st/system-design-pastebin/blob/main/database-model.png)

* Here, ‘URlHash’ is the URL equivalent of the TinyURL and ‘ContentKey’ is a reference to an external object storing the contents of the paste

### Which kind of database should we use?

* We can segregate our storage layer with one database storing metadata related to each paste, users, etc.
* while the other storing the paste contents in some object storage (like Amazon S3).
* This division of data will also allow us to scale them individually.

## 7. Low-level Design

![](https://github.com/shamy1st/system-design-pastebin/blob/main/lld.png)

### Solution 01

### Solution 02

## 8. Bottlenecks

### Data Partitioning and Replication

### Cache

### Load Balancer (LB)

### Purging or DB cleanup

## 9. Security and Permissions

