<?php
include '../incluides/owner.php';

$result = "";

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $command = trim($_POST["command"] ?? "");

    $usersFile = "../data/users.json";
    $users = json_decode(file_get_contents($usersFile), true);

    if (preg_match('/\/desuspenderUser\s+#execute\s+&user\s*=>\s*\("(.+)"\)/', $command, $matches)) {
        $targetName = $matches[1];
        $found = false;

        foreach ($users as $email => &$user) {
            if ($user["name"] === $targetName) {
                $user["suspended_until"] = null;
                $found = true;
                $result = "Usuario desuspendido correctamente: " . $targetName;
                break;
            }
        }

        if (!$found) {
            $result = "No se encontró el usuario exacto: " . $targetName;
        }

        file_put_contents($usersFile, json_encode($users, JSON_PRETTY_PRINT));
    }

    elseif (preg_match('/\/desbanUser\s+#execute\s+&user\s*=>\s*\("(.+)"\)/', $command, $matches)) {
        $targetName = $matches[1];
        $found = false;

        foreach ($users as $email => &$user) {
            if ($user["name"] === $targetName) {
                $user["banned"] = false;
                $found = true;
                $result = "Usuario desbaneado correctamente: " . $targetName;
                break;
            }
        }

        if (!$found) {
            $result = "No se encontró el usuario exacto: " . $targetName;
        }

        file_put_contents($usersFile, json_encode($users, JSON_PRETTY_PRINT));
    }

    else {
        $result = "Comando no reconocido.";
    }
}

$pageTitle = "Terminal Owner";
include '../incluides/header.php';
?>

<a class="volver" href="index.php">← Volver al panel</a>

<h1>Terminal Owner</h1>

<div class="version-card">
    <p>Comandos disponibles:</p>

    <pre>/desuspenderUser
#execute &user => ("NombreExacto")</pre>

    <pre>/desbanUser
#execute &user => ("NombreExacto")</pre>
</div>

<form method="POST" class="comment-form">
    <textarea name="command" placeholder='Escribe el comando Owner aquí...' required></textarea>
    <button type="submit">Ejecutar comando</button>
</form>

<?php if($result !== ""): ?>
    <div class="safe-box">
        <p><?php echo htmlspecialchars($result); ?></p>
    </div>
<?php endif; ?>

<?php include '../incluides/footer.php'; ?>
