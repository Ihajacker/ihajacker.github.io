---
title: 音乐墙生成器 (Topsters)
date: 2026-07-18 15:30:00
type: "music-grid"
---

<link href="https://cdn.bootcdn.net/ajax/libs/cropperjs/1.5.13/cropper.min.css" rel="stylesheet">
<script src="https://cdn.bootcdn.net/ajax/libs/cropperjs/1.5.13/cropper.min.js"></script>

<div style="max-width: 800px; margin: 0 auto; padding: 20px; background: rgba(255,255,255,0.7); backdrop-filter: blur(10px); border-radius: 16px; box-shadow: 0 8px 32px rgba(255,179,193,0.15); border: 1px solid rgba(255,255,255,0.4);">
  <style>
    /* 核心修复：强行禁用 Butterfly 容易裁剪十位数数字的列表样式，恢复完美且不缩水的浏览器原生数字编号 */
    #tracks-container ol {
      list-style-type: decimal !important;
      padding-left: 28px !important;
    }
    #tracks-container ol li::before {
      content: none !important;
    }
  </style>
  <p style="margin-top: 0; color: #555;">请输入网易云公开歌单分享链接</p>
  <div style="display: flex; gap: 15px; margin-bottom: 15px; flex-wrap: wrap;">
    <input type="text" id="playlist-id-input" placeholder="请输入歌单分享链接，或直接输入歌单 ID" style="flex: 1; padding: 12px 16px; border: 2px solid #ffe5ec; border-radius: 8px; outline: none; transition: border-color 0.3s; font-family: inherit;" />
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
  <!-- 整合美化：复选框与格式切换下拉框并排排版 -->
  <div style="margin-bottom: 15px; display: flex; flex-wrap: wrap; gap: 15px; align-items: center;">
    <label style="display: flex; align-items: center; gap: 8px; color: #ffb3c1; font-weight: bold; cursor: pointer; font-size: 15px; margin: 0;">
      <input type="checkbox" id="include-text-input" style="cursor: pointer;" checked /> 导出的下载图片中包含专辑名列表
    </label>
    <label style="display: flex; align-items: center; gap: 8px; color: #ffb3c1; font-weight: bold; cursor: pointer; font-size: 15px; margin: 0;">
      <input type="checkbox" id="dedup-input" style="cursor: pointer;" /> 去重（自动跳过重复封面）
    </label>
    <div style="display: flex; gap: 8px; align-items: center;">
      <span style="color: #ffb3c1; font-weight: bold; font-size: 15px;">显示格式：</span>
      <select id="text-type-input" style="padding: 6px 12px; border: 2px solid #ffe5ec; border-radius: 8px; font-family: inherit; outline: none; background: white; color: #555; cursor: pointer;">
        <option value="album" selected>歌手 - 《专辑名》</option>
        <option value="song">歌手 - 《歌曲名》</option>
      </select>
    </div>
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
      <a id="download-link" style="display: inline-block; padding: 12px 24px; background: #e84343; color: white; text-decoration: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: background 0.3s;">下载这张拼图</a>
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
  const textTypeInput = document.getElementById('text-type-input');
  const dedupCheckbox = document.getElementById('dedup-input');
  const ctx = canvas.getContext('2d');
  const imgSize = 200;
  let cropper = null;

  function extractPlaylistId(rawInput) {
    const text = rawInput.trim();
    if (!text) return null;

    // 兼容用户仍然直接输入纯数字 ID 的旧习惯
    if (/^\d+$/.test(text)) {
      return text;
    }

    // 优先匹配常见网易云歌单链接中的 id 参数
    const queryMatch = text.match(/(?:playlist|playlist\/detail)[^"'`\s]*?[?&]id=(\d+)/i);
    if (queryMatch) {
      return queryMatch[1];
    }

    // 兼容形如 /playlist/123456789 的路径式链接
    const pathMatch = text.match(/playlist\/(\d+)/i);
    if (pathMatch) {
      return pathMatch[1];
    }

    return null;
  }

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
    const playlistId = extractPlaylistId(input.value);
    const cols = parseInt(document.getElementById('cols-input').value, 10);
    const rows = parseInt(document.getElementById('rows-input').value, 10);
    const totalSongs = cols * rows;
    const includeText = includeTextCheckbox.checked;
    const opacity = parseFloat(bgOpacityInput.value);
    const textType = textTypeInput.value;
    const dedup = dedupCheckbox.checked;

    if (!playlistId) {
      alert('请输入正确的网易云歌单分享链接，或直接输入纯数字歌单 ID');
      return;
    }

    tips.style.display = 'block';
    tips.innerText = '正在从网易云调取歌单数据...';
    resultArea.style.display = 'none';
    tracksContainer.innerHTML = '';

    canvas.width = includeText ? (cols * imgSize + 450) : (cols * imgSize);
    canvas.height = rows * imgSize;

    try {
      let tracks = [];
      let allTracks = []; // 去重时展示用，包含所有已读取歌曲

      // -------------------------------------------------------------
      // 核心“双引擎”算法：根据用户选择，完美融合并自动分流两套数据逻辑
      // -------------------------------------------------------------
      if (textType === 'song') {
        // 【方案二（歌曲名模式）】：完全使用你测试成功的 Meting 接口（本地/线上秒开，绝无跨域拦截和10首限制）
        const metingUrl = `https://api.injahow.cn/meting/?server=netease&type=playlist&id=${playlistId}`;
        const response = await fetch(metingUrl);
        const data = await response.json();
        
        if (!data || data.length === 0) {
          throw new Error('歌单为空或获取数据失败！');
        }

        // 统一格式化为相同的内部数据结构
        tracks = data.slice(0, totalSongs).map(track => ({
          artist: track.artist, // 歌手名
          titleName: track.name, // 歌曲名作为代替
          picUrl: track.pic      // 封面图
        }));

        if (dedup) {
          const seenUrls = new Set();
          const unique = [];
          const all = [];
          for (const track of data) {
            if (unique.length >= totalSongs) break;
            const mapped = { artist: track.artist, titleName: track.name, picUrl: track.pic };
            all.push(mapped);
            if (!seenUrls.has(mapped.picUrl)) { seenUrls.add(mapped.picUrl); unique.push(mapped); }
          }
          tracks = unique;
          allTracks = all;
        } else {
          allTracks = tracks;
        }

      } else {
        // 【方案一（专辑名模式）】：完全使用你测试成功的 官方 V3 双步请求 接口（抓取真正的专辑名）
        const playlistUrl = `https://corsproxy.io/?https://music.163.com/api/v3/playlist/detail?id=${playlistId}`;
        const response1 = await fetch(playlistUrl);
        const data1 = await response1.json();

        if (!data1 || !data1.playlist || !data1.playlist.trackIds) {
          throw new Error('歌单不存在、未公开，或无法读取！');
        }

        const trackIds = data1.playlist.trackIds.slice(0, totalSongs).map(item => item.id);
        if (trackIds.length === 0) {
          throw new Error('歌单内没有歌曲！');
        }

        tips.innerText = `成功读取到 ${trackIds.length} 首歌曲，正在调取完整的歌手与真实专辑数据...`;

        const songDetailUrl = `https://corsproxy.io/?https://music.163.com/api/song/detail?ids=[${trackIds.join(',')}]`;
        const response2 = await fetch(songDetailUrl);
        const data2 = await response2.json();

        if (!data2 || !data2.songs || data2.songs.length === 0) {
          throw new Error('获取歌曲详细信息失败！');
        }

        // 统一格式化为相同的内部数据结构（包含真正的 album.name 专辑名字！）
        tracks = data2.songs.map(track => ({
          artist: track.artists.map(a => a.name).join('/'), // 真实歌手名
          titleName: track.album.name,                       // 真实专辑名！
          picUrl: track.album.picUrl                       // 原版高清大图
        }));

        if (dedup) {
          const seenUrls = new Set();
          const unique = [];
          const all = [];
          for (const t of tracks) {
            all.push(t);
            if (!seenUrls.has(t.picUrl)) { seenUrls.add(t.picUrl); unique.push(t); }
          }
          if (unique.length < totalSongs) {
            const allTrackIds = data1.playlist.trackIds.map(item => item.id);
            for (let i = totalSongs; i < allTrackIds.length && unique.length < totalSongs; i += 50) {
              const batch = allTrackIds.slice(i, i + 50);
              tips.innerText = `去重中：已扫 ${all.length} 首，找到 ${unique.length}/${totalSongs} 张不同封面...`;
              const batchUrl = `https://corsproxy.io/?https://music.163.com/api/song/detail?ids=[${batch.join(',')}]`;
              const batchResp = await fetch(batchUrl);
              const batchData = await batchResp.json();
              if (!batchData || !batchData.songs) continue;
              for (const track of batchData.songs) {
                if (unique.length >= totalSongs) break;
                const mapped = {
                  artist: track.artists.map(a => a.name).join('/'),
                  titleName: track.album.name,
                  picUrl: track.album.picUrl
                };
                all.push(mapped);
                if (!seenUrls.has(mapped.picUrl)) { seenUrls.add(mapped.picUrl); unique.push(mapped); }
              }
            }
          }
          tracks = unique.slice(0, totalSongs);
          allTracks = all;
        } else {
          allTracks = tracks;
        }
      }

      tips.innerText = `数据已全部加载完毕！正在下载封面并拼接大图...`;

      // DOM 列表（去重时展示全部已读歌曲，标记重复）
      let listHtml = '<ol style="padding-left: 20px; line-height: 1.8; font-size: 14px; color: #555;">';
      if (dedup) {
        const seenPics = new Set();
        allTracks.forEach(track => {
          const isDup = seenPics.has(track.picUrl);
          seenPics.add(track.picUrl);
          const note = isDup ? ' <span style="color:#bbb;font-size:12px;">🔄重复</span>' : '';
          listHtml += `<li>${track.artist} - 《${track.titleName}》${note}</li>`;
        });
      } else {
        allTracks.forEach(track => {
          listHtml += `<li>${track.artist} - 《${track.titleName}》</li>`;
        });
      }
      listHtml += '</ol>';

      // 网格图片始终用 tracks（去重后的唯一封面）
      const imagePromises = [];
      tracks.forEach((track, index) => {
        const rawPicUrl = track.picUrl;

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
        
        const displayList = allTracks.length > 0 ? allTracks : tracks;
        const fontSize = Math.max(10, Math.min(18, Math.floor((canvas.height - 80) / displayList.length) - 4));
        ctx.font = `${fontSize}px "LXGW WenKai Screen", sans-serif`;
        
        const paddingLeft = cols * imgSize + 30;
        const paddingTop = 40;
        const lineHeight = (canvas.height - paddingTop * 2) / displayList.length;

        displayList.forEach((track, index) => {
          const artists = track.artist;
          const albumName = track.titleName; 
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
