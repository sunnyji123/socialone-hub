# SocialOne Firestore Security Specification

## 1. Data Invariants
1. **User Profiles (`/users/{userId}`)**: A user profile can only be created or modified by the authenticated owner (`request.auth.uid == userId`) or an admin. Users cannot self-assign `role: 'admin'`.
2. **Posts (`/posts/{postId}`)**: Only authenticated users with verified emails can create posts. The `authorId` must strictly equal `request.auth.uid`. Updates are action-based (likes count update, or content edit by original author).
3. **Videos (`/videos/{videoId}`)**: Short videos must have valid creator ID matching `request.auth.uid`. View and like increments must preserve original creator fields.
4. **Stories (`/stories/{storyId}`)**: Temporary stories can only be created by their author (`authorId == request.auth.uid`) and expire in 24 hours.
5. **Messages (`/messages/{messageId}`)**: Chat messages must have `senderId == request.auth.uid`.
6. **Notifications (`/notifications/{notificationId}`)**: Only the recipient or an authenticated system action can read/update notification status.
7. **Reports (`/reports/{reportId}`)**: Any authenticated user can submit a report. Only admins can read, update, or dismiss reports.

## 2. The Dirty Dozen Payloads
1. Attempting to create a user profile with `role: 'admin'` as a normal user -> REJECTED.
2. Attempting to write a post with `authorId` spoofed to another user -> REJECTED.
3. Unauthenticated read/write to sensitive collections -> REJECTED.
4. Attempting to modify `creatorId` during video like update -> REJECTED.
5. Injecting oversized content (1MB string in bio or post) -> REJECTED.
6. Attempting to delete someone else's post or video -> REJECTED.
7. Attempting to alter reports collection without admin privileges -> REJECTED.
8. Unverified email write attempt -> REJECTED.
9. Attempting to delete user document without authorization -> REJECTED.
10. Attempting to bypass path variable limits with malicious IDs -> REJECTED.
11. Malformed message type or payload -> REJECTED.
12. Shadow fields injected into post creation -> REJECTED.
