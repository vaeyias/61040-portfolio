# Assignment 2

## Problem Statement

**Domain: Group Trips**

This past summer, I traveled frequently with large groups and was usually the primary planner and organizer. While the trips were extremely fun, I faced many challenges in coordinating group activities, from tracking everyone's preferences to ensuring everyone stayed up to date with the latest updates. I realized how much time and effort is required to keep everything running smoothly. Although I love group trips, I believe that reducing the stress of managing them would make the experience more enjoyable for everyone.


**Problem: Tracking Expenses**

Managing expenses is a constant source of stress during trips, especially keeping track of who owes whom what and also trying to track personal expenses. On many of our trips, each person maintained their own record of expenses in notes or spreadsheets, which often led to conflicting information and tension within the group. Additionally, with people covering costs for one another, it is easy to lose track of how much you personally spent. A solution to this seems feasible and would be a major help in managing  expenses.


**Tracking Expenses**
- **Group Organizer**: Responsible for keeping group finances clear. Trip face a lot of stress in resolving potential financial disputes.
- **Friends/Members**: People sharing expenses and reimbursing each other. They often experience stress and confusion over tracking many expenses.
- **Payment Platforms**: Payment apps like Venmo and CashApp


### Evidence and Comparables
**Evidence**
1. [Money Disputes when Traveling](https://www.experian.com/blogs/ask-experian/survey-financial-stress-of-traveling-with-friends/): The blog reports over half of Gen Z and millennial travelers have experienced money-related disagreements with friends while traveling, and 1 in 5 have ended a friendship over financial issues during a trip.

2. [Awkwardness of Shared Costs](https://www.whimstay.com/blog/group-travel-hacks-how-to-handle-shared-costs-without-the-awkwardness/): This blog talks about the awkwardness around figuring out who owes what and how it can ruin the vibe of a trip, often leading to resentment among group members.

3. [Stressing about Money on Vacation](https://www.budgettravel.com/article/money-is-biggest-stress-on-vacation-survey-shows_11820): The article reports that about 26% of Americans in a survey admit to squabbling about money during vacations. In this same survey, 64% of women and 41% of men reported feeling guilty about spending money while on vacation, showing how expenses underlie much of the emotional strain during group travel.

4. [Awkwardness of Budgets on Group Trips](https://www.nerdwallet.com/article/travel/how-to-handle-awkward-money-situations-on-a-group-trip): The article talks about how the failure to discuss budgets and cost-sharing openly in advance is cited as a common source of awkwardness, arguments, and resentment during and after group trips.


**Comparables**
1. [Splitwise](https://www.splitwise.com/about): An app that allows users to create groups, add expenses, and see net balances. Limitations: lack of organization options apart from creating groups, doesn't have a clear way to track personal expenses and doesn't address the issue of managing budgets.


5. [Google Sheets](https://workspace.google.com/products/sheets/): A platform for collaborative spreadsheets. It allows users to customize how they track expenses, but requires manual setup and maintenance and is error-prone with multiple collaborators.

## Pitch: Moneta

Managing shared and personal expenses during trips often leads to confusion over who owes whom and stress about staying within budget. **Moneta** simplifies expense management with a flexible, customizable system that keeps both personal and group spending neat, budgets clear, and maintains an organized historical record of past expenses.

### Key Features
1. **Expense Tracker**
Allows users to track both personal and group expenses. Users can create expense groups and invite others to collaborate, or use a group solely for personal tracking. When adding an expense, users can specify the category, who paid, and who was covered, making it easy to keep track of spending and shared costs. This lets users clearly see who owes whom what.

2. **Budget Manager**
Lets users set and manage budgets at different levels—per group, by category, or overall. The app tracks spending across all groups and supports recurring budgets for things like weekly groceries or regular bills, helping users stay on top of their finances. If users choose to share their budgets in a group, this can help groups manage their spending more effectively.

2. **Organization Dashboard**
Serves as the central hub for organizing expenses across all groups. Users can create custom folders for different groups or time periods, like "Summer 2025". This structure makes it easy to keep a well-organized historical record of past and current expenses, tailored to each user’s preferred way of organizing information.

**Moneta** reduces stress and saves time for both trip organizers and participants. By keeping expenses transparent and organized, it prevents disputes and simplifies reimbursements. With clear budget tracking, users can easily monitor their spending and stay within their limits


## Concept Design
### Concepts
```
concept Authentication
    purpose limit access to known users
    principle after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user.
    state
        a set of Users with
            a username String
            a password String

    actions
        register (username: String, password: String): (user: User)
            requires username to not already exist
            effects creates a new User with given username and password

        authenticate (username: String, password: String): (user: User)
            requires a User to exist with the given username
            effects if a User with the given username exists and the given password matches the User's password then access is granted. Otherwise, access is denied.
```

```
concept ExpenseTracking
    purpose allows users to group expenses and coordinate shared expenses/budgets
    principle after a group is created, users can collaborate in the same group and add shared expenses to the group. each expense keeps track of who paid and who was covered. users can check how much they personally spent in a group.
    state
        a set of Groups with
            a name String
            a creator User
            a set of Users
            a set of Debts
            a set of Expenses

        a set of GroupExpenses
            a title String
            a description String
            a category String
            a totalCost Number
            a payer User
            a date Date
            a set of PersonalExpenses

        a set of PersonalExpenses
            a date Date
            a category String
            a user User
            a payer User
            a personalCost Number


    actions
        createGroup(groupName: String, creator: User, description: String): (group:Group)
            requires creator exists
            effect creates a new group with the given groupName, the creator, the set of users, the description, and the startDate/endDate set to None.

        leaveGroup(group:Group, user:User):
            requires group exists, user is part of group, and user does not owe/is not owed anything in the group
            effect removes the user from the group

        addUser(user: User, friend: User, group: Group):
            requires user exists, friend exists, group exists, user is in group and is friends with the friend. user is not equal to friend.
            effect adds friend to the group

        removeUser(owner: User, user:User, group:Group)
            requires owner exists, user exists, group exists, owner is in the group and is the owner of the group, user is in the group
            effect removes the user from the group

        addExpense(payer:User, title:String, description:String, category: String, totalCost: Number, date:Date, debtMapping: Map<User:Number>):(expense:GroupExpense)
            requires creator exists, totalCost is positive, the values (costs) of the debtMapping add up to the totalCost and all values are positive numbers.
            effect: calls createPersonalExpense for every debt in the debtMapping with given group, category, date, user user in the mapping as the user, and the number the user maps to as the personalCost. Also, creates a GroupExpense with given payer, title, description category, totalCost, date, and the set of PersonalExpenses just created. Updates group's startDate/endDate based on the given date.

        deleteExpense(payer: User, group:Group, groupExpense: GroupExpense): (groupExpense:GroupExpense)
            requires payer and expense to exist, payer is the payer of the expense, and expense is in group.
            effect deletes the groupExpense and all the PersonalExpenses associated with it.

        editExpense(payer:User, oldExpense: GroupExpense, title:String, description:String, category: String, totalCost: Number, date:Date, debtMapping: Map<User:Number>): (newExpense:GroupExpense):
            requires payer and oldExpense to exist and payer is the payer of the expense.
            effect: updates the oldExpense with the new given arguments. calls editPersonalExpense on each old personalExpense associated with the oldExpense and updates each given the given arguments. If a personalExpense is no longer associated with the group expense, call deletePersonalExpense on it. The new groupExpense stores this set of new PersonalExpenses. Updates group's startDate/endDate based on the given date.

        createPersonalExpense(user:User, payer: User, group:Group, category:String, date:Date, personalCost:Number):
            requires user, payer, group to exist. user and payer are in group.
            effect: creates a PersonalExpense with the given user, payer, group, category,date, and personalCost

        editPersonalExpense(oldPersonalExpense:PersonalExpense, group: Group, category:String,date:Date,personalCost:Number):
            requires personalExpense exists and is associated with group.
            effect: edits the PersonalExpense with the given user,payer, group, category,date, and personalCost

        deletePersonalExpense(personalExpense: PersonalExpense):
            requires personalExpense to exist
            effect: deletes the personalExpense.

        getUserSpending(user:User,group:Group):(spending:Number)
            requires user and group to exist, user is in group
            effect gets all the PersonalExpenses associated with the user in the group, adds them up, and returns them.
```

```
concept Debts [User]
    purpose allows users to track debts
    principle after creating a debt between two users, the system will update the debts as users add more expenses or repay each other. users can check how much they owe or are owed with another user.
    state
        a set of Debts
            a userA User
            a userB User
            a group Group
            a debt Number
    actions
        createDebt(payer:User, receiver:User, group:Group,cost:Number): (debt:Debt)
            requires payer, receiver, group exists, a debt with payer, receiver, and group doesn't already exist (even with the userA and userB switched), and cost is a postive number. payer is not equal to receiver
            effect creates a new Debt with payer as userA, receiver as userB, group as group, and cost as the debt

        updateDebt(payer:User, receiver:User, group:Group, cost:Number): (debt:Debt)
            requires player, receiver, group exists, payer and receiver are in the group, a debt between payer and receiver already exists for the group (payer can be either userA or userB), cost is positive
            effect finds the debt associated with payer, receiver, and group. If payer is userA, add cost to the debt. Otherwise, subtract cost from the debt.

        getDebt(user:User, otherUser:User, group:Group): (debt:Debt)
            requires user, otherUser, and group exists and a debt between the three to exist
            effect returns the amount that the user owes the other user. If the number is negative, then the otherUser owes the user.
```

```
concept BudgetTracking
    purpose allows users to manage how much they spend
    principle after creating a budget, the system will update total spending for that budget, and users can check how much of their budget they have used.
    state
        a set of Budgets with
            a user User
            a group Group
            a category String
            a limit Number
            a totalSpent Number
            a startDate Date
            an endDate Date
            a pinned Flag

    actions
        createBudget (user: User, group: Group, category:String, limit: Number, startDate: Date, endDate: Date, pinned: Flag) : (budget:Budget)
            requires user exist, user is part of the group, limit is a positive number, endDate to be after startDate. budget with all the same parameters does not exist yet
            effect creates a new budget for the user's given group of expenses from the start date to end date with the limit and a pinned flag.

        editBudget (user: User, budget: Budget, newLimit: Number, newStartDate: Date, newEndDate: Date, pinned: Flag)
            requires user and budget to exist and budget is associated with the user
            effect edits the budget with the new information: newLimit, newStartDate, newEndDatem pinned

        deleteBudget (budget: Budget)
            requires budget to exist and user is associated with the budget
            effect deletes the budget

        addTotalSpent (expense: PersonalExpense)
            requires expense to exist and a budget with the user, group, category, and date to exist (date is within the start and end date of a budget).
            effect for each budget associated with the user, group, category, and date of the given expense, update the totalSpent to add the cost of the given expense.

        decreaseTotalSpent (expense: PersonalExpense)
            requires expense to exist and a budget with the user, group, category, and date to exist (date is within the start and end date of a budget).
            effect for each budget associated with the user, group, category, and date of the given expense, update the totalSpent to subtract the cost of the given expense.

        checkBudget (user:User, budget: Budget) : (budget: Budget)
            requires budget to exist and user is associated with the budget
            effect returns how much the user has spent and how much the user has left of the budget.
```

```
concept Friends
    purpose allows users to connect with other users for easier group formations
    principle users can form friendship by knowing each other's usernames. users can also delete friends and view their friends list.
    state
        a set of Friendships with
            a userA User
            a userB user

    actions
        addFriend(user:User, otherUsername:String): (friendship:Friendship)
            requires user to exist and otherUsername to be associated with an existing user. and a friendship between them does not already exist. and user is not equal to otherUser.
            effect creates a friendship between user and otherUser.

        removeFriend(user:User, otherUser:User):
            requires user and otherUesr exists and a friendship between them exists
            effect deletes the friendship between the user and otherUser

        getFriends(user:User):
            requires user to exist
            effect gets all the friendships associated with the user. If no friendships with the user exist, return none
```


```
concept Folder
    purpose allows users to organize groups into custom structures
    principle
    state
        a set of Folders with
            a parent Folder
            a name String
            an owner User
            a set of Groups
    actions
        createFolder(owner:User, name:String): (folder:Folder)
            requires owner and folder name doesn't already exist with the owner
            effect creates a new folder with the given name and owner

        addToFolder(user:User,folderName:String, group:Group):
            requires folder with folderName and group exists. folder belongs to owner and user is in the group
            effect adds the group into the folder

        removeFromFolder(user:User,folder:Folder, group:Group):
            requires user, folder and group exists. folder belongs to user, user is in the group, and group is inside folder
            effect removes the group from the folder

        deleteFolder(user:User, folder:Folder):
            requires user and folder exists. folder belongs to user
            effect deletes the folder and moves all groups to the home page (no inside any folder)

        renameFolder(user:User, folder:Folder, name:String):
            requires user and folder exists and user is the folder's owner
            effect changes name of folder to name
```

### syncs

```
sync trackSpending
    when ExpenseTracking.createPersonalExpense(): (expense:PersonalExpense):
    then BudgetTracker.addTotalSpent(expense)
```

```
sync removeSpending
    when ExpenseTracking.deletePersonalExpense(): (expense:PersonalExpense):
    then BudgetTracker.decreaseTotalSpent(expense)
```


```
sync editSpendingDecrease
    when ExpenseTracking.editPersonalExpense(oldExpense:PersonalExpense):
    then BudgetTracker.decreaseTotalSpent(expense:oldExpense)
```

```
sync editSpendingIncrease
    when ExpenseTracking.editPersonalExpense(): (newExpense:PersonalExpense):
    then BudgetTracker.addTotalSpent(expense:newExpense)
```

```
sync createDebt
    when ExpenseTracking.createGroup(members):(group:Group)
    then Debt.createDebt(userA, userB, group) between each possible pair of users in the set of members.
```

```
sync addDebt
    when ExpenseTracking.addPersonalExpense(user, payer, group, personalCost): ():
    then Debt.updateDebt(payer:payer,receiver:user, group:group, cost:personalCost)
```
```
sync removeDebt
    when ExpenseTracking.deletePersonalExpense(user,payer,group,personalCost):()
    then Debt.updateDebt(payer: user, receiver: payer, cost:personalCost)
```

```
sync budgetExceededNotif
    when BudgetTracking.totalSpent > BudgetTracking.budget
    then notify the user that their budget has been exceeded
```

```
sync createHomeFolder
    when Authentication.register():(user:User)
    then Folder.createFolder(user:user,name:"home")
```
```
sync addGroupToDash
    when ExpenseTracking.createGroup(creator):(group:Group) >
    then Folder.addToFolder(group:group, user:creator, folderName:"home")
```
```
sync addSharedGroupToDash
    when ExpenseTracking.addUser(friend):(group:Group) >
    then Folder.addToFolder(group:group, user:friend, folderName:"home")
```



### Concepts in Context
Each concept in Moneta plays a distinct role in supporting expense management while interacting with others through synchronizations. Authentication establishes the identity of users and controls access to other concepts: only authenticated users can create or join groups, add expenses, or view debts. Users must be part oa a group to view the group. ExpenseTracking provides the core structure for organizing expenses, grouping them by trips or activities. Debts ensures that shared expenses are fairly distributed by mapping payers to receivers, and is automatically updated whenever a new expense is added. BudgetTracking supports financial awareness by maintaining user and group budgets, which are updated when expenses are logged, and triggers notifications when limits are exceeded. The Friendship system makes it easier to form groups and share expenses. Finally, Folders provides a customizable organizational layer, letting users sort groups or budgets in ways that suit their personal preferences. When a group is created, it is automatically added to each associated user's dashboard. The User type in all concepts come from Authentication.

## UI Sketches
![UI_sketch](/assets/sketches.jpeg)


## User Journey

Jay is planning a summer trip to Southeast Asia with four friends. In the past, handling money on group trips has always been stressful for him, especially since he is the primary organizer and often fronts large payments. Without knowing how much he spent on himself, Jay tends to overspend, thinking most of his money went toward covering others. Determined to simplify things, Jay suggests using Moneta to manage shared costs and track personal expenses.

All five friends create Moneta accounts and add each other as friends. Anticipating multiple stops—Vietnam, Singapore, and Thailand—Jay creates a folder on his dashboard called “SE Asia 2025.” As the primary organizer of Vietnam, he creates a group named “Vietnam” inside this folder and adds his friends. Megan sets up the “Singapore” and “Thailand” groups and adds Jay. Jay moves these groups into his SE Asia folder to keep everything organized.

Before the trip, Jay sets budgets for each category for each leg of the trip. He sets specific budgets for food and lodging in each country but prefers an overall budget for shopping across all three countries. Using the financial overview tab, he creates a new budget for shopping spanning the dates of the entire trip and pins it to his dashboard for quick access.

Expenses start with Jay paying for flights and Megan paying for Airbnbs. Both add their expenses in the appropriate groups, assign categories, and split costs among members. Moneta automatically updates Jay’s budgets, and he sees that lodging costs were lower than expected. He adjusts his budget, reallocating some lodging funds to food.

During the trip, Jay continues using Moneta to track both shared and personal expenses, easily seeing how much of his budget he has spent.

At the end of the trip, the friends review all expenses and spending on Moneta at the airport. They check what they owe each other and settle payments on the spot, recording them in Moneta. Jay is relieved that all the money matters went smoothly this time and now he has a nice record of all their expenses throughout the trip!
