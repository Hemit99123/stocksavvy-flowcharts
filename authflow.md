## Authentication Flow

This flow contains 2 apps:
- The main website
- A CLI tool used by developers to assign users to admin roles

The flow contains logic for a session-based authentication and a role based authentication system with a user and admin role. 

### 🚀 Main website:
```mermaid 
flowchart 
    A([Start Website]) --> B[OTP Generation]
    B --> C[Store OTP in Redis]
    C --> D([OTP Expiry])
    D --> O([End])
  
    C --> E{User's OTP matches Redis records}
    E -- Yes --> F[Assign Session]
    E -- No --> O
  
    F --> G[Assign Session to Redis]
    G --> H[Session Details<br/>• Email<br/>• Time Created<br/>• Role]
    H --> I[Create Cookie with Session ID]
    I --> J{Check Session ID with Redis?}
    J -- True --> K{Check Role?}
    J -- False --> O
  
    K -- User --> L[User App]
    K -- Admin --> M[Admin App]
    L --> N([Logout])
    M --> N
    N --> P([Delete Sessions])
    P --> O


```

### 🎮 RACT (Role Assignment CLI Tool):

All commands run in infinite loops so they will always return to the start of the program. To stop it, you must press Ctrl+C like any other CLI-based tool.

```mermaid
flowchart 
    Start["Start"] --> CommandChoice{"Developer selects a command"}
    
    CommandChoice -- Add/Update Configuration --> RequestDBCredentials(["Prompt for PostgreSQL and Redis connection URLs"])
    RequestDBCredentials --> CheckConfigFile{"Does 'credentials.json' exist?"}
    CheckConfigFile -- Yes --> LoadAndUpdateConfig(["Load 'credentials.json' and update database fields"])
    CheckConfigFile -- No --> CreateConfigFile(["Create 'credentials.json' and add database fields"])
    LoadAndUpdateConfig --> ReturnToStart1(["Return to start"])
    CreateConfigFile --> ReturnToStart1
    ReturnToStart1 --> Start

    CommandChoice -- Update User Role --> RequestUserEmail(["Prompt for target user's email"])
    RequestUserEmail --> CheckCredentialsFile{"Does 'credentials.json' exist?"}
    CheckCredentialsFile -- Exists --> ReadCredentials(["Read credentials from file"])
    CheckCredentialsFile -- Does Not Exist --> ReturnToStart2(["Return to start"])
    ReadCredentials --> ConnectToDB(["Use credentials to connect to PostgreSQL and Redis"])
    ConnectToDB --> RetrieveUserData(["Retrieve user sessions from Redis and user record from PostgreSQL"])

    RetrieveUserData --> DeleteUserSessions(["Delete all user sessions via RediSearch index"])
    DeleteUserSessions --> PromptForRoleChange{"Select new role type"}
    PromptForRoleChange -- Admin --> UpdateToAdmin(["Set user role to 'admin' in PostgreSQL"])
    PromptForRoleChange -- User --> UpdateToUser(["Set user role to 'user' in PostgreSQL"])
    UpdateToAdmin --> ReturnToStart2(["Return to start"])
    UpdateToUser --> ReturnToStart2
    ReturnToStart2 --> Start
```
