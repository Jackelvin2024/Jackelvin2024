Part 1: Implement “Delete User” Feature

Backend - authController.js:

Firstly, in authController.js file we will create a delete_user_by_username function which will take care of authentication and authorization checks before deletion of user by their username.
// authController.js

const { UserModel } = require('../models/UserModel');  // Assuming UserModel is your model

// Controller function to delete user by username
exports.delete_user_by_username = async (req, res) => {
    try {
        const { username } = req.body;
        
        // Check if user exists
        const user = await UserModel.findOne({ where: { username } });
        if (!user) {
            return res.status(404).json({ message: "User not found" });
        }
        
        // Delete the user
        await UserModel.destroy({
            where: { username }
        });

        return res.status(200).json({ message: "User deleted successfully" });
    } catch (error) {
        return res.status(500).json({ message: "Internal Server Error", error: error.message });
    }
};
Routes - authHandling.js:

The next step is to create the route that will execute the controller function that will delete the user. We shall employ the PUT method to show that a resource (user) has been modified (deleted).

// authHandling.js

const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');
const { authentication, authorisation } = require('../middleware/authMiddleware');

// Route for deleting a user
router.put(
    "/delete/user",
    authentication,    // Middleware to check if the user is authenticated
    authorisation({ isAdmin: false }),  // Middleware to authorize (example: users can delete themselves, but not others)
    (req, res) => authController.delete_user_by_username(req, res)
);

module.exports = router;
Frontend - deleteUser.js:

A simple form has to be included where a user can remove the username he wants after submitting it. A JavaScript snippet that listens for the form submission, sends the username on the backend and handles response is also required.

<!-- deleteUser.html -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Delete User</title>
</head>
<body>
    <h1>Delete User</h1>
    <form id="delete-user-form">
        <label for="other-username">Username:</label>
        <input type="text" id="other-username" name="username" required>
        <button type="submit">Delete User</button>
    </form>

    <script src="deleteUser.js"></script>
</body>
</html>
// deleteUser.js

document.getElementById("delete-user-form").addEventListener("submit", async (event) => {
    event.preventDefault();
    
    const username = document.getElementById("other-username").value;
    
    try {
        const response = await fetch(`http://localhost:4001/auth/delete/user`, {
            method: "PUT",  // Use PUT to match the backend route
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify({ username })
        });

        const data = await response.json();

        if (response.ok) {
            alert(data.message);
        } else {
            alert(`Error: ${data.message}`);
        }
    } catch (error) {
        alert("An error occurred: " + error.message);
    }
});

Part 2: Explanation of Authentication and Authorization

Is it Good or Bad to Have “Delete User Functionality After Authentication?”

Explanation:

Authentication is the process which confirms that a person is who he/she claims to be. It is a means of verifying the identity (login with username and password, tokens etc.) of a user trying to gain access to an application.

Authorization refers to deciding on what actions authenticated users are allowed to perform. It involves checking their permissions as well as determining whether they are authorized for carrying out certain operations such as deleting a user or viewing confidential data.

Authentication Before Delete Operation:

It’s necessary and good that authentication should be required in case of delete operation. This makes sure that only verified users could access the application functionality including delete operation. In absence f authentication, any user (even anonymous ones) can attempt at deleting accounts thereby creating serious security problems.

Authorization After Authentication:

Authentication by itself is not enough but we also need authorization so that the user has permission to delete accounts. Only admins should have rights when it comes to deleting any user while regular users might afford deletion of their own accounts alone.

In this context, the provided example allows authenticated u

Diagram:

A simple flow of how authentication and authorization work

User Request -> Authentication -> Authorized? -> Yes -> Perform Delete Operation
                                               -> No  -> Deny Access

