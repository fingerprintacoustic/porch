# Porch

A Christian social network for sharing prayer requests, testimonies, praise reports, and scripture. It's a single-file web app (`index.html`) on Firebase Auth, Firestore, and Storage, hosted on GitHub Pages.

## Features

- Email/password and Google sign-in, unique usernames, password reset
- Profiles with photo, bio, follower and following counts, and follower lists
- Follow and unfollow
- **Following** feed (you + people you follow) and **Discover** feed (everyone), with infinite scroll
- Post types: Post, Prayer request, Praise report, Testimony, and Scripture, each tagged in the feed
- **Prayer Wall** feed of prayer requests; reacting says **Praying** (on prayer requests) or **Amen** (on everything else)
- Verse of the day (KJV) in the sidebar and at the top of Home on smaller screens
- Bible references in posts (e.g. `John 3:16`, `1 Cor 13:4-7`) link to BibleGateway
- Text and photo posts (photos resized to 1600px JPEG in the browser before upload)
- Comments, share links, @mentions, and auto-linked URLs
- Live notification badge for likes, comments, and follows
- People search by name or username
- Block (also removes follows both ways) and report (posts, comments, and accounts)
- Moderation page for admins: remove content, suspend accounts, dismiss reports
- Light and dark themes, responsive down to mobile with a bottom tab bar

## Setup

1. **Create a Firebase project** at console.firebase.google.com and add a Web app. Copy its config into `firebaseConfig` near the top of the script in `index.html`. Change `APP_NAME` there if you rename the app.
2. **Authentication:** enable the Email/Password and Google providers. Under *Settings → Authorized domains*, add your GitHub Pages subdomain.
3. **Firestore:** create the database in production mode, then paste `firestore.rules` into the *Rules* tab.
4. **Indexes:** create the three composite indexes in `firestore.indexes.json` under Firestore → Indexes:
   - `posts`: `authorId` ascending, `createdAt` descending
   - `posts`: `kind` ascending, `createdAt` descending (Prayer Wall)
   - `reports`: `status` ascending, `createdAt` descending

   If you skip this, the first feed load logs an error in the console with a one-click link to create the missing index.
5. **Storage:** enable Cloud Storage and paste `storage.rules` into its *Rules* tab. Newer Firebase projects need the pay-as-you-go Blaze plan for Storage. It keeps a free tier, but set a budget alert.
6. **Make yourself an admin:** in Firestore, create a collection called `admins` with a document whose ID is your user UID (from Authentication → Users). Add any field, such as `role: "admin"`. The Moderation link appears after you reload.

Alternatively, with the Firebase CLI, run `firebase deploy --only firestore,storage` to push the rules and indexes in one go (`firebase.json` is included).

## Put it in git and deploy (Windows CMD)

```
cd C:\Users\finge\StudioProjects
mkdir porch && cd porch
REM copy index.html, firestore.rules, storage.rules, firestore.indexes.json, firebase.json, README.md here
git init
git add .
git commit -m "Porch social network MVP"
git remote add origin https://github.com/fingerprintacoustic/porch.git
git push -u origin master:main
```

Then enable GitHub Pages on `main`, add a Dynadot CNAME for the subdomain, and add that domain to Firebase's authorized domains.

## Data model

| Path | Contents |
|---|---|
| `users/{uid}` | profile: username, displayName, bio, photoURL, banned |
| `usernames/{lowercase}` | `{ uid }`, which reserves the username |
| `users/{uid}/following/{uid}` · `followers/{uid}` | follow edges, always written as a pair |
| `users/{uid}/blocked/{uid}` | private block list |
| `users/{uid}/notifications/{id}` | like, comment, and follow notifications |
| `posts/{id}` | authorId, kind (`post` · `prayer` · `praise` · `testimony` · `verse`), text, imageURL, likeCount (Amens / Praying), commentCount |
| `posts/{id}/likes/{uid}` · `comments/{id}` | likes and comments |
| `reports/{id}` | moderation queue |
| `admins/{uid}` | admin allow-list (console only) |

The security rules enforce ownership, make blocked users unable to follow, like, comment, or notify, freeze suspended accounts, and tie `likeCount` changes to a like document actually being created or deleted.

## Known MVP limits (next steps)

- **Feed fan-in:** the Following feed queries posts in batches of 30 authors. It's fine for hundreds of follows, but slower past about 1,000. The upgrade is a Cloud Function that fans each new post out to followers' feed documents.
- **Comment count trust:** the rules allow `commentCount` to move by one at a time but can't verify a matching comment was written. A Cloud Function trigger would make it exact.
- **Orphaned images:** when an admin removes someone else's post, the image file stays in Storage.
- **No push notifications** outside the open app, **no DMs, groups, or video** yet. These are phase two.
- **Posts are all public** to signed-in users. Followers-only visibility is also phase two.
