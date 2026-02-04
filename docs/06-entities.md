# 06 - Entity CRUD

> **Agent:** Entities Agent 📦
> **Goal:** Create complete CRUD for all Entities

---

## Context

**Product:** Women's Soccer Hub
**Entities:** User, Post
**Entity Count:** 2

---

## Overview

Per Entity will be created:
- 1x TypeScript Types File
- 1x Zod Schema File
- 2x API Routes (List/Create + Detail)
- 1x SWR Hooks File
- 2x Pages (List + Detail)
- 2x Components (List + Detail)

**Total Files per Entity:** 9
**Total Files overall:** 18

| Entity | Table | API Endpoints | Pages |
|--------|-------|---------------|-------|
| User | users | /api/users | /users, /users/[id] |
| Post | posts | /api/posts | /posts, /posts/[id] |

---


---

## Entity 1: User

**Description:** A registered user of the platform
**Table Name:** `users`

**Fields:**
- `id`: uuid - Primary key
- `created_at`: datetime - Creation timestamp
- `updated_at`: datetime - Last update timestamp
- `user_id`: uuid - Owner user ID
- `username`: string - Username chosen by the user
- `email`: string - Email address of the user

**Relationships:**
- one_to_many → Post: A user can create many posts

**Related Flows:**
- **User Registration:** User submits registration form → System sends verification email
- **Post Creation:** User submits post form → System creates new post



### Instructions: User

#### 1. TypeScript Types

Create **Types File** (`types/user.ts`):

**⚠️ CRITICAL: No Duplicate Fields!**
Each field must appear EXACTLY ONCE. Use consistent naming (snake_case for DB fields).

```typescript
// ✅ CORRECT: Each field once
export interface User {
  id: string;
  user_id: string;
  created_at: string;
  updated_at: string;
  // Custom fields:
  username: string;
  email: string;
}

// ❌ WRONG: Duplicate fields!
export interface User {
  id: string;        // 1x
  userId: string;    // mixed camelCase
  created_at: Date;  // wrong type (use string)
  user_id: string;   // DUPLICATE (snake_case)!
  id: string;        // DUPLICATE!
}
```

**UserListItem Interface:**
- Reduced version for List Views
- Only essential fields: id, created_at + first 3 custom fields

#### 2. Zod Validation Schema

Create **Schema File** (`lib/schemas/user.ts`):

**CreateUserSchema:**
- Zod Object with all custom fields
- Required vs Optional based on Field Definition
- Type Mapping:
  - string/text → z.string()
  - number/integer → z.number() / z.number().int()
  - boolean → z.boolean()
  - date/datetime → z.string() (ISO format)

**UpdateUserSchema:**
- Use Create Schema with .partial()
- All fields optional for updates

**Exports:**
- Schemas + Inferred Types

#### 3. API Routes - List & Create

Create **List/Create Route** (`app/api/users/route.ts`):

**GET /api/users (List):**
- Auth Check: User from Supabase
- Query Params: limit (default 20), cursor (pagination)
- Supabase Query:
  - Filter: user_id = current user
  - Order: created_at DESC
  - Limit: limit + 1 (for hasMore detection)
  - Cursor: created_at < cursor (if provided)
- Response: { data: items, meta: { next_cursor, has_more } }
- Error Handling: 401 (unauthorized), 500 (server error)

**POST /api/users (Create):**
- Auth Check: User from Supabase
- Body Validation: CreateUserSchema with safeParse
- Insert: Merge validated data + user_id
- Select: .select().single() for Created Object
- Response: { data } with 201 Status
- Error Handling: 400 (validation), 500 (server error)

#### 4. API Routes - Detail Operations

Create **Detail Route** (`app/api/users/[id]/route.ts`):

**GET /api/users/[id] (Read):**
- Params: id (await params promise)
- Auth Check: User from Supabase
- Query: .select().eq("id", id).eq("user_id", user.id).single()
- Response: { data }
- Error: 404 if not found

**PATCH /api/users/[id] (Update):**
- Params: id (await params promise)
- Auth Check + Body Validation
- Update: .update().eq("id").eq("user_id").select().single()
- Response: { data }
- Error: 400 (validation), 404 (not found)

**DELETE /api/users/[id] (Delete):**
- Params: id (await params promise)
- Auth Check
- Delete: .delete().eq("id").eq("user_id")
- Response: 204 No Content
- Error: 404 (not found)

**Important for all Routes:**
- User ID Check prevents access to other users' data
- NextRequest/NextResponse for Types
- await params as Promise (Next.js 15)

#### 5. SWR Data Fetching Hooks

Create **Hooks File** (`hooks/useUsers.ts`):

**useUsers() Hook:**
- SWR for List Fetching
- URL: /api/users
- Returns: { users, isLoading, error, mutate }

**useUser(id) Hook:**
- SWR for Single Item Fetching
- URL: /api/users/{id}
- Conditional: only fetch if id is provided
- Returns: { user, isLoading, error, mutate }

**useCreateUser() Hook:**
- POST Request Function
- Mutate List after Success
- Error Handling with throw

**useUpdateUser(id) Hook:**
- PATCH Request Function
- Mutate Item + List after Success
- Error Handling with throw

**useDeleteUser() Hook:**
- DELETE Request Function
- Mutate List after Success
- Error Handling with throw

**Important:**
- All Hooks are Client Components ("use client")
- Fetcher Function for SWR
- Proper Error Handling

#### 6. List Page

Create **List Page** (`app/(app)/users/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Client Component for List Rendering

**Layout:**
- Container with Padding
- Header: Title + "New" Button (right)
- List Component Integration

#### 7. List Component

Create **List Component** (`components/user/UserList.tsx`):

**States:**
- Loading: Skeleton Grid (6 items)
- Error: Error Message Card
- Empty: EmptyState Component with "Create New" Action
- Success: Grid with Items

**Grid Layout:**
- Responsive: 1 col (mobile), 2 col (tablet), 3 col (desktop)
- Gap: gap-4

**Item Cards:**
- Link to Detail Page
- Hover Effect: border-primary
- Content: Title + Created Date (adapt to Entity fields)

**Important:**
- "use client" Directive
- useUsers() Hook for Data
- Proper Loading/Error/Empty States

#### 8. Detail Page

Create **Detail Page** (`app/(app)/users/[id]/page.tsx`):

**Functionality:**
- Server Component
- Auth Check
- Direct Supabase Fetch for SSR
- notFound() if Item doesn't exist
- Pass Data to Detail Component

**Security:**
- user_id Check in Query

#### 9. Detail Component

Create **Detail Component** (`components/user/UserDetail.tsx`):

**Props:**
- `user`: User Object

**Layout:**
- Back Button (top left)
- Delete Button (top right)
- Card with Details:
  - ID
  - All Custom Fields (from Entity Definition)
  - Created/Updated Timestamps

**Delete Functionality:**
- Confirm Dialog
- useDeleteUser() Hook
- Router Push to List after Success
- Loading State during Delete

**Important:**
- "use client" Directive
- Router for Navigation
- Confirmation before Delete



---

## Entity 2: Post

**Description:** A discussion post created by a user
**Table Name:** `posts`

**Fields:**
- `id`: uuid - Primary key
- `created_at`: datetime - Creation timestamp
- `updated_at`: datetime - Last update timestamp
- `user_id`: uuid - Owner user ID
- `title`: string - Title of the post
- `content`: text - Content of the post

**Relationships:**
- many_to_one → User: A post is created by one user

**Related Flows:**
- **Post Creation:** User submits post form → System creates new post

**API Endpoints:**
- `GET /api/posts`: Retrieve a list of discussion posts
- `POST /api/posts`: Create a new discussion post

### Instructions: Post

#### 1. TypeScript Types

Create **Types File** (`types/post.ts`):

**⚠️ CRITICAL: No Duplicate Fields!**
Each field must appear EXACTLY ONCE. Use consistent naming (snake_case for DB fields).

```typescript
// ✅ CORRECT: Each field once
export interface Post {
  id: string;
  user_id: string;
  created_at: string;
  updated_at: string;
  // Custom fields:
  title: string;
  content: string;
}

// ❌ WRONG: Duplicate fields!
export interface Post {
  id: string;        // 1x
  userId: string;    // mixed camelCase
  created_at: Date;  // wrong type (use string)
  user_id: string;   // DUPLICATE (snake_case)!
  id: string;        // DUPLICATE!
}
```

**PostListItem Interface:**
- Reduced version for List Views
- Only essential fields: id, created_at + first 3 custom fields

#### 2. Zod Validation Schema

Create **Schema File** (`lib/schemas/post.ts`):

**CreatePostSchema:**
- Zod Object with all custom fields
- Required vs Optional based on Field Definition
- Type Mapping:
  - string/text → z.string()
  - number/integer → z.number() / z.number().int()
  - boolean → z.boolean()
  - date/datetime → z.string() (ISO format)

**UpdatePostSchema:**
- Use Create Schema with .partial()
- All fields optional for updates

**Exports:**
- Schemas + Inferred Types

#### 3. API Routes - List & Create

Create **List/Create Route** (`app/api/posts/route.ts`):

**GET /api/posts (List):**
- Auth Check: User from Supabase
- Query Params: limit (default 20), cursor (pagination)
- Supabase Query:
  - Filter: user_id = current user
  - Order: created_at DESC
  - Limit: limit + 1 (for hasMore detection)
  - Cursor: created_at < cursor (if provided)
- Response: { data: items, meta: { next_cursor, has_more } }
- Error Handling: 401 (unauthorized), 500 (server error)

**POST /api/posts (Create):**
- Auth Check: User from Supabase
- Body Validation: CreatePostSchema with safeParse
- Insert: Merge validated data + user_id
- Select: .select().single() for Created Object
- Response: { data } with 201 Status
- Error Handling: 400 (validation), 500 (server error)

#### 4. API Routes - Detail Operations

Create **Detail Route** (`app/api/posts/[id]/route.ts`):

**GET /api/posts/[id] (Read):**
- Params: id (await params promise)
- Auth Check: User from Supabase
- Query: .select().eq("id", id).eq("user_id", user.id).single()
- Response: { data }
- Error: 404 if not found

**PATCH /api/posts/[id] (Update):**
- Params: id (await params promise)
- Auth Check + Body Validation
- Update: .update().eq("id").eq("user_id").select().single()
- Response: { data }
- Error: 400 (validation), 404 (not found)

**DELETE /api/posts/[id] (Delete):**
- Params: id (await params promise)
- Auth Check
- Delete: .delete().eq("id").eq("user_id")
- Response: 204 No Content
- Error: 404 (not found)

**Important for all Routes:**
- User ID Check prevents access to other users' data
- NextRequest/NextResponse for Types
- await params as Promise (Next.js 15)

#### 5. SWR Data Fetching Hooks

Create **Hooks File** (`hooks/usePosts.ts`):

**usePosts() Hook:**
- SWR for List Fetching
- URL: /api/posts
- Returns: { posts, isLoading, error, mutate }

**usePost(id) Hook:**
- SWR for Single Item Fetching
- URL: /api/posts/{id}
- Conditional: only fetch if id is provided
- Returns: { post, isLoading, error, mutate }

**useCreatePost() Hook:**
- POST Request Function
- Mutate List after Success
- Error Handling with throw

**useUpdatePost(id) Hook:**
- PATCH Request Function
- Mutate Item + List after Success
- Error Handling with throw

**useDeletePost() Hook:**
- DELETE Request Function
- Mutate List after Success
- Error Handling with throw

**Important:**
- All Hooks are Client Components ("use client")
- Fetcher Function for SWR
- Proper Error Handling

#### 6. List Page

Create **List Page** (`app/(app)/posts/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Client Component for List Rendering

**Layout:**
- Container with Padding
- Header: Title + "New" Button (right)
- List Component Integration

#### 7. List Component

Create **List Component** (`components/post/PostList.tsx`):

**States:**
- Loading: Skeleton Grid (6 items)
- Error: Error Message Card
- Empty: EmptyState Component with "Create New" Action
- Success: Grid with Items

**Grid Layout:**
- Responsive: 1 col (mobile), 2 col (tablet), 3 col (desktop)
- Gap: gap-4

**Item Cards:**
- Link to Detail Page
- Hover Effect: border-primary
- Content: Title + Created Date (adapt to Entity fields)

**Important:**
- "use client" Directive
- usePosts() Hook for Data
- Proper Loading/Error/Empty States

#### 8. Detail Page

Create **Detail Page** (`app/(app)/posts/[id]/page.tsx`):

**Functionality:**
- Server Component
- Auth Check
- Direct Supabase Fetch for SSR
- notFound() if Item doesn't exist
- Pass Data to Detail Component

**Security:**
- user_id Check in Query

#### 9. Detail Component

Create **Detail Component** (`components/post/PostDetail.tsx`):

**Props:**
- `post`: Post Object

**Layout:**
- Back Button (top left)
- Delete Button (top right)
- Card with Details:
  - ID
  - All Custom Fields (from Entity Definition)
  - Created/Updated Timestamps

**Delete Functionality:**
- Confirm Dialog
- useDeletePost() Hook
- Router Push to List after Success
- Loading State during Delete

**Important:**
- "use client" Directive
- Router for Navigation
- Confirmation before Delete


---

## Validation Checklist

Verify for EACH Entity:
- [ ] Types File created with correct Interfaces
- [ ] Zod Schema File with Create + Update Schemas
- [ ] API List/Create Route works
- [ ] API Detail Route works (GET, PATCH, DELETE)
- [ ] SWR Hooks File with all 5 Hooks
- [ ] List Page with Auth Check
- [ ] List Component with Loading/Error/Empty States
- [ ] Detail Page with Auth Check
- [ ] Detail Component with Delete Functionality

**Test per Entity:**
1. **Create:** Go to /users/new → Create Item → Redirect to List
2. **List:** List shows new Item
3. **Read:** Click on Item → Detail Page loads
4. **Update:** (built in Phase 08-forms)
5. **Delete:** Delete Button → Confirmation → Back to List

**API Tests:**
- POST /api/[entity]s → 201 with Created Item
- GET /api/[entity]s → 200 with Array
- GET /api/[entity]s/[id] → 200 with Object
- PATCH /api/[entity]s/[id] → 200 with Updated Item
- DELETE /api/[entity]s/[id] → 204

---

## Troubleshooting

**API 401 Errors:**
- Check Auth Check in all Routes
- Supabase Client correctly created?
- User is correctly retrieved?

**RLS Policy Errors:**
- Check if user_id is in Query
- RLS Policies active in Supabase?

**Pagination not working:**
- Cursor Logic: created_at < cursor
- hasMore Detection: length > limit

**SWR not updating:**
- Call mutate() after Mutations
- Fetcher Function correct?

**TypeScript Errors:**
- Types from database.types.ts generated?
- Zod Schemas export inferred Types

---

**Continue to:** `07-dashboard.md`
