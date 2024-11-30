<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Codeforces Progress Tracker</title>
    <style>
        /* Global Styles */
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background: #111111;
            color: #e1e1e1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-start;
            height: 100%;
            transition: all 0.3s ease;
            overflow-y: auto;
        }

        h1 {
            margin-top: 40px;
            margin-bottom: 20px;
            text-shadow: 0 2px 5px rgba(255, 0, 0, 0.515);
        }

        .user-list {
            width: 80%;
            max-width: 1000px;
            text-align: center;
            margin-top: 40px;
        }

        /* Cards layout */
        .card-container {
            display: flex;
            justify-content: space-around;
            gap: 20px;
            flex-wrap: wrap;
            margin-bottom: 40px;
        }

        .card {
            background: #333;
            border-radius: 10px;
            padding: 20px;
            width: 200px;
            text-align: center;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.6);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
            margin-bottom: 20px;
        }

        .card:hover {
            transform: scale(1.05);
            box-shadow: 0 6px 15px rgba(0, 0, 0, 0.8);
        }

        .card img {
            width: 60px;
            height: 60px;
            border-radius: 50%;
            margin-bottom: 10px;
        }

        .card a {
            color: #fff;
            text-decoration: none;
            font-size: 16px;
            font-weight: bold;
            display: block;
            margin-top: 10px;
            background-color: #000; /* Black button background */
            border: 2px solid #ff5e57; /* Red border */
            padding: 8px;
            border-radius: 8px;
            transition: background-color 0.3s ease, border-color 0.3s ease;
        }

        .card a:hover {
            background-color: #444; /* Darken the background on hover */
            border-color: #ff5e57; /* Keep the red border */
        }

        /* Table layout for remaining users */
        table {
            width: 100%;
            border-collapse: collapse;
            font-size: 14px;
            margin-top: 40px;
        }

        th, td {
            padding: 12px 16px;
            border: 1px solid #444;
            text-align: left;
        }

        th {
            background-color: #333;
            color: #ff5e57;
            cursor: pointer;
        }

        td {
            background-color: #2b2b2b;
        }

        /* Alternate row colors for better readability */
        tr:nth-child(even) {
            background-color: #3b3b3b;
        }

        tr:hover {
            background-color: #444;
        }

        .loader {
            display: none;
            margin-top: 20px;
            color: #ff5e57;
        }

        /* Avatar styling for table */
        .user-avatar {
            width: 32px;
            height: 32px;
            border-radius: 50%;
            margin-right: 12px;
        }

        /* Button-like username links */
        a {
            color: #fff;
            text-decoration: none;
            display: flex;
            align-items: center;
            padding: 10px 20px;
            background-color: #000; /* Black button background */
            border: 2px solid #ff5e57; /* Red border */
            border-radius: 8px;
            transition: background-color 0.3s, transform 0.2s, border-color 0.3s;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5);
        }

        a:hover {
            background-color: #444; /* Darken background on hover */
            transform: scale(1.05);
            border-color: #ff5e57; /* Red border on hover */
        }

        a:active {
            transform: scale(1);
        }

        a img {
            margin-right: 12px;
        }
    </style>
</head>
<body>
    <h1>Code_RED Tracker</h1>
    <div class="user-list" id="user-list">
        <p>Fetching data...</p>
    </div>
    <div class="loader" id="loader">Loading...</div>

    <script>
        // List of Codeforces usernames
        const usernames = [
            "tanvee", "Omar_Faruk_", "apkabiraj", "Md_pranto", "alpha004",
            "ash_kingxD", "joy94", "loser_s", "khan1203", "ifaz007",
            "faysal.ahmed", "antar262", "imranbubt02", "MullaRohan"
        ];

        const userListDiv = document.getElementById("user-list");
        const loader = document.getElementById("loader");

        // Fetch user data from Codeforces API
        async function fetchUserData(handle) {
            try {
                const response = await fetch(`https://codeforces.com/api/user.info?handles=${handle}`);
                const data = await response.json();
                if (data.status === "OK") {
                    const user = data.result[0];
                    return {
                        username: user.handle,
                        avatar: user.avatar,
                        rating: user.rating // Get the rating here
                    };
                }
            } catch (error) {
                console.error(`Failed to fetch data for ${handle}:`, error);
            }
            return null;
        }

        // Fetch user submissions (same as before)
        async function fetchUserSubmissions(handle) {
            try {
                const response = await fetch(`https://codeforces.com/api/user.status?handle=${handle}`);
                const data = await response.json();
                if (data.status === "OK") {
                    const submissions = data.result;
                    const uniqueProblems = new Set(
                        submissions.filter(sub => sub.verdict === "OK").map(sub => sub.problem.contestId + sub.problem.index)
                    );
                    return uniqueProblems.size; // Return the count of unique problems solved
                }
            } catch (error) {
                console.error(`Failed to fetch data for ${handle}:`, error);
            }
            return 0; // Return 0 if there was an error
        }

        // Fetch data for all users and display
        async function displayUserProgress() {
            userListDiv.innerHTML = "<p>Fetching data...</p>";
            loader.style.display = "block";

            const userData = [];
            for (const username of usernames) {
                const user = await fetchUserData(username);
                if (user) {
                    const solvedCount = await fetchUserSubmissions(username);
                    userData.push({
                        username,
                        solvedCount,
                        rating: user.rating, // Include rating in the user data
                        avatar: user.avatar // Get the avatar URL
                    });
                }
            }

            // Sort users by problems solved in descending order
            userData.sort((a, b) => b.solvedCount - a.solvedCount);

            // Generate HTML content for top 5 users in card style
            let cardContent = `
                <div class="card-container">
            `;
            userData.slice(0, 5).forEach(user => {
                cardContent += `
                    <div class="card">
                        <img src="${user.avatar}" alt="${user.username}'s avatar">
                        <a href="https://codeforces.com/profile/${user.username}" target="_blank">
                            ${user.username} (${user.rating})  <!-- Rating added here -->
                        </a>
                        <p>Problems Solved: ${user.solvedCount}</p>
                    </div>
                `;
            });
            cardContent += `</div>`;

            // Generate HTML content for the remaining users in table format
            let tableContent = `
                <table id="user-table">
                    <thead>
                        <tr>
                            <th onclick="sortTable(0)">Rank</th>
                            <th onclick="sortTable(1)">Username</th>
                            <th onclick="sortTable(2)">Problems Solved</th>
                        </tr>
                    </thead>
                    <tbody>
            `;
            userData.slice(5).forEach((user, index) => {
                tableContent += `
                    <tr>
                        <td>${index + 6}</td>
                        <td>
                            <a href="https://codeforces.com/profile/${user.username}" target="_blank">
                                <img src="${user.avatar}" alt="${user.username}'s avatar" class="user-avatar">
                                ${user.username} (${user.rating})  <!-- Rating added here -->
                            </a>
                        </td>
                        <td>${user.solvedCount}</td>
                    </tr>
                `;
            });
            tableContent += "</tbody></table>";

            // Update the page
            userListDiv.innerHTML = cardContent + tableContent;
            loader.style.display = "none";
        }

        // Function to sort the table by column index
        function sortTable(n) {
            const table = document.getElementById("user-table");
            let rows, switching, i, x, y, shouldSwitch, dir, switchcount = 0;
            switching = true;
            dir = "asc"; // Set the sorting direction to ascending

            while (switching) {
                switching = false;
                rows = table.rows;

                for (i = 1; i < (rows.length - 1); i++) {
                    shouldSwitch = false;
                    x = rows[i].getElementsByTagName("TD")[n];
                    y = rows[i + 1].getElementsByTagName("TD")[n];

                    if (dir == "asc") {
                        if (x.innerHTML.toLowerCase() > y.innerHTML.toLowerCase()) {
                            shouldSwitch = true;
                            break;
                        }
                    } else if (dir == "desc") {
                        if (x.innerHTML.toLowerCase() < y.innerHTML.toLowerCase()) {
                            shouldSwitch = true;
                            break;
                        }
                    }
                }

                if (shouldSwitch) {
                    rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
                    switching = true;
                    switchcount++;
                } else {
                    if (switchcount === 0 && dir === "asc") {
                        dir = "desc";
                        switching = true;
                    }
                }
            }
        }

        // Start displaying user progress
        displayUserProgress();
    </script>
</body>
</html>