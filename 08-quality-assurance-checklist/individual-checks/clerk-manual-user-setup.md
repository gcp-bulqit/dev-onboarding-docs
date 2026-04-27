# Clerk Manual User Setup (Development/Preview)

Quick guide for manually creating and linking Clerk users to vendor/homeowner accounts.

## Steps

### 1. Create User in Clerk Dashboard

1. Go to [Clerk Dashboard](https://dashboard.clerk.com)
2. Select your application (development or preview environment)
3. Navigate to **Users** → **Create User**
4. Fill in user details (email, password, etc.)

### 2. Set User Role (Public Metadata)

After creating the user:

1. Click on the newly created user
2. Go to **Metadata** tab
3. Under **Public metadata**, click **Edit**
4. Add the following JSON:
   ```json
   {
     "role": "vendor"
   }
   ```
   Or for homeowner:
   ```json
   {
     "role": "homeowner"
   }
   ```
5. Click **Save**

### 3. Copy Clerk User ID

1. In the user details page, copy the **User ID** (starts with `user_...`)
2. Keep this ID for the next step

### 4. Link to Database Record

**For Vendors:**
```sql
UPDATE vendors
SET clerk_user_id = 'user_xxxxxxxxxxxxxxxxxxxxx'
WHERE email = 'vendor@example.com';
```

**For Homeowners:**
```sql
UPDATE home_owners
SET clerk_user_id = 'user_xxxxxxxxxxxxxxxxxxxxx'
WHERE email = 'homeowner@example.com';
```

## Quick Test Users (Local/Preview)

You can use these pre-configured test accounts:

### Vendor Test Account
- **Email:** `vendor-test@bulqit.local`
- **Password:** `TestVendor123!`
- **Role:** `vendor`
- **Clerk User ID:** *(Get from Clerk Dashboard)*

### Homeowner Test Account
- **Email:** `homeowner-test@bulqit.local`
- **Password:** `TestHomeowner123!`
- **Role:** `homeowner`
- **Clerk User ID:** *(Get from Clerk Dashboard)*

---

## Alternative: Automated Setup (Registration Flow)

Instead of manual setup, you can use the normal registration flow:

1. Go to `/register/vendor` (for vendors) or `/register/homeowner` (for homeowners)
2. Complete the registration form
3. The system will automatically:
   - Create Clerk user
   - Set the role in `publicMetadata`
   - Link `clerkUserId` to the database record

This is done via the `setUserRole()` function in [updateClerkRole.ts](../apps/web/src/actions/updateClerkRole.ts).

---

## Troubleshooting

**"Account Not Found" error on dashboard:**
- Verify the `clerk_user_id` is correctly set in the database
- Check that the role in Clerk matches the user type (vendor/homeowner)

**Can't access dashboard:**
- Ensure `publicMetadata.role` is set in Clerk
- Verify the user exists in the database with matching email
