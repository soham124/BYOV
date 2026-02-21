# Fixing Permission and Display Issues

## Issues Found

1. **"Missing or insufficient permissions" error when posting** - Firestore security rules were not configured
2. **Verses not showing in home tab** - Draft posts were being displayed alongside published posts

## Solutions Implemented

### 1. Created Firestore Security Rules
A new `firestore.rules` file has been created with proper security rules. These rules:
- Allow **anyone** to read public posts (not private, not draft)
- Allow **authenticated users** to create new posts
- Allow users to **update/delete only their own posts**
- Protect **private verses** from unauthorized access
- Allow **anyone** to read user profiles
- Allow **authenticated users** to like and comment

### 2. Updated Home Page Filter
The home page now properly filters posts to show only:
- Posts where `isPrivate !== true` (excludes private posts)
- Posts where `isDraft !== true` (excludes draft posts)

This ensures only published, public verses are visible on the home tab.

## Required Action: Deploy Firestore Rules to Firebase

⚠️ **IMPORTANT**: The permission error will persist until you deploy the security rules to your Firebase project. Follow these steps:

### Step 1: Open Firebase Console
1. Go to [Firebase Console](https://console.firebase.google.com)
2. Select your "bringyourownverse" project
3. Navigate to **Firestore Database** in the left sidebar

### Step 2: Update Security Rules
1. Click on the **"Rules"** tab at the top
2. Copy the entire content from `/workspaces/BYOV/firestore.rules`
3. Replace all existing rules with the copied content
4. Click **"Publish"** button

### Step 3: Verify Deployment
1. Wait for the rules to be published (usually takes a few seconds)
2. Try posting a new verse
3. The verse should now appear on the home page (if it's public)

## Testing Your Changes

After deploying the rules, test these scenarios:

### Scenario 1: Post a Public Verse
1. Sign in to your account
2. Go to `/create`
3. Write a verse and leave "Keep this verse private" **unchecked**
4. Click "Publish"
5. Go to home tab - verse should appear

### Scenario 2: Post a Private Verse
1. Go to `/create`
2. Write a verse and **check** "Keep this verse private"
3. Click "Publish"
4. Go to home tab - verse should **NOT** appear (only visible in your profile)

### Scenario 3: Save a Draft
1. Go to `/create`
2. Write a verse
3. Click "Save Draft"
4. Go to home tab - draft should **NOT** appear (only visible in your profile)

## Firestore Rules Explanation

```javascript
// Posts collection rules
match /posts/{postId} {
  // Only read PUBLIC and PUBLISHED posts
  allow read: if resource.data.isPrivate != true && resource.data.isDraft != true;
  
  // Any authenticated user can create posts
  allow create: if request.auth != null;
  
  // Only the author can update/delete
  allow update, delete: if request.auth.uid == resource.data.authorId;
}
```

## Troubleshooting

### Still getting "Missing or insufficient permissions" error?
- Verify you clicked the **"Publish"** button in Firebase
- Clear your browser cache and hard refresh (Ctrl+Shift+R or Cmd+Shift+R)
- Check that you're signed in with an authenticated user

### Verses still not showing?
- Ensure the verse is **not marked as private**
- Ensure the verse is **not saved as draft**
- Check the browser console for any error messages
- Wait a few seconds after publishing for the post to appear

### Can't see someone else's verses?
- Make sure their verses are **not marked as private**
- They must have **published** the verse (not saved as draft)

## File Changes Summary

- **New file**: `firestore.rules` - Contains security rules for Firebase
- **Modified**: `app/page.tsx` - Updated filter to exclude draft posts
