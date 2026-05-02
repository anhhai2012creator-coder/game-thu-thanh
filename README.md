<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Đánh Bài Bảo Vệ Thành - PeerJS</title>
  <script src="https://unpkg.com/peerjs@1.5.4/dist/peerjs.min.js"></script>
  <style>
    :root {
      --bg: #101827;
      --panel: #172235;
      --panel-2: #202f48;
      --text: #f5f7fb;
      --muted: #aeb8ca;
      --accent: #64d2ff;
      --danger: #ff6b6b;
      --good: #5ce1a5;
      --gold: #ffd166;
      --purple: #b197fc;
      --line: rgba(255,255,255,.12);
      --tile: rgba(255,255,255,.055);
    }

    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: radial-gradient(circle at top, #263a5c 0, var(--bg) 55%);
      color: var(--text);
      min-height: 100vh;
    }
    button, input { font: inherit; }
    button {
      border: 0;
      border-radius: 12px;
      padding: 10px 14px;
      cursor: pointer;
      background: var(--accent);
      color: #07111f;
      font-weight: 700;
      transition: transform .1s, opacity .1s;
    }
    button:hover { transform: translateY(-1px); }
    button:disabled { opacity: .45; cursor: not-allowed; transform: none; }
    input {
      width: 100%;
      border: 1px solid var(--line);
      border-radius: 12px;
      padding: 11px 12px;
      color: var(--text);
      background: #0d1422;
      outline: none;
    }
    .app { width: min(1380px, 100%); margin: 0 auto; padding: 18px; }
    .screen { display: none; }
    .screen.active { display: block; }
    .hero { min-height: calc(100vh - 36px); display: grid; place-items: center; }
    .lobby-card {
      width: min(860px, 100%);
      background: rgba(23,34,53,.92);
      border: 1px solid var(--line);
      border-radius: 24px;
      padding: 28px;
      box-shadow: 0 20px 70px rgba(0,0,0,.35);
    }
    .title { font-size: clamp(30px, 5vw, 54px); margin: 0 0 10px; line-height: 1.05; }
    .subtitle { margin: 0 0 24px; color: var(--muted); line-height: 1.55; }
    .lobby-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; margin-top: 18px; }
    .box, .panel {
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 18px;
      padding: 16px;
    }
    .row { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
    .copy-code { font-size: 26px; color: var(--gold); font-weight: 900; letter-spacing: 1px; user-select: all; }
    .status { margin-top: 12px; color: var(--muted); min-height: 24px; }
    .topbar {
      display: grid;
      grid-template-columns: 1fr auto 1fr;
      gap: 12px;
      align-items: center;
      margin-bottom: 14px;
    }
    .badge {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      padding: 8px 10px;
      border-radius: 999px;
      background: rgba(255,255,255,.08);
      border: 1px solid var(--line);
      color: var(--muted);
      font-size: 14px;
    }
    .main-grid { display: grid; grid-template-columns: 1fr 350px; gap: 14px; }
    .battlefield { display: grid; gap: 14px; }
    .board-wrap {
      background: linear-gradient(135deg, rgba(255,255,255,.065), rgba(255,255,255,.025));
      border: 1px solid var(--line);
      border-radius: 24px;
      padding: 16px;
      overflow: hidden;
    }
    .board-title { display: flex; justify-content: space-between; align-items: center; gap: 12px; margin-bottom: 12px; }
    .board-title h2 { margin: 0; font-size: 22px; }
    .board {
      display: grid;
      grid-template-columns: repeat(7, minmax(72px, 1fr));
      grid-template-rows: repeat(5, minmax(72px, auto));
      gap: 8px;
      background:
        linear-gradient(90deg, rgba(255,255,255,.04) 1px, transparent 1px),
        linear-gradient(rgba(255,255,255,.04) 1px, transparent 1px);
      background-size: 60px 60px;
    }
    .tile {
      min-height: 78px;
      border: 1px solid rgba(255,255,255,.09);
      background: var(--tile);
      border-radius: 16px;
      display: grid;
      place-items: center;
      position: relative;
      padding: 8px;
    }
    .tile.path::after {
      content: '';
      position: absolute;
      inset: 47% -8px auto -8px;
      border-top: 2px dashed rgba(255,255,255,.16);
      pointer-events: none;
    }
    .structure {
      width: 100%;
      min-height: 66px;
      border-radius: 16px;
      border: 1px solid var(--line);
      background: rgba(0,0,0,.18);
      padding: 8px;
      text-align: center;
      box-shadow: 0 10px 28px rgba(0,0,0,.22);
      cursor: pointer;
    }
    .structure.mine { border-color: rgba(100,210,255,.55); }
    .structure.enemy { border-color: rgba(255,107,107,.55); }
    .structure.selected { outline: 3px solid var(--accent); }
    .structure.destroyed { opacity: .45; filter: grayscale(.7); }
    .structure .icon { font-size: 22px; line-height: 1; }
    .structure .sname { font-weight: 900; font-size: 12px; margin-top: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .structure .owner { color: var(--muted); font-size: 11px; margin-top: 2px; }
    .mini-hp { height: 8px; margin-top: 6px; border-radius: 999px; overflow: hidden; background: rgba(255,255,255,.12); }
    .mini-hp > span { display: block; height: 100%; background: linear-gradient(90deg, var(--danger), var(--gold), var(--good)); }
    .castle-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; }
    .castle {
      background: linear-gradient(135deg, rgba(255,255,255,.06), rgba(255,255,255,.02));
      border: 1px solid var(--line);
      border-radius: 22px;
      padding: 16px;
      min-height: 190px;
    }
    .castle.current { outline: 2px solid var(--accent); }
    .castle.enemy { outline: 2px solid rgba(255,107,107,.55); }
    .castle h2 { margin: 0 0 8px; display: flex; justify-content: space-between; gap: 10px; align-items: center; font-size: 22px; }
    .hpbar { width: 100%; height: 16px; border-radius: 999px; overflow: hidden; background: rgba(255,255,255,.1); border: 1px solid var(--line); margin: 10px 0; }
    .hpfill { height: 100%; width: 100%; background: linear-gradient(90deg, var(--danger), var(--gold), var(--good)); transition: width .25s; }
    .stat-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; margin-top: 12px; }
    .stat { background: rgba(0,0,0,.16); border-radius: 14px; padding: 10px; color: var(--muted); font-size: 14px; }
    .stat b { color: var(--text); display: block; font-size: 18px; margin-top: 3px; }
    .event-card { background: linear-gradient(135deg, rgba(177,151,252,.22), rgba(100,210,255,.08)); border: 1px solid rgba(177,151,252,.35); border-radius: 20px; padding: 16px; }
    .event-card h3, .side h3, .hand-panel h3 { margin: 0 0 10px; }
    .hand-panel { background: var(--panel); border: 1px solid var(--line); border-radius: 22px; padding: 14px; }
    .hand { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); gap: 10px; max-height: 410px; overflow: auto; padding-right: 4px; }
    .card { position: relative; min-height: 172px; border-radius: 18px; padding: 12px; background: var(--panel-2); border: 1px solid var(--line); display: flex; flex-direction: column; gap: 7px; box-shadow: 0 10px 26px rgba(0,0,0,.18); }
    .card.attack { border-color: rgba(255,107,107,.55); }
    .card.defense { border-color: rgba(92,225,165,.55); }
    .card.skill { border-color: rgba(255,209,102,.75); }
    .card.support { border-color: rgba(100,210,255,.55); }
    .card .type { font-size: 12px; color: var(--muted); text-transform: uppercase; letter-spacing: .08em; }
    .card .name { font-size: 16px; font-weight: 900; line-height: 1.2; }
    .card .desc { color: var(--muted); font-size: 13px; line-height: 1.35; flex: 1; }
    .card .power { font-size: 13px; display: flex; justify-content: space-between; color: var(--gold); }
    .card.selected { outline: 3px solid var(--accent); transform: translateY(-2px); }
    .side { display: grid; gap: 14px; align-content: start; }
    .log { max-height: 340px; overflow: auto; display: grid; gap: 8px; color: var(--muted); font-size: 14px; line-height: 1.4; }
    .log-entry { padding: 9px 10px; background: rgba(0,0,0,.14); border-radius: 12px; border: 1px solid rgba(255,255,255,.06); }
    .actions, .target-list { display: grid; gap: 10px; }
    .danger-btn { background: var(--danger); color: white; }
    .good-btn { background: var(--good); color: #04170e; }
    .ghost-btn { background: transparent; color: var(--text); border: 1px solid var(--line); }
    .target-btn { width: 100%; background: rgba(255,255,255,.08); color: var(--text); border: 1px solid var(--line); text-align: left; display: flex; justify-content: space-between; }
    .target-btn.active { border-color: var(--accent); background: rgba(100,210,255,.16); }
    .waiting { display: none; margin-top: 14px; }
    .waiting.active { display: block; }
    .winner { position: fixed; inset: 0; display: none; place-items: center; background: rgba(0,0,0,.72); z-index: 10; padding: 18px; }
    .winner.active { display: grid; }
    .winner-card { width: min(520px, 100%); background: var(--panel); border: 1px solid var(--line); border-radius: 24px; padding: 26px; text-align: center; box-shadow: 0 20px 80px rgba(0,0,0,.5); }
    .rules { color: var(--muted); line-height: 1.55; font-size: 14px; }
    .tiny { color: var(--muted); font-size: 12px; }
    @media (max-width: 980px) { .main-grid, .castle-grid, .lobby-grid, .topbar { grid-template-columns: 1fr; } .board { grid-template-columns: repeat(5, minmax(58px, 1fr)); } }
  </style>
</head>
<body>
  <div class="app">
    <section id="lobby" class="screen active hero">
      <div class="lobby-card">
        <h1 class="title">Đánh Bài Bảo Vệ Thành</h1>
        <p class="subtitle">Game online 2 người bằng mã phòng PeerJS. Chủ phòng tạo phòng, khách vào phòng, sau đó chủ phòng bấm Bắt đầu để chia bài.</p>

        <div class="box">
          <h3>Tên người chơi</h3>
          <input id="nameInput" placeholder="Nhập tên của bạn" maxlength="18" />
        </div>

        <div class="lobby-grid">
          <div class="box">
            <h3>Tạo phòng</h3>
            <p class="rules">Chủ phòng là người xáo bộ bài và chỉ khi bấm Bắt đầu trận mới tạo game.</p>
            <button id="createRoomBtn">Tạo mã phòng</button>
            <p id="roomCode" class="copy-code"></p>
          </div>
          <div class="box">
            <h3>Vào phòng</h3>
            <input id="joinCodeInput" placeholder="Nhập mã phòng" maxlength="20" />
            <br><br>
            <button id="joinRoomBtn">Vào phòng</button>
          </div>
        </div>

        <div id="waitingPanel" class="box waiting">
          <h3>Phòng chờ</h3>
          <p id="waitingInfo" class="rules">Đang chờ người chơi...</p>
          <button id="startGameBtn" class="good-btn">Bắt đầu trận</button>
        </div>

        <p id="netStatus" class="status">Chưa kết nối.</p>
        <div class="box" style="margin-top:14px">
          <h3>Luật chính</h3>
          <div class="rules">Đầu game: 1 kỹ năng thành, 4 công, 2 thủ, 2 trợ. Mỗi ngày mỗi người có 3 lượt đánh/bỏ. Khi cả hai hết lượt, sang ngày mới: rút 2 công, 1 thủ, lật sự kiện. Bàn cờ có thành chính và nhiều trụ phụ; phá thành chính đối thủ là thắng.</div>
        </div>
      </div>
    </section>

    <section id="game" class="screen">
      <div class="topbar">
        <div class="row"><span class="badge">Phòng: <b id="topRoomCode">...</b></span><span class="badge">Bạn là: <b id="playerName">...</b></span></div>
        <div class="row" style="justify-content:center"><span class="badge">Ngày <b id="dayNumber">1</b></span><span class="badge">Lượt: <b id="turnInfo">0/3</b></span></div>
        <div class="row" style="justify-content:flex-end"><span class="badge" id="syncStatus">Đã đồng bộ</span></div>
      </div>

      <div class="main-grid">
        <main class="battlefield">
          <div class="board-wrap">
            <div class="board-title"><h2>Bàn cờ thành - trụ</h2><span class="tiny">Thẻ công đánh được thành/trụ địch. Thẻ thủ hồi/giáp cho công trình phe mình.</span></div>
            <div id="board" class="board"></div>
          </div>
          <div class="castle-grid" id="castleGrid"></div>
          <div class="event-card"><h3>Sự kiện hôm nay</h3><div id="eventText">Chưa có sự kiện.</div></div>
          <div class="hand-panel">
            <div class="row" style="justify-content:space-between; margin-bottom:10px"><h3>Bài trên tay</h3><span class="tiny">Chọn thẻ, chọn công trình mục tiêu, rồi bấm Đánh thẻ.</span></div>
            <div id="hand" class="hand"></div>
          </div>
        </main>
        <aside class="side">
          <div class="box"><h3>Hành động</h3><div class="actions"><button id="playCardBtn" class="good-btn">Đánh thẻ</button><button id="endActionBtn" class="ghost-btn">Bỏ 1 lượt đánh</button><button id="copyStateBtn" class="ghost-btn">Chép trạng thái debug</button></div><p id="actionHint" class="tiny"></p></div>
          <div class="box"><h3>Chọn mục tiêu</h3><div id="targetList" class="target-list"></div></div>
          <div class="box"><h3>Nhật ký hiệu ứng</h3><div id="log" class="log"></div></div>
        </aside>
      </div>
    </section>
  </div>

  <div id="winner" class="winner"><div class="winner-card"><h1 id="winnerText">Chiến thắng!</h1><p id="winnerSub" class="subtitle"></p><button onclick="location.reload()">Chơi lại</button></div></div>

  <script>
    const CARD_LIBRARY = {
      attack: [
        ['A01','Mũi Tên Lửa',8,'Gây 8 sát thương lên thành/trụ địch.'], ['A02','Máy Bắn Đá',12,'Gây 12 sát thương.'], ['A03','Đột Kích Đêm',10,'Gây 10 sát thương, bỏ qua 3 giáp.'], ['A04','Kỵ Binh Xung Phong',14,'Gây 14 sát thương.'], ['A05','Cầu Lửa',16,'Gây 16 sát thương, tự mất 2 giáp nếu có.'], ['A06','Phá Cổng',18,'Gây 18 nếu mục tiêu có giáp dưới 10, ngược lại gây 10.'], ['A07','Tên Độc',7,'Gây 7 và đặt độc 3 sát thương vào đầu ngày sau.'], ['A08','Bão Phi Tiêu',11,'Gây 11 sát thương.'], ['A09','Chiến Xa Gỗ',13,'Gây 13 sát thương.'], ['A10','Phù Thủy Lửa',15,'Gây 15 sát thương.'], ['A11','Bộc Phá Tường',20,'Gây 20 nhưng bạn bỏ 1 thẻ ngẫu nhiên.'], ['A12','Cung Thủ Cao Thành',9,'Gây 9 và thành chính của bạn nhận 2 giáp.'], ['A13','Hỏa Pháo Cũ',17,'50% gây 17, 50% gây 8.'], ['A14','Đội Cảm Tử',19,'Gây 19, thành chính bạn mất 5 HP.'], ['A15','Rồng Giấy',6,'Gây 6 và rút 1 thẻ công.'], ['A16','Dao Găm Bóng Tối',5,'Gây 5 trực tiếp, không bị giảm bởi giáp.'], ['A17','Nỏ Liên Châu',12,'Gây 12 sát thương chia thành 2 đợt.'], ['A18','Búa Phá Thành',21,'Gây 21, chỉ dùng sau ngày 2.'], ['A19','Mưa Đá',10,'Gây 10 và giảm 4 giáp mục tiêu.'], ['A20','Thủy Công',13,'Gây 13, thêm 5 nếu sự kiện là mưa/lũ/bão.'], ['A21','Hầm Ngầm',15,'Gây 15, bỏ qua khiên một lần.'], ['A22','Sét Đánh Tháp Canh',18,'Gây 18, không thể dùng nếu sự kiện Cấm Phép.'], ['A23','Hỏa Tiễn',14,'Gây 14 và đốt mục tiêu 2 HP mỗi đầu ngày đến hết ngày.'], ['A24','Quân Tiên Phong',9,'Gây 9, nếu là thẻ đầu trong ngày thì gây 15.'], ['A25','Lưỡi Cưa Công Thành',16,'Gây 16, giảm thêm 6 giáp mục tiêu.'], ['A26','Bắn Tỉa Chỉ Huy',7,'Gây 7 và địch mất 1 lượt đánh trong ngày.'], ['A27','Tập Kích Kho Lương',8,'Gây 8, địch không được rút 1 công ngày sau.'], ['A28','Golem Đá',22,'Gây 22, lượt sau của bạn bị bỏ qua.'], ['A29','Đạn Xuyên Giáp',12,'Gây 12 trực tiếp vào HP.'], ['A30','Tổng Công Kích',25,'Gây 25, chỉ dùng khi bạn còn dưới 40 HP.']
      ],
      defense: [
        ['D01','Tường Gỗ',0,'Mục tiêu phe mình nhận 10 giáp.'], ['D02','Tường Đá',0,'Mục tiêu phe mình nhận 16 giáp.'], ['D03','Hào Nước',0,'Nhận 12 giáp, đòn công kế tiếp giảm thêm 5.'], ['D04','Thợ Sửa Thành',0,'Hồi 12 HP cho công trình phe mình.'], ['D05','Đội Khiên Lớn',0,'Nhận 20 giáp, hết ngày mất bớt giáp.'], ['D06','Chuông Báo Động',0,'Chặn 50% sát thương đòn kế tiếp.'], ['D07','Cổng Sắt',0,'Nhận 14 giáp, miễn bỏ qua giáp 1 lần.'], ['D08','Tu Sửa Khẩn Cấp',0,'Hồi 18 HP nếu dưới 50%, ngược lại hồi 8.'], ['D09','Pháp Trận Bảo Hộ',0,'Nhận khiên phép, chặn 1 thẻ phép công.'], ['D10','Lính Gác Đêm',0,'Giảm 8 sát thương từ Đột Kích/Dao Găm trong ngày.'], ['D11','Kho Đá Dự Phòng',0,'Nhận 8 giáp và rút 1 thẻ thủ.'], ['D12','Nâng Cấp Tháp',0,'Tăng HP tối đa công trình thêm 8 và hồi 8.'], ['D13','Bẫy Chông',0,'Địch tấn công mục tiêu lần sau tự nhận 6 sát thương.'], ['D14','Cầu Treo',0,'Chặn hoàn toàn 1 đòn công dưới 12 sát thương.'], ['D15','Hầm Trú Ẩn',0,'Hôm nay mọi sát thương vào mục tiêu giảm 4.'], ['D16','Lò Rèn Giáp',0,'Nhận 6 giáp, thẻ thủ hôm nay thêm 4.'], ['D17','Phù Hiệu Kiên Cường',0,'Nếu bị phá về 0 HP trong ngày này, còn lại 1 HP một lần.'], ['D18','Đại Tu Thành',0,'Hồi 25 HP, chỉ dùng sau ngày 3.']
      ],
      skill: [
        ['S01','Thành Rồng Lửa',0,'HP thành chính 115. Mỗi ngày thẻ công đầu +3 sát thương.'], ['S02','Thành Băng Vĩnh Cửu',0,'HP thành chính 105. Mỗi ngày mọi công trình còn sống nhận 2 giáp.'], ['S03','Thành Thương Nhân',0,'HP thành chính 100. Mỗi ngày rút thêm 1 thẻ trợ.'], ['S04','Thành Cổ Linh Thiêng',0,'HP thành chính 110. Mỗi ngày thẻ thủ đầu hồi thêm 3 HP.'], ['S05','Pháo Đài Bóng Đêm',0,'HP thành chính 95. Thẻ công đầu mỗi ngày bỏ qua 4 giáp.']
      ],
      support: [
        ['U01','Khắc Sự Kiện',0,'Bỏ qua hiệu ứng sự kiện hôm nay cho bạn.'], ['U02','Trinh Sát',0,'Rút 1 thẻ công.'], ['U03','Tiếp Tế',0,'Rút 1 công và 1 thủ.'], ['U04','Đổi Gió',0,'Bỏ 1 thẻ ngẫu nhiên, rút 2 thẻ ngẫu nhiên.'], ['U05','Thuê Thợ',0,'Thẻ thủ kế tiếp trong ngày mạnh thêm 8.'], ['U06','Lệnh Tổng Động Viên',0,'Hôm nay bạn được đánh thêm 1 thẻ.'], ['U07','Mưu Kế',0,'Thẻ công kế tiếp trong ngày mạnh thêm 6.'], ['U08','Kho Bí Mật',0,'Thành chính nhận 5 giáp và rút 1 thẻ trợ.'], ['U09','Phản Gián',0,'Xóa độc/cháy/bỏ lượt trên phe bạn.'], ['U10','Hiệp Ước Tạm Thời',0,'Mọi thành chính nhận 6 giáp, bạn rút 1 thẻ.']
      ],
      event: [
        ['E01','Mưa Lớn','Mọi thẻ công giảm 3 sát thương hôm nay.'], ['E02','Nắng Gắt','Mọi thẻ thủ hồi/giáp giảm 3 hôm nay.'], ['E03','Lễ Hội','Mỗi người rút thêm 1 thẻ trợ ngay.'], ['E04','Động Đất','Mỗi thành chính mất 5 HP trực tiếp.'], ['E05','Sương Mù','Thẻ công đầu tiên mỗi người hôm nay có 40% trượt.'], ['E06','Mưa Sao Băng','Thẻ công phép tăng 5 sát thương.'], ['E07','Cấm Phép','Không thể dùng thẻ phép/sét/pháp trận hôm nay.'], ['E08','Lũ Quét','Mỗi công trình mất 4 giáp, không âm.'], ['E09','Gió Thuận','Thẻ công đầu mỗi người tăng 4 sát thương.'], ['E10','Dịch Bệnh','Không thể hồi quá 8 HP mỗi lần hôm nay.'], ['E11','Đêm Không Trăng','Đột kích/dao găm tăng 5 sát thương.'], ['E12','Tăng Thuế','Mỗi người bỏ 1 thẻ ngẫu nhiên nếu có hơn 6 thẻ.'], ['E13','Kho Lương Đầy','Mỗi người rút 1 thẻ thủ ngay.'], ['E14','Thợ Giỏi Đến Thành','Thẻ thủ đầu hôm nay tăng 6.'], ['E15','Quái Vật Lang Thang','Cuối ngày, người có ít tổng giáp hơn mất 6 HP thành chính.'], ['E16','Cổng Dịch Chuyển','Thẻ công đầu hôm nay bỏ qua 5 giáp.'], ['E17','Mất Liên Lạc','Mỗi người chỉ được giữ tối đa 8 thẻ cuối ngày.'], ['E18','Linh Khí Dồi Dào','Kỹ năng thành kích hoạt mạnh gấp đôi hôm nay.'], ['E19','Bão Cát','Chỉ được tấn công công trình đối thủ.'], ['E20','Bội Thu','Đầu ngày sau rút thêm 1 công.'], ['E21','Kẻ Trộm','Mỗi người mất 1 thẻ trợ ngẫu nhiên nếu có.'], ['E22','Cầu Vồng','Mọi hồi HP tăng 4.'], ['E23','Rạn Nứt Tường Thành','Mọi sát thương trực tiếp tăng 3.'], ['E24','Hội Chợ Vũ Khí','Thẻ công dưới 10 sát thương tăng 5.'], ['E25','Ngày Bình Yên','Mỗi thành chính hồi 3 HP.'], ['E26','Lời Nguyền','Ai đánh quá 2 thẻ hôm nay mất 4 HP thành chính.'], ['E27','Sấm Sét','Thẻ sét +7, 30% phản 3 sát thương.'], ['E28','Quân Nổi Loạn','Người có HP thành chính cao hơn mất 6 HP.'], ['E29','Đội Buôn Đi Qua','Mỗi người rút 1 công.'], ['E30','Trăng Máu','Mọi sát thương tăng 4, mọi hồi máu giảm 4.']
      ]
    };

    const TYPE_LABEL = { attack:'Tấn công', defense:'Phòng thủ', skill:'Kỹ năng thành', support:'Bổ trợ' };
    const STRUCTURE_NAMES = { main:'Thành Chính', towerL:'Trụ Trái', towerR:'Trụ Phải', gate:'Cổng Thành' };
    const STRUCTURE_ICONS = { main:'🏰', towerL:'🗼', towerR:'🗼', gate:'🚪' };
    const state = { peer:null, conn:null, isHost:false, myId:null, roomCode:'', selectedCardId:null, selectedTarget:null, game:null, connected:false, peerNames:{ host:'Chủ phòng', guest:'Khách' }, joinTimer:null };
    const $ = id => document.getElementById(id);
    function assert(condition, message){ if(!condition) throw new Error(message); }
    function escapeHtml(str){ return String(str).replace(/[&<>'"]/g, s => ({'&':'&amp;','<':'&lt;','>':'&gt;',"'":'&#39;','"':'&quot;'}[s])); }
    function clamp(n,min,max){ return Math.max(min, Math.min(max,n)); }
    function shuffle(arr){ const copy=[...arr]; for(let i=copy.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [copy[i],copy[j]]=[copy[j],copy[i]]; } return copy; }
    function draw(deck,count){ const out=[]; for(let i=0;i<count;i++){ if(deck.length) out.push(deck.shift()); } return out; }
    function makeCard(type,raw){ return { id:raw[0], uid:raw[0]+'-'+Math.random().toString(36).slice(2,9), type, name:raw[1], power:raw[2], desc:raw[3] }; }
    function getInputName(){ return ($('nameInput').value.trim() || (state.isHost ? 'Chủ phòng' : 'Khách')).slice(0,18); }
    function normalizeRoomCode(value){ return String(value || '').trim().split('').filter(ch => ch > ' ').join('').toLowerCase(); }
    function setNetStatus(text){ $('netStatus').textContent = text; }
    function markConnected(){ state.connected = true; if(state.joinTimer){ clearTimeout(state.joinTimer); state.joinTimer = null; } renderWaiting(); if(state.game) render(); }
    function peerOptions(){ return { debug: 2, secure: true, config: { iceServers: [{ urls: 'stun:stun.l.google.com:19302' }, { urls: 'stun:global.stun.twilio.com:3478' }] } }; }
    function normalizeRoomCode(value){ return String(value || '').trim().replace(/\s+/g, '').toLowerCase(); }
    function setNetStatus(text){ $('netStatus').textContent = text; }
    function markConnected(){ state.connected = true; if(state.joinTimer){ clearTimeout(state.joinTimer); state.joinTimer = null; } renderWaiting(); if(state.game) render(); }
    function peerOptions(){ return { debug: 2, secure: true, config: { iceServers: [{ urls: 'stun:stun.l.google.com:19302' }, { urls: 'stun:global.stun.twilio.com:3478' }] } }; }
    function createEffects(){ return { nextAttackBonus:0,nextDefenseBonus:0,nextIncomingHalf:false,nextIncomingSmallBlock:false,moat:false,ironGate:false,magicShield:false,nightGuard:false,shelter:false,forge:false,thorns:false,endure:false,poison:0,burn:0,skipActions:0,loseAttackDraws:0,bonusAttackNextDay:0,extraLimit:0,firstAttackDone:false,firstDefenseDone:false,firstFogAttackDone:false }; }
    function ensureEffects(obj){ obj.effects = { ...createEffects(), ...(obj.effects || {}) }; }
    function createStructure(owner,id,type,name,maxHp){ return { id: owner+'-'+id, owner, type, name, hp:maxHp, maxHp, armor:0, destroyed:false, effects:createEffects() }; }
    function createStructures(owner, mainHp){ return [ createStructure(owner,'main','main','Thành Chính',mainHp), createStructure(owner,'towerL','towerL','Trụ Trái',45), createStructure(owner,'towerR','towerR','Trụ Phải',45), createStructure(owner,'gate','gate','Cổng Thành',35) ]; }

    function makePlayer(id, name, skillDeck, attackDeck, defenseDeck, supportDeck) {
      const skill = draw(skillDeck, 1)[0];
      let maxHp = 100;
      if (skill.id === 'S01') maxHp = 115;
      if (skill.id === 'S02') maxHp = 105;
      if (skill.id === 'S04') maxHp = 110;
      if (skill.id === 'S05') maxHp = 95;
      return { id, name, skill, hp:maxHp, maxHp, armor:0, hand:[...draw(attackDeck,4),...draw(defenseDeck,2),...draw(supportDeck,2)], playedToday:0, skipEventToday:false, effects:createEffects(), structures:createStructures(id,maxHp) };
    }

    function createInitialGame(names = {}) {
      const attackDeck = shuffle(CARD_LIBRARY.attack.map(c => makeCard('attack', c)));
      const defenseDeck = shuffle(CARD_LIBRARY.defense.map(c => makeCard('defense', c)));
      const supportDeck = shuffle(CARD_LIBRARY.support.flatMap(c => [makeCard('support', c), makeCard('support', c), makeCard('support', c)]));
      const skillDeck = shuffle(CARD_LIBRARY.skill.map(c => makeCard('skill', c)));
      const eventDeck = shuffle(CARD_LIBRARY.event.map(c => ({ id: c[0], name: c[1], desc: c[2] })));
      const players = [ makePlayer('host', names.host || 'Chủ phòng', skillDeck, attackDeck, defenseDeck, supportDeck), makePlayer('guest', names.guest || 'Khách', skillDeck, attackDeck, defenseDeck, supportDeck) ];
      const game = { version:3, status:'playing', day:1, perDayLimit:3, currentEvent:null, decks:{ attackDeck, defenseDeck, supportDeck, eventDeck }, players, log:[], winner:null, seedNote:Date.now(), lastEffect:null };
      startNewDay(game, true);
      game.log.unshift('Trận đấu bắt đầu. Chủ phòng đã bấm Bắt đầu và chia bài.');
      validateGame(game);
      return game;
    }

    function getMe(){ return state.game.players.find(p => p.id === state.myId); }
    function getOpponent(){ return state.game.players.find(p => p.id !== state.myId); }
    function opponentOf(game, player){ return game.players.find(p => p.id !== player.id); }
    function getAllStructures(game){ return game.players.flatMap(p => p.structures || []); }
    function getMain(player){ return player.structures.find(s => s.type === 'main'); }
    function syncPlayerFromMain(player){ const m=getMain(player); player.hp=m.hp; player.maxHp=m.maxHp; player.armor=m.armor; }
    function syncMainFromPlayer(player){ const m=getMain(player); m.hp=player.hp; m.maxHp=player.maxHp; m.armor=player.armor; }
    function getTargetStructure(game, targetId, fallbackPlayer){ return getAllStructures(game).find(s => s.id === targetId && !s.destroyed) || getMain(fallbackPlayer); }
    function log(game,text){ game.log.unshift(text); game.log=game.log.slice(0,100); game.lastEffect=text; }
    function effectLog(game, card, text){ log(game, `✨ ${card.name}: ${text}`); }

    function validateGame(game){
      assert(game && Array.isArray(game.players) && game.players.length===2, 'Game phải có đúng 2 người chơi.');
      for(const p of game.players){
        ensureEffects(p);
        if(!Array.isArray(p.structures) || p.structures.length !== 4) p.structures = createStructures(p.id, p.maxHp || 100);
        p.structures.forEach(ensureEffects);
        const main=getMain(p); assert(main, 'Thiếu thành chính.'); syncPlayerFromMain(p);
        assert(p.hp>=0 && p.hp<=p.maxHp, 'HP người chơi không hợp lệ.');
        assert(Array.isArray(p.hand), 'Tay bài không hợp lệ.');
      }
      assert(game.decks.attackDeck && game.decks.defenseDeck && game.decks.supportDeck && game.decks.eventDeck, 'Thiếu chồng bài.');
      return true;
    }

    function startNewDay(game, firstDay=false){
      if(!firstDay) game.day += 1;
      const event = draw(game.decks.eventDeck,1)[0];
      if(!event){ game.decks.eventDeck = shuffle(CARD_LIBRARY.event.map(c => ({ id:c[0], name:c[1], desc:c[2] }))); game.currentEvent = draw(game.decks.eventDeck,1)[0]; } else game.currentEvent = event;
      for(const p of game.players){
        p.playedToday=0; p.skipEventToday=false; p.effects.firstAttackDone=false; p.effects.firstDefenseDone=false; p.effects.firstFogAttackDone=false;
        p.structures.forEach(s => { ensureEffects(s); s.effects.firstAttackDone=false; s.effects.firstDefenseDone=false; s.effects.firstFogAttackDone=false; });
        if(!firstDay){
          const attackDrawCount = clamp(2 + p.effects.bonusAttackNextDay - p.effects.loseAttackDraws, 0, 5);
          p.hand.push(...draw(game.decks.attackDeck, attackDrawCount));
          p.hand.push(...draw(game.decks.defenseDeck, 1));
          if(p.skill.id === 'S03') p.hand.push(...draw(game.decks.supportDeck, 1));
          p.effects.bonusAttackNextDay=0; p.effects.loseAttackDraws=0;
        }
        for(const s of p.structures.filter(x => !x.destroyed)){
          if(!firstDay && s.effects.poison){ applyDirectDamage(game, s, s.effects.poison, 'độc'); s.effects.poison=0; }
          if(!firstDay && s.effects.burn){ applyDirectDamage(game, s, s.effects.burn, 'cháy'); s.effects.burn=0; }
          if(p.skill.id === 'S02') s.armor += game.currentEvent?.id === 'E18' ? 4 : 2;
        }
        syncPlayerFromMain(p);
      }
      if(!firstDay) applyStartEvent(game, game.currentEvent);
      log(game, `Ngày ${game.day}: Sự kiện "${game.currentEvent.name}" - ${game.currentEvent.desc}`);
      checkWinner(game);
    }

    function applyStartEvent(game,event){
      if(!event) return;
      const activePlayers = game.players.filter(p => !p.skipEventToday);
      switch(event.id){
        case 'E03': activePlayers.forEach(p => p.hand.push(...draw(game.decks.supportDeck,1))); break;
        case 'E04': activePlayers.forEach(p => applyDirectDamage(game, getMain(p), 5, 'động đất')); break;
        case 'E08': activePlayers.forEach(p => p.structures.forEach(s => s.armor = Math.max(0, s.armor-4))); break;
        case 'E12': activePlayers.forEach(p => { if(p.hand.length>6) discardRandom(p); }); break;
        case 'E13': activePlayers.forEach(p => p.hand.push(...draw(game.decks.defenseDeck,1))); break;
        case 'E20': activePlayers.forEach(p => p.effects.bonusAttackNextDay += 1); break;
        case 'E21': activePlayers.forEach(p => discardRandomType(p,'support')); break;
        case 'E25': activePlayers.forEach(p => heal(game, getMain(p), 3)); break;
        case 'E28': { const [a,b]=game.players; if(a.hp>b.hp) applyDirectDamage(game,getMain(a),6,'quân nổi loạn'); else if(b.hp>a.hp) applyDirectDamage(game,getMain(b),6,'quân nổi loạn'); break; }
        case 'E29': activePlayers.forEach(p => p.hand.push(...draw(game.decks.attackDeck,1))); break;
      }
      game.players.forEach(syncPlayerFromMain);
    }
    function applyEndDayEvent(game){
      const event=game.currentEvent; if(!event) return;
      if(event.id==='E15'){ const totals=game.players.map(p=>({p,total:p.structures.reduce((sum,s)=>sum+s.armor,0)})); if(totals[0].total<totals[1].total) applyDirectDamage(game,getMain(totals[0].p),6,'quái vật'); else if(totals[1].total<totals[0].total) applyDirectDamage(game,getMain(totals[1].p),6,'quái vật'); }
      if(event.id==='E17') game.players.filter(p=>!p.skipEventToday).forEach(p => { while(p.hand.length>8) discardRandom(p); });
    }
    function discardRandom(player){ if(!player.hand.length) return null; const i=Math.floor(Math.random()*player.hand.length); return player.hand.splice(i,1)[0]; }
    function discardRandomType(player,type){ const cards=player.hand.map((c,i)=>({c,i})).filter(x=>x.c.type===type); if(!cards.length) return null; const pick=cards[Math.floor(Math.random()*cards.length)]; return player.hand.splice(pick.i,1)[0]; }
    function heal(game, target, amount){ let value=amount; const owner=game.players.find(p=>p.id===target.owner); const event=game.currentEvent; if(event && owner && !owner.skipEventToday){ if(event.id==='E10') value=Math.min(value,8); if(event.id==='E22') value+=4; if(event.id==='E30') value=Math.max(0,value-4); } target.hp=clamp(target.hp+value,0,target.maxHp); if(owner && target.type==='main') syncPlayerFromMain(owner); return value; }
    function applyDirectDamage(game,target,amount,source='sát thương trực tiếp'){ let dmg=amount; const owner=game.players.find(p=>p.id===target.owner); if(game.currentEvent && owner && !owner.skipEventToday && game.currentEvent.id==='E23') dmg+=3; target.hp=clamp(target.hp-dmg,0,target.maxHp); if(target.hp<=0 && target.type!=='main'){ target.destroyed=true; target.hp=0; log(game, `${target.name} của ${owner.name} bị phá hủy do ${source}.`); } else log(game, `${target.name} của ${owner.name} mất ${dmg} HP do ${source}.`); if(owner && target.type==='main') syncPlayerFromMain(owner); checkWinner(game); return dmg; }

    function applyAttackDamage(game, attacker, target, baseDamage, card){
      let damage=baseDamage; const event=game.currentEvent; const eventActive=event && !attacker.skipEventToday;
      ensureEffects(target);
      if(attacker.effects.skipActions>0){ attacker.effects.skipActions-=1; effectLog(game,card,`${attacker.name} bị mất lượt nên thẻ không kích hoạt.`); return 0; }
      if(eventActive){
        if(event.id==='E01') damage-=3; if(event.id==='E06' && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)) damage+=5; if(event.id==='E09' && !attacker.effects.firstAttackDone) damage+=4; if(event.id==='E11' && /Đột Kích|Dao Găm/i.test(card.name)) damage+=5; if(event.id==='E16' && !attacker.effects.firstAttackDone) card.tempPierce=(card.tempPierce||0)+5; if(event.id==='E23' && /trực tiếp|xuyên/i.test(card.desc)) damage+=3; if(event.id==='E24' && baseDamage<10) damage+=5; if(event.id==='E27' && /Sét/i.test(card.name)){ damage+=7; if(Math.random()<.3) applyDirectDamage(game,getMain(attacker),3,'sét phản'); } if(event.id==='E30') damage+=4; if(event.id==='E05' && !attacker.effects.firstFogAttackDone){ attacker.effects.firstFogAttackDone=true; if(Math.random()<.4){ effectLog(game,card,'trượt vì Sương Mù.'); return 0; } }
      }
      if(attacker.skill.id==='S01' && !attacker.effects.firstAttackDone) damage += event?.id==='E18' ? 6 : 3;
      if(attacker.skill.id==='S05' && !attacker.effects.firstAttackDone) card.tempPierce=(card.tempPierce||0)+(event?.id==='E18'?8:4);
      if(attacker.effects.nextAttackBonus){ damage+=attacker.effects.nextAttackBonus; attacker.effects.nextAttackBonus=0; }
      attacker.effects.firstAttackDone=true;
      if(target.effects.nightGuard && /Đột Kích|Dao Găm/i.test(card.name)) damage-=8; if(target.effects.shelter) damage-=4; if(target.effects.nextIncomingHalf){ damage=Math.ceil(damage/2); target.effects.nextIncomingHalf=false; } if(target.effects.nextIncomingSmallBlock && damage<12){ target.effects.nextIncomingSmallBlock=false; effectLog(game,card,`${target.name} chặn hoàn toàn đòn công dưới 12 sát thương.`); return 0; } if(target.effects.moat){ damage-=5; target.effects.moat=false; } if(target.effects.magicShield && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)){ target.effects.magicShield=false; effectLog(game,card,`${target.name} dùng khiên phép chặn thẻ.`); return 0; } if(target.effects.thorns){ applyDirectDamage(game,getMain(attacker),6,'bẫy chông'); target.effects.thorns=false; }
      damage=Math.max(0,damage); let pierce=card.tempPierce||0; if(/bỏ qua 3 giáp/i.test(card.desc)) pierce+=3; if(/bỏ qua khiên/i.test(card.desc)) pierce+=999; if(/trực tiếp vào HP/i.test(card.desc) || card.id==='A16') pierce+=999; if(target.effects.ironGate){ pierce=0; target.effects.ironGate=false; }
      const usableArmor=Math.max(0,target.armor-pierce); const blocked=Math.min(usableArmor,damage); target.armor=Math.max(0,target.armor-blocked); const hpDamage=damage-blocked; target.hp=clamp(target.hp-hpDamage,0,target.maxHp);
      const owner=game.players.find(p=>p.id===target.owner); if(target.hp<=0 && target.type!=='main'){ target.destroyed=true; target.hp=0; }
      if(owner && target.type==='main') syncPlayerFromMain(owner); checkWinner(game);
      effectLog(game,card,`${attacker.name} đánh ${target.name} của ${owner.name}: ${hpDamage} sát thương HP, ${blocked} bị giáp chặn${target.destroyed?' — công trình bị phá!':''}.`);
      return hpDamage;
    }

    function applyCard(game, playerId, cardUid, targetId){
      validateGame(game); if(game.winner) throw new Error('Trận đấu đã kết thúc.');
      const player=game.players.find(p=>p.id===playerId); assert(player,'Không tìm thấy người chơi.');
      const limit=game.perDayLimit+player.effects.extraLimit; assert(player.playedToday<limit,'Bạn đã hết lượt đánh trong ngày.');
      const index=player.hand.findIndex(c=>c.uid===cardUid); assert(index>=0,'Không tìm thấy thẻ trên tay.'); const card=player.hand[index];
      if(game.currentEvent && !player.skipEventToday && game.currentEvent.id==='E07' && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)) throw new Error('Sự kiện Cấm Phép đang chặn thẻ này.');
      if(card.id==='A18' && game.day<=2) throw new Error('Búa Phá Thành chỉ dùng sau ngày 2.'); if(card.id==='A30' && player.hp>=40) throw new Error('Tổng Công Kích chỉ dùng khi bạn dưới 40 HP.'); if(card.id==='D18' && game.day<=3) throw new Error('Đại Tu Thành chỉ dùng sau ngày 3.');
      player.hand.splice(index,1); player.playedToday+=1;
      const opponent=opponentOf(game,player); let target=getTargetStructure(game,targetId, card.type==='attack'?opponent:player);
      if(card.type==='attack' && target.owner===player.id) target=getMain(opponent);
      if(card.type!=='attack' && target.owner!==player.id) target=getMain(player);
      if(card.type==='attack') handleAttackCard(game,player,target,card); if(card.type==='defense') handleDefenseCard(game,player,target,card); if(card.type==='support') handleSupportCard(game,player,card);
      if(game.currentEvent && !player.skipEventToday && game.currentEvent.id==='E26' && player.playedToday>2) applyDirectDamage(game,getMain(player),4,'lời nguyền đánh quá 2 thẻ');
      game.players.forEach(syncPlayerFromMain); checkDayEnd(game); validateGame(game);
    }

    function handleAttackCard(game,player,target,card){
      let damage=card.power;
      switch(card.id){
        case 'A05': getMain(player).armor=Math.max(0,getMain(player).armor-2); effectLog(game,card,'lửa công thành bùng nổ, bạn tự mất 2 giáp thành chính.'); break;
        case 'A06': damage=target.armor<10?18:10; effectLog(game,card,target.armor<10?'cổng yếu, sát thương tăng lên 18.':'giáp dày, sát thương còn 10.'); break;
        case 'A07': target.effects.poison+=3; effectLog(game,card,`${target.name} bị độc 3 sát thương đầu ngày sau.`); break;
        case 'A11': discardRandom(player); effectLog(game,card,'sức nổ lớn, bạn phải bỏ 1 thẻ ngẫu nhiên.'); break;
        case 'A12': getMain(player).armor+=2; effectLog(game,card,'cung thủ giữ vị trí, thành chính bạn +2 giáp.'); break;
        case 'A13': damage=Math.random()<.5?17:8; effectLog(game,card,`pháo cũ bắn ${damage===17?'trúng mạnh':'yếu'}, sát thương ${damage}.`); break;
        case 'A14': applyDirectDamage(game,getMain(player),5,'đội cảm tử'); effectLog(game,card,'đổi máu: thành chính bạn mất 5 HP.'); break;
        case 'A15': player.hand.push(...draw(game.decks.attackDeck,1)); effectLog(game,card,'rồng giấy bay về, bạn rút 1 thẻ công.'); break;
        case 'A19': target.armor=Math.max(0,target.armor-4); effectLog(game,card,`${target.name} bị mưa đá phá 4 giáp.`); break;
        case 'A20': if(game.currentEvent && /Mưa|Lũ|Bão/i.test(game.currentEvent.name)){ damage+=5; effectLog(game,card,'nước dâng theo thời tiết, +5 sát thương.'); } break;
        case 'A23': target.effects.burn+=2; effectLog(game,card,`${target.name} bị cháy 2 HP đầu ngày sau.`); break;
        case 'A24': if(player.playedToday===1){ damage=15; effectLog(game,card,'thẻ đầu trong ngày, sát thương thành 15.'); } break;
        case 'A25': target.armor=Math.max(0,target.armor-6); effectLog(game,card,`${target.name} bị cưa mất 6 giáp.`); break;
        case 'A26': opponentOf(game,player).effects.skipActions+=1; effectLog(game,card,'bắn hạ chỉ huy, đối thủ mất 1 lượt đánh.'); break;
        case 'A27': opponentOf(game,player).effects.loseAttackDraws+=1; effectLog(game,card,'đốt kho lương, ngày sau đối thủ rút thiếu 1 công.'); break;
        case 'A28': player.effects.skipActions+=1; effectLog(game,card,'Golem nặng nề: lượt đánh sau của bạn bị bỏ qua.'); break;
      }
      applyAttackDamage(game,player,target,damage,card);
    }
    function handleDefenseCard(game,player,target,card){
      let gain=0, healed=0; const event=game.currentEvent; const eventActive=event && !player.skipEventToday; const bonus=player.effects.nextDefenseBonus+(target.effects.forge?4:0);
      if(eventActive){ if(event.id==='E02') gain-=3; if(event.id==='E14' && !player.effects.firstDefenseDone) gain+=6; }
      switch(card.id){
        case 'D01': gain+=10; break; case 'D02': gain+=16; break; case 'D03': gain+=12; target.effects.moat=true; break; case 'D04': healed=heal(game,target,12); break; case 'D05': gain+=20; break; case 'D06': target.effects.nextIncomingHalf=true; break; case 'D07': gain+=14; target.effects.ironGate=true; break; case 'D08': healed=heal(game,target,target.hp<target.maxHp/2?18:8); break; case 'D09': target.effects.magicShield=true; break; case 'D10': target.effects.nightGuard=true; break; case 'D11': gain+=8; player.hand.push(...draw(game.decks.defenseDeck,1)); break; case 'D12': target.maxHp+=8; healed=heal(game,target,8); break; case 'D13': target.effects.thorns=true; break; case 'D14': target.effects.nextIncomingSmallBlock=true; break; case 'D15': target.effects.shelter=true; break; case 'D16': gain+=6; target.effects.forge=true; break; case 'D17': target.effects.endure=true; break; case 'D18': healed=heal(game,target,25); break;
      }
      if(player.skill.id==='S04' && !player.effects.firstDefenseDone) healed += heal(game,target,event?.id==='E18'?6:3);
      gain=Math.max(0,gain+bonus); target.armor+=gain; player.effects.nextDefenseBonus=0; player.effects.firstDefenseDone=true; if(target.type==='main') syncPlayerFromMain(player);
      effectLog(game,card,`${target.name} của ${player.name}: +${gain} giáp, hồi ${healed} HP. Hiệu ứng riêng đã được gắn vào công trình này nếu có.`);
    }
    function handleSupportCard(game,player,card){
      switch(card.id){
        case 'U01': player.skipEventToday=true; effectLog(game,card,'bạn bỏ qua toàn bộ hiệu ứng sự kiện hôm nay.'); break;
        case 'U02': player.hand.push(...draw(game.decks.attackDeck,1)); effectLog(game,card,'trinh sát thành công, rút 1 thẻ công.'); break;
        case 'U03': player.hand.push(...draw(game.decks.attackDeck,1),...draw(game.decks.defenseDeck,1)); effectLog(game,card,'tiếp tế tới nơi, rút 1 công và 1 thủ.'); break;
        case 'U04': discardRandom(player); player.hand.push(...drawRandomCards(game,2)); effectLog(game,card,'đổi chiến thuật: bỏ 1 thẻ ngẫu nhiên, rút 2 thẻ mới.'); break;
        case 'U05': player.effects.nextDefenseBonus+=8; effectLog(game,card,'thợ đã thuê, thẻ thủ kế tiếp +8.'); break;
        case 'U06': player.effects.extraLimit+=1; effectLog(game,card,'tổng động viên, hôm nay bạn được đánh thêm 1 thẻ.'); break;
        case 'U07': player.effects.nextAttackBonus+=6; effectLog(game,card,'mưu kế sẵn sàng, thẻ công kế tiếp +6.'); break;
        case 'U08': getMain(player).armor+=5; player.hand.push(...draw(game.decks.supportDeck,1)); syncPlayerFromMain(player); effectLog(game,card,'kho bí mật mở ra: thành chính +5 giáp, rút 1 trợ.'); break;
        case 'U09': player.structures.forEach(s=>{s.effects.poison=0;s.effects.burn=0;}); player.effects.skipActions=0; effectLog(game,card,'phản gián xóa độc/cháy trên công trình và bỏ lượt trên bạn.'); break;
        case 'U10': game.players.forEach(p=>getMain(p).armor+=6); player.hand.push(...drawRandomCards(game,1)); game.players.forEach(syncPlayerFromMain); effectLog(game,card,'hiệp ước: mọi thành chính +6 giáp, bạn rút 1 thẻ.'); break;
      }
    }
    function drawRandomCards(game,count){ const out=[]; for(let i=0;i<count;i++){ const available=['attackDeck','defenseDeck','supportDeck'].filter(k=>game.decks[k].length); if(!available.length) break; const key=available[Math.floor(Math.random()*available.length)]; out.push(...draw(game.decks[key],1)); } return out; }
    function skipAction(game,playerId){ const p=game.players.find(x=>x.id===playerId); if(!p||game.winner) return; const limit=game.perDayLimit+p.effects.extraLimit; if(p.playedToday>=limit) return; p.playedToday+=1; log(game,`${p.name} bỏ 1 lượt đánh.`); checkDayEnd(game); }
    function checkDayEnd(game){ const allDone=game.players.every(p=>p.playedToday>=game.perDayLimit+p.effects.extraLimit); if(allDone && !game.winner){ applyEndDayEvent(game); for(const p of game.players){ p.effects.extraLimit=0; p.structures.forEach(s=>{ s.effects.shelter=false; s.effects.nightGuard=false; s.effects.forge=false; if(s.armor>0) s.armor=Math.floor(s.armor*.75); }); } if(!game.winner) startNewDay(game,false); } }
    function checkWinner(game){ for(const p of game.players){ for(const s of p.structures){ if(s.hp<=0 && s.effects.endure){ s.hp=1; s.effects.endure=false; log(game,`${s.name} của ${p.name} kích hoạt Kiên Cường và còn 1 HP.`); } } syncPlayerFromMain(p); } const dead=game.players.find(p=>getMain(p).hp<=0); if(dead){ const win=game.players.find(p=>p.id!==dead.id); game.winner=win.id; log(game,`${win.name} đã phá Thành Chính của ${dead.name} và chiến thắng!`); } }

    function showLobbyWaiting(){ $('waitingPanel').classList.add('active'); renderWaiting(); }
    function renderWaiting(){ const names=state.peerNames; const readyGuest=state.connected; $('waitingInfo').innerHTML = `Chủ phòng: <b>${escapeHtml(names.host||'Chủ phòng')}</b><br>Khách: <b>${escapeHtml(readyGuest ? (names.guest||'Khách') : 'đang chờ...')}</b>`; $('startGameBtn').disabled = !state.isHost || !readyGuest; $('startGameBtn').style.display = state.isHost ? 'block' : 'none'; }
    function showGame(){ $('lobby').classList.remove('active'); $('game').classList.add('active'); }
    function render(){ if(!state.game) return; validateGame(state.game); const game=state.game, me=getMe(), opp=getOpponent(); $('dayNumber').textContent=game.day; $('turnInfo').textContent=`${me.playedToday}/${game.perDayLimit+me.effects.extraLimit}`; $('playerName').textContent=me.name; $('topRoomCode').textContent=state.roomCode||'(offline)'; $('eventText').innerHTML=game.currentEvent?`<b>${escapeHtml(game.currentEvent.name)}</b>: ${escapeHtml(game.currentEvent.desc)}`:'Chưa có sự kiện.'; $('syncStatus').textContent=state.connected?'Online':'Offline / đang chờ'; renderBoard(game); renderCastles(me,opp); renderHand(me); renderTargets(game,me,opp); renderLog(game); const limit=game.perDayLimit+me.effects.extraLimit; const canPlay=!game.winner&&me.playedToday<limit; $('playCardBtn').disabled=!canPlay||!state.selectedCardId; $('endActionBtn').disabled=!canPlay; $('actionHint').textContent=canPlay?'Bạn còn lượt đánh trong ngày.':'Bạn đã hết lượt hôm nay, chờ sang ngày mới.'; if(game.winner) showWinner(game.winner===state.myId); }
    function renderBoard(game){ const map={ '0-0':'host-towerL','1-0':'host-gate','2-0':'host-main','3-0':'host-towerR','0-4':'guest-towerL','1-4':'guest-gate','2-4':'guest-main','3-4':'guest-towerR' }; const structures=Object.fromEntries(getAllStructures(game).map(s=>[s.id,s])); let html=''; for(let r=0;r<5;r++){ for(let c=0;c<7;c++){ const sid=map[`${c}-${r}`] || map[`${c-1}-${r}`]; const s=sid?structures[sid]:null; if(s){ const owner=game.players.find(p=>p.id===s.owner); const pct=clamp((s.hp/s.maxHp)*100,0,100); html+=`<div class="tile"><div class="structure ${s.owner===state.myId?'mine':'enemy'} ${state.selectedTarget===s.id?'selected':''} ${s.destroyed?'destroyed':''}" onclick="selectTarget('${s.id}')"><div class="icon">${STRUCTURE_ICONS[s.type]}</div><div class="sname">${escapeHtml(s.name)}</div><div class="owner">${escapeHtml(owner.name)}</div><div class="mini-hp"><span style="width:${pct}%"></span></div><div class="tiny">${s.hp}/${s.maxHp} · Giáp ${s.armor}</div></div></div>`; } else html+=`<div class="tile path"></div>`; } } $('board').innerHTML=html; }
    function renderCastles(me,opp){ $('castleGrid').innerHTML=[me,opp].map(p=>{ const m=getMain(p); const pct=clamp((m.hp/m.maxHp)*100,0,100); const alive=p.structures.filter(s=>!s.destroyed).length; return `<div class="castle ${p.id===state.myId?'current':'enemy'}"><h2>${escapeHtml(p.name)}<span>${m.hp}/${m.maxHp} HP</span></h2><div class="hpbar"><div class="hpfill" style="width:${pct}%"></div></div><p class="tiny"><b>${escapeHtml(p.skill.name)}</b>: ${escapeHtml(p.skill.desc)}</p><div class="stat-grid"><div class="stat">Giáp thành chính<b>${m.armor}</b></div><div class="stat">Công trình sống<b>${alive}/4</b></div><div class="stat">Bài trên tay<b>${p.hand.length}</b></div><div class="stat">Đã đánh hôm nay<b>${p.playedToday}</b></div></div></div>`; }).join(''); }
    function renderHand(me){ $('hand').innerHTML=me.hand.map(card=>`<div class="card ${card.type} ${state.selectedCardId===card.uid?'selected':''}" onclick="selectCard('${card.uid}')"><div class="type">${TYPE_LABEL[card.type]}</div><div class="name">${escapeHtml(card.name)}</div><div class="desc">${escapeHtml(card.desc)}</div><div class="power"><span>${card.power?'Sức mạnh':'Hiệu ứng'}</span><b>${card.power||'—'}</b></div></div>`).join(''); }
    function renderTargets(game,me,opp){ const card=me.hand.find(c=>c.uid===state.selectedCardId); const pool=(card && card.type==='attack'?opp.structures:me.structures).filter(s=>!s.destroyed); if(!state.selectedTarget || !pool.some(s=>s.id===state.selectedTarget)) state.selectedTarget=pool.find(s=>s.type==='main')?.id || pool[0]?.id; $('targetList').innerHTML=pool.map(s=>`<button class="target-btn ${state.selectedTarget===s.id?'active':''}" onclick="selectTarget('${s.id}')"><span>${escapeHtml(s.name)}</span><b>${s.hp}/${s.maxHp}</b></button>`).join(''); }
    function renderLog(game){ $('log').innerHTML=game.log.map(x=>`<div class="log-entry">${escapeHtml(x)}</div>`).join(''); }
    window.selectCard=function(uid){ const me=getMe(); state.selectedCardId=state.selectedCardId===uid?null:uid; const card=me.hand.find(c=>c.uid===state.selectedCardId); const opp=getOpponent(); if(card && card.type==='attack') state.selectedTarget=getMain(opp).id; if(card && card.type!=='attack') state.selectedTarget=getMain(me).id; render(); };
    window.selectTarget=function(id){ state.selectedTargfunction setupConnection(conn){
      state.conn = conn;

      function sendLobbyFromHost(){
        if(!state.isHost) return;
        state.peerNames.host = getInputName();
        send({ type:'lobby', names:state.peerNames, roomCode:state.roomCode });
      }

      function afterOpen(){
        markConnected();
        setNetStatus('Đã kết nối với bạn bè. Chủ phòng cần bấm Bắt đầu để vào trận.');
        showLobbyWaiting();
        if(state.isHost){
          sendLobbyFromHost();
        } else {
          state.peerNames.guest = getInputName();
          send({ type:'joinName', name:state.peerNames.guest, roomCode:state.roomCode });
        }
      }

      conn.on('open', afterOpen);

      // Một số trình duyệt/PeerJS có thể nhận connection ở phía chủ phòng sau khi data channel đã mở,
      // làm sự kiện open bị lỡ. Đoạn này ép chạy handshake nếu conn.open đã true.
      setTimeout(() => {
        if(conn && conn.open && !state.connected) afterOpen();
      }, 250);

      conn.on('data', data => {
        try {
          if(!data || !data.type) return;

          if(data.type === 'ping') {
            send({ type:'pong' });
            return;
          }
          if(data.type === 'pong') {
            markConnected();
            return;
          }

          if(data.type === 'joinName' && state.isHost){
            markConnected();
            state.peerNames.guest = data.name || 'Khách';
            renderWaiting();
            sendLobbyFromHost();
          }

          if(data.type === 'lobby'){
            markConnected();
            state.peerNames = data.names || state.peerNames;
            if(data.roomCode) state.roomCode = data.roomCode;
            showLobbyWaiting();
            renderWaiting();
          }

          if(data.type === 'state'){
            markConnected();
            state.game = data.game;
            if(data.roomCode) state.roomCode = data.roomCode;
            if(!state.myId) state.myId = state.isHost ? 'host' : 'guest';
            validateGame(state.game);
            showGame();
            render();
          }
        } catch(err) {
          console.error(err);
          alert('Dữ liệu nhận bị lỗi: ' + err.message);
        }
      });

      conn.on('close', () => {
        state.connected = false;
        setNetStatus('Kết nối đã đóng. Kiểm tra lại mạng hoặc tạo mã phòng mới.');
        renderWaiting();
        render();
      });

      conn.on('error', err => {
        state.connected = false;
        setNetStatus('Lỗi kết nối phòng: ' + readablePeerError(err));
        renderWaiting();
        render();
      });

      // Ping giúp phát hiện trường hợp data channel mở nhưng chưa có gói lobby.
      setTimeout(() => {
        if(conn && conn.open) send({ type:'ping' });
      }, 700);
    }

    function readablePeerError(err){
      const type = err && err.type ? err.type : '';
      const msg = err && err.message ? err.message : String(err || 'không rõ lỗi');
      if(type === 'peer-unavailable') return 'Không tìm thấy mã phòng. Hãy kiểm tra mã, hoặc chờ chủ phòng tạo xong rồi vào lại.';
      if(type === 'unavailable-id') return 'Mã phòng này đang bị dùng hoặc chưa được giải phóng. Chủ phòng hãy tạo mã mới.';
      if(type === 'network') return 'Không kết nối được máy chủ PeerJS. Kiểm tra mạng, VPN, tường lửa, hoặc thử mở bằng HTTPS/Live Server.';
      if(type === 'browser-incompatible') return 'Trình duyệt không hỗ trợ WebRTC/PeerJS.';
      return msg;
    }

    function showWinner(isMe)data.game; if(data.roomCode) state.roomCode=data.roomCode; if(!state.myId) state.myId=state.isHost?'host':'guest'; validateGame(state.game); showGame(); render(); } }catch(err){ console.error(err); alert('Dữ li$('createRoomBtn').addEventListener('click',()=>{
      const code = normalizeRoomCode('thanh-' + Math.random().toString(36).slice(2,7));
      state.roomCode = code;
      state.isHost = true;
      state.myId = 'host';
      state.connected = false;
      state.peerNames.host = getInputName();
      $('roomCode').textContent = code;
      setNetStatus('Đang tạo phòng...');
      showLobbyWaiting();

      if(state.peer && !state.peer.destroyed) state.peer.destroy();
      state.peer = new Peer(code, peerOptions());

      state.peer.on('open', id => {
        state.roomCode = id;
        $('roomCode').textContent = id;
        setNetStatus('Phòng đã tạo. Gửi đúng mã này cho bạn bè: ' + id);
        renderWaiting();
      });

      state.peer.on('connection', conn => {
        setNetStatus('Có người đang kết nối vào phòng...');
        setupConnection(conn);
      });

      state.peer.on('disconnected', () => {
        setNetStatus('Mất kết nối máy chủ PeerJS. Đang thử kết nối lại...');
        try { state.peer.reconnect(); } catch(e) { console.warn(e); }
      });

      state.peer.on('error', err => {
        setNetStatus('Lỗi tạo phòng: ' + readablePeerError(err));
        renderWaiting();
      });
    });

    $('joinRoomBtn').addEventListener('click',()=>{
      const code = normalizeRoomCode($('joinCodeInput').value);
      if(!code) return alert('Nhập mã phòng trước.');
      state.roomCode = code;
      state.isHost = false;
      state.myId = 'guest';
      state.connected = false;
      state.peerNames.guest = getInputName();
      setNetStatus('Đang vào phòng ' + code + '...');
      showLobbyWaiting();

      if(state.peer && !state.peer.destroyed) state.peer.destroy();
      state.peer = new Peer(undefined, peerOptions());

      state.peer.on('open', () => {
        const conn = state.peer.connect(code, { reliable:true, serialization:'json' });
        setupConnection(conn);
        state.joinTimer = setTimeout(() => {
          if(!state.connected){
            setNetStatus('Chưa vào được phòng. Hãy kiểm tra: mã phòng đúng chưa, chủ phòng còn mở tab chưa, hai máy có mạng ổn định không. Nếu vẫn lỗi, chủ phòng hãy tạo mã mới.');
          }
        }, 8000);
      });

      state.peer.on('disconnected', () => {
        setNetStatus('Mất kết nối máy chủ PeerJS. Đang thử kết nối lại...');
        try { state.peer.reconnect(); } catch(e) { console.warn(e); }
      });

      state.peer.on('error', err => {
        if(state.joinTimer){ clearTimeout(state.joinTimer); state.joinTimer = null; }
        setNetStatus('Lỗi vào phòng: ' + readablePeerError(err));
        renderWaiting();
      });
    });rồi bấm Bắt đầu.'; renderWaiting();}); state.peer.on('connection',conn=>setupConnection(conn)); state.peer.on('error',err=>$('netStatus').textContent='Lỗi PeerJS: '+err.message); });
    $('joinRoomBtn').addEventListener('click',()=>{ const code=$('joinCodeInput').value.trim(); if(!code) return alert('Nhập mã phòng trước.'); state.roomCode=code; state.isHost=false; state.myId='guest'; state.peerNames.guest=getInputName(); $('netStatus').textContent='Đang vào phòng...'; showLobbyWaiting(); state.peer=new Peer(undefined,{debug:1}); state.peer.on('open',()=>{ const conn=state.peer.connect(code,{reliable:true}); setupConnection(conn); }); state.peer.on('error',err=>$('netStatus').textContent='Lỗi PeerJS: '+err.message); });
    $('startGameBtn').addEventListener('click',()=>{ if(!state.isHost) return; if(!state.connected) return alert('Cần có khách vào phòng trước.'); state.peerNames.host=getInputName(); state.game=createInitialGame(state.peerNames); send({type:'state',game:state.game,roomCode:state.roomCode}); showGame(); render(); });
    $('playCardBtn').addEventListener('click',()=>{ if(!state.selectedCardId) return; localAction('play',{cardUid:state.selectedCardId,targetId:state.selectedTarget}); });
    $('endActionBtn').addEventListener('click',()=>localAction('skip'));
    $('copyStateBtn').addEventListener('click',async()=>{ const text=JSON.stringify(state.game,null,2); try{ await navigatfunction runSelfTests(){ try{ validateLibrary(); assert(normalizeRoomCode('  Thanh-ABC  ') === 'thanh-abc', 'Mã phòng phải tự bỏ khoảng trắng và không phân biệt hoa/thường.');ng thái debug.'); }catch{ prompt('Copy trạng thái debug:',text); } });

    function validateLibrary(){ assert(CARD_LIBRARY.attack.length===30,'Phải có đúng 30 thẻ công.'); assert(CARD_LIBRARY.defense.length===18,'Phải có đúng 18 thẻ thủ.'); assert(CARD_LIBRARY.skill.length===5,'Phải có đúng 5 thẻ kỹ năng.'); assert(CARD_LIBRARY.event.length===30,'Phải có đúng 30 thẻ sự kiện.'); }
    function counts(p){ return p.hand.reduce((a,c)=>{a[c.type]=(a[c.type]||0)+1; return a;},{}); }
    function runSelfTests(){ try{ validateLibrary(); for(let i=0;i<50;i++){ const g=createInitialGame({host:'A',guest:'B'}); validateGame(g); assert(g.players.every(p=>p.hand.length===8),'Đầu game mỗi người phải có 8 thẻ thường.'); assert(g.players.every(p=>p.structures.length===4),'Mỗi người phải có 4 công trình trên bàn cờ.'); assert(g.players.every(p=>getMain(p).hp===p.hp),'Thành chính phải đồng bộ HP với người chơi.'); for(const p of g.players){ const c=counts(p); assert(c.attack===4,'Đầu game phải có 4 công.'); assert(c.defense===2,'Đầu game phải có 2 thủ.'); assert(c.support===2,'Đầu game phải có 2 trợ.'); } const p=g.players[0], enemy=g.players[1]; const attack=p.hand.find(c=>c.type==='attack'&&!['A18','A30'].includes(c.id)); if(attack) applyCard(g,p.id,attack.uid,getMain(enemy).id); validateGame(g); } const g2=createInitialGame({host:'HostName',guest:'GuestName'}); assert(g2.players[0].name==='HostName' && g2.players[1].name==='GuestName','Tên người chơi phải được lưu vào game.'); const before=g2.day; g2.players.forEach(p=>p.playedToday=g2.perDayLimit); checkDayEnd(g2); assert(g2.day===before+1,'Hết 3 lượt mỗi người phải sang ngày mới.'); console.log('[SELF TEST PASS] Tên, phòng chờ, bàn cờ, thẻ và sang ngày mới ổn.'); }catch(err){ console.error('[SELF TEST FAIL]',err); alert('Self-test phát hiện lỗi: '+(err.message||String(err))); } }
    runSelfTests();
  </script>
</body>
</html>
      border-radius: 12px;
      padding: 10px 14px;
      cursor: pointer;
      background: var(--accent);
      color: #07111f;
      font-weight: 700;
      transition: transform .1s, opacity .1s;
    }
    button:hover { transform: translateY(-1px); }
    button:disabled { opacity: .45; cursor: not-allowed; transform: none; }
    input {
      width: 100%;
      border: 1px solid var(--line);
      border-radius: 12px;
      padding: 11px 12px;
      color: var(--text);
      background: #0d1422;
      outline: none;
    }
    .app { width: min(1380px, 100%); margin: 0 auto; padding: 18px; }
    .screen { display: none; }
    .screen.active { display: block; }
    .hero { min-height: calc(100vh - 36px); display: grid; place-items: center; }
    .lobby-card {
      width: min(860px, 100%);
      background: rgba(23,34,53,.92);
      border: 1px solid var(--line);
      border-radius: 24px;
      padding: 28px;
      box-shadow: 0 20px 70px rgba(0,0,0,.35);
    }
    .title { font-size: clamp(30px, 5vw, 54px); margin: 0 0 10px; line-height: 1.05; }
    .subtitle { margin: 0 0 24px; color: var(--muted); line-height: 1.55; }
    .lobby-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; margin-top: 18px; }
    .box, .panel {
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 18px;
      padding: 16px;
    }
    .row { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
    .copy-code { font-size: 26px; color: var(--gold); font-weight: 900; letter-spacing: 1px; user-select: all; }
    .status { margin-top: 12px; color: var(--muted); min-height: 24px; }
    .topbar {
      display: grid;
      grid-template-columns: 1fr auto 1fr;
      gap: 12px;
      align-items: center;
      margin-bottom: 14px;
    }
    .badge {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      padding: 8px 10px;
      border-radius: 999px;
      background: rgba(255,255,255,.08);
      border: 1px solid var(--line);
      color: var(--muted);
      font-size: 14px;
    }
    .main-grid { display: grid; grid-template-columns: 1fr 350px; gap: 14px; }
    .battlefield { display: grid; gap: 14px; }
    .board-wrap {
      background: linear-gradient(135deg, rgba(255,255,255,.065), rgba(255,255,255,.025));
      border: 1px solid var(--line);
      border-radius: 24px;
      padding: 16px;
      overflow: hidden;
    }
    .board-title { display: flex; justify-content: space-between; align-items: center; gap: 12px; margin-bottom: 12px; }
    .board-title h2 { margin: 0; font-size: 22px; }
    .board {
      display: grid;
      grid-template-columns: repeat(7, minmax(72px, 1fr));
      grid-template-rows: repeat(5, minmax(72px, auto));
      gap: 8px;
      background:
        linear-gradient(90deg, rgba(255,255,255,.04) 1px, transparent 1px),
        linear-gradient(rgba(255,255,255,.04) 1px, transparent 1px);
      background-size: 60px 60px;
    }
    .tile {
      min-height: 78px;
      border: 1px solid rgba(255,255,255,.09);
      background: var(--tile);
      border-radius: 16px;
      display: grid;
      place-items: center;
      position: relative;
      padding: 8px;
    }
    .tile.path::after {
      content: '';
      position: absolute;
      inset: 47% -8px auto -8px;
      border-top: 2px dashed rgba(255,255,255,.16);
      pointer-events: none;
    }
    .structure {
      width: 100%;
      min-height: 66px;
      border-radius: 16px;
      border: 1px solid var(--line);
      background: rgba(0,0,0,.18);
      padding: 8px;
      text-align: center;
      box-shadow: 0 10px 28px rgba(0,0,0,.22);
      cursor: pointer;
    }
    .structure.mine { border-color: rgba(100,210,255,.55); }
    .structure.enemy { border-color: rgba(255,107,107,.55); }
    .structure.selected { outline: 3px solid var(--accent); }
    .structure.destroyed { opacity: .45; filter: grayscale(.7); }
    .structure .icon { font-size: 22px; line-height: 1; }
    .structure .sname { font-weight: 900; font-size: 12px; margin-top: 4px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .structure .owner { color: var(--muted); font-size: 11px; margin-top: 2px; }
    .mini-hp { height: 8px; margin-top: 6px; border-radius: 999px; overflow: hidden; background: rgba(255,255,255,.12); }
    .mini-hp > span { display: block; height: 100%; background: linear-gradient(90deg, var(--danger), var(--gold), var(--good)); }
    .castle-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; }
    .castle {
      background: linear-gradient(135deg, rgba(255,255,255,.06), rgba(255,255,255,.02));
      border: 1px solid var(--line);
      border-radius: 22px;
      padding: 16px;
      min-height: 190px;
    }
    .castle.current { outline: 2px solid var(--accent); }
    .castle.enemy { outline: 2px solid rgba(255,107,107,.55); }
    .castle h2 { margin: 0 0 8px; display: flex; justify-content: space-between; gap: 10px; align-items: center; font-size: 22px; }
    .hpbar { width: 100%; height: 16px; border-radius: 999px; overflow: hidden; background: rgba(255,255,255,.1); border: 1px solid var(--line); margin: 10px 0; }
    .hpfill { height: 100%; width: 100%; background: linear-gradient(90deg, var(--danger), var(--gold), var(--good)); transition: width .25s; }
    .stat-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; margin-top: 12px; }
    .stat { background: rgba(0,0,0,.16); border-radius: 14px; padding: 10px; color: var(--muted); font-size: 14px; }
    .stat b { color: var(--text); display: block; font-size: 18px; margin-top: 3px; }
    .event-card { background: linear-gradient(135deg, rgba(177,151,252,.22), rgba(100,210,255,.08)); border: 1px solid rgba(177,151,252,.35); border-radius: 20px; padding: 16px; }
    .event-card h3, .side h3, .hand-panel h3 { margin: 0 0 10px; }
    .hand-panel { background: var(--panel); border: 1px solid var(--line); border-radius: 22px; padding: 14px; }
    .hand { display: grid; grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); gap: 10px; max-height: 410px; overflow: auto; padding-right: 4px; }
    .card { position: relative; min-height: 172px; border-radius: 18px; padding: 12px; background: var(--panel-2); border: 1px solid var(--line); display: flex; flex-direction: column; gap: 7px; box-shadow: 0 10px 26px rgba(0,0,0,.18); }
    .card.attack { border-color: rgba(255,107,107,.55); }
    .card.defense { border-color: rgba(92,225,165,.55); }
    .card.skill { border-color: rgba(255,209,102,.75); }
    .card.support { border-color: rgba(100,210,255,.55); }
    .card .type { font-size: 12px; color: var(--muted); text-transform: uppercase; letter-spacing: .08em; }
    .card .name { font-size: 16px; font-weight: 900; line-height: 1.2; }
    .card .desc { color: var(--muted); font-size: 13px; line-height: 1.35; flex: 1; }
    .card .power { font-size: 13px; display: flex; justify-content: space-between; color: var(--gold); }
    .card.selected { outline: 3px solid var(--accent); transform: translateY(-2px); }
    .side { display: grid; gap: 14px; align-content: start; }
    .log { max-height: 340px; overflow: auto; display: grid; gap: 8px; color: var(--muted); font-size: 14px; line-height: 1.4; }
    .log-entry { padding: 9px 10px; background: rgba(0,0,0,.14); border-radius: 12px; border: 1px solid rgba(255,255,255,.06); }
    .actions, .target-list { display: grid; gap: 10px; }
    .danger-btn { background: var(--danger); color: white; }
    .good-btn { background: var(--good); color: #04170e; }
    .ghost-btn { background: transparent; color: var(--text); border: 1px solid var(--line); }
    .target-btn { width: 100%; background: rgba(255,255,255,.08); color: var(--text); border: 1px solid var(--line); text-align: left; display: flex; justify-content: space-between; }
    .target-btn.active { border-color: var(--accent); background: rgba(100,210,255,.16); }
    .waiting { display: none; margin-top: 14px; }
    .waiting.active { display: block; }
    .winner { position: fixed; inset: 0; display: none; place-items: center; background: rgba(0,0,0,.72); z-index: 10; padding: 18px; }
    .winner.active { display: grid; }
    .winner-card { width: min(520px, 100%); background: var(--panel); border: 1px solid var(--line); border-radius: 24px; padding: 26px; text-align: center; box-shadow: 0 20px 80px rgba(0,0,0,.5); }
    .rules { color: var(--muted); line-height: 1.55; font-size: 14px; }
    .tiny { color: var(--muted); font-size: 12px; }
    @media (max-width: 980px) { .main-grid, .castle-grid, .lobby-grid, .topbar { grid-template-columns: 1fr; } .board { grid-template-columns: repeat(5, minmax(58px, 1fr)); } }
  </style>
</head>
<body>
  <div class="app">
    <section id="lobby" class="screen active hero">
      <div class="lobby-card">
        <h1 class="title">Đánh Bài Bảo Vệ Thành</h1>
        <p class="subtitle">Game online 2 người bằng mã phòng PeerJS. Chủ phòng tạo phòng, khách vào phòng, sau đó chủ phòng bấm Bắt đầu để chia bài.</p>

        <div class="box">
          <h3>Tên người chơi</h3>
          <input id="nameInput" placeholder="Nhập tên của bạn" maxlength="18" />
        </div>

        <div class="lobby-grid">
          <div class="box">
            <h3>Tạo phòng</h3>
            <p class="rules">Chủ phòng là người xáo bộ bài và chỉ khi bấm Bắt đầu trận mới tạo game.</p>
            <button id="createRoomBtn">Tạo mã phòng</button>
            <p id="roomCode" class="copy-code"></p>
          </div>
          <div class="box">
            <h3>Vào phòng</h3>
            <input id="joinCodeInput" placeholder="Nhập mã phòng" maxlength="20" />
            <br><br>
            <button id="joinRoomBtn">Vào phòng</button>
          </div>
        </div>

        <div id="waitingPanel" class="box waiting">
          <h3>Phòng chờ</h3>
          <p id="waitingInfo" class="rules">Đang chờ người chơi...</p>
          <button id="startGameBtn" class="good-btn">Bắt đầu trận</button>
        </div>

        <p id="netStatus" class="status">Chưa kết nối.</p>
        <div class="box" style="margin-top:14px">
          <h3>Luật chính</h3>
          <div class="rules">Đầu game: 1 kỹ năng thành, 4 công, 2 thủ, 2 trợ. Mỗi ngày mỗi người có 3 lượt đánh/bỏ. Khi cả hai hết lượt, sang ngày mới: rút 2 công, 1 thủ, lật sự kiện. Bàn cờ có thành chính và nhiều trụ phụ; phá thành chính đối thủ là thắng.</div>
        </div>
      </div>
    </section>

    <section id="game" class="screen">
      <div class="topbar">
        <div class="row"><span class="badge">Phòng: <b id="topRoomCode">...</b></span><span class="badge">Bạn là: <b id="playerName">...</b></span></div>
        <div class="row" style="justify-content:center"><span class="badge">Ngày <b id="dayNumber">1</b></span><span class="badge">Lượt: <b id="turnInfo">0/3</b></span></div>
        <div class="row" style="justify-content:flex-end"><span class="badge" id="syncStatus">Đã đồng bộ</span></div>
      </div>

      <div class="main-grid">
        <main class="battlefield">
          <div class="board-wrap">
            <div class="board-title"><h2>Bàn cờ thành - trụ</h2><span class="tiny">Thẻ công đánh được thành/trụ địch. Thẻ thủ hồi/giáp cho công trình phe mình.</span></div>
            <div id="board" class="board"></div>
          </div>
          <div class="castle-grid" id="castleGrid"></div>
          <div class="event-card"><h3>Sự kiện hôm nay</h3><div id="eventText">Chưa có sự kiện.</div></div>
          <div class="hand-panel">
            <div class="row" style="justify-content:space-between; margin-bottom:10px"><h3>Bài trên tay</h3><span class="tiny">Chọn thẻ, chọn công trình mục tiêu, rồi bấm Đánh thẻ.</span></div>
            <div id="hand" class="hand"></div>
          </div>
        </main>
        <aside class="side">
          <div class="box"><h3>Hành động</h3><div class="actions"><button id="playCardBtn" class="good-btn">Đánh thẻ</button><button id="endActionBtn" class="ghost-btn">Bỏ 1 lượt đánh</button><button id="copyStateBtn" class="ghost-btn">Chép trạng thái debug</button></div><p id="actionHint" class="tiny"></p></div>
          <div class="box"><h3>Chọn mục tiêu</h3><div id="targetList" class="target-list"></div></div>
          <div class="box"><h3>Nhật ký hiệu ứng</h3><div id="log" class="log"></div></div>
        </aside>
      </div>
    </section>
  </div>

  <div id="winner" class="winner"><div class="winner-card"><h1 id="winnerText">Chiến thắng!</h1><p id="winnerSub" class="subtitle"></p><button onclick="location.reload()">Chơi lại</button></div></div>

  <script>
    const CARD_LIBRARY = {
      attack: [
        ['A01','Mũi Tên Lửa',8,'Gây 8 sát thương lên thành/trụ địch.'], ['A02','Máy Bắn Đá',12,'Gây 12 sát thương.'], ['A03','Đột Kích Đêm',10,'Gây 10 sát thương, bỏ qua 3 giáp.'], ['A04','Kỵ Binh Xung Phong',14,'Gây 14 sát thương.'], ['A05','Cầu Lửa',16,'Gây 16 sát thương, tự mất 2 giáp nếu có.'], ['A06','Phá Cổng',18,'Gây 18 nếu mục tiêu có giáp dưới 10, ngược lại gây 10.'], ['A07','Tên Độc',7,'Gây 7 và đặt độc 3 sát thương vào đầu ngày sau.'], ['A08','Bão Phi Tiêu',11,'Gây 11 sát thương.'], ['A09','Chiến Xa Gỗ',13,'Gây 13 sát thương.'], ['A10','Phù Thủy Lửa',15,'Gây 15 sát thương.'], ['A11','Bộc Phá Tường',20,'Gây 20 nhưng bạn bỏ 1 thẻ ngẫu nhiên.'], ['A12','Cung Thủ Cao Thành',9,'Gây 9 và thành chính của bạn nhận 2 giáp.'], ['A13','Hỏa Pháo Cũ',17,'50% gây 17, 50% gây 8.'], ['A14','Đội Cảm Tử',19,'Gây 19, thành chính bạn mất 5 HP.'], ['A15','Rồng Giấy',6,'Gây 6 và rút 1 thẻ công.'], ['A16','Dao Găm Bóng Tối',5,'Gây 5 trực tiếp, không bị giảm bởi giáp.'], ['A17','Nỏ Liên Châu',12,'Gây 12 sát thương chia thành 2 đợt.'], ['A18','Búa Phá Thành',21,'Gây 21, chỉ dùng sau ngày 2.'], ['A19','Mưa Đá',10,'Gây 10 và giảm 4 giáp mục tiêu.'], ['A20','Thủy Công',13,'Gây 13, thêm 5 nếu sự kiện là mưa/lũ/bão.'], ['A21','Hầm Ngầm',15,'Gây 15, bỏ qua khiên một lần.'], ['A22','Sét Đánh Tháp Canh',18,'Gây 18, không thể dùng nếu sự kiện Cấm Phép.'], ['A23','Hỏa Tiễn',14,'Gây 14 và đốt mục tiêu 2 HP mỗi đầu ngày đến hết ngày.'], ['A24','Quân Tiên Phong',9,'Gây 9, nếu là thẻ đầu trong ngày thì gây 15.'], ['A25','Lưỡi Cưa Công Thành',16,'Gây 16, giảm thêm 6 giáp mục tiêu.'], ['A26','Bắn Tỉa Chỉ Huy',7,'Gây 7 và địch mất 1 lượt đánh trong ngày.'], ['A27','Tập Kích Kho Lương',8,'Gây 8, địch không được rút 1 công ngày sau.'], ['A28','Golem Đá',22,'Gây 22, lượt sau của bạn bị bỏ qua.'], ['A29','Đạn Xuyên Giáp',12,'Gây 12 trực tiếp vào HP.'], ['A30','Tổng Công Kích',25,'Gây 25, chỉ dùng khi bạn còn dưới 40 HP.']
      ],
      defense: [
        ['D01','Tường Gỗ',0,'Mục tiêu phe mình nhận 10 giáp.'], ['D02','Tường Đá',0,'Mục tiêu phe mình nhận 16 giáp.'], ['D03','Hào Nước',0,'Nhận 12 giáp, đòn công kế tiếp giảm thêm 5.'], ['D04','Thợ Sửa Thành',0,'Hồi 12 HP cho công trình phe mình.'], ['D05','Đội Khiên Lớn',0,'Nhận 20 giáp, hết ngày mất bớt giáp.'], ['D06','Chuông Báo Động',0,'Chặn 50% sát thương đòn kế tiếp.'], ['D07','Cổng Sắt',0,'Nhận 14 giáp, miễn bỏ qua giáp 1 lần.'], ['D08','Tu Sửa Khẩn Cấp',0,'Hồi 18 HP nếu dưới 50%, ngược lại hồi 8.'], ['D09','Pháp Trận Bảo Hộ',0,'Nhận khiên phép, chặn 1 thẻ phép công.'], ['D10','Lính Gác Đêm',0,'Giảm 8 sát thương từ Đột Kích/Dao Găm trong ngày.'], ['D11','Kho Đá Dự Phòng',0,'Nhận 8 giáp và rút 1 thẻ thủ.'], ['D12','Nâng Cấp Tháp',0,'Tăng HP tối đa công trình thêm 8 và hồi 8.'], ['D13','Bẫy Chông',0,'Địch tấn công mục tiêu lần sau tự nhận 6 sát thương.'], ['D14','Cầu Treo',0,'Chặn hoàn toàn 1 đòn công dưới 12 sát thương.'], ['D15','Hầm Trú Ẩn',0,'Hôm nay mọi sát thương vào mục tiêu giảm 4.'], ['D16','Lò Rèn Giáp',0,'Nhận 6 giáp, thẻ thủ hôm nay thêm 4.'], ['D17','Phù Hiệu Kiên Cường',0,'Nếu bị phá về 0 HP trong ngày này, còn lại 1 HP một lần.'], ['D18','Đại Tu Thành',0,'Hồi 25 HP, chỉ dùng sau ngày 3.']
      ],
      skill: [
        ['S01','Thành Rồng Lửa',0,'HP thành chính 115. Mỗi ngày thẻ công đầu +3 sát thương.'], ['S02','Thành Băng Vĩnh Cửu',0,'HP thành chính 105. Mỗi ngày mọi công trình còn sống nhận 2 giáp.'], ['S03','Thành Thương Nhân',0,'HP thành chính 100. Mỗi ngày rút thêm 1 thẻ trợ.'], ['S04','Thành Cổ Linh Thiêng',0,'HP thành chính 110. Mỗi ngày thẻ thủ đầu hồi thêm 3 HP.'], ['S05','Pháo Đài Bóng Đêm',0,'HP thành chính 95. Thẻ công đầu mỗi ngày bỏ qua 4 giáp.']
      ],
      support: [
        ['U01','Khắc Sự Kiện',0,'Bỏ qua hiệu ứng sự kiện hôm nay cho bạn.'], ['U02','Trinh Sát',0,'Rút 1 thẻ công.'], ['U03','Tiếp Tế',0,'Rút 1 công và 1 thủ.'], ['U04','Đổi Gió',0,'Bỏ 1 thẻ ngẫu nhiên, rút 2 thẻ ngẫu nhiên.'], ['U05','Thuê Thợ',0,'Thẻ thủ kế tiếp trong ngày mạnh thêm 8.'], ['U06','Lệnh Tổng Động Viên',0,'Hôm nay bạn được đánh thêm 1 thẻ.'], ['U07','Mưu Kế',0,'Thẻ công kế tiếp trong ngày mạnh thêm 6.'], ['U08','Kho Bí Mật',0,'Thành chính nhận 5 giáp và rút 1 thẻ trợ.'], ['U09','Phản Gián',0,'Xóa độc/cháy/bỏ lượt trên phe bạn.'], ['U10','Hiệp Ước Tạm Thời',0,'Mọi thành chính nhận 6 giáp, bạn rút 1 thẻ.']
      ],
      event: [
        ['E01','Mưa Lớn','Mọi thẻ công giảm 3 sát thương hôm nay.'], ['E02','Nắng Gắt','Mọi thẻ thủ hồi/giáp giảm 3 hôm nay.'], ['E03','Lễ Hội','Mỗi người rút thêm 1 thẻ trợ ngay.'], ['E04','Động Đất','Mỗi thành chính mất 5 HP trực tiếp.'], ['E05','Sương Mù','Thẻ công đầu tiên mỗi người hôm nay có 40% trượt.'], ['E06','Mưa Sao Băng','Thẻ công phép tăng 5 sát thương.'], ['E07','Cấm Phép','Không thể dùng thẻ phép/sét/pháp trận hôm nay.'], ['E08','Lũ Quét','Mỗi công trình mất 4 giáp, không âm.'], ['E09','Gió Thuận','Thẻ công đầu mỗi người tăng 4 sát thương.'], ['E10','Dịch Bệnh','Không thể hồi quá 8 HP mỗi lần hôm nay.'], ['E11','Đêm Không Trăng','Đột kích/dao găm tăng 5 sát thương.'], ['E12','Tăng Thuế','Mỗi người bỏ 1 thẻ ngẫu nhiên nếu có hơn 6 thẻ.'], ['E13','Kho Lương Đầy','Mỗi người rút 1 thẻ thủ ngay.'], ['E14','Thợ Giỏi Đến Thành','Thẻ thủ đầu hôm nay tăng 6.'], ['E15','Quái Vật Lang Thang','Cuối ngày, người có ít tổng giáp hơn mất 6 HP thành chính.'], ['E16','Cổng Dịch Chuyển','Thẻ công đầu hôm nay bỏ qua 5 giáp.'], ['E17','Mất Liên Lạc','Mỗi người chỉ được giữ tối đa 8 thẻ cuối ngày.'], ['E18','Linh Khí Dồi Dào','Kỹ năng thành kích hoạt mạnh gấp đôi hôm nay.'], ['E19','Bão Cát','Chỉ được tấn công công trình đối thủ.'], ['E20','Bội Thu','Đầu ngày sau rút thêm 1 công.'], ['E21','Kẻ Trộm','Mỗi người mất 1 thẻ trợ ngẫu nhiên nếu có.'], ['E22','Cầu Vồng','Mọi hồi HP tăng 4.'], ['E23','Rạn Nứt Tường Thành','Mọi sát thương trực tiếp tăng 3.'], ['E24','Hội Chợ Vũ Khí','Thẻ công dưới 10 sát thương tăng 5.'], ['E25','Ngày Bình Yên','Mỗi thành chính hồi 3 HP.'], ['E26','Lời Nguyền','Ai đánh quá 2 thẻ hôm nay mất 4 HP thành chính.'], ['E27','Sấm Sét','Thẻ sét +7, 30% phản 3 sát thương.'], ['E28','Quân Nổi Loạn','Người có HP thành chính cao hơn mất 6 HP.'], ['E29','Đội Buôn Đi Qua','Mỗi người rút 1 công.'], ['E30','Trăng Máu','Mọi sát thương tăng 4, mọi hồi máu giảm 4.']
      ]
    };

    const TYPE_LABEL = { attack:'Tấn công', defense:'Phòng thủ', skill:'Kỹ năng thành', support:'Bổ trợ' };
    const STRUCTURE_NAMES = { main:'Thành Chính', towerL:'Trụ Trái', towerR:'Trụ Phải', gate:'Cổng Thành' };
    const STRUCTURE_ICONS = { main:'🏰', towerL:'🗼', towerR:'🗼', gate:'🚪' };
    const state = { peer:null, conn:null, isHost:false, myId:null, roomCode:'', selectedCardId:null, selectedTarget:null, game:null, connected:false, peerNames:{ host:'Chủ phòng', guest:'Khách' } };
    const $ = id => document.getElementById(id);
    function assert(condition, message){ if(!condition) throw new Error(message); }
    function escapeHtml(str){ return String(str).replace(/[&<>'"]/g, s => ({'&':'&amp;','<':'&lt;','>':'&gt;',"'":'&#39;','"':'&quot;'}[s])); }
    function clamp(n,min,max){ return Math.max(min, Math.min(max,n)); }
    function shuffle(arr){ const copy=[...arr]; for(let i=copy.length-1;i>0;i--){ const j=Math.floor(Math.random()*(i+1)); [copy[i],copy[j]]=[copy[j],copy[i]]; } return copy; }
    function draw(deck,count){ const out=[]; for(let i=0;i<count;i++){ if(deck.length) out.push(deck.shift()); } return out; }
    function makeCard(type,raw){ return { id:raw[0], uid:raw[0]+'-'+Math.random().toString(36).slice(2,9), type, name:raw[1], power:raw[2], desc:raw[3] }; }
    function getInputName(){ return ($('nameInput').value.trim() || (state.isHost ? 'Chủ phòng' : 'Khách')).slice(0,18); }
    function createEffects(){ return { nextAttackBonus:0,nextDefenseBonus:0,nextIncomingHalf:false,nextIncomingSmallBlock:false,moat:false,ironGate:false,magicShield:false,nightGuard:false,shelter:false,forge:false,thorns:false,endure:false,poison:0,burn:0,skipActions:0,loseAttackDraws:0,bonusAttackNextDay:0,extraLimit:0,firstAttackDone:false,firstDefenseDone:false,firstFogAttackDone:false }; }
    function ensureEffects(obj){ obj.effects = { ...createEffects(), ...(obj.effects || {}) }; }
    function createStructure(owner,id,type,name,maxHp){ return { id: owner+'-'+id, owner, type, name, hp:maxHp, maxHp, armor:0, destroyed:false, effects:createEffects() }; }
    function createStructures(owner, mainHp){ return [ createStructure(owner,'main','main','Thành Chính',mainHp), createStructure(owner,'towerL','towerL','Trụ Trái',45), createStructure(owner,'towerR','towerR','Trụ Phải',45), createStructure(owner,'gate','gate','Cổng Thành',35) ]; }

    function makePlayer(id, name, skillDeck, attackDeck, defenseDeck, supportDeck) {
      const skill = draw(skillDeck, 1)[0];
      let maxHp = 100;
      if (skill.id === 'S01') maxHp = 115;
      if (skill.id === 'S02') maxHp = 105;
      if (skill.id === 'S04') maxHp = 110;
      if (skill.id === 'S05') maxHp = 95;
      return { id, name, skill, hp:maxHp, maxHp, armor:0, hand:[...draw(attackDeck,4),...draw(defenseDeck,2),...draw(supportDeck,2)], playedToday:0, skipEventToday:false, effects:createEffects(), structures:createStructures(id,maxHp) };
    }

    function createInitialGame(names = {}) {
      const attackDeck = shuffle(CARD_LIBRARY.attack.map(c => makeCard('attack', c)));
      const defenseDeck = shuffle(CARD_LIBRARY.defense.map(c => makeCard('defense', c)));
      const supportDeck = shuffle(CARD_LIBRARY.support.flatMap(c => [makeCard('support', c), makeCard('support', c), makeCard('support', c)]));
      const skillDeck = shuffle(CARD_LIBRARY.skill.map(c => makeCard('skill', c)));
      const eventDeck = shuffle(CARD_LIBRARY.event.map(c => ({ id: c[0], name: c[1], desc: c[2] })));
      const players = [ makePlayer('host', names.host || 'Chủ phòng', skillDeck, attackDeck, defenseDeck, supportDeck), makePlayer('guest', names.guest || 'Khách', skillDeck, attackDeck, defenseDeck, supportDeck) ];
      const game = { version:3, status:'playing', day:1, perDayLimit:3, currentEvent:null, decks:{ attackDeck, defenseDeck, supportDeck, eventDeck }, players, log:[], winner:null, seedNote:Date.now(), lastEffect:null };
      startNewDay(game, true);
      game.log.unshift('Trận đấu bắt đầu. Chủ phòng đã bấm Bắt đầu và chia bài.');
      validateGame(game);
      return game;
    }

    function getMe(){ return state.game.players.find(p => p.id === state.myId); }
    function getOpponent(){ return state.game.players.find(p => p.id !== state.myId); }
    function opponentOf(game, player){ return game.players.find(p => p.id !== player.id); }
    function getAllStructures(game){ return game.players.flatMap(p => p.structures || []); }
    function getMain(player){ return player.structures.find(s => s.type === 'main'); }
    function syncPlayerFromMain(player){ const m=getMain(player); player.hp=m.hp; player.maxHp=m.maxHp; player.armor=m.armor; }
    function syncMainFromPlayer(player){ const m=getMain(player); m.hp=player.hp; m.maxHp=player.maxHp; m.armor=player.armor; }
    function getTargetStructure(game, targetId, fallbackPlayer){ return getAllStructures(game).find(s => s.id === targetId && !s.destroyed) || getMain(fallbackPlayer); }
    function log(game,text){ game.log.unshift(text); game.log=game.log.slice(0,100); game.lastEffect=text; }
    function effectLog(game, card, text){ log(game, `✨ ${card.name}: ${text}`); }

    function validateGame(game){
      assert(game && Array.isArray(game.players) && game.players.length===2, 'Game phải có đúng 2 người chơi.');
      for(const p of game.players){
        ensureEffects(p);
        if(!Array.isArray(p.structures) || p.structures.length !== 4) p.structures = createStructures(p.id, p.maxHp || 100);
        p.structures.forEach(ensureEffects);
        const main=getMain(p); assert(main, 'Thiếu thành chính.'); syncPlayerFromMain(p);
        assert(p.hp>=0 && p.hp<=p.maxHp, 'HP người chơi không hợp lệ.');
        assert(Array.isArray(p.hand), 'Tay bài không hợp lệ.');
      }
      assert(game.decks.attackDeck && game.decks.defenseDeck && game.decks.supportDeck && game.decks.eventDeck, 'Thiếu chồng bài.');
      return true;
    }

    function startNewDay(game, firstDay=false){
      if(!firstDay) game.day += 1;
      const event = draw(game.decks.eventDeck,1)[0];
      if(!event){ game.decks.eventDeck = shuffle(CARD_LIBRARY.event.map(c => ({ id:c[0], name:c[1], desc:c[2] }))); game.currentEvent = draw(game.decks.eventDeck,1)[0]; } else game.currentEvent = event;
      for(const p of game.players){
        p.playedToday=0; p.skipEventToday=false; p.effects.firstAttackDone=false; p.effects.firstDefenseDone=false; p.effects.firstFogAttackDone=false;
        p.structures.forEach(s => { ensureEffects(s); s.effects.firstAttackDone=false; s.effects.firstDefenseDone=false; s.effects.firstFogAttackDone=false; });
        if(!firstDay){
          const attackDrawCount = clamp(2 + p.effects.bonusAttackNextDay - p.effects.loseAttackDraws, 0, 5);
          p.hand.push(...draw(game.decks.attackDeck, attackDrawCount));
          p.hand.push(...draw(game.decks.defenseDeck, 1));
          if(p.skill.id === 'S03') p.hand.push(...draw(game.decks.supportDeck, 1));
          p.effects.bonusAttackNextDay=0; p.effects.loseAttackDraws=0;
        }
        for(const s of p.structures.filter(x => !x.destroyed)){
          if(!firstDay && s.effects.poison){ applyDirectDamage(game, s, s.effects.poison, 'độc'); s.effects.poison=0; }
          if(!firstDay && s.effects.burn){ applyDirectDamage(game, s, s.effects.burn, 'cháy'); s.effects.burn=0; }
          if(p.skill.id === 'S02') s.armor += game.currentEvent?.id === 'E18' ? 4 : 2;
        }
        syncPlayerFromMain(p);
      }
      if(!firstDay) applyStartEvent(game, game.currentEvent);
      log(game, `Ngày ${game.day}: Sự kiện "${game.currentEvent.name}" - ${game.currentEvent.desc}`);
      checkWinner(game);
    }

    function applyStartEvent(game,event){
      if(!event) return;
      const activePlayers = game.players.filter(p => !p.skipEventToday);
      switch(event.id){
        case 'E03': activePlayers.forEach(p => p.hand.push(...draw(game.decks.supportDeck,1))); break;
        case 'E04': activePlayers.forEach(p => applyDirectDamage(game, getMain(p), 5, 'động đất')); break;
        case 'E08': activePlayers.forEach(p => p.structures.forEach(s => s.armor = Math.max(0, s.armor-4))); break;
        case 'E12': activePlayers.forEach(p => { if(p.hand.length>6) discardRandom(p); }); break;
        case 'E13': activePlayers.forEach(p => p.hand.push(...draw(game.decks.defenseDeck,1))); break;
        case 'E20': activePlayers.forEach(p => p.effects.bonusAttackNextDay += 1); break;
        case 'E21': activePlayers.forEach(p => discardRandomType(p,'support')); break;
        case 'E25': activePlayers.forEach(p => heal(game, getMain(p), 3)); break;
        case 'E28': { const [a,b]=game.players; if(a.hp>b.hp) applyDirectDamage(game,getMain(a),6,'quân nổi loạn'); else if(b.hp>a.hp) applyDirectDamage(game,getMain(b),6,'quân nổi loạn'); break; }
        case 'E29': activePlayers.forEach(p => p.hand.push(...draw(game.decks.attackDeck,1))); break;
      }
      game.players.forEach(syncPlayerFromMain);
    }
    function applyEndDayEvent(game){
      const event=game.currentEvent; if(!event) return;
      if(event.id==='E15'){ const totals=game.players.map(p=>({p,total:p.structures.reduce((sum,s)=>sum+s.armor,0)})); if(totals[0].total<totals[1].total) applyDirectDamage(game,getMain(totals[0].p),6,'quái vật'); else if(totals[1].total<totals[0].total) applyDirectDamage(game,getMain(totals[1].p),6,'quái vật'); }
      if(event.id==='E17') game.players.filter(p=>!p.skipEventToday).forEach(p => { while(p.hand.length>8) discardRandom(p); });
    }
    function discardRandom(player){ if(!player.hand.length) return null; const i=Math.floor(Math.random()*player.hand.length); return player.hand.splice(i,1)[0]; }
    function discardRandomType(player,type){ const cards=player.hand.map((c,i)=>({c,i})).filter(x=>x.c.type===type); if(!cards.length) return null; const pick=cards[Math.floor(Math.random()*cards.length)]; return player.hand.splice(pick.i,1)[0]; }
    function heal(game, target, amount){ let value=amount; const owner=game.players.find(p=>p.id===target.owner); const event=game.currentEvent; if(event && owner && !owner.skipEventToday){ if(event.id==='E10') value=Math.min(value,8); if(event.id==='E22') value+=4; if(event.id==='E30') value=Math.max(0,value-4); } target.hp=clamp(target.hp+value,0,target.maxHp); if(owner && target.type==='main') syncPlayerFromMain(owner); return value; }
    function applyDirectDamage(game,target,amount,source='sát thương trực tiếp'){ let dmg=amount; const owner=game.players.find(p=>p.id===target.owner); if(game.currentEvent && owner && !owner.skipEventToday && game.currentEvent.id==='E23') dmg+=3; target.hp=clamp(target.hp-dmg,0,target.maxHp); if(target.hp<=0 && target.type!=='main'){ target.destroyed=true; target.hp=0; log(game, `${target.name} của ${owner.name} bị phá hủy do ${source}.`); } else log(game, `${target.name} của ${owner.name} mất ${dmg} HP do ${source}.`); if(owner && target.type==='main') syncPlayerFromMain(owner); checkWinner(game); return dmg; }

    function applyAttackDamage(game, attacker, target, baseDamage, card){
      let damage=baseDamage; const event=game.currentEvent; const eventActive=event && !attacker.skipEventToday;
      ensureEffects(target);
      if(attacker.effects.skipActions>0){ attacker.effects.skipActions-=1; effectLog(game,card,`${attacker.name} bị mất lượt nên thẻ không kích hoạt.`); return 0; }
      if(eventActive){
        if(event.id==='E01') damage-=3; if(event.id==='E06' && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)) damage+=5; if(event.id==='E09' && !attacker.effects.firstAttackDone) damage+=4; if(event.id==='E11' && /Đột Kích|Dao Găm/i.test(card.name)) damage+=5; if(event.id==='E16' && !attacker.effects.firstAttackDone) card.tempPierce=(card.tempPierce||0)+5; if(event.id==='E23' && /trực tiếp|xuyên/i.test(card.desc)) damage+=3; if(event.id==='E24' && baseDamage<10) damage+=5; if(event.id==='E27' && /Sét/i.test(card.name)){ damage+=7; if(Math.random()<.3) applyDirectDamage(game,getMain(attacker),3,'sét phản'); } if(event.id==='E30') damage+=4; if(event.id==='E05' && !attacker.effects.firstFogAttackDone){ attacker.effects.firstFogAttackDone=true; if(Math.random()<.4){ effectLog(game,card,'trượt vì Sương Mù.'); return 0; } }
      }
      if(attacker.skill.id==='S01' && !attacker.effects.firstAttackDone) damage += event?.id==='E18' ? 6 : 3;
      if(attacker.skill.id==='S05' && !attacker.effects.firstAttackDone) card.tempPierce=(card.tempPierce||0)+(event?.id==='E18'?8:4);
      if(attacker.effects.nextAttackBonus){ damage+=attacker.effects.nextAttackBonus; attacker.effects.nextAttackBonus=0; }
      attacker.effects.firstAttackDone=true;
      if(target.effects.nightGuard && /Đột Kích|Dao Găm/i.test(card.name)) damage-=8; if(target.effects.shelter) damage-=4; if(target.effects.nextIncomingHalf){ damage=Math.ceil(damage/2); target.effects.nextIncomingHalf=false; } if(target.effects.nextIncomingSmallBlock && damage<12){ target.effects.nextIncomingSmallBlock=false; effectLog(game,card,`${target.name} chặn hoàn toàn đòn công dưới 12 sát thương.`); return 0; } if(target.effects.moat){ damage-=5; target.effects.moat=false; } if(target.effects.magicShield && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)){ target.effects.magicShield=false; effectLog(game,card,`${target.name} dùng khiên phép chặn thẻ.`); return 0; } if(target.effects.thorns){ applyDirectDamage(game,getMain(attacker),6,'bẫy chông'); target.effects.thorns=false; }
      damage=Math.max(0,damage); let pierce=card.tempPierce||0; if(/bỏ qua 3 giáp/i.test(card.desc)) pierce+=3; if(/bỏ qua khiên/i.test(card.desc)) pierce+=999; if(/trực tiếp vào HP/i.test(card.desc) || card.id==='A16') pierce+=999; if(target.effects.ironGate){ pierce=0; target.effects.ironGate=false; }
      const usableArmor=Math.max(0,target.armor-pierce); const blocked=Math.min(usableArmor,damage); target.armor=Math.max(0,target.armor-blocked); const hpDamage=damage-blocked; target.hp=clamp(target.hp-hpDamage,0,target.maxHp);
      const owner=game.players.find(p=>p.id===target.owner); if(target.hp<=0 && target.type!=='main'){ target.destroyed=true; target.hp=0; }
      if(owner && target.type==='main') syncPlayerFromMain(owner); checkWinner(game);
      effectLog(game,card,`${attacker.name} đánh ${target.name} của ${owner.name}: ${hpDamage} sát thương HP, ${blocked} bị giáp chặn${target.destroyed?' — công trình bị phá!':''}.`);
      return hpDamage;
    }

    function applyCard(game, playerId, cardUid, targetId){
      validateGame(game); if(game.winner) throw new Error('Trận đấu đã kết thúc.');
      const player=game.players.find(p=>p.id===playerId); assert(player,'Không tìm thấy người chơi.');
      const limit=game.perDayLimit+player.effects.extraLimit; assert(player.playedToday<limit,'Bạn đã hết lượt đánh trong ngày.');
      const index=player.hand.findIndex(c=>c.uid===cardUid); assert(index>=0,'Không tìm thấy thẻ trên tay.'); const card=player.hand[index];
      if(game.currentEvent && !player.skipEventToday && game.currentEvent.id==='E07' && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)) throw new Error('Sự kiện Cấm Phép đang chặn thẻ này.');
      if(card.id==='A18' && game.day<=2) throw new Error('Búa Phá Thành chỉ dùng sau ngày 2.'); if(card.id==='A30' && player.hp>=40) throw new Error('Tổng Công Kích chỉ dùng khi bạn dưới 40 HP.'); if(card.id==='D18' && game.day<=3) throw new Error('Đại Tu Thành chỉ dùng sau ngày 3.');
      player.hand.splice(index,1); player.playedToday+=1;
      const opponent=opponentOf(game,player); let target=getTargetStructure(game,targetId, card.type==='attack'?opponent:player);
      if(card.type==='attack' && target.owner===player.id) target=getMain(opponent);
      if(card.type!=='attack' && target.owner!==player.id) target=getMain(player);
      if(card.type==='attack') handleAttackCard(game,player,target,card); if(card.type==='defense') handleDefenseCard(game,player,target,card); if(card.type==='support') handleSupportCard(game,player,card);
      if(game.currentEvent && !player.skipEventToday && game.currentEvent.id==='E26' && player.playedToday>2) applyDirectDamage(game,getMain(player),4,'lời nguyền đánh quá 2 thẻ');
      game.players.forEach(syncPlayerFromMain); checkDayEnd(game); validateGame(game);
    }

    function handleAttackCard(game,player,target,card){
      let damage=card.power;
      switch(card.id){
        case 'A05': getMain(player).armor=Math.max(0,getMain(player).armor-2); effectLog(game,card,'lửa công thành bùng nổ, bạn tự mất 2 giáp thành chính.'); break;
        case 'A06': damage=target.armor<10?18:10; effectLog(game,card,target.armor<10?'cổng yếu, sát thương tăng lên 18.':'giáp dày, sát thương còn 10.'); break;
        case 'A07': target.effects.poison+=3; effectLog(game,card,`${target.name} bị độc 3 sát thương đầu ngày sau.`); break;
        case 'A11': discardRandom(player); effectLog(game,card,'sức nổ lớn, bạn phải bỏ 1 thẻ ngẫu nhiên.'); break;
        case 'A12': getMain(player).armor+=2; effectLog(game,card,'cung thủ giữ vị trí, thành chính bạn +2 giáp.'); break;
        case 'A13': damage=Math.random()<.5?17:8; effectLog(game,card,`pháo cũ bắn ${damage===17?'trúng mạnh':'yếu'}, sát thương ${damage}.`); break;
        case 'A14': applyDirectDamage(game,getMain(player),5,'đội cảm tử'); effectLog(game,card,'đổi máu: thành chính bạn mất 5 HP.'); break;
        case 'A15': player.hand.push(...draw(game.decks.attackDeck,1)); effectLog(game,card,'rồng giấy bay về, bạn rút 1 thẻ công.'); break;
        case 'A19': target.armor=Math.max(0,target.armor-4); effectLog(game,card,`${target.name} bị mưa đá phá 4 giáp.`); break;
        case 'A20': if(game.currentEvent && /Mưa|Lũ|Bão/i.test(game.currentEvent.name)){ damage+=5; effectLog(game,card,'nước dâng theo thời tiết, +5 sát thương.'); } break;
        case 'A23': target.effects.burn+=2; effectLog(game,card,`${target.name} bị cháy 2 HP đầu ngày sau.`); break;
        case 'A24': if(player.playedToday===1){ damage=15; effectLog(game,card,'thẻ đầu trong ngày, sát thương thành 15.'); } break;
        case 'A25': target.armor=Math.max(0,target.armor-6); effectLog(game,card,`${target.name} bị cưa mất 6 giáp.`); break;
        case 'A26': opponentOf(game,player).effects.skipActions+=1; effectLog(game,card,'bắn hạ chỉ huy, đối thủ mất 1 lượt đánh.'); break;
        case 'A27': opponentOf(game,player).effects.loseAttackDraws+=1; effectLog(game,card,'đốt kho lương, ngày sau đối thủ rút thiếu 1 công.'); break;
        case 'A28': player.effects.skipActions+=1; effectLog(game,card,'Golem nặng nề: lượt đánh sau của bạn bị bỏ qua.'); break;
      }
      applyAttackDamage(game,player,target,damage,card);
    }
    function handleDefenseCard(game,player,target,card){
      let gain=0, healed=0; const event=game.currentEvent; const eventActive=event && !player.skipEventToday; const bonus=player.effects.nextDefenseBonus+(target.effects.forge?4:0);
      if(eventActive){ if(event.id==='E02') gain-=3; if(event.id==='E14' && !player.effects.firstDefenseDone) gain+=6; }
      switch(card.id){
        case 'D01': gain+=10; break; case 'D02': gain+=16; break; case 'D03': gain+=12; target.effects.moat=true; break; case 'D04': healed=heal(game,target,12); break; case 'D05': gain+=20; break; case 'D06': target.effects.nextIncomingHalf=true; break; case 'D07': gain+=14; target.effects.ironGate=true; break; case 'D08': healed=heal(game,target,target.hp<target.maxHp/2?18:8); break; case 'D09': target.effects.magicShield=true; break; case 'D10': target.effects.nightGuard=true; break; case 'D11': gain+=8; player.hand.push(...draw(game.decks.defenseDeck,1)); break; case 'D12': target.maxHp+=8; healed=heal(game,target,8); break; case 'D13': target.effects.thorns=true; break; case 'D14': target.effects.nextIncomingSmallBlock=true; break; case 'D15': target.effects.shelter=true; break; case 'D16': gain+=6; target.effects.forge=true; break; case 'D17': target.effects.endure=true; break; case 'D18': healed=heal(game,target,25); break;
      }
      if(player.skill.id==='S04' && !player.effects.firstDefenseDone) healed += heal(game,target,event?.id==='E18'?6:3);
      gain=Math.max(0,gain+bonus); target.armor+=gain; player.effects.nextDefenseBonus=0; player.effects.firstDefenseDone=true; if(target.type==='main') syncPlayerFromMain(player);
      effectLog(game,card,`${target.name} của ${player.name}: +${gain} giáp, hồi ${healed} HP. Hiệu ứng riêng đã được gắn vào công trình này nếu có.`);
    }
    function handleSupportCard(game,player,card){
      switch(card.id){
        case 'U01': player.skipEventToday=true; effectLog(game,card,'bạn bỏ qua toàn bộ hiệu ứng sự kiện hôm nay.'); break;
        case 'U02': player.hand.push(...draw(game.decks.attackDeck,1)); effectLog(game,card,'trinh sát thành công, rút 1 thẻ công.'); break;
        case 'U03': player.hand.push(...draw(game.decks.attackDeck,1),...draw(game.decks.defenseDeck,1)); effectLog(game,card,'tiếp tế tới nơi, rút 1 công và 1 thủ.'); break;
        case 'U04': discardRandom(player); player.hand.push(...drawRandomCards(game,2)); effectLog(game,card,'đổi chiến thuật: bỏ 1 thẻ ngẫu nhiên, rút 2 thẻ mới.'); break;
        case 'U05': player.effects.nextDefenseBonus+=8; effectLog(game,card,'thợ đã thuê, thẻ thủ kế tiếp +8.'); break;
        case 'U06': player.effects.extraLimit+=1; effectLog(game,card,'tổng động viên, hôm nay bạn được đánh thêm 1 thẻ.'); break;
        case 'U07': player.effects.nextAttackBonus+=6; effectLog(game,card,'mưu kế sẵn sàng, thẻ công kế tiếp +6.'); break;
        case 'U08': getMain(player).armor+=5; player.hand.push(...draw(game.decks.supportDeck,1)); syncPlayerFromMain(player); effectLog(game,card,'kho bí mật mở ra: thành chính +5 giáp, rút 1 trợ.'); break;
        case 'U09': player.structures.forEach(s=>{s.effects.poison=0;s.effects.burn=0;}); player.effects.skipActions=0; effectLog(game,card,'phản gián xóa độc/cháy trên công trình và bỏ lượt trên bạn.'); break;
        case 'U10': game.players.forEach(p=>getMain(p).armor+=6); player.hand.push(...drawRandomCards(game,1)); game.players.forEach(syncPlayerFromMain); effectLog(game,card,'hiệp ước: mọi thành chính +6 giáp, bạn rút 1 thẻ.'); break;
      }
    }
    function drawRandomCards(game,count){ const out=[]; for(let i=0;i<count;i++){ const available=['attackDeck','defenseDeck','supportDeck'].filter(k=>game.decks[k].length); if(!available.length) break; const key=available[Math.floor(Math.random()*available.length)]; out.push(...draw(game.decks[key],1)); } return out; }
    function skipAction(game,playerId){ const p=game.players.find(x=>x.id===playerId); if(!p||game.winner) return; const limit=game.perDayLimit+p.effects.extraLimit; if(p.playedToday>=limit) return; p.playedToday+=1; log(game,`${p.name} bỏ 1 lượt đánh.`); checkDayEnd(game); }
    function checkDayEnd(game){ const allDone=game.players.every(p=>p.playedToday>=game.perDayLimit+p.effects.extraLimit); if(allDone && !game.winner){ applyEndDayEvent(game); for(const p of game.players){ p.effects.extraLimit=0; p.structures.forEach(s=>{ s.effects.shelter=false; s.effects.nightGuard=false; s.effects.forge=false; if(s.armor>0) s.armor=Math.floor(s.armor*.75); }); } if(!game.winner) startNewDay(game,false); } }
    function checkWinner(game){ for(const p of game.players){ for(const s of p.structures){ if(s.hp<=0 && s.effects.endure){ s.hp=1; s.effects.endure=false; log(game,`${s.name} của ${p.name} kích hoạt Kiên Cường và còn 1 HP.`); } } syncPlayerFromMain(p); } const dead=game.players.find(p=>getMain(p).hp<=0); if(dead){ const win=game.players.find(p=>p.id!==dead.id); game.winner=win.id; log(game,`${win.name} đã phá Thành Chính của ${dead.name} và chiến thắng!`); } }

    function showLobbyWaiting(){ $('waitingPanel').classList.add('active'); renderWaiting(); }
    function renderWaiting(){ const names=state.peerNames; const readyGuest=state.connected; $('waitingInfo').innerHTML = `Chủ phòng: <b>${escapeHtml(names.host||'Chủ phòng')}</b><br>Khách: <b>${escapeHtml(readyGuest ? (names.guest||'Khách') : 'đang chờ...')}</b>`; $('startGameBtn').disabled = !state.isHost || !readyGuest; $('startGameBtn').style.display = state.isHost ? 'block' : 'none'; }
    function showGame(){ $('lobby').classList.remove('active'); $('game').classList.add('active'); }
    function render(){ if(!state.game) return; validateGame(state.game); const game=state.game, me=getMe(), opp=getOpponent(); $('dayNumber').textContent=game.day; $('turnInfo').textContent=`${me.playedToday}/${game.perDayLimit+me.effects.extraLimit}`; $('playerName').textContent=me.name; $('topRoomCode').textContent=state.roomCode||'(offline)'; $('eventText').innerHTML=game.currentEvent?`<b>${escapeHtml(game.currentEvent.name)}</b>: ${escapeHtml(game.currentEvent.desc)}`:'Chưa có sự kiện.'; $('syncStatus').textContent=state.connected?'Online':'Offline / đang chờ'; renderBoard(game); renderCastles(me,opp); renderHand(me); renderTargets(game,me,opp); renderLog(game); const limit=game.perDayLimit+me.effects.extraLimit; const canPlay=!game.winner&&me.playedToday<limit; $('playCardBtn').disabled=!canPlay||!state.selectedCardId; $('endActionBtn').disabled=!canPlay; $('actionHint').textContent=canPlay?'Bạn còn lượt đánh trong ngày.':'Bạn đã hết lượt hôm nay, chờ sang ngày mới.'; if(game.winner) showWinner(game.winner===state.myId); }
    function renderBoard(game){ const map={ '0-0':'host-towerL','1-0':'host-gate','2-0':'host-main','3-0':'host-towerR','0-4':'guest-towerL','1-4':'guest-gate','2-4':'guest-main','3-4':'guest-towerR' }; const structures=Object.fromEntries(getAllStructures(game).map(s=>[s.id,s])); let html=''; for(let r=0;r<5;r++){ for(let c=0;c<7;c++){ const sid=map[`${c}-${r}`] || map[`${c-1}-${r}`]; const s=sid?structures[sid]:null; if(s){ const owner=game.players.find(p=>p.id===s.owner); const pct=clamp((s.hp/s.maxHp)*100,0,100); html+=`<div class="tile"><div class="structure ${s.owner===state.myId?'mine':'enemy'} ${state.selectedTarget===s.id?'selected':''} ${s.destroyed?'destroyed':''}" onclick="selectTarget('${s.id}')"><div class="icon">${STRUCTURE_ICONS[s.type]}</div><div class="sname">${escapeHtml(s.name)}</div><div class="owner">${escapeHtml(owner.name)}</div><div class="mini-hp"><span style="width:${pct}%"></span></div><div class="tiny">${s.hp}/${s.maxHp} · Giáp ${s.armor}</div></div></div>`; } else html+=`<div class="tile path"></div>`; } } $('board').innerHTML=html; }
    function renderCastles(me,opp){ $('castleGrid').innerHTML=[me,opp].map(p=>{ const m=getMain(p); const pct=clamp((m.hp/m.maxHp)*100,0,100); const alive=p.structures.filter(s=>!s.destroyed).length; return `<div class="castle ${p.id===state.myId?'current':'enemy'}"><h2>${escapeHtml(p.name)}<span>${m.hp}/${m.maxHp} HP</span></h2><div class="hpbar"><div class="hpfill" style="width:${pct}%"></div></div><p class="tiny"><b>${escapeHtml(p.skill.name)}</b>: ${escapeHtml(p.skill.desc)}</p><div class="stat-grid"><div class="stat">Giáp thành chính<b>${m.armor}</b></div><div class="stat">Công trình sống<b>${alive}/4</b></div><div class="stat">Bài trên tay<b>${p.hand.length}</b></div><div class="stat">Đã đánh hôm nay<b>${p.playedToday}</b></div></div></div>`; }).join(''); }
    function renderHand(me){ $('hand').innerHTML=me.hand.map(card=>`<div class="card ${card.type} ${state.selectedCardId===card.uid?'selected':''}" onclick="selectCard('${card.uid}')"><div class="type">${TYPE_LABEL[card.type]}</div><div class="name">${escapeHtml(card.name)}</div><div class="desc">${escapeHtml(card.desc)}</div><div class="power"><span>${card.power?'Sức mạnh':'Hiệu ứng'}</span><b>${card.power||'—'}</b></div></div>`).join(''); }
    function renderTargets(game,me,opp){ const card=me.hand.find(c=>c.uid===state.selectedCardId); const pool=(card && card.type==='attack'?opp.structures:me.structures).filter(s=>!s.destroyed); if(!state.selectedTarget || !pool.some(s=>s.id===state.selectedTarget)) state.selectedTarget=pool.find(s=>s.type==='main')?.id || pool[0]?.id; $('targetList').innerHTML=pool.map(s=>`<button class="target-btn ${state.selectedTarget===s.id?'active':''}" onclick="selectTarget('${s.id}')"><span>${escapeHtml(s.name)}</span><b>${s.hp}/${s.maxHp}</b></button>`).join(''); }
    function renderLog(game){ $('log').innerHTML=game.log.map(x=>`<div class="log-entry">${escapeHtml(x)}</div>`).join(''); }
    window.selectCard=function(uid){ const me=getMe(); state.selectedCardId=state.selectedCardId===uid?null:uid; const card=me.hand.find(c=>c.uid===state.selectedCardId); const opp=getOpponent(); if(card && card.type==='attack') state.selectedTarget=getMain(opp).id; if(card && card.type!=='attack') state.selectedTarget=getMain(me).id; render(); };
    window.selectTarget=function(id){ state.selectedTarget=id; render(); };
    function localAction(type,payload={}){ if(!state.game||state.game.winner) return; try{ if(type==='play') applyCard(state.game,state.myId,payload.cardUid,payload.targetId); if(type==='skip') skipAction(state.game,state.myId); state.selectedCardId=null; validateGame(state.game); send({type:'state',game:state.game}); render(); }catch(err){ alert(err.message||'Có lỗi khi đánh thẻ.'); render(); } }
    function send(msg){ if(state.conn && state.conn.open) state.conn.send(JSON.parse(JSON.stringify(msg))); }
    function setupConnection(conn){ state.conn=conn; conn.on('open',()=>{ state.connected=true; $('netStatus').textContent='Đã kết nối với bạn bè.'; showLobbyWaiting(); if(state.isHost){ state.peerNames.host=getInputName(); send({type:'lobby', names:state.peerNames, roomCode:state.roomCode}); } else { state.peerNames.guest=getInputName(); send({type:'joinName', name:state.peerNames.guest}); } renderWaiting(); }); conn.on('data',data=>{ try{ if(data.type==='joinName' && state.isHost){ state.peerNames.guest=data.name||'Khách'; renderWaiting(); send({type:'lobby', names:state.peerNames, roomCode:state.roomCode}); } if(data.type==='lobby'){ state.peerNames=data.names||state.peerNames; if(data.roomCode) state.roomCode=data.roomCode; showLobbyWaiting(); renderWaiting(); } if(data.type==='state'){ state.game=data.game; if(data.roomCode) state.roomCode=data.roomCode; if(!state.myId) state.myId=state.isHost?'host':'guest'; validateGame(state.game); showGame(); render(); } }catch(err){ console.error(err); alert('Dữ liệu nhận bị lỗi: '+err.message); } }); conn.on('close',()=>{ state.connected=false; $('netStatus').textContent='Kết nối đã đóng.'; renderWaiting(); render(); }); conn.on('error',err=>{ state.connected=false; $('netStatus').textContent='Lỗi kết nối: '+err.message; renderWaiting(); render(); }); }
    function showWinner(isMe){ $('winner').classList.add('active'); $('winnerText').textContent=isMe?'Bạn đã thắng!':'Bạn đã thua!'; $('winnerSub').textContent=isMe?'Bạn đã phá Thành Chính đối thủ.':'Thành Chính của bạn đã bị phá.'; }
    $('createRoomBtn').addEventListener('click',()=>{ const code='thanh-'+Math.random().toString(36).slice(2,7); state.roomCode=code; state.isHost=true; state.myId='host'; state.peerNames.host=getInputName(); $('roomCode').textContent=code; $('netStatus').textContent='Đang tạo phòng...'; showLobbyWaiting(); state.peer=new Peer(code,{debug:1}); state.peer.on('open',()=>{$('netStatus').textContent='Phòng đã tạo. Gửi mã cho bạn bè rồi bấm Bắt đầu.'; renderWaiting();}); state.peer.on('connection',conn=>setupConnection(conn)); state.peer.on('error',err=>$('netStatus').textContent='Lỗi PeerJS: '+err.message); });
    $('joinRoomBtn').addEventListener('click',()=>{ const code=$('joinCodeInput').value.trim(); if(!code) return alert('Nhập mã phòng trước.'); state.roomCode=code; state.isHost=false; state.myId='guest'; state.peerNames.guest=getInputName(); $('netStatus').textContent='Đang vào phòng...'; showLobbyWaiting(); state.peer=new Peer(undefined,{debug:1}); state.peer.on('open',()=>{ const conn=state.peer.connect(code,{reliable:true}); setupConnection(conn); }); state.peer.on('error',err=>$('netStatus').textContent='Lỗi PeerJS: '+err.message); });
    $('startGameBtn').addEventListener('click',()=>{ if(!state.isHost) return; if(!state.connected) return alert('Cần có khách vào phòng trước.'); state.peerNames.host=getInputName(); state.game=createInitialGame(state.peerNames); send({type:'state',game:state.game,roomCode:state.roomCode}); showGame(); render(); });
    $('playCardBtn').addEventListener('click',()=>{ if(!state.selectedCardId) return; localAction('play',{cardUid:state.selectedCardId,targetId:state.selectedTarget}); });
    $('endActionBtn').addEventListener('click',()=>localAction('skip'));
    $('copyStateBtn').addEventListener('click',async()=>{ const text=JSON.stringify(state.game,null,2); try{ await navigator.clipboard.writeText(text); alert('Đã chép trạng thái debug.'); }catch{ prompt('Copy trạng thái debug:',text); } });

    function validateLibrary(){ assert(CARD_LIBRARY.attack.length===30,'Phải có đúng 30 thẻ công.'); assert(CARD_LIBRARY.defense.length===18,'Phải có đúng 18 thẻ thủ.'); assert(CARD_LIBRARY.skill.length===5,'Phải có đúng 5 thẻ kỹ năng.'); assert(CARD_LIBRARY.event.length===30,'Phải có đúng 30 thẻ sự kiện.'); }
    function counts(p){ return p.hand.reduce((a,c)=>{a[c.type]=(a[c.type]||0)+1; return a;},{}); }
    function runSelfTests(){ try{ validateLibrary(); for(let i=0;i<50;i++){ const g=createInitialGame({host:'A',guest:'B'}); validateGame(g); assert(g.players.every(p=>p.hand.length===8),'Đầu game mỗi người phải có 8 thẻ thường.'); assert(g.players.every(p=>p.structures.length===4),'Mỗi người phải có 4 công trình trên bàn cờ.'); assert(g.players.every(p=>getMain(p).hp===p.hp),'Thành chính phải đồng bộ HP với người chơi.'); for(const p of g.players){ const c=counts(p); assert(c.attack===4,'Đầu game phải có 4 công.'); assert(c.defense===2,'Đầu game phải có 2 thủ.'); assert(c.support===2,'Đầu game phải có 2 trợ.'); } const p=g.players[0], enemy=g.players[1]; const attack=p.hand.find(c=>c.type==='attack'&&!['A18','A30'].includes(c.id)); if(attack) applyCard(g,p.id,attack.uid,getMain(enemy).id); validateGame(g); } const g2=createInitialGame({host:'HostName',guest:'GuestName'}); assert(g2.players[0].name==='HostName' && g2.players[1].name==='GuestName','Tên người chơi phải được lưu vào game.'); const before=g2.day; g2.players.forEach(p=>p.playedToday=g2.perDayLimit); checkDayEnd(g2); assert(g2.day===before+1,'Hết 3 lượt mỗi người phải sang ngày mới.'); console.log('[SELF TEST PASS] Tên, phòng chờ, bàn cờ, thẻ và sang ngày mới ổn.'); }catch(err){ console.error('[SELF TEST FAIL]',err); alert('Self-test phát hiện lỗi: '+(err.message||String(err))); } }
    runSelfTests();
  </script>
</body>
</html>
  </style>
</head>
<body>
  <div class="app">
    <section id="lobby" class="screen active hero">
      <div class="lobby-card">
        <h1 class="title">Đánh Bài Bảo Vệ Thành</h1>
        <p class="subtitle">Game online 2 người bằng mã phòng PeerJS. Đặt tên, tạo hoặc vào phòng, rồi phá thành đối thủ để thắng.</p>
        <div class="lobby-grid">
          <div class="box">
            <h3>Tạo phòng</h3>
            <p class="rules">Chủ phòng là máy chủ trận: xáo bài, xử lý luật và đồng bộ cho khách.</p>
            <input id="hostNameInput" placeholder="Tên của bạn" maxlength="18" />
            <br><br><button id="createRoomBtn">Tạo mã phòng</button>
            <p id="roomCode" class="copy-code"></p>
          </div>
          <div class="box">
            <h3>Vào phòng</h3>
            <input id="guestNameInput" placeholder="Tên của bạn" maxlength="18" />
            <br><br><input id="joinCodeInput" placeholder="Nhập mã phòng" maxlength="20" />
            <br><br><button id="joinRoomBtn">Vào phòng</button>
          </div>
        </div>
        <p id="netStatus" class="status">Chưa kết nối.</p>
        <div class="box" style="margin-top:14px"><h3>Luật chính</h3><div class="rules">Đầu game: 1 kỹ năng thành, 4 công, 2 thủ, 2 trợ. Mỗi ngày mỗi người có 3 lượt đánh/bỏ. Khi cả hai hết lượt, sang ngày mới: rút 2 công, 1 thủ và lật 1 sự kiện. Ngày 1 có sự kiện nhưng không tự rút/bỏ bài trước lượt đầu. Thẻ Khắc Sự Kiện bỏ qua hiệu ứng sự kiện cho người dùng.</div></div>
      </div>
    </section>

    <section id="game" class="screen">
      <div class="topbar">
        <div class="row"><span class="badge">Phòng: <b id="topRoomCode">...</b></span><span class="badge">Bạn là: <b id="playerName">...</b></span></div>
        <div class="row" style="justify-content:center"><span class="badge">Ngày <b id="dayNumber">1</b></span><span class="badge">Lượt: <b id="turnInfo">0/3</b></span></div>
        <div class="row" style="justify-content:flex-end"><span class="badge" id="syncStatus">Đã đồng bộ</span></div>
      </div>
      <div class="main-grid">
        <main class="battlefield">
          <div class="castle-grid" id="castleGrid"></div>
          <div class="event-card"><h3>Sự kiện hôm nay</h3><div id="eventText">Chưa có sự kiện.</div></div>
          <div class="hand-panel"><div class="row" style="justify-content:space-between;margin-bottom:10px"><h3>Bài trên tay</h3><span class="tiny">Chọn thẻ, chọn mục tiêu nếu cần, rồi bấm Đánh thẻ.</span></div><div id="hand" class="hand"></div></div>
        </main>
        <aside class="side">
          <div class="box"><h3>Hành động</h3><div class="actions"><button id="playCardBtn" class="good-btn">Đánh thẻ</button><button id="endActionBtn" class="ghost-btn">Bỏ 1 lượt đánh</button><button id="copyStateBtn" class="ghost-btn">Chép trạng thái debug</button></div><p id="actionHint" class="tiny"></p></div>
          <div class="box"><h3>Chọn mục tiêu</h3><div id="targetList" class="target-list"></div></div>
          <div class="box"><h3>Nhật ký</h3><div id="log" class="log"></div></div>
        </aside>
      </div>
    </section>
  </div>
  <div id="winner" class="winner"><div class="winner-card"><h1 id="winnerText">Chiến thắng!</h1><p id="winnerSub" class="subtitle"></p><button onclick="location.reload()">Chơi lại</button></div></div>

  <script>
    const CARD_LIBRARY = {
      attack: [
        ['A01','Mũi Tên Lửa',8,'Gây 8 sát thương lên thành địch.'],['A02','Máy Bắn Đá',12,'Gây 12 sát thương.'],['A03','Đột Kích Đêm',10,'Gây 10 sát thương, bỏ qua 3 giáp.'],['A04','Kỵ Binh Xung Phong',14,'Gây 14 sát thương.'],['A05','Cầu Lửa',16,'Gây 16 sát thương, tự mất 2 giáp nếu có.'],['A06','Phá Cổng',18,'Gây 18 sát thương nếu mục tiêu có giáp dưới 10, ngược lại gây 10.'],['A07','Tên Độc',7,'Gây 7 sát thương và đặt độc 3 sát thương vào đầu ngày sau.'],['A08','Bão Phi Tiêu',11,'Gây 11 sát thương.'],['A09','Chiến Xa Gỗ',13,'Gây 13 sát thương.'],['A10','Phù Thủy Lửa',15,'Gây 15 sát thương.'],['A11','Bộc Phá Tường',20,'Gây 20 sát thương nhưng bạn bỏ 1 thẻ ngẫu nhiên.'],['A12','Cung Thủ Cao Thành',9,'Gây 9 sát thương và bạn nhận 2 giáp.'],['A13','Hỏa Pháo Cũ',17,'50% gây 17, 50% gây 8.'],['A14','Đội Cảm Tử',19,'Gây 19 sát thương, thành bạn mất 5 HP.'],['A15','Rồng Giấy',6,'Gây 6 sát thương và rút 1 thẻ công.'],['A16','Dao Găm Bóng Tối',5,'Gây 5 sát thương, không bị giảm bởi giáp.'],['A17','Nỏ Liên Châu',12,'Gây 12 sát thương chia thành 2 đợt.'],['A18','Búa Phá Thành',21,'Gây 21 sát thương, chỉ dùng được sau ngày 2.'],['A19','Mưa Đá',10,'Gây 10 sát thương và giảm 4 giáp địch.'],['A20','Thủy Công',13,'Gây 13 sát thương, thêm 5 nếu sự kiện là mưa/lũ/bão.'],['A21','Hầm Ngầm',15,'Gây 15 sát thương, bỏ qua khiên một lần.'],['A22','Sét Đánh Tháp Canh',18,'Gây 18 sát thương, không thể dùng nếu sự kiện Cấm Phép.'],['A23','Hỏa Tiễn',14,'Gây 14 sát thương và đốt 2 HP mỗi lượt đến hết ngày.'],['A24','Quân Tiên Phong',9,'Gây 9 sát thương, nếu là thẻ đầu trong ngày thì gây 15.'],['A25','Lưỡi Cưa Công Thành',16,'Gây 16 sát thương, giảm thêm 6 giáp.'],['A26','Bắn Tỉa Chỉ Huy',7,'Gây 7 sát thương và địch mất 1 lượt đánh trong ngày.'],['A27','Tập Kích Kho Lương',8,'Gây 8 sát thương, địch không được rút 1 thẻ công ngày sau.'],['A28','Golem Đá',22,'Gây 22 sát thương, lượt sau của bạn bị bỏ qua.'],['A29','Đạn Xuyên Giáp',12,'Gây 12 sát thương trực tiếp vào HP.'],['A30','Tổng Công Kích',25,'Gây 25 sát thương, chỉ dùng khi bạn còn dưới 40 HP.']
      ],
      defense: [
        ['D01','Tường Gỗ',0,'Nhận 10 giáp.'],['D02','Tường Đá',0,'Nhận 16 giáp.'],['D03','Hào Nước',0,'Nhận 12 giáp, đòn công kế tiếp giảm thêm 5.'],['D04','Thợ Sửa Thành',0,'Hồi 12 HP.'],['D05','Đội Khiên Lớn',0,'Nhận 20 giáp, hết ngày mất 5 giáp.'],['D06','Chuông Báo Động',0,'Chặn 50% sát thương đòn kế tiếp.'],['D07','Cổng Sắt',0,'Nhận 14 giáp, miễn bỏ qua giáp 1 lần.'],['D08','Tu Sửa Khẩn Cấp',0,'Hồi 18 HP nếu bạn dưới 50 HP, ngược lại hồi 8.'],['D09','Pháp Trận Bảo Hộ',0,'Nhận khiên phép, chặn 1 thẻ phép công.'],['D10','Lính Gác Đêm',0,'Giảm 8 sát thương từ Đột Kích/Dao Găm trong ngày.'],['D11','Kho Đá Dự Phòng',0,'Nhận 8 giáp và rút 1 thẻ thủ.'],['D12','Nâng Cấp Tháp',0,'Tăng HP tối đa thêm 8 và hồi 8.'],['D13','Bẫy Chông',0,'Địch tấn công bạn lần sau tự nhận 6 sát thương.'],['D14','Cầu Treo',0,'Chặn hoàn toàn 1 đòn công dưới 12 sát thương.'],['D15','Hầm Trú Ẩn',0,'Hết ngày này, mọi sát thương vào bạn giảm 4.'],['D16','Lò Rèn Giáp',0,'Nhận 6 giáp, sau đó mọi thẻ thủ hôm nay thêm 4 giáp/hồi.'],['D17','Phù Hiệu Kiên Cường',0,'Nếu bị phá về dưới 1 HP trong ngày này, còn lại 1 HP một lần.'],['D18','Đại Tu Thành',0,'Hồi 25 HP, chỉ dùng được sau ngày 3.']
      ],
      skill: [['S01','Thành Rồng Lửa',0,'HP thành 115. Thẻ công đầu mỗi ngày +3 sát thương.'],['S02','Thành Băng Vĩnh Cửu',0,'HP thành 105. Mỗi ngày nhận sẵn 3 giáp.'],['S03','Thành Thương Nhân',0,'HP thành 100. Mỗi ngày rút thêm 1 thẻ trợ.'],['S04','Thành Cổ Linh Thiêng',0,'HP thành 110. Lần đầu đánh thẻ thủ mỗi ngày hồi 3 HP.'],['S05','Pháo Đài Bóng Đêm',0,'HP thành 95. Thẻ công đầu mỗi ngày bỏ qua 4 giáp.']],
      support: [['U01','Khắc Sự Kiện',0,'Bỏ qua hiệu ứng sự kiện hôm nay cho bạn.'],['U02','Trinh Sát',0,'Rút 1 thẻ công.'],['U03','Tiếp Tế',0,'Rút 1 thẻ công và 1 thẻ thủ.'],['U04','Đổi Gió',0,'Bỏ 1 thẻ ngẫu nhiên, rút 2 thẻ ngẫu nhiên.'],['U05','Thuê Thợ',0,'Thẻ thủ kế tiếp trong ngày mạnh thêm 8.'],['U06','Lệnh Tổng Động Viên',0,'Hôm nay bạn được đánh thêm 1 thẻ.'],['U07','Mưu Kế',0,'Thẻ công kế tiếp trong ngày mạnh thêm 6.'],['U08','Kho Bí Mật',0,'Nhận 5 giáp và rút 1 thẻ trợ.'],['U09','Phản Gián',0,'Xóa độc/cháy/bỏ lượt trên bạn.'],['U10','Hiệp Ước Tạm Thời',0,'Cả hai nhận 6 giáp, bạn rút 1 thẻ.']],
      event: [['E01','Mưa Lớn','Mọi thẻ công giảm 3 sát thương hôm nay.'],['E02','Nắng Gắt','Mọi thẻ thủ hồi/giáp giảm 3 hôm nay.'],['E03','Lễ Hội','Mỗi người rút thêm 1 thẻ trợ ngay.'],['E04','Động Đất','Mỗi thành mất 5 HP trực tiếp.'],['E05','Sương Mù','Thẻ công đầu tiên mỗi người hôm nay có 40% trượt.'],['E06','Mưa Sao Băng','Thẻ công phép tăng 5 sát thương.'],['E07','Cấm Phép','Không thể dùng thẻ có phép/sét/pháp trận hôm nay.'],['E08','Lũ Quét','Mỗi người mất 4 giáp, không âm.'],['E09','Gió Thuận','Thẻ công đầu mỗi người tăng 4 sát thương.'],['E10','Dịch Bệnh','Không thể hồi quá 8 HP mỗi lần hôm nay.'],['E11','Đêm Không Trăng','Đột kích/dao găm tăng 5 sát thương.'],['E12','Tăng Thuế','Mỗi người bỏ 1 thẻ ngẫu nhiên nếu có hơn 6 thẻ.'],['E13','Kho Lương Đầy','Mỗi người rút 1 thẻ thủ ngay.'],['E14','Thợ Giỏi Đến Thành','Thẻ thủ đầu hôm nay tăng 6.'],['E15','Quái Vật Lang Thang','Cuối ngày, người có ít giáp hơn mất 6 HP.'],['E16','Cổng Dịch Chuyển','Thẻ công đầu hôm nay bỏ qua 5 giáp.'],['E17','Mất Liên Lạc','Mỗi người chỉ được giữ tối đa 8 thẻ cuối ngày.'],['E18','Linh Khí Dồi Dào','Kỹ năng thành mạnh gấp đôi hôm nay nếu có hiệu ứng ngày.'],['E19','Bão Cát','Không có hiệu ứng đặc biệt trong bản 2 người.'],['E20','Bội Thu','Đầu ngày sau rút thêm 1 công.'],['E21','Kẻ Trộm','Mỗi người mất 1 thẻ trợ ngẫu nhiên nếu có.'],['E22','Cầu Vồng','Mọi hồi HP tăng 4.'],['E23','Rạn Nứt Tường Thành','Mọi sát thương trực tiếp tăng 3.'],['E24','Hội Chợ Vũ Khí','Thẻ công dưới 10 sát thương tăng 5.'],['E25','Ngày Bình Yên','Không có hiệu ứng xấu. Mỗi người hồi 3 HP.'],['E26','Lời Nguyền','Ai đánh quá 2 thẻ hôm nay mất 4 HP.'],['E27','Sấm Sét','Thẻ sét gây thêm 7, nhưng 30% phản 3 sát thương.'],['E28','Quân Nổi Loạn','Người có HP cao hơn mất 6 HP.'],['E29','Đội Buôn Đi Qua','Mỗi người rút 1 thẻ công.'],['E30','Trăng Máu','Mọi sát thương tăng 4, mọi hồi máu giảm 4.']]
    };

    const TYPE_LABEL={attack:'Tấn công',defense:'Phòng thủ',support:'Bổ trợ'};
    const state={peer:null,conn:null,isHost:false,myId:null,roomCode:'',selectedCardId:null,selectedTarget:null,game:null,connected:false,localName:''};
    const $=id=>document.getElementById(id);
    function assert(ok,msg){if(!ok)throw new Error(msg)}
    function escapeHtml(str){return String(str).replace(/[&<>'"]/g,s=>({'&':'&amp;','<':'&lt;','>':'&gt;',"'":'&#39;','"':'&quot;'}[s]))}
    function sanitizeName(name,fallback='Người chơi'){const cleaned=String(name||'').trim().split(/\s+/).join(' ').slice(0,18);return cleaned||fallback}
    function makeCard(type,raw){return{id:raw[0],uid:raw[0]+'-'+Math.random().toString(36).slice(2,9),type,name:raw[1],power:raw[2],desc:raw[3]}}
    function shuffle(arr){const a=[...arr];for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]]}return a}
    function draw(deck,count){const out=[];for(let i=0;i<count;i++){if(deck.length)out.push(deck.shift())}return out}
    function clamp(n,min,max){return Math.max(min,Math.min(max,n))}
    function createDecks(){return{attackDeck:shuffle(CARD_LIBRARY.attack.map(c=>makeCard('attack',c))),defenseDeck:shuffle(CARD_LIBRARY.defense.map(c=>makeCard('defense',c))),supportDeck:shuffle(CARD_LIBRARY.support.flatMap(c=>[makeCard('support',c),makeCard('support',c),makeCard('support',c)])),skillDeck:shuffle(CARD_LIBRARY.skill.map(c=>makeCard('skill',c))),eventDeck:shuffle(CARD_LIBRARY.event.map(c=>({id:c[0],name:c[1],desc:c[2]})))}}
    function createEffects(){return{nextAttackBonus:0,nextDefenseBonus:0,nextIncomingHalf:false,nextIncomingSmallBlock:false,moat:false,ironGate:false,magicShield:false,nightGuard:false,shelter:false,forge:false,thorns:false,endure:false,poison:0,burn:0,skipActions:0,loseAttackDraws:0,bonusAttackNextDay:0,extraLimit:0,firstAttackDone:false,firstDefenseDone:false,firstFogAttackDone:false}}
    function ensureEffects(p){p.effects={...createEffects(),...(p.effects||{})}}
    function makePlayer(id,name,skillDeck,attackDeck,defenseDeck,supportDeck){const skill=draw(skillDeck,1)[0];let maxHp=100;if(skill.id==='S01')maxHp=115;if(skill.id==='S02')maxHp=105;if(skill.id==='S04')maxHp=110;if(skill.id==='S05')maxHp=95;return{id,name:sanitizeName(name,id==='host'?'Chủ phòng':'Khách'),skill,hp:maxHp,maxHp,armor:0,hand:[...draw(attackDeck,4),...draw(defenseDeck,2),...draw(supportDeck,2)],playedToday:0,skipEventToday:false,effects:createEffects()}}
    function createInitialGame(hostName='Chủ phòng',guestName='Khách'){const decks=createDecks();const game={version:3,day:1,perDayLimit:3,currentEvent:null,decks:{attackDeck:decks.attackDeck,defenseDeck:decks.defenseDeck,supportDeck:decks.supportDeck,eventDeck:decks.eventDeck},players:[makePlayer('host',hostName,decks.skillDeck,decks.attackDeck,decks.defenseDeck,decks.supportDeck),makePlayer('guest',guestName,decks.skillDeck,decks.attackDeck,decks.defenseDeck,decks.supportDeck)],log:[],winner:null,seedNote:Date.now()};startNewDay(game,true);log(game,'Trận đấu bắt đầu. Mỗi người nhận 1 kỹ năng thành, 4 công, 2 thủ, 2 trợ.');validateGame(game);return game}
    function getMe(){return state.game.players.find(p=>p.id===state.myId)}
    function getOpponent(){return state.game.players.find(p=>p.id!==state.myId)}
    function opponentOf(game,p){return game.players.find(x=>x.id!==p.id)}
    function log(game,text){game.log.unshift(text);game.log=game.log.slice(0,80)}
    function validateUniqueIds(list,label){const ids=list.map(x=>x[0]);assert(ids.length===new Set(ids).size,label+' có ID bị trùng.')}
    function validateLibrary(){assert(CARD_LIBRARY.attack.length===30,'Phải có đúng 30 thẻ công.');assert(CARD_LIBRARY.defense.length===18,'Phải có đúng 18 thẻ thủ.');assert(CARD_LIBRARY.skill.length===5,'Phải có đúng 5 thẻ kỹ năng.');assert(CARD_LIBRARY.event.length===30,'Phải có đúng 30 thẻ sự kiện.');assert(CARD_LIBRARY.support.length>=1,'Phải có ít nhất 1 thẻ bổ trợ.');validateUniqueIds(CARD_LIBRARY.attack,'Thẻ công');validateUniqueIds(CARD_LIBRARY.defense,'Thẻ thủ');validateUniqueIds(CARD_LIBRARY.skill,'Thẻ kỹ năng');validateUniqueIds(CARD_LIBRARY.support,'Thẻ bổ trợ');validateUniqueIds(CARD_LIBRARY.event,'Thẻ sự kiện')}
    function validateGame(game){assert(game&&Array.isArray(game.players)&&game.players.length===2,'Game phải có đúng 2 người chơi.');game.players.forEach(ensureEffects);assert(game.players.every(p=>p.id&&p.name&&p.skill),'Thiếu dữ liệu người chơi.');assert(game.players.every(p=>typeof p.name==='string'&&p.name.length>0&&p.name.length<=18),'Tên người chơi không hợp lệ.');assert(game.players.every(p=>Number.isFinite(p.hp)&&Number.isFinite(p.maxHp)&&p.hp>=0&&p.hp<=p.maxHp),'HP người chơi không hợp lệ.');assert(game.players.every(p=>Number.isFinite(p.armor)&&p.armor>=0),'Giáp người chơi không hợp lệ.');assert(game.players.every(p=>Array.isArray(p.hand)),'Tay bài không hợp lệ.');assert(game.decks&&game.decks.attackDeck&&game.decks.defenseDeck&&game.decks.supportDeck&&game.decks.eventDeck,'Thiếu chồng bài.');assert(game.currentEvent&&game.currentEvent.id&&game.currentEvent.name,'Thiếu sự kiện hiện tại.');return true}
    function drawEvent(game){let e=draw(game.decks.eventDeck,1)[0];if(!e){game.decks.eventDeck=shuffle(CARD_LIBRARY.event.map(c=>({id:c[0],name:c[1],desc:c[2]})));e=draw(game.decks.eventDeck,1)[0]}return e}
    function startNewDay(game,firstDay=false){if(!firstDay)game.day++;game.currentEvent=drawEvent(game);for(const p of game.players){ensureEffects(p);p.playedToday=0;p.skipEventToday=false;p.effects.firstAttackDone=false;p.effects.firstDefenseDone=false;p.effects.firstFogAttackDone=false;if(!firstDay){const n=clamp(2+p.effects.bonusAttackNextDay-p.effects.loseAttackDraws,0,5);p.hand.push(...draw(game.decks.attackDeck,n));p.hand.push(...draw(game.decks.defenseDeck,1));p.effects.bonusAttackNextDay=0;p.effects.loseAttackDraws=0;if(p.skill.id==='S03')p.hand.push(...draw(game.decks.supportDeck,1));if(p.effects.poison){applyDirectDamage(game,p,p.effects.poison,'độc');p.effects.poison=0}if(p.effects.burn)applyDirectDamage(game,p,p.effects.burn,'cháy')}if(p.skill.id==='S02')p.armor+=3}if(!firstDay)applyStartEvent(game,game.currentEvent);log(game,`Ngày ${game.day}: Sự kiện "${game.currentEvent.name}" - ${game.currentEvent.desc}`);checkWinner(game)}
    function applyStartEvent(game,event){const active=game.players.filter(p=>!p.skipEventToday);switch(event.id){case'E03':active.forEach(p=>p.hand.push(...draw(game.decks.supportDeck,1)));break;case'E04':active.forEach(p=>applyDirectDamage(game,p,5,'động đất'));break;case'E08':active.forEach(p=>p.armor=Math.max(0,p.armor-4));break;case'E12':active.forEach(p=>{if(p.hand.length>6)discardRandom(p)});break;case'E13':active.forEach(p=>p.hand.push(...draw(game.decks.defenseDeck,1)));break;case'E20':active.forEach(p=>p.effects.bonusAttackNextDay++);break;case'E21':active.forEach(p=>discardRandomType(p,'support'));break;case'E25':active.forEach(p=>heal(game,p,3));break;case'E28':{const[a,b]=game.players;if(a.hp>b.hp)applyDirectDamage(game,a,6,'quân nổi loạn');else if(b.hp>a.hp)applyDirectDamage(game,b,6,'quân nổi loạn');break}case'E29':active.forEach(p=>p.hand.push(...draw(game.decks.attackDeck,1)));break}}
    function applyEndDayEvent(game){const e=game.currentEvent;if(!e)return;if(e.id==='E15'){const[a,b]=game.players;if(a.armor<b.armor)applyDirectDamage(game,a,6,'quái vật');else if(b.armor<a.armor)applyDirectDamage(game,b,6,'quái vật')}if(e.id==='E17')game.players.filter(p=>!p.skipEventToday).forEach(p=>{while(p.hand.length>8)discardRandom(p)})}
    function discardRandom(p){if(!p.hand.length)return null;const i=Math.floor(Math.random()*p.hand.length);return p.hand.splice(i,1)[0]}
    function discardRandomType(p,type){const cards=p.hand.map((c,i)=>({c,i})).filter(x=>x.c.type===type);if(!cards.length)return null;const pick=cards[Math.floor(Math.random()*cards.length)];return p.hand.splice(pick.i,1)[0]}
    function heal(game,p,amount){let v=amount,e=game.currentEvent;if(e&&!p.skipEventToday){if(e.id==='E10')v=Math.min(v,8);if(e.id==='E22')v+=4;if(e.id==='E30')v=Math.max(0,v-4)}p.hp=clamp(p.hp+v,0,p.maxHp);return v}
    function applyDirectDamage(game,target,amount,source='sát thương trực tiếp'){let dmg=amount;if(game.currentEvent&&!target.skipEventToday&&game.currentEvent.id==='E23')dmg+=3;target.hp=clamp(target.hp-dmg,0,target.maxHp);log(game,`${target.name} mất ${dmg} HP do ${source}.`);checkWinner(game);return dmg}
    function applyAttackDamage(game,attacker,target,baseDamage,card){let damage=baseDamage;const e=game.currentEvent,active=e&&!attacker.skipEventToday;if(attacker.effects.skipActions>0){attacker.effects.skipActions--;log(game,`${attacker.name} bị mất lượt nên ${card.name} không kích hoạt.`);return 0}if(active){if(e.id==='E01')damage-=3;if(e.id==='E06'&&/Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name))damage+=5;if(e.id==='E09'&&!attacker.effects.firstAttackDone)damage+=4;if(e.id==='E11'&&/Đột Kích|Dao Găm/i.test(card.name))damage+=5;if(e.id==='E16'&&!attacker.effects.firstAttackDone)card.tempPierce=(card.tempPierce||0)+5;if(e.id==='E23'&&/trực tiếp|xuyên/i.test(card.desc))damage+=3;if(e.id==='E24'&&baseDamage<10)damage+=5;if(e.id==='E27'&&/Sét/i.test(card.name)){damage+=7;if(Math.random()<.3)applyDirectDamage(game,attacker,3,'sét phản')}if(e.id==='E30')damage+=4;if(e.id==='E05'&&!attacker.effects.firstFogAttackDone){attacker.effects.firstFogAttackDone=true;if(Math.random()<.4){log(game,`${card.name} của ${attacker.name} trượt vì sương mù.`);return 0}}}if(attacker.skill.id==='S01'&&!attacker.effects.firstAttackDone)damage+=e?.id==='E18'?6:3;if(attacker.skill.id==='S05'&&!attacker.effects.firstAttackDone)card.tempPierce=(card.tempPierce||0)+(e?.id==='E18'?8:4);if(attacker.effects.nextAttackBonus){damage+=attacker.effects.nextAttackBonus;attacker.effects.nextAttackBonus=0}attacker.effects.firstAttackDone=true;if(target.effects.nightGuard&&/Đột Kích|Dao Găm/i.test(card.name))damage-=8;if(target.effects.shelter)damage-=4;if(target.effects.nextIncomingHalf){damage=Math.ceil(damage/2);target.effects.nextIncomingHalf=false}if(target.effects.nextIncomingSmallBlock&&damage<12){log(game,`${target.name} chặn hoàn toàn ${card.name}.`);target.effects.nextIncomingSmallBlock=false;return 0}if(target.effects.moat){damage-=5;target.effects.moat=false}if(target.effects.magicShield&&/Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)){log(game,`Khiên phép của ${target.name} chặn ${card.name}.`);target.effects.magicShield=false;return 0}if(target.effects.thorns){applyDirectDamage(game,attacker,6,'bẫy chông');target.effects.thorns=false}damage=Math.max(0,damage);let pierce=card.tempPierce||0;if(/bỏ qua 3 giáp/i.test(card.desc))pierce+=3;if(/bỏ qua khiên/i.test(card.desc))pierce+=999;if(/trực tiếp vào HP/i.test(card.desc)||card.id==='A16')pierce+=999;if(target.effects.ironGate){pierce=0;target.effects.ironGate=false}const usableArmor=Math.max(0,target.armor-pierce),blocked=Math.min(usableArmor,damage);target.armor=Math.max(0,target.armor-blocked);const hpDamage=damage-blocked;target.hp=clamp(target.hp-hpDamage,0,target.maxHp);log(game,`${attacker.name} dùng ${card.name} gây ${hpDamage} HP, bị chặn ${blocked} bởi giáp.`);checkWinner(game);return hpDamage}
    function applyCard(game,playerId,cardUid,targetId){validateGame(game);if(game.winner)throw new Error('Trận đấu đã kết thúc.');const p=game.players.find(x=>x.id===playerId);assert(p,'Không tìm thấy người chơi.');ensureEffects(p);assert(p.playedToday<game.perDayLimit+p.effects.extraLimit,'Bạn đã hết lượt đánh trong ngày.');const i=p.hand.findIndex(c=>c.uid===cardUid);assert(i>=0,'Không tìm thấy thẻ trên tay.');const card=p.hand[i];if(game.currentEvent&&!p.skipEventToday&&game.currentEvent.id==='E07'&&/Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name))throw new Error('Sự kiện Cấm Phép đang chặn thẻ này.');if(card.id==='A18'&&game.day<=2)throw new Error('Búa Phá Thành chỉ dùng sau ngày 2.');if(card.id==='A30'&&p.hp>=40)throw new Error('Tổng Công Kích chỉ dùng khi bạn dưới 40 HP.');if(card.id==='D18'&&game.day<=3)throw new Error('Đại Tu Thành chỉ dùng sau ngày 3.');p.hand.splice(i,1);p.playedToday++;const target=targetId?game.players.find(x=>x.id===targetId):opponentOf(game,p);assert(target,'Mục tiêu không hợp lệ.');ensureEffects(target);if(card.type==='attack')handleAttackCard(game,p,target,card);if(card.type==='defense')handleDefenseCard(game,p,card);if(card.type==='support')handleSupportCard(game,p,card);if(game.currentEvent&&!p.skipEventToday&&game.currentEvent.id==='E26'&&p.playedToday>2)applyDirectDamage(game,p,4,'lời nguyền đánh quá 2 thẻ');checkDayEnd(game);validateGame(game)}
    function handleAttackCard(game,p,t,card){let d=card.power;switch(card.id){case'A05':p.armor=Math.max(0,p.armor-2);break;case'A06':d=t.armor<10?18:10;break;case'A07':t.effects.poison+=3;break;case'A11':discardRandom(p);break;case'A12':p.armor+=2;break;case'A13':d=Math.random()<.5?17:8;break;case'A14':applyDirectDamage(game,p,5,'đội cảm tử');break;case'A15':p.hand.push(...draw(game.decks.attackDeck,1));break;case'A19':t.armor=Math.max(0,t.armor-4);break;case'A20':if(game.currentEvent&&/Mưa|Lũ|Bão/i.test(game.currentEvent.name))d+=5;break;case'A23':t.effects.burn+=2;break;case'A24':if(p.playedToday===1)d=15;break;case'A25':t.armor=Math.max(0,t.armor-6);break;case'A26':t.effects.skipActions++;break;case'A27':t.effects.loseAttackDraws++;break;case'A28':p.effects.skipActions++;break}applyAttackDamage(game,p,t,d,card)}
    function handleDefenseCard(game,p,card){let gain=0,healed=0,e=game.currentEvent,active=e&&!p.skipEventToday,bonus=p.effects.nextDefenseBonus+(p.effects.forge?4:0);if(active){if(e.id==='E02')gain-=3;if(e.id==='E14'&&!p.effects.firstDefenseDone)gain+=6}switch(card.id){case'D01':gain+=10;break;case'D02':gain+=16;break;case'D03':gain+=12;p.effects.moat=true;break;case'D04':healed=heal(game,p,12);break;case'D05':gain+=20;break;case'D06':p.effects.nextIncomingHalf=true;break;case'D07':gain+=14;p.effects.ironGate=true;break;case'D08':healed=heal(game,p,p.hp<50?18:8);break;case'D09':p.effects.magicShield=true;break;case'D10':p.effects.nightGuard=true;break;case'D11':gain+=8;p.hand.push(...draw(game.decks.defenseDeck,1));break;case'D12':p.maxHp+=8;healed=heal(game,p,8);break;case'D13':p.effects.thorns=true;break;case'D14':p.effects.nextIncomingSmallBlock=true;break;case'D15':p.effects.shelter=true;break;case'D16':gain+=6;p.effects.forge=true;break;case'D17':p.effects.endure=true;break;case'D18':healed=heal(game,p,25);break}if(p.skill.id==='S04')healed+=heal(game,p,e?.id==='E18'?6:3);gain=Math.max(0,gain+bonus);p.armor+=gain;p.effects.nextDefenseBonus=0;p.effects.firstDefenseDone=true;log(game,`${p.name} dùng ${card.name}: +${gain} giáp, hồi ${healed} HP.`)}
    function handleSupportCard(game,p,card){switch(card.id){case'U01':p.skipEventToday=true;log(game,`${p.name} dùng Khắc Sự Kiện.`);break;case'U02':p.hand.push(...draw(game.decks.attackDeck,1));break;case'U03':p.hand.push(...draw(game.decks.attackDeck,1),...draw(game.decks.defenseDeck,1));break;case'U04':discardRandom(p);p.hand.push(...drawRandomCards(game,2));break;case'U05':p.effects.nextDefenseBonus+=8;break;case'U06':p.effects.extraLimit++;break;case'U07':p.effects.nextAttackBonus+=6;break;case'U08':p.armor+=5;p.hand.push(...draw(game.decks.supportDeck,1));break;case'U09':p.effects.poison=0;p.effects.burn=0;p.effects.skipActions=0;break;case'U10':game.players.forEach(x=>x.armor+=6);p.hand.push(...drawRandomCards(game,1));break}log(game,`${p.name} dùng ${card.name}.`)}
    function drawRandomCards(game,count){const out=[];for(let i=0;i<count;i++){const keys=['attackDeck','defenseDeck','supportDeck'].filter(k=>game.decks[k].length);if(!keys.length)break;out.push(...draw(game.decks[keys[Math.floor(Math.random()*keys.length)]],1))}return out}
    function skipAction(game,playerId){const p=game.players.find(x=>x.id===playerId);if(!p||game.winner)return;ensureEffects(p);if(p.playedToday>=game.perDayLimit+p.effects.extraLimit)return;p.playedToday++;log(game,`${p.name} bỏ 1 lượt đánh.`);checkDayEnd(game)}
    function checkDayEnd(game){if(game.players.every(p=>p.playedToday>=game.perDayLimit+p.effects.extraLimit)&&!game.winner){applyEndDayEvent(game);for(const p of game.players){p.effects.extraLimit=0;p.effects.shelter=false;p.effects.nightGuard=false;p.effects.forge=false;p.effects.burn=0;if(p.armor>0)p.armor=Math.floor(p.armor*.75)}if(!game.winner)startNewDay(game,false)}}
    function checkWinner(game){for(const p of game.players){if(p.hp<=0&&p.effects.endure){p.hp=1;p.effects.endure=false;log(game,`${p.name} sống sót với 1 HP.`)}}const dead=game.players.find(p=>p.hp<=0);if(dead){const win=game.players.find(p=>p.id!==dead.id);game.winner=win.id;log(game,`${win.name} đã phá thành ${dead.name} và chiến thắng!`)}}
    function render(){if(!state.game)return;const game=state.game,me=getMe(),opp=getOpponent();if(!me||!opp)return;$('dayNumber').textContent=game.day;$('turnInfo').textContent=`${me.playedToday}/${game.perDayLimit+me.effects.extraLimit}`;$('playerName').textContent=me.name;$('topRoomCode').textContent=state.roomCode||'(offline)';$('eventText').innerHTML=game.currentEvent?`<b>${escapeHtml(game.currentEvent.name)}</b>: ${escapeHtml(game.currentEvent.desc)}`:'Chưa có sự kiện.';$('syncStatus').textContent=state.connected?'Online':'Offline / đang chờ';renderCastles(me,opp);renderHand(me);renderTargets(opp);renderLog(game);const canPlay=!game.winner&&me.playedToday<game.perDayLimit+me.effects.extraLimit;$('playCardBtn').disabled=!canPlay||!state.selectedCardId;$('endActionBtn').disabled=!canPlay;$('actionHint').textContent=canPlay?'Bạn còn lượt đánh trong ngày.':'Bạn đã hết lượt hôm nay.';if(game.winner)showWinner(game.winner===state.myId)}
    function renderCastles(me,opp){$('castleGrid').innerHTML=[me,opp].map(p=>{const pct=clamp(p.hp/p.maxHp*100,0,100);return`<div class="castle ${p.id===state.myId?'current':'enemy'}"><h2>${escapeHtml(p.name)}<span>${p.hp}/${p.maxHp} HP</span></h2><div class="hpbar"><div class="hpfill" style="width:${pct}%"></div></div><p class="tiny"><b>${escapeHtml(p.skill.name)}</b>: ${escapeHtml(p.skill.desc)}</p><div class="stat-grid"><div class="stat">Giáp<b>${p.armor}</b></div><div class="stat">Bài trên tay<b>${p.hand.length}</b></div><div class="stat">Đã đánh hôm nay<b>${p.playedToday}</b></div><div class="stat">Bỏ sự kiện<b>${p.skipEventToday?'Có':'Không'}</b></div></div></div>`}).join('')}
    function renderHand(me){$('hand').innerHTML=me.hand.map(card=>`<div class="card ${card.type} ${state.selectedCardId===card.uid?'selected':''}" onclick="selectCard('${card.uid}')"><div class="type">${TYPE_LABEL[card.type]}</div><div class="name">${escapeHtml(card.name)}</div><div class="desc">${escapeHtml(card.desc)}</div><div class="power"><span>${card.power?'Sức mạnh':'Hiệu ứng'}</span><b>${card.power||'—'}</b></div></div>`).join('')}
    function renderTargets(opp){const me=getMe(),targets=[opp,me];if(!state.selectedTarget||!targets.some(t=>t.id===state.selectedTarget))state.selectedTarget=opp.id;$('targetList').innerHTML=targets.map(t=>`<button class="target-btn ${state.selectedTarget===t.id?'active':''}" onclick="selectTarget('${t.id}')"><span>${t.id===state.myId?'Bản thân':'Đối thủ'}</span><b>${t.hp} HP</b></button>`).join('')}
    function renderLog(game){$('log').innerHTML=game.log.map(x=>`<div class="log-entry">${escapeHtml(x)}</div>`).join('')}
    window.selectCard=function(uid){const me=getMe();if(!me)return;state.selectedCardId=state.selectedCardId===uid?null:uid;const card=me.hand.find(c=>c.uid===state.selectedCardId);if(card&&card.type!=='attack')state.selectedTarget=state.myId;if(card&&card.type==='attack')state.selectedTarget=getOpponent().id;render()};window.selectTarget=function(id){state.selectedTarget=id;render()};
    function localAction(type,payload={}){if(!state.game||state.game.winner)return;try{if(type==='play')applyCard(state.game,state.myId,payload.cardUid,payload.targetId);if(type==='skip')skipAction(state.game,state.myId);state.selectedCardId=null;validateGame(state.game);send({type:'state',game:state.game});render()}catch(err){alert(err.message||'Có lỗi khi đánh thẻ.');render()}}
    function send(msg){if(state.conn&&state.conn.open)state.conn.send(JSON.parse(JSON.stringify(msg)))}
    function setupConnection(conn){state.conn=conn;conn.on('open',()=>{state.connected=true;$('netStatus').textContent='Đã kết nối với bạn bè.';if(state.isHost){state.myId='host';state.game=createInitialGame(state.localName||'Chủ phòng','Khách');send({type:'state',game:state.game,roomCode:state.roomCode});showGame()}else send({type:'hello',name:state.localName||'Khách'});render()});conn.on('data',data=>{try{if(data.type==='hello'&&state.isHost&&state.game){const guest=state.game.players.find(p=>p.id==='guest');if(guest){guest.name=sanitizeName(data.name,'Khách');log(state.game,`${guest.name} đã vào phòng.`);send({type:'state',game:state.game,roomCode:state.roomCode});render()}}if(data.type==='state'){state.game=data.game;if(data.roomCode)state.roomCode=data.roomCode;if(!state.myId)state.myId=state.isHost?'host':'guest';validateGame(state.game);showGame();render()}}catch(err){console.error(err);alert('Dữ liệu nhận bị lỗi: '+err.message)}});conn.on('close',()=>{state.connected=false;$('netStatus').textContent='Kết nối đã đóng.';render()});conn.on('error',err=>{state.connected=false;$('netStatus').textContent='Lỗi kết nối: '+err.message;render()})}
    function showGame(){$('lobby').classList.remove('active');$('game').classList.add('active')}function showWinner(isMe){$('winner').classList.add('active');$('winnerText').textContent=isMe?'Bạn đã thắng!':'Bạn đã thua!';$('winnerSub').textContent=isMe?'Bạn đã phá xong thành đối thủ.':'Thành của bạn đã bị phá.'}
    $('createRoomBtn').addEventListener('click',()=>{const code='thanh-'+Math.random().toString(36).slice(2,7);state.roomCode=code;state.isHost=true;state.myId='host';state.localName=sanitizeName($('hostNameInput').value,'Chủ phòng');$('roomCode').textContent=code;$('netStatus').textContent='Đang tạo phòng...';state.peer=new Peer(code,{debug:1});state.peer.on('open',()=>$('netStatus').textContent='Phòng đã tạo. Gửi mã cho bạn bè.');state.peer.on('connection',conn=>setupConnection(conn));state.peer.on('error',err=>$('netStatus').textContent='Lỗi PeerJS: '+err.message)});
    $('joinRoomBtn').addEventListener('click',()=>{const code=$('joinCodeInput').value.trim();if(!code)return alert('Nhập mã phòng trước.');state.roomCode=code;state.isHost=false;state.myId='guest';state.localName=sanitizeName($('guestNameInput').value,'Khách');$('netStatus').textContent='Đang vào phòng...';state.peer=new Peer(undefined,{debug:1});state.peer.on('open',()=>setupConnection(state.peer.connect(code,{reliable:true})));state.peer.on('error',err=>$('netStatus').textContent='Lỗi PeerJS: '+err.message)});
    $('playCardBtn').addEventListener('click',()=>{if(state.selectedCardId)localAction('play',{cardUid:state.selectedCardId,targetId:state.selectedTarget})});$('endActionBtn').addEventListener('click',()=>localAction('skip'));$('copyStateBtn').addEventListener('click',async()=>{const text=JSON.stringify(state.game,null,2);try{await navigator.clipboard.writeText(text);alert('Đã chép trạng thái debug.')}catch{prompt('Copy trạng thái debug:',text)}});
    function getCardCounts(p){return p.hand.reduce((a,c)=>{a[c.type]=(a[c.type]||0)+1;return a},{})}
    function runSelfTests(){try{validateLibrary();assert(sanitizeName('', 'Mặc định')==='Mặc định','Tên rỗng phải dùng tên mặc định.');assert(sanitizeName('   A   B   ','X')==='A B','Tên phải được cắt khoảng trắng thừa.');assert(sanitizeName('12345678901234567890','X').length===18,'Tên phải bị giới hạn 18 ký tự.');const named=createInitialGame('An Hải','Bạn Thân');assert(named.players[0].name==='An Hải','Tên chủ phòng phải được áp dụng.');assert(named.players[1].name==='Bạn Thân','Tên khách phải được áp dụng.');validateGame(named);for(let i=0;i<80;i++){const g=createInitialGame();validateGame(g);assert(g.players.every(p=>p.hand.length===8),'Đầu game mỗi người phải có đúng 8 thẻ thường.');assert(g.players.every(p=>p.skill&&p.skill.type==='skill'),'Mỗi người phải có 1 kỹ năng thành.');assert(g.currentEvent,'Ngày 1 phải có sự kiện.');assert(g.decks.eventDeck.length===29,'Sau khi lật sự kiện ngày 1, chồng sự kiện phải còn 29 thẻ.');for(const p of g.players){const c=getCardCounts(p);assert(c.attack===4,'Đầu game mỗi người phải có 4 thẻ công.');assert(c.defense===2,'Đầu game mỗi người phải có 2 thẻ thủ.');assert(c.support===2,'Đầu game mỗi người phải có 2 thẻ trợ.');assert(p.playedToday===0,'Đầu game chưa ai được tính là đã đánh thẻ.')}const p=g.players[0],enemy=g.players[1],atk=p.hand.find(c=>c.type==='attack'&&!['A18','A30'].includes(c.id));if(atk)applyCard(g,p.id,atk.uid,enemy.id);validateGame(g)}const g2=createInitialGame();const before=g2.day;const handsBefore=g2.players.map(p=>p.hand.length);g2.players.forEach(p=>{p.playedToday=g2.perDayLimit});checkDayEnd(g2);validateGame(g2);assert(g2.day===before+1,'Khi cả hai người hết 3 lượt, phải sang ngày mới.');assert(g2.players.every(p=>p.playedToday===0),'Sang ngày mới phải reset lượt đã đánh.');assert(g2.currentEvent&&g2.decks.eventDeck.length===28,'Sang ngày mới phải lật thêm 1 sự kiện.');assert(g2.players.every((p,idx)=>p.hand.length>=Math.max(0,handsBefore[idx]-1)),'Sau ngày mới tay bài phải còn hợp lệ dù sự kiện có thể rút/bỏ bài.');console.log('[SELF TEST PASS] Bộ bài, đặt tên, khởi tạo, sự kiện ngày 1, đánh thẻ, sang ngày mới đều ổn.')}catch(err){console.error('[SELF TEST FAIL]',err);alert('Self-test phát hiện lỗi: '+(err&&err.message?err.message:String(err)))}}
    runSelfTests();
  </script>
</body>
</html>
    }

    .title { font-size: clamp(30px, 5vw, 54px); margin: 0 0 10px; line-height: 1.05; }
    .subtitle { margin: 0 0 24px; color: var(--muted); line-height: 1.55; }
    .lobby-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; margin-top: 18px; }

    .box {
      background: var(--panel);
      border: 1px solid var(--line);
      border-radius: 18px;
      padding: 16px;
    }

    .row { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
    .copy-code { font-size: 26px; color: var(--gold); font-weight: 900; letter-spacing: 1px; user-select: all; }
    .status { margin-top: 12px; color: var(--muted); min-height: 24px; }

    .topbar {
      display: grid;
      grid-template-columns: 1fr auto 1fr;
      gap: 12px;
      align-items: center;
      margin-bottom: 14px;
    }

    .badge {
      display: inline-flex;
      align-items: center;
      gap: 6px;
      padding: 8px 10px;
      border-radius: 999px;
      background: rgba(255,255,255,.08);
      border: 1px solid var(--line);
      color: var(--muted);
      font-size: 14px;
    }

    .main-grid { display: grid; grid-template-columns: 1fr 340px; gap: 14px; }
    .battlefield { display: grid; gap: 14px; }
    .castle-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 14px; }

    .castle {
      background: linear-gradient(135deg, rgba(255,255,255,.06), rgba(255,255,255,.02));
      border: 1px solid var(--line);
      border-radius: 22px;
      padding: 16px;
      min-height: 210px;
    }

    .castle.current { outline: 2px solid var(--accent); }
    .castle.enemy { outline: 2px solid rgba(255,107,107,.55); }

    .castle h2 {
      margin: 0 0 8px;
      display: flex;
      justify-content: space-between;
      gap: 10px;
      align-items: center;
      font-size: 22px;
    }

    .hpbar {
      width: 100%;
      height: 16px;
      border-radius: 999px;
      overflow: hidden;
      background: rgba(255,255,255,.1);
      border: 1px solid var(--line);
      margin: 10px 0;
    }

    .hpfill {
      height: 100%;
      width: 100%;
      background: linear-gradient(90deg, var(--danger), var(--gold), var(--good));
      transition: width .25s;
    }

    .stat-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; margin-top: 12px; }
    .stat { background: rgba(0,0,0,.16); border-radius: 14px; padding: 10px; color: var(--muted); font-size: 14px; }
    .stat b { color: var(--text); display: block; font-size: 18px; margin-top: 3px; }

    .event-card {
      background: linear-gradient(135deg, rgba(177,151,252,.22), rgba(100,210,255,.08));
      border: 1px solid rgba(177,151,252,.35);
      border-radius: 20px;
      padding: 16px;
    }

    .event-card h3, .side h3, .hand-panel h3 { margin: 0 0 10px; }
    .hand-panel { background: var(--panel); border: 1px solid var(--line); border-radius: 22px; padding: 14px; }

    .hand {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
      gap: 10px;
      max-height: 410px;
      overflow: auto;
      padding-right: 4px;
    }

    .card {
      position: relative;
      min-height: 172px;
      border-radius: 18px;
      padding: 12px;
      background: var(--panel-2);
      border: 1px solid var(--line);
      display: flex;
      flex-direction: column;
      gap: 7px;
      box-shadow: 0 10px 26px rgba(0,0,0,.18);
    }

    .card.attack { border-color: rgba(255,107,107,.55); }
    .card.defense { border-color: rgba(92,225,165,.55); }
    .card.skill { border-color: rgba(255,209,102,.75); }
    .card.support { border-color: rgba(100,210,255,.55); }
    .card .type { font-size: 12px; color: var(--muted); text-transform: uppercase; letter-spacing: .08em; }
    .card .name { font-size: 16px; font-weight: 900; line-height: 1.2; }
    .card .desc { color: var(--muted); font-size: 13px; line-height: 1.35; flex: 1; }
    .card .power { font-size: 13px; display: flex; justify-content: space-between; color: var(--gold); }
    .card.selected { outline: 3px solid var(--accent); transform: translateY(-2px); }

    .side { display: grid; gap: 14px; align-content: start; }
    .log { max-height: 340px; overflow: auto; display: grid; gap: 8px; color: var(--muted); font-size: 14px; line-height: 1.4; }
    .log-entry { padding: 9px 10px; background: rgba(0,0,0,.14); border-radius: 12px; border: 1px solid rgba(255,255,255,.06); }
    .actions { display: grid; gap: 10px; }
    .good-btn { background: var(--good); color: #04170e; }
    .ghost-btn { background: transparent; color: var(--text); border: 1px solid var(--line); }
    .target-list { display: grid; gap: 8px; }

    .target-btn {
      width: 100%;
      background: rgba(255,255,255,.08);
      color: var(--text);
      border: 1px solid var(--line);
      text-align: left;
      display: flex;
      justify-content: space-between;
    }

    .target-btn.active { border-color: var(--accent); background: rgba(100,210,255,.16); }

    .winner {
      position: fixed;
      inset: 0;
      display: none;
      place-items: center;
      background: rgba(0,0,0,.72);
      z-index: 10;
      padding: 18px;
    }

    .winner.active { display: grid; }
    .winner-card { width: min(520px, 100%); background: var(--panel); border: 1px solid var(--line); border-radius: 24px; padding: 26px; text-align: center; box-shadow: 0 20px 80px rgba(0,0,0,.5); }
    .rules { color: var(--muted); line-height: 1.55; font-size: 14px; }
    .tiny { color: var(--muted); font-size: 12px; }

    @media (max-width: 920px) {
      .main-grid, .castle-grid, .lobby-grid, .topbar { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
  <div class="app">
    <section id="lobby" class="screen active hero">
      <div class="lobby-card">
        <h1 class="title">Đánh Bài Bảo Vệ Thành</h1>
        <p class="subtitle">Game online 2 người bằng mã phòng PeerJS. Mỗi người có thành riêng, dùng thẻ công để phá thành địch, thẻ thủ để chắn/chữa, thẻ kỹ năng đại diện cho thành, và sự kiện xuất hiện mỗi ngày.</p>

        <div class="lobby-grid">
          <div class="box">
            <h3>Tạo phòng</h3>
            <p class="rules">Chủ phòng tạo mã, gửi mã cho bạn bè. Chủ phòng là người xáo bộ bài và đồng bộ toàn bộ trận để tránh lệch dữ liệu.</p>
            <button id="createRoomBtn">Tạo mã phòng</button>
            <p id="roomCode" class="copy-code"></p>
          </div>

          <div class="box">
            <h3>Vào phòng</h3>
            <input id="joinCodeInput" placeholder="Nhập mã phòng" maxlength="20" />
            <br><br>
            <button id="joinRoomBtn">Vào phòng</button>
          </div>
        </div>

        <p id="netStatus" class="status">Chưa kết nối.</p>

        <div class="box" style="margin-top:14px">
          <h3>Luật chính</h3>
          <div class="rules">
            Đầu game: 1 thẻ kỹ năng thành, 4 thẻ công, 2 thẻ thủ, 2 thẻ trợ. Mỗi người được đánh tối đa 3 thẻ mỗi ngày. Khi cả hai người đã đánh đủ 3 thẻ hoặc bỏ lượt đến hết ngày, sang ngày mới: mỗi người rút 2 công, 1 thủ, và lật 1 sự kiện. Sự kiện ngày 1 được lật để biết luật hôm nay nhưng không làm thay đổi bài khởi đầu trước khi người chơi đánh. Ai phá HP thành đối thủ về 0 trước thì thắng. Thẻ Khắc Sự Kiện có thể bỏ qua sự kiện hiện tại cho người dùng nó.
          </div>
        </div>
      </div>
    </section>

    <section id="game" class="screen">
      <div class="topbar">
        <div class="row">
          <span class="badge">Phòng: <b id="topRoomCode">...</b></span>
          <span class="badge">Bạn là: <b id="playerName">...</b></span>
        </div>
        <div class="row" style="justify-content:center">
          <span class="badge">Ngày <b id="dayNumber">1</b></span>
          <span class="badge">Lượt trong ngày: <b id="turnInfo">0/3</b></span>
        </div>
        <div class="row" style="justify-content:flex-end">
          <span class="badge" id="syncStatus">Đã đồng bộ</span>
        </div>
      </div>

      <div class="main-grid">
        <main class="battlefield">
          <div class="castle-grid" id="castleGrid"></div>

          <div class="event-card">
            <h3>Sự kiện hôm nay</h3>
            <div id="eventText">Chưa có sự kiện.</div>
          </div>

          <div class="hand-panel">
            <div class="row" style="justify-content:space-between; margin-bottom:10px">
              <h3>Bài trên tay</h3>
              <span class="tiny">Chọn 1 thẻ, chọn mục tiêu nếu cần, rồi bấm Đánh thẻ.</span>
            </div>
            <div id="hand" class="hand"></div>
          </div>
        </main>

        <aside class="side">
          <div class="box">
            <h3>Hành động</h3>
            <div class="actions">
              <button id="playCardBtn" class="good-btn">Đánh thẻ</button>
              <button id="endActionBtn" class="ghost-btn">Bỏ 1 lượt đánh</button>
              <button id="copyStateBtn" class="ghost-btn">Chép trạng thái debug</button>
            </div>
            <p id="actionHint" class="tiny"></p>
          </div>

          <div class="box">
            <h3>Chọn mục tiêu</h3>
            <div id="targetList" class="target-list"></div>
          </div>

          <div class="box">
            <h3>Nhật ký</h3>
            <div id="log" class="log"></div>
          </div>
        </aside>
      </div>
    </section>
  </div>

  <div id="winner" class="winner">
    <div class="winner-card">
      <h1 id="winnerText">Chiến thắng!</h1>
      <p id="winnerSub" class="subtitle"></p>
      <button onclick="location.reload()">Chơi lại</button>
    </div>
  </div>

  <script>
    const CARD_LIBRARY = {
      attack: [
        ['A01','Mũi Tên Lửa',8,'Gây 8 sát thương lên thành địch.'],
        ['A02','Máy Bắn Đá',12,'Gây 12 sát thương.'],
        ['A03','Đột Kích Đêm',10,'Gây 10 sát thương, bỏ qua 3 giáp.'],
        ['A04','Kỵ Binh Xung Phong',14,'Gây 14 sát thương.'],
        ['A05','Cầu Lửa',16,'Gây 16 sát thương, tự mất 2 giáp nếu có.'],
        ['A06','Phá Cổng',18,'Gây 18 sát thương nếu mục tiêu có giáp dưới 10, ngược lại gây 10.'],
        ['A07','Tên Độc',7,'Gây 7 sát thương và đặt độc 3 sát thương vào đầu ngày sau.'],
        ['A08','Bão Phi Tiêu',11,'Gây 11 sát thương.'],
        ['A09','Chiến Xa Gỗ',13,'Gây 13 sát thương.'],
        ['A10','Phù Thủy Lửa',15,'Gây 15 sát thương.'],
        ['A11','Bộc Phá Tường',20,'Gây 20 sát thương nhưng bạn bỏ 1 thẻ ngẫu nhiên.'],
        ['A12','Cung Thủ Cao Thành',9,'Gây 9 sát thương và bạn nhận 2 giáp.'],
        ['A13','Hỏa Pháo Cũ',17,'50% gây 17, 50% gây 8.'],
        ['A14','Đội Cảm Tử',19,'Gây 19 sát thương, thành bạn mất 5 HP.'],
        ['A15','Rồng Giấy',6,'Gây 6 sát thương và rút 1 thẻ công.'],
        ['A16','Dao Găm Bóng Tối',5,'Gây 5 sát thương, không bị giảm bởi giáp.'],
        ['A17','Nỏ Liên Châu',12,'Gây 12 sát thương chia thành 2 đợt.'],
        ['A18','Búa Phá Thành',21,'Gây 21 sát thương, chỉ dùng được sau ngày 2.'],
        ['A19','Mưa Đá',10,'Gây 10 sát thương và giảm 4 giáp địch.'],
        ['A20','Thủy Công',13,'Gây 13 sát thương, thêm 5 nếu sự kiện là mưa/lũ/bão.'],
        ['A21','Hầm Ngầm',15,'Gây 15 sát thương, bỏ qua khiên một lần.'],
        ['A22','Sét Đánh Tháp Canh',18,'Gây 18 sát thương, không thể dùng nếu sự kiện Cấm Phép.'],
        ['A23','Hỏa Tiễn',14,'Gây 14 sát thương và đốt 2 HP mỗi lượt đến hết ngày.'],
        ['A24','Quân Tiên Phong',9,'Gây 9 sát thương, nếu là thẻ đầu trong ngày thì gây 15.'],
        ['A25','Lưỡi Cưa Công Thành',16,'Gây 16 sát thương, giảm thêm 6 giáp.'],
        ['A26','Bắn Tỉa Chỉ Huy',7,'Gây 7 sát thương và địch mất 1 lượt đánh trong ngày.'],
        ['A27','Tập Kích Kho Lương',8,'Gây 8 sát thương, địch không được rút 1 thẻ công ngày sau.'],
        ['A28','Golem Đá',22,'Gây 22 sát thương, lượt sau của bạn bị bỏ qua.'],
        ['A29','Đạn Xuyên Giáp',12,'Gây 12 sát thương trực tiếp vào HP.'],
        ['A30','Tổng Công Kích',25,'Gây 25 sát thương, chỉ dùng khi bạn còn dưới 40 HP.']
      ],
      defense: [
        ['D01','Tường Gỗ',0,'Nhận 10 giáp.'],
        ['D02','Tường Đá',0,'Nhận 16 giáp.'],
        ['D03','Hào Nước',0,'Nhận 12 giáp, đòn công kế tiếp giảm thêm 5.'],
        ['D04','Thợ Sửa Thành',0,'Hồi 12 HP.'],
        ['D05','Đội Khiên Lớn',0,'Nhận 20 giáp, hết ngày mất 5 giáp.'],
        ['D06','Chuông Báo Động',0,'Chặn 50% sát thương đòn kế tiếp.'],
        ['D07','Cổng Sắt',0,'Nhận 14 giáp, miễn bỏ qua giáp 1 lần.'],
        ['D08','Tu Sửa Khẩn Cấp',0,'Hồi 18 HP nếu bạn dưới 50 HP, ngược lại hồi 8.'],
        ['D09','Pháp Trận Bảo Hộ',0,'Nhận khiên phép, chặn 1 thẻ phép công.'],
        ['D10','Lính Gác Đêm',0,'Giảm 8 sát thương từ Đột Kích/Dao Găm trong ngày.'],
        ['D11','Kho Đá Dự Phòng',0,'Nhận 8 giáp và rút 1 thẻ thủ.'],
        ['D12','Nâng Cấp Tháp',0,'Tăng HP tối đa thêm 8 và hồi 8.'],
        ['D13','Bẫy Chông',0,'Địch tấn công bạn lần sau tự nhận 6 sát thương.'],
        ['D14','Cầu Treo',0,'Chặn hoàn toàn 1 đòn công dưới 12 sát thương.'],
        ['D15','Hầm Trú Ẩn',0,'Hết ngày này, mọi sát thương vào bạn giảm 4.'],
        ['D16','Lò Rèn Giáp',0,'Nhận 6 giáp, sau đó mọi thẻ thủ hôm nay thêm 4 giáp/hồi.'],
        ['D17','Phù Hiệu Kiên Cường',0,'Nếu bị phá về dưới 1 HP trong ngày này, còn lại 1 HP một lần.'],
        ['D18','Đại Tu Thành',0,'Hồi 25 HP, chỉ dùng được sau ngày 3.']
      ],
      skill: [
        ['S01','Thành Rồng Lửa',0,'HP thành 115. Mỗi ngày đầu tiên bạn đánh thẻ công, cộng 3 sát thương.'],
        ['S02','Thành Băng Vĩnh Cửu',0,'HP thành 105. Mỗi ngày nhận sẵn 3 giáp.'],
        ['S03','Thành Thương Nhân',0,'HP thành 100. Mỗi ngày rút thêm 1 thẻ trợ.'],
        ['S04','Thành Cổ Linh Thiêng',0,'HP thành 110. Một lần mỗi ngày, hồi 3 HP khi đánh thẻ thủ.'],
        ['S05','Pháo Đài Bóng Đêm',0,'HP thành 95. Thẻ công đầu mỗi ngày bỏ qua 4 giáp.']
      ],
      support: [
        ['U01','Khắc Sự Kiện',0,'Bỏ qua hiệu ứng sự kiện hôm nay cho bạn.'],
        ['U02','Trinh Sát',0,'Xem 3 thẻ trên cùng của chồng công, lấy 1.'],
        ['U03','Tiếp Tế',0,'Rút 1 thẻ công và 1 thẻ thủ.'],
        ['U04','Đổi Gió',0,'Bỏ 1 thẻ bất kỳ, rút 2 thẻ ngẫu nhiên.'],
        ['U05','Thuê Thợ',0,'Thẻ thủ kế tiếp trong ngày mạnh thêm 8.'],
        ['U06','Lệnh Tổng Động Viên',0,'Hôm nay bạn được đánh thêm 1 thẻ.'],
        ['U07','Mưu Kế',0,'Thẻ công kế tiếp trong ngày mạnh thêm 6.'],
        ['U08','Kho Bí Mật',0,'Nhận 5 giáp và rút 1 thẻ trợ.'],
        ['U09','Phản Gián',0,'Xóa 1 hiệu ứng độc/cháy/bỏ lượt trên bạn.'],
        ['U10','Hiệp Ước Tạm Thời',0,'Cả hai người nhận 6 giáp, bạn rút 1 thẻ.']
      ],
      event: [
        ['E01','Mưa Lớn','Mọi thẻ công giảm 3 sát thương hôm nay.'],
        ['E02','Nắng Gắt','Mọi thẻ thủ hồi/giáp giảm 3 hôm nay.'],
        ['E03','Lễ Hội','Mỗi người rút thêm 1 thẻ trợ ngay.'],
        ['E04','Động Đất','Mỗi thành mất 5 HP trực tiếp.'],
        ['E05','Sương Mù','Thẻ công đầu tiên mỗi người hôm nay có 40% trượt.'],
        ['E06','Mưa Sao Băng','Thẻ công phép tăng 5 sát thương.'],
        ['E07','Cấm Phép','Không thể dùng thẻ có phép/sét/pháp trận hôm nay.'],
        ['E08','Lũ Quét','Mỗi người mất 4 giáp, không âm.'],
        ['E09','Gió Thuận','Thẻ công đầu mỗi người tăng 4 sát thương.'],
        ['E10','Dịch Bệnh','Không thể hồi quá 8 HP mỗi lần hôm nay.'],
        ['E11','Đêm Không Trăng','Đột kích/dao găm tăng 5 sát thương.'],
        ['E12','Tăng Thuế','Mỗi người bỏ 1 thẻ ngẫu nhiên nếu có hơn 6 thẻ.'],
        ['E13','Kho Lương Đầy','Mỗi người rút 1 thẻ thủ ngay.'],
        ['E14','Thợ Giỏi Đến Thành','Thẻ thủ đầu hôm nay tăng 6.'],
        ['E15','Quái Vật Lang Thang','Cuối ngày, người có ít giáp hơn mất 6 HP.'],
        ['E16','Cổng Dịch Chuyển','Thẻ công đầu hôm nay bỏ qua 5 giáp.'],
        ['E17','Mất Liên Lạc','Mỗi người chỉ được giữ tối đa 8 thẻ cuối ngày.'],
        ['E18','Linh Khí Dồi Dào','Thẻ kỹ năng thành kích hoạt mạnh gấp đôi hôm nay nếu có hiệu ứng ngày.'],
        ['E19','Bão Cát','Không thể chọn mục tiêu khác ngoài đối thủ trực tiếp.'],
        ['E20','Bội Thu','Đầu ngày sau rút thêm 1 công.'],
        ['E21','Kẻ Trộm','Mỗi người mất 1 thẻ trợ ngẫu nhiên nếu có.'],
        ['E22','Cầu Vồng','Mọi hồi HP tăng 4.'],
        ['E23','Rạn Nứt Tường Thành','Mọi sát thương trực tiếp tăng 3.'],
        ['E24','Hội Chợ Vũ Khí','Thẻ công có sát thương dưới 10 tăng 5.'],
        ['E25','Ngày Bình Yên','Không có hiệu ứng xấu. Mỗi người hồi 3 HP.'],
        ['E26','Lời Nguyền','Ai đánh quá 2 thẻ hôm nay mất 4 HP.'],
        ['E27','Sấm Sét','Mỗi thẻ sét gây thêm 7, nhưng 30% phản 3 sát thương.'],
        ['E28','Quân Nổi Loạn','Người có HP cao hơn mất 6 HP.'],
        ['E29','Đội Buôn Đi Qua','Mỗi người chọn rút 1 công hoặc 1 thủ ngẫu nhiên.'],
        ['E30','Trăng Máu','Mọi sát thương tăng 4, mọi hồi máu giảm 4.']
      ]
    };

    const TYPE_LABEL = { attack: 'Tấn công', defense: 'Phòng thủ', skill: 'Kỹ năng thành', support: 'Bổ trợ', event: 'Sự kiện' };

    const state = {
      peer: null,
      conn: null,
      isHost: false,
      myId: null,
      roomCode: '',
      selectedCardId: null,
      selectedTarget: null,
      game: null,
      connected: false
    };

    const $ = (id) => document.getElementById(id);

    function assert(condition, message) {
      if (!condition) throw new Error(message);
    }

    function makeCard(type, raw) {
      return {
        id: raw[0],
        uid: raw[0] + '-' + Math.random().toString(36).slice(2, 9),
        type,
        name: raw[1],
        power: raw[2],
        desc: raw[3]
      };
    }

    function shuffle(arr) {
      const copy = [...arr];
      for (let i = copy.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [copy[i], copy[j]] = [copy[j], copy[i]];
      }
      return copy;
    }

    function draw(deck, count) {
      const out = [];
      for (let i = 0; i < count; i++) {
        if (deck.length) out.push(deck.shift());
      }
      return out;
    }

    function clamp(n, min, max) {
      return Math.max(min, Math.min(max, n));
    }

    function createDecks() {
      return {
        attackDeck: shuffle(CARD_LIBRARY.attack.map(c => makeCard('attack', c))),
        defenseDeck: shuffle(CARD_LIBRARY.defense.map(c => makeCard('defense', c))),
        supportDeck: shuffle(CARD_LIBRARY.support.flatMap(c => [makeCard('support', c), makeCard('support', c), makeCard('support', c)])),
        skillDeck: shuffle(CARD_LIBRARY.skill.map(c => makeCard('skill', c))),
        eventDeck: shuffle(CARD_LIBRARY.event.map(c => ({ id: c[0], name: c[1], desc: c[2] })))
      };
    }

    function createInitialGame() {
      const decks = createDecks();
      const players = [
        makePlayer('host', 'Chủ phòng', decks.skillDeck, decks.attackDeck, decks.defenseDeck, decks.supportDeck),
        makePlayer('guest', 'Khách', decks.skillDeck, decks.attackDeck, decks.defenseDeck, decks.supportDeck)
      ];

      const game = {
        version: 2,
        day: 1,
        perDayLimit: 3,
        currentEvent: null,
        firstDayStartEventApplied: false,
        decks: {
          attackDeck: decks.attackDeck,
          defenseDeck: decks.defenseDeck,
          supportDeck: decks.supportDeck,
          eventDeck: decks.eventDeck
        },
        players,
        log: [],
        winner: null,
        seedNote: Date.now()
      };

      startNewDay(game, true);
      game.log.unshift('Trận đấu bắt đầu. Mỗi người nhận 1 kỹ năng thành, 4 công, 2 thủ, 2 trợ. Sự kiện ngày 1 chỉ ảnh hưởng khi đánh thẻ, không tự rút/bỏ bài trước lượt đầu.');
      validateGame(game);
      return game;
    }

    function makePlayer(id, name, skillDeck, attackDeck, defenseDeck, supportDeck) {
      const skill = draw(skillDeck, 1)[0];
      let maxHp = 100;
      if (skill.id === 'S01') maxHp = 115;
      if (skill.id === 'S02') maxHp = 105;
      if (skill.id === 'S04') maxHp = 110;
      if (skill.id === 'S05') maxHp = 95;

      return {
        id,
        name,
        skill,
        hp: maxHp,
        maxHp,
        armor: 0,
        hand: [...draw(attackDeck, 4), ...draw(defenseDeck, 2), ...draw(supportDeck, 2)],
        playedToday: 0,
        skipEventToday: false,
        effects: createEffects()
      };
    }

    function createEffects() {
      return {
        nextAttackBonus: 0,
        nextDefenseBonus: 0,
        nextIncomingHalf: false,
        nextIncomingSmallBlock: false,
        moat: false,
        ironGate: false,
        magicShield: false,
        nightGuard: false,
        shelter: false,
        forge: false,
        thorns: false,
        endure: false,
        poison: 0,
        burn: 0,
        skipActions: 0,
        loseAttackDraws: 0,
        bonusAttackNextDay: 0,
        extraLimit: 0,
        firstAttackDone: false,
        firstDefenseDone: false,
        firstFogAttackDone: false
      };
    }

    function ensureEffects(player) {
      player.effects = { ...createEffects(), ...(player.effects || {}) };
    }

    function getMe() { return state.game.players.find(p => p.id === state.myId); }
    function getOpponent() { return state.game.players.find(p => p.id !== state.myId); }
    function opponentOf(game, player) { return game.players.find(p => p.id !== player.id); }

    function log(game, text) {
      game.log.unshift(text);
      game.log = game.log.slice(0, 80);
    }

    function validateUniqueIds(list, label) {
      const ids = list.map(x => x[0]);
      const unique = new Set(ids);
      assert(ids.length === unique.size, `${label} có ID bị trùng.`);
    }

    function validateLibrary() {
      assert(CARD_LIBRARY.attack.length === 30, 'Phải có đúng 30 thẻ công.');
      assert(CARD_LIBRARY.defense.length === 18, 'Phải có đúng 18 thẻ thủ.');
      assert(CARD_LIBRARY.skill.length === 5, 'Phải có đúng 5 thẻ kỹ năng.');
      assert(CARD_LIBRARY.event.length === 30, 'Phải có đúng 30 thẻ sự kiện.');
      assert(CARD_LIBRARY.support.length >= 1, 'Phải có ít nhất 1 thẻ bổ trợ.');
      validateUniqueIds(CARD_LIBRARY.attack, 'Thẻ công');
      validateUniqueIds(CARD_LIBRARY.defense, 'Thẻ thủ');
      validateUniqueIds(CARD_LIBRARY.skill, 'Thẻ kỹ năng');
      validateUniqueIds(CARD_LIBRARY.support, 'Thẻ bổ trợ');
      validateUniqueIds(CARD_LIBRARY.event, 'Thẻ sự kiện');
    }

    function validateGame(game) {
      assert(game && Array.isArray(game.players) && game.players.length === 2, 'Game phải có đúng 2 người chơi.');
      assert(game.players.every(p => p.id && p.name && p.skill), 'Thiếu dữ liệu người chơi.');
      game.players.forEach(ensureEffects);
      assert(game.players.every(p => Number.isFinite(p.hp) && Number.isFinite(p.maxHp) && p.hp <= p.maxHp && p.hp >= 0), 'HP người chơi không hợp lệ.');
      assert(game.players.every(p => Number.isFinite(p.armor) && p.armor >= 0), 'Giáp người chơi không hợp lệ.');
      assert(game.players.every(p => Array.isArray(p.hand)), 'Tay bài không hợp lệ.');
      assert(game.decks && game.decks.attackDeck && game.decks.defenseDeck && game.decks.supportDeck && game.decks.eventDeck, 'Thiếu chồng bài.');
      assert(game.currentEvent && game.currentEvent.id && game.currentEvent.name, 'Thiếu sự kiện hiện tại.');
      return true;
    }

    function startNewDay(game, firstDay = false) {
      if (!firstDay) game.day += 1;

      const event = drawEvent(game);
      game.currentEvent = event;

      for (const p of game.players) {
        ensureEffects(p);
        p.playedToday = 0;
        p.skipEventToday = false;
        p.effects.firstAttackDone = false;
        p.effects.firstDefenseDone = false;
        p.effects.firstFogAttackDone = false;

        if (!firstDay) {
          const attackDrawCount = clamp(2 + p.effects.bonusAttackNextDay - p.effects.loseAttackDraws, 0, 5);
          p.hand.push(...draw(game.decks.attackDeck, attackDrawCount));
          p.hand.push(...draw(game.decks.defenseDeck, 1));
          p.effects.bonusAttackNextDay = 0;
          p.effects.loseAttackDraws = 0;
        }

        if (!firstDay && p.effects.poison) {
          applyDirectDamage(game, p, p.effects.poison, 'độc');
          p.effects.poison = 0;
        }
        if (!firstDay && p.effects.burn) {
          applyDirectDamage(game, p, p.effects.burn, 'cháy');
        }

        if (p.skill.id === 'S02') p.armor += 3;
      }

      // Fix lỗi self-test: sự kiện ngày 1 vẫn được lật, nhưng các hiệu ứng tự động
      // như rút/bỏ bài/mất HP không chạy trước khi người chơi có đúng tay bài khởi đầu.
      if (!firstDay) applyStartEvent(game, game.currentEvent);

      log(game, `Ngày ${game.day}: Sự kiện "${game.currentEvent.name}" - ${game.currentEvent.desc}`);
      checkWinner(game);
    }

    function drawEvent(game) {
      let event = draw(game.decks.eventDeck, 1)[0];
      if (!event) {
        game.decks.eventDeck = shuffle(CARD_LIBRARY.event.map(c => ({ id: c[0], name: c[1], desc: c[2] })));
        event = draw(game.decks.eventDeck, 1)[0];
      }
      return event;
    }

    function applyStartEvent(game, event) {
      if (!event) return;
      const activePlayers = game.players.filter(p => !p.skipEventToday);
      switch (event.id) {
        case 'E03': activePlayers.forEach(p => p.hand.push(...draw(game.decks.supportDeck, 1))); break;
        case 'E04': activePlayers.forEach(p => applyDirectDamage(game, p, 5, 'động đất')); break;
        case 'E08': activePlayers.forEach(p => p.armor = Math.max(0, p.armor - 4)); break;
        case 'E12': activePlayers.forEach(p => { if (p.hand.length > 6) discardRandom(p); }); break;
        case 'E13': activePlayers.forEach(p => p.hand.push(...draw(game.decks.defenseDeck, 1))); break;
        case 'E20': activePlayers.forEach(p => p.effects.bonusAttackNextDay += 1); break;
        case 'E21': activePlayers.forEach(p => discardRandomType(p, 'support')); break;
        case 'E25': activePlayers.forEach(p => heal(game, p, 3)); break;
        case 'E28': {
          const [a, b] = game.players;
          if (a.hp > b.hp) applyDirectDamage(game, a, 6, 'quân nổi loạn');
          else if (b.hp > a.hp) applyDirectDamage(game, b, 6, 'quân nổi loạn');
          break;
        }
        case 'E29': activePlayers.forEach(p => p.hand.push(...draw(game.decks.attackDeck, 1))); break;
      }
    }

    function applyEndDayEvent(game) {
      const event = game.currentEvent;
      if (!event) return;
      const active = game.players.filter(p => !p.skipEventToday);
      if (event.id === 'E15') {
        const [a, b] = game.players;
        if (a.armor < b.armor) applyDirectDamage(game, a, 6, 'quái vật');
        else if (b.armor < a.armor) applyDirectDamage(game, b, 6, 'quái vật');
      }
      if (event.id === 'E17') {
        active.forEach(p => { while (p.hand.length > 8) discardRandom(p); });
      }
    }

    function discardRandom(player) {
      if (!player.hand.length) return null;
      const index = Math.floor(Math.random() * player.hand.length);
      return player.hand.splice(index, 1)[0];
    }

    function discardRandomType(player, type) {
      const cards = player.hand.map((c, i) => ({ c, i })).filter(x => x.c.type === type);
      if (!cards.length) return null;
      const pick = cards[Math.floor(Math.random() * cards.length)];
      return player.hand.splice(pick.i, 1)[0];
    }

    function heal(game, player, amount) {
      const event = game?.currentEvent;
      let value = amount;
      if (event && !player.skipEventToday) {
        if (event.id === 'E10') value = Math.min(value, 8);
        if (event.id === 'E22') value += 4;
        if (event.id === 'E30') value = Math.max(0, value - 4);
      }
      player.hp = clamp(player.hp + value, 0, player.maxHp);
      return value;
    }

    function applyDirectDamage(game, target, amount, source = 'sát thương trực tiếp') {
      let dmg = amount;
      if (game.currentEvent && !target.skipEventToday && game.currentEvent.id === 'E23') dmg += 3;
      target.hp = clamp(target.hp - dmg, 0, target.maxHp);
      log(game, `${target.name} mất ${dmg} HP do ${source}.`);
      checkWinner(game);
      return dmg;
    }

    function applyAttackDamage(game, attacker, target, baseDamage, card) {
      let damage = baseDamage;
      const event = game.currentEvent;
      const eventActive = event && !attacker.skipEventToday;

      if (attacker.effects.skipActions > 0) {
        attacker.effects.skipActions -= 1;
        log(game, `${attacker.name} bị mất lượt nên thẻ ${card.name} không kích hoạt.`);
        return 0;
      }

      if (eventActive) {
        if (event.id === 'E01') damage -= 3;
        if (event.id === 'E06' && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)) damage += 5;
        if (event.id === 'E09' && !attacker.effects.firstAttackDone) damage += 4;
        if (event.id === 'E11' && /Đột Kích|Dao Găm/i.test(card.name)) damage += 5;
        if (event.id === 'E16' && !attacker.effects.firstAttackDone) card.tempPierce = (card.tempPierce || 0) + 5;
        if (event.id === 'E23' && /trực tiếp|xuyên/i.test(card.desc)) damage += 3;
        if (event.id === 'E24' && baseDamage < 10) damage += 5;
        if (event.id === 'E27' && /Sét/i.test(card.name)) {
          damage += 7;
          if (Math.random() < .3) applyDirectDamage(game, attacker, 3, 'sét phản');
        }
        if (event.id === 'E30') damage += 4;
        if (event.id === 'E05' && !attacker.effects.firstFogAttackDone) {
          attacker.effects.firstFogAttackDone = true;
          if (Math.random() < .4) {
            log(game, `${card.name} của ${attacker.name} trượt vì sương mù.`);
            return 0;
          }
        }
      }

      if (attacker.skill.id === 'S01' && !attacker.effects.firstAttackDone) damage += event?.id === 'E18' ? 6 : 3;
      if (attacker.skill.id === 'S05' && !attacker.effects.firstAttackDone) card.tempPierce = (card.tempPierce || 0) + (event?.id === 'E18' ? 8 : 4);
      if (attacker.effects.nextAttackBonus) {
        damage += attacker.effects.nextAttackBonus;
        attacker.effects.nextAttackBonus = 0;
      }

      attacker.effects.firstAttackDone = true;

      if (target.effects.nightGuard && /Đột Kích|Dao Găm/i.test(card.name)) damage -= 8;
      if (target.effects.shelter) damage -= 4;
      if (target.effects.nextIncomingHalf) {
        damage = Math.ceil(damage / 2);
        target.effects.nextIncomingHalf = false;
      }
      if (target.effects.nextIncomingSmallBlock && damage < 12) {
        log(game, `${target.name} chặn hoàn toàn ${card.name}.`);
        target.effects.nextIncomingSmallBlock = false;
        return 0;
      }
      if (target.effects.moat) {
        damage -= 5;
        target.effects.moat = false;
      }
      if (target.effects.magicShield && /Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)) {
        log(game, `Khiên phép của ${target.name} chặn ${card.name}.`);
        target.effects.magicShield = false;
        return 0;
      }
      if (target.effects.thorns) {
        applyDirectDamage(game, attacker, 6, 'bẫy chông');
        target.effects.thorns = false;
      }

      damage = Math.max(0, damage);
      let pierce = card.tempPierce || 0;
      if (/bỏ qua 3 giáp/i.test(card.desc)) pierce += 3;
      if (/bỏ qua khiên/i.test(card.desc)) pierce += 999;
      if (/trực tiếp vào HP/i.test(card.desc) || card.id === 'A16') pierce += 999;
      if (target.effects.ironGate) {
        pierce = 0;
        target.effects.ironGate = false;
      }

      const usableArmor = Math.max(0, target.armor - pierce);
      const blocked = Math.min(usableArmor, damage);
      target.armor = Math.max(0, target.armor - blocked);
      const hpDamage = damage - blocked;

      target.hp = clamp(target.hp - hpDamage, 0, target.maxHp);
      log(game, `${attacker.name} dùng ${card.name} gây ${hpDamage} HP, bị chặn ${blocked} bởi giáp.`);
      checkWinner(game);
      return hpDamage;
    }

    function applyCard(game, playerId, cardUid, targetId) {
      validateGame(game);
      if (game.winner) throw new Error('Trận đấu đã kết thúc.');
      const player = game.players.find(p => p.id === playerId);
      assert(player, 'Không tìm thấy người chơi.');
      ensureEffects(player);
      const limit = game.perDayLimit + player.effects.extraLimit;
      assert(player.playedToday < limit, 'Bạn đã hết lượt đánh trong ngày.');
      const index = player.hand.findIndex(c => c.uid === cardUid);
      assert(index >= 0, 'Không tìm thấy thẻ trên tay.');
      const card = player.hand[index];

      if (game.currentEvent && !player.skipEventToday && game.currentEvent.id === 'E07') {
        if (/Lửa|Phù Thủy|Sét|Pháp|Hỏa/i.test(card.name)) throw new Error('Sự kiện Cấm Phép đang chặn thẻ này.');
      }
      if (card.id === 'A18' && game.day <= 2) throw new Error('Búa Phá Thành chỉ dùng sau ngày 2.');
      if (card.id === 'A30' && player.hp >= 40) throw new Error('Tổng Công Kích chỉ dùng khi bạn dưới 40 HP.');
      if (card.id === 'D18' && game.day <= 3) throw new Error('Đại Tu Thành chỉ dùng sau ngày 3.');

      player.hand.splice(index, 1);
      player.playedToday += 1;

      const target = targetId ? game.players.find(p => p.id === targetId) : opponentOf(game, player);
      assert(target, 'Mục tiêu không hợp lệ.');
      ensureEffects(target);

      if (card.type === 'attack') handleAttackCard(game, player, target, card);
      if (card.type === 'defense') handleDefenseCard(game, player, card);
      if (card.type === 'support') handleSupportCard(game, player, card);

      if (game.currentEvent && !player.skipEventToday && game.currentEvent.id === 'E26' && player.playedToday > 2) {
        applyDirectDamage(game, player, 4, 'lời nguyền đánh quá 2 thẻ');
      }

      checkDayEnd(game);
      validateGame(game);
    }

    function handleAttackCard(game, player, target, card) {
      let damage = card.power;
      switch (card.id) {
        case 'A05': player.armor = Math.max(0, player.armor - 2); break;
        case 'A06': damage = target.armor < 10 ? 18 : 10; break;
        case 'A07': target.effects.poison += 3; break;
        case 'A11': discardRandom(player); break;
        case 'A12': player.armor += 2; break;
        case 'A13': damage = Math.random() < .5 ? 17 : 8; break;
        case 'A14': applyDirectDamage(game, player, 5, 'đội cảm tử'); break;
        case 'A15': player.hand.push(...draw(game.decks.attackDeck, 1)); break;
        case 'A19': target.armor = Math.max(0, target.armor - 4); break;
        case 'A20': if (game.currentEvent && /Mưa|Lũ|Bão/i.test(game.currentEvent.name)) damage += 5; break;
        case 'A23': target.effects.burn += 2; break;
        case 'A24': if (player.playedToday === 1) damage = 15; break;
        case 'A25': target.armor = Math.max(0, target.armor - 6); break;
        case 'A26': target.effects.skipActions += 1; break;
        case 'A27': target.effects.loseAttackDraws += 1; break;
        case 'A28': player.effects.skipActions += 1; break;
      }
      applyAttackDamage(game, player, target, damage, card);
    }

    function handleDefenseCard(game, player, card) {
      let gain = 0;
      let healed = 0;
      const event = game.currentEvent;
      const eventActive = event && !player.skipEventToday;
      const defenseBonus = player.effects.nextDefenseBonus + (player.effects.forge ? 4 : 0);

      if (eventActive) {
        if (event.id === 'E02') gain -= 3;
        if (event.id === 'E14' && !player.effects.firstDefenseDone) gain += 6;
      }

      switch (card.id) {
        case 'D01': gain += 10; break;
        case 'D02': gain += 16; break;
        case 'D03': gain += 12; player.effects.moat = true; break;
        case 'D04': healed = heal(game, player, 12); break;
        case 'D05': gain += 20; break;
        case 'D06': player.effects.nextIncomingHalf = true; break;
        case 'D07': gain += 14; player.effects.ironGate = true; break;
        case 'D08': healed = heal(game, player, player.hp < 50 ? 18 : 8); break;
        case 'D09': player.effects.magicShield = true; break;
        case 'D10': player.effects.nightGuard = true; break;
        case 'D11': gain += 8; player.hand.push(...draw(game.decks.defenseDeck, 1)); break;
        case 'D12': player.maxHp += 8; healed = heal(game, player, 8); break;
        case 'D13': player.effects.thorns = true; break;
        case 'D14': player.effects.nextIncomingSmallBlock = true; break;
        case 'D15': player.effects.shelter = true; break;
        case 'D16': gain += 6; player.effects.forge = true; break;
        case 'D17': player.effects.endure = true; break;
        case 'D18': healed = heal(game, player, 25); break;
      }

      if (player.skill.id === 'S04') healed += heal(game, player, event?.id === 'E18' ? 6 : 3);
      gain = Math.max(0, gain + defenseBonus);
      player.armor += gain;
      player.effects.nextDefenseBonus = 0;
      player.effects.firstDefenseDone = true;
      log(game, `${player.name} dùng ${card.name}: +${gain} giáp, hồi ${healed} HP.`);
    }

    function handleSupportCard(game, player, card) {
      switch (card.id) {
        case 'U01': player.skipEventToday = true; log(game, `${player.name} dùng Khắc Sự Kiện và bỏ qua sự kiện hôm nay.`); break;
        case 'U02': player.hand.push(...draw(game.decks.attackDeck, 1)); log(game, `${player.name} trinh sát và lấy 1 thẻ công.`); break;
        case 'U03': player.hand.push(...draw(game.decks.attackDeck, 1), ...draw(game.decks.defenseDeck, 1)); log(game, `${player.name} nhận tiếp tế.`); break;
        case 'U04': discardRandom(player); player.hand.push(...drawRandomCards(game, 2)); log(game, `${player.name} đổi gió, bỏ 1 rút 2.`); break;
        case 'U05': player.effects.nextDefenseBonus += 8; log(game, `${player.name} thuê thợ: thẻ thủ kế tiếp +8.`); break;
        case 'U06': player.effects.extraLimit += 1; log(game, `${player.name} được đánh thêm 1 thẻ hôm nay.`); break;
        case 'U07': player.effects.nextAttackBonus += 6; log(game, `${player.name} mưu kế: thẻ công kế tiếp +6.`); break;
        case 'U08': player.armor += 5; player.hand.push(...draw(game.decks.supportDeck, 1)); log(game, `${player.name} mở kho bí mật.`); break;
        case 'U09': player.effects.poison = 0; player.effects.burn = 0; player.effects.skipActions = 0; log(game, `${player.name} phản gián, xóa độc/cháy/mất lượt.`); break;
        case 'U10': game.players.forEach(p => p.armor += 6); player.hand.push(...drawRandomCards(game, 1)); log(game, `${player.name} ký hiệp ước tạm thời.`); break;
      }
    }

    function drawRandomCards(game, count) {
      const out = [];
      for (let i = 0; i < count; i++) {
        const available = ['attackDeck','defenseDeck','supportDeck'].filter(k => game.decks[k].length);
        if (!available.length) break;
        const key = available[Math.floor(Math.random() * available.length)];
        out.push(...draw(game.decks[key], 1));
      }
      return out;
    }

    function skipAction(game, playerId) {
      const p = game.players.find(x => x.id === playerId);
      if (!p || game.winner) return;
      ensureEffects(p);
      const limit = game.perDayLimit + p.effects.extraLimit;
      if (p.playedToday >= limit) return;
      p.playedToday += 1;
      log(game, `${p.name} bỏ 1 lượt đánh.`);
      checkDayEnd(game);
    }

    function checkDayEnd(game) {
      const allDone = game.players.every(p => p.playedToday >= game.perDayLimit + p.effects.extraLimit);
      if (allDone && !game.winner) {
        applyEndDayEvent(game);
        for (const p of game.players) {
          p.effects.extraLimit = 0;
          p.effects.shelter = false;
          p.effects.nightGuard = false;
          p.effects.forge = false;
          if (p.effects.burn) p.effects.burn = 0;
          if (p.armor > 0) p.armor = Math.floor(p.armor * 0.75);
        }
        if (!game.winner) startNewDay(game, false);
      }
    }

    function checkWinner(game) {
      for (const p of game.players) {
        if (p.hp <= 0 && p.effects.endure) {
          p.hp = 1;
          p.effects.endure = false;
          log(game, `${p.name} kích hoạt Kiên Cường và sống sót với 1 HP.`);
        }
      }
      const dead = game.players.find(p => p.hp <= 0);
      if (dead) {
        const win = game.players.find(p => p.id !== dead.id);
        game.winner = win.id;
        log(game, `${win.name} đã phá thành ${dead.name} và chiến thắng!`);
      }
    }

    function render() {
      if (!state.game) return;
      const game = state.game;
      const me = getMe();
      const opp = getOpponent();
      if (!me || !opp) return;

      $('dayNumber').textContent = game.day;
      $('turnInfo').textContent = `${me.playedToday}/${game.perDayLimit + me.effects.extraLimit}`;
      $('playerName').textContent = me.name;
      $('topRoomCode').textContent = state.roomCode || '(offline)';
      $('eventText').innerHTML = game.currentEvent ? `<b>${escapeHtml(game.currentEvent.name)}</b>: ${escapeHtml(game.currentEvent.desc)}` : 'Chưa có sự kiện.';
      $('syncStatus').textContent = state.connected ? 'Online' : 'Offline / đang chờ';

      renderCastles(me, opp);
      renderHand(me);
      renderTargets(opp);
      renderLog(game);

      const limit = game.perDayLimit + me.effects.extraLimit;
      const canPlay = !game.winner && me.playedToday < limit;
      $('playCardBtn').disabled = !canPlay || !state.selectedCardId;
      $('endActionBtn').disabled = !canPlay;
      $('actionHint').textContent = canPlay ? 'Bạn còn lượt đánh trong ngày.' : 'Bạn đã hết lượt hôm nay, chờ sang ngày mới.';

      if (game.winner) showWinner(game.winner === state.myId);
    }

    function renderCastles(me, opp) {
      $('castleGrid').innerHTML = [me, opp].map(p => {
        const percent = clamp((p.hp / p.maxHp) * 100, 0, 100);
        return `
          <div class="castle ${p.id === state.myId ? 'current' : 'enemy'}">
            <h2>${escapeHtml(p.name)}<span>${p.hp}/${p.maxHp} HP</span></h2>
            <div class="hpbar"><div class="hpfill" style="width:${percent}%"></div></div>
            <p class="tiny"><b>${escapeHtml(p.skill.name)}</b>: ${escapeHtml(p.skill.desc)}</p>
            <div class="stat-grid">
              <div class="stat">Giáp<b>${p.armor}</b></div>
              <div class="stat">Bài trên tay<b>${p.hand.length}</b></div>
              <div class="stat">Đã đánh hôm nay<b>${p.playedToday}</b></div>
              <div class="stat">Bỏ sự kiện<b>${p.skipEventToday ? 'Có' : 'Không'}</b></div>
            </div>
          </div>`;
      }).join('');
    }

    function renderHand(me) {
      $('hand').innerHTML = me.hand.map(card => `
        <div class="card ${card.type} ${state.selectedCardId === card.uid ? 'selected' : ''}" onclick="selectCard('${card.uid}')">
          <div class="type">${TYPE_LABEL[card.type]}</div>
          <div class="name">${escapeHtml(card.name)}</div>
          <div class="desc">${escapeHtml(card.desc)}</div>
          <div class="power"><span>${card.power ? 'Sức mạnh' : 'Hiệu ứng'}</span><b>${card.power || '—'}</b></div>
        </div>
      `).join('');
    }

    function renderTargets(opp) {
      const me = getMe();
      const targets = [opp, me];
      if (!state.selectedTarget || !targets.some(t => t.id === state.selectedTarget)) state.selectedTarget = opp.id;
      $('targetList').innerHTML = targets.map(t => `
        <button class="target-btn ${state.selectedTarget === t.id ? 'active' : ''}" onclick="selectTarget('${t.id}')">
          <span>${t.id === state.myId ? 'Bản thân' : 'Đối thủ'}</span><b>${t.hp} HP</b>
        </button>
      `).join('');
    }

    function renderLog(game) {
      $('log').innerHTML = game.log.map(x => `<div class="log-entry">${escapeHtml(x)}</div>`).join('');
    }

    function escapeHtml(str) {
      return String(str).replace(/[&<>'"]/g, s => ({'&':'&amp;','<':'&lt;','>':'&gt;',"'":'&#39;','"':'&quot;'}[s]));
    }

    window.selectCard = function(uid) {
      const me = getMe();
      if (!me) return;
      state.selectedCardId = state.selectedCardId === uid ? null : uid;
      const card = me.hand.find(c => c.uid === state.selectedCardId);
      if (card && card.type !== 'attack') state.selectedTarget = state.myId;
      if (card && card.type === 'attack') state.selectedTarget = getOpponent().id;
      render();
    };

    window.selectTarget = function(id) {
      state.selectedTarget = id;
      render();
    };

    function localAction(type, payload = {}) {
      if (!state.game || state.game.winner) return;
      try {
        if (type === 'play') applyCard(state.game, state.myId, payload.cardUid, payload.targetId);
        if (type === 'skip') skipAction(state.game, state.myId);
        state.selectedCardId = null;
        validateGame(state.game);
        send({ type: 'state', game: state.game });
        render();
      } catch (err) {
        alert(err.message || 'Có lỗi khi đánh thẻ.');
        render();
      }
    }

    function send(msg) {
      if (state.conn && state.conn.open) state.conn.send(JSON.parse(JSON.stringify(msg)));
    }

    function setupConnection(conn) {
      state.conn = conn;
      conn.on('open', () => {
        state.connected = true;
        $('netStatus').textContent = 'Đã kết nối với bạn bè.';
        if (state.isHost) {
          state.myId = 'host';
          state.game = createInitialGame();
          send({ type: 'state', game: state.game, roomCode: state.roomCode });
          showGame();
        }
        render();
      });

      conn.on('data', data => {
        try {
          if (data.type === 'state') {
            state.game = data.game;
            if (data.roomCode) state.roomCode = data.roomCode;
            if (!state.myId) state.myId = state.isHost ? 'host' : 'guest';
            validateGame(state.game);
            showGame();
            render();
          }
        } catch (err) {
          console.error(err);
          alert('Dữ liệu nhận bị lỗi: ' + err.message);
        }
      });

      conn.on('close', () => {
        state.connected = false;
        $('netStatus').textContent = 'Kết nối đã đóng.';
        render();
      });

      conn.on('error', err => {
        state.connected = false;
        $('netStatus').textContent = 'Lỗi kết nối: ' + err.message;
        render();
      });
    }

    function showGame() {
      $('lobby').classList.remove('active');
      $('game').classList.add('active');
    }

    function showWinner(isMe) {
      $('winner').classList.add('active');
      $('winnerText').textContent = isMe ? 'Bạn đã thắng!' : 'Bạn đã thua!';
      $('winnerSub').textContent = isMe ? 'Bạn đã phá xong thành đối thủ.' : 'Thành của bạn đã bị phá.';
    }

    $('createRoomBtn').addEventListener('click', () => {
      const code = 'thanh-' + Math.random().toString(36).slice(2, 7);
      state.roomCode = code;
      state.isHost = true;
      state.myId = 'host';
      $('roomCode').textContent = code;
      $('netStatus').textContent = 'Đang tạo phòng...';
      state.peer = new Peer(code, { debug: 1 });
      state.peer.on('open', () => $('netStatus').textContent = 'Phòng đã tạo. Gửi mã cho bạn bè.');
      state.peer.on('connection', conn => setupConnection(conn));
      state.peer.on('error', err => $('netStatus').textContent = 'Lỗi PeerJS: ' + err.message);
    });

    $('joinRoomBtn').addEventListener('click', () => {
      const code = $('joinCodeInput').value.trim();
      if (!code) return alert('Nhập mã phòng trước.');
      state.roomCode = code;
      state.isHost = false;
      state.myId = 'guest';
      $('netStatus').textContent = 'Đang vào phòng...';
      state.peer = new Peer(undefined, { debug: 1 });
      state.peer.on('open', () => {
        const conn = state.peer.connect(code, { reliable: true });
        setupConnection(conn);
      });
      state.peer.on('error', err => $('netStatus').textContent = 'Lỗi PeerJS: ' + err.message);
    });

    $('playCardBtn').addEventListener('click', () => {
      if (!state.selectedCardId) return;
      localAction('play', { cardUid: state.selectedCardId, targetId: state.selectedTarget });
    });

    $('endActionBtn').addEventListener('click', () => localAction('skip'));

    $('copyStateBtn').addEventListener('click', async () => {
      const text = JSON.stringify(state.game, null, 2);
      try {
        await navigator.clipboard.writeText(text);
        alert('Đã chép trạng thái debug.');
      } catch {
        prompt('Copy trạng thái debug:', text);
      }
    });

    function getCardCounts(player) {
      return player.hand.reduce((acc, card) => {
        acc[card.type] = (acc[card.type] || 0) + 1;
        return acc;
      }, {});
    }

    function runSelfTests() {
      try {
        validateLibrary();

        for (let i = 0; i < 60; i++) {
          const g = createInitialGame();
          validateGame(g);

          assert(g.players.every(p => p.hand.length === 8), 'Đầu game mỗi người phải có đúng 8 thẻ thường, không bị sự kiện ngày 1 làm rút/bỏ thêm.');
          assert(g.players.every(p => p.skill && p.skill.type === 'skill'), 'Mỗi người phải có 1 kỹ năng thành.');
          assert(g.currentEvent, 'Ngày 1 phải có sự kiện.');
          assert(g.decks.eventDeck.length === 29, 'Sau khi lật sự kiện ngày 1, chồng sự kiện phải còn 29 thẻ.');

          for (const p of g.players) {
            const counts = getCardCounts(p);
            assert(counts.attack === 4, 'Đầu game mỗi người phải có 4 thẻ công.');
            assert(counts.defense === 2, 'Đầu game mỗi người phải có 2 thẻ thủ.');
            assert(counts.support === 2, 'Đầu game mỗi người phải có 2 thẻ trợ.');
            assert(p.playedToday === 0, 'Đầu game chưa ai được tính là đã đánh thẻ.');
          }

          const p = g.players[0];
          const enemy = g.players[1];
          const attack = p.hand.find(c => c.type === 'attack' && !['A18', 'A30'].includes(c.id));
          if (attack) applyCard(g, p.id, attack.uid, enemy.id);
          validateGame(g);
        }

        const g2 = createInitialGame();
        const beforeDay = g2.day;
        g2.players.forEach(p => { p.playedToday = g2.perDayLimit; });
        checkDayEnd(g2);
        validateGame(g2);
        assert(g2.day === beforeDay + 1, 'Khi cả hai người hết 3 lượt, phải sang ngày mới.');
        assert(g2.players.every(p => p.playedToday === 0), 'Sang ngày mới phải reset lượt đã đánh.');
        assert(g2.players.every(p => p.hand.length >= 11 || g2.winner), 'Sang ngày mới mỗi người thường nhận thêm 2 công + 1 thủ, trừ khi sự kiện ngày mới tác động.');

        console.log('[SELF TEST PASS] Bộ bài, khởi tạo, sự kiện ngày 1, đánh thẻ, sang ngày mới đều ổn.');
      } catch (err) {
        console.error('[SELF TEST FAIL]', err);
        alert('Self-test phát hiện lỗi: ' + (err && err.message ? err.message : String(err)));
      }
    }

    runSelfTests();
  </script>
</body>
</html>
