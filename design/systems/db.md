# Database

## User

### User Entries

```javascript
const User = new mongoose.Schema({
    username: String,
    profile: {
        name: String,
        dob: Date,
        email: String,
        ... etc        
    },
    feedRatio: [ Number, Number, Number],
            // in-network, near-network, overall
    pInclude: {
        // These keys are used to populate the profile views for different network states.
        in: [ ObjectKeys ],
        near: [ ObjectKeys ],
        overall: [ ObjectKeys ]
    }
    posts: [ Post._id ],
    questions: [ Q._id ],
    answers: [ A._id ],  // The answer schema references the question that was answered
    network: [ User._id ]
});
```

### Post

```javascript
const Post = new mongoose.Schema({
    title: String,
    body: String,
    href: String,   // e.g. linked image, article, video, etc...
    tags: [ Tag._id ],
    mentions: [ User._id ],     // tag people
    comments: [ {user: User._id, body: String} ],
    score: [ {user: User._id, vote: Number} ]
});
```

## QA

### Question

```javascript
const Q = new mongoose.Schema({
    title: String,
    body: String,
    tags: [ Tag._id ],
    comments: [ Qcomment._id ],
    score: [ {user: User._id, vote: Number} ]
});
```

### Answer

``` javascript
const A = new mongoose.Schema({
    body: String,
    score: [ {user: User._id, vote: Number} ],
    ref: Q._id
});
```

## Docs

TODO: Research and planning needed

## API

### Mod

TODO: Research and planning needed