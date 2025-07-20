### Part 1: The Login Process (The Top Section)

1.  **User & Login:** The process begins when a `User` initiates a `Login`. This is the act of providing credentials (like a username and password) to prove their identity.

2.  **AWS IAM: The Security System:** The login request is sent to the central `AWS IAM` service. As the diagram shows, IAM is designed to handle two key functions:
    *   **`1. Authentication`**: IAM first checks if the user's login credentials are correct.
    *   **`2. Authorization`**: IAM is also responsible for checking permissions, which we'll see in the next step.

3.  **The "Authentication" Arrow:** The arrow leaving the IAM box labeled `Authentication` signifies that the authentication process was successful. The outcome is a confirmed identity.

### Part 2: The "Authenticated User" and Their Actions (The Bottom Section)

4.  **Authenticated User:** The green box represents the user's state after a successful login. AWS now knows *who* the user is. However, knowing who they are doesn't automatically grant them permission to do everything.

5.  **Accessing Services (Authorization in Action):** This is the most crucial part of the diagram for understanding the difference. When the `Authenticated User` tries to perform any action—like accessing `EC2`, `S3`, or `Lambda`—a new check occurs.

    As the text on the arrows clearly explains, **this check is AUTHORIZATION**.
