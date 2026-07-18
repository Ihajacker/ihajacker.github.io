---
title: 音乐墙生成器 (Topsters)
date: 2026-07-18 15:30:00
type: "music-grid"
comments: false      
---

<link href="https://cdn.bootcdn.net/ajax/libs/cropperjs/1.5.13/cropper.min.css" rel="stylesheet">
<script src="https://cdn.bootcdn.net/ajax/libs/cropperjs/1.5.13/cropper.min.js"></script>

<div style="max-width: 800px; margin: 0 auto; padding: 20px; background: rgba(255,255,255,0.7); backdrop-filter: blur(10px); border-radius: 16px; box-shadow: 0 8px 32px rgba(255,179,193,0.15); border: 1px solid rgba(255,255,255,0.4);">
  <p style="margin-top: 0; color: #555;">请输入网易云公开歌单ID</p>
  <div style="display: flex; gap: 15px; margin-bottom: 15px; flex-wrap: wrap;">
    <input type="text" id="playlist-id-input" placeholder="请输入网易云歌单 ID" style="flex: 2; min-width: 200px; padding: 12px 16px; border: 2px solid #ffe5ec; border-radius: 8px; outline: none; transition: border-color 0.3s; font-family: inherit;" />
    <div style="display: flex; gap: 8px; align-items: center; flex: 1; min-width: 180px;">
      <select id="cols-input" style="padding: 12px; border: 2px solid #ffe5ec; border-radius: 8px; font-family: inherit; outline: none; background: white; flex: 1; color: #555;">
        <option value="3">3 列</option>
        <option value="4">4 列</option>
        <option value="5" selected>5 列</option>
        <option value="6">6 列</option>
        <option value="7">7 列</option>
        <option value="8">8 列</option>
        <option value="9">9 列</option>
        <option value="10">10 列</option>
      </select>
      <span style="color: #ffb3c1; font-weight: bold;">×</span>
      <select id="rows-input" style="padding: 12px; border: 2px solid #ffe5ec; border-radius: 8px; font-family: inherit; outline: none; background: white; flex: 1; color: #555;">
        <option value="3">3 行</option>
        <option value="4">4 行</option>
        <option value="5" selected>5 行</option>
        <option value="6">6 行</option>
        <option value="7">7 行</option>
        <option value="8">8 行</option>
        <option value="9">9 行</option>
        <option value="10">10 行</option>
      </select>
    </div>
    <button id="generate-btn" style="padding: 12px 24px; background: #ffb3c1; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: background 0.3s; flex: 0.5; min-width: 100px;">开始生成</button>
  </div>
  <div style="margin-bottom: 15px;">
    <label style="display: flex; align-items: center; gap: 8px; color: #ffb3c1; font-weight: bold; cursor: pointer; font-size: 15px;">
      <input type="checkbox" id="include-text-input" style="cursor: pointer;" checked /> 导出的下载图片中包含右侧文字清单
    </label>
  </div>
  <div id="custom-bg-panel" style="display: flex; flex-wrap: wrap; gap: 15px; align-items: center; margin-bottom: 25px; padding: 15px; background: rgba(255,179,193,0.05); border-radius: 8px; border: 1px dashed #ffe5ec;">
    <div style="flex: 1.2; min-width: 200px;">
      <span style="color: #ffb3c1; font-weight: bold; font-size: 13px; display: block; margin-bottom: 6px;">上传右侧自定义背景图 (Optional)</span>
      <input type="file" id="bg-upload-input" accept="image/*" style="font-size: 12px; color: #555; width: 100%; cursor: pointer;" />
    </div>
    <div style="flex: 0.8; min-width: 150px;">
      <span style="color: #ffb3c1; font-weight: bold; font-size: 13px; display: block; margin-bottom: 6px;">背景图不透明度 (<span id="opacity-val">0.5</span>)</span>
      <input type="range" id="bg-opacity-input" min="0" max="1" step="0.05" value="0.5" style="width: 100%; cursor: pointer;" />
    </div>
  </div>
  <div id="cropper-container" style="display: none; margin-bottom: 25px; text-align: center; background: #fafafa; border-radius: 8px; padding: 15px; border: 1px dashed #ffe5ec;">
    <span style="color: #ffb3c1; font-weight: bold; font-size: 14px; display: block; margin-bottom: 10px;">请在下方调整剪裁区域（拖拽/缩放选框选择你想要保留的区域）</span>
    <div style="max-height: 350px; overflow: hidden; display: inline-block; max-width: 100%;">
      <img id="cropper-img" style="max-width: 100%; max-height: 350px; display: block;" />
    </div>
  </div>
  <div id="status-tips" style="text-align: center; color: #ffb3c1; font-weight: bold; margin-bottom: 15px; display: none;">正在调取数据中，请稍候...</div>
  <canvas id="export-canvas" style="display: none;"></canvas>
  <div id="result-area" style="display: flex; flex-wrap: wrap; gap: 30px; display: none;">
    <div style="flex: 1.2; min-width: 280px; text-align: center;">
      <img id="preview-img" style="width: 100%; border-radius: 12px; box-shadow: 0 8px 24px rgba(0,0,0,0.1); margin-bottom: 15px;" />
      <a id="download-link" style="display: inline-block; padding: 12px 24px; background: #e84343; color: white; text-decoration: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: background 0.3s;">📥 下载这张拼图</a>
    </div>
    <div style="flex: 1; min-width: 250px; max-height: 400px; overflow-y: auto; padding-right: 10px;">
      <h3 style="margin-top: 0; color: #ffb3c1; border-bottom: 2px solid #ffe5ec; padding-bottom: 8px;">专辑收录清单</h3>
      <div id="tracks-container"></div>
    </div>
  </div>
</div>

<script>
  const input = document.getElementById('playlist-id-input');
  const btn = document.getElementById('generate-btn');
  const tips = document.getElementById('status-tips');
  const resultArea = document.getElementById('result-area');
  const canvas = document.getElementById('export-canvas');
  const previewImg = document.getElementById('preview-img');
  const downloadLink = document.getElementById('download-link');
  const tracksContainer = document.getElementById('tracks-container');
  const includeTextCheckbox = document.getElementById('include-text-input');
  const bgUploadInput = document.getElementById('bg-upload-input');
  const bgOpacityInput = document.getElementById('bg-opacity-input');
  const opacityValSpan = document.getElementById('opacity-val');
  const cropperContainer = document.getElementById('cropper-container');
  const cropperImg = document.getElementById('cropper-img');
  const ctx = canvas.getContext('2d');
  const imgSize = 200;
  let cropper = null;

  bgOpacityInput.addEventListener('input', (e) => {
    opacityValSpan.innerText = e.target.value;
  });

  bgUploadInput.addEventListener('change', (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        if (cropper) {
          cropper.destroy();
        }
        cropperImg.src = event.target.result;
        cropperContainer.style.display = 'block';

        const cols = parseInt(document.getElementById('cols-input').value, 10);
        const rows = parseInt(document.getElementById('rows-input').value, 10);

        cropper = new Cropper(cropperImg, {
          aspectRatio: 450 / (rows * imgSize),
          viewMode: 1,
          dragMode: 'move',
          autoCropArea: 0.8,
          restore: false,
          guides: true,
          center: true,
          highlight: false,
          cropBoxMovable: true,
          cropBoxResizable: true,
          toggleDragModeOnDblclick: false,
        });
      };
      reader.readAsDataURL(file);
    } else {
      if (cropper) {
        cropper.destroy();
        cropper = null;
      }
      cropperContainer.style.display = 'none';
    }
  });

  const updateCropRatio = () => {
    if (cropper) {
      const rows = parseInt(document.getElementById('rows-input').value, 10);
      cropper.setAspectRatio(450 / (rows * imgSize));
    }
  };
  document.getElementById('rows-input').addEventListener('change', updateCropRatio);
  document.getElementById('cols-input').addEventListener('change', updateCropRatio);

  btn.addEventListener('click', async () => {
    const playlistId = input.value.trim();
    const cols = parseInt(document.getElementById('cols-input').value, 10);
    const rows = parseInt(document.getElementById('rows-input').value, 10);
    const totalSongs = cols * rows;
    const includeText = includeTextCheckbox.checked;
    const opacity = parseFloat(bgOpacityInput.value);

    if (!playlistId) {
      alert('请输入正确的歌单ID');
      return;
    }

    tips.style.display = 'block';
    tips.innerText = '正在从网易云调取歌单数据...';
    resultArea.style.display = 'none';
    tracksContainer.innerHTML = '';

    canvas.width = includeText ? (cols * imgSize + 450) : (cols * imgSize);
    canvas.height = rows * imgSize;

    try {
      const proxyUrl = `https://api.injahow.cn/meting/?server=netease&type=playlist&id=${playlistId}`;
      const response = await fetch(proxyUrl);
      const data = await response.json();

      if (!data || data.length === 0) {
        throw new Error('歌单为空或获取数据失败！');
      }

      const tracks = data.slice(0, totalSongs);
      tips.innerText = `成功加载 ${tracks.length} 首歌曲！正在下载封面并拼接大图...`;

      let listHtml = '<ol style="padding-left: 20px; line-height: 1.8; font-size: 14px; color: #555;">';
      const imagePromises = [];

      tracks.forEach((track, index) => {
        const artists = track.artist;  
        const albumName = track.name;   
        const rawPicUrl = track.pic;    

        listHtml += `<li>${artists} - 《${albumName}》</li>`;

        const cleanUrl = rawPicUrl.replace('http://', '').replace('https://', '');
        const corsPicUrl = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}&w=200&h=200&fit=cover&output=jpg`;

        const p = new Promise((resolve) => {
          const img = new Image();
          img.crossOrigin = 'anonymous';
          img.onload = () => {
            const row = Math.floor(index / cols);
            const col = index % cols;
            resolve({ img, x: col * imgSize, y: row * imgSize });
          };
          img.onerror = () => {
            const placeholder = new Image();
            resolve({ img: placeholder, x: (index % cols) * imgSize, y: Math.floor(index / cols) * imgSize });
          };
          img.src = corsPicUrl;
        });
        imagePromises.push(p);
      });

      listHtml += '</ol>';

      const results = await Promise.all(imagePromises);

      ctx.fillStyle = includeText ? '#151515' : '#ffffff';
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      results.forEach(({ img, x, y }) => {
        try {
          ctx.drawImage(img, x, y, imgSize, imgSize);
        } catch (e) {
          ctx.fillRect(x, y, imgSize, imgSize);
        }
      });

      if (includeText) {
        if (cropper) {
          ctx.save();
          ctx.globalAlpha = opacity;
          
          const textX = cols * imgSize;
          const textWidth = 450;
          
          const croppedCanvas = cropper.getCroppedCanvas({
            width: textWidth,
            height: canvas.height
          });
          
          ctx.drawImage(croppedCanvas, textX, 0, textWidth, canvas.height);
          ctx.restore();
        }

        ctx.fillStyle = '#ffffff';
        
        const fontSize = Math.max(10, Math.min(18, Math.floor((canvas.height - 80) / tracks.length) - 4));
        ctx.font = `${fontSize}px "LXGW WenKai Screen", sans-serif`;
        
        const paddingLeft = cols * imgSize + 30;
        const paddingTop = 40;
        const lineHeight = (canvas.height - paddingTop * 2) / tracks.length;

        tracks.forEach((track, index) => {
          const artists = track.artist;
          const albumName = track.name;
          const textLine = `${index + 1}. ${artists} - ${albumName}`;
          
          let truncatedText = textLine;
          if (ctx.measureText(truncatedText).width > 400) {
            while (ctx.measureText(truncatedText + '...').width > 400 && truncatedText.length > 0) {
              truncatedText = truncatedText.slice(0, -1);
            }
            truncatedText += '...';
          }
          
          ctx.fillText(truncatedText, paddingLeft, paddingTop + index * lineHeight + fontSize);
        });
      }

      const dataURL = canvas.toDataURL('image/jpeg', 0.9);

      previewImg.src = dataURL;
      downloadLink.href = dataURL;
      downloadLink.download = `my-topsters-${playlistId}-${cols}x${rows}.jpg`;
      tracksContainer.innerHTML = listHtml;

      tips.style.display = 'none';
      resultArea.style.display = 'flex';

    } catch (err) {
      tips.innerText = `生成失败了：${err.message}。请确保输入的是数字歌单ID，且歌单已设为公开！`;
    }
  });
</script>