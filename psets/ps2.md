# Problem Set 2: Composing Concepts

## Concept Questions

### Questions
1. Contexts. The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

Contexts create different spaces so that the uniqueness of nonces are enforced per context rather than globally. This is so that identical nonces can exist in different contexts. In the URL Shortening App, the contexts are the short URL bases and each context has its unique set of suffixes.

2. Storing used strings. Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

NonceGeneration stores sets of used string in order to ensure uniqueness among the nonces generated. Each new nonce generated should not have been already generated before in that context.

In the counter-based implementation we keep a  counter per context and define function that maps integers to nonces, ensuring that each integer correspond to a unique nonce. The counter is just another way of ensuring unique nonces" it indirectly records which strings are already taken, because every number up to the counter value corresponds to a string that has already been used.


3. Words as nonces. One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?

One advantage of this scheme is that it is easy for users to remember these words and thus easy to type the URL. One disavantage of this scheme is that the word may be misleading or confusing for the user because the word could be unrelated to the link. Moreover, there are limited words in the dictionary.

I would modify the NonceGeneration concept to realize this idea by adding a set of available words for each context. When a nonce is generated, it has to be from this set of available words. Once a nonce is chosen, the word will be removed from the set in this context.



## Synchronization Questions

### Questions
1. Partial matching. In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?

The generate sync only needs the shortURLBase because it only needs to generate a unique nonce that has not been used before with the shortURLBase. The targetUrl is irrelevant for this purpose. The register sync needs both the shortUrlBase and the targetUrl because it creates the mapping between the shortUrl and targetUrl.

2. Omitting names. The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?

The omission convention is not used in every case because in some cases, the names do not align because of different contexts. Different names make the interaction between concepts clearer. For example, in the generate sync the Request.shortenUrl action provides the argument shortUrlBase, and the NonceGeneration.generate action expects a context.

3. Inclusion of request. Why is the request action included in the first two syncs but not the third one?

The first sync needs the shortUrlBase variable and the second sync needs the shortUrlBase and tarUrl variables from the Request action. On the other hand, the third sync only needs the shortUrl from the UrlShortening.register action so the request action is not included.

4. Fixed domain. Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?

In the generate and register syncs, I would remove the shortUrlBase variables and pass in "bit.ly" as the context when performing the NonceGernation.generate action and "bit.ly" as the shortUrlBase in the UrlShortening.register action:

    sync generate
    when Request.shortenUrl
    then NonceGeneration.generate (context: "bit.ly")

    sync register
    when Request.shortenUrl (targetUrl)
    NonceGeneration.generate (): (nonce)
    then UrlShortening.register (shortUrlSuffix: nonce, shortUrlBase: "bit.ly", targetUrl)

5. Adding a sync. These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.

Sync Implementation:

    sync expire
    when ExpiringResource.expireResource ():(resource)
    then UrlShortening.delete (shortUrl: resource)


## Extending the design

### Additional Concepts

**Analytics**

    concept Analytics [Shortening]
    purpose track the number of times a short URL is accessed
    principle When a shortUrl is created, starts tracking the number of visits. When a shortUrl is accessed, increment the visitCount by 1. Return the visitCount when requested.

    state
        a set of VisitAnalytics with
            shortUrl String
            visitCount Number

    actions
        initialize (shortUrl: String)
            requires no analytics exists for shortUrl
            effect creates a new ShorteningAnalytics entry with count = 0

        addVisit (shortUrl: String)
            requires analytics exists for shortUrl
            effect increases the visitCount by 1

        getCount (shortUrl: String): (count: Number)
            requires analytics exists for shortUrl
            effect returns the current access count

**UserAccess**

    concept UserAccess [User, Shortening]
    purpose only allow creator of a short URL to view its analytics
    principle When a shortUrl is created, a UserShortening is created that associates a shortUrl with a user. When a user tries to authorize themselves, this checks if the shortUrl is associated with the user.

    state
    a set of UserShortenings with
        user User
        a set of shortUrl strings

    actions
    makeOwner (user: User, shortUrl: String) : (userShortening : UserShortening)
        requires shortUrl is not associated with another user
        effect if a UserShortening with the user already exists, add shortUrl to the set of shortUrl strings. Otherwise, create a UserShortening with the user and the shortUrl.

    authorize (user: User, shortUrl: String): (authorized: Boolean)
        requires user and shortUrl exists
        effect authorizes the user to if the shortUrl is in the set of shortUrl string associated with the user.

### Syncs

**When shortenings are created**

    sync createAnalytics
    when
        Request.shortenUrl (user)
        UrlShortening.register (): (shortUrl)
    then
        Analytics.initialize (shortUrl)
        UserAccess.makeOwner (user, shortUrl)


**When shortenings are translated to targets**

    sync recordVisit
    when UrlShortening.lookup(shortUrl)
    then Analytics.addVisit(shortUrl)


**When user examines analytics**

    sync viewAnalytics
    when
        Request.viewAnalytics (user, shortUrl)
    then
        Analytics.getCount (user, shortUrl)



### Feature Requests

- Allowing users to choose their own short URLs

Add a sync that uses the shortUrlSuffix provided in Request.shortenUrl rather than generating a random nonce.

- Using the “word as nonce” strategy

Add a set of available words for each context to the NonceGeneration concept. When the generate action is called, randomly select a word from the set of available words and remove the word from the set of the context when it is used.

- Including the target URL in analytics

Add a state TargetVisitAnalytics that stores the targetUrl, a set of shortUrl strings, and a visitCount Number. When a shortening is created, if the targetUrl is new, create a new TargetVisitAnalytics and add the shortUrl to the set of shortUrl strings. Otherwise, just add the shortUrl to the set of shortUrl strings in the associated TargetVisitAnalytic. Now when any of the shortUrls in the set of shortUrls is visited, the visit count for that TargetVisitAnalytic is icremented.

Note that this feature can only track visits from shortUrls associated with this targetUrl and not visits to the targetUrl directly.


- Generate short URLs that are not easily guessed

We can modify NonceGeneration to generate cryptographically random strings. Everything else still functions the same way.

- Reporting analytics to creators who have not registered as users

This is undesirable and should not be included because this is a privacy and security concern. Not being registered as users makes it difficult to securely authenticate a user. This could compromise a lot of private information stored in the analytics if we loosen the security checks by showing analytics to unregistered users.
