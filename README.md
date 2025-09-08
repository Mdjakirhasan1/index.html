<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Tap Game Online Leaderboard</title>
<style>
  body {
    margin:0; padding:0;
    font-family: Arial;
    background: linear-gradient(135deg,#ff9a9e,#fad0c4,#a18cd1);
    display:flex; flex-direction:column;
    justify-content:center; align-items:center;
    height:100vh; text-align:center;
  }
  h1{color:#fff;margin-bottom:10px;}
  #tapButton{padding:25px 60px;font-size:30px;background:#ff6f61;color:white;border:none;border-radius:20px;cursor:pointer;box-shadow:0 5px #c0392b;transition:all 0.1s;}
  #tapButton:active{transform:translateY(5px);box-shadow:0 2px #c0392b;}
  #score,#timer,#level,#highscore,#leaderboard{margin-top:10px;color:#fff;font-size:22px;}
</style>
</head>
<body>

<h1>🔥 Tap Game Leaderboard 🔥</h1>
<button id="tapButton">Tap Me!</button>
<div id="score">Score: 0</div>
<div id="timer">Time Left: 10s</div>
<div id="level">Level: 1</div>
<div id="highscore">Highscore: 0</div>
<div id="leaderboard">Loading Leaderboard...</div>

<audio id="clickSound" src="https://www.soundjay.com/button/beep-07.mp3"></audio>

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>

<script>
  // ===== Firebase Config =====
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT.firebaseio.com",
    projectId: "YOUR_PROJECT",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };
  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  // ===== Game Variables =====
  let score=0, timeLeft=10, level=1, maxLevels=3, gameOver=false;
  const scoreDisplay=document.getElementById('score');
  const timerDisplay=document.getElementById('timer');
  const levelDisplay=document.getElementById('level');
  const highscoreDisplay=document.getElementById('highscore');
  const leaderboardDisplay=document.getElementById('leaderboard');
  const button=document.getElementById('tapButton');
  const clickSound=document.getElementById('clickSound');

  // Local Highscore
  let highscore = localStorage.getItem('tapGameHighscore') || 0;
  highscoreDisplay.textContent = "Highscore: "+highscore;

  function updateLeaderboard(){
    db.ref('leaderboard').orderByChild('score').limitToLast(5).once('value', snapshot=>{
      let list='';
      const data = [];
      snapshot.forEach(snap=>data.push({name:snap.key,score:snap.val().score}));
      data.reverse().forEach((d,i)=>{
        list+=`${i+1}. ${d.name}: ${d.score}  `;
      });
      leaderboardDisplay.textContent = "Leaderboard: "+list;
    });
  }

  function postScore(username, score){
    db.ref('leaderboard/'+username).set({score: score});
    updateLeaderboard();
  }

  function startLevel(levelTime){
    timeLeft=levelTime;
    timerDisplay.textContent="Time Left: "+timeLeft+"s";
    gameOver=false;
    button.disabled=false;

    const timer=setInterval(()=>{
      if(timeLeft>0){ timeLeft--; timerDisplay.textContent="Time Left: "+timeLeft+"s"; }
      else{
        clearInterval(timer);
        gameOver=true;
        button.disabled=true;
        // Update Highscore
        if(score>highscore){
          highscore=score;
          localStorage.setItem('tapGameHighscore',highscore);
          highscoreDisplay.textContent="Highscore: "+highscore;
        }
        // Prompt user for name
        const username = prompt("Enter your name for Leaderboard:","Player");
        if(username) postScore(username, score);

        if(level<maxLevels){
          alert(`⏱️ Level ${level} over! Score: ${score}\nNext Level Unlocked!`);
          level++;
          levelDisplay.textContent="Level: "+level;
          startLevel(10-(level-1)*2);
        } else {
          alert(`🏆 Game Over! Final Score: ${score}`);
        }
      }
    },1000);
  }

  button.addEventListener('click', ()=>{
    if(!gameOver){
      score++;
      scoreDisplay.textContent="Score: "+score;
      clickSound.currentTime=0; clickSound.play();
      button.style.background=`hsl(${Math.random()*360},70%,60%)`;
    }
  });

  // Start first level
  startLevel(10);
  updateLeaderboard();
</script>

</body>
</html>
