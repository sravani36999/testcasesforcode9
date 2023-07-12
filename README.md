# Authentication

Given an `app.js` file and a database file `userData.db` consisting of a table `user`.

Write APIs to perform operations on the table `user` containing the following columns,

**User Table**

| Column   | Type |
| -------- | ---- |
| username | TEXT |
| name     | TEXT |
| password | TEXT |
| gender   | TEXT |
| location | TEXT |

### API 1

#### Path: `/register`

#### Method: `POST`

**Request**

```
{
  "username": "adam_richard",
  "name": "Adam Richard",
  "password": "richard_567",
  "gender": "male",
  "location": "Detroit"
}
```

- **Scenario 1**

  - **Description**:

    If the username already exists

  - **Response**
    - **Status code**
      ```
      400
      ```
    - **Status text**
      ```
      User already exists
      ```

- **Scenario 2**

  - **Description**:

    If the registrant provides a password with less than 5 characters

  - **Response**
    - **Status code**
      ```
      400
      ```
    - **Status text**
      ```
      Password is too short
      ```

- **Scenario 3**

  - **Description**:

    Successful registration of the registrant

  - **Response**
    - **Status code**
      ```
      200
      ```
    - **Status text**
    ```
    User created successfully
    ```

### API 2

#### Path: `/login`

#### Method: `POST`

**Request**

```
{
  "username": "adam_richard",
  "password": "richard_567"
}
```

- **Scenario 1**

  - **Description**:

    If an unregistered user tries to login

  - **Response**
    - **Status code**
      ```
      400
      ```
    - **Status text**
      ```
      Invalid user
      ```

- **Scenario 2**

  - **Description**:

    If the user provides incorrect password

  - **Response**
    - **Status code**
      ```
      400
      ```
    - **Status text**
      ```
      Invalid password
      ```

- **Scenario 3**

  - **Description**:

    Successful login of the user

  - **Response**
    - **Status code**
      ```
      200
      ```
    - **Status text**
      ```
      Login success!
      ```

### API 3

#### Path: `/change-password`

#### Method: `PUT`

**Request**

```
{
  "username": "adam_richard",
  "oldPassword": "richard_567",
  "newPassword": "richard@123"
}
```

- **Scenario 1**

  - **Description**:

    If the user provides incorrect current password

  - **Response**
    - **Status code**
      ```
      400
      ```
    - **Status text**
      ```
      Invalid current password
      ```

- **Scenario 2**

  - **Description**:

    If the user provides new password with less than 5 characters

  - **Response**
    - **Status code**
      ```
      400
      ```
    - **Status text**
      ```
      Password is too short
      ```

- **Scenario 3**

  - **Description**:

    Successful password update

  - **Response**
    - **Status code**
      ```
      200
      ```
    - **Status text**
      ```
      Password updated
      ```

<br/>

Use `npm install` to install the packages.

**Export the express instance using the default export syntax.**

**Use Common JS module syntax.**

const express = require("express");
const { open } = require("sqlite");
const sqlite3 = require("sqlite3");
const path = require("path");

const databasePath = path.join(\_\_dirname, "userData.db");

const app = express();

app.use(express.json());

let database = null;

const initializeDbAndServer = async () => {
try {
database = await open({
filename: databasePath,
driver: sqlite3.Database,
});

    app.listen(3003, () =>
      console.log("Server Running at http://localhost:3003/")
    );

} catch (error) {
console.log(`DB Error: ${error.message}`);
process.exit(1);
}
};

initializeDbAndServer();

const validatePassword = (password) => {
return password.length > 4;
};

app.post("/register", async (request, response) => {
const { username, name, password, gender, location } = request.body;
const hashedPassword = await bcrypt.hash(password, 10);
const selectUserQuery = `SELECT * FROM user WHERE username = '${username}';`;
const databaseUser = await database.get(selectUserQuery);

if (databaseUser === undefined) {
const createUserQuery = ` INSERT INTO user(username, name, password, gender, location) VALUES ( '${username}', '${name}', '${hashedPassword}', '${gender}', '${location}' ); `;
if (validatePassword(password)) {
await database.run(createUserQuery);
response.send("User created successfully");
} else {
response.status(400);
response.send("Password is too short");
}
} else {
response.status(400);
response.send("User already exists");
}
});

//2
app.post("/login", async (request, response) => {
const { username, password } = request.body;
const selectUserQuery = `SELECT * FROM user WHERE username = '${username}';`;
const databaseUser = await database.get(selectUserQuery);

if (databaseUser === undefined) {
response.status(400);
response.send("Invalid user");
} else {
const isPasswordMatched = await bcrypt.compare(
password,
databaseUser.password
);
if (isPasswordMatched === true) {
response.send("Login success!");
} else {
response.status(400);
response.send("Invalid password");
}
}
});
//3

app.put("/change-password", async (request, response) => {
const { username, oldPassword, newPassword } = request.body;
const selectUserQuery = `SELECT * FROM user WHERE username = '${username}';`;
const databaseUser = await database.get(selectUserQuery);

if (databaseUser === undefined) {
response.status(400);
response.send();
}
});
