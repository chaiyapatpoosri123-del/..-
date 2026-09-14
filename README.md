<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>เกม WebAR แยกขยะ</title>
  <!-- โหลด A-Frame library สำหรับ AR 3D บนเว็บ -->
  <script src="https://aframe.io/releases/1.4.2/aframe.min.js"></script>
  <style>
    body { margin: 0; overflow: hidden; font-family: sans-serif; }
    #ui-container {
      position: absolute;
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 999;
      text-align: center;
      width: 90%;
      pointer-events: none;
    }
    .score-board {
      background: rgba(0, 0, 0, 0.7);
      color: white;
      padding: 10px 20px;
      border-radius: 20px;
      font-size: 18px;
      font-weight: bold;
      display: inline-block;
      margin-bottom: 10px;
    }
    .bin-buttons {
      display: flex;
      justify-content: center;
      gap: 8px;
      pointer-events: auto;
    }
    .btn-bin {
      padding: 10px 12px;
      border: none;
      border-radius: 8px;
      color: white;
      font-weight: bold;
      font-size: 14px;
      cursor: pointer;
      box-shadow: 0 4px 6px rgba(0,0,0,0.3);
    }
    .btn-organic { background-color: #4CAF50; } /* เขียว */
    .btn-recycle { background-color: #FBC02D; color: black; } /* เหลือง */
    .btn-general { background-color: #2196F3; } /* น้ำเงิน */
    .btn-hazard { background-color: #F44336; } /* แดง */
    
    #message {
      margin-top: 10px;
      font-size: 16px;
      font-weight: bold;
      text-shadow: 0 0 4px black;
    }
  </style>
</head>
<body>

  <!-- หน้าจอ UIOverlay ด้านบน -->
  <div id="ui-container">
    <div class="score-board">คะแนน: <span id="score">0</span></div>
    <div id="message" style="color: yellow;">ทิ้งขยะชิ้นนี้ลงถังไหนดี?</div>
    <div class="bin-buttons">
      <button class="btn-bin btn-organic" onclick="dropTrash('organic')">เขียว (ย่อยสลาย)</button>
      <button class="btn-bin btn-recycle" onclick="dropTrash('recycle')">เหลือง (รีไซเคิล)</button>
      <button class="btn-bin btn-general" onclick="dropTrash('general')">น้ำเงิน (ทั่วไป)</button>
      <button class="btn-bin btn-hazard" onclick="dropTrash('hazard')">แดง (อันตราย)</button>
    </div>
  </div>

  <!-- A-Frame AR Scene -->
  <a-scene webxr="optionalFeatures: hit-test;" embedded arjs="sourceType: webcam; debugUIEnabled: false;">
    
    <!-- แสงไฟในฉาก -->
    <a-entity light="type: ambient; intensity: 1.5;"></a-entity>
    <a-entity light="type: directional; intensity: 1;" position="1 2 1"></a-entity>

    <!-- วัตถุขยะที่จะปรากฏตรงหน้ากล้อง -->
    <a-entity id="trash-object" position="0 0 -1.2">
      <!-- รูปทรงขยะ (3D Primitive) -->
      <a-sphere id="trash-mesh" radius="0.25" color="#FF5722"></a-sphere>
      <!-- ข้อความบอกชื่อขยะ -->
      <a-text id="trash-label" value="ขยะ" align="center" position="0 0.4 0" scale="0.6 0.6 0.6" color="#FFFFFF"></a-text>
    </a-entity>

    <!-- กล้องถ่ายภาพจริง -->
    <a-entity camera></a-entity>
  </a-scene>

  <script>
    // รายการขยะในเกม
    const trashItems = [
      { name: "เปลือกกล้วย", type: "organic", color: "#8BC34A", shape: "sphere" },
      { name: "เศษใบไม้", type: "organic", color: "#4CAF50", shape: "cone" },
      { name: "ขวดน้ำพลาสติก", type: "recycle", color: "#00BCD4", shape: "cylinder" },
      { name: "กระป๋องน้ำอัดลม", type: "recycle", color: "#FFC107", shape: "cylinder" },
      { name: "กล่องโฟม", type: "general", color: "#E0E0E0", shape: "box" },
      { name: "ถุงพลาสติก", type: "general", color: "#9E9E9E", shape: "box" },
      { name: "ถ่านไฟฉาย", type: "hazard", color: "#F44336", shape: "cylinder" },
      { name: "กระป๋องสเปรย์", type: "hazard", color: "#E91E63", shape: "cylinder" }
    ];

    let currentScore = 0;
    let currentTrash = null;

    // ฟังก์ชันสุ่มขยะชิ้นใหม่
    function spawnTrash() {
      const randomIndex = Math.floor(Math.random() * trashItems.length);
      currentTrash = trashItems[randomIndex];

      const trashMesh = document.getElementById('trash-mesh');
      const trashLabel = document.getElementById('trash-label');

      // เปลี่ยนรูปทรง สี และชื่อขยะ
      trashMesh.setAttribute('geometry', { primitive: currentTrash.shape, radius: 0.2, height: 0.4, depth: 0.3, width: 0.3 });
      trashMesh.setAttribute('material', 'color', currentTrash.color);
      trashLabel.setAttribute('value', currentTrash.name);

      // Reset ตำแหน่งขยะให้ลอยตรงหน้า
      const trashObj = document.getElementById('trash-object');
      trashObj.setAttribute('position', '0 0 -1.2');
      trashObj.setAttribute('scale', '1 1 1');
    }

    // ฟังก์ชันเมื่อผู้เล่นกดเลือกทิ้งขยะลงถัง
    function dropTrash(selectedType) {
      if (!currentTrash) return;

      const msgContainer = document.getElementById('message');
      
      if (selectedType === currentTrash.type) {
        currentScore += 10;
        msgContainer.style.color = '#4CAF50';
        msgContainer.innerText = `ถูกต้อง! +10 คะแนน (${currentTrash.name})`;
      } else {
        currentScore = Math.max(0, currentScore - 5);
        msgContainer.style.color = '#F44336';
        msgContainer.innerText = `ผิดถัง! -5 คะแนน (${currentTrash.name} ต้องทิ้งลงถัง ${getTypeName(currentTrash.type)})`;
      }

      document.getElementById('score').innerText = currentScore;

      // สุ่มขยะชิ้นถัดไป
      setTimeout(() => {
        spawnTrash();
      }, 1200);
    }

    function getTypeName(type) {
      switch(type) {
        case 'organic': return 'ขยะอินทรีย์ (สีเขียว)';
        case 'recycle': return 'ขยะรีไซเคิล (สีเหลือง)';
        case 'general': return 'ขยะทั่วไป (สีน้ำเงิน)';
        case 'hazard': return 'ขยะอันตราย (สีแดง)';
      }
    }

    // เริ่มต้นเกมสุ่มขยะชิ้นแรก
    window.onload = () => {
      spawnTrash();
    };
  </script>
</body>
</html>

