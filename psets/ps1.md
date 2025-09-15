# Problem 1: Problem Framing

## Exercise 1: Reading a Concept

### Questions
1. **Invariants**:
1: Count of requests should always be nonnegative. 2: Every purchase must correspond to only one existing request. Ensuring that the count of requests is always nonnegative is more important because it prevents users from purchasing more than necessary. The purchase action is most affected by it. A purchase can only be made if request count is at least the count of the purchase.
2. **Fixing an action**: addItem action could potentially break this important invariant. If the count is allowed to be negative, then the number of requests could also be negative. The problem would be fixed by requiring the count to be positive.
3. **Inferring behavior**: A registry can be opened and closed repeatedly. A reason to allow this is because a user might want to temporarily hide the registry from other users. For example, a user cannot receive items temporarily because they are travelling somewhere so they need to close the registry. They can then open the registry when they're back.
4. **Registry deletion**: This might matter depending on the user's preferences. The user may not want to see every registry they ever created. However, just looking at the practical use of the registries, not having a delete action does not affect functionality of the registries.
5. **Queries**: Registry owner would need to know which items were purchased and by whom. Giver would need to know which items still need to be purchased and how many.
6. **Hiding purchases**: We can add a visibile Flag to each Registry to control whether the user can see purchases in that registry.
7. **Generic types**: The Item type is preferable because names, description, prices are not unique and can often change. Item type, like SKU codes, are a reliable identifier of the item.



## Exercise 2: Extending a familiar concept
### Questions
1. Concept State

**concept** PasswordAuthentication [User]

**purpose** limit access to known users

**principle** after a user registers with a username and a password,
    they can authenticate with that same username and password
    and be treated each time as the same user.

**state**

    a set of Users with
        a username String
        a password String


2. Defining Actions

**actions**

    register (username: String, password: String): (user: User)
        requires username to not already exist
        effects creates a new User with given username and password

    authenticate (username: String, password: String): (user: User)
        requires a User to exist with a username and password matching the given username and password
        effects if a User with the given username exists and the given password matches the User's password, then access is granted. Otherwise, access is denied.

3. The invariant is that there are no two users with the same username. It is preserved by not allowing a User to be created if a username already exists.


4. PasswordEmailAuthentication


**concept** PasswordEmailAuthentication [User]

**purpose** limit access to known users

**principle** after a user registers with a username and a password, they will receive an email with a special token to confirm their registration. Then,
    they can authenticate with that same username and password
    and be treated each time as the same user

**state**

    a set of Users with
        a username String
        a password String

    a set of PendingUsers with
        a username String
        a password String
        a secretToken String



**actions**

    register (username: String, password: String; email: String): (user: PendingUser)
        requires username to not already exist
        effects creates a new PendingUser with given username and password and a unique secret token. An email with a secret token is sent to the given email via a sync

    authenticate (username: String, password: String): (user: User)
        requires a User to exist with a username and password matching the given username and password
        effects if username and password matches, then user is authenticated and the associated user is retrieved.

    confirm (user: PendingUser, token: String): (user: User)
        requires token to match the secret token associated with the user
        effects if token matches, completes registration of new user by creating a User with given username and password and deleting the PendingUser.


## Exercise 3: Comparing concepts

**concept** PersonalAccessToken [User]

**purpose** limit access to known users

**principle** when a personal access token is generated, a User can use it to authenticate themselves and grant themselves access within a certain scope. When a personal access token is deleted, the token can no longer be used.

**state**

    a set of Users with
        a username String
        a set of PersonalAccessTokens

    a set of PersonalAccessTokens
        an accessToken String
        an expireDate Time
        a set of Scopes


**actions**

    generateToken (user: User, expireDate: Time, scope: [String]): (token: PersonalAccessTokens)
        requires username to not already exist and expireDate has to be after the current date.
        effects creates a new PersonalAccessToken associated with the given User that expires on the given expireDate and gives access according to the specifications of the given scope.

    deleteToken (token: PersonalAccessToken):
        requires the given token to exist
        effects deletes the given token.

    authenticate (user: User, token: String, scope:String): (user: User)
        requires user exists, given token is associated with the given user, the token is active, and the scope is within the token's list of scopes it can access.
        effects authenticates the user and retrieves the user, granting access.


How PersonalAccessToken differs from PasswordAuthentification:

1. Multiple tokens can be created while with PasswordAuthentification there can only be one password.
2. Tokens can be created and deleted anytime. Passwords are created when the User is created and cannot be deleted or changed.
3. Tokens have expiration dates.
4. Tokens can specify the scope you are able to access.


I would change the Github documentation to reflect these differences more clearly. Instead of saying "an alternative to using passwords", I would highlight how tokens can be generated anytime and multiple can exist at once. Unlike passwords, in PasswordAuthentification, tokens are generally time-limited with an expiration date. Moreover, with tokens, you can specify the scope you are able to access with each token.


## Exercise 4: Defining familiar Concepts
###

URL Shortener


**concept** URLShortener [URL]

**purpose** turn long URLs into shorter URLs to make navigating to these sources more convenient.

**principle** when a shortened URL is created using a long URL and optionally a URL suffix, users can navigate to the website by using the shortened URL with the URL suffix instead of the long URL.

**state**

    a set of ShortenedURLs with
        a longURL String
        a set of URL suffixes

**actions**

    generateRandomShortURL (url: String): (shortURL: ShortenedURL)
        requires given url is a valid url
        effects creats a ShortenedURL with full URL as the given url and a randomly generated suffix as the URL suffix

    createShortenedURL (url: String, suffix: String): (shortURL: ShortenedURL)
        requires given url is a valid url and suffix is unique and does not exist as a URL suffix among the existing ShortenedURLs
        effects creates a ShortenedURL with full URL as the given url. If a valid suffix is given, add the suffix to the set of URL suffixes associated with the ShortenedURL.

    redirect (suffix: String):
        requires: the suffix exists among the one of the sets of a ShortenedURL.
        effects redirects users to the long URL associated with the ShortenedURL.

Notes:
- Similarly to tinyurl.com, all URLs are shortend with a unique suffix. For example, tinyurl.com/suffix.
- One long URL can have multiple suffixes, but one suffix maps to exactly one URL.

### Conference Room Booking

**concept** RoomBooking [User]

**purpose** enable people to reserve conference rooms

**principle** When a room time slot created, users can reserve the room at that time slot. When someone books a conference room for a certain hour, a reservation is created and no one else can reserve the same room at the same hour. When someone no longer needs to use the room they reserved at a certain time, they can retract their reservation and other users can now book the same room at that hour.

**state**

    a set of RoomTimeSlots with
        a roomNumber Number
        a time
        an available Flag

    a set of Reservations with
        a host User
        a RoomTimeSlot


**actions**

    createRoomTimeSlot (roomNumber: Number, time: Time): (room: Room)
        requires a RoomTimeSlot with the given roomNumber and time to not already exist
        effects creates a new RoomTimeSlot with given roomNumber and time and sets available Flag to True.

    reserve (user: User, booking: RoomTimeSlot): (reservation: Reservation)
        requires user to exist and booking to exist and to be available.
        effects creates a Reservation with given user as the host and the booking as the RoomTimeSlot. Sets the RoomTimeSlot's available Flag to unavailable.

    unreserve (reservation: Reservation): (reservation: Reservation)
        requires reservation to exist
        effects deletes the reservation and sets the available Flag of the RoomTimeSlot under the reservation to available.
Notes:
- RoomTimeSlot times are assumed to be by the hour. (eg. 12pm)
- Each reservation is for an hour.



### Billable Hours Tracking

**concept** BillableHoursTracking [Employee, Time]

**purpose**
   track billable hours of employees on projects

**principle**
  an employee starts a work session by selecting a project, giving a description, and setting a default end time. If the employee has no other active sessions, a session is created from this information and the timestamp of when the session starts.
  The employee ends the session, and the current time is set as the end time of the session
  if a session is not closed after the deafult end time, it will close automatically.

**state**

    a set of Projects with
        a set of Sessions
        a projectName String

    a set of Sessions with
        an Employee
        a start Time
        a description String
        an end Time
        an active Flag


**actions**

    startSession(project: Project, employee: Employee, startTime: Time, defaultEndTime: Time, description: String) : (session: Session)
        requires employee to have no other active Sessions and project to exist
        effects create a new session within the given project with the given employee, startTime set equal to the current time, a default end time set by the employee, given description, and active Flag set to True.

    endSession(session: Session, endTime: Time): (session: Session)
        requires active Flag of session to be True
        effects modifies the session's end time to be the current time and sets the active flag to inactive (False).

    automaticallyEnd(session: Session, expiration: Time): (session: Session)
        requires session is active and the current time is past the set end time.
        effects set the session's active flag to inactive (False).

Notes:
- all sessions automatically end at a default time set by the employee when they start the session.
