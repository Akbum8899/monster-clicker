<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Đánh Quái Vật</title>
  <style>
    body {
      font-family: 'Arial', sans-serif;
      text-align: center;
      background-color: #f5f5f5;
    }
    h1 {
      color: #333;
    }
    .monster-container {
      position: relative;
      display: inline-block;
      margin-top: 30px;
    }
    .monster {
      width: 200px;
      cursor: pointer;
      transition: transform 0.1s;
    }
    .monster.hit {
      transform: scale(1.1) rotate(5deg);
    }
    .weapon {
      width: 50px;
      position: absolute;
      top: 20px;
      left: 10px;
    }
    .health-bar {
      width: 200px;
      height: 20px;
      background-color: #ddd;
      border: 1px solid #999;
      margin-top: 10px;
    }
    .health-fill {
      height: 100%;
      background-color: #e74c3c;
      width: 100%;
    }
    .score, .gold, .level, .upgrade, .leaderboard, .shop {
      margin-top: 15px;
      font-size: 20px;
    }
    .upgrade button, .leaderboard button, .shop button {
      padding: 10px 20px;
      margin-top: 10px;
      font-size: 16px;
      cursor: pointer;
    }
    .leaderboard-list {
      margin-top: 10px;
      text-align: left;
      display: inline-block;
    }
    input[type="text"] {
      padding: 5px;
      font-size: 16px;
      margin-top: 10px;
    }
  </style>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
</head>
<body>
  <h1>🧟 Game Đánh Quái Vật</h1>
  <div>
    Tên người chơi: <input type="text" id="playerName" placeholder="Nhập tên của bạn" />
  </div>
  <div class="monster-container">
    <img src="https://i.imgur.com/3yN3JNe.png" alt="Monster" class="monster" id="monster" onclick="attackMonster()" />
    <img src="https://i.imgur.com/Vu7I61P.png" id="weaponImg" class="weapon" alt="Vũ khí" />
    <div class="health-bar">
      <div class="health-fill" id="healthFill"></div>
    </div>
  </div>
  <div class="score">🪓 Điểm: <span id="score">0</span></div>
  <div class="gold">💰 Vàng: <span id="gold">0</span></div>
  <div class="level">📈 Cấp độ quái: <span id="level">1</span></div>
  <div class="upgrade">
    <button onclick="upgradeDamage()">Tăng sát thương (+10 vàng) | Hiện tại: <span id="damage">10</span></button>
  </div>
  <div class="shop">
    <h2>🛒 Cửa Hàng</h2>
    <button onclick="buySword(1)">🪵 Mua Kiếm Gỗ (+2 dmg - 15 vàng)</button><br>
    <button onclick="buySword(2)">⚔️ Mua Kiếm Sắt (+5 dmg - 40 vàng)</button><br>
    <button onclick="buySword(3)">🗡️ Mua Kiếm Vàng (+10 dmg - 100 vàng)</button>
  </div>
  <div class="leaderboard">
    <h2>🏆 Bảng Xếp Hạng</h2>
    <button onclick="loadLeaderboard()">🔄 Làm mới BXH</button>
    <div class="leaderboard-list" id="leaderboardList"></div>
  </div>

  <script>
    const firebaseConfig = {
      apiKey: "AIzaSyCxPhLmb06EMMN1XLfTUK3GEftz3gkPNEg",
      authDomain: "monster-clicker-demo.firebaseapp.com",
      databaseURL: "https://monster-clicker-demo-default-rtdb.asia-southeast1.firebasedatabase.app",
      projectId: "monster-clicker-demo",
      storageBucket: "monster-clicker-demo.appspot.com",
      messagingSenderId: "629246925375",
      appId: "1:629246925375:web:a14e22fe5f1ccf42978b34"
    };
    firebase.initializeApp(firebaseConfig);
    const database = firebase.database();

    let level = 1;
    let baseHealth = 100;
    let currentHealth = baseHealth;
    let score = 0;
    let gold = 0;
    let damage = 10;
    let currentWeapon = "wood";

    const monsterImages = [
      "https://i.imgur.com/3yN3JNe.png",
      "/mnt/data/476734573_1152862999645332_576150975096542255_n.jpg",
      "/mnt/data/483446922_1162889638304599_1139742185810240445_n.jpg",
      "/mnt/data/480442164_7553961351394695_5025492805514046743_n.jpg"
    ];

    const monster = document.getElementById("monster");
    const healthFill = document.getElementById("healthFill");
    const scoreSpan = document.getElementById("score");
    const goldSpan = document.getElementById("gold");
    const levelSpan = document.getElementById("level");
    const damageSpan = document.getElementById("damage");
    const weaponImg = document.getElementById("weaponImg");

    function attackMonster() {
      currentHealth -= damage;
      monster.classList.add("hit");
      setTimeout(() => monster.classList.remove("hit"), 100);

      if (currentHealth <= 0) {
        score++;
        gold += 5;
        level++;
        baseHealth += 50;
        currentHealth = baseHealth;

        // Boss mỗi 20 cấp
        if (level % 20 === 0) {
          monster.src = "/mnt/data/487568516_2118057265324310_6608022614889603934_n.jpg";
        } else {
          const randomIndex = Math.floor(Math.random() * monsterImages.length);
          monster.src = monsterImages[randomIndex];
        }

        updateInfo();
        saveScore();
      }
      updateHealthBar();
    }

    function updateHealthBar() {
      const percent = (currentHealth / baseHealth) * 100;
      healthFill.style.width = percent + "%";
    }

    function updateInfo() {
      scoreSpan.innerText = score;
      goldSpan.innerText = gold;
      levelSpan.innerText = level;
    }

    function upgradeDamage() {
      if (gold >= 10) {
        gold -= 10;
        damage += 5;
        damageSpan.innerText = damage;
        goldSpan.innerText = gold;
      } else {
        alert("Không đủ vàng để nâng cấp!");
      }
    }

    function buySword(type) {
      let cost = 0, bonus = 0, image = "";
      if (type === 1) { cost = 15; bonus = 2; image = "https://i.imgur.com/Vu7I61P.png"; currentWeapon = "wood"; }
      if (type === 2) { cost = 40; bonus = 5; image = "https://i.imgur.com/oEOYxEx.png"; currentWeapon = "iron"; }
      if (type === 3) { cost = 100; bonus = 10; image = "https://i.imgur.com/B4aLid2.png"; currentWeapon = "gold"; }

      if (gold >= cost) {
        gold -= cost;
        damage += bonus;
        damageSpan.innerText = damage;
        goldSpan.innerText = gold;
        weaponImg.src = image;
        alert(`Đã mua kiếm mới! +${bonus} sát thương`);
      } else {
        alert("Không đủ vàng để mua!");
      }
    }

    function saveScore() {
      const name = document.getElementById("playerName").value || "Người chơi ẩn danh";
      firebase.database().ref("leaderboard/" + name).set({
        name: name,
        score: score
      });
    }

    function loadLeaderboard() {
      const listDiv = document.getElementById("leaderboardList");
      listDiv.innerHTML = "<em>Đang tải...</em>";
      firebase.database().ref("leaderboard").orderByChild("score").limitToLast(5).once("value", snapshot => {
        const data = [];
        snapshot.forEach(child => data.push(child.val()));
        data.reverse();

        let html = '<ol>';
        data.forEach(player => {
          html += `<li><strong>${player.name}</strong>: ${player.score} điểm</li>`;
        });
        html += '</ol>';
        listDiv.innerHTML = html;
      });
    }

    updateHealthBar();
    updateInfo();
    loadLeaderboard();
  </script>
</body>
</html>
