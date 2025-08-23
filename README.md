<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>AREXMADO 다운로드</title>
    <style>
        /* 버튼 공통 스타일 */
        .btn {
            font-size: 18px;
            padding: 10px 20px;
            margin: 10px 0;   /* 위아래 간격 */
            display: block;   /* 세로 배치 */
            width: 250px;     /* 버튼 넓이 고정 (선택) */
        }
    </style>
</head>
<body>
    
    <!-- 제목 -->
    <h1>AREXMADO 다운로드</h1>

    <!-- 설명 -->
    <p>안녕하세요, 이 코드는 arexmado 코드 다운로드 페이지의 설명 입니다.
     그리고 다운 가능한 사이트입니다</p>

   <!-- 홈 -->
    <h2>홈 페이지</h2>

<div>
  <a href="lgoe.html"><button class="btn">다운로드</button></a>
</div>
<div>
  <a href="templates/indexs.html"><button class="btn">업로드,다운로드 페이지로 이동</button></a>
</div>
<div>
  <a href="downloads_updated.html"><button class="btn">다운로드 페이지로 이동</button></a>
</div>

<section id="latest-videos" class="mx-auto max-w-6xl px-4 mt-16">
  <h2 class="text-2xl font-bold mb-4">최신 영상</h2>
  <div id="video-list" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
    <!-- 최신 영상이 여기에 동적으로 삽입됩니다 -->
  </div>
</section>

<script>
  const apiKey = 'YOUR_API_KEY'; // 발급받은 API 키를 여기에 입력하세요
  const channelId = 'UCXXXXXXX'; // AREXMADO 채널의 ID를 여기에 입력하세요

  async function fetchLatestVideos() {
    const response = await fetch(`https://www.googleapis.com/youtube/v3/search?key=${apiKey}&channelId=${channelId}&order=date&part=snippet&type=video&maxResults=3`);
    const data = await response.json();

    const videoList = document.getElementById('video-list');
    data.items.forEach(item => {
      const videoId = item.id.videoId;
      const title = item.snippet.title;
      const thumbnail = item.snippet.thumbnails.high.url;
      const videoUrl = `https://www.youtube.com/watch?v=${videoId}`;

      const videoCard = document.createElement('div');
      videoCard.classList.add('rounded-2xl', 'bg-slate-800', 'overflow-hidden', 'shadow-lg');
      videoCard.innerHTML = `
        <a href="${videoUrl}" target="_blank">
          <img class="w-full h-48 object-cover" src="${thumbnail}" alt="${title}">
          <div class="p-4">
            <h3 class="text-lg font-semibold text-white">${title}</h3>
          </div>
        </a>
      `;
      videoList.appendChild(videoCard);
    });
  }

  fetchLatestVideos();
</script>


</body>
</html>
