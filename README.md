#Gg
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>桌遊地圖上色器</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      color: #343a40;
    }
    header {
      width: 100%;
      background-color: #495057;
      color: #fff;
      padding: 20px 0;
      text-align: center;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    header h1 {
      margin: 0;
      font-size: 2rem;
      letter-spacing: 1px;
      text-transform: uppercase;
    }
    .controls {
      margin: 20px 0;
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      gap: 20px;
      background-color: #fff;
      padding: 15px 20px;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.05);
    }
    .palette {
      display: flex;
      gap: 8px;
    }
    .color-btn {
      width: 34px;
      height: 34px;
      border: 2px solid #ced4da;
      border-radius: 4px;
      cursor: pointer;
      transition: transform 0.1s, border-color 0.2s;
    }
    .color-btn:hover {
      transform: scale(1.1);
      border-color: #495057;
    }
    .color-btn.selected {
      border-color: #343a40;
      box-shadow: 0 0 5px rgba(0,0,0,0.2);
    }
    button.eraser-btn,
    button.mode-btn,
    button.undo-btn {
      font-size: 14px;
      padding: 6px 12px;
      background-color: #e9ecef;
      color: #495057;
      border: 2px solid #ced4da;
      border-radius: 6px;
      cursor: pointer;
      transition: background-color 0.2s, border-color 0.2s;
    }
    button.eraser-btn:hover,
    button.mode-btn:hover,
    button.undo-btn:hover {
      background-color: #dee2e6;
      border-color: #adb5bd;
    }
    button.eraser-btn.selected,
    button.mode-btn.active {
      background-color: #adb5bd;
      border-color: #495057;
      color: #fff;
    }
    .count-panel {
      display: flex;
      flex-wrap: wrap;
      gap: 15px;
      background-color: #fff;
      padding: 12px 20px;
      border-radius: 8px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.05);
      margin-bottom: 20px;
    }
    .count-panel span {
      font-weight: bold;
      white-space: nowrap;
      display: flex;
      align-items: center;
      gap: 4px;
    }
    .count-panel span::before {
      content: '';
      display: inline-block;
      width: 16px;
      height: 16px;
      background-color: currentColor;
      border: 1px solid #adb5bd;
      border-radius: 3px;
    }
    .map-container {
      position: relative;
      width: 692px;
      height: 692px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 6px 15px rgba(0,0,0,0.1);
      overflow: hidden;
    }
    .cell {
      position: absolute;
      width: 40px;
      height: 40px;
      background-color: #fff;
      border: 1px solid #dee2e6;
      cursor: pointer;
      transition: background-color 0.2s;
      box-sizing: border-box;
    }
    .segment {
      position: absolute;
      background-color: #dee2e6;
      cursor: pointer;
      transition: background-color 0.2s;
      box-sizing: border-box;
    }
    .segment.vertical { width: 4px; }
    .segment.horizontal { height: 4px; }
  </style>
</head>
<body>
  <header>
    <h1>桌遊地圖上色器</h1>
  </header>
  <div class="controls">
    <div class="palette" id="palette">
      <div class="color-btn" data-color="#FF0000" style="background-color: #FF0000;"></div>
      <div class="color-btn" data-color="#00FF00" style="background-color: #00FF00;"></div>
      <div class="color-btn" data-color="#0000FF" style="background-color: #0000FF;"></div>
      <div class="color-btn" data-color="#FFFF00" style="background-color: #FFFF00;"></div>
      <div class="color-btn" data-color="#FF00FF" style="background-color: #FF00FF;"></div>
      <div class="color-btn" data-color="#00FFFF" style="background-color: #00FFFF;"></div>
    </div>
    <button class="eraser-btn" id="eraser">橡皮擦</button>
    <button class="mode-btn" id="toggle-mode">目前：塗格子</button>
    <button class="undo-btn" id="undo-btn">返回上一步</button>
  </div>
  <div class="count-panel" id="count-panel"></div>
  <div class="map-container" id="map"></div>

  <script>
    const colors = ['#FF0000','#00FF00','#0000FF','#FFFF00','#FF00FF','#00FFFF'];
    const colorMap = {
      '#FF0000':'紅色','#00FF00':'綠色','#0000FF':'藍色',
      '#FFFF00':'黃色','#FF00FF':'洋紅','#00FFFF':'青色'
    };
    const palette = document.getElementById('palette');
    const eraserBtn = document.getElementById('eraser');
    const toggleBtn = document.getElementById('toggle-mode');
    const undoBtn = document.getElementById('undo-btn');
    const map = document.getElementById('map');
    const countPanel = document.getElementById('count-panel');
    let selectedColor=null, isEraser=false, isLineMode=false;
    const history=[], cellSize=40, lineSize=4, grid=16;
    const cells=[], segs=[];

    palette.querySelectorAll('.color-btn').forEach(btn=>{
      btn.addEventListener('click', ()=>{
        palette.querySelectorAll('.color-btn').forEach(b=>b.classList.remove('selected'));
        eraserBtn.classList.remove('selected');
        isEraser=false;
        selectedColor=btn.dataset.color;
        btn.classList.add('selected');
      });
    });
    eraserBtn.addEventListener('click', ()=>{
      palette.querySelectorAll('.color-btn').forEach(b=>b.classList.remove('selected'));
      eraserBtn.classList.add('selected');
      isEraser=true;
    });
    toggleBtn.addEventListener('click', ()=>{
      isLineMode=!isLineMode;
      toggleBtn.classList.toggle('active');
      toggleBtn.textContent=isLineMode?'目前：塗格線':'目前：塗格子';
    });

    for(let r=0;r<grid;r++){
      for(let c=0;c<grid;c++){
        const cell=document.createElement('div');
        cell.className='cell';
        cell.style.top=`${r*(cellSize+lineSize)}px`;
        cell.style.left=`${c*(cellSize+lineSize)}px`;
        cell.dataset.type='cell';
        cell.addEventListener('click',()=>paint(cell));
        map.appendChild(cell); cells.push(cell);
      }
    }

    for(let i=1;i<grid;i++){
      const isThick = i % 4 === 0;
      const thickSize = 6;
      const thinSize = 4;

      for(let r=0;r<grid;r++){
        const seg=document.createElement('div');
        seg.className='segment vertical';
        seg.style.height=`${cellSize}px`;
        seg.style.top=`${r*(cellSize+lineSize)}px`;
        seg.style.left=`${i*(cellSize+lineSize)-(isThick?thickSize:lineSize)}px`;
        seg.style.width = `${isThick ? thickSize : lineSize}px`;
        seg.dataset.type='line';
        seg.style.backgroundColor=isThick?'#000000':'#dee2e6';
        seg.addEventListener('click',()=>paint(seg));
        map.appendChild(seg); segs.push(seg);
      }

      for(let c=0;c<grid;c++){
        const seg=document.createElement('div');
        seg.className='segment horizontal';
        seg.style.width=`${cellSize}px`;
        seg.style.left=`${c*(cellSize+lineSize)}px`;
        seg.style.top=`${i*(cellSize+lineSize)-(isThick?thickSize:lineSize)}px`;
        seg.style.height = `${isThick ? thickSize : lineSize}px`;
        seg.dataset.type='line';
        seg.style.backgroundColor=isThick?'#000000':'#dee2e6';
        seg.addEventListener('click',()=>paint(seg));
        map.appendChild(seg); segs.push(seg);
      }
    }

    function paint(el){
      if((isLineMode&&el.dataset.type!=='line')||(!isLineMode&&el.dataset.type!=='cell'))return;
      if(!selectedColor&&!isEraser)return;
      history.push({el,prev:el.style.backgroundColor});
      if(isEraser){
        el.style.backgroundColor = el.dataset.type==='line' ? '#dee2e6' : '#fff';
      } else {
        if(el.dataset.type==='line'){
          el.style.backgroundColor = '#000000'; // 線只能塗黑
        } else {
          el.style.backgroundColor = selectedColor;
        }
      }
      updateCounts();
    }

    undoBtn.addEventListener('click',()=>{
      const last=history.pop();
      if(last){ last.el.style.backgroundColor=last.prev; updateCounts(); }
    });

    function updateCounts(){
      const cnt={}; colors.forEach(c=>cnt[c]=0);
      cells.forEach(el=>{
        const bg=el.style.backgroundColor.toUpperCase();
        if(colors.includes(bg))cnt[bg]++;
      });
      countPanel.innerHTML='';
      colors.forEach(c=>{
        const span=document.createElement('span');
        span.textContent=`${colorMap[c]}：${cnt[c]}`;
        span.style.color=c;
        countPanel.appendChild(span);
      });
    }

    updateCounts();
  </script>
</body>
</html>
