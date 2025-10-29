# DuckYou
DuckYou
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DuckYou | Developer Store</title>
<style>
  * { box-sizing: border-box; margin:0; padding:0; }
  body { font-family: 'Inter', sans-serif; background:#000; color:#fff; min-height:100vh; }
  .fade { opacity:0; transform: translateY(20px); transition: all 0.5s ease; }
  .fade.active { opacity:1; transform: translateY(0); }

  /* LOGIN */
  #login { display:flex; flex-direction:column; align-items:center; gap:10px; width:300px; margin:auto; padding-top:50px; }
  #login input { width:100%; padding:10px; border-radius:6px; border:none; background:#1a1a1a; color:#fff; text-align:center; }
  #login button { width:100%; padding:10px; border:none; border-radius:6px; background:#fff; color:#000; font-weight:600; cursor:pointer; }
  #login button:hover { opacity:0.8; }
  #login p { font-size:13px; color:#aaa; margin-top:10px; cursor:pointer; }

  /* SHOP */
  #shop { display:none; flex-direction:column; align-items:center; width:90%; max-width:1000px; padding-top:20px; }
  #shop-header { width:100%; display:flex; justify-content:flex-end; align-items:center; margin-bottom:20px; }

  /* 더보기 버튼 */
  #more { 
    position:relative; 
    cursor:pointer; 
    font-size:28px;
    user-select:none;
  }

  /* 더보기 메뉴 */
  #more-content { 
    position:absolute; 
    top:55px; 
    right:0; 
    background:#111; 
    padding:15px; 
    border-radius:8px; 
    display:none; 
    width:260px;
    opacity:0;
    transform:translateY(10px);
    transition:all 0.4s ease;
  }
  #more-content.show {
    display:block;
    opacity:1;
    transform:translateY(0);
  }
  #more-content h4 { margin-bottom:8px; }
  #more-content ul { list-style:none; max-height:200px; overflow-y:auto; }
  #more-content ul li { padding:5px 0; border-bottom:1px solid #222; font-size:14px; }

  .grid { display:flex; flex-wrap:wrap; gap:20px; justify-content:center; }
  .card { background:#0d0d0d; border-radius:12px; width:250px; height:250px; display:flex; flex-direction:column; justify-content:flex-end; align-items:center; padding:15px; cursor:pointer; transition:all 0.3s ease; }
  .card:hover { transform:scale(1.05); background:#1a1a1a; }
  .thumb { flex:1; width:100%; background:#1a1a1a; border-radius:8px; margin-bottom:10px; }
  .card h2 { font-size:16px; font-weight:600; margin-bottom:5px; }
  .card p { font-size:13px; color:#999; margin:0; }

  /* Toast */
  #toast { position:fixed; bottom:30px; left:50%; transform:translateX(-50%) translateY(50px); background:#333; padding:15px 25px; border-radius:10px; color:#fff; opacity:0; pointer-events:none; transition:all 0.5s ease; }
  #toast.show { opacity:1; transform:translateX(-50%) translateY(0); }
</style>
</head>
<body>

<!-- LOGIN -->
<div id="login" class="fade active">
  <h1 id="login-title">DuckYou Login</h1>
  <input type="text" id="login-username" placeholder="Username">
  <input type="password" id="login-password" placeholder="Password">
  <button onclick="handleAuth()">Sign In</button>
  <p id="toggle-auth" onclick="toggleAuthMode()">Don't have an account? <u>Sign Up</u></p>
</div>

<!-- SHOP -->
<div id="shop" class="fade">
  <div id="shop-header">
    <div id="more">⋮
      <div id="more-content">
        <h4>Downloaded Items</h4>
        <ul id="download-history"></ul>
      </div>
    </div>
  </div>

  <h1>DuckYou Developer Store</h1>
  <p></p>

  <div class="grid">
    <div class="card" onclick="downloadItem('Roblox Studio Run System', 'RUNSYSTEM.rbxl')">
      <div class="thumb"></div>
      <h2>Roblox Studio Run System</h2>
      <p>This is really high quality and FREE</p>
    </div>

    <div class="card" onclick="downloadItem('Roblox Studio Gun Model', 'GUNMODEL.rbxl')">
      <div class="thumb"></div>
      <h2>Roblox Studio Gun Model</h2>
      <p>This is really high quality, 5 models included, FREE</p>
    </div>
  </div>
</div>

<div id="toast"></div>

<script>
  const loginDiv = document.getElementById('login');
  const shopDiv = document.getElementById('shop');
  const toast = document.getElementById('toast');
  const more = document.getElementById('more');
  const moreContent = document.getElementById('more-content');
  const historyList = document.getElementById('download-history');
  const toggleAuthText = document.getElementById('toggle-auth');
  const loginTitle = document.getElementById('login-title');

  let isSignUp = false;

  function showToast(message) {
    toast.textContent = message;
    toast.classList.add('show');
    setTimeout(()=>toast.classList.remove('show'),2000);
  }

  function toggleAuthMode() {
    isSignUp = !isSignUp;
    if (isSignUp) {
      loginTitle.textContent = "DuckYou Sign Up";
      document.querySelector("#login button").textContent = "Sign Up";
      toggleAuthText.innerHTML = "Already have an account? <u>Sign In</u>";
    } else {
      loginTitle.textContent = "DuckYou Login";
      document.querySelector("#login button").textContent = "Sign In";
      toggleAuthText.innerHTML = "Don't have an account? <u>Sign Up</u>";
    }
  }

  function handleAuth() {
    const user = document.getElementById('login-username').value.trim();
    const pass = document.getElementById('login-password').value.trim();
    if (!user || !pass) {
      showToast('Please fill in all fields.');
      return;
    }

    let accounts = JSON.parse(localStorage.getItem('duckyouAccounts') || '{}');

    if (isSignUp) {
      if (accounts[user]) {
        showToast('Username already exists.');
      } else {
        accounts[user] = pass;
        localStorage.setItem('duckyouAccounts', JSON.stringify(accounts));
        showToast('Account created successfully!');
        toggleAuthMode(); // 로그인 화면으로 전환
      }
    } else {
      if (accounts[user] && accounts[user] === pass) {
        loginSuccess();
      } else {
        showToast('Invalid username or password.');
      }
    }
  }

  function loginSuccess() {
    loginDiv.classList.remove('active');
    setTimeout(()=>{
      loginDiv.style.display='none';
      shopDiv.style.display='flex';
      loadHistory();
      setTimeout(()=>shopDiv.classList.add('active'),50);
    },500);
  }

  function downloadItem(name, file){
    showToast('Downloading ' + name + '...');
    let history = JSON.parse(localStorage.getItem('duckyouHistory')||'[]');
    history.push(name);
    localStorage.setItem('duckyouHistory', JSON.stringify(history));
    loadHistory();
    setTimeout(()=>{
      const a = document.createElement('a');
      a.href = file;
      a.download = file;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
    },1000);
  }

  function loadHistory(){
    const history = JSON.parse(localStorage.getItem('duckyouHistory')||'[]');
    historyList.innerHTML = '';
    history.forEach(item=>{
      const li = document.createElement('li');
      li.textContent = item;
      historyList.appendChild(li);
    });
  }

  // 더보기 클릭 시 페이드인/아웃
  more.addEventListener('click', ()=>{
    if(moreContent.classList.contains('show')){
      moreContent.classList.remove('show');
      setTimeout(()=>moreContent.style.display='none',400);
    } else {
      moreContent.style.display='block';
      setTimeout(()=>moreContent.classList.add('show'),10);
    }
  });
</script>

</body>
</html>
