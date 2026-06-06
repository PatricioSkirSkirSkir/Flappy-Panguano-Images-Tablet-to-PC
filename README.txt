<script>
const player = document.getElementById("player");
const album = document.getElementById("album");
const progressBar = document.getElementById("progressBar");
const currentTimeText = document.getElementById("currentTime");
const durationTimeText = document.getElementById("durationTime");

let wasPlaying = false;
let isSeeking = false;

function playMusic(){
    player.play();
    album.classList.add("spinning");
}

function pauseMusic(){
    player.pause();
    album.classList.remove("spinning");
}

function formatTime(seconds){
    if (isNaN(seconds)) return "0:00";

    const min = Math.floor(seconds / 60);
    const sec = Math.floor(seconds % 60).toString().padStart(2, "0");

    return `${min}:${sec}`;
}

function setupDuration(){
    if(player.duration){
        durationTimeText.textContent = formatTime(player.duration);
    }
}

function startSeeking(){
    isSeeking = true;
    wasPlaying = !player.paused;
    player.pause();
    album.classList.remove("spinning");
}

function finishSeeking(){
    const percent = Number(progressBar.value);

    if(player.duration){
        player.currentTime = (percent / 100) * player.duration;
    }

    isSeeking = false;

    if(wasPlaying){
        player.play();
        album.classList.add("spinning");
    }
}

player.addEventListener("loadedmetadata", setupDuration);

progressBar.addEventListener("pointerdown", startSeeking);
progressBar.addEventListener("touchstart", startSeeking);

progressBar.addEventListener("input", () => {
    if(player.duration){
        const percent = Number(progressBar.value);
        const previewTime = (percent / 100) * player.duration;
        currentTimeText.textContent = formatTime(previewTime);
    }
});

progressBar.addEventListener("change", finishSeeking);
progressBar.addEventListener("pointerup", finishSeeking);
progressBar.addEventListener("touchend", finishSeeking);

player.addEventListener("timeupdate", () => {
    if(!isSeeking && player.duration){
        progressBar.value = (player.currentTime / player.duration) * 100;
        currentTimeText.textContent = formatTime(player.currentTime);
    }
});

player.addEventListener("ended", () => {
    album.classList.remove("spinning");
    progressBar.value = 0;
    currentTimeText.textContent = "0:00";
});

if(player.readyState >= 1){
    setupDuration();
}
</script>
