## IX. TP9 - Templates

### 1. Creer un fichier de Template `index.php.j2` au bon endroit avec le contenu suivant (à vous de remplacer les variables)

```php
<?php

$servername = ????
$username = ???
$password = ???
$dbname = ???

$conn = new mysqli("localhost", $username, $password, $dbname);

if ($conn->connect_error) {
    die("Connection failed:" . $conn->connect_error);
}

$sql = "SELECT name FROM mes_users";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    while($row = $result->fetch_assoc()) {
        echo "Hello " . $row["name"]. "<br>";
    }
} else {
    echo "0 result";
}

$conn->close();
?>
```

### 2. Testez le résultat !

### 3. Modifier votre index.php pour faire une vraie page HTML

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title> A VARIABILISER </title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

    <h1>Bienvenue</h1>
    <p>Ceci est un exemple de page HTML avec un fichier CSS externe.</p>

</body>
</html>
```

### 4. Templatiser le fichier `style.css` pour modifier les parametres suivants (tout doit être variabilisé)

```css
body {
    background-color: ????
    font-family: Arial, sans-serif;
    font-size: 14px;
}
```

### 5. Testez à nouveau !
