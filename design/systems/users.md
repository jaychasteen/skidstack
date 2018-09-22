# Users

## Application Design

Note: the users API does not interact directly with the QA database entries. They are retrieved via `_id` reference only

see [QA](qa.md) for more information.

The base URI for the Users application is `/u`

---
### User/Login Routes

When a user wants to log in, they click the login link which redirects to `/u/login`. At this page, they are presented with a form for local auth, or a set of buttons to choose their OAuth provider.

### OAuth

If the user selects an OAuth provider, they are sent off to the provider to log in, then redirected `/u` if the user has already been created, otherwise to `/u/new/finalize`.

### Local

At the `/u/login` page, the user enters their username or email and password, then the login request is POSTed to `/u/login` and if successful, the user is redirected to `/u`.

---
### User/Profile Routes

#### Create

The user creation page is shown at `/u/new`

Then, the user chooses to create a new account using either OAuth or local.

**If OAuth** -> Route through provider, then callback to `/u/new/finalize`. Submit sends POST to `/u/new` containing the completed User object.

**If Local** -> Send to `/u/new?auth=local` then POST `/u/new` and redirect to `/u`. This method creates the user with a single step.

#### Read

A users public profile can be retrieved via `/u/:username`

For a logged in user, this and `/u/:loggedInUsersUsername/profile` are virtually identical. For any other user, /profile redirects to `/u/:username`

The content of this page is determined by checkboxes in the `/u/:username/profile` view. Users can choose what information is shown on their public profile, what is show to everyone, and what is shown to noone.

Full profile information for a user is accessed at `/u/:username/profile` but requires the named user to be logged in.

#### Update

POST request to `/u/:username/profile` will update a logged in user's database entry.

#### Destroy

DELETE request sent to `/u/:username` will permanently remove all references to the USER only. All posts and qa (including comments) will remain, but names, avatars, etc will not be included.

---
### User/Post Routes

#### Create

A logged in user can add a post by sending a POST request to `/u/:username/p/new`

Logged in users can add comments to any post of a user in their network. Comments are POSTed to `/u/:username/p/:postId/addcomment`

#### Read

Posts can be read directly, but more commonly they will be read by a feed. Either way, a GET request to `/u/:username/p/:postId` will return the post data requested via query. e.g.:

`/u/:username/p/:postId?comments=false?truncate=100`

Depending upon the context, the comments can be loaded or disabled. The query string 'comments' is a boolean that determines whether or not comments will be fetched. The default is false

Comments are fetched via `/u/:username/p/c/:commentId`. This will return an array of Comments. It is up to the fetching code to fetch the User data if it is needed via the provided `_id`.

#### Update

A post can be updated by sending a UPDATE request to `/u/:username/p/:postId`. This should happen via on-page edit controls, i.e. on the user's feeds.

#### Destroy

To delete a post, send a delete request to `/u/:username/p/:postId`

This will also delete any comments attached to it from the DB.



### User/Network Routes

#### Create

Network entries can be created in a user's data if and only if the request is approved by both parties. One user sends the request to the other user via link on their profile `/u/:username/request`

This places a notification on the receiving user's `/u` page that will ask them to approve or ignore the request. Clicking the approve link sends a POST request to `/u/link?init=initiatorsUsername?recv=receivingUsername` which adds an entry to each's network property array consisting of the other user's `_id`

#### Read

A user's network can be viewed in two formats: brief or full. The brief form will be a number chosen at random. Full will return all of the user's connections. A GET request is sent to `/u/:username/n` with an optional query string `format=brief`, where the default is full.

#### Update

TODO: Does network need an update route?

#### Destroy

Sending a DELETE request to `/u/:username/n/:userId`, where username is the logged in user and userId refers to the network connection to be deleted.

### User/Feed Routes

#### Create

The `/u` feed is created internally upon request, based upon the preference of the requesting user. This feed object is only used to populate the page and is disposed of immediately. Creation is triggered by hitting `/u`.

Note: a user's own feed (`/u/:username/p`) is not created, it is a sortable list of all of a user's posts

#### Read

When a user hits `/u`, we must start by gathering posts from in-network users, then gathering popular near-network posts (above a certain threshold), then most popular posts overall and sorting them by age (recent first). The ratio each of these categories can be tuned in user profile.

The feed will also support sort by score and most commented.

Filter by in-network, near-network and overall will also be supported.

`/u?sort=[recent,score,comments]&filter=[network,near,overall]`

#### Update

Feed preferences are updated via UPDATE `/u/:username/feed` where the update request is an object adding preferences such as mute and star user.

#### Destroy

A feed is not destroyed. It is a volatile server-memory based object createdh on demand.

### Users/Groups Routes

The Groups subsystem has the base URI `/u/g`

Note: User's contributions to the message board and other features of the group subsystem are only recorded in the Group database entry and not referenced in User.

#### Create

Any logged in user can create a group by POSTing to `/u/g/new`

There is a form at GET `/u/g/new` that allows a user to set the group's metadata and to invite other in-network users to join the group.

#### Read

The main group page can be found at GET `/u/g/new`.
This page shows the metadata, a message board, and links to relevent pages.

User must be a logged in group member to access `/u/g/:groupId`. The parameter here can either be a group name or the `_id`

#### Update

The group administrator (the founder) has access to `/u/g/:groupId/update` where groupId must be the `_id`

This is a form page that allows UPDATE `/u/g/:groupId`

#### Destroy

Send a DELETE request to `/u/g/:groupId` as the group admin to delete all group information. This includes all posts to the message board. 

## Pages

More or less in order of how they would be used.

#### /u/new

A landing page for the user creation sub. Gives the option of logging in via a selection of OAuth providers or creating a local account.

#### /u/new?auth=local

If the query string is used, show a sign-up form.

#### /u/new/finalize

After successful OAuth, call back to this page. This allows the user to preview, proofread and edit the information taken from the provider before creating their account.

#### /u

The landing page for a logged in user. Shows their current feed.

TODO: What else do we include on the logged in landing page?

#### /u/:username

Shows a user profile in 'complete' form

#### /u/:username/profile

For a logged in user only, show the profile but with 'forms' for editing info

#### /u/:username/p/:postId

A get request to this URI will send back the selected post information. Optional query strings allow disabling comments or truncating results.

#### /u/:username/p/c/:commentId

This URI returns a single comment and is used to populate a posts comments. The Post DB entry has a list of comments which are fetched at this URI.

#### /u/:username/n

This page displays the selected user's network with an optional query string that limits the returned data. To get a limited listing, use query string `format=brief`

#### /u/:username/p

Lists all posts by a user, sortable and filterable. This is essentially a feed of a single user.

#### /u/g/new

A form that allows for the creation of a group and inviting users to join.

#### /u/g/:groupId

Short-form group info

Message Board

Links

Members

Admin have special features such as delete message.

#### /u/g/:groupId/update

A form allowing for editing the metadata for the group and for managing membership, i.e. blocking, removing and promoting members to admin.