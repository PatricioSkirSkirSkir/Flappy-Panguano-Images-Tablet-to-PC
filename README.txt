<?php
session_start();

include 'config.php';
include 'database/database.php';

$error = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $name = trim($_POST["name"]);
    $email = strtolower(trim($_POST["email"]));
    $password = $_POST["password"];

    if ($name === "" || $email === "" || $password === "") {
        $error = "Completa todos los campos.";
    } else {
        $stmt = $pdo->prepare("SELECT id FROM users WHERE email = ?");
        $stmt->execute([$email]);

        if ($stmt->fetch()) {
            $error = "Ese correo ya está registrado.";
        } else {
            $verified = $email === $officialOwnerEmail ? 1 : 0;
            $role = $email === $officialOwnerEmail ? "Owner" : "Usuario";
            $immune = $email === $officialOwnerEmail ? 1 : 0;

            $stmt = $pdo->prepare("
                INSERT INTO users 
                (name, email, password, role, verified, warnings, suspended_until, banned, immune, created_at)
                VALUES 
                (?, ?, ?, ?, ?, 0, NULL, 0, ?, ?)
            ");

            $stmt->execute([
                htmlspecialchars($name),
                $email,
                password_hash($password, PASSWORD_DEFAULT),
                $role,
                $verified,
                $immune,
                date("d/m/Y H:i")
            ]);

            $_SESSION["user"] = [
                "name" => htmlspecialchars($name),
                "email" => $email,
                "role" => $role,
                "verified" => (bool)$verified,
                "warnings" => 0,
                "suspended_until" => null,
                "banned" => false,
                "immune" => (bool)$immune,
                "created_at" => date("d/m/Y H:i")
            ];

            header("Location: index.php");
            exit;
        }
    }
}
?>

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Registrarse</title>
    <link rel="icon" type="image/svg+xml" href="assets/logos/launcher.svg">
    <link rel="stylesheet" href="style.css">
</head>
<body>

<a class="volver" href="index.php">← Volver</a>

<h1>Crear cuenta</h1>

<form method="POST" class="comment-form">
    <input type="text" name="name" placeholder="Nombre de usuario" required>
    <input type="email" name="email" placeholder="Correo" required>
    <input type="password" name="password" placeholder="Contraseña" required>
    <button type="submit">Registrarme</button>
</form>

<p><?php echo $error; ?></p>

</body>
</html>
