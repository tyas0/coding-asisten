# coding-asisten<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CodeForge AI — Asisten Coding Cerdas</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;600;700&family=Space+Grotesk:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism-tomorrow.min.css" rel="stylesheet"/>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css"/>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        display: ['Space Grotesk', 'sans-serif'],
                        mono: ['JetBrains Mono', 'monospace'],
                    }
                }
            }
        }
    </script>
    <style>
        :root {
            --bg-primary: #08080d;
            --bg-secondary: #0f0f18;
            --bg-tertiary: #161622;
            --bg-elevated: #1e1e2e;
            --accent: #00e68a;
            --accent-hover: #00ff99;
            --accent-dim: #00b36b;
            --accent-glow: rgba(0, 230, 138, 0.1);
            --accent-glow-strong: rgba(0, 230, 138, 0.25);
            --text-primary: #e8e8f0;
            --text-secondary: #8888a0;
            --text-muted: #505068;
            --border: #22223a;
            --border-hover: #33334d;
            --success: #00e68a;
            --warning: #ffaa22;
            --error: #ff4466;
            --code-bg: #0b0b14;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            font-family: 'Space Grotesk', sans-serif;
            background: var(--bg-primary);
            color: var(--text-primary);
            overflow: hidden;
            height: 100vh;
        }

        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: var(--border-hover); }

        ::selection { background: var(--accent-glow-strong); color: var(--accent); }

        .sidebar {
            width: 280px;
            min-width: 280px;
            background: var(--bg-secondary);
            border-right: 1px solid var(--border);
            display: flex;
            flex-direction: column;
            transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            z-index: 40;
        }

        .sidebar-overlay {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.6);
            z-index: 35;
            backdrop-filter: blur(4px);
        }
        .sidebar-overlay.active { display: block; }

        .history-item {
            padding: 10px 14px;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.2s;
            border: 1px solid transparent;
            position: relative;
        }
        .history-item:hover {
            background: var(--bg-tertiary);
            border-color: var(--border);
        }
        .history-item.active {
            background: var(--accent-glow);
            border-color: var(--accent-dim);
        }
        .history-item .delete-btn {
            opacity: 0;
            transition: opacity 0.2s;
        }
        .history-item:hover .delete-btn { opacity: 1; }

        .code-container {
            background: var(--code-bg);
            border: 1px solid var(--border);
            border-radius: 12px;
            overflow: hidden;
            transition: border-color 0.3s;
        }
        .code-container:hover { border-color: var(--border-hover); }

        .code-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 12px 16px;
            background: var(--bg-tertiary);
            border-bottom: 1px solid var(--border);
            flex-wrap: wrap;
            gap: 8px;
        }

        .code-body {
            display: flex;
            overflow: auto;
            max-height: 65vh;
            position: relative;
        }

        .line-numbers {
            padding: 16px 0;
            text-align: right;
            color: var(--text-muted);
            font-family: 'JetBrains Mono', monospace;
            font-size: 13px;
            line-height: 1.7;
            user-select: none;
            min-width: 48px;
            padding-right: 12px;
            padding-left: 12px;
            border-right: 1px solid var(--border);
            flex-shrink: 0;
        }

        .code-body pre {
            margin: 0 !important;
            padding: 16px !important;
            background: transparent !important;
            font-size: 13px !important;
            line-height: 1.7 !important;
            flex: 1;
            overflow: visible;
        }
        .code-body pre code {
            font-family: 'JetBrains Mono', monospace !important;
            background: transparent !important;
            border: none !important;
            text-shadow: none !important;
        }

        .token.comment, .token.prolog, .token.doctype, .token.cdata { color: #505068; font-style: italic; }
        .token.punctuation { color: #8888a0; }
        .token.property, .token.tag, .token.boolean, .token.number, .token.constant, .token.symbol { color: #d19a66; }
        .token.selector, .token.attr-name, .token.string, .token.char, .token.builtin { color: #98c379; }
        .token.operator, .token.entity, .token.url { color: #56b6c2; }
        .token.atrule, .token.attr-value, .token.keyword { color: #c678dd; }
        .token.function, .token.class-name { color: #61afef; }
        .token.regex, .token.important, .token.variable { color: #e5c07b; }

        .typing-cursor::after {
            content: '';
            display: inline-block;
            width: 2px;
            height: 1.1em;
            background: var(--accent);
            margin-left: 2px;
            vertical-align: text-bottom;
            animation: cursorBlink 0.8s infinite;
        }

        @keyframes cursorBlink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0; }
        }

        .input-wrapper {
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: 14px;
            transition: border-color 0.3s, box-shadow 0.3s;
        }
        .input-wrapper:focus-within {
            border-color: var(--accent-dim);
            box-shadow: 0 0 0 3px var(--accent-glow), 0 0 30px var(--accent-glow);
        }

        .prompt-textarea {
            background: transparent;
            color: var(--text-primary);
            font-family: 'Space Grotesk', sans-serif;
            font-size: 15px;
            resize: none;
            outline: none;
            border: none;
            width: 100%;
            min-height: 48px;
            max-height: 160px;
            padding: 14px 16px;
            line-height: 1.5;
        }
        .prompt-textarea::placeholder { color: var(--text-muted); }

        .lang-select {
            background: var(--bg-tertiary);
            color: var(--text-secondary);
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 6px 12px;
            font-family: 'Space Grotesk', sans-serif;
            font-size: 13px;
            outline: none;
            cursor: pointer;
            appearance: none;
            -webkit-appearance: none;
            background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 24 24' fill='none' stroke='%238888a0' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='6 9 12 15 18 9'%3E%3C/polyline%3E%3C/svg%3E");
            background-repeat: no-repeat;
            background-position: right 8px center;
            padding-right: 30px;
            transition: border-color 0.2s;
        }
        .lang-select:hover { border-color: var(--border-hover); }
        .lang-select:focus { border-color: var(--accent-dim); }
        .lang-select option { background: var(--bg-tertiary); color: var(--text-primary); }

        .generate-btn {
            background: var(--accent);
            color: #080810;
            border: none;
            border-radius: 10px;
            padding: 10px 20px;
            font-family: 'Space Grotesk', sans-serif;
            font-weight: 600;
            font-size: 14px;
            cursor: pointer;
            transition: all 0.25s;
            display: flex;
            align-items: center;
            gap: 8px;
            white-space: nowrap;
        }
        .generate-btn:hover:not(:disabled) {
            background: var(--accent-hover);
            box-shadow: 0 0 20px var(--accent-glow-strong);
            transform: translateY(-1px);
        }
        .generate-btn:active:not(:disabled) { transform: translateY(0); }
        .generate-btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }

        .code-action-btn {
            background: var(--bg-elevated);
            color: var(--text-secondary);
            border: 1px solid var(--border);
            border-radius: 7px;
            padding: 5px 10px;
            font-size: 12px;
            font-family: 'Space Grotesk', sans-serif;
            cursor: pointer;
            transition: all 0.2s;
            display: inline-flex;
            align-items: center;
            gap: 5px;
        }
        .code-action-btn:hover {
            color: var(--text-primary);
            border-color: var(--border-hover);
            background: var(--bg-tertiary);
        }
        .code-action-btn.copied {
            color: var(--success);
            border-color: var(--accent-dim);
        }

        .lang-badge {
            font-size: 11px;
            font-weight: 600;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            padding: 3px 10px;
            border-radius: 5px;
            background: var(--accent-glow);
            color: var(--accent);
            border: 1px solid rgba(0,230,138,0.15);
        }

        .suggestion-chip {
            background: var(--bg-tertiary);
            border: 1px solid var(--border);
            color: var(--text-secondary);
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 13px;
            cursor: pointer;
            transition: all 0.25s;
            white-space: nowrap;
        }
        .suggestion-chip:hover {
            border-color: var(--accent-dim);
            color: var(--accent);
            background: var(--accent-glow);
            transform: translateY(-2px);
        }

        .toast-container {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 100;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .toast {
            padding: 12px 20px;
            border-radius: 10px;
            font-size: 13px;
            font-weight: 500;
            display: flex;
            align-items: center;
            gap: 10px;
            animation: toastIn 0.35s cubic-bezier(0.4, 0, 0.2, 1);
            backdrop-filter: blur(12px);
            border: 1px solid;
            box-shadow: 0 8px 30px rgba(0,0,0,0.4);
        }
        .toast.success { background: rgba(0,230,138,0.12); border-color: rgba(0,230,138,0.25); color: var(--accent); }
        .toast.error { background: rgba(255,68,102,0.12); border-color: rgba(255,68,102,0.25); color: var(--error); }
        .toast.info { background: rgba(97,175,239,0.12); border-color: rgba(97,175,239,0.25); color: #61afef; }
        .toast.out { animation: toastOut 0.3s forwards; }

        @keyframes toastIn {
            from { opacity: 0; transform: translateX(40px) scale(0.95); }
            to { opacity: 1; transform: translateX(0) scale(1); }
        }
        @keyframes toastOut {
            to { opacity: 0; transform: translateX(40px) scale(0.95); }
        }

        .thinking-dots { display: inline-flex; gap: 4px; align-items: center; padding: 8px 0; }
        .thinking-dots span {
            width: 8px; height: 8px;
            border-radius: 50%;
            background: var(--accent);
            animation: dotBounce 1.2s infinite;
        }
        .thinking-dots span:nth-child(2) { animation-delay: 0.15s; }
        .thinking-dots span:nth-child(3) { animation-delay: 0.3s; }

        @keyframes dotBounce {
            0%, 60%, 100% { transform: translateY(0); opacity: 0.3; }
            30% { transform: translateY(-8px); opacity: 1; }
        }

        .bg-orb {
            position: fixed;
            border-radius: 50%;
            filter: blur(100px);
            pointer-events: none;
            z-index: 0;
            opacity: 0.4;
        }
        .bg-orb-1 {
            width: 400px; height: 400px;
            background: radial-gradient(circle, rgba(0,230,138,0.08), transparent 70%);
            top: -100px; right: -100px;
            animation: orbFloat1 20s ease-in-out infinite;
        }
        .bg-orb-2 {
            width: 350px; height: 350px;
            background: radial-gradient(circle, rgba(0,180,107,0.06), transparent 70%);
            bottom: -80px; left: 20%;
            animation: orbFloat2 25s ease-in-out infinite;
        }
        .bg-orb-3 {
            width: 250px; height: 250px;
            background: radial-gradient(circle, rgba(0,230,138,0.05), transparent 70%);
            top: 40%; left: -50px;
            animation: orbFloat3 18s ease-in-out infinite;
        }

        @keyframes orbFloat1 {
            0%, 100% { transform: translate(0, 0) scale(1); }
            33% { transform: translate(-40px, 60px) scale(1.1); }
            66% { transform: translate(30px, -30px) scale(0.95); }
        }
        @keyframes orbFloat2 {
            0%, 100% { transform: translate(0, 0) scale(1); }
            50% { transform: translate(60px, -40px) scale(1.15); }
        }
        @keyframes orbFloat3 {
            0%, 100% { transform: translate(0, 0); }
            40% { transform: translate(30px, 50px); }
            80% { transform: translate(-20px, 20px); }
        }

        @keyframes fadeUp {
            from { opacity: 0; transform: translateY(16px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .fade-up { animation: fadeUp 0.5s cubic-bezier(0.4, 0, 0.2, 1) forwards; }
        .fade-up-d1 { animation-delay: 0.1s; opacity: 0; }
        .fade-up-d2 { animation-delay: 0.2s; opacity: 0; }
        .fade-up-d3 { animation-delay: 0.3s; opacity: 0; }
        .fade-up-d4 { animation-delay: 0.4s; opacity: 0; }

        .logo-icon {
            width: 36px; height: 36px;
            background: linear-gradient(135deg, var(--accent), var(--accent-dim));
            border-radius: 10px;
            display: flex; align-items: center; justify-content: center;
            font-size: 18px; color: #080810; font-weight: 700;
            box-shadow: 0 0 20px var(--accent-glow);
            position: relative;
        }
        .logo-icon::after {
            content: '';
            position: absolute;
            inset: -3px;
            border-radius: 13px;
            background: linear-gradient(135deg, var(--accent), transparent);
            opacity: 0.2;
            z-index: -1;
            animation: logoPulse 3s ease-in-out infinite;
        }
        @keyframes logoPulse {
            0%, 100% { opacity: 0.15; transform: scale(1); }
            50% { opacity: 0.3; transform: scale(1.05); }
        }

        .welcome-logo {
            width: 72px; height: 72px;
            background: linear-gradient(135deg, var(--accent), #00b36b);
            border-radius: 20px;
            display: flex; align-items: center; justify-content: center;
            font-size: 32px; color: #080810; font-weight: 700;
            box-shadow: 0 0 50px var(--accent-glow-strong);
            position: relative;
        }
        .welcome-logo::before {
            content: '';
            position: absolute;
            inset: -6px;
            border-radius: 26px;
            border: 1px solid rgba(0,230,138,0.15);
            animation: welcomePulse 2.5s ease-in-out infinite;
        }
        @keyframes welcomePulse {
            0%, 100% { transform: scale(1); opacity: 0.5; }
            50% { transform: scale(1.08); opacity: 1; }
        }

        .grid-pattern {
            position: fixed;
            inset: 0;
            background-image:
                linear-gradient(rgba(255,255,255,0.015) 1px, transparent 1px),
                linear-gradient(90deg, rgba(255,255,255,0.015) 1px, transparent 1px);
            background-size: 60px 60px;
            pointer-events: none;
            z-index: 0;
        }

        .new-chat-btn {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 10px 14px;
            border-radius: 10px;
            border: 1px dashed var(--border);
            background: transparent;
            color: var(--text-secondary);
            font-family: 'Space Grotesk', sans-serif;
            font-size: 13px;
            cursor: pointer;
            transition: all 0.2s;
            width: 100%;
        }
        .new-chat-btn:hover {
            border-color: var(--accent-dim);
            color: var(--accent);
            background: var(--accent-glow);
        }

        .hamburger-btn {
            display: none;
            background: var(--bg-tertiary);
            border: 1px solid var(--border);
            color: var(--text-secondary);
            width: 38px; height: 38px;
            border-radius: 8px;
            cursor: pointer;
            align-items: center;
            justify-content: center;
            font-size: 16px;
            transition: all 0.2s;
        }
        .hamburger-btn:hover { color: var(--text-primary); border-color: var(--border-hover); }

        /* Share modal */
        .modal-overlay {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.7);
            z-index: 80;
            backdrop-filter: blur(6px);
            align-items: center;
            justify-content: center;
        }
        .modal-overlay.active { display: flex; }
        .modal-box {
            background: var(--bg-secondary);
            border: 1px solid var(--border);
            border-radius: 16px;
            padding: 28px;
            max-width: 520px;
            width: 90%;
            box-shadow: 0 20px 60px rgba(0,0,0,0.5);
            animation: fadeUp 0.3s ease;
        }
        .modal-box h2 {
            font-size: 18px;
            font-weight: 700;
            margin-bottom: 8px;
            color: var(--text-primary);
        }
        .modal-box p {
            font-size: 13px;
            color: var(--text-secondary);
            line-height: 1.6;
            margin-bottom: 16px;
        }
        .host-step {
            display: flex;
            gap: 12px;
            align-items: flex-start;
            padding: 12px;
            border-radius: 10px;
            background: var(--bg-tertiary);
            border: 1px solid var(--border);
            margin-bottom: 8px;
            cursor: pointer;
            transition: all 0.2s;
        }
        .host-step:hover {
            border-color: var(--accent-dim);
            background: var(--accent-glow);
        }
        .host-step .step-num {
            width: 28px; height: 28px;
            border-radius: 8px;
            background: var(--accent-glow);
            color: var(--accent);
            display: flex; align-items: center; justify-content: center;
            font-size: 13px; font-weight: 700;
            flex-shrink: 0;
        }
        .host-step .step-title {
            font-size: 14px; font-weight: 600; color: var(--text-primary);
        }
        .host-step .step-desc {
            font-size: 12px; color: var(--text-secondary); margin-top: 2px;
        }
        .modal-close {
            background: var(--bg-elevated);
            border: 1px solid var(--border);
            color: var(--text-secondary);
            padding: 8px 20px;
            border-radius: 8px;
            font-family: 'Space Grotesk', sans-serif;
            font-size: 13px;
            cursor: pointer;
            transition: all 0.2s;
            margin-top: 12px;
            width: 100%;
            text-align: center;
        }
        .modal-close:hover {
            color: var(--text-primary);
            border-color: var(--border-hover);
        }

        .deco-line {
            width: 60px; height: 2px;
            background: linear-gradient(90deg, var(--accent), transparent);
            border-radius: 1px;
        }

        @media (max-width: 768px) {
            .sidebar {
                position: fixed;
                left: 0; top: 0; bottom: 0;
                transform: translateX(-100%);
            }
            .sidebar.open { transform: translateX(0); }
            .hamburger-btn { display: flex; }
            .code-body { max-height: 50vh; }
            .code-header { padding: 10px 12px; }
        }

        @media (prefers-reduced-motion: reduce) {
            *, *::before, *::after {
                animation-duration: 0.01ms !important;
                transition-duration: 0.01ms !important;
            }
        }
    </style>
</head>
<body>
    <div class="grid-pattern"></div>
    <div class="bg-orb bg-orb-1"></div>
    <div class="bg-orb bg-orb-2"></div>
    <div class="bg-orb bg-orb-3"></div>

    <div class="toast-container" id="toastContainer"></div>

    <!-- Share Modal -->
    <div class="modal-overlay" id="shareModal">
        <div class="modal-box">
            <h2><i class="fa-solid fa-share-nodes" style="color: var(--accent); margin-right: 8px;"></i>Bagikan Website Ini</h2>
            <p>File HTML ini berjalan lokal di browser kamu. Untuk mendapatkan link yang bisa dibagikan, upload ke salah satu layanan gratis berikut:</p>

            <a href="https://app.netlify.com/drop" target="_blank" rel="noopener" class="host-step" style="text-decoration: none;">
                <div class="step-num">1</div>
                <div>
                    <div class="step-title">Netlify Drop <i class="fa-solid fa-arrow-up-right-from-square text-[10px]" style="color: var(--text-muted);"></i></div>
                    <div class="step-desc">Drag & drop file ini — langsung dapat link dalam 10 detik. Paling mudah.</div>
                </div>
            </a>

            <a href="https://pages.github.com/" target="_blank" rel="noopener" class="host-step" style="text-decoration: none;">
                <div class="step-num">2</div>
                <div>
                    <div class="step-title">GitHub Pages <i class="fa-solid fa-arrow-up-right-from-square text-[10px]" style="color: var(--text-muted);"></i></div>
                    <div class="step-desc">Upload ke repository GitHub, aktifkan Pages di Settings. Gratis & permanen.</div>
                </div>
            </a>

            <a href="https://vercel.com" target="_blank" rel="noopener" class="host-step" style="text-decoration: none;">
                <div class="step-num">3</div>
                <div>
                    <div class="step-title">Vercel <i class="fa-solid fa-arrow-up-right-from-square text-[10px]" style="color: var(--text-muted);"></i></div>
                    <div class="step-desc">Deploy instan dari dashboard. Cukup upload folder berisi file ini.</div>
                </div>
            </a>

            <button class="modal-close" id="closeShareModal">Tutup</button>
        </div>
    </div>

    <div class="sidebar-overlay" id="sidebarOverlay"></div>

    <div class="flex h-screen relative z-10">

        <aside class="sidebar" id="sidebar">
            <div class="p-4 border-b" style="border-color: var(--border);">
                <div class="flex items-center gap-3 mb-4">
                    <div class="logo-icon"><i class="fa-solid fa-code"></i></div>
                    <div>
                        <div class="font-bold text-sm" style="color: var(--text-primary);">CodeForge AI</div>
                        <div class="text-xs" style="color: var(--text-muted);">Asisten Coding</div>
                    </div>
                </div>
                <button class="new-chat-btn" id="newChatBtn">
                    <i class="fa-solid fa-plus"></i>
                    Sesi Baru
                </button>
            </div>

            <div class="flex-1 overflow-y-auto p-3" id="historyList">
                <div class="text-xs font-semibold uppercase tracking-wider px-2 py-2" style="color: var(--text-muted);">Riwayat</div>
                <div id="historyItems"></div>
                <div id="emptyHistory" class="text-center py-8">
                    <i class="fa-regular fa-clock text-2xl mb-2" style="color: var(--text-muted);"></i>
                    <p class="text-xs" style="color: var(--text-muted);">Belum ada riwayat</p>
                </div>
            </div>

            <div class="p-3 border-t flex flex-col gap-2" style="border-color: var(--border);">
                <button class="code-action-btn w-full justify-center" id="shareBtn">
                    <i class="fa-solid fa-share-nodes"></i>
                    Bagikan Website
                </button>
                <button class="code-action-btn w-full justify-center" id="clearHistoryBtn">
                    <i class="fa-solid fa-trash-can"></i>
                    Hapus Semua Riwayat
                </button>
            </div>
        </aside>

        <main class="flex-1 flex flex-col min-w-0">

            <header class="flex items-center gap-3 px-4 py-3 border-b" style="border-color: var(--border); background: var(--bg-secondary);">
                <button class="hamburger-btn" id="hamburgerBtn">
                    <i class="fa-solid fa-bars"></i>
                </button>
                <div class="flex items-center gap-2">
                    <div class="w-2 h-2 rounded-full" style="background: var(--accent); box-shadow: 0 0 8px var(--accent-glow-strong);"></div>
                    <span class="text-sm font-medium" style="color: var(--text-secondary);">CodeForge AI</span>
                </div>
                <div class="ml-auto flex items-center gap-2">
                    <button class="code-action-btn" id="shareBtnTop" title="Bagikan">
                        <i class="fa-solid fa-share-nodes"></i>
                        <span class="hidden sm:inline">Bagikan</span>
                    </button>
                    <span class="text-xs px-2 py-1 rounded-md" style="background: var(--bg-tertiary); color: var(--text-muted); border: 1px solid var(--border);">
                        <i class="fa-solid fa-circle text-[6px] mr-1" style="color: var(--accent);"></i>Mock Mode
                    </span>
                </div>
            </header>

            <div class="flex-1 overflow-y-auto p-4 md:p-6" id="outputArea">

                <div id="welcomeState" class="flex flex-col items-center justify-center min-h-full text-center px-4">
                    <div class="welcome-logo fade-up mb-6">
                        <i class="fa-solid fa-bolt"></i>
                    </div>
                    <h1 class="text-3xl md:text-4xl font-bold mb-3 fade-up fade-up-d1" style="color: var(--text-primary);">
                        CodeForge AI
                    </h1>
                    <p class="text-base mb-2 fade-up fade-up-d2" style="color: var(--text-secondary); max-width: 480px;">
                        Asisten coding cerdas yang membantu Anda menulis, memahami, dan men-debug kode program dalam berbagai bahasa pemrograman.
                    </p>
                    <div class="deco-line mx-auto mb-8 fade-up fade-up-d2"></div>
                    <p class="text-xs uppercase tracking-widest mb-4 fade-up fade-up-d3" style="color: var(--text-muted);">Coba salah satu</p>
                    <div class="flex flex-wrap justify-center gap-2 fade-up fade-up-d4" id="suggestionsContainer"></div>
                </div>

                <div id="codeState" class="hidden max-w-4xl mx-auto">
                    <div id="thinkingIndicator" class="hidden mb-4">
                        <div class="flex items-center gap-3 px-1">
                            <div class="thinking-dots">
                                <span></span><span></span><span></span>
                            </div>
                            <span class="text-sm" style="color: var(--text-secondary);">Menganalisis prompt dan menghasilkan kode...</span>
                        </div>
                    </div>

                    <div id="promptDisplay" class="hidden mb-4 p-4 rounded-xl" style="background: var(--bg-tertiary); border: 1px solid var(--border);">
                        <div class="flex items-start gap-3">
                            <div class="w-7 h-7 rounded-lg flex items-center justify-center flex-shrink-0 mt-0.5" style="background: var(--accent-glow); color: var(--accent);">
                                <i class="fa-solid fa-user text-xs"></i>
                            </div>
                            <p class="text-sm leading-relaxed" style="color: var(--text-primary);" id="promptText"></p>
                        </div>
                    </div>

                    <div id="codeBlock" class="hidden code-container">
                        <div class="code-header">
                            <div class="flex items-center gap-3">
                                <div class="flex gap-1.5">
                                    <span class="w-3 h-3 rounded-full" style="background: #ff5f57;"></span>
                                    <span class="w-3 h-3 rounded-full" style="background: #febc2e;"></span>
                                    <span class="w-3 h-3 rounded-full" style="background: #28c840;"></span>
                                </div>
                                <span class="lang-badge" id="codeLangBadge">PYTHON</span>
                            </div>
                            <div class="flex items-center gap-2">
                                <button class="code-action-btn" id="copyBtn" title="Salin kode">
                                    <i class="fa-regular fa-copy"></i>
                                    <span>Salin</span>
                                </button>
                                <button class="code-action-btn" id="downloadBtn" title="Unduh file">
                                    <i class="fa-solid fa-download"></i>
                                    <span>Unduh</span>
                                </button>
                                <button class="code-action-btn" id="regenerateBtn" title="Regenerasi">
                                    <i class="fa-solid fa-rotate"></i>
                                </button>
                            </div>
                        </div>
                        <div class="code-body">
                            <div class="line-numbers" id="lineNumbers">1</div>
                            <pre><code id="codeOutput"></code></pre>
                        </div>
                    </div>
                </div>
            </div>

            <div class="p-3 md:p-4 border-t" style="border-color: var(--border); background: var(--bg-secondary);">
                <div class="max-w-4xl mx-auto">
                    <div class="input-wrapper">
                        <div class="flex items-end gap-2 px-3 pb-3">
                            <select class="lang-select" id="languageSelect" aria-label="Pilih bahasa pemrograman">
                                <option value="python">Python</option>
                                <option value="javascript">JavaScript</option>
                                <option value="cpp">C++</option>
                                <option value="java">Java</option>
                                <option value="php">PHP</option>
                            </select>
                            <div class="flex-1"></div>
                            <button class="generate-btn" id="generateBtn" disabled>
                                <i class="fa-solid fa-wand-magic-sparkles"></i>
                                <span>Generate</span>
                            </button>
                        </div>
                        <textarea
                            class="prompt-textarea"
                            id="promptInput"
                            placeholder="Deskripsikan kode yang ingin Anda buat... (Ctrl+Enter untuk generate)"
                            rows="1"
                            aria-label="Prompt kode"
                        ></textarea>
                    </div>
                    <div class="flex items-center justify-between mt-2 px-1">
                        <span class="text-xs" style="color: var(--text-muted);" id="charCount">0 karakter</span>
                        <span class="text-xs" style="color: var(--text-muted);">
                            <kbd class="px-1.5 py-0.5 rounded text-[10px]" style="background: var(--bg-tertiary); border: 1px solid var(--border);">Ctrl</kbd>
                            +
                            <kbd class="px-1.5 py-0.5 rounded text-[10px]" style="background: var(--bg-tertiary); border: 1px solid var(--border);">Enter</kbd>
                        </span>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-python.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-cpp.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-java.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-php.min.js"></script>

    <script>
    (function() {
        'use strict';

        const state = {
            history: JSON.parse(localStorage.getItem('codeforge_history') || '[]'),
            currentId: null,
            isGenerating: false,
            currentCode: '',
            currentLanguage: 'python',
        };

        const $ = (sel) => document.querySelector(sel);

        const els = {
            sidebar: $('#sidebar'),
            sidebarOverlay: $('#sidebarOverlay'),
            hamburgerBtn: $('#hamburgerBtn'),
            newChatBtn: $('#newChatBtn'),
            clearHistoryBtn: $('#clearHistoryBtn'),
            shareBtn: $('#shareBtn'),
            shareBtnTop: $('#shareBtnTop'),
            shareModal: $('#shareModal'),
            closeShareModal: $('#closeShareModal'),
            historyItems: $('#historyItems'),
            emptyHistory: $('#emptyHistory'),
            welcomeState: $('#welcomeState'),
            codeState: $('#codeState'),
            thinkingIndicator: $('#thinkingIndicator'),
            promptDisplay: $('#promptDisplay'),
            promptText: $('#promptText'),
            codeBlock: $('#codeBlock'),
            codeOutput: $('#codeOutput'),
            lineNumbers: $('#lineNumbers'),
            codeLangBadge: $('#codeLangBadge'),
            copyBtn: $('#copyBtn'),
            downloadBtn: $('#downloadBtn'),
            regenerateBtn: $('#regenerateBtn'),
            promptInput: $('#promptInput'),
            languageSelect: $('#languageSelect'),
            generateBtn: $('#generateBtn'),
            charCount: $('#charCount'),
            suggestionsContainer: $('#suggestionsContainer'),
            toastContainer: $('#toastContainer'),
            outputArea: $('#outputArea'),
        };

        const langConfig = {
            python: { label: 'Python', prism: 'python', ext: '.py' },
            javascript: { label: 'JavaScript', prism: 'javascript', ext: '.js' },
            cpp: { label: 'C++', prism: 'cpp', ext: '.cpp' },
            java: { label: 'Java', prism: 'java', ext: '.java' },
            php: { label: 'PHP', prism: 'php', ext: '.php' },
        };

        const suggestions = [
            { text: 'Buat fungsi sorting di Python', lang: 'python' },
            { text: 'REST API dengan Fetch di JavaScript', lang: 'javascript' },
            { text: 'Implementasi linked list di C++', lang: 'cpp' },
            { text: 'Aplikasi CRUD dengan class di Java', lang: 'java' },
            { text: 'Form handling dan validasi di PHP', lang: 'php' },
            { text: 'Kalkulator scientific di Python', lang: 'python' },
        ];

        function renderSuggestions() {
            els.suggestionsContainer.innerHTML = suggestions.map(s => `
                <button class="suggestion-chip" data-text="${s.text}" data-lang="${s.lang}">
                    ${s.text}
                </button>
            `).join('');

            els.suggestionsContainer.querySelectorAll('.suggestion-chip').forEach(chip => {
                chip.addEventListener('click', () => {
                    els.promptInput.value = chip.dataset.text;
                    els.languageSelect.value = chip.dataset.lang;
                    updateCharCount();
                    updateGenerateBtn();
                    els.promptInput.focus();
                });
            });
        }

        // =============================================
        // DATABASE KODE MOCK
        // =============================================
        const codeDB = {
            python: {
                fibonacci: `def fibonacci(n, memo={}):
    """Menghitung bilangan Fibonacci ke-n menggunakan memoization."""
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo)
    return memo[n]

# Menampilkan 15 bilangan Fibonacci pertama
print("=== Deret Fibonacci ===")
for i in range(15):
    print(f"F({i}) = {fibonacci(i)}")`,
                sorting: `def quick_sort(arr):
    """Implementasi Quick Sort secara rekursif."""
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quick_sort(left) + middle + quick_sort(right)

def merge_sort(arr):
    """Implementasi Merge Sort."""
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result

data = [64, 34, 25, 12, 22, 11, 90, 45, 78, 3]
print(f"Data awal  : {data}")
print(f"Quick Sort : {quick_sort(data)}")
print(f"Merge Sort : {merge_sort(data)}")`,
                calculator: `class Kalkulator:
    """Kalkulator saintifik dengan riwayat perhitungan."""
    def __init__(self):
        self.riwayat = []
        self._hasil_terakhir = None

    def tambah(self, a, b):
        return self._hitung(a, b, lambda x, y: x + y, "+")

    def kurang(self, a, b):
        return self._hitung(a, b, lambda x, y: x - y, "-")

    def kali(self, a, b):
        return self._hitung(a, b, lambda x, y: x * y, "x")

    def bagi(self, a, b):
        if b == 0:
            raise ValueError("Pembagi tidak boleh nol!")
        return self._hitung(a, b, lambda x, y: x / y, "/")

    def pangkat(self, a, b):
        return self._hitung(a, b, lambda x, y: x ** y, "**")

    def akar(self, a):
        import math
        hasil = math.sqrt(a)
        self.riwayat.append(f"sqrt({a}) = {hasil}")
        self._hasil_terakhir = hasil
        return hasil

    def _hitung(self, a, b, operasi, simbol):
        hasil = operasi(a, b)
        self._hasil_terakhir = hasil
        self.riwayat.append(f"{a} {simbol} {b} = {hasil}")
        return hasil

    def tampilkan_riwayat(self):
        print("=== Riwayat Perhitungan ===")
        for i, item in enumerate(self.riwayat, 1):
            print(f"  {i}. {item}")

calc = Kalkulator()
print(calc.tambah(15, 7))
print(calc.kali(6, 8))
print(calc.pangkat(2, 10))
print(calc.bagi(1024, 4))
print(calc.akar(144))
calc.tampilkan_riwayat()`,
                default: null
            },
            javascript: {
                fibonacci: `function* fibonacciGenerator() {
    let a = 0, b = 1;
    while (true) {
        yield a;
        [a, b] = [b, a + b];
    }
}

function fibonacci(n, memo = new Map()) {
    if (memo.has(n)) return memo.get(n);
    if (n <= 1) return n;
    const result = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
    memo.set(n, result);
    return result;
}

const gen = fibonacciGenerator();
console.log("Fibonacci (Generator):");
for (let i = 0; i < 15; i++) {
    console.log(\`  F(\${i}) = \${gen.next().value}\`);
}

console.log("\\nFibonacci (Memoization):");
for (let i = 0; i < 15; i++) {
    console.log(\`  F(\${i}) = \${fibonacci(i)}\`);
}`,
                sorting: `function quickSort(arr) {
    if (arr.length <= 1) return arr;
    const pivot = arr[Math.floor(arr.length / 2)];
    const left = arr.filter(x => x < pivot);
    const middle = arr.filter(x => x === pivot);
    const right = arr.filter(x => x > pivot);
    return [...quickSort(left), ...middle, ...quickSort(right)];
}

function mergeSort(arr) {
    if (arr.length <= 1) return arr;
    const mid = Math.floor(arr.length / 2);
    const left = mergeSort(arr.slice(0, mid));
    const right = mergeSort(arr.slice(mid));
    return merge(left, right);
}

function merge(left, right) {
    const result = [];
    let i = 0, j = 0;
    while (i < left.length && j < right.length) {
        if (left[i] <= right[j]) result.push(left[i++]);
        else result.push(right[j++]);
    }
    return [...result, ...left.slice(i), ...right.slice(j)];
}

const data = [64, 34, 25, 12, 22, 11, 90, 45, 78, 3];
console.log("Data awal:", data);
console.log("Quick Sort:", quickSort(data));
console.log("Merge Sort:", mergeSort(data));`,
                fetch: `async function fetchUserData(userId) {
    const baseUrl = 'https://jsonplaceholder.typicode.com';
    try {
        const userRes = await fetch(\`\${baseUrl}/users/\${userId}\`);
        if (!userRes.ok) throw new Error(\`HTTP \${userRes.status}\`);
        const user = await userRes.json();

        const postsRes = await fetch(\`\${baseUrl}/posts?userId=\${userId}\`);
        if (!postsRes.ok) throw new Error(\`HTTP \${postsRes.status}\`);
        const posts = await postsRes.json();

        return {
            user: { name: user.name, email: user.email, company: user.company.name },
            totalPosts: posts.length,
            recentPosts: posts.slice(0, 3).map(p => ({
                title: p.title,
                body: p.body.substring(0, 80) + '...',
            })),
        };
    } catch (error) {
        console.error('Gagal mengambil data:', error.message);
        return null;
    }
}

fetchUserData(1).then(data => {
    if (data) {
        console.log(\`User: \${data.user.name}\`);
        console.log(\`Email: \${data.user.email}\`);
        console.log(\`Total Posts: \${data.totalPosts}\`);
        console.log('Recent Posts:', data.recentPosts);
    }
});`,
                default: null
            },
            cpp: {
                fibonacci: `#include <iostream>
#include <vector>
#include <unordered_map>
using namespace std;

long long fibonacci(int n, unordered_map<int, long long>& memo) {
    if (memo.count(n)) return memo[n];
    if (n <= 1) return n;
    memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
    return memo[n];
}

long long fibonacciIteratif(int n) {
    if (n <= 1) return n;
    long long prev2 = 0, prev1 = 1, current;
    for (int i = 2; i <= n; i++) {
        current = prev2 + prev1;
        prev2 = prev1;
        prev1 = current;
    }
    return prev1;
}

int main() {
    unordered_map<int, long long> memo;
    cout << "=== Deret Fibonacci ===" << endl;
    cout << "n\\tRekursif\\tIteratif" << endl;
    for (int i = 0; i <= 20; i++) {
        cout << i << "\\t" << fibonacci(i, memo)
             << "\\t\\t" << fibonacciIteratif(i) << endl;
    }
    return 0;
}`,
                sorting: `#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

template<typename T>
void quickSort(vector<T>& arr, int low, int high) {
    if (low < high) {
        T pivot = arr[high];
        int i = low - 1;
        for (int j = low; j < high; j++) {
            if (arr[j] <= pivot) {
                i++;
                swap(arr[i], arr[j]);
            }
        }
        swap(arr[i + 1], arr[high]);
        int pi = i + 1;
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

template<typename T>
void merge(vector<T>& arr, int left, int mid, int right) {
    vector<T> temp(right - left + 1);
    int i = left, j = mid + 1, k = 0;
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) temp[k++] = arr[i++];
        else temp[k++] = arr[j++];
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    for (int i = 0; i < k; i++) arr[left + i] = temp[i];
}

template<typename T>
void mergeSort(vector<T>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

int main() {
    vector<int> data = {64, 34, 25, 12, 22, 11, 90, 45, 78, 3};
    vector<int> d1 = data, d2 = data;
    quickSort(d1, 0, d1.size() - 1);
    mergeSort(d2, 0, d2.size() - 1);

    cout << "Data awal  : ";
    for (int x : data) cout << x << " ";
    cout << "\\nQuick Sort : ";
    for (int x : d1) cout << x << " ";
    cout << "\\nMerge Sort : ";
    for (int x : d2) cout << x << " ";
    return 0;
}`,
                class_template: `#include <iostream>
#include <string>
#include <vector>
using namespace std;

class Mahasiswa {
private:
    string nama;
    string nim;
    vector<double> nilai;
public:
    Mahasiswa(string n, string id) : nama(n), nim(id) {}
    void tambahNilai(double n) { nilai.push_back(n); }
    double hitungRataRata() const {
        if (nilai.empty()) return 0.0;
        double total = 0;
        for (double n : nilai) total += n;
        return total / nilai.size();
    }
    string getPredikat() const {
        double rata = hitungRataRata();
        if (rata >= 85) return "A (Sangat Baik)";
        if (rata >= 75) return "B (Baik)";
        if (rata >= 60) return "C (Cukup)";
        if (rata >= 50) return "D (Kurang)";
        return "E (Tidak Lulus)";
    }
    void tampilkanInfo() const {
        cout << "Nama     : " << nama << endl;
        cout << "NIM      : " << nim << endl;
        cout << "Rata-rata: " << hitungRataRata() << endl;
        cout << "Predikat : " << getPredikat() << endl;
    }
};

int main() {
    Mahasiswa m1("Budi Santoso", "2024001");
    m1.tambahNilai(85); m1.tambahNilai(90);
    m1.tambahNilai(78); m1.tambahNilai(92);
    m1.tampilkanInfo();
    cout << endl;
    Mahasiswa m2("Siti Aminah", "2024002");
    m2.tambahNilai(70); m2.tambahNilai(65);
    m2.tambahNilai(72);
    m2.tampilkanInfo();
    return 0;
}`,
                default: null
            },
            java: {
                fibonacci: `import java.util.HashMap;
import java.util.Map;

public class Fibonacci {
    private static Map<Integer, Long> memo = new HashMap<>();

    public static long fibonacci(int n) {
        if (n <= 1) return n;
        if (memo.containsKey(n)) return memo.get(n);
        long result = fibonacci(n - 1) + fibonacci(n - 2);
        memo.put(n, result);
        return result;
    }

    public static long fibonacciIteratif(int n) {
        if (n <= 1) return n;
        long prev2 = 0, prev1 = 1, current = 0;
        for (int i = 2; i <= n; i++) {
            current = prev2 + prev1;
            prev2 = prev1;
            prev1 = current;
        }
        return prev1;
    }

    public static void main(String[] args) {
        System.out.println("=== Deret Fibonacci ===");
        System.out.println("n\\tRekursif\\tIteratif");
        for (int i = 0; i <= 20; i++) {
            System.out.printf("%d\\t%d\\t\\t%d%n",
                i, fibonacci(i), fibonacciIteratif(i));
        }
    }
}`,
                sorting: `import java.util.Arrays;

public class SortingDemo {
    public static <T extends Comparable<T>> void quickSort(T[] arr, int low, int high) {
        if (low < high) {
            int pi = partition(arr, low, high);
            quickSort(arr, low, pi - 1);
            quickSort(arr, pi + 1, high);
        }
    }

    private static <T extends Comparable<T>> int partition(T[] arr, int low, int high) {
        T pivot = arr[high];
        int i = low - 1;
        for (int j = low; j < high; j++) {
            if (arr[j].compareTo(pivot) <= 0) {
                i++;
                T temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;
            }
        }
        T temp = arr[i + 1]; arr[i + 1] = arr[high]; arr[high] = temp;
        return i + 1;
    }

    public static <T extends Comparable<T>> void mergeSort(T[] arr, int left, int right) {
        if (left < right) {
            int mid = left + (right - left) / 2;
            mergeSort(arr, left, mid);
            mergeSort(arr, mid + 1, right);
            merge(arr, left, mid, right);
        }
    }

    private static <T extends Comparable<T>> void merge(T[] arr, int left, int mid, int right) {
        Object[] temp = new Object[right - left + 1];
        int i = left, j = mid + 1, k = 0;
        while (i <= mid && j <= right) {
            if (arr[i].compareTo(arr[j]) <= 0) temp[k++] = arr[i++];
            else temp[k++] = arr[j++];
        }
        while (i <= mid) temp[k++] = arr[i++];
        while (j <= right) temp[k++] = arr[j++];
        for (int idx = 0; idx < k; idx++) arr[left + idx] = (T) temp[idx];
    }

    public static void main(String[] args) {
        Integer[] data = {64, 34, 25, 12, 22, 11, 90, 45, 78, 3};
        Integer[] d1 = data.clone(), d2 = data.clone();
        quickSort(d1, 0, d1.length - 1);
        mergeSort(d2, 0, d2.length - 1);
        System.out.println("Data awal  : " + Arrays.toString(data));
        System.out.println("Quick Sort : " + Arrays.toString(d1));
        System.out.println("Merge Sort : " + Arrays.toString(d2));
    }
}`,
                class_oop: `import java.util.ArrayList;
import java.util.List;

class Produk {
    private String nama;
    private double harga;
    private int stok;

    public Produk(String nama, double harga, int stok) {
        this.nama = nama; this.harga = harga; this.stok = stok;
    }
    public String getNama() { return nama; }
    public double getHarga() { return harga; }
    public int getStok() { return stok; }
    public void tambahStok(int jumlah) { if (jumlah > 0) stok += jumlah; }
    public boolean kurangiStok(int jumlah) {
        if (jumlah <= stok) { stok -= jumlah; return true; }
        return false;
    }
    @Override
    public String toString() {
        return String.format("%-20s | Rp %,10.2f | Stok: %d", nama, harga, stok);
    }
}

class Toko {
    private String namaToko;
    private List<Produk> daftarProduk = new ArrayList<>();

    public Toko(String namaToko) { this.namaToko = namaToko; }
    public void tambahProduk(Produk p) { daftarProduk.add(p); }
    public void tampilkanSemua() {
        System.out.println("=== " + namaToko + " ===");
        System.out.printf("%-20s | %13s | %s%n", "Nama", "Harga", "Stok");
        System.out.println("-".repeat(50));
        for (Produk p : daftarProduk) System.out.println(p);
    }
    public double hitungTotalNilai() {
        return daftarProduk.stream()
            .mapToDouble(p -> p.getHarga() * p.getStok()).sum();
    }
}

public class Main {
    public static void main(String[] args) {
        Toko toko = new Toko("Toko Sejahtera");
        toko.tambahProduk(new Produk("Laptop ASUS", 12500000, 5));
        toko.tambahProduk(new Produk("Mouse Logitech", 350000, 20));
        toko.tambahProduk(new Produk("Keyboard Mechanical", 850000, 12));
        toko.tambahProduk(new Produk("Monitor 27 inch", 3200000, 8));
        toko.tampilkanSemua();
        System.out.printf("%nTotal Nilai: Rp %,.2f%n", toko.hitungTotalNilai());
    }
}`,
                default: null
            },
            php: {
                fibonacci: `<?php

function fibonacci(int $n, array &$memo = []): int {
    if (array_key_exists($n, $memo)) return $memo[$n];
    if ($n <= 1) return $n;
    $memo[$n] = fibonacci($n - 1, $memo) + fibonacci($n - 2, $memo);
    return $memo[$n];
}

echo "=== Deret Fibonacci ===\\n";
echo str_pad("n", 5) . str_pad("Nilai", 15) . "\\n";
echo str_repeat("-", 20) . "\\n";

for ($i = 0; $i <= 20; $i++) {
    echo str_pad((string) $i, 5)
       . str_pad((string) fibonacci($i), 15) . "\\n";
}`,
                sorting: `<?php

function quickSort(array $arr): array {
    if (count($arr) <= 1) return $arr;
    $pivot = $arr[array_key_last($arr)];
    $left = $middle = $right = [];
    foreach ($arr as $val) {
        if ($val < $pivot) $left[] = $val;
        elseif ($val === $pivot) $middle[] = $val;
        else $right[] = $val;
    }
    return array_merge(quickSort($left), $middle, quickSort($right));
}

function mergeSort(array $arr): array {
    if (count($arr) <= 1) return $arr;
    $mid = intdiv(count($arr), 2);
    $left = mergeSort(array_slice($arr, 0, $mid));
    $right = mergeSort(array_slice($arr, $mid));
    return merge($left, $right);
}

function merge(array $left, array $right): array {
    $result = []; $i = $j = 0;
    while ($i < count($left) && $j < count($right)) {
        if ($left[$i] <= $right[$j]) $result[] = $left[$i++];
        else $result[] = $right[$j++];
    }
    while ($i < count($left)) $result[] = $left[$i++];
    while ($j < count($right)) $result[] = $right[$j++];
    return $result;
}

 $data = [64, 34, 25, 12, 22, 11, 90, 45, 78, 3];
echo "Data awal  : " . implode(", ", $data) . "\\n";
echo "Quick Sort : " . implode(", ", quickSort($data)) . "\\n";
echo "Merge Sort : " . implode(", ", mergeSort($data)) . "\\n";`,
                form: `<?php

class FormValidator {
    private array $errors = [];
    private array $data = [];

    public function validate(array $data, array $rules): bool {
        $this->data = $data;
        $this->errors = [];
        foreach ($rules as $field => $fieldRules) {
            $value = $data[$field] ?? '';
            foreach (explode('|', $fieldRules) as $rule) {
                $this->applyRule($field, $value, $rule);
            }
        }
        return empty($this->errors);
    }

    private function applyRule(string $field, string $value, string $rule): void {
        $param = null;
        if (str_contains($rule, ':')) {
            [$rule, $param] = explode(':', $rule, 2);
        }
        switch ($rule) {
            case 'required':
                if (empty(trim($value)))
                    $this->errors[$field][] = "Field {$field} wajib diisi";
                break;
            case 'email':
                if (!empty($value) && !filter_var($value, FILTER_VALIDATE_EMAIL))
                    $this->errors[$field][] = "Format email tidak valid";
                break;
            case 'min':
                if (strlen($value) < (int)$param)
                    $this->errors[$field][] = "Minimal {$param} karakter";
                break;
            case 'max':
                if (strlen($value) > (int)$param)
                    $this->errors[$field][] = "Maksimal {$param} karakter";
                break;
            case 'numeric':
                if (!empty($value) && !is_numeric($value))
                    $this->errors[$field][] = "Harus berupa angka";
                break;
        }
    }

    public function getErrors(): array { return $this->errors; }
    public function getData(): array { return $this->data; }

    public function displayErrors(): string {
        if (empty($this->errors)) return '';
        $html = "<div style='color:#ff4466;'>";
        foreach ($this->errors as $field => $errs) {
            foreach ($errs as $err) $html .= "<p>- {$err}</p>";
        }
        return $html . "</div>";
    }
}

// Simulasi pengiriman form
 $_POST = [
    'nama'  => 'Budi Santoso',
    'email' => 'budi@example.com',
    'umur'  => '25',
    'pesan' => 'Halo, ini pesan tes.',
];

 $validator = new FormValidator();
 $rules = [
    'nama'  => 'required|min:3|max:50',
    'email' => 'required|email',
    'umur'  => 'required|numeric|min:1|max:3',
    'pesan' => 'required|min:10',
];

if ($validator->validate($_POST, $rules)) {
    echo "<h3>Form berhasil disubmit!</h3>";
    echo "<pre>" . print_r($validator->getData(), true) . "</pre>";
} else {
    echo $validator->displayErrors();
}`,
                default: null
            }
        };

        function generateDefaultCode(prompt, language) {
            const ln = { python:'Python', javascript:'JavaScript', cpp:'C++', java:'Java', php:'PHP' };
            if (language === 'python') return `# ============================================\n# Solusi untuk: "${prompt}"\n# Bahasa: ${ln[language]}\n# ============================================\n\ndef main():\n    """Fungsi utama program."""\n    print("Memulai program...")\n    print(f"Prompt: ${prompt}")\n\n    # --- Tulis logika di sini ---\n    data = [i ** 2 for i in range(1, 6)]\n    print(f"\\nHasil: {data}")\n    return data\n\nif __name__ == "__main__":\n    hasil = main()\n    print(f"\\nSelesai. Total: {len(hasil)} item")`;
            if (language === 'javascript') return `// ============================================\n// Solusi untuk: "${prompt}"\n// Bahasa: ${ln[language]}\n// ============================================\n\nfunction main() {\n    console.log("Memulai program...");\n    console.log(\`Prompt: ${prompt}\`);\n\n    // --- Tulis logika di sini ---\n    const data = Array.from({length: 5}, (_, i) => (i + 1) ** 2);\n    console.log("\\nHasil:", data);\n    return data;\n}\n\nconst hasil = main();\nconsole.log(\`\\nSelesai. Total: \${hasil.length} item\`);`;
            if (language === 'cpp') return `// ============================================\n// Solusi untuk: "${prompt}"\n// Bahasa: ${ln[language]}\n// ============================================\n\n#include <iostream>\n#include <vector>\nusing namespace std;\n\nint main() {\n    cout << "Memulai program..." << endl;\n    cout << "Prompt: ${prompt}" << endl;\n\n    // --- Tulis logika di sini ---\n    vector<int> data;\n    for (int i = 1; i <= 5; i++) data.push_back(i * i);\n\n    cout << "\\nHasil: ";\n    for (int x : data) cout << x << " ";\n    cout << "\\nSelesai. Total: " << data.size() << " item" << endl;\n    return 0;\n}`;
            if (language === 'java') return `// ============================================\n// Solusi untuk: "${prompt}"\n// Bahasa: ${ln[language]}\n// ============================================\n\nimport java.util.ArrayList;\nimport java.util.List;\n\npublic class Solution {\n    public static void main(String[] args) {\n        System.out.println("Memulai program...");\n        System.out.println("Prompt: ${prompt}");\n\n        // --- Tulis logika di sini ---\n        List<Integer> data = new ArrayList<>();\n        for (int i = 1; i <= 5; i++) data.add(i * i);\n\n        System.out.println("\\nHasil: " + data);\n        System.out.println("Selesai. Total: " + data.size() + " item");\n    }\n}`;
            if (language === 'php') return `<?php\n// ============================================\n// Solusi untuk: "${prompt}"\n// Bahasa: ${ln[language]}\n// ============================================\n\nfunction main(): array {\n    echo "Memulai program...\\n";\n    echo "Prompt: ${prompt}\\n";\n\n    // --- Tulis logika di sini ---\n    $data = [];\n    for ($i = 1; $i <= 5; $i++) $data[] = $i * $i;\n\n    echo "\\nHasil: " . implode(", ", $data) . "\\n";\n    echo "Selesai. Total: " . count($data) . " item\\n";\n    return $data;\n}\n\n$hasil = main();`;
            return '// Kode tidak tersedia.';
        }

        function detectTopic(prompt) {
            const p = prompt.toLowerCase();
            const topics = [
                { id: 'fibonacci', keys: ['fibonacci', 'fib', 'deret fib'] },
                { id: 'sorting', keys: ['sort', 'urut', 'bubble sort', 'quick sort', 'merge sort', 'selection sort', 'insertion sort', 'pengurutan', 'mengurutkan'] },
                { id: 'calculator', keys: ['kalkulator', 'calculator', 'hitung', 'operasi matematika', 'aritmatika', 'scientific'] },
                { id: 'fetch', keys: ['fetch', 'api', 'http', 'request', 'rest', 'endpoint', 'get data'] },
                { id: 'class_template', keys: ['class', 'oop', 'object', 'template', 'generic', 'linked list'] },
                { id: 'class_oop', keys: ['class', 'oop', 'object', 'toko', 'produk', 'inventory', 'crud'] },
                { id: 'form', keys: ['form', 'validasi', 'validation', 'input', 'submit', 'post'] },
            ];
            for (const t of topics) if (t.keys.some(k => p.includes(k))) return t.id;
            return 'default';
        }

        // =============================================
        // GENERATE CODE — GANTI BAGIAN INI UNTUK API
        // =============================================
        async function generateCode(prompt, language) {
            // --------------------------------------------------
            // HUBUNGKAN KE API AI DI SINI
            //
            // Contoh OpenAI:
            //   const res = await fetch('https://api.openai.com/v1/chat/completions', {
            //       method: 'POST',
            //       headers: {
            //           'Content-Type': 'application/json',
            //           'Authorization': 'Bearer KUNCI_API_ANDA'
            //       },
            //       body: JSON.stringify({
            //           model: 'gpt-4',
            //           messages: [
            //               { role: 'system', content: 'Kamu asisten coding. Hanya berikan kode.' },
            //               { role: 'user', content: `Tulis kode ${language}: ${prompt}` }
            //           ],
            //           temperature: 0.3
            //       })
            //   });
            //   const data = await res.json();
            //   return data.choices[0].message.content;
            //
            // Contoh Ollama (lokal):
            //   const res = await fetch('http://localhost:11434/api/generate', {
            //       method: 'POST',
            //       headers: { 'Content-Type': 'application/json' },
            //       body: JSON.stringify({
            //           model: 'codellama',
            //           prompt: `Tulis kode ${language}: ${prompt}`,
            //           stream: false
            //       })
            //   });
            //   const data = await res.json();
            //   return data.response;
            // --------------------------------------------------

            await sleep(1200 + Math.random() * 800);
            const topic = detectTopic(prompt);
            const langTemplates = codeDB[language] || {};
            return langTemplates[topic] || generateDefaultCode(prompt, language);
        }

        async function typeCodeAnimation(codeText, language) {
            const codeEl = els.codeOutput;
            const lines = codeText.split('\n');
            let current = '';
            codeEl.className = '';
            codeEl.textContent = '';
            codeEl.classList.add('typing-cursor');
            updateLineNumbers('');

            for (let i = 0; i < lines.length; i++) {
                current += (i > 0 ? '\n' : '') + lines[i];
                codeEl.textContent = current;
                updateLineNumbers(current);
                const codeBody = codeEl.closest('.code-body');
                if (codeBody) codeBody.scrollTop = codeBody.scrollHeight;
                const lineLen = lines[i].length;
                const delay = lineLen > 60 ? 12 : lineLen > 30 ? 22 : 32;
                await sleep(delay);
            }

            codeEl.classList.remove('typing-cursor');
            codeEl.textContent = codeText;
            codeEl.className = `language-${langConfig[language].prism}`;
            Prism.highlightElement(codeEl);
            updateLineNumbers(codeText);
        }

        function updateLineNumbers(text) {
            const count = text ? text.split('\n').length : 1;
            els.lineNumbers.textContent = Array.from({ length: count }, (_, i) => i + 1).join('\n');
        }

        function showWelcomeState() {
            els.welcomeState.classList.remove('hidden');
            els.codeState.classList.add('hidden');
        }

        function showCodeState() {
            els.welcomeState.classList.add('hidden');
            els.codeState.classList.remove('hidden');
        }

        async function handleGenerate() {
            const prompt = els.promptInput.value.trim();
            if (!prompt || state.isGenerating) return;

            state.isGenerating = true;
            const language = els.languageSelect.value;
            state.currentLanguage = language;

            showCodeState();
            els.promptDisplay.classList.remove('hidden');
            els.promptText.textContent = prompt;
            els.thinkingIndicator.classList.remove('hidden');
            els.codeBlock.classList.add('hidden');
            updateGenerateBtn();
            els.outputArea.scrollTop = 0;

            try {
                const code = await generateCode(prompt, language);
                state.currentCode = code;
                els.thinkingIndicator.classList.add('hidden');
                els.codeBlock.classList.remove('hidden');
                els.codeLangBadge.textContent = langConfig[language].label.toUpperCase();
                await typeCodeAnimation(code, language);
                saveToHistory(prompt, language, code);
                resetCopyBtn();
            } catch (err) {
                els.thinkingIndicator.classList.add('hidden');
                showToast('Gagal menghasilkan kode: ' + err.message, 'error');
            }

            state.isGenerating = false;
            updateGenerateBtn();
        }

        // =============================================
        // RIWAYAT
        // =============================================
        function saveToHistory(prompt, language, code) {
            const item = {
                id: Date.now().toString(36) + Math.random().toString(36).substr(2, 5),
                prompt, language, code,
                timestamp: Date.now(),
            };
            state.history.unshift(item);
            state.currentId = item.id;
            if (state.history.length > 50) state.history.pop();
            saveHistoryToStorage();
            renderHistory();
        }

        function deleteHistoryItem(id) {
            state.history = state.history.filter(h => h.id !== id);
            if (state.currentId === id) { state.currentId = null; showWelcomeState(); }
            saveHistoryToStorage();
            renderHistory();
            showToast('Item dihapus dari riwayat', 'info');
        }

        function clearAllHistory() {
            if (state.history.length === 0) return;
            state.history = [];
            state.currentId = null;
            saveHistoryToStorage();
            renderHistory();
            showWelcomeState();
            showToast('Semua riwayat dihapus', 'info');
        }

        function loadHistoryItem(id) {
            const item = state.history.find(h => h.id === id);
            if (!item) return;
            state.currentId = item.id;
            state.currentCode = item.code;
            state.currentLanguage = item.language;
            els.promptInput.value = item.prompt;
            els.languageSelect.value = item.language;
            updateCharCount();
            updateGenerateBtn();

            showCodeState();
            els.promptDisplay.classList.remove('hidden');
            els.promptText.textContent = item.prompt;
            els.thinkingIndicator.classList.add('hidden');
            els.codeBlock.classList.remove('hidden');
            els.codeLangBadge.textContent = langConfig[item.language].label.toUpperCase();

            const codeEl = els.codeOutput;
            codeEl.className = `language-${langConfig[item.language].prism}`;
            codeEl.textContent = item.code;
            Prism.highlightElement(codeEl);
            updateLineNumbers(item.code);
            resetCopyBtn();
            renderHistory();
            closeSidebar();
        }

        function renderHistory() {
            const items = state.history;
            els.emptyHistory.classList.toggle('hidden', items.length > 0);
            els.historyItems.innerHTML = items.map(item => {
                const time = formatTime(item.timestamp);
                const isActive = item.id === state.currentId;
                const truncated = item.prompt.length > 38 ? item.prompt.substring(0, 38) + '...' : item.prompt;
                return `<div class="history-item ${isActive ? 'active' : ''} mb-1" data-id="${item.id}">
                    <div class="flex items-start gap-2">
                        <i class="fa-solid fa-code text-[10px] mt-1.5 flex-shrink-0" style="color: var(--text-muted);"></i>
                        <div class="flex-1 min-w-0">
                            <p class="text-xs truncate" style="color: ${isActive ? 'var(--accent)' : 'var(--text-secondary)'};">${escapeHtml(truncated)}</p>
                            <div class="flex items-center gap-2 mt-1">
                                <span class="text-[10px] px-1.5 py-0.5 rounded" style="background: var(--bg-primary); color: var(--text-muted);">${langConfig[item.language].label}</span>
                                <span class="text-[10px]" style="color: var(--text-muted);">${time}</span>
                            </div>
                        </div>
                        <button class="delete-btn text-[10px] p-1 rounded hover:text-red-400" style="color: var(--text-muted);" data-delete="${item.id}" title="Hapus">
                            <i class="fa-solid fa-xmark"></i>
                        </button>
                    </div>
                </div>`;
            }).join('');

            els.historyItems.querySelectorAll('.history-item').forEach(el => {
                el.addEventListener('click', (e) => {
                    if (e.target.closest('.delete-btn')) return;
                    loadHistoryItem(el.dataset.id);
                });
            });
            els.historyItems.querySelectorAll('.delete-btn').forEach(btn => {
                btn.addEventListener('click', (e) => { e.stopPropagation(); deleteHistoryItem(btn.dataset.delete); });
            });
        }

        function saveHistoryToStorage() {
            try { localStorage.setItem('codeforge_history', JSON.stringify(state.history)); }
            catch (e) { /* Storage penuh */ }
        }

        // =============================================
        // SALIN KODE — FIXED: bekerja di file:// juga
        // =============================================
        async function copyCode() {
            if (!state.currentCode) return;

            let success = false;

            // Coba Clipboard API (hanya HTTPS/localhost)
            if (navigator.clipboard && window.isSecureContext) {
                try {
                    await navigator.clipboard.writeText(state.currentCode);
                    success = true;
                } catch (e) {
                    success = false;
                }
            }

            // Fallback: textarea tersembunyi (berfungsi di file://)
            if (!success) {
                try {
                    const ta = document.createElement('textarea');
                    ta.value = state.currentCode;
                    ta.style.cssText = 'position:fixed;left:-9999px;top:-9999px;opacity:0;width:1px;height:1px;';
                    document.body.appendChild(ta);
                    ta.focus();
                    ta.select();
                    ta.setSelectionRange(0, 99999999);
                    const ok = document.execCommand('copy');
                    document.body.removeChild(ta);
                    success = !!ok;
                } catch (e) {
                    success = false;
                }
            }

            if (success) {
                els.copyBtn.classList.add('copied');
                els.copyBtn.querySelector('span').textContent = 'Tersalin';
                els.copyBtn.querySelector('i').className = 'fa-solid fa-check';
                showToast('Kode berhasil disalin ke clipboard', 'success');
                setTimeout(resetCopyBtn, 2000);
            } else {
                // Jika semua gagal, buka prompt manual
                try {
                    window.prompt('Salin kode secara manual (Ctrl+C):', state.currentCode);
                    showToast('Gunakan Ctrl+C untuk menyalin', 'info');
                } catch (e) {
                    showToast('Gagal menyalin. Coba pilih kode secara manual.', 'error');
                }
            }
        }

        function resetCopyBtn() {
            els.copyBtn.classList.remove('copied');
            els.copyBtn.querySelector('span').textContent = 'Salin';
            els.copyBtn.querySelector('i').className = 'fa-regular fa-copy';
        }

        function downloadCode() {
            if (!state.currentCode) return;
            const ext = langConfig[state.currentLanguage].ext;
            const filename = `codeforge_output${ext}`;
            const blob = new Blob([state.currentCode], { type: 'text/plain' });
            const url = URL.createObjectURL(url);
            const a = document.createElement('a');
            a.href = URL.createObjectURL(blob);
            a.download = filename;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(a.href);
            showToast(`File ${filename} berhasil diunduh`, 'success');
        }

        // =============================================
        // TOAST
        // =============================================
        function showToast(message, type = 'info') {
            const toast = document.createElement('div');
            toast.className = `toast ${type}`;
            const icons = { success: 'fa-check-circle', error: 'fa-exclamation-circle', info: 'fa-info-circle' };
            toast.innerHTML = `<i class="fa-solid ${icons[type] || icons.info}"></i><span>${escapeHtml(message)}</span>`;
            els.toastContainer.appendChild(toast);
            setTimeout(() => {
                toast.classList.add('out');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }

        // =============================================
        // SIDEBAR & MODAL
        // =============================================
        function openSidebar() { els.sidebar.classList.add('open'); els.sidebarOverlay.classList.add('active'); }
        function closeSidebar() { els.sidebar.classList.remove('open'); els.sidebarOverlay.classList.remove('active'); }

        // =============================================
        // UTILITAS
        // =============================================
        function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }
        function escapeHtml(str) { const d = document.createElement('div'); d.textContent = str; return d.innerHTML; }
        function formatTime(ts) {
            const diff = Date.now() - ts;
            const m = Math.floor(diff / 60000), h = Math.floor(diff / 3600000), d = Math.floor(diff / 86400000);
            if (m < 1) return 'Baru saja';
            if (m < 60) return `${m}m lalu`;
            if (h < 24) return `${h}j lalu`;
            return `${d}h lalu`;
        }

        function updateCharCount() { els.charCount.textContent = `${els.promptInput.value.length} karakter`; }

        function updateGenerateBtn() {
            const hasText = els.promptInput.value.trim().length > 0;
            els.generateBtn.disabled = !hasText || state.isGenerating;
            if (state.isGenerating) {
                els.generateBtn.querySelector('span').textContent = 'Generating...';
                els.generateBtn.querySelector('i').className = 'fa-solid fa-spinner fa-spin';
            } else {
                els.generateBtn.querySelector('span').textContent = 'Generate';
                els.generateBtn.querySelector('i').className = 'fa-solid fa-wand-magic-sparkles';
            }
        }

        function autoResizeTextarea() {
            const ta = els.promptInput;
            ta.style.height = 'auto';
            ta.style.height = Math.min(ta.scrollHeight, 160) + 'px';
        }

        function newChat() {
            state.currentId = null;
            state.currentCode = '';
            els.promptInput.value = '';
            updateCharCount();
            updateGenerateBtn();
            autoResizeTextarea();
            showWelcomeState();
            closeSidebar();
            els.promptInput.focus();
        }

        // =============================================
        // EVENT LISTENERS
        // =============================================
        els.generateBtn.addEventListener('click', handleGenerate);
        els.promptInput.addEventListener('input', () => { updateCharCount(); updateGenerateBtn(); autoResizeTextarea(); });
        els.promptInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && (e.ctrlKey || e.metaKey)) { e.preventDefault(); handleGenerate(); }
        });
        els.copyBtn.addEventListener('click', copyCode);
        els.downloadBtn.addEventListener('click', downloadCode);
        els.regenerateBtn.addEventListener('click', () => { if (els.promptInput.value.trim() && !state.isGenerating) handleGenerate(); });
        els.hamburgerBtn.addEventListener('click', openSidebar);
        els.sidebarOverlay.addEventListener('click', closeSidebar);
        els.newChatBtn.addEventListener('click', newChat);
        els.clearHistoryBtn.addEventListener('click', clearAllHistory);

        // Share modal
        els.shareBtn.addEventListener('click', () => { els.shareModal.classList.add('active'); });
        els.shareBtnTop.addEventListener('click', () => { els.shareModal.classList.add('active'); });
        els.closeShareModal.addEventListener('click', () => { els.shareModal.classList.remove('active'); });
        els.shareModal.addEventListener('click', (e) => { if (e.target === els.shareModal) els.shareModal.classList.remove('active'); });

        document.addEventListener('keydown', (e) => {
            if (e.key === 'Escape') { closeSidebar(); els.shareModal.classList.remove('active'); }
        });

        // =============================================
        // INISIALISASI
        // =============================================
        renderSuggestions();
        renderHistory();
        updateGenerateBtn();

    })();
    </script>
</body>
</html>
