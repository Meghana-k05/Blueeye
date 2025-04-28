<?php
$servername = "localhost";
$username = "root";
$password = "";
$database = "blueeye_photography";

// Create connection
$conn = new mysqli($servername, $username, $password, $database);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
?>
<?php include('db_connect.php'); ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Register - Blueeye Photography</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div class="form-container">
        <h2>User Registration</h2>
        <form action="register.php" method="post">
            <input type="text" name="firstname" placeholder="First Name" required>
            <input type="text" name="lastname" placeholder="Last Name" required>
            <input type="text" name="username" placeholder="Username" required>
            <input type="password" name="password" placeholder="Password" required>
            <button type="submit" name="register">Register</button>
        </form>
    </div>

<?php
if (isset($_POST['register'])) {
    $firstname = $_POST['firstname'];
    $lastname  = $_POST['lastname'];
    $username  = $_POST['username'];
    $password  = password_hash($_POST['password'], PASSWORD_BCRYPT);

    $sql = "INSERT INTO users (firstname, lastname, username, password) 
            VALUES ('$firstname', '$lastname', '$username', '$password')";
    if ($conn->query($sql)) {
        echo "<p>Registration successful! You can now <a href='login.php'>login</a>.</p>";
    } else {
        echo "Error: " . $conn->error;
    }
}
?>
</body>
</html>
<?php include('db_connect.php'); session_start(); ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login - Blueeye Photography</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div class="form-container">
        <h2>User Login</h2>
        <form action="login.php" method="post">
            <input type="text" name="username" placeholder="Username" required>
            <input type="password" name="password" placeholder="Password" required>
            <button type="submit" name="login">Login</button>
        </form>
    </div>

<?php
if (isset($_POST['login'])) {
    $username = $_POST['username'];
    $password = $_POST['password'];

    $query = "SELECT * FROM users WHERE username='$username'";
    $result = $conn->query($query);

    if ($result->num_rows == 1) {
        $user = $result->fetch_assoc();
        if (password_verify($password, $user['password'])) {
            $_SESSION['username'] = $username;
            header('Location: gallery.php');
        } else {
            echo "<p>Invalid password.</p>";
        }
    } else {
        echo "<p>User not found.</p>";
    }
}
?>
</body>
</html>
<?php include('db_connect.php'); session_start(); ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Gallery - Blueeye Photography</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <h2>Welcome to the Gallery!</h2>
    <div class="gallery">
        <?php
        $sql = "SELECT * FROM galleries";
        $result = $conn->query($sql);

        while($row = $result->fetch_assoc()) {
            echo "<div class='photo'>";
            echo "<img src='uploads/" . $row['image'] . "' alt='Photo'>";
            echo "<p>" . $row['title'] . "</p>";
            echo "</div>";
        }
        ?>
    </div>
    <a href="logout.php">Logout</a>
</body>
</html>
<?php include('db_connect.php'); session_start(); ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Upload Photo - Admin</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div class="form-container">
        <h2>Upload a New Photo</h2>
        <form action="upload.php" method="post" enctype="multipart/form-data">
            <input type="text" name="title" placeholder="Photo Title" required>
            <input type="file" name="image" required>
            <button type="submit" name="upload">Upload</button>
        </form>
    </div>

<?php
if (isset($_POST['upload'])) {
    $title = $_POST['title'];
    $image = $_FILES['image']['name'];
    $target = "uploads/" . basename($image);

    $sql = "INSERT INTO galleries (title, image) VALUES ('$title', '$image')";
    if ($conn->query($sql)) {
        move_uploaded_file($_FILES['image']['tmp_name'], $target);
        echo "<p>Photo uploaded successfully!</p>";
    } else {
        echo "Error: " . $conn->error;
    }
}
?>
</body>
</html>
<?php include('db_connect.php'); ?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Feedback - Blueeye Photography</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <div class="form-container">
        <h2>Leave Your Feedback</h2>
        <form action="feedback.php" method="post">
            <input type="text" name="name" placeholder="Your Name" required>
            <input type="email" name="email" placeholder="Your Email" required>
            <textarea name="message" placeholder="Write your message..." required></textarea>
            <button type="submit" name="submit_feedback">Submit Feedback</button>
        </form>
    </div>

<?php
if (isset($_POST['submit_feedback'])) {
    $name = $_POST['name'];
    $email = $_POST['email'];
    $message = $_POST['message'];

    $sql = "INSERT INTO feedback (name, email, message) VALUES ('$name', '$email', '$message')";
    if ($conn->query($sql)) {
        echo "<p>Thank you for your feedback!</p>";
    } else {
        echo "Error: " . $conn->error;
    }
}
?>
</body>
</html>
<?php
session_start();
session_destroy();
header("Location: login.php");
exit();
?>
body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #f4f4f9;
    margin: 0;
    padding: 0;
}

h2 {
    text-align: center;
    color: #333;
    margin-top: 20px;
}

.form-container {
    width: 300px;
    margin: 30px auto;
    padding: 20px;
    background: white;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
    border-radius: 8px;
}

input, textarea {
    width: 100%;
    padding: 10px;
    margin-top: 10px;
    margin-bottom: 15px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

button {
    width: 100%;
    background-color: #0077ff;
    color: white;
    padding: 10px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

button:hover {
    background-color: #005dc1;
}

.gallery {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
}

.photo {
    margin: 15px;
    text-align: center;
}

.photo img {
    width: 250px;
    height: auto;
    border-radius: 8px;
}
