<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ピッキング照合（高速版）</title>
    <style>
        body { font-family: sans-serif; text-align: center; background: #f0f2f5; padding: 20px; }
        .container { background: white; border-radius: 15px; padding: 20px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); max-width: 400px; margin: 0 auto; transition: 0.2s; }
        h1 { color: #1a73e8; font-size: 1.5rem; }
        #status { font-weight: bold; margin: 15px 0; min-height: 1.2em; color: #5f6368; }
        
        .mode-selector { display: flex; justify-content: center; gap: 10px; margin-bottom: 20px; }
        .mode-btn { padding: 8px 15px; border: 2px solid #1a73e8; border-radius: 20px; background: white; color: #1a73e8; cursor: pointer; font-weight: bold; flex: 1; outline: none; }
        .mode-btn.active { background: #1a73e8; color: white; }

        .input-group { margin: 15px 0; text-align: left; }
        label { display: block; font-size: 0.9rem; color: #5f6368; margin-bottom: 5px; }
        input { width: 100%; padding: 12px; font-size: 1.5rem; border: 2px solid #dadce0; border-radius: 8px; box-sizing: border-box; outline: none; text-transform: uppercase; }
        input:focus { border-color: #1a73e8; background: #fff; }
        input:disabled { background: #f8f9fa; }
        
        .flash-ok { background-color: #e6f4ea !important; }
        .flash-ng { background-color: #fce8e6 !important; }
        .result-area { font-size: 2rem; margin: 20px 0; min-height: 2.5rem; }
        .ok { color: #188038; font-weight: bold; }
        .ng { color: #d93025; font-weight: bold; }
        .btn-reset { background: #5f6368; color: white; border: none; padding: 10px; border-radius: 5px; cursor: pointer; width: 100%; margin-top: 10px; }
    </style>
</head>
<body>

<div class="container" id="main-container">
    <h1>ピッキング照合</h1>
    
    <div class="mode-selector">
        <button type="button" id="mode-num" class="mode-btn active">数字優先</button>
        <button type="button" id="mode-alpha" class="mode-btn">英数字</button>
    </div>

    <div id="status">リスト品番を入力してください</div>
    
    <div class="input-group">
        <label>1. リスト品番</label>
        <input type="text" id="list-input" placeholder="入力してEnter" inputmode="text">
    </div>

    <div class="input-group">
        <label>2. 現物品番</label>
        <input type="text" id="actual-input" placeholder="入力してEnter" inputmode="text" disabled>
    </div>

    <div id="result-msg" class="result-area"></div>
    
    <button type="button" class="btn-reset" id="reset-all-btn">全リセット</button>
</div>

<script>
    const container = document.getElementById('main-container');
    const listInput = document.getElementById('list-input');
    const actualInput = document.getElementById('actual-input');
    const resultMsg = document.getElementById('result-msg');
    const status = document.getElementById('status');
    const btnNum = document.getElementById('mode-num');
    const btnAlpha = document.getElementById('mode-alpha');
    const btnReset = document.getElementById('reset-all-btn');

    // 読み上げ関数の改善（反応を速くするためにキャンセルを入れる）
    function speak(text) {
        window.speechSynthesis.cancel(); 
        const uttr = new SpeechSynthesisUtterance(text);
        uttr.lang = 'ja-JP';
        uttr.rate = 1.2; // 読み上げ速度も少しアップ
        window.speechSynthesis.speak(uttr);
    }

    function fullReset() {
        listInput.value = "";
        actualInput.value = "";
        actualInput.disabled = true;
        resultMsg.innerText = "";
        container.classList.remove('flash-ok', 'flash-ng');
        status.innerText = "リスト品番を入力してください";
        listInput.focus();
    }

    // 入力モード切替（inputmodeのヒントを変更）
    function setInputMode(mode) {
        if (mode === 'numeric') {
            listInput.inputmode = "numeric";
            actualInput.inputmode = "numeric";
            btnNum.classList.add('active');
            btnAlpha.classList.remove('active');
        } else {
            listInput.inputmode = "text";
            actualInput.inputmode = "text";
            btnNum.classList.remove('active');
            btnAlpha.classList.add('active');
        }
        fullReset();
    }

    btnNum.addEventListener('click', () => setInputMode('numeric'));
    btnAlpha.addEventListener('click', () => setInputMode('text'));
    btnReset.addEventListener('click', fullReset);

    listInput.addEventListener('keypress', (e) => {
        if (e.key === 'Enter' && listInput.value) {
            actualInput.disabled = false;
            actualInput.focus();
            status.innerText = "現物の品番を入力してください";
        }
    });

    actualInput.addEventListener('keypress', (e) => {
        if (e.key === 'Enter' && actualInput.value) {
            const listVal = listInput.value.trim().toUpperCase();
            const actualVal = actualInput.value.trim().toUpperCase();

            if (listVal === actualVal) {
                resultMsg.innerHTML = '<span class="ok">OKです</span>';
                container.classList.add('flash-ok');
                speak("OK");
                // 待ち時間を1000ms -> 400msに短縮
                setTimeout(fullReset, 400);
            } else {
                resultMsg.innerHTML = '<span class="ng">不一致！</span>';
                container.classList.add('flash-ng');
                speak("やり直し");
                // 待ち時間を1500ms -> 800msに短縮
                setTimeout(() => {
                    actualInput.value = "";
                    actualInput.focus();
                    resultMsg.innerText = "";
                    container.classList.remove('flash-ng');
                    status.innerText = "現物品番を正しく入力してください";
                }, 800);
            }
        }
    });

    window.onload = () => listInput.focus();
</script>

</body>
</html>
