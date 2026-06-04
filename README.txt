<?php
session_start();

include 'config.php';
include 'database/database.php';
include 'moderation.php';
include 'incluides/csrf.php';

csrf_check();

$id = $_GET["id"] ?? "";

$newsFile = "data/news.txt";

if (!file_exists($newsFile)) {
    die("No hay noticias.");
}

$news = json_decode(file_get_contents($newsFile), true);

if (!isset($news[$id])) {
    die("Noticia no encontrada.");
}

$item = $news[$id];

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    if (!isset($_SESSION["user"])) {
        die("Debes iniciar sesión para comentar.");
    }

    $message = trim($_POST["message"] ?? "");

    if (($_SESSION["user"]["banned"] ?? false) === true) {
        die('Tu cuenta está baneada. <a href="support.php?type=Apelar baneo">Apelar baneo</a>');
    }

    if (!empty($_SESSION["user"]["suspended_until"]) && strtotime($_SESSION["user"]["suspended_until"]) > time()) {
        die('Tu cuenta está suspendida temporalmente. <a href="support.php?type=Apelar suspensión">Apelar suspensión</a>');
    }

    if (stripos($message, "testban") !== false) {
        addWarning($_SESSION["user"]["email"], "Prueba de moderación");
        die("Comentario bloqueado por moderación. Advertencia agregada.");
    }

    if ($message !== "") {
        $stmt = $pdo->prepare("
            INSERT INTO comments (news_id, user_email, user_name, message)
            VALUES (?, ?, ?, ?)
        ");

        $stmt->execute([
            $id,
            $_SESSION["user"]["email"],
            $_SESSION["user"]["name"],
            $message
        ]);

        header("Location: news_detail.php?id=" . urlencode($id));
        exit;
    }
}

$stmt = $pdo->prepare("SELECT * FROM comments WHERE news_id = ? ORDER BY id DESC");
$stmt->execute([$id]);
$commentsList = $stmt->fetchAll(PDO::FETCH_ASSOC);

$pageTitle = htmlspecialchars($item["title"]) . " - " . $siteName;
include 'incluides/header.php';
?>

<a class="volver" href="news.php">← Volver a noticias</a>

<div class="version-card">
    <h1><?php echo htmlspecialchars($item["title"]); ?></h1>
    <p><?php echo htmlspecialchars($item["content"]); ?></p>
</div>

<h2>Comentarios</h2>

<?php if(isset($_SESSION["user"])): ?>

<form method="POST" class="comment-form">
    <input type="hidden" name="csrf_token" value="<?php echo csrf_token(); ?>">
    <textarea name="message" placeholder="Escribe un comentario..." required></textarea>
    <button type="submit">Enviar comentario</button>
</form>

<?php else: ?>

<div class="safe-box">
    <p>Debes iniciar sesión para comentar.</p>
    <a class="download-btn" href="login.php">Iniciar sesión</a>
    <a class="download-btn" href="register.php">Crear cuenta</a>
</div>

<?php endif; ?>

<?php if(empty($commentsList)): ?>
    <p>No hay comentarios todavía.</p>
<?php endif; ?>

<?php foreach($commentsList as $comment): ?>
    <div class="version-card">
        <h3><?php echo htmlspecialchars($comment["user_name"]); ?></h3>
        <p><?php echo htmlspecialchars($comment["message"]); ?></p>
        <small><?php echo htmlspecialchars($comment["created_at"]); ?></small>
    </div>
<?php endforeach; ?>

<?php include 'incluides/footer.php'; ?>
