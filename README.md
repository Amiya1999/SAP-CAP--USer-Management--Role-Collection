<!-- fetch('odata/v4/terminal/Terminal')
    .then(response => response.json())
    .then(data => {
        console.log(data.value);

        // Find the index of the logged-in user in the data
        const selfIndex = data.value.findIndex(user => user.id === loggedInId);
        console.log(selfIndex);

        // Get details of the logged-in user
        const loggedInUser = data.value.find(user => user.id === loggedInId);
        console.log(loggedInUser);

        // Display details of the logged-in user in the header
        displaySelfInHeader(loggedInUser);

        // Filter out the logged-in user from the list of other users
        const otherUsers = data.value.filter((user, index) => index !== selfIndex);
        console.log(otherUsers);

        // Append other users to the table
        appendOtherUsersToTable(otherUsers);
    })
    .catch(error => {
        console.error('Error fetching user data:', error);
    }); -->




<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>User Dashboard</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
        }

        header {
            background-color: #e2e1e1;
            color: #fff;
            padding: 10px;
            text-align: right;
        }

        .logout-btn {
            background-color: #fff;
            color: #333;
            padding: 5px 10px;
            text-decoration: none;
            border: 1px solid #333;
            border-radius: 5px;
            margin-right: 10px;
        }

        table {
            width: 80%;
            margin: 20px auto;
            border-collapse: collapse;
        }

        th, td {
            border: 1px solid #ddd;
            padding: 10px;
            text-align: left;
        }

        th {
            background-color: #f2f2f2;
        }

        select {
            width: 100%;
            padding: 5px;
            box-sizing: border-box;
        }
    </style>
</head>
<body>

    <header>
        <h5 id="loggedInUser"></h5>
        <button class="logout-btn" onclick="logout()">Logout</button>
    </header>

    <table id="memberList">
        <thead>
            <tr>
                <th>Name</th>
                <th>Email</th>
                <th>Roles</th>
                <th>Assign Roles</th>
            </tr>
        </thead>
        <tbody>
            <!-- User data will be appended here dynamically -->
        </tbody>
    </table>

<script>
    function logout(){
        localStorage.removeItem('token');
        window.location.href = 'userlogin.html';
}
function parseJwt(token) {
  try {
    const decoded = JSON.parse(atob(token.split('.')[1]));
    return decoded;
  } catch (error) {
    console.error('Error parsing JWT token:', error);
    return null;
  }
}

let email;
let loggedInUserRole;
let roleid;
// let rolesdata={};
const authToken = localStorage.getItem('token');

// Parse JWT token to get the logged-in user's email
const decodedToken = parseJwt(authToken);
const loggedInId = decodedToken ? decodedToken.logID : null;
console.log('Logged-in User ID:', loggedInId);

// Function to create a dropdown for assigning roles in the table
function createRoleDropdownInTable(onChangeHandler) {
    const dropdown = document.createElement('select');
    dropdown.id = 'assignRolesDropdown'; 
    dropdown.name = 'assignRolesDropdown';

    // Fetch roles from your roles entity (replace with your actual API endpoint)
    fetch('odata/v4/roles/Roles')
        .then(response => response.json())
        .then(data => {
            console.log(data.value);
            const emptyOption = document.createElement('option');
            emptyOption.value = '';  // Set the value as per your requirement
            emptyOption.text = 'Select Role';  // Set the text as per your requirement
            dropdown.appendChild(emptyOption);

            // Create and append option elements for each role
            data.value.forEach(role => {
                const option = document.createElement('option');
                option.value = role.role; // Adjust accordingly based on your data structure
                option.text = role.role;   // Adjust accordingly based on your data structure
                dropdown.appendChild(option);
            });

            // Add event listener for the change event
            dropdown.addEventListener('change', onChangeHandler);
        })
        .catch(error => {
            console.error('Error fetching roles:', error);
        });

    return dropdown;
}

// Function to append other users to the table
// Function to append other users to the table
function appendOtherUsersToTable(users) {
    const memberListBody = document.querySelector('#memberList tbody');

    // Iterate over other users
    users.forEach(user => {
        // Create an empty table row
        const tableRow = document.createElement('tr');

        // Create table cells for user details
        const nameCell = document.createElement('td');
        nameCell.textContent = user.username;
        const emailCell = document.createElement('td');
        emailCell.textContent = user.email;
        const rolesCell = document.createElement('td');

     fetch(`/odata/v4/user-roles/UserRole`)
    .then(response => response.json())
    .then(userRolesData => {
        console.log(userRolesData.value);
        console.log(user.email);
        let memberRoles = [];
        userRolesData.value.forEach(userRole => {
            if (userRole.user_id === user.id) {
                memberRoles.push(userRole.role_id);
                console.log(userRole.role_id,'poppppp');
                fetch(`/odata/v4/roles/Roles?$filter=id eq ${userRole.role_id}`)
                    .then(response => response.json())
                    .then(roleData => {
                        if (roleData.value && roleData.value.length > 0) {
                            // Extract role names from the array
                            const roleNames = roleData.value.map(role => role.role);

                            // Set the content of rolesCell with comma-separated role names
                            rolesCell.textContent = roleNames.join(', ');
                        } else {
                            // No roles found
                            rolesCell.textContent = 'No Role';
                        }
                    })
                    .catch(error => {
                        console.error('Error fetching role details:', error);
                    });
            }
        });
    })
    .catch(error => {
        console.error('Error fetching member roles:', error);
    });

        // Create a cell for the assign roles dropdown
        const assignRolesCell = document.createElement('td');
        const assignRolesDropdown = createRoleDropdownInTable(() => {
            let role = roleid;
            id = user.id;
            console.log(roleid,'ddddddddddd');
            assignRoles(id, role);
        });
        assignRolesCell.appendChild(assignRolesDropdown);

        // Append cells to the row
        tableRow.appendChild(nameCell);
        tableRow.appendChild(emailCell);
        tableRow.appendChild(rolesCell);
        tableRow.appendChild(assignRolesCell);

        // Append the row to the table body
        memberListBody.appendChild(tableRow);
    });
}


// Function to display details of the logged-in user in the header
function displaySelfInHeader(selfUser) {
    const loggedInUserHeader = document.getElementById('loggedInUser');
    loggedInUserHeader.textContent = `Logged-in User: ${selfUser.username} (${selfUser.email}) - Role: ${loggedInUserRole || 'No Role'}`;
}

// Fetch user data from the Terminal table
fetch('odata/v4/terminal/Terminal')
    .then(response => response.json())
    .then(data => {
        console.log(data.value);

        // Find the index of the logged-in user in the data
        const selfIndex = data.value.findIndex(user => user.id === loggedInId);
        console.log(selfIndex);

        // Get details of the logged-in user
        const loggedInUser = data.value.find(user => user.id === loggedInId);
        console.log(loggedInUser);
        loggedInUserRole=// Fetch roles for the logged-in user from the UserRole table
fetch(`/odata/v4/user-roles/UserRole?$filter=user_id eq ${loggedInId}`)
    .then(response => response.json())
    .then(loggedInUserRolesData => {
        console.log(loggedInUserRolesData.value);
        let loggedInUserRoles = [];

        loggedInUserRolesData.value.forEach(userRole => {
            loggedInUserRoles.push(userRole.role_id);

            // Fetch role details from Roles entity based on role_id
            fetch(`/odata/v4/roles/Roles?$filter=id eq ${userRole.role_id}`)
                .then(response => response.json())
                .then(roleData => {
                    if (roleData.value && roleData.value.length > 0) {
                        // Extract role names from the array
                        const roleNames = roleData.value.map(role => role.role);

                        // Set the content of loggedInUserRole with comma-separated role names
                        loggedInUserRole = roleNames.join(', ');

                        // Display details of the logged-in user in the header
                        displaySelfInHeader(loggedInUser);
                    } else {
                        // No roles found
                        loggedInUserRole = 'No Role';
                    }
                })
                .catch(error => {
                    console.error('Error fetching role details:', error);
                });
        });
    })
    .catch(error => {
        console.error('Error fetching roles for the logged-in user:', error);
    });

        // Filter out the logged-in user from the list of other users
        const otherUsers = data.value.filter((user, index) => index !== selfIndex);
        console.log(otherUsers);

        // Append other users to the table
        appendOtherUsersToTable(otherUsers);
    })
    .catch(error => {
        console.error('Error fetching user data:', error);
    });

function assignRoles( id, role) {
  const roleData = {
    user_id: id,
    role_id: role
  };
  console.log(roleData);

  fetch(`/odata/v4/user-roles/AssignRoles`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(roleData)
  })
  .then(response => response.json())
  .then(data => {
    console.log('Roles assigned successfully:', data);
  })
  .catch(error => {
    console.error('Error assigning roles:', error);
  });
}
</script>
</body>
</html>

