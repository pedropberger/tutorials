# How I manage users in my elasticsearch cluster

Okay, grab a coffee (or your beverage of choice), let's chat about Elasticsearch users and roles. Not too long ago, I was working on this project – medium-sized, couple of teams hitting the same Elasticsearch cluster for logs, metrics, you name it. Initially, it was the wild west, everyone kinda used the built-in `elastic` superuser or maybe a shared basic account. Yeah, not great. 😅

Things started getting messy. We needed to restrict who could see what, who could *change* things (definitely didn't want accidental index deletions!), and who just needed dashboard access in Kibana. It was time to get serious about users and roles.

Now, you *can* do all this through the Kibana UI, which is pretty slick. But honestly? I found myself needing to script things, especially when setting up new environments or needing repeatable configurations. Plus, I'm a big Python fan, and I love keeping little utility scripts in Jupyter notebooks. Makes it super easy to document, tweak, and re-run. So, I ended up doing most of the heavy lifting with `curl` (for quick checks) and Python scripts (for the repeatable stuff).

Here's a rundown of how I tackled it, hoping it helps you out too!

**Disclaimer:** Always be careful with security configurations! Test in a non-production environment first. And please, use strong passwords and manage credentials securely (like using environment variables or secrets management, not hardcoding!). For clarity, I'm putting placeholders like `YOUR_ELASTIC_PASSWORD` directly in the examples here. Don't do that in production!


![pixel art image of user management in a system](https://github.com/user-attachments/assets/ed491cc5-4379-4f93-ba5e-755107ba21c2)

**Prerequisites:**

1.  **Elasticsearch Cluster Running:** Obviously, if you don`t have, but want one [click here](https://github.com/pedropberger/tutorials/blob/main/elastic/es_basic_security_cluster.md).
2.  **Security Enabled:** This whole user/role thing only works if Elasticsearch security features are turned on. If you haven't done this, it usually involves adding `xpack.security.enabled: true` to your `elasticsearch.yml` and running a setup command. Check the official docs, as the exact steps can vary. You'll get a password for the built-in `elastic` superuser during this process – guard it well!
3.  **Credentials:** You'll need the `elastic` user's password (or another user with sufficient privileges, like `superuser`) to manage users and roles.
4.  **Tools:** `curl` installed, and Python (ideally 3.x) with the `requests` library (`pip install requests`).

**The Basics: Talking to Elasticsearch Securely**

First things first, any API call related to security needs authentication.

**With `curl`:**

You use the `-u` flag. Simple enough.

```bash
# Replace with your actual host and port
ELASTIC_HOST="http://localhost:9200"
ELASTIC_USER="elastic"
ELASTIC_PASSWORD="YOUR_ELASTIC_PASSWORD" # Keep this secure!

# Example: Get cluster health (just to test connection)
curl -k -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" "${ELASTIC_HOST}/_cluster/health?pretty"
# Note: -k is used here to ignore SSL certificate verification, common in dev setups.
# In production, you should configure proper SSL verification.
```

**With Python (using `requests`):**

This is where my notebooks come in handy. I usually have a setup cell at the top.

```python
import requests
import json
import os

# --- Configuration ---
# Best practice: Use environment variables or a config file in real projects
ELASTIC_HOST = "http://localhost:9200"
ELASTIC_USER = "elastic"
ELASTIC_PASSWORD = os.environ.get("ELASTIC_PASSWORD", "YOUR_ELASTIC_PASSWORD") # Example using env var fallback

# Disable warnings for self-signed certificates if needed (like -k in curl)
# In production, configure proper SSL verification!
requests.packages.urllib3.disable_warnings()
VERIFY_SSL = False # Set to True or path to CA bundle in production

# --- Helper Function (Optional but nice) ---
def es_request(method, endpoint, payload=None):
    """Makes an authenticated request to Elasticsearch."""
    headers = {'Content-Type': 'application/json'}
    url = f"{ELASTIC_HOST}/{endpoint}"
    try:
        response = requests.request(
            method,
            url,
            auth=(ELASTIC_USER, ELASTIC_PASSWORD),
            headers=headers,
            json=payload,
            verify=VERIFY_SSL # Use SSL verification setting
        )
        response.raise_for_status() # Raise an exception for bad status codes (4xx or 5xx)
        # Handle potential empty responses for methods like DELETE
        if response.content:
            return response.json()
        else:
            return {"acknowledged": True, "status_code": response.status_code} # Or some other success indicator
    except requests.exceptions.RequestException as e:
        print(f"Error making request to {method} {url}: {e}")
        if hasattr(e, 'response') and e.response is not None:
            print(f"Response status: {e.response.status_code}")
            print(f"Response body: {e.response.text}")
        return None

# --- Test Connection ---
health = es_request("GET", "_cluster/health?pretty")
if health:
    print("Successfully connected to Elasticsearch cluster:")
    print(json.dumps(health, indent=2))
else:
    print("Failed to connect to Elasticsearch.")

```
See? Having that `es_request` function in my notebook makes subsequent calls much cleaner.

**Step 1: Defining Roles - Who Can Do What?**

Roles are the heart of this. They define *permissions*. You can grant permissions at the cluster level (like `monitor`, `manage_security`) or index level (like `read`, `write`, `create_index` on specific index patterns).

Elasticsearch comes with built-in roles (like `kibana_admin`, `logstash_writer`, `superuser`). Check them out first, maybe they cover your needs. But often, you need custom roles based on the Principle of Least Privilege.

Let's create a role, say, `app_log_reader`, who can only *read* indices matching the pattern `app-logs-*`.

**With `curl`:**

```bash
# Make sure ELASTIC_HOST, ELASTIC_USER, ELASTIC_PASSWORD are set from before

ROLE_NAME="app_log_reader"
curl -k -X PUT -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" \
     "${ELASTIC_HOST}/_security/role/${ROLE_NAME}" \
     -H 'Content-Type: application/json' \
     -d'
{
  "cluster": ["monitor"],
  "indices": [
    {
      "names": [ "app-logs-*" ],
      "privileges": ["read", "view_index_metadata"],
      "allow_restricted_indices": false
    }
  ]
}
'
```
*   `cluster: ["monitor"]`: Often useful so users can see cluster status, helps Kibana basic functions.
*   `indices`: An array where you define index-level permissions.
*   `names`: List of index patterns this rule applies to.
*   `privileges`: What they can do. `read` is common, `view_index_metadata` helps UIs like Kibana discover fields.

**With Python:**

Using our helper function:

```python
def create_or_update_role(role_name, role_definition):
    """Creates or updates an Elasticsearch role."""
    print(f"Attempting to create/update role: {role_name}")
    endpoint = f"_security/role/{role_name}"
    response = es_request("PUT", endpoint, payload=role_definition)
    if response and response.get("role", {}).get("created") or response.get("role", {}).get("updated"):
         print(f"Successfully {'created' if response['role']['created'] else 'updated'} role: {role_name}")
         return True
    # PUT can also return acknowledged:true on no-change updates in some versions
    elif response and response.get("acknowledged"):
         print(f"Role {role_name} acknowledged (likely updated or already exists).")
         return True
    else:
        print(f"Failed to create/update role: {role_name}")
        return False

# Define the role structure
app_log_reader_role = {
  "cluster": ["monitor"],
  "indices": [
    {
      "names": [ "app-logs-*" ],
      "privileges": ["read", "view_index_metadata"],
      "allow_restricted_indices": false
    }
  ]
}

# Create the role
create_or_update_role("app_log_reader", app_log_reader_role)

# Maybe another role? Read-only access to everything? Use carefully!
read_all_role = {
    "indices": [
        {
            "names": ["*"],
            "privileges": ["read", "view_index_metadata"]
        }
    ]
}
# create_or_update_role("global_reader", read_all_role) # Uncomment to create
```
This Python function is reusable. I can just define my role structure as a dictionary and call the function. Super handy in a notebook!

**Viewing Roles:**

```bash
# Curl - Get a specific role
curl -k -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" "${ELASTIC_HOST}/_security/role/app_log_reader?pretty"

# Curl - Get all roles
curl -k -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" "${ELASTIC_HOST}/_security/role?pretty"
```

```python
# Python - Get a specific role
role_info = es_request("GET", "_security/role/app_log_reader?pretty")
if role_info:
    print(json.dumps(role_info, indent=2))

# Python - Get all roles
all_roles = es_request("GET", "_security/role?pretty")
# if all_roles: # This can be very verbose!
#     print(json.dumps(all_roles, indent=2))
```

**Step 2: Creating Users - Who Are They?**

Now we create the actual users and assign them the roles we defined.

Let's create a user `dev_team_member` who needs our `app_log_reader` role and maybe the built-in `kibana_user` role to access Kibana dashboards.

**With `curl`:**

```bash
# Make sure ELASTIC_HOST, ELASTIC_USER, ELASTIC_PASSWORD are set

USER_NAME="dev_team_member"
USER_PASSWORD="A_VERY_STRONG_PASSWORD_HERE" # Change this!

curl -k -X POST -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" \
     "${ELASTIC_HOST}/_security/user/${USER_NAME}" \
     -H 'Content-Type: application/json' \
     -d'
{
  "password" : "'"${USER_PASSWORD}"'",
  "roles" : [ "app_log_reader", "kibana_user" ],
  "full_name" : "Dev Team Member",
  "email" : "dev@example.com",
  "metadata" : {
    "team" : "backend",
    "access_level": "read-only-logs"
  }
}
'
# Note: Using POST or PUT works for creating/updating users. POST feels slightly more "create".
```
*   `password`: Set the user's password.
*   `roles`: List of role names to assign.
*   `full_name`, `email`, `metadata`: Optional fields for extra info. Metadata can be useful for tracking!

**With Python:**

```python
def create_or_update_user(username, user_definition):
    """Creates or updates an Elasticsearch user."""
    print(f"Attempting to create/update user: {username}")
    endpoint = f"_security/user/{username}"
    # Using PUT for idempotency (create or update)
    response = es_request("PUT", endpoint, payload=user_definition)
    if response and (response.get("created") or response.get("updated") or response.get("acknowledged")): # Newer versions might just return acknowledged
        status = "created" if response.get("created", False) else "updated/acknowledged"
        print(f"Successfully {status} user: {username}")
        return True
    else:
        print(f"Failed to create/update user: {username}")
        return False

# Define the user
dev_user_details = {
  "password" : "A_VERY_STRONG_PASSWORD_HERE", # Change this and manage securely!
  "roles" : [ "app_log_reader", "kibana_user" ],
  "full_name" : "Dev Team Member",
  "email" : "dev@example.com",
  "metadata" : {
    "team" : "backend",
    "access_level": "read-only-logs"
  }
}

# Create the user
create_or_update_user("dev_team_member", dev_user_details)

# Example: Create another user for a different team
ops_user_details = {
  "password" : "ANOTHER_SECURE_PASSWORD",
  "roles" : [ "kibana_user" ], # Maybe they only need Kibana access for now
  "full_name" : "Ops Team Member",
   "metadata" : { "team": "operations" }
}
# create_or_update_user("ops_monitor", ops_user_details) # Uncomment to create
```
Again, having that function makes it easy to add more users later in my notebook.

**Managing Users (Viewing, Updating Password, Deleting)**

**Viewing Users:**

```bash
# Curl - Get a specific user
curl -k -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" "${ELASTIC_HOST}/_security/user/dev_team_member?pretty"

# Curl - Get all users
curl -k -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" "${ELASTIC_HOST}/_security/user?pretty"
```

```python
# Python - Get a specific user
user_info = es_request("GET", "_security/user/dev_team_member?pretty")
if user_info:
    print(json.dumps(user_info, indent=2))

# Python - Get all users (can be verbose)
# all_users = es_request("GET", "_security/user?pretty")
# if all_users:
#    print(json.dumps(all_users, indent=2))
```

**Changing a User's Password:**

This uses a different endpoint.

```bash
# Curl
USER_NAME="dev_team_member"
NEW_PASSWORD="EVEN_STRONGER_PASSWORD_123"

curl -k -X PUT -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" \
     "${ELASTIC_HOST}/_security/user/${USER_NAME}/_password" \
     -H 'Content-Type: application/json' \
     -d'
{
  "password" : "'"${NEW_PASSWORD}"'"
}
'
```

```python
# Python
def change_user_password(username, new_password):
    """Changes an Elasticsearch user's password."""
    print(f"Attempting to change password for user: {username}")
    endpoint = f"_security/user/{username}/_password"
    payload = {"password": new_password}
    response = es_request("PUT", endpoint, payload=payload)
    if response and response.get("acknowledged"):
         print(f"Successfully changed password for user: {username}")
         return True
    else:
        print(f"Failed to change password for user: {username}")
        return False

# change_user_password("dev_team_member", "EVEN_STRONGER_PASSWORD_123") # Uncomment to run
```

**Updating User Roles/Metadata:**

You just `PUT` the *entire* user definition again to the `_security/user/<username>` endpoint, same as creating. It overwrites the existing user data (except the password, unless you explicitly include it).

**Deleting a User:**

```bash
# Curl
USER_TO_DELETE="old_user_account"
curl -k -X DELETE -u "${ELASTIC_USER}:${ELASTIC_PASSWORD}" \
     "${ELASTIC_HOST}/_security/user/${USER_TO_DELETE}"
```

```python
# Python
def delete_user(username):
    """Deletes an Elasticsearch user."""
    print(f"Attempting to delete user: {username}")
    endpoint = f"_security/user/{username}"
    response = es_request("DELETE", endpoint)
    # DELETE often returns {"found": true, "acknowledged": true} or similar
    if response and (response.get("acknowledged") or response.get("found")):
         print(f"Successfully deleted user: {username}")
         return True
    else:
        # Handle case where user might not exist (DELETE is often idempotent)
        # Check status code if available in response dict, or check response text in case of error
        status_code = response.get("status_code") if isinstance(response, dict) else None
        if status_code == 404:
             print(f"User {username} not found (already deleted?).")
             return True # Consider deletion successful if not found
        print(f"Failed to delete user: {username}. Response: {response}")
        return False

# delete_user("old_user_account") # Uncomment to run
```

**Why the Python Notebook Approach Rocks (for me, anyway)**

1.  **Repeatability:** Setting up dev/staging/prod environments? Just change the `ELASTIC_HOST` and credentials, run the notebook. Boom.
2.  **Documentation:** The notebook itself becomes documentation. Markdown cells explain *why* a role exists, code cells show *how* it's created.
3.  **Source Control:** Check that `.ipynb` file into Git. Track changes to your security setup.
4.  **Experimentation:** Easy to tweak a role definition in a cell, run it, test it, change it back if needed.
5.  **Bulk Operations:** Need to create 10 similar users? Easy to loop in Python.

**Wrapping Up**

So yeah, that's basically how I stumbled through setting up user management. It went from a necessary chore to something quite manageable, especially once I started scripting it in Python notebooks. It forces you to think about permissions properly, and the scripts make it consistent across environments.

Remember the golden rule: **Least Privilege**. Only grant the permissions users absolutely need. Start small, test, and add more if required. Don't hand out `superuser` like candy!

Hope sharing my experience helps you navigate the Elasticsearch security landscape a bit more easily. Happy indexing (and securing)!
