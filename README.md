
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1">
<title>MINH AN CACHE MAKE</title>
<style>
:root{color-scheme:dark}
*{box-sizing:border-box}
body{margin:0;padding:18px;background:#0b0d12;color:#eee;
font:15px -apple-system,BlinkMacSystemFont,sans-serif}
.app{max-width:620px;margin:auto}
h1{font-size:25px;color:#f1d56b;margin:8px 0}
.sub{color:#9299a9;font-size:12px;letter-spacing:1px}
.panel{background:#151923;border:1px solid #2b3040;
border-radius:15px;padding:16px;margin:14px 0}
h2{font-size:16px;margin:0 0 12px}
button,.pick{display:block;width:100%;padding:14px;
border:0;border-radius:10px;background:#e6cc63;color:#151515;
font-size:15px;font-weight:700;text-align:center;margin-top:10px}
button:disabled{opacity:.4}
input[type=file]{width:100%;font-size:13px}
input[type=text]{width:100%;padding:12px;background:#0b0d12;
border:1px solid #343b4c;border-radius:9px;color:white}
.muted{color:#9aa3b5;font-size:13px;line-height:1.55}
.result{white-space:pre-wrap;overflow-wrap:anywhere;
background:#0b0d12;border-radius:10px;padding:12px;
font:12px ui-monospace,monospace;max-height:330px;overflow:auto}
.status{color:#e6cc63;font-weight:600}
</style>
</head>
<body>
<div class="app">
  <h1>MINH AN CACHE MAKE</h1>
  <div class="sub">BONE_HEAD SCANNER · IPHONE EDITION</div>

  <section class="panel">
    <h2>01 · Chọn cache gốc</h2>
    <input id="file" type="file">
    <p class="muted">Trong cửa sổ chọn tệp của iPhone, chọn file đã sao lưu
    từ Filza. Tool chỉ đọc file được chọn.</p>
    <div id="fileinfo" class="muted">Chưa chọn file.</div>
  </section>

  <section class="panel">
    <h2>02 · Thiết lập kiểm tra</h2>
    <label for="needle">Chuỗi cần tìm</label>
    <input id="needle" type="text" value="bone_Head"
           autocomplete="off" spellcheck="false">
    <button id="scan" disabled>KIỂM TRA CACHE</button>
  </section>

  <section class="panel">
    <h2>03 · Kết quả kiểm tra</h2>
    <div id="status" class="status">Đang chờ file.</div>
    <pre id="result" class="result">Chưa có kết quả.</pre>
    <button id="export" disabled></button>
  </section>

  <section class="panel">
    <h2>04 · Lưu ý</h2>
    <p class="muted">
      Tìm thấy bone_Head chỉ xác nhận chuỗi xuất hiện trong dữ liệu có thể
      đọc được. Kết quả này không xác nhận đã tìm thấy collider hợp lệ.
      Cache nén hoặc tuần tự hóa có thể cần bộ phân tích Unity phù hợp.
      
    </p>
  </section>
</div>

<script>
(() => {
  const $ = id => document.getElementById(id);
  let selectedFile = null;
  let report = "";

  const formatSize = n => {
    if (n < 1024) return n + " byte";
    if (n < 1048576) return (n / 1024).toFixed(2) + " KB";
    return (n / 1048576).toFixed(2) + " MB";
  };

  $("file").addEventListener("change", () => {
    selectedFile = $("file").files[0] || null;
    $("scan").disabled = !selectedFile;
    $("export").disabled = true;
    report = "";

    if (!selectedFile) {
      $("fileinfo").textContent = "Chưa chọn file.";
      return;
    }

    $("fileinfo").textContent =
      "Tên: " + selectedFile.name +
      "\nDung lượng: " + formatSize(selectedFile.size) +
      "\nLoại MIME: " + (selectedFile.type || "Không xác định");

    $("status").textContent = "Đã chọn file. Sẵn sàng kiểm tra.";
    $("result").textContent = "Nhấn KIỂM TRA CACHE để bắt đầu.";
  });

  $("scan").addEventListener("click", async () => {
    if (!selectedFile) return;

    const needle = $("needle").value;
    if (!needle) {
      $("status").textContent = "Lỗi: hãy nhập chuỗi cần tìm.";
      return;
    }

    $("scan").disabled = true;
    $("export").disabled = true;
    $("status").textContent = "Đang đọc và quét file...";
    $("result").textContent = "";

    try {
      const buffer = await selectedFile.arrayBuffer();
      const bytes = new Uint8Array(buffer);

      // Nhận diện một số dấu hiệu phổ biến, không giải nén file.
      const head = Array.from(bytes.slice(0, 16))
        .map(b => b.toString(16).padStart(2, "0")).join(" ");
      const ascii = Array.from(bytes.slice(0, 16))
        .map(b => b >= 32 && b <= 126 ? String.fromCharCode(b) : ".")
        .join("");

      let type = "Chưa xác định";
      if (ascii.startsWith("UnityFS")) type = "UnityFS AssetBundle";
      else if (ascii.startsWith("UnityWeb")) type = "UnityWeb (dấu hiệu đầu file)";
      else if (ascii.startsWith("UnityRaw")) type = "UnityRaw (dấu hiệu đầu file)";
      else if (bytes[0] === 0x50 && bytes[1] === 0x4b)
        type = "ZIP hoặc định dạng dựa trên ZIP";
      else if (bytes[0] === 0x7b || bytes[0] === 0x5b)
        type = "Có thể là JSON/văn bản";

      // Tìm chuỗi ASCII trong byte của file.
      const target = new TextEncoder().encode(needle);
      const offsets = [];
      let nextAllowed = 0;

      for (let i = 0; i <= bytes.length - target.length; i++) {
        let match = true;
        for (let j = 0; j < target.length; j++) {
          if (bytes[i + j] !== target[j]) {
            match = false;
            break;
          }
        }
        if (match) {
          offsets.push(i);
          i += target.length - 1;
        }
      }

      const lines = [];
      lines.push("MINH AN CACHE MAKE — BÁO CÁO KIỂM TRA");
      lines.push("File: " + selectedFile.name);
      lines.push("Dung lượng: " + selectedFile.size + " byte");
      lines.push("Dấu hiệu định dạng: " + type);
      lines.push("16 byte đầu (hex): " + head);
      lines.push("Chuỗi tìm kiếm: " + needle);
      lines.push("Số vị trí khớp ASCII: " + offsets.length);
      lines.push("");
      lines.push("CÁC VỊ TRÍ VÀ DỮ LIỆU LÂN CẬN:");

      if (offsets.length === 0) {
        lines.push("Không tìm thấy chuỗi ASCII trong dữ liệu quét.");
        lines.push("Lưu ý: chuỗi có thể bị nén, mã hóa hoặc lưu dạng khác.");
      } else {
        for (const offset of offsets.slice(0, 200)) {
          const start = Math.max(0, offset - 16);
          const end = Math.min(bytes.length, offset + target.length + 24);
          const context = Array.from(bytes.slice(start, end))
            .map(b => b.toString(16).padStart(2, "0")).join(" ");
          lines.push("Offset 0x" + offset.toString(16).toUpperCase() +
                     " (" + offset + ")");
          lines.push("  " + context);
        }
        if (offsets.length > 200)
          lines.push("Chỉ hiển thị 200 vị trí đầu tiên.");
      }

      lines.push("");
      lines.push("GIỚI HẠN:");
      lines.push("- Đây là quét chuỗi byte, không phải giải mã Unity.");
      lines.push("- Không xác nhận collider hay giá trị m_Center.");
      lines.push("- Không sửa file và không ghi đè game gốc.");

      report = lines.join("\n");
      $("result").textContent = report;
      $("status").textContent =
        "Hoàn tất. Tìm thấy " + offsets.length + " vị trí khớp.";
      $("export").disabled = false;
    } catch (err) {
      $("status").textContent = "Không đọc được file.";
      $("result").textContent = String(err);
    } finally {
      $("scan").disabled = !selectedFile;
    }
  });

  $("export").addEventListener("click", () => {
    if (!report) return;
    const blob = new Blob([report], {type: "text/plain;charset=utf-8"});
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = "MINH_AN_CACHE_REPORT.txt";
    document.body.appendChild(a);
    a.click();
    a.remove();
    setTimeout(() => URL.revokeObjectURL(url), 1500);
  });
})();
</script>
</body>
</html>
