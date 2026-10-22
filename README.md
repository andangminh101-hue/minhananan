<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tool Make Code Cache</title>
    <style>
        :root {
            --bg-color: #12141c;
            --card-bg: #1a1d29;
            --text-color: #ffffff;
            --text-secondary: #94a3b8;
            --accent-red: #ef4444;
            --accent-yellow: #eab308;
            --border-color: #2a2e3d;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background-color: var(--bg-color); color: var(--text-color); padding: 16px; }
        .header { display: flex; align-items: center; justify-content: space-between; background: var(--card-bg); padding: 12px 16px; border-radius: 12px; margin-bottom: 16px; border: 1px solid var(--border-color); }
        .header h1 { font-size: 16px; display: flex; align-items: center; gap: 8px; }
        .header h1 span { background: #3b82f6; color: white; padding: 2px 8px; border-radius: 6px; font-size: 14px; }
        
        .preview-box { background: var(--card-bg); border-radius: 12px; padding: 16px; text-align: center; margin-bottom: 16px; border: 1px solid var(--border-color); position: relative; height: 220px; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .preview-title { position: absolute; top: 12px; left: 16px; font-size: 14px; color: var(--text-secondary); }
        
        /* Mô phỏng nhân vật và hitbox */
        .character-container { position: relative; width: 80px; height: 160px; margin-top: 15px; }
        .dummy-head { width: 36px; height: 36px; background: #d1d5db; border-radius: 50%; position: absolute; top: 0; left: 22px; }
        .dummy-body { width: 50px; height: 80px; background: #374151; border-radius: 8px; position: absolute; top: 38px; left: 15px; }
        
        .hitbox-circle { position: absolute; border-radius: 50%; pointer-events: none; transition: all 0.1s ease; }
        #preview-red { border: 2px dashed var(--accent-red); top: -10px; left: 12px; width: 56px; height: 56px; }
        #preview-yellow { border: 2px dashed var(--accent-yellow); top: 30px; left: 5px; width: 70px; height: 90px; }

        .section { background: var(--card-bg); border-radius: 12px; padding: 16px; margin-bottom: 16px; border: 1px solid var(--border-color); }
        .section-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; font-weight: bold; font-size: 14px; }
        .badge-red { color: var(--accent-red); background: rgba(239, 68, 68, 0.1); padding: 2px 8px; border-radius: 4px; font-size: 12px; }
        .badge-yellow { color: var(--accent-yellow); background: rgba(234, 179, 8, 0.1); padding: 2px 8px; border-radius: 4px; font-size: 12px; }
        
        .control-group { margin-bottom: 10px; }
        .control-label { display: flex; justify-content: space-between; font-size: 13px; color: var(--text-secondary); margin-bottom: 4px; }
        .control-label span:last-child { color: var(--text-color); font-family: monospace; }
        
        input[type="range"] { width: 100%; accent-color: #3b82f6; cursor: pointer; }
        
        .code-box { background: #0b0c10; border-radius: 8px; padding: 12px; font-family: monospace; font-size: 12px; color: #34d399; overflow-x: auto; white-space: pre; max-height: 200px; border: 1px solid var(--border-color); margin-bottom: 10px; }
        
        .btn { background: #2563eb; color: white; border: none; padding: 10px 16px; border-radius: 8px; font-weight: bold; width: 100%; cursor: pointer; font-size: 14px; display: flex; align-items: center; justify-content: center; gap: 6px; }
        .btn:active { background: #1d4ed8; }
        .reset-btn { background: transparent; border: 1px solid var(--border-color); color: var(--text-secondary); padding: 2px 8px; border-radius: 4px; font-size: 11px; cursor: pointer; }
    </style>
</head>
<body>

    <div class="header">
        <h1><span>C</span> TOOL MAKE CODE CACHE</h1>
    </div>

    <!-- KHU VỰC PREVIEW -->
    <div class="preview-box">
        <div class="preview-title">👁️ Preview — Đỏ & Vàng riêng</div>
        <div class="character-container">
            <div id="preview-red" class="hitbox-circle"></div>
            <div id="preview-yellow" class="hitbox-circle"></div>
            <div class="dummy-head"></div>
            <div class="dummy-body"></div>
        </div>
    </div>

    <!-- VÒNG ĐỎ - ĐẦU -->
    <div class="section">
        <div class="section-header">
            <span style="color: var(--accent-red)">🔴 VÒNG ĐỎ — ĐẦU</span>
            <button class="reset-btn" onclick="resetGroup('red')">Reset</button>
        </div>
        <div class="control-group">
            <div class="control-label"><span>Đỏ - X</span><span id="val-red-x">0.0000</span></div>
            <input type="range" id="red-x" min="-0.05" max="0.05" step="0.0001" value="0" oninput="updateTool()">
        </div>
        <div class="control-group">
            <div class="control-label"><span>Đỏ - Y</span><span id="val-red-y">0.0000</span></div>
            <input type="range" id="red-y" min="-0.05" max="0.05" step="0.0001" value="0" oninput="updateTool()">
        </div>
        <div class="control-group">
            <div class="control-label"><span>Đỏ - Z</span><span id="val-red-z">0.0000</span></div>
            <input type="range" id="red-z" min="-0.05" max="0.05" step="0.0001" value="0" oninput="updateTool()">
        </div>
        <div class="control-group">
            <div class="control-label"><span>Kích thước (Radius)</span><span id="val-red-size">100%</span></div>
            <input type="range" id="red-size" min="70" max="140" step="1" value="100" oninput="updateTool()">
        </div>
    </div>

    <!-- VÒNG VÀNG - THÂN -->
    <div class="section">
        <div class="section-header">
            <span style="color: var(--accent-yellow)">🟡 VÒNG VÀNG — THÂN</span>
            <button class="reset-btn" onclick="resetGroup('yellow')">Reset</button>
        </div>
        <div class="control-group">
            <div class="control-label"><span>Vàng - X</span><span id="val-yellow-x">0.0000</span></div>
            <input type="range" id="yellow-x" min="-0.05" max="0.05" step="0.0001" value="0" oninput="updateTool()">
        </div>
        <div class="control-group">
            <div class="control-label"><span>Vàng - Y</span><span id="val-yellow-y">0.0000</span></div>
            <input type="range" id="yellow-y" min="-0.05" max="0.05" step="0.0001" value="0" oninput="updateTool()">
        </div>
        <div class="control-group">
            <div class="control-label"><span>Vàng - Z</span><span id="val-yellow-z">0.0000</span></div>
            <input type="range" id="yellow-z" min="-0.05" max="0.05" step="0.0001" value="0" oninput="updateTool()">
        </div>
        <div class="control-group">
            <div class="control-label"><span>Kích thước (Radius)</span><span id="val-yellow-size">100%</span></div>
            <input type="range" id="yellow-size" min="70" max="140" step="1" value="100" oninput="updateTool()">
        </div>
    </div>

    <!-- XUẤT CODE -->
    <div class="section">
        <div class="section-header"><span>💻 CODE — Đỏ & Vàng (Male + Female)</span></div>
        <div class="code-box" id="output-code"></div>
        <button class="btn" onclick="copyCode()">📋 Sao chép tất cả</button>
    </div>

    <script>
        function updateTool() {
            // Lấy giá trị Đỏ
            let rx = parseFloat(document.getElementById('red-x').value);
            let ry = parseFloat(document.getElementById('red-y').value);
            let rz = parseFloat(document.getElementById('red-z').value);
            let rSize = parseInt(document.getElementById('red-size').value);

            document.getElementById('val-red-x').innerText = rx.toFixed(4);
            document.getElementById('val-red-y').innerText = ry.toFixed(4);
            document.getElementById('val-red-z').innerText = rz.toFixed(4);
            document.getElementById('val-red-size').innerText = rSize + '%';

            // Lấy giá trị Vàng
            let yx = parseFloat(document.getElementById('yellow-x').value);
            let yy = parseFloat(document.getElementById('yellow-y').value);
            let yz = parseFloat(document.getElementById('yellow-z').value);
            let ySize = parseInt(document.getElementById('yellow-size').value);

            document.getElementById('val-yellow-x').innerText = yx.toFixed(4);
            document.getElementById('val-yellow-y').innerText = yy.toFixed(4);
            document.getElementById('val-yellow-z').innerText = yz.toFixed(4);
            document.getElementById('val-yellow-size').innerText = ySize + '%';

            // Cập nhật giao diện Preview trực quan bằng CSS transform
            let redBox = document.getElementById('preview-red');
            let yellowBox = document.getElementById('preview-yellow');

            redBox.style.transform = `translate(${rx * 500}px, ${-ry * 500}px) scale(${rSize / 100})`;
            yellowBox.style.transform = `translate(${yx * 500}px, ${-yy * 500}px) scale(${ySize / 100})`;

            // Sinh đoạn Code kết quả mô phỏng cấu trúc cấu hình game
            let generatedText = `// --- VÒNG ĐỎ (Radius: ${rSize}%) ---
{
  "m_GameObject": {"m_FileID": 0, "m_PathID": -363676262939371015},
  "m_Radius": ${(0.25 * (rSize/100)).toFixed(6)},
  "m_Center": {"x": ${rx.toFixed(6)}, "y": ${(0.8178598 + ry).toFixed(6)}, "z": ${rz.toFixed(6)}}
},
// --- VÒNG VÀNG (Radius: ${ySize}%) ---
{
  "m_GameObject": {"m_FileID": 0, "m_PathID": 599865478686802449},
  "m_Radius": ${(0.8991539 * (ySize/100)).toFixed(6)},
  "m_Center": {"x": ${yx.toFixed(6)}, "y": ${(-0.1308626 + yy).toFixed(6)}, "z": ${yz.toFixed(6)}}
}`;

            document.getElementById('output-code').innerText = generatedText;
        }

        function resetGroup(type) {
            if(type === 'red') {
                document.getElementById('red-x').value = 0;
                document.getElementById('red-y').value = 0;
                document.getElementById('red-z').value = 0;
                document.getElementById('red-size').value = 100;
            } else {
                document.getElementById('yellow-x').value = 0;
                document.getElementById('yellow-y').value = 0;
                document.getElementById('yellow-z').value = 0;
                document.getElementById('yellow-size').value = 100;
            }
            updateTool();
        }

        function copyCode() {
            let codeText = document.getElementById('output-code').innerText;
            navigator.clipboard.writeText(codeText).then(() => {
                alert('Đã sao chép toàn bộ mã thành công!');
            });
        }

        // Khởi chạy lần đầu
        updateTool();
    </script>
</body>
</html>
