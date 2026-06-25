<!DOCTYPE html>
<html lang="zh-Hant" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>6A 班級聯絡與 Open Mind Pro 平台</title>
  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            slate: {
              850: '#1e293b',
              950: '#020617'
            }
          }
        }
      }
    }
  </script>
  <style>
    /* 隱藏滾動條但保留滾動功能 */
    .no-scrollbar::-webkit-scrollbar {
      display: none;
    }
    .no-scrollbar {
      -ms-overflow-style: none;
      scrollbar-width: none;
    }
    /* 自訂精美滾動條 */
    .custom-scrollbar::-webkit-scrollbar {
      width: 6px;
      height: 6px;
    }
    .custom-scrollbar::-webkit-scrollbar-track {
      background: rgba(15, 23, 42, 0.6);
    }
    .custom-scrollbar::-webkit-scrollbar-thumb {
      background: rgba(148, 163, 184, 0.3);
      border-radius: 9999px;
    }
    .custom-scrollbar::-webkit-scrollbar-thumb:hover {
      background: rgba(148, 163, 184, 0.5);
    }
  </style>
</head>
<body class="bg-slate-950 text-slate-100 flex flex-col min-h-screen font-sans antialiased overflow-hidden select-none">

  <!-- 1. 登入畫面 (若 localStorage 無 active_user 則顯示) -->
  <div id="login-screen" class="fixed inset-0 bg-slate-950 flex flex-col justify-center items-center px-4 z-50 transition-all duration-300">
    <div class="absolute top-1/4 left-1/4 w-72 h-72 bg-emerald-500/10 rounded-full blur-3xl pointer-events-none"></div>
    <div class="absolute bottom-1/4 right-1/4 w-72 h-72 bg-pink-500/10 rounded-full blur-3xl pointer-events-none"></div>

    <div class="bg-slate-900 border border-slate-800 text-white rounded-3xl p-8 shadow-2xl max-w-md w-full relative z-10">
      <div class="flex flex-col items-center mb-8">
        <div class="w-20 h-20 bg-gradient-to-tr from-emerald-500 to-teal-400 rounded-2xl flex items-center justify-center text-4xl mb-4 shadow-lg shadow-emerald-500/20">
          🎓
        </div>
        <h1 class="text-2xl font-black tracking-wider text-emerald-400 bg-gradient-to-r from-emerald-400 to-teal-300 bg-clip-text text-transparent">
          6A 班級全能互聯網
        </h1>
        <p class="text-slate-400 text-xs mt-2 font-medium tracking-wide">
          免密碼快速通報入口 • 本地模擬實時同步
        </p>
      </div>

      <form id="login-form" class="space-y-6">
        <div>
          <label class="block text-xs font-bold uppercase tracking-wider mb-2 text-slate-300">
            選擇您的學號
          </label>
          <select id="login-select" class="w-full bg-slate-800 border border-slate-700 rounded-xl py-3.5 px-4 text-white focus:outline-none focus:ring-2 focus:ring-emerald-500 transition-all text-sm font-semibold cursor-pointer" required>
            <option value="">-- 請點選成員列表 --</option>
            <!-- 由 JS 動態渲染 6A01 - 6A32 -->
          </select>
        </div>

        <button type="submit" class="w-full bg-gradient-to-r from-emerald-500 to-teal-500 hover:from-emerald-400 hover:to-teal-400 active:scale-95 text-white font-bold py-4 px-4 rounded-xl shadow-lg shadow-emerald-500/20 transition-all duration-150 text-sm">
          一鍵登入大廳
        </button>
      </form>

      <div class="mt-8 pt-4 border-t border-slate-800 text-center text-[10px] text-slate-500 font-bold leading-relaxed">
        自動過濾粗言穢語機制已啟用 • 維護所有學生隱私保護
      </div>
    </div>
  </div>

  <!-- 2. 主介面 (登入後解鎖) -->
  <div id="main-app" class="hidden flex-1 flex flex-col h-screen overflow-hidden">
    
    <!-- 頂部主導航欄 -->
    <header class="bg-slate-900 border-b border-slate-800/80 px-4 py-3 flex justify-between items-center shadow-xl shrink-0 z-40">
      <div class="flex items-center space-x-3">
        <div class="w-10 h-10 bg-gradient-to-tr from-emerald-500 to-indigo-500 rounded-xl flex items-center justify-center font-black text-lg shadow-inner text-white">
          6A
        </div>
        <div>
          <h1 class="font-extrabold text-sm tracking-tight bg-gradient-to-r from-emerald-400 to-teal-300 bg-clip-text text-transparent">
            6A 班級聯絡與分享平台
          </h1>
          <p class="text-xs text-slate-400">登入身份: <span id="header-user-display" class="text-emerald-400 font-bold">--</span></p>
        </div>
      </div>

      <!-- 桌面端分頁選單 -->
      <div class="hidden lg:flex bg-slate-950 rounded-2xl p-1 border border-slate-800 text-xs gap-1">
        <button onclick="switchTab('chat')" id="tab-btn-chat" class="tab-btn px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 bg-emerald-500 text-white">
          💬 密聊聊天室
        </button>
        <button onclick="switchTab('instagram')" id="tab-btn-instagram" class="tab-btn px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 text-slate-400 hover:text-white">
          📸 動態分享
        </button>
        <button onclick="switchTab('voting')" id="tab-btn-voting" class="tab-btn px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 text-slate-400 hover:text-white">
          🗳️ 功能投票
        </button>
        <button onclick="switchTab('kahoot')" id="tab-btn-kahoot" class="tab-btn px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 text-slate-400 hover:text-white">
          🎮 Kahoot! 問答
        </button>
        <!-- 僅 6A32 顯示 -->
        <button onclick="switchTab('admin')" id="tab-btn-admin" class="tab-btn hidden px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 text-slate-400 hover:text-white">
          🛠️ 班級維護
        </button>
      </div>

      <div class="flex items-center space-x-2">
        <button onclick="handleLogout()" class="hidden sm:block text-slate-400 hover:text-rose-400 hover:border-rose-500/40 text-xs bg-slate-850 px-4 py-2 rounded-xl border border-slate-800 transition font-bold">
          退出
        </button>
        <button onclick="toggleMobileMenu()" class="lg:hidden p-2 text-slate-400 hover:text-white hover:bg-slate-800 rounded-xl transition">
          <svg id="menu-icon-open" class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path></svg>
          <svg id="menu-icon-close" class="w-5 h-5 hidden" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path></svg>
        </button>
      </div>
    </header>

    <!-- 行動端下拉選單 -->
    <div id="mobile-menu" class="hidden lg:hidden bg-slate-900 border-b border-slate-800 p-4 space-y-2 text-sm z-30 shrink-0">
      <button onclick="switchTab('chat'); toggleMobileMenu();" class="mobile-tab-btn w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 bg-emerald-500/10 text-emerald-400">
        <span>💬</span> <span>密聊聊天室</span>
      </button>
      <button onclick="switchTab('instagram'); toggleMobileMenu();" class="mobile-tab-btn w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 text-slate-300">
        <span>📸</span> <span>動態分享</span>
      </button>
      <button onclick="switchTab('voting'); toggleMobileMenu();" class="mobile-tab-btn w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 text-slate-300">
        <span>🗳️</span> <span>功能投票</span>
      </button>
      <button onclick="switchTab('kahoot'); toggleMobileMenu();" class="mobile-tab-btn w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 text-slate-300">
        <span>🎮</span> <span>Kahoot! 問答</span>
      </button>
      <button onclick="switchTab('admin'); toggleMobileMenu();" id="mobile-tab-btn-admin" class="mobile-tab-btn hidden w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 text-slate-300">
        <span>🛠️</span> <span>班級維護</span>
      </button>
      <button onclick="handleLogout()" class="w-full text-left py-2.5 px-4 rounded-xl font-bold text-rose-400 border border-rose-500/20 bg-rose-500/5 flex items-center space-x-2">
        <span>🚪</span> <span>登出帳號</span>
      </button>
    </div>

    <!-- 主區塊內容 -->
    <div class="flex-1 flex overflow-hidden relative">

      <!-- TAB 1: WhatsApp 密聊聊天室 -->
      <div id="tab-content-chat" class="tab-pane flex-1 flex flex-col md:flex-row bg-slate-950 overflow-hidden h-full">
        
        <!-- 左側對話室與師長列表 -->
        <div class="w-full md:w-80 bg-slate-900 border-b md:border-b-0 md:border-r border-slate-800 flex flex-col h-1/3 md:h-full overflow-y-auto p-3 shrink-0 custom-scrollbar">
          <div class="flex justify-between items-center mb-4">
            <h3 class="font-extrabold text-sm text-slate-200">我的聊天頻道</h3>
            <button onclick="openCreateRoomModal()" class="bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-black text-xs py-1.5 px-3 rounded-lg shadow transition active:scale-95">
              ➕ 新對話
            </button>
          </div>

          <!-- 聊天室頻道動態載入區 -->
          <div id="room-list-container" class="space-y-1.5 flex-1">
            <!-- 預載入與後續自建 -->
          </div>

          <!-- 快速老師私人對話發起 -->
          <div class="pt-4 border-t border-slate-800/60 mt-3">
            <span class="text-[10px] font-bold text-slate-500 uppercase tracking-wider block mb-2 px-1">
              🏫 聯絡任教老師
            </span>
            <div id="teacher-contact-container" class="space-y-1.5">
              <!-- JS 動態寫入 -->
            </div>
          </div>
        </div>

        <!-- 右側密聊核心聊天流 -->
        <div class="flex-1 flex flex-col bg-slate-900/40 relative h-2/3 md:h-full overflow-hidden">
          <div class="absolute inset-0 opacity-5 pointer-events-none bg-[radial-gradient(#10b981_1px,transparent_1px)] [background-size:16px_16px]"></div>

          <!-- 聊天室標頭 -->
          <div class="bg-slate-900/90 backdrop-blur border-b border-slate-800 p-3.5 flex justify-between items-center relative z-20 shrink-0">
            <div class="truncate">
              <h4 id="active-room-title" class="font-extrabold text-sm text-slate-100 truncate">
                大廳公共廣播 📢
              </h4>
              <p id="active-room-desc" class="text-[10px] text-slate-400 truncate mt-0.5">
                全班廣播對話
              </p>
            </div>
          </div>

          <!-- 訊息展示區 -->
          <div id="chat-messages-scroll" class="flex-1 overflow-y-auto p-4 space-y-4 relative z-10 custom-scrollbar">
            <!-- 訊息串流 -->
          </div>

          <!-- 輸入欄與附加程式區 -->
          <div class="bg-slate-900 border-t border-slate-800 p-3 space-y-2 relative z-10 shrink-0">
            <form id="chat-input-form" onsubmit="handleSendMsg(event)" class="space-y-2">
              <div class="flex flex-wrap items-center gap-2">
                <!-- 相片上傳 -->
                <label class="flex items-center space-x-1 bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs font-bold py-2 px-3 rounded-xl border border-slate-700 cursor-pointer transition active:scale-95">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z"></path></svg>
                  <span id="chat-pic-status">加入相片</span>
                  <input type="file" id="chat-image-input" accept="image/*" class="hidden" onchange="uploadChatPic(event)">
                </label>

                <!-- 上載網頁連結 -->
                <button type="button" onclick="openLinkModal()" id="chat-link-btn" class="flex items-center space-x-1 text-xs font-bold py-2 px-3 rounded-xl border bg-slate-800 hover:bg-slate-700 text-slate-300 border-slate-700 transition active:scale-95">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1"></path></svg>
                  <span id="chat-link-status">上載連結</span>
                </button>

                <!-- 附加自製 HTML 程式 -->
                <button type="button" onclick="openHtmlModal()" id="chat-html-btn" class="flex items-center space-x-1 text-xs font-bold py-2 px-3 rounded-xl border bg-slate-800 hover:bg-slate-700 text-slate-300 border-slate-700 transition active:scale-95">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4"></path></svg>
                  <span id="chat-html-status">附加網頁程式</span>
                </button>

                <!-- 清空當前附屬快取按鈕 -->
                <button type="button" id="clear-attachments-btn" onclick="clearAttachments()" class="hidden text-[10px] text-rose-400 hover:underline font-bold">
                  清空附加內容
                </button>
              </div>

              <!-- 輸入框與發送 -->
              <div class="flex items-center space-x-2">
                <input type="text" id="chat-text-input" placeholder="說點甚麼嗎... (自動屏蔽粗口)" class="flex-1 bg-slate-800 border border-slate-700 rounded-full py-2.5 px-4 text-white text-sm placeholder-slate-500 focus:outline-none focus:ring-2 focus:ring-emerald-500">
                <button type="submit" class="bg-emerald-500 hover:bg-emerald-600 active:scale-95 text-white p-3 rounded-full shadow-lg transition duration-150 flex items-center justify-center shrink-0">
                  <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3"></path></svg>
                </button>
              </div>
            </form>
            <div class="text-[10px] text-slate-500 text-center mt-2 flex items-center justify-center space-x-1">
              <span>💡 指南：</span>
              <span class="text-purple-400">omp [關鍵字]</span><span>查維基 |</span>
              <span class="text-purple-400">更多</span><span>看長文 |</span>
              <span class="text-purple-400">omp learn [詞彙]: [解釋]</span><span>教導 AI</span>
            </div>
          </div>
        </div>
      </div>

      <!-- TAB 2: Instagram 動態分享 -->
      <div id="tab-content-instagram" class="tab-pane hidden flex-1 overflow-y-auto p-4 space-y-6 custom-scrollbar">
        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-4 flex items-center justify-between shadow-lg max-w-md mx-auto">
          <div class="flex items-center space-x-3">
            <div class="w-12 h-12 bg-gradient-to-tr from-yellow-400 via-pink-500 to-purple-600 rounded-full flex items-center justify-center text-xl font-black">
              📸
            </div>
            <div>
              <h4 class="font-extrabold text-sm text-slate-100">發布班級 Instagram Post！</h4>
              <p class="text-xs text-slate-400">分享相片、撰寫心得，分享日常生活。</p>
            </div>
          </div>
          <button onclick="openCreatePostModal()" class="bg-pink-600 hover:bg-pink-500 active:scale-95 text-white text-xs font-bold py-2.5 px-4 rounded-full shadow-md transition">
            建立新貼文
          </button>
        </div>

        <!-- 貼文動態流 -->
        <div id="instagram-feed-container" class="max-w-md mx-auto space-y-6">
          <!-- 貼文清單 -->
        </div>
      </div>

      <!-- TAB 3: 功能建議投票 -->
      <div id="tab-content-voting" class="tab-pane hidden flex-1 overflow-y-auto p-4 space-y-6 custom-scrollbar">
        <div class="bg-gradient-to-r from-amber-500/20 to-orange-500/10 border border-amber-500/30 rounded-2xl p-5 max-w-3xl mx-auto">
          <h2 class="text-lg font-black text-amber-400 flex items-center space-x-2">
            <span>🗳️</span>
            <span>6A 專屬班級民主功能提案區</span>
          </h2>
          <p class="text-xs text-slate-400 mt-1">
            有甚麼有趣的網頁功能或新構想？提出提案讓全班投票，高票者將優先加入開發！
          </p>
        </div>

        <div class="flex justify-between items-center max-w-3xl mx-auto">
          <h3 class="font-black text-sm text-slate-200">當前提案列表</h3>
          <button onclick="openCreatePollModal()" class="bg-amber-500 hover:bg-amber-400 text-slate-950 font-extrabold text-xs py-2 px-4 rounded-xl shadow-lg transition active:scale-95">
            💡 發起新功能提案
          </button>
        </div>

        <!-- 投票列表 -->
        <div id="polls-grid" class="grid grid-cols-1 md:grid-cols-2 gap-4 max-w-3xl mx-auto">
          <!-- 提案列表 -->
        </div>
      </div>

      <!-- TAB 4: Kahoot! 問答 -->
      <div id="tab-content-kahoot" class="tab-pane hidden flex-1 overflow-y-auto p-4 space-y-6 custom-scrollbar">
        <div class="bg-gradient-to-r from-indigo-500/20 to-purple-500/10 border border-indigo-500/30 rounded-2xl p-5 max-w-4xl mx-auto">
          <h2 class="text-lg font-black text-indigo-400 flex items-center space-x-2">
            <span>🎮</span>
            <span>6A 班級問答王 (Kahoot! 版)</span>
          </h2>
          <p class="text-xs text-slate-400 mt-1">
            出題考驗全班默契，實時計時答題，獲得班級學霸最高頭銜！
          </p>
        </div>

        <!-- 問答操作面板 (載入進行中，或是大廳) -->
        <div id="kahoot-board-wrapper" class="max-w-4xl mx-auto">
          <!-- 動態載入問答介面或大廳 -->
        </div>
      </div>

      <!-- TAB 5: 隱藏維護 (僅 6A32 登入時會解鎖) -->
      <div id="tab-content-admin" class="tab-pane hidden flex-1 overflow-y-auto p-4 space-y-6 custom-scrollbar">
        <div class="bg-red-500/10 border border-red-500/30 rounded-2xl p-5 shadow-lg max-w-3xl mx-auto">
          <h2 class="text-lg font-bold text-red-400 flex items-center space-x-2">
            <span>🛡️</span>
            <span>班級維護管理通道</span>
          </h2>
          <p class="text-xs text-slate-400 mt-1">
            此通道已對同學深度隱藏身分，嚴格防範違規言論。
          </p>
        </div>

        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-4 max-w-3xl mx-auto">
          <h3 class="font-bold text-sm mb-4 pb-2 border-b border-slate-800">班級成員禁言維護列表</h3>
          <div id="banned-members-list" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3">
            <!-- 禁言對應選單 -->
          </div>
        </div>
      </div>

    </div>
  </div>

  <!-- ==================== MODALS 彈出視窗 ==================== -->

  <!-- MODAL 1: 新增私人/群組對話 (含老師與同學名單) -->
  <div id="create-room-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm hidden items-center justify-center z-50 p-4">
    <div class="bg-slate-900 border border-slate-700 rounded-3xl w-full max-w-md p-5 shadow-2xl space-y-4">
      <div class="flex justify-between items-center pb-2 border-b border-slate-800">
        <h3 class="font-extrabold text-base text-emerald-400 flex items-center space-x-2">
          <span>💬</span>
          <span>開啟新私人對話 / 小組群組</span>
        </h3>
        <button onclick="closeCreateRoomModal()" class="text-slate-400 hover:text-white">✕</button>
      </div>

      <form onsubmit="handleCreateRoom(event)" class="space-y-4">
        <div>
          <label class="block text-xs font-bold mb-1 text-slate-300">小組群組名稱 (選填)</label>
          <input type="text" id="new-room-group-name" placeholder="例如：6A 專題報告小組" class="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs text-white focus:outline-none">
        </div>

        <div>
          <label class="block text-xs font-bold mb-2 text-slate-300">🏫 選擇任教老師與主任</label>
          <div id="teachers-select-grid" class="grid grid-cols-2 gap-2 p-2 bg-slate-950 rounded-xl border border-slate-800 mb-3">
            <!-- 動態寫入老師勾選 -->
          </div>
        </div>

        <div>
          <label class="block text-xs font-bold mb-2 text-slate-300">👤 選擇 6A 班級同學</label>
          <div id="students-select-grid" class="grid grid-cols-3 gap-2 max-h-36 overflow-y-auto p-2 bg-slate-950 rounded-xl border border-slate-800 custom-scrollbar">
            <!-- 動態寫入學生勾選 -->
          </div>
        </div>

        <div class="flex justify-end space-x-2 pt-2">
          <button type="button" onclick="closeCreateRoomModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs">取消</button>
          <button type="submit" class="px-5 py-2 bg-emerald-500 hover:bg-emerald-400 text-slate-950 rounded-xl text-xs font-black shadow">建立對話</button>
        </div>
      </form>
    </div>
  </div>

  <!-- MODAL 2: 附加網頁程式 (HTML) -->
  <div id="html-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm hidden items-center justify-center z-50 p-4">
    <div class="bg-slate-900 border border-slate-700 rounded-2xl w-full max-w-xl p-5 shadow-2xl space-y-4">
      <div class="flex justify-between items-center pb-2 border-b border-slate-800">
        <h3 class="font-extrabold text-base text-purple-400 flex items-center space-x-2">
          <span>💻</span>
          <span>附加/上傳 HTML 互動小工具</span>
        </h3>
        <button onclick="closeHtmlModal()" class="text-slate-400 hover:text-white">✕</button>
      </div>

      <div class="space-y-3">
        <textarea id="html-code-input" placeholder="<!DOCTYPE html><html><body><h3>應用程式範例</h3></body></html>" rows="9" class="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-xs font-mono text-emerald-400 focus:outline-none focus:border-purple-500 resize-none custom-scrollbar"></textarea>
      </div>

      <div class="flex justify-end space-x-2 pt-2">
        <button type="button" onclick="closeHtmlModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs">取消</button>
        <button type="button" onclick="confirmAttachHtml()" class="px-5 py-2 bg-purple-600 hover:bg-purple-500 rounded-xl text-xs font-bold text-white shadow">確認附加</button>
      </div>
    </div>
  </div>

  <!-- MODAL 3: 上載連結 -->
  <div id="link-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm hidden items-center justify-center z-50 p-4">
    <div class="bg-slate-900 border border-slate-700 rounded-2xl w-full max-w-md p-5 shadow-2xl space-y-4">
      <div class="flex justify-between items-center pb-2 border-b border-slate-800">
        <h3 class="font-extrabold text-base text-teal-400 flex items-center space-x-2">
          <span>🔗</span>
          <span>上載網頁分享連結</span>
        </h3>
        <button onclick="closeLinkModal()" class="text-slate-400 hover:text-white">✕</button>
      </div>

      <div class="space-y-3">
        <div>
          <label class="block text-xs font-bold mb-1 text-slate-300">連結標題</label>
          <input type="text" id="link-title-input" placeholder="例如：學校官方網頁" class="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs text-white outline-none">
        </div>
        <div>
          <label class="block text-xs font-bold mb-1 text-slate-300">網址 URL</label>
          <input type="text" id="link-url-input" placeholder="www.example.com" class="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs text-white outline-none">
        </div>
      </div>

      <div class="flex justify-end space-x-2 pt-2">
        <button type="button" onclick="closeLinkModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs">取消</button>
        <button type="button" onclick="confirmAttachLink()" class="px-5 py-2 bg-teal-600 hover:bg-teal-500 rounded-xl text-xs font-bold text-white">確認附加</button>
      </div>
    </div>
  </div>

  <!-- MODAL 4: 建立貼文 (Instagram) -->
  <div id="create-post-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm hidden items-center justify-center z-50 p-4">
    <div class="bg-slate-900 border border-slate-700 rounded-2xl w-full max-w-md p-5 shadow-2xl space-y-4">
      <div class="flex justify-between items-center pb-2 border-b border-slate-800">
        <h3 class="font-extrabold text-base text-pink-400">分享新動態貼文</h3>
        <button onclick="closeCreatePostModal()" class="text-slate-400 hover:text-white">✕</button>
      </div>
      <form onsubmit="handleCreatePost(event)" class="space-y-4">
        <div>
          <label class="block text-xs font-bold mb-2 text-slate-300">選擇本機相片</label>
          <div class="flex items-center space-x-3">
            <label class="bg-slate-800 hover:bg-slate-700 border border-slate-700 rounded-xl px-4 py-2.5 text-xs font-bold cursor-pointer text-slate-200 transition">
              選取圖片檔案
              <input type="file" id="post-image-file-input" accept="image/*" class="hidden" onchange="uploadPostPic(event)">
            </label>
            <span id="post-pic-status" class="text-[10px] text-slate-500 font-bold">尚未選擇圖片</span>
          </div>
          <div id="post-pic-preview-wrapper" class="mt-3 rounded-xl overflow-hidden max-w-xs border border-slate-800 hidden">
            <img id="post-pic-preview" src="" alt="預覽" class="max-h-40 object-cover" />
          </div>
        </div>

        <div>
          <label class="block text-xs font-bold mb-1 text-slate-300">今日想法 / 貼文內容</label>
          <textarea id="post-text-input" placeholder="今日心情如何？寫些想分享的事情吧..." rows="3" class="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-sm text-white focus:outline-none focus:border-pink-500 resize-none custom-scrollbar" required></textarea>
        </div>

        <div class="flex justify-end space-x-2 pt-2">
          <button type="button" onclick="closeCreatePostModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs">取消</button>
          <button type="submit" class="px-5 py-2 bg-pink-600 hover:bg-pink-500 rounded-xl text-xs font-bold text-white shadow">發布貼文</button>
        </div>
      </form>
    </div>
  </div>

  <!-- MODAL 5: 發起新提案 (Voting) -->
  <div id="create-poll-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm hidden items-center justify-center z-50 p-4">
    <div class="bg-slate-900 border border-slate-700 rounded-2xl w-full max-w-md p-5 shadow-2xl space-y-4">
      <div class="flex justify-between items-center pb-2 border-b border-slate-800">
        <h3 class="font-extrabold text-sm text-amber-400">發起功能公投提案</h3>
        <button onclick="closeCreatePollModal()" class="text-slate-400 hover:text-white">✕</button>
      </div>
      <form onsubmit="handleCreateProposal(event)" class="space-y-3">
        <div>
          <label class="block text-xs font-bold mb-1 text-slate-300">功能名稱 (例如：全班每日點名工具)</label>
          <input type="text" id="poll-title-input" class="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-sm text-white focus:outline-none focus:border-amber-500" required>
        </div>
        <div>
          <label class="block text-xs font-bold mb-1 text-slate-300">功能詳細介紹</label>
          <textarea id="poll-desc-input" placeholder="簡單描述此功能可提供怎樣的便利..." rows="3" class="w-full bg-slate-800 border border-slate-700 rounded-xl p-3 text-xs text-white focus:outline-none focus:border-amber-500 resize-none custom-scrollbar"></textarea>
        </div>
        <div class="flex justify-end space-x-2 pt-1">
          <button type="button" onclick="closeCreatePollModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs">取消</button>
          <button type="submit" class="px-5 py-2 bg-amber-500 hover:bg-amber-400 text-slate-950 rounded-xl text-xs font-extrabold">發起公投</button>
        </div>
      </form>
    </div>
  </div>

  <!-- MODAL 6: 建立 Kahoot 題目 -->
  <div id="create-quiz-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm hidden items-center justify-center z-50 p-4 overflow-y-auto custom-scrollbar">
    <div class="bg-slate-900 border border-indigo-500/30 rounded-2xl w-full max-w-xl p-5 shadow-2xl my-8 space-y-4">
      <div class="flex justify-between items-center pb-2 border-b border-slate-800">
        <h4 class="font-extrabold text-sm text-indigo-400">設計新 Kahoot! 班級問答</h4>
        <button onclick="closeCreateQuizModal()" class="text-slate-400 hover:text-white">✕</button>
      </div>

      <form onsubmit="handleCreateQuiz(event)" class="space-y-4">
        <div>
          <label class="block text-xs font-bold mb-1 text-slate-300">問答主題名稱</label>
          <input type="text" id="quiz-title-input" placeholder="例如：6A 秘密聚會大搜密" class="w-full bg-slate-800 border border-slate-700 rounded-xl py-2 px-3 text-xs text-white focus:outline-none focus:border-indigo-500" required>
        </div>

        <div id="quiz-questions-edit-wrapper" class="space-y-4">
          <!-- 載入動態問題範本 -->
        </div>

        <div class="flex justify-between items-center pt-2">
          <button type="button" onclick="addQuestionToQuizDraft()" class="text-xs text-indigo-400 hover:underline font-bold">
            ➕ 新增一題問題
          </button>
          
          <div class="flex space-x-2">
            <button type="button" onclick="closeCreateQuizModal()" class="px-4 py-2 bg-slate-800 hover:bg-slate-700 rounded-xl text-xs">取消</button>
            <button type="submit" class="px-5 py-2 bg-indigo-600 hover:bg-indigo-500 rounded-xl text-xs font-black text-white shadow">發布問答</button>
          </div>
        </div>
      </form>
    </div>
  </div>

  <!-- MODAL 7: HTML 測試沙盒與「Console 終端偵錯機」 -->
  <div id="html-sandbox-fullscreen" class="fixed inset-0 bg-black/95 backdrop-blur-md hidden flex-col z-50">
    <!-- 沙盒頂部控制欄 -->
    <div class="bg-slate-900 border-b border-slate-800 p-4 flex justify-between items-center shrink-0">
      <div class="flex items-center space-x-2">
        <span class="w-3.5 h-3.5 rounded-full bg-purple-500 animate-pulse"></span>
        <span id="sandbox-running-title" class="font-extrabold text-sm text-slate-200">
          正在執行程式
        </span>
      </div>
      <div class="flex items-center space-x-2">
        <button onclick="reloadSandbox()" class="bg-slate-800 hover:bg-slate-700 text-slate-200 px-3 py-1.5 rounded-lg text-xs font-bold transition">
          🔄 重整
        </button>
        <button onclick="toggleSandboxSourceView()" id="sandbox-view-source-btn" class="bg-slate-800 hover:bg-slate-700 text-slate-200 px-3 py-1.5 rounded-lg text-xs font-bold transition">
          💻 檢視源碼
        </button>
        <button onclick="closeSandbox()" class="bg-rose-600 hover:bg-rose-500 text-white px-4 py-1.5 rounded-lg text-xs font-bold transition">
          ✕ 關閉沙盒
        </button>
      </div>
    </div>

    <!-- 運行主區域 -->
    <div class="flex-1 flex flex-col lg:flex-row overflow-hidden">
      
      <!-- 程式原始碼檢視器 (隱藏/開啟) -->
      <div id="sandbox-source-viewer" class="hidden lg:w-1/3 bg-slate-950 border-r border-slate-800 overflow-auto p-4 shrink-0 font-mono text-xs text-emerald-400 custom-scrollbar select-text selection:bg-slate-800">
        <pre id="sandbox-source-code-block" class="whitespace-pre-wrap break-all"></pre>
      </div>

      <!-- Iframe 沙盒 & Debug Console 雙欄佈局 -->
      <div class="flex-1 flex flex-col h-full overflow-hidden">
        <!-- Iframe 運行 -->
        <div class="flex-1 bg-white relative">
          <iframe id="sandbox-iframe" sandbox="allow-scripts" class="w-full h-full border-none"></iframe>
        </div>

        <!-- 虛擬 Console 終端除錯控制台 (Debugger Terminal) -->
        <div class="h-40 bg-slate-950 border-t border-slate-800 flex flex-col overflow-hidden shrink-0 font-mono text-xs">
          <div class="bg-slate-900 px-4 py-1.5 flex justify-between items-center text-slate-400 border-b border-slate-850">
            <span class="font-bold flex items-center space-x-1.5 text-slate-300">
              <span class="w-2 h-2 rounded-full bg-indigo-500"></span>
              <span>CONSOLE 終端偵錯機 (Debugger Output)</span>
            </span>
            <button onclick="clearSandboxConsole()" class="text-[10px] hover:text-white underline">清空</button>
          </div>
          <!-- 日誌流 -->
          <div id="sandbox-console-log-flow" class="flex-1 overflow-y-auto p-3 space-y-1 text-[11px] leading-relaxed custom-scrollbar select-text selection:bg-slate-800">
            <!-- 日誌注入 -->
          </div>
        </div>
      </div>
    </div>
  </div>


  <!-- ==================== CORE JAVASCRIPT ==================== -->
  <script>
    // --- 0. 基礎資料與輔助函數 ---
    const CLASS_MEMBERS = Array.from({ length: 32 }, (_, i) => {
      const num = String(i + 1).padStart(2, '0');
      return `6A${num}`;
    });

    const TEACHERS = [
      { id: 'teacher_hui', name: '許老師', role: '老師' },
      { id: 'teacher_chu', name: '朱老師', role: '老師' },
      { id: 'teacher_liao', name: '廖老師', role: '老師' },
      { id: 'director_tang', name: '唐主任', role: '主任' }
    ];

    const SECRET_ADMIN = "6A32";

    const TOXIC_WORDS = [
      "屌", "閪", "鳩", "柒", "仆街", "咸濕", "鹹濕", "戇九", "戇𨶙", "戇x", "戇c", 
      "吊你", "操你", "他媽的", "幹你娘", "fuck", "bitch", "shit", "asshole"
    ];

    function filterBadWords(text) {
      if (!text) return { censoredText: "", isHighlyToxic: false };
      let tempText = text;
      let matches = 0;
      TOXIC_WORDS.forEach(word => {
        const regex = new RegExp(word, "gi");
        if (regex.test(tempText)) {
          tempText = tempText.replace(regex, "***");
          matches++;
        }
      });
      return { censoredText: tempText, isHighlyToxic: matches >= 4 };
    }

    function getLocalData(key, defaultValue) {
      const stored = localStorage.getItem(key);
      try {
        return stored ? JSON.parse(stored) : defaultValue;
      } catch (e) {
        return defaultValue;
      }
    }

    function saveData(key, val) {
      localStorage.setItem(key, JSON.stringify(val));
    }

    function isBanned() {
      if (!currentUser) return false;
      return memberStatus[currentUser.memberId]?.isBanned || false;
    }

    function toggleBanUser(memberId) {
      if (currentUser?.memberId !== SECRET_ADMIN) return;
      const currentBan = memberStatus[memberId]?.isBanned || false;
      memberStatus[memberId] = { ...memberStatus[memberId], isBanned: !currentBan };
      saveData("6a_member_status", memberStatus);
      renderBannedMembers();
    }

    function handleLocalImageUpload(e, callback) {
      const file = e.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = (event) => {
        const img = new Image();
        img.src = event.target.result;
        img.onload = () => {
          const canvas = document.createElement('canvas');
          const MAX_WIDTH = 400;
          const MAX_HEIGHT = 400;
          let width = img.width;
          let height = img.height;

          if (width > height) {
            if (width > MAX_WIDTH) {
              height *= MAX_WIDTH / width;
              width = MAX_WIDTH;
            }
          } else {
            if (height > MAX_HEIGHT) {
              width *= MAX_HEIGHT / height;
              height = MAX_HEIGHT;
            }
          }
          canvas.width = width;
          canvas.height = height;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0, width, height);
          const compressedBase64 = canvas.toDataURL('image/jpeg', 0.6);
          callback(compressedBase64);
        };
      };
    }

    // --- 1. 全域狀態初始化 & LocalStorage 持久化獲取 ---
    let currentUser = null; // 格式: { memberId: '6A01', isAdmin: false }
    let activeRoomId = "global";
    let currentTab = "chat";

    // 自製網頁程式測試中
    let activeHtmlPayload = null;
    let isViewingSource = false;

    // 暫存聊天室待發送附屬資源
    let tempChatImage = "";
    let tempChatLinkUrl = "";
    let tempChatLinkTitle = "";
    let tempChatHtml = "";

    // 新增房間參與者暫存
    let newRoomParticipants = [];
    let newPostImageBase64 = "";

    // OMP AI 狀態
    let lastWikiTopic = getLocalData("omp_last_topic", "");
    let lastWikiContent = getLocalData("omp_last_content", "");

    // 取得/初始化資料
    let rooms = getLocalData("6a_rooms", [
      { id: "global", name: "大廳公共廣播 📢", participants: CLASS_MEMBERS, isGroup: true, type: "public", description: "全班廣播對話" }
    ]);
    let messages = getLocalData("6a_messages", [
      { id: "msg-welcome", sender: "Open Mind Pro 🤖", roomId: "global", text: "歡迎來到 6A 班級空間！試吓同我單獨密聊，或者輸入「omp 香港」查資料啦！✨", createdAt: Date.now() - 60000 }
    ]);
    let posts = getLocalData("6a_posts", []);
    let polls = getLocalData("6a_polls", []);
    let quizzes = getLocalData("6a_quizzes", [
      {
        id: "quiz-sample",
        creator: "學校行政室",
        title: "6A 班級默契大考驗 🧠",
        questions: [
          { question: "6A 班的隱藏維護特權學號是？", options: ["6A01", "6A12", "6A32", "秘密身分"], correctIndex: 2, timeLimit: 15 },
          { question: "哪一位不是本班的任教老師？", options: ["許老師", "朱老師", "廖老師", "王老師"], correctIndex: 3, timeLimit: 15 }
        ]
      }
    ]);
    let quizScores = getLocalData("6a_quiz_scores", []);
    let infoCentre = getLocalData("6a_information_centre", {});
    let memberStatus = getLocalData("6a_member_status", {});

    // Kahoot 執行態
    let activeQuiz = null;
    let currentQuestionIndex = 0;
    let quizScore = 0;
    let quizTimer = 15;
    let quizTimerInterval = null;
    let quizFinished = false;
    let selectedAnswerIndex = null;
    let quizDraftQuestions = [{ question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 }];

    // --- 2. 登入/登出核心邏輯 ---
    window.addEventListener("DOMContentLoaded", () => {
      renderLoginSelect();
      checkAuthSession();
    });

    function renderLoginSelect() {
      const select = document.getElementById("login-select");
      select.innerHTML = '<option value="">-- 請點選成員列表 --</option>';
      CLASS_MEMBERS.forEach(m => {
        const opt = document.createElement("option");
        opt.value = m;
        opt.textContent = m === SECRET_ADMIN ? `${m} 同學` : `${m} 同學`;
        select.appendChild(opt);
      });
    }

    function checkAuthSession() {
      const savedUser = localStorage.getItem("6a_active_member");
      if (savedUser) {
        currentUser = {
          memberId: savedUser,
          isAdmin: savedUser === SECRET_ADMIN
        };
        document.getElementById("login-screen").classList.add("hidden");
        document.getElementById("main-app").classList.remove("hidden");
        document.getElementById("header-user-display").textContent = currentUser.memberId;
        
        // 如果是管理者，悄悄為其解鎖隱藏選單
        if (currentUser.isAdmin) {
          document.getElementById("tab-btn-admin").classList.remove("hidden");
          document.getElementById("mobile-tab-btn-admin").classList.remove("hidden");
        }

        // 初始化資料渲染
        renderRoomList();
        renderTeacherContacts();
        renderChatMessages();
        renderInstagramFeed();
        renderPolls();
        renderKahootLobby();
        renderBannedMembers();
      } else {
        document.getElementById("login-screen").classList.remove("hidden");
        document.getElementById("main-app").classList.add("hidden");
      }
    }

    document.getElementById("login-form").addEventListener("submit", (e) => {
      e.preventDefault();
      const selected = document.getElementById("login-select").value;
      if (!selected) return;

      localStorage.setItem("6a_active_member", selected);
      checkAuthSession();
    });

    function handleLogout() {
      localStorage.removeItem("6a_active_member");
      currentUser = null;
      checkAuthSession();
    }

    // --- 3. 跨視窗/多開實時資料同步 ---
    window.addEventListener("storage", (e) => {
      if (e.key === "6a_rooms") { rooms = JSON.parse(e.newValue || "[]"); renderRoomList(); }
      if (e.key === "6a_messages") { messages = JSON.parse(e.newValue || "[]"); renderChatMessages(); }
      if (e.key === "6a_posts") { posts = JSON.parse(e.newValue || "[]"); renderInstagramFeed(); }
      if (e.key === "6a_polls") { polls = JSON.parse(e.newValue || "[]"); renderPolls(); }
      if (e.key === "6a_quizzes") { quizzes = JSON.parse(e.newValue || "[]"); renderKahootLobby(); }
      if (e.key === "6a_quiz_scores") { quizScores = JSON.parse(e.newValue || "[]"); renderKahootLobby(); }
      if (e.key === "6a_information_centre") { infoCentre = JSON.parse(e.newValue || "{}"); }
      if (e.key === "6a_member_status") { 
        memberStatus = JSON.parse(e.newValue || "{}"); 
        if (isBanned()) {
          alert("通知：您的網路通訊受限。");
        }
        renderBannedMembers();
      }
    });

    // --- 4. 系統分頁切換 ---
    function switchTab(tabId) {
      document.querySelectorAll(".tab-pane").forEach(pane => pane.classList.add("hidden"));
      document.getElementById(`tab-content-${tabId}`).classList.remove("hidden");

      document.querySelectorAll(".tab-btn").forEach(btn => {
        btn.className = "tab-btn px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 text-slate-400 hover:text-white";
      });
      
      const themeColors = {
        chat: "bg-emerald-500 text-white",
        instagram: "bg-pink-500 text-white",
        voting: "bg-amber-500 text-slate-950",
        kahoot: "bg-indigo-500 text-white",
        admin: "bg-red-600 text-white animate-pulse"
      };

      const desktopBtn = document.getElementById(`tab-btn-${tabId}`);
      if (desktopBtn) {
        desktopBtn.className = `tab-btn px-4 py-2 rounded-xl font-bold transition duration-150 flex items-center space-x-1.5 ${themeColors[tabId]}`;
      }

      // 行動端按鈕更新
      document.querySelectorAll(".mobile-tab-btn").forEach(btn => {
        btn.className = "mobile-tab-btn w-full text-left py-2.5 px-4 rounded-xl font-bold flex items-center space-x-2 text-slate-300";
      });
      // 找到觸發對象更新樣式
      const activeMobileColors = {
        chat: "bg-emerald-500/10 text-emerald-400",
        instagram: "bg-pink-500/10 text-pink-400",
        voting: "bg-amber-500/10 text-amber-400",
        kahoot: "bg-indigo-500/10 text-indigo-400",
        admin: "bg-red-500/10 text-red-400"
      };
      
      currentTab = tabId;
    }

    function toggleMobileMenu() {
      const menu = document.getElementById("mobile-menu");
      const openIcon = document.getElementById("menu-icon-open");
      const closeIcon = document.getElementById("menu-icon-close");
      
      if (menu.classList.contains("hidden")) {
        menu.classList.remove("hidden");
        openIcon.classList.add("hidden");
        closeIcon.classList.remove("hidden");
      } else {
        menu.classList.add("hidden");
        openIcon.classList.remove("hidden");
        closeIcon.classList.add("hidden");
      }
    }

    // --- 5. 對話頻道管理 (WhatsApp 功能區) ---
    function renderRoomList() {
      const container = document.getElementById("room-list-container");
      container.innerHTML = "";

      // 1. 公共大堂
      const globalBtn = document.createElement("button");
      globalBtn.className = `w-full text-left p-3 rounded-xl transition flex items-center justify-between ${
        activeRoomId === "global" 
          ? "bg-emerald-500/20 border border-emerald-500/40 text-emerald-300" 
          : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"
      }`;
      globalBtn.onclick = () => selectRoom("global");
      globalBtn.innerHTML = `
        <div class="truncate">
          <span class="font-black text-xs block">📢 大廳公共廣播</span>
          <span class="text-[10px] text-slate-400">全班廣播對話</span>
        </div>
        <span class="text-xs shrink-0">🌍</span>
      `;
      container.appendChild(globalBtn);

      // 2. Open Mind Pro 私聊
      const aiBtn = document.createElement("button");
      aiBtn.className = `w-full text-left p-3 rounded-xl transition flex items-center justify-between ${
        activeRoomId === `private-omp-${currentUser.memberId}` 
          ? "bg-purple-500/20 border border-purple-500/40 text-purple-300" 
          : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"
      }`;
      aiBtn.onclick = () => selectRoom(`private-omp-${currentUser.memberId}`);
      aiBtn.innerHTML = `
        <div class="truncate">
          <span class="font-black text-xs block">🤖 Open Mind Pro</span>
          <span class="text-[10px] text-slate-400">與維基百科 AI 私聊</span>
        </div>
        <span class="text-xs shrink-0">✨</span>
      `;
      container.appendChild(aiBtn);

      // 3. 過濾自建且與當前使用者有關的房間
      const myRooms = rooms.filter(r => r.participants?.includes(currentUser.memberId));
      myRooms.forEach(room => {
        const roomBtn = document.createElement("button");
        roomBtn.className = `w-full text-left p-3 rounded-xl transition flex items-center justify-between ${
          activeRoomId === room.id 
            ? "bg-emerald-500/20 border border-emerald-500/40 text-emerald-300" 
            : "bg-slate-800/40 hover:bg-slate-800 border border-transparent text-slate-300"
        }`;
        roomBtn.onclick = () => selectRoom(room.id);
        roomBtn.innerHTML = `
          <div class="truncate pr-2">
            <span class="font-black text-xs block truncate">
              ${room.isGroup ? "👥" : "👤"} ${room.name}
            </span>
            <span class="text-[10px] text-slate-400 block truncate font-semibold">
              成員: ${room.participants?.join(", ")}
            </span>
          </div>
        `;
        container.appendChild(roomBtn);
      });
    }

    function renderTeacherContacts() {
      const container = document.getElementById("teacher-contact-container");
      container.innerHTML = "";
      TEACHERS.forEach(t => {
        const btn = document.createElement("button");
        btn.className = "w-full text-left p-2.5 bg-slate-950/60 hover:bg-slate-800/40 border border-slate-850 rounded-xl transition flex items-center justify-between text-xs";
        btn.onclick = () => handleStartTeacherChat(t);
        btn.innerHTML = `
          <span class="font-semibold text-slate-200">${t.name} (${t.role})</span>
          <span class="text-[10px] text-emerald-400">發起對話 💬</span>
        `;
        container.appendChild(btn);
      });
    }

    function selectRoom(roomId) {
      activeRoomId = roomId;
      renderRoomList();
      
      // 更新標題資訊
      const title = document.getElementById("active-room-title");
      const desc = document.getElementById("active-room-desc");

      if (roomId === "global") {
        title.innerHTML = "大廳公共廣播 📢";
        desc.innerHTML = "全班廣播對話";
      } else if (roomId.startsWith("private-omp-")) {
        title.innerHTML = "Open Mind Pro 🤖";
        desc.innerHTML = "由 Wikipedia 與本地資料庫整合的智能對話機器人";
      } else {
        const r = rooms.find(room => room.id === roomId);
        if (r) {
          title.innerHTML = `${r.isGroup ? "👥" : "👤"} ${r.name}`;
          desc.innerHTML = `成員：${r.participants.join(", ")}`;
        }
      }

      renderChatMessages();
    }

    // --- 6. 聊天室主邏輯 & Wikipedia AI 「Open Mind Pro」 ---
    function renderChatMessages() {
      const container = document.getElementById("chat-messages-scroll");
      container.innerHTML = "";

      const filtered = messages.filter(m => m.roomId === activeRoomId);

      if (filtered.length === 0) {
        container.innerHTML = `<div class="text-center py-20 text-slate-500 text-xs">🌌 此對話室目前尚無任何訊息，快發言打破沉默吧！</div>`;
        return;
      }

      filtered.forEach(msg => {
        const isMe = msg.sender === currentUser.memberId;
        const isOmp = msg.sender === "Open Mind Pro 🤖";

        const msgDiv = document.createElement("div");
        msgDiv.className = `flex flex-col ${isMe ? 'items-end' : 'items-start'}`;

        // 靜默刪除按鈕 (僅 6A32 登入時會悄悄顯示，外觀完全與普通流程一模一樣，完美隱藏管理者身分)
        const deleteBtnHtml = currentUser.memberId === SECRET_ADMIN 
          ? `<button onclick="handleDeleteMsg('${msg.id}')" class="text-red-400 hover:text-red-300 text-[10px] mt-1 underline block ml-auto mr-1 font-bold">剷除言論</button>`
          : "";

        // 附屬多媒體內容
        let mediaHtml = "";
        if (msg.image) {
          mediaHtml += `<div class="mt-2 rounded-xl overflow-hidden border border-black/30 bg-slate-950 max-w-xs"><img src="${msg.image}" alt="照片" class="max-h-60 w-full object-cover"></div>`;
        }
        if (msg.link) {
          mediaHtml += `
            <a href="${msg.link.url.startsWith("http") ? msg.link.url : "https://" + msg.link.url}" target="_blank" rel="noopener noreferrer" class="mt-2.5 block p-3 bg-slate-950/80 hover:bg-slate-950 border border-slate-700/60 rounded-xl transition duration-150">
              <div class="flex items-center justify-between text-xs text-emerald-400 font-bold mb-1">
                <span>🔗 ${msg.link.title}</span>
                <span class="text-[10px] text-slate-500">外部連結 ↗</span>
              </div>
              <span class="text-[11px] text-slate-400 block truncate">${msg.link.url}</span>
            </a>`;
        }
        if (msg.html) {
          mediaHtml += `
            <div class="mt-2.5 p-3 bg-slate-950 rounded-xl border border-slate-800 flex flex-col space-y-2">
              <div class="flex items-center justify-between text-xs text-purple-400 pb-1.5 border-b border-slate-800/80 font-bold">
                <span class="flex items-center space-x-1">🌐 <span>HTML 應用程式</span></span>
              </div>
              <button onclick="runHtmlSandbox('${msg.sender}', \`${encodeURIComponent(msg.html)}\`)" class="w-full py-2 bg-purple-600 hover:bg-purple-500 text-white text-xs font-bold rounded-lg shadow-md transition active:scale-95">
                ▶️ 執行此應用程式
              </button>
            </div>`;
        }

        msgDiv.innerHTML = `
          <div class="max-w-[85%] md:max-w-[75%]">
            <div class="flex items-center space-x-2 px-1 mb-1 text-[10px] text-slate-400">
              <span class="font-black ${isMe ? 'text-indigo-400' : isOmp ? 'text-purple-400' : 'text-slate-400'}">${msg.sender}</span>
              <span>•</span>
              <span>${new Date(msg.createdAt).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}</span>
            </div>
            <div class="p-3.5 rounded-2xl shadow-lg border text-sm ${
              isMe 
                ? 'bg-emerald-600/90 border-emerald-500/80 text-white rounded-tr-none' 
                : 'bg-slate-800 border-slate-700/80 text-slate-100 rounded-tl-none'
            }">
              ${msg.text ? `<p class="whitespace-pre-wrap break-all leading-relaxed font-medium">${msg.text}</p>` : ""}
              ${mediaHtml}
            </div>
            ${deleteBtnHtml}
          </div>
        `;
        container.appendChild(msgDiv);
      });

      // 自動置底
      container.scrollTop = container.scrollHeight;
    }

    async function handleSendMsg(e) {
      e.preventDefault();
      if (isBanned()) {
        alert("禁言警告：您目前無法在群組中發言。");
        return;
      }

      const input = document.getElementById("chat-text-input");
      const textVal = input.value;

      // 敏感詞自動過濾為星號
      const { censoredText, isHighlyToxic } = filterBadWords(textVal);
      if (isHighlyToxic) {
        alert("⚠️ 系統偵測到極端惡意言論，已被自動屏蔽，請注意班規及發言禮儀！");
        return;
      }

      if (!censoredText.trim() && !tempChatImage && !tempChatHtml && !tempChatLinkUrl) return;

      const newMsg = {
        id: `msg-${Date.now()}`,
        sender: currentUser.memberId,
        roomId: activeRoomId,
        text: censoredText,
        image: tempChatImage || null,
        html: tempChatHtml || null,
        link: tempChatLinkUrl ? { url: tempChatLinkUrl, title: tempChatLinkTitle || "分享連結" } : null,
        createdAt: Date.now()
      };

      messages.push(newMsg);
      saveData("6a_messages", messages);
      input.value = "";
      clearAttachments();
      renderChatMessages();

      // 觸發 OMP (Open Mind Pro) AI 智能助理
      const isAiRoom = activeRoomId.startsWith("private-omp-");
      const textLower = censoredText.trim().toLowerCase();
      const isOmpTrigger = /^(omp|openmind|open mind)\s+/i.test(textLower) || 
                           textLower === "更多" || 
                           textLower === "mor" || 
                           textLower === "more";

      if (isAiRoom || isOmpTrigger) {
        // 模擬延遲打字效果
        setTimeout(async () => {
          const aiResponseText = await getNaturalAIResponse(censoredText);
          const aiMsg = {
            id: `msg-ai-${Date.now()}`,
            sender: "Open Mind Pro 🤖",
            roomId: activeRoomId,
            text: aiResponseText,
            createdAt: Date.now()
          };
          messages.push(aiMsg);
          saveData("6a_messages", messages);
          renderChatMessages();
        }, 1000);
      }
    }

    // --- 7. OMP 粵語擬人 AI 機制與 Wikipedia 資料庫抓取 ---
    async function getNaturalAIResponse(inputStr) {
      const textLower = inputStr.trim().toLowerCase();
      
      // 1. 學習指令: omp learn [keyword]: [content]
      const learnRegex = /^(?:omp|openmind|open mind)\s+learn\s+([^:]+):(.*)/i;
      if (learnRegex.test(textLower)) {
        const match = inputStr.match(learnRegex);
        if (match) {
          const keyword = match[1].trim();
          const content = match[2].trim();
          if (keyword && content) {
            infoCentre[keyword.toLowerCase()] = {
              content: content,
              author: currentUser.memberId,
              timestamp: Date.now()
            };
            saveData("6a_information_centre", infoCentre);

            const replies = [
              `嘩！真係唔該晒你呀，等我快啲記底先。📝 我已經將「${keyword}」儲存到資訊中心啦！下次有人問我，我就會大大聲話俾佢地聽：『${content}』！`,
              `收到！「${keyword}」原來係咁解，我又學到新嘢啦！多謝你教我呀！🧠`,
              `好嘢！我又聰明左啦。關於「${keyword}」嘅解釋已經存入左資訊中心。多謝你呀，你真係好幫得手！✨`
            ];
            const chosen = replies[Math.floor(Math.random() * replies.length)];
            speakVoice(`多謝你教我呀！我已經記底咗關於${keyword}嘅知識啦。`);
            return chosen;
          }
        }
      }

      // 2. 請求更多/MOR
      if (textLower === "更多" || textLower === "mor" || textLower === "more") {
        if (lastWikiTopic && lastWikiContent) {
          speakVoice(`好啊，等我同你講多啲關於 ${lastWikiTopic} 嘅內容啦。`);
          return `好啊！等我將知道關於《${lastWikiTopic}》嘅全部細節都話大眾知，你想睇多啲對不對？👇\n\n${lastWikiContent}\n\n希望呢啲資料幫到你啦！🥰`;
        } else {
          return "咦？你係咪想聽多啲？不過我依家個腦袋空空如也，你仲未同我開始任何話題啵。試下同我講「omp [你想知嘅野]」先啦！🔍";
        }
      }

      // 3. 問候語
      const greetings = ["你好", "嗨", "hello", "hi", "喂", "在嗎", "早晨", "早安"];
      if (greetings.some(g => textLower.includes(g))) {
        const responses = [
          `嗨！你好呀！今日有咩好玩嘅事分享？定係你想查啲咩？話我知啦！😊`,
          `喂！今日過得點呀？我係 Open Mind Pro，今日想同我傾啲咩？`,
          `Hello！見到你真係高興！今日等我隨時為你提供維基百科同班級資訊！✨`
        ];
        return responses[Math.floor(Math.random() * responses.length)];
      }

      // 4. 抽取搜尋條目
      let queryTerm = inputStr.replace(/^(omp|openmind|open mind)\s+/i, "").trim();
      const isAiPrivateRoom = activeRoomId.startsWith("private-omp-");
      if (!queryTerm && isAiPrivateRoom) {
        queryTerm = inputStr.trim();
      }

      if (queryTerm) {
        // A. 先去維基百科查實時 API
        const wiki = await queryWikiData(queryTerm);
        if (wiki) {
          speakVoice(wiki.summary);
          return `等我幫你上維基百科查吓先…… 🔍 搵到啦！原來【${wiki.title}】係咁樣嘅：\n\n「${wiki.summary}」\n\n如果你想睇埋其餘部分，可以同我講「更多」或者「MOR」㗎！\n🔗 傳送門：${wiki.url}`;
        }

        // B. 維基沒有，查本地資訊中心
        const local = infoCentre[queryTerm.toLowerCase()];
        if (local) {
          speakVoice(local.content);
          return `咦！維基百科就查唔到，不過好在之前有同學喺「資訊中心」教過我！💡\n\n原來【${queryTerm}】係指：\n「${local.content}」\n\n呢個係由 ${local.author} 同學話我知嘅！`;
        }

        // C. 都找不到，撒嬌發起學術要求
        const queryAsk = [
          `哎呀，我查過維基百科同埋我個小腦袋，都唔知咩係「${queryTerm}」喎…… 😢\n\n你可唔可以做我老師教吓我呀？\n只要打：\n👉「omp learn ${queryTerm}: [你嘅解釋]」\n我就會一世記住你教我嘅嘢㗎啦！`,
          `唔好意思呀，我喺維基同埋資訊中心都搵唔到「${queryTerm}」嘅資料。🧐\n\n不如你教吓我啦？你可以用以下格式教我：\n👉「omp learn ${queryTerm}: [你嘅解釋]」\n拜託你啦！`,
          `咦？「${queryTerm}」係啲咩黎㗎？聽落好深奧，連維基都無寫！\n\n你可唔可以喺度解釋俾我聽？請打：\n👉「omp learn ${queryTerm}: [你嘅解釋]」\n我等緊你教我㗎！🎒`
        ];
        speakVoice(`我真係唔識得咩係${queryTerm}呀，你可以教我嗎？`);
        return queryAsk[Math.floor(Math.random() * queryAsk.length)];
      }

      return "嗯…… 我好似聽唔係好明，你可以試下輸入「omp [你想查嘅主題]」或者同我隨便傾兩句呀！";
    }

    async function queryWikiData(topic) {
      try {
        const searchUrl = `https://zh.wikipedia.org/w/api.php?action=query&list=search&srsearch=${encodeURIComponent(topic)}&format=json&origin=*`;
        const res = await fetch(searchUrl);
        const data = await res.json();
        
        if (data.query?.search?.length > 0) {
          const match = data.query.search[0];
          const pageTitle = match.title;

          const extractUrl = `https://zh.wikipedia.org/w/api.php?action=query&prop=extracts&exintro&explaintext&titles=${encodeURIComponent(pageTitle)}&format=json&origin=*`;
          const exRes = await fetch(extractUrl);
          const exData = await exRes.json();
          
          const pages = exData.query?.pages;
          const pageId = Object.keys(pages)[0];
          const text = pages[pageId].extract || "";

          if (text.trim()) {
            lastWikiTopic = pageTitle;
            lastWikiContent = text;
            saveData("omp_last_topic", pageTitle);
            saveData("omp_last_content", text);

            return {
              title: pageTitle,
              summary: text.substring(0, 150) + (text.length > 150 ? "..." : ""),
              full: text,
              url: `https://zh.wikipedia.org/wiki/${encodeURIComponent(pageTitle)}`
            };
          }
        }
        return null;
      } catch (e) {
        console.error(e);
        return null;
      }
    }

    // --- 8. 建立新對話 & 私人對話發起 (包含老師勾選) ---
    function openCreateRoomModal() {
      // 載入老師名單勾選
      const teachersContainer = document.getElementById("teachers-select-grid");
      teachersContainer.innerHTML = "";
      TEACHERS.forEach(t => {
        const btn = document.createElement("button");
        btn.type = "button";
        btn.className = "room-part-btn py-1.5 px-2 rounded-lg text-xs font-bold border border-slate-700/40 text-slate-400 bg-slate-800/40 hover:bg-slate-800 transition text-left flex justify-between items-center";
        btn.onclick = () => toggleRoomParticipant(t.name, btn);
        btn.innerHTML = `<span>${t.name}</span> <span class="text-[8px] bg-slate-900 text-slate-500 px-1 rounded">${t.role}</span>`;
        teachersContainer.appendChild(btn);
      });

      // 載入同學名單勾選
      const studentsContainer = document.getElementById("students-select-grid");
      studentsContainer.innerHTML = "";
      CLASS_MEMBERS.filter(m => m !== currentUser.memberId).forEach(m => {
        const btn = document.createElement("button");
        btn.type = "button";
        btn.className = "room-part-btn py-1.5 px-2 rounded-lg text-xs font-bold border border-slate-700/40 text-slate-400 bg-slate-800/40 hover:bg-slate-800 transition";
        btn.onclick = () => toggleRoomParticipant(m, btn);
        btn.textContent = m;
        studentsContainer.appendChild(btn);
      });

      document.getElementById("create-room-modal").style.display = "flex";
    }

    function closeCreateRoomModal() {
      document.getElementById("create-room-modal").style.display = "none";
      newRoomParticipants = [];
      document.getElementById("new-room-group-name").value = "";
    }

    function toggleRoomParticipant(id, element) {
      if (newRoomParticipants.includes(id)) {
        newRoomParticipants = newRoomParticipants.filter(p => p !== id);
        element.classList.remove("bg-emerald-500/20", "border-emerald-500", "text-emerald-400");
        element.classList.add("border-slate-700/40", "text-slate-400", "bg-slate-800/40");
      } else {
        newRoomParticipants.push(id);
        element.classList.add("bg-emerald-500/20", "border-emerald-500", "text-emerald-400");
        element.classList.remove("border-slate-700/40", "text-slate-400", "bg-slate-800/40");
      }
    }

    function handleCreateRoom(e) {
      e.preventDefault();
      if (newRoomParticipants.length === 0) return;

      const allParticipants = Array.from(new Set([currentUser.memberId, ...newRoomParticipants])).sort();
      const groupNameInput = document.getElementById("new-room-group-name").value;

      // 如果是 1 對 1 私聊，檢查先前是否有開啟過
      if (allParticipants.length === 2) {
        const existing = rooms.find(r => 
          r.participants?.length === 2 && 
          r.participants.includes(allParticipants[0]) && 
          r.participants.includes(allParticipants[1])
        );
        if (existing) {
          selectRoom(existing.id);
          closeCreateRoomModal();
          return;
        }
      }

      const newRoom = {
        id: `room-${Date.now()}`,
        name: groupNameInput.trim() || allParticipants.filter(p => p !== currentUser.memberId).join(", "),
        participants: allParticipants,
        isGroup: allParticipants.length > 2,
        createdBy: currentUser.memberId,
        createdAt: Date.now()
      };

      rooms.push(newRoom);
      saveData("6a_rooms", rooms);
      closeCreateRoomModal();
      selectRoom(newRoom.id);
    }

    function handleStartTeacherChat(teacher) {
      const allParticipants = [currentUser.memberId, teacher.name].sort();
      const existing = rooms.find(r => 
        r.participants?.length === 2 && 
        r.participants.includes(allParticipants[0]) && 
        r.participants.includes(allParticipants[1])
      );

      if (existing) {
        selectRoom(existing.id);
        switchTab("chat");
        return;
      }

      const newRoom = {
        id: `room-${Date.now()}`,
        name: `${teacher.name} (${teacher.role})`,
        participants: allParticipants,
        isGroup: false,
        createdBy: currentUser.memberId,
        createdAt: Date.now()
      };

      rooms.push(newRoom);
      saveData("6a_rooms", rooms);

      // 自動引導發送一條老師貼心問候
      const teacherIntro = {
        id: `msg-teacher-intro-${Date.now()}`,
        sender: teacher.name,
        roomId: newRoom.id,
        text: `同學你好，有關於功課或者學習需要幫手，都可以隨時留言俾我，我見到會儘快覆你。`,
        createdAt: Date.now()
      };
      messages.push(teacherIntro);
      saveData("6a_messages", messages);

      selectRoom(newRoom.id);
      switchTab("chat");
    }

    // --- 9. 多媒體上傳與緩存 ---
    function uploadChatPic(e) {
      handleLocalImageUpload(e, (base64) => {
        tempChatImage = base64;
        document.getElementById("chat-pic-status").textContent = "✓ 相片已載入";
        document.getElementById("clear-attachments-btn").classList.remove("hidden");
      });
    }

    function openHtmlModal() { document.getElementById("html-modal").style.display = "flex"; }
    function closeHtmlModal() { document.getElementById("html-modal").style.display = "none"; }
    function confirmAttachHtml() {
      const code = document.getElementById("html-code-input").value;
      if (code.trim()) {
        tempChatHtml = code;
        document.getElementById("chat-html-status").textContent = "✓ 已編譯程式";
        document.getElementById("clear-attachments-btn").classList.remove("hidden");
        closeHtmlModal();
      }
    }

    function openLinkModal() { document.getElementById("link-modal").style.display = "flex"; }
    function closeLinkModal() { document.getElementById("link-modal").style.display = "none"; }
    function confirmAttachLink() {
      const url = document.getElementById("link-url-input").value;
      const title = document.getElementById("link-title-input").value;
      if (url.trim()) {
        tempChatLinkUrl = url;
        tempChatLinkTitle = title || "分享連結";
        document.getElementById("chat-link-status").textContent = "✓ 已載入連結";
        document.getElementById("clear-attachments-btn").classList.remove("hidden");
        closeLinkModal();
      }
    }

    function clearAttachments() {
      tempChatImage = "";
      tempChatHtml = "";
      tempChatLinkUrl = "";
      tempChatLinkTitle = "";
      document.getElementById("chat-pic-status").textContent = "加入相片";
      document.getElementById("chat-html-status").textContent = "附加網頁程式";
      document.getElementById("chat-link-status").textContent = "上載連結";
      document.getElementById("clear-attachments-btn").classList.add("hidden");
      document.getElementById("chat-image-input").value = "";
      document.getElementById("html-code-input").value = "";
      document.getElementById("link-url-input").value = "";
      document.getElementById("link-title-input").value = "";
    }

    // --- 10. 程式測試沙盒 Fullscreen & Console 除錯引擎 ---
    function runHtmlSandbox(sender, encodedCode) {
      const code = decodeURIComponent(encodedCode);
      activeHtmlPayload = { sender, code };

      document.getElementById("sandbox-running-title").innerHTML = `正在執行來自 <span class="text-purple-400 font-black">${sender}</span> 的 HTML 應用程式`;
      document.getElementById("html-sandbox-fullscreen").style.display = "flex";
      
      isViewingSource = false;
      document.getElementById("sandbox-source-viewer").classList.add("hidden");
      document.getElementById("sandbox-view-source-btn").textContent = "💻 檢視源碼";

      clearSandboxConsole();
      reloadSandbox();
    }

    function closeSandbox() {
      document.getElementById("html-sandbox-fullscreen").style.display = "none";
      activeHtmlPayload = null;
    }

    function reloadSandbox() {
      if (!activeHtmlPayload) return;

      const iframe = document.getElementById("sandbox-iframe");
      
      // 核心除錯注入：重寫 console 函式與 JavaScript 語法 runtime 錯誤捕獲，實時將日誌寫入除錯終端機
      const consoleOverrideScript = `
        <script>
          (function() {
            const _log = console.log;
            const _error = console.error;
            const _warn = console.warn;

            function sendToParent(type, args) {
              const msg = Array.from(args).map(arg => {
                if (typeof arg === 'object') {
                  try { return JSON.stringify(arg); } catch(e) { return String(arg); }
                }
                return String(arg);
              }).join(' ');
              window.parent.postMessage({ type: 'sandbox-console', logType: type, content: msg }, '*');
            }

            console.log = function() {
              sendToParent('log', arguments);
              _log.apply(console, arguments);
            };
            console.error = function() {
              sendToParent('error', arguments);
              _error.apply(console, arguments);
            };
            console.warn = function() {
              sendToParent('warn', arguments);
              _warn.apply(console, arguments);
            };

            window.addEventListener('error', function(e) {
              window.parent.postMessage({ type: 'sandbox-console', logType: 'runtime-error', content: e.message + ' (line ' + e.lineno + ')' }, '*');
            });
          })();
        <\/script>
      `;

      // 組合注入
      const injectedCode = consoleOverrideScript + activeHtmlPayload.code;
      iframe.srcdoc = injectedCode;
    }

    // 監聽來自 Iframe 內 Console 模擬捕獲的事件
    window.addEventListener("message", (e) => {
      if (e.data && e.type === "message" && e.data.type === "sandbox-console") {
        appendSandboxConsoleLog(e.data.logType, e.data.content);
      }
      // 處理來自 iframe postMessage 的通用格式
      if (e.data && e.data.type === "sandbox-console") {
        appendSandboxConsoleLog(e.data.logType, e.data.content);
      }
    });

    function appendSandboxConsoleLog(type, content) {
      const container = document.getElementById("sandbox-console-log-flow");
      const div = document.createElement("div");

      const colors = {
        log: "text-slate-300",
        warn: "text-yellow-400 bg-yellow-500/10 px-1 rounded",
        error: "text-rose-400 bg-rose-500/10 px-1 rounded font-bold",
        "runtime-error": "text-rose-500 bg-rose-600/20 px-1 rounded font-black"
      };

      const prefix = {
        log: "💬 [Log]",
        warn: "⚠️ [Warn]",
        error: "❌ [Error]",
        "runtime-error": "🚨 [Runtime Error]"
      };

      div.className = `${colors[type] || "text-slate-300"} break-all`;
      div.textContent = `${prefix[type] || "💬"} ${content}`;
      container.appendChild(div);

      container.scrollTop = container.scrollHeight;
    }

    function clearSandboxConsole() {
      document.getElementById("sandbox-console-log-flow").innerHTML = "";
    }

    function toggleSandboxSourceView() {
      const viewer = document.getElementById("sandbox-source-viewer");
      const codeBlock = document.getElementById("sandbox-source-code-block");
      const btn = document.getElementById("sandbox-view-source-btn");

      if (isViewingSource) {
        viewer.classList.add("hidden");
        btn.textContent = "💻 檢視源碼";
        isViewingSource = false;
      } else {
        codeBlock.textContent = activeHtmlPayload ? activeHtmlPayload.code : "無程式碼。";
        viewer.classList.remove("hidden");
        btn.textContent = "👁️ 預覽運行";
        isViewingSource = true;
      }
    }

    // --- 11. Instagram 動態牆 ---
    function openCreatePostModal() {
      document.getElementById("create-post-modal").style.display = "flex";
    }

    function closeCreatePostModal() {
      document.getElementById("create-post-modal").style.display = "none";
      document.getElementById("post-image-file-input").value = "";
      document.getElementById("post-pic-status").textContent = "尚未選擇圖片";
      document.getElementById("post-pic-preview-wrapper").classList.add("hidden");
      document.getElementById("post-text-input").value = "";
      newPostImageBase64 = "";
    }

    function uploadPostPic(e) {
      handleLocalImageUpload(e, (base64) => {
        newPostImageBase64 = base64;
        document.getElementById("post-pic-status").textContent = "✓ 圖片已載入並自動壓縮";
        const preview = document.getElementById("post-pic-preview");
        preview.src = base64;
        document.getElementById("post-pic-preview-wrapper").classList.remove("hidden");
      });
    }

    function handleCreatePost(e) {
      e.preventDefault();
      if (isBanned()) {
        alert("禁言警告：您目前通訊受限。");
        return;
      }

      const textVal = document.getElementById("post-text-input").value;
      const { censoredText, isHighlyToxic } = filterBadWords(textVal);
      if (isHighlyToxic) {
        alert("⚠️ 貼文內容包含敏感粗口言論。");
        return;
      }

      if (!newPostImageBase64) {
        alert("請選擇一張動態相片進行上傳！");
        return;
      }

      const newPost = {
        id: `post-${Date.now()}`,
        author: currentUser.memberId,
        text: censoredText,
        image: newPostImageBase64,
        likes: [],
        createdAt: Date.now()
      };

      posts.unshift(newPost);
      saveData("6a_posts", posts);
      closeCreatePostModal();
      renderInstagramFeed();
    }

    function renderInstagramFeed() {
      const container = document.getElementById("instagram-feed-container");
      container.innerHTML = "";

      if (posts.length === 0) {
        container.innerHTML = `<div class="text-center text-slate-500 py-20 text-xs">📭 目前還沒有人發布動態，快來分享第一張貼文吧！</div>`;
        return;
      }

      posts.forEach(post => {
        const isLiked = post.likes?.includes(currentUser.memberId);
        
        const card = document.createElement("article");
        card.className = "bg-slate-900 border border-slate-800 rounded-2xl overflow-hidden shadow-xl max-w-md mx-auto";

        const deleteBtnHtml = (currentUser.memberId === SECRET_ADMIN || post.author === currentUser.memberId)
          ? `<button onclick="handleDeletePost('${post.id}')" class="text-slate-400 hover:text-red-400 text-xs bg-slate-800/80 p-2 rounded-xl transition"><svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg></button>`
          : "";

        card.innerHTML = `
          <div class="p-4 flex items-center justify-between border-b border-slate-800/60">
            <div class="flex items-center space-x-3">
              <div class="w-10 h-10 rounded-full bg-gradient-to-tr from-yellow-400 via-pink-500 to-purple-600 p-0.5">
                <div class="w-full h-full rounded-full bg-slate-900 flex items-center justify-center text-xs font-black">
                  ${post.author}
                </div>
              </div>
              <div>
                <span class="font-extrabold text-sm block text-slate-100">${post.author}</span>
                <span class="text-[10px] text-slate-500 font-medium">${new Date(post.createdAt).toLocaleDateString()}</span>
              </div>
            </div>
            ${deleteBtnHtml}
          </div>

          <div class="aspect-square bg-slate-950 flex items-center justify-center overflow-hidden">
            <img src="${post.image}" alt="貼文圖片" class="w-full h-full object-cover">
          </div>

          <div class="p-4 space-y-2">
            <button onclick="handleLikePost('${post.id}')" class="flex items-center space-x-2 text-slate-300 hover:text-pink-500 transition group">
              <svg class="w-5 h-5 ${isLiked ? 'text-pink-500 fill-pink-500' : 'text-slate-400'}" fill="${isLiked ? 'currentColor' : 'none'}" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"></path></svg>
              <span class="text-xs font-black">${post.likes?.length || 0} 個人覺得讚</span>
            </button>

            <div class="text-sm leading-relaxed">
              <span class="font-black mr-2 text-pink-400">${post.author}</span>
              <span class="text-slate-200 whitespace-pre-wrap break-all font-medium">${post.text}</span>
            </div>
          </div>
        `;
        container.appendChild(card);
      });
    }

    function handleLikePost(id) {
      const post = posts.find(p => p.id === id);
      if (!post) return;

      if (post.likes.includes(currentUser.memberId)) {
        post.likes = post.likes.filter(uid => uid !== currentUser.memberId);
      } else {
        post.likes.push(currentUser.memberId);
      }

      saveData("6a_posts", posts);
      renderInstagramFeed();
    }

    function handleDeletePost(id) {
      posts = posts.filter(p => p.id !== id);
      saveData("6a_posts", posts);
      renderInstagramFeed();
    }

    // --- 12. 功能提案公投 ---
    function openCreatePollModal() {
      document.getElementById("create-poll-modal").style.display = "flex";
    }

    function closeCreatePollModal() {
      document.getElementById("create-poll-modal").style.display = "none";
      document.getElementById("poll-title-input").value = "";
      document.getElementById("poll-desc-input").value = "";
    }

    function handleCreateProposal(e) {
      e.preventDefault();
      const title = document.getElementById("poll-title-input").value;
      const desc = document.getElementById("poll-desc-input").value;

      const newPoll = {
        id: `poll-${Date.now()}`,
        proposer: currentUser.memberId,
        title,
        description: desc,
        votes: [],
        createdAt: Date.now()
      };

      polls.unshift(newPoll);
      saveData("6a_polls", polls);
      closeCreatePollModal();
      renderPolls();
    }

    function renderPolls() {
      const grid = document.getElementById("polls-grid");
      grid.innerHTML = "";

      if (polls.length === 0) {
        grid.innerHTML = `<div class="text-center text-slate-500 col-span-2 py-10 text-xs">💡 目前沒有提案。點擊右上角按鈕發起第一個班級公投！</div>`;
        return;
      }

      polls.forEach(poll => {
        const totalVotes = poll.votes?.length || 0;
        const hasVoted = poll.votes?.includes(currentUser.memberId);
        const percentage = Math.min(Math.round((totalVotes / 32) * 100), 100);

        const card = document.createElement("div");
        card.className = "bg-slate-900 border border-slate-800 rounded-2xl p-5 space-y-4 shadow-lg";

        card.innerHTML = `
          <div class="flex justify-between items-start">
            <div>
              <span class="text-[10px] bg-slate-800 text-amber-400 px-2 py-0.5 rounded font-black">提案者: ${poll.proposer}</span>
              <h4 class="font-extrabold text-base text-slate-100 mt-2">${poll.title}</h4>
            </div>
            <button onclick="handleVotePoll('${poll.id}')" class="px-3 py-1.5 rounded-xl text-xs font-extrabold transition active:scale-95 flex items-center space-x-1 ${
              hasVoted ? "bg-amber-500 text-slate-950" : "bg-slate-800 hover:bg-slate-700 text-amber-400 border border-amber-500/30"
            }">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 10h4.764a2 2 0 011.789 2.894l-3.5 7A2 2 0 0115.263 21h-4.017c-.163-.024-.298-.111-.406-.242l-3-4a1 1 0 01.164-1.355l1.5-1.5a1 1 0 011.08-.204l1.41 1.41c.143.143.376.143.519 0l1.41-1.41a1 1 0 011.08-.204l1.5 1.5a1 1 0 01.164 1.355l-3 4"></path></svg>
              <span>${hasVoted ? "已投一票" : "投一票"}</span>
            </button>
          </div>

          <p class="text-xs text-slate-400 leading-relaxed font-medium">${poll.description || "未提供詳細介紹。"}</p>

          <div class="space-y-1.5">
            <div class="flex justify-between text-[11px] font-bold text-slate-400">
              <span>全班支持度 (${totalVotes}/32)</span>
              <span class="text-amber-400 font-extrabold">${percentage}%</span>
            </div>
            <div class="w-full bg-slate-950 h-2.5 rounded-full overflow-hidden border border-slate-800">
              <div class="bg-gradient-to-r from-amber-500 to-orange-500 h-full rounded-full transition-all duration-500" style="width: ${percentage}%"></div>
            </div>
          </div>
        `;
        grid.appendChild(card);
      });
    }

    function handleVotePoll(id) {
      const poll = polls.find(p => p.id === id);
      if (!poll) return;

      if (poll.votes.includes(currentUser.memberId)) {
        poll.votes = poll.votes.filter(uid => uid !== currentUser.memberId);
      } else {
        poll.votes.push(currentUser.memberId);
      }

      saveData("6a_polls", polls);
      renderPolls();
    }

    // --- 13. Kahoot! 問答系統 ---
    function renderKahootLobby() {
      const wrapper = document.getElementById("kahoot-board-wrapper");
      
      if (activeQuiz) {
        // 正進行遊戲中，此處略過渲染，保持遊戲邏輯運作
        return;
      }

      wrapper.innerHTML = `
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div class="md:col-span-2 space-y-4">
            <div class="flex justify-between items-center">
              <h3 class="font-extrabold text-sm text-slate-200">可挑戰的問答題目 (${quizzes.length})</h3>
              <button onclick="openCreateQuizModal()" class="bg-indigo-500 hover:bg-indigo-400 text-white font-extrabold text-xs py-2 px-3.5 rounded-xl transition active:scale-95">
                ➕ 新增自訂問答
              </button>
            </div>
            <div id="quiz-list-grid" class="grid grid-cols-1 sm:grid-cols-2 gap-4"></div>
          </div>

          <div class="bg-slate-900 border border-slate-800 rounded-2xl p-4 h-fit">
            <h3 class="font-black text-sm text-yellow-400 flex items-center space-x-1.5 mb-3 pb-2 border-b border-slate-800">
              <span>👑</span> <span>班級問答英雄榜</span>
            </h3>
            <div id="quiz-leaderboard" class="space-y-2.5"></div>
          </div>
        </div>
      `;

      // 渲染題目清單
      const listGrid = document.getElementById("quiz-list-grid");
      quizzes.forEach(quiz => {
        const div = document.createElement("div");
        div.className = "bg-slate-900 border border-slate-800 rounded-2xl p-5 space-y-4 shadow-md";
        div.innerHTML = `
          <div>
            <span class="text-[10px] bg-indigo-500/20 text-indigo-400 px-2 py-0.5 rounded font-black">出題人: ${quiz.creator}</span>
            <h4 class="font-extrabold text-base text-slate-200 mt-2">${quiz.title}</h4>
          </div>
          <button onclick="startKahootGame('${quiz.id}')" class="w-full bg-indigo-600 hover:bg-indigo-500 active:scale-95 text-white font-extrabold text-xs py-2 px-4 rounded-xl shadow transition">
            ⚡ 開始挑戰
          </button>
        `;
        listGrid.appendChild(div);
      });

      // 渲染排行榜
      const leaderboard = document.getElementById("quiz-leaderboard");
      if (quizScores.length === 0) {
        leaderboard.innerHTML = `<p class="text-slate-500 text-xs text-center py-4">目前尚無挑戰得分紀錄</p>`;
        return;
      }
      quizScores.slice(0, 8).forEach((score, index) => {
        const medal = index === 0 ? "🥇" : index === 1 ? "🥈" : index === 2 ? "🥉" : `${index + 1}.`;
        const item = document.createElement("div");
        item.className = "flex justify-between items-center bg-slate-950 p-2.5 rounded-xl border border-slate-800 text-xs";
        item.innerHTML = `
          <div class="flex items-center space-x-2">
            <span class="font-black text-sm text-yellow-500">${medal}</span>
            <div>
              <span class="font-black text-slate-200 block">${score.player}</span>
              <span class="text-[9px] text-slate-500 block truncate max-w-[120px]">${score.quizTitle}</span>
            </div>
          </div>
          <span class="font-black text-indigo-400">${score.score} 分</span>
        `;
        leaderboard.appendChild(item);
      });
    }

    function openCreateQuizModal() {
      renderQuizDraftQuestions();
      document.getElementById("create-quiz-modal").style.display = "flex";
    }

    function closeCreateQuizModal() {
      document.getElementById("create-quiz-modal").style.display = "none";
      document.getElementById("quiz-title-input").value = "";
      quizDraftQuestions = [{ question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 }];
    }

    function renderQuizDraftQuestions() {
      const wrapper = document.getElementById("quiz-questions-edit-wrapper");
      wrapper.innerHTML = "";

      quizDraftQuestions.forEach((q, idx) => {
        const div = document.createElement("div");
        div.className = "bg-slate-950 p-3.5 rounded-xl border border-slate-800 space-y-2.5";
        div.innerHTML = `
          <span class="text-[10px] bg-indigo-500/20 text-indigo-400 px-2 py-0.5 rounded font-black">問題 ${idx + 1}</span>
          <input type="text" placeholder="輸入問題本體" class="w-full bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs focus:outline-none text-white" value="${q.question}" onchange="updateQuizDraftQuestion(${idx}, 'question', this.value)" required>
          <div class="grid grid-cols-2 gap-2">
            <input type="text" placeholder="選項 1" class="bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs text-white focus:outline-none" value="${q.options[0]}" onchange="updateQuizDraftOption(${idx}, 0, this.value)" required>
            <input type="text" placeholder="選項 2" class="bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs text-white focus:outline-none" value="${q.options[1]}" onchange="updateQuizDraftOption(${idx}, 1, this.value)" required>
            <input type="text" placeholder="選項 3" class="bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs text-white focus:outline-none" value="${q.options[2]}" onchange="updateQuizDraftOption(${idx}, 2, this.value)" required>
            <input type="text" placeholder="選項 4" class="bg-slate-800 border border-slate-700 rounded-lg py-1.5 px-3 text-xs text-white focus:outline-none" value="${q.options[3]}" onchange="updateQuizDraftOption(${idx}, 3, this.value)" required>
          </div>
          <div class="flex justify-between items-center pt-1.5">
            <div class="flex items-center space-x-2 text-xs">
              <span class="text-slate-400">正確答案選項:</span>
              <select class="bg-slate-800 border border-slate-700 rounded p-1 text-xs" onchange="updateQuizDraftQuestion(${idx}, 'correctIndex', parseInt(this.value))">
                <option value="0" ${q.correctIndex === 0 ? "selected" : ""}>選項 1</option>
                <option value="1" ${q.correctIndex === 1 ? "selected" : ""}>選項 2</option>
                <option value="2" ${q.correctIndex === 2 ? "selected" : ""}>選項 3</option>
                <option value="3" ${q.correctIndex === 3 ? "selected" : ""}>選項 4</option>
              </select>
            </div>
          </div>
        `;
        wrapper.appendChild(div);
      });
    }

    function addQuestionToQuizDraft() {
      quizDraftQuestions.push({ question: "", options: ["", "", "", ""], correctIndex: 0, timeLimit: 15 });
      renderQuizDraftQuestions();
    }

    function updateQuizDraftQuestion(idx, field, value) {
      quizDraftQuestions[idx][field] = value;
    }

    function updateQuizDraftOption(qIdx, optIdx, value) {
      quizDraftQuestions[qIdx].options[optIdx] = value;
    }

    function handleCreateQuiz(e) {
      e.preventDefault();
      const title = document.getElementById("quiz-title-input").value;
      if (!title.trim()) return;

      const newQuiz = {
        id: `quiz-${Date.now()}`,
        creator: currentUser.memberId,
        title: title,
        questions: quizDraftQuestions,
        createdAt: Date.now()
      };

      quizzes.unshift(newQuiz);
      saveData("6a_quizzes", quizzes);
      closeCreateQuizModal();
      renderKahootLobby();
    }

    function startKahootGame(id) {
      const q = quizzes.find(item => item.id === id);
      if (!q) return;

      activeQuiz = q;
      currentQuestionIndex = 0;
      quizScore = 0;
      quizFinished = false;
      selectedAnswerIndex = null;
      quizTimer = q.questions[0].timeLimit || 15;

      renderKahootGameScreen();
    }

    function renderKahootGameScreen() {
      const wrapper = document.getElementById("kahoot-board-wrapper");
      
      if (quizFinished) {
        wrapper.innerHTML = `
          <div class="bg-slate-900 border border-indigo-500/40 rounded-3xl p-6 shadow-2xl text-center py-8 space-y-5">
            <span class="text-6xl block">🏆</span>
            <h3 class="text-2xl font-black text-indigo-400">恭喜完成挑戰！</h3>
            <div class="text-4xl font-black text-yellow-400 tracking-wider">${quizScore} 分</div>
            <div class="pt-4">
              <button onclick="exitKahootGame()" class="bg-indigo-600 hover:bg-indigo-500 text-white font-extrabold text-xs py-3 px-6 rounded-xl shadow-lg transition active:scale-95">
                返回問答大廳
              </button>
            </div>
          </div>
        `;
        return;
      }

      const q = activeQuiz.questions[currentQuestionIndex];
      wrapper.innerHTML = `
        <div class="bg-slate-900 border border-indigo-500/40 rounded-3xl p-6 space-y-6 shadow-2xl relative overflow-hidden">
          <div class="absolute -top-10 -right-10 w-40 h-40 bg-indigo-500/10 rounded-full blur-2xl"></div>

          <div class="flex justify-between items-center pb-3 border-b border-slate-800">
            <div>
              <span class="text-xs text-indigo-400 font-black tracking-wider uppercase">問答: ${activeQuiz.title}</span>
              <h4 class="text-xs text-slate-400 mt-1 font-bold">問題數: ${currentQuestionIndex + 1} / ${activeQuiz.questions.length}</h4>
            </div>
            
            <div class="flex items-center space-x-2">
              <span class="text-xs text-slate-400 font-bold">限時剩餘:</span>
              <span id="kahoot-timer-display" class="text-xl font-black px-3 py-1 rounded-xl bg-indigo-500/20 text-indigo-400">
                ⏱️ ${quizTimer}s
              </span>
            </div>
          </div>

          <div class="text-center py-6">
            <h3 class="text-xl font-extrabold text-slate-100 leading-relaxed">${q.question}</h3>
          </div>

          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3" id="kahoot-answers-grid"></div>

          <div class="text-right text-xs text-slate-400 font-bold">
            累積得分: <span class="text-indigo-400 text-sm font-black">${quizScore} 分</span>
          </div>
        </div>
      `;

      // 渲染選項按鈕
      const grid = document.getElementById("kahoot-answers-grid");
      const colors = [
        "bg-rose-600 hover:bg-rose-500 text-white",
        "bg-blue-600 hover:bg-blue-500 text-white",
        "bg-yellow-500 hover:bg-yellow-400 text-slate-950",
        "bg-emerald-600 hover:bg-emerald-500 text-white"
      ];
      const symbols = ["🔺", "🔷", "🟡", "🟩"];

      q.options.forEach((option, idx) => {
        const btn = document.createElement("button");
        btn.className = `p-4 rounded-2xl font-black text-sm flex items-center space-x-3 transition duration-150 transform active:scale-95 text-left ${colors[idx]}`;
        btn.onclick = () => submitKahootAnswer(idx);
        btn.innerHTML = `<span class="text-base">${symbols[idx]}</span> <span class="flex-1">${option}</span>`;
        grid.appendChild(btn);
      });
    }

    function submitKahootAnswer(idx) {
      if (selectedAnswerIndex !== null) return; // 防止重複答題
      selectedAnswerIndex = idx;

      const q = activeQuiz.questions[currentQuestionIndex];
      const correct = q.correctIndex;
      
      let scoreGained = 0;
      if (idx === correct) {
        // 答對，獲得基礎分數 + 計時時間差加權
        scoreGained = 100 + Math.round(quizTimer * 5);
        quizScore += scoreGained;
      }

      // 視覺反饋
      const grid = document.getElementById("kahoot-answers-grid");
      grid.innerHTML = "";
      const colors = [
        "bg-rose-600 hover:bg-rose-500 text-white",
        "bg-blue-600 hover:bg-blue-500 text-white",
        "bg-yellow-500 hover:bg-yellow-400 text-slate-950",
        "bg-emerald-600 hover:bg-emerald-500 text-white"
      ];
      const symbols = ["🔺", "🔷", "🟡", "🟩"];

      q.options.forEach((option, oIdx) => {
        const btn = document.createElement("button");
        btn.disabled = true;

        let statusStyle = colors[oIdx];
        if (oIdx === correct) {
          statusStyle = "bg-emerald-500 text-white ring-4 ring-emerald-300 animate-pulse";
        } else if (oIdx === idx) {
          statusStyle = "bg-rose-500 text-white ring-4 ring-rose-300 opacity-90";
        } else {
          statusStyle = "bg-slate-800 text-slate-500 opacity-30 cursor-not-allowed";
        }

        btn.className = `p-4 rounded-2xl font-black text-sm flex items-center space-x-3 text-left ${statusStyle}`;
        btn.innerHTML = `<span class="text-base">${symbols[oIdx]}</span> <span class="flex-1">${option}</span>`;
        grid.appendChild(btn);
      });

      // 1.2 秒後切換下一題
      setTimeout(() => {
        if (currentQuestionIndex + 1 < activeQuiz.questions.length) {
          currentQuestionIndex++;
          selectedAnswerIndex = null;
          quizTimer = activeQuiz.questions[currentQuestionIndex].timeLimit || 15;
          renderKahootGameScreen();
        } else {
          quizFinished = true;
          // 儲存排行榜分數
          const record = {
            id: `score-${Date.now()}`,
            player: currentUser.memberId,
            quizTitle: activeQuiz.title,
            score: quizScore,
            timestamp: Date.now()
          };
          quizScores.push(record);
          quizScores.sort((a, b) => b.score - a.score);
          saveData("6a_quiz_scores", quizScores);

          renderKahootGameScreen();
        }
      }, 1200);
    }

    function exitKahootGame() {
      activeQuiz = null;
      renderKahootLobby();
    }

    // --- 14. 隱形管理維護管理 (僅 6A32 特權) ---
    function renderBannedMembers() {
      const container = document.getElementById("banned-members-list");
      if (!container) return;
      container.innerHTML = "";

      CLASS_MEMBERS.filter(m => m !== SECRET_ADMIN).forEach(m => {
        const isUserBanned = memberStatus[m]?.isBanned || false;
        const card = document.createElement("div");
        card.className = `p-3 rounded-xl border flex items-center justify-between transition ${
          isUserBanned ? 'bg-red-950/20 border-red-900/60' : 'bg-slate-800/50 border-slate-700/60'
        }`;

        card.innerHTML = `
          <div class="flex items-center space-x-2">
            <div class="w-8 h-8 rounded-full flex items-center justify-center text-xs font-bold ${
              isUserBanned ? 'bg-red-600 text-white' : 'bg-emerald-600 text-white'
            }">
              ${m[2]+m[3]}
            </div>
            <div>
              <span class="font-bold text-sm text-slate-100">${m}</span>
            </div>
          </div>
          <button onclick="toggleBanUser('${m}')" class="text-xs px-2.5 py-1.5 rounded-lg font-bold transition ${
            isUserBanned ? 'bg-emerald-500 hover:bg-emerald-600 text-slate-950 shadow-md' : 'bg-red-500/20 hover:bg-red-500/30 text-red-400 border border-red-500/30'
          }">
            ${isUserBanned ? "解禁" : "禁言"}
          </button>
        `;
        container.appendChild(card);
      });
    }

    // --- 15. 刪除言論 & 貼文行為控制 ---
    function handleDeleteMsg(id) {
      if (currentUser.memberId !== SECRET_ADMIN) return;
      messages = messages.filter(m => m.id !== id);
      saveData("6a_messages", messages);
      renderChatMessages();
    }
  </script>
</body>
</html>
