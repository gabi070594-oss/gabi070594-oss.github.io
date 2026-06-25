<!-- Chosen Palette: Slate, Indigo, Forest, Rose, Deep Dark -->
<!-- Application Structure Plan: The SPA features a dual-column layout. The left column focuses on data input (raw paste and discrete form fields) for data extraction. The right column acts as a live dashboard, presenting the formatted transcript output, a data completeness chart (Chart.js), and quick actions for copying/downloading. A theme customizer is integrated into the header to allow user personalization without code editing, adhering to the requirement of a visual-only customization method. -->
<!-- Visualization & Content Choices: The application uses a dynamic HTML output block to preview the transcript. A Chart.js Donut chart is used to visualize 'Data Completeness' (fields filled vs. empty) to gamify and encourage complete data entry. The theme picker uses standard HTML/CSS interactions and localStorage for state persistence. NO SVG or Mermaid JS is used. Icons are rendered via FontAwesome CSS classes. -->
<!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Email Transcript Tool</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght=300;400;500;600;700;800&family=JetBrains+Mono:wght=400;500&display=swap');
        
        body {
            font-family: 'Plus Jakarta Sans', sans-serif;
            transition: background-color 0.3s ease, color 0.3s ease;
        }
        .code-font {
            font-family: 'JetBrains Mono', monospace;
        }
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
        .dark ::-webkit-scrollbar-thumb {
            background: #475569;
        }
        .dark ::-webkit-scrollbar-thumb:hover {
            background: #64748b;
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 300px;
            margin-left: auto;
            margin-right: auto;
            height: 150px;
            max-height: 200px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 180px;
            }
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 dark:bg-slate-900 dark:text-slate-100 min-h-screen">

    <header class="border-b border-slate-200 dark:border-slate-800 bg-white/80 dark:bg-slate-900/80 backdrop-blur sticky top-0 z-50">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
            <div class="flex items-center gap-3">
                <div class="h-10 w-10 rounded-xl bg-indigo-600 flex items-center justify-center text-white shadow-md shadow-indigo-500/20">
                    <i class="fa-solid fa-file-invoice text-lg"></i>
                </div>
                <div>
                    <h1 class="text-lg font-bold tracking-tight text-slate-900 dark:text-white">TranscriptPro</h1>
                    <p class="text-xs text-slate-500 dark:text-slate-400">AI-Powered Support Note Formatter</p>
                </div>
            </div>
            
            <div class="flex items-center gap-2">
                <div id="cloudStatusBadge" class="hidden sm:flex items-center gap-1.5 px-3 py-1 rounded-full bg-slate-100 dark:bg-slate-800 text-xs font-semibold text-slate-500 dark:text-slate-400">
                    <span class="h-2 w-2 rounded-full bg-slate-400 animate-pulse"></span>
                    <span>Offline Sync Only</span>
                </div>
                <button id="colorSettingsBtn" class="flex items-center gap-2 px-3 py-1.5 rounded-lg bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 transition-colors text-sm font-semibold">
                    <i class="fa-solid fa-palette text-indigo-500"></i> Theme
                </button>
                <button id="themeToggle" class="p-2 rounded-lg bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 transition-colors">
                    <i id="themeIcon" class="fa-solid fa-moon text-slate-600 dark:text-slate-300"></i>
                </button>
                <button id="helpBtn" class="p-2 rounded-lg bg-slate-100 dark:bg-slate-800 hover:bg-slate-200 dark:hover:bg-slate-700 transition-colors">
                    <i class="fa-solid fa-circle-question text-slate-600 dark:text-slate-300"></i>
                </button>
            </div>
        </div>
    </header>

    <div id="sharedLoadBanner" class="hidden bg-gradient-to-r from-emerald-500 to-teal-600 text-white py-2.5 px-4 text-center text-xs font-bold shadow-sm flex items-center justify-center gap-2">
        <i class="fa-solid fa-cloud-arrow-down animate-bounce"></i>
        <span>Successfully loaded shared cloud transcript. You are free to edit, format, or sync it again.</span>
        <button id="closeBannerBtn" class="ml-2 hover:opacity-80 transition-opacity">
            <i class="fa-solid fa-times"></i>
        </button>
    </div>

    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8">
            
            <section class="lg:col-span-7 space-y-6">
                
                <div class="bg-white dark:bg-slate-800 rounded-2xl p-6 shadow-sm border border-slate-100 dark:border-slate-700/60 transition-all duration-300 hover:shadow-md">
                    <div class="flex justify-between items-center mb-4">
                        <div class="flex items-center gap-2">
                            <span class="flex h-2 w-2 rounded-full bg-indigo-50"></span>
                            <h2 class="text-sm font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400">Paste Raw Email / Conversation</h2>
                        </div>
                        <button id="clearRawBtn" class="text-xs text-rose-500 hover:text-rose-600 font-medium transition-colors">
                            <i class="fa-solid fa-eraser mr-1"></i> Clear
                        </button>
                    </div>
                    
                    <textarea id="rawInput" rows="5" class="w-full rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-900/50 p-4 text-sm focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition-all placeholder:text-slate-400 resize-y" placeholder="Paste the entire email headers, email body, chat dialogue, or notes here. Press 'Auto-Parse with AI' or use standard presets to auto-fill details below..."></textarea>
                    
                    <div class="mt-4 flex flex-wrap gap-3">
                        <button id="aiParseBtn" class="flex-1 min-w-[180px] inline-flex justify-center items-center gap-2 px-4 py-2.5 bg-gradient-to-r from-indigo-600 to-violet-600 hover:from-indigo-700 hover:to-violet-700 text-white rounded-xl text-sm font-semibold shadow-lg shadow-indigo-500/10 hover:shadow-indigo-500/20 active:scale-[0.98] transition-all">
                            <i class="fa-solid fa-wand-magic-sparkles animate-pulse"></i>
                            Auto-Parse with AI
                        </button>
                        <button id="quickRegexBtn" class="inline-flex justify-center items-center gap-2 px-4 py-2.5 bg-slate-100 dark:bg-slate-700 hover:bg-slate-200 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 rounded-xl text-sm font-semibold transition-all">
                            <i class="fa-solid fa-bolt"></i>
                            Quick Parse (Local)
                        </button>
                        <button id="demoPresetBtn" class="inline-flex justify-center items-center gap-2 px-4 py-2.5 border border-slate-200 dark:border-slate-700 hover:bg-slate-50 dark:hover:bg-slate-800 text-slate-600 dark:text-slate-300 rounded-xl text-sm font-semibold transition-all">
                            <i class="fa-solid fa-vial"></i>
                            Load Demo Email
                        </button>
                    </div>
                </div>

                <div class="bg-white dark:bg-slate-800 rounded-2xl p-6 shadow-sm border border-slate-100 dark:border-slate-700/60">
                    <div class="flex justify-between items-center mb-6 pb-4 border-b border-slate-100 dark:border-slate-700">
                        <h3 class="font-bold text-slate-900 dark:text-white flex items-center gap-2">
                            <i class="fa-solid fa-pen-to-square text-indigo-500"></i>
                            Transcript Form Fields
                        </h3>
                        <button id="resetFormBtn" class="text-xs text-amber-500 hover:text-amber-600 font-medium transition-colors">
                            <i class="fa-solid fa-rotate-left mr-1"></i> Reset Fields
                        </button>
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div class="md:col-span-2">
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Subject</label>
                            <input type="text" id="fieldSubject" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Customer Name</label>
                            <input type="text" id="fieldName" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Account Name</label>
                            <input type="text" id="fieldAccount" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Case Number / ID</label>
                            <input type="text" id="fieldCase" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all">
                        </div>
                        <div>
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Phone</label>
                            <input type="text" id="fieldPhone" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all">
                        </div>
                        <div class="md:col-span-2">
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Email Address</label>
                            <input type="email" id="fieldEmail" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all">
                        </div>
                        <div class="md:col-span-2">
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Issue / Problem Description</label>
                            <textarea id="fieldIssue" rows="3" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all"></textarea>
                        </div>
                        <div class="md:col-span-2 space-y-2">
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400">Steps Taken to Resolve (Separated Line-by-Line)</label>
                            <div id="stepsContainer" class="space-y-2"></div>
                            <div class="flex gap-2 pt-1">
                                <input type="text" id="newStepInput" class="flex-1 rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 px-3 py-2 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all" placeholder="Add another resolution step...">
                                <button type="button" id="addStepBtn" class="px-4 py-2 bg-indigo-50 text-indigo-600 dark:bg-indigo-950/40 dark:text-indigo-400 hover:bg-indigo-100 dark:hover:bg-indigo-950/60 rounded-lg text-sm font-semibold transition-all">
                                    <i class="fa-solid fa-plus mr-1"></i> Add
                                </button>
                            </div>
                        </div>
                        <div class="md:col-span-2">
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Resolution Status / Details</label>
                            <textarea id="fieldResolution" rows="3" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all"></textarea>
                        </div>
                        <div class="md:col-span-2">
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Next Steps (if applicable)</label>
                            <input type="text" id="fieldNextSteps" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all">
                        </div>
                        <div class="md:col-span-2">
                            <label class="block text-xs font-semibold text-slate-500 dark:text-slate-400 mb-1">Disclaimer</label>
                            <textarea id="fieldDisclaimer" rows="2" class="form-input w-full rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 p-2.5 text-sm focus:ring-2 focus:ring-indigo-500/20 focus:border-indigo-500 transition-all"></textarea>
                        </div>
                    </div>
                </div>

            </section>
            
            <section class="lg:col-span-5 space-y-6 lg:sticky lg:top-24 h-fit">
                
                <div class="bg-white dark:bg-slate-800 rounded-2xl p-6 shadow-sm border border-slate-100 dark:border-slate-700/60 text-center space-y-3">
                    <h3 class="font-bold text-slate-900 dark:text-white flex items-center justify-center gap-2">
                        <i class="fa-solid fa-chart-pie text-indigo-500"></i> Data Completeness
                    </h3>
                    <div class="chart-container">
                        <canvas id="completenessChart"></canvas>
                    </div>
                    <p class="text-xs text-slate-500 dark:text-slate-400" id="completenessText">0/11 Fields Filled</p>
                </div>

                <div class="bg-white dark:bg-slate-800 rounded-2xl shadow-md border border-slate-100 dark:border-slate-700 overflow-hidden">
                    <div class="px-6 py-4 bg-slate-50 dark:bg-slate-900/50 border-b border-slate-100 dark:border-slate-700 flex items-center justify-between">
                        <div class="flex items-center gap-2">
                            <i class="fa-solid fa-eye text-emerald-500"></i>
                            <h3 class="font-bold text-slate-950 dark:text-white">Formatted Output Preview</h3>
                        </div>
                        <span class="text-[10px] px-2 py-0.5 rounded-full bg-emerald-50 dark:bg-emerald-950 text-emerald-600 dark:text-emerald-400 font-bold uppercase tracking-wider">Live</span>
                    </div>

                    <div class="px-6 py-3 border-b border-slate-100 dark:border-slate-700/60 bg-white dark:bg-slate-800 flex flex-wrap items-center justify-between gap-3 text-xs">
                        <span class="text-slate-500 font-medium">Steps Style:</span>
                        <div class="flex bg-slate-100 dark:bg-slate-900 p-1 rounded-lg gap-1">
                            <button id="styleBullet" class="style-toggle px-2 py-1 rounded bg-white dark:bg-slate-800 shadow-sm font-semibold text-indigo-600 dark:text-indigo-400 transition-all" data-style="bullet">
                                <i class="fa-solid fa-list mr-1"></i> Bullets
                            </button>
                            <button id="styleNumber" class="style-toggle px-2 py-1 rounded text-slate-500 dark:text-slate-400 hover:text-slate-700 dark:hover:text-slate-300 font-semibold transition-all" data-style="number">
                                <i class="fa-solid fa-list-ol mr-1"></i> Numbers
                            </button>
                            <button id="styleLine" class="style-toggle px-2 py-1 rounded text-slate-500 dark:text-slate-400 hover:text-slate-700 dark:hover:text-slate-300 font-semibold transition-all" data-style="line">
                                <i class="fa-solid fa-minus mr-1"></i> Lines
                            </button>
                        </div>
                    </div>

                    <div class="p-6 bg-slate-950 relative group">
                        <div class="absolute top-4 right-4 z-10 flex gap-2">
                            <button id="copyBtn" class="flex items-center gap-1.5 px-3.5 py-1.5 bg-indigo-600 hover:bg-indigo-700 text-white rounded-lg text-xs font-bold shadow-md shadow-indigo-600/20 active:scale-[0.97] transition-all">
                                <i class="fa-solid fa-copy"></i>
                                <span>Copy Transcript</span>
                            </button>
                        </div>
                        <div class="overflow-x-auto">
                            <pre id="transcriptOutput" class="code-font text-xs text-slate-300 leading-relaxed whitespace-pre-wrap select-all focus:outline-none max-h-[300px] overflow-y-auto pr-2" tabindex="0"></pre>
                        </div>
                    </div>

                    <div class="p-4 bg-slate-50 dark:bg-slate-900/50 border-t border-slate-100 dark:border-slate-700 flex flex-wrap gap-2 justify-between">
                        <button id="cloudShareBtn" class="flex-1 min-w-[140px] flex justify-center items-center gap-2 px-4 py-3 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-semibold transition-all shadow-md shadow-indigo-500/10 active:scale-[0.98]">
                            <i class="fa-solid fa-cloud-arrow-up"></i>
                            <span>Share to Cloud</span>
                        </button>
                        <button id="copyAltBtn" class="flex-1 min-w-[140px] flex justify-center items-center gap-2 px-4 py-3 bg-indigo-50 dark:bg-indigo-950/40 text-indigo-600 dark:text-indigo-400 hover:bg-indigo-100 dark:hover:bg-indigo-950/60 rounded-xl text-sm font-semibold transition-all">
                            <i class="fa-regular fa-clipboard"></i>
                            Copy Raw Text
                        </button>
                        <button id="downloadTextBtn" class="px-4 py-3 border border-slate-200 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-800 text-slate-700 dark:text-slate-300 rounded-xl text-sm font-semibold transition-all" title="Download TXT">
                            <i class="fa-solid fa-file-arrow-down"></i>
                        </button>
                    </div>
                </div>

                <div id="recentCloudPanel" class="hidden bg-white dark:bg-slate-800 rounded-2xl p-6 shadow-sm border border-slate-100 dark:border-slate-700/60 space-y-4">
                    <div class="flex items-center justify-between pb-2 border-b border-slate-100 dark:border-slate-700">
                        <h4 class="font-bold text-sm text-slate-900 dark:text-white flex items-center gap-2">
                            <i class="fa-solid fa-globe text-indigo-500"></i>
                            <span>Recent Cloud Transcripts</span>
                        </h4>
                        <button id="refreshCloudListBtn" class="text-xs text-indigo-600 dark:text-indigo-400 font-semibold hover:underline">
                            <i class="fa-solid fa-arrows-rotate mr-1"></i> Refresh
                        </button>
                    </div>
                    <div id="recentCloudList" class="space-y-2 max-h-[220px] overflow-y-auto pr-1 text-xs">
                        <div class="text-center py-4 text-slate-400">Loading public cloud transcripts...</div>
                    </div>
                </div>

                <div class="bg-white dark:bg-slate-800 rounded-2xl p-5 shadow-sm border border-slate-100 dark:border-slate-700/60 text-xs text-slate-500 space-y-3">
                    <div class="flex items-center gap-1.5 font-bold text-slate-800 dark:text-slate-200">
                        <i class="fa-solid fa-shield-halved text-slate-400"></i>
                        <span>Disclaimer Preset Controls</span>
                    </div>
                    <div class="flex gap-2">
                        <button id="saveDisclaimerBtn" class="px-3 py-1.5 bg-emerald-50 dark:bg-emerald-950/40 text-emerald-600 dark:text-emerald-400 hover:bg-emerald-100 dark:hover:bg-emerald-950/60 font-semibold rounded-lg transition-all">
                            Save current as Default
                        </button>
                        <button id="resetDisclaimerBtn" class="px-3 py-1.5 text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-700 font-semibold rounded-lg transition-all">
                            Reset to System Default
                        </button>
                    </div>
                </div>

            </section>

        </div>
    </main>

    <div id="colorModal" class="fixed inset-0 bg-slate-950/50 backdrop-blur-sm flex items-center justify-center z-50 hidden">
        <div class="bg-white dark:bg-slate-800 max-w-sm w-full mx-4 rounded-2xl p-6 shadow-xl border border-slate-100 dark:border-slate-700 space-y-4">
            <div class="flex justify-between items-center pb-2 border-b border-slate-100 dark:border-slate-700">
                <h4 class="font-bold text-lg text-slate-900 dark:text-white flex items-center gap-2">
                    <i class="fa-solid fa-palette text-indigo-500"></i>
                    <span>Background Theme</span>
                </h4>
                <button id="closeColorBtn" class="p-1 text-slate-400 hover:text-slate-600 dark:hover:text-slate-300">
                    <i class="fa-solid fa-times"></i>
                </button>
            </div>
            <div class="space-y-2">
                <span class="text-xs font-semibold text-slate-500 dark:text-slate-400">Preset Theme Colors</span>
                <div class="grid grid-cols-5 gap-3">
                    <button class="theme-select-btn h-10 rounded-xl bg-slate-100 border-2 border-transparent hover:scale-105 transition-transform" data-theme="slate" title="Slate"></button>
                    <button class="theme-select-btn h-10 rounded-xl bg-indigo-100 border-2 border-transparent hover:scale-105 transition-transform" data-theme="indigo" title="Indigo"></button>
                    <button class="theme-select-btn h-10 rounded-xl bg-emerald-100 border-2 border-transparent hover:scale-105 transition-transform" data-theme="forest" title="Forest"></button>
                    <button class="theme-select-btn h-10 rounded-xl bg-rose-100 border-2 border-transparent hover:scale-105 transition-transform" data-theme="rose" title="Rose"></button>
                    <button class="theme-select-btn h-10 rounded-xl bg-slate-950 border-2 border-transparent hover:scale-105 transition-transform" data-theme="dark" title="Deep Dark"></button>
                </div>
            </div>
            <div class="space-y-2 pt-3 border-t border-slate-100 dark:border-slate-700">
                <span class="text-xs font-semibold text-slate-500 dark:text-slate-400">Or Select Custom Color Wheel</span>
                <div class="flex items-center gap-3">
                    <input type="color" id="customColorWheel" class="w-10 h-10 rounded-lg cursor-pointer border-0 bg-transparent">
                    <input type="text" id="customColorHex" class="flex-1 rounded-lg border border-slate-200 dark:border-slate-700 bg-white dark:bg-slate-950 px-3 py-2 text-sm text-center font-mono uppercase text-slate-700 dark:text-slate-300" placeholder="#6366F1">
                </div>
            </div>
            <div class="flex justify-end pt-2">
                <button id="closeColorBtnOk" class="px-5 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold rounded-xl transition-all shadow-md shadow-indigo-600/15">Done</button>
            </div>
        </div>
    </div>

    <div id="shareLinkModal" class="fixed inset-0 bg-slate-950/50 backdrop-blur-sm flex items-center justify-center z-50 hidden">
        <div class="bg-white dark:bg-slate-800 max-w-md w-full mx-4 rounded-2xl p-6 shadow-xl border border-slate-100 dark:border-slate-700 space-y-4">
            <div class="flex justify-between items-center pb-2 border-b border-slate-100 dark:border-slate-700">
                <h4 class="font-bold text-lg text-slate-900 dark:text-white flex items-center gap-2">
                    <i class="fa-solid fa-share-nodes text-emerald-500"></i>
                    <span>Share Transcript Link</span>
                </h4>
                <button id="closeShareBtn" class="p-1 text-slate-400 hover:text-slate-600 dark:hover:text-slate-300">
                    <i class="fa-solid fa-times"></i>
                </button>
            </div>
            <p class="text-xs text-slate-500 dark:text-slate-400">Anyone with this link can view and import this exact transcript without logging in or needing special permissions.</p>
            <div class="flex gap-2">
                <input type="text" id="shareUrlInput" readonly class="flex-1 rounded-xl border border-slate-200 dark:border-slate-700 bg-slate-50 dark:bg-slate-900/50 p-3 text-xs font-mono text-slate-700 dark:text-slate-300" value="">
                <button id="copyShareUrlBtn" class="px-4 py-3 bg-emerald-600 hover:bg-emerald-700 text-white rounded-xl text-xs font-semibold transition-all">Copy</button>
            </div>
            <div class="flex justify-end pt-2">
                <button id="closeShareBtnOk" class="px-5 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold rounded-xl transition-all shadow-md shadow-indigo-600/15">Done</button>
            </div>
        </div>
    </div>

    <div id="statusModal" class="fixed inset-0 bg-slate-950/50 backdrop-blur-sm flex items-center justify-center z-50 hidden">
        <div class="bg-white dark:bg-slate-800 max-w-md w-full mx-4 rounded-2xl p-6 shadow-xl border border-slate-100 dark:border-slate-700 space-y-4">
            <div class="flex items-center justify-between">
                <h4 id="statusTitle" class="font-bold text-lg text-slate-900 dark:text-white flex items-center gap-2">
                    <i id="statusIcon" class="fa-solid fa-spinner animate-spin text-indigo-500"></i>
                    <span>Parsing Email Transcript...</span>
                </h4>
            </div>
            <p id="statusDescription" class="text-sm text-slate-500 dark:text-slate-400 leading-relaxed"></p>
            <div class="h-2 w-full bg-slate-100 dark:bg-slate-700 rounded-full overflow-hidden">
                <div id="statusProgressBar" class="h-full bg-gradient-to-r from-indigo-500 to-violet-500 rounded-full transition-all duration-300 w-1/12"></div>
            </div>
            <div class="flex justify-end pt-2">
                <button id="cancelStatusBtn" class="px-4 py-2 text-sm font-semibold bg-slate-100 dark:bg-slate-700 hover:bg-slate-200 dark:hover:bg-slate-600 rounded-xl transition-all">Cancel</button>
            </div>
        </div>
    </div>

    <div id="toastNotification" class="fixed bottom-6 right-6 z-50 bg-slate-900 text-white dark:bg-white dark:text-slate-900 px-4 py-3 rounded-xl shadow-lg border border-slate-800 dark:border-slate-200 flex items-center gap-3 transition-all duration-300 transform translate-y-12 opacity-0 pointer-events-none">
        <div class="h-6 w-6 rounded-full bg-emerald-500 text-white flex items-center justify-center text-xs">
            <i class="fa-solid fa-check"></i>
        </div>
        <p id="toastMessage" class="text-xs font-semibold">Transcript copied successfully!</p>
    </div>

    <div id="helpModal" class="fixed inset-0 bg-slate-950/50 backdrop-blur-sm flex items-center justify-center z-50 hidden">
        <div class="bg-white dark:bg-slate-800 max-w-lg w-full mx-4 rounded-2xl p-6 shadow-xl border border-slate-100 dark:border-slate-700 space-y-4">
            <div class="flex justify-between items-center pb-2 border-b border-slate-100 dark:border-slate-700">
                <h4 class="font-bold text-lg text-slate-900 dark:text-white flex items-center gap-2">
                    <i class="fa-solid fa-circle-info text-indigo-500"></i>
                    <span>How to use TranscriptPro</span>
                </h4>
                <button id="closeHelpBtn" class="p-1 text-slate-400 hover:text-slate-600 dark:hover:text-slate-300">
                    <i class="fa-solid fa-times"></i>
                </button>
            </div>
            <div class="space-y-3 text-sm text-slate-600 dark:text-slate-300 max-h-[400px] overflow-y-auto pr-2">
                <p class="font-medium text-slate-800 dark:text-white">A powerful system for formatting customer service transcripts.</p>
                <ol class="list-decimal pl-5 space-y-2.5">
                    <li>Paste raw emails into the top input window.</li>
                    <li>Click "Auto-Parse with AI" to extract details automatically.</li>
                    <li>Use "Quick Parse (Local)" for instant offline extraction.</li>
                    <li>Change the background theme using the palette icon in the header!</li>
                    <li>Use "Share to Cloud" to upload this transcript instantly and receive a shareable view link.</li>
                </ol>
            </div>
            <div class="flex justify-end pt-2">
                <button id="closeHelpBtnOk" class="px-5 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-semibold rounded-xl transition-all shadow-md shadow-indigo-600/15">Get Started</button>
            </div>
        </div>
    </div>

    <!-- Firebase App Modules Configuration -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithCustomToken, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, collection, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables provided by runtime environment
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let db = null;
        let auth = null;
        let currentUser = null;

        if (firebaseConfig) {
            try {
                const app = initializeApp(firebaseConfig);
                auth = getAuth(app);
                db = getFirestore(app);
                
                const cloudStatusBadge = document.getElementById('cloudStatusBadge');
                cloudStatusBadge.classList.remove('hidden');
                
                const initAuth = async () => {
                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }
                };

                initAuth();

                onAuthStateChanged(auth, (user) => {
                    currentUser = user;
                    if (user) {
                        cloudStatusBadge.innerHTML = `<span class="h-2 w-2 rounded-full bg-emerald-500 animate-pulse"></span><span>Cloud Sync Active</span>`;
                        cloudStatusBadge.className = "flex items-center gap-1.5 px-3 py-1 rounded-full bg-emerald-50 dark:bg-emerald-950/40 text-xs font-semibold text-emerald-600 dark:text-emerald-400";
                        
                        document.getElementById('recentCloudPanel').classList.remove('hidden');
                        checkUrlForShareId();
                        loadRecentSharedTranscripts();
                    } else {
                        cloudStatusBadge.innerHTML = `<span class="h-2 w-2 rounded-full bg-slate-400"></span><span>Disconnected</span>`;
                        cloudStatusBadge.className = "flex items-center gap-1.5 px-3 py-1 rounded-full bg-slate-100 dark:bg-slate-800 text-xs font-semibold text-slate-500 dark:text-slate-400";
                    }
                });
            } catch (err) {
                console.error("Firebase setup failed:", err);
            }
        }

        async function uploadTranscriptToCloud() {
            if (!db || !currentUser) {
                showToast("Cloud features are unavailable. Make sure database configs are active.", true);
                return;
            }

            showStatusModal("Sharing to Cloud...", "Publishing your transcript structure to our public collaborative sync environment...", 30);

            try {
                const payload = {
                    subject: fieldSubject.value.trim(),
                    customerName: fieldName.value.trim(),
                    accountName: fieldAccount.value.trim(),
                    case: fieldCase.value.trim(),
                    phone: fieldPhone.value.trim(),
                    email: fieldEmail.value.trim(),
                    issue: fieldIssue.value.trim(),
                    resolution: fieldResolution.value.trim(),
                    nextSteps: fieldNextSteps.value.trim(),
                    disclaimer: fieldDisclaimer.value.trim(),
                    activeSteps: activeSteps,
                    currentCustomTheme: currentCustomTheme,
                    customColorHexValue: customColorHexValue,
                    createdAt: Date.now()
                };

                const publicDataCollection = collection(db, 'artifacts', appId, 'public', 'data', 'transcripts');
                const docRef = await addDoc(publicDataCollection, payload);

                hideStatusModal();
                showToast("Transcript successfully shared to Cloud!");

                const shareUrl = `${window.location.origin}${window.location.pathname}?shareId=${docRef.id}`;
                
                const shareUrlInput = document.getElementById('shareUrlInput');
                shareUrlInput.value = shareUrl;
                document.getElementById('shareLinkModal').classList.remove('hidden');
                
                loadRecentSharedTranscripts();

            } catch (err) {
                console.error("Upload error details:", err);
                hideStatusModal();
                showToast("Failed to share transcript to cloud.", true);
            }
        }

        async function checkUrlForShareId() {
            const urlParams = new URLSearchParams(window.location.search);
            const shareId = urlParams.get('shareId');
            if (shareId && db) {
                showStatusModal("Loading Shared Transcript...", "Fetching secure public sync values from the cloud catalog...", 40);
                try {
                    const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'transcripts', shareId);
                    const docSnap = await getDoc(docRef);

                    if (docSnap.exists()) {
                        const result = docSnap.data();
                        
                        fieldSubject.value = result.subject || "";
                        fieldName.value = result.customerName || "";
                        fieldAccount.value = result.accountName || "";
                        fieldCase.value = result.case || "";
                        fieldPhone.value = result.phone || "";
                        fieldEmail.value = result.email || "";
                        fieldIssue.value = result.issue || "";
                        fieldResolution.value = result.resolution || "";
                        fieldNextSteps.value = result.nextSteps || "";
                        fieldDisclaimer.value = result.disclaimer || "";
                        
                        if (Array.isArray(result.activeSteps)) {
                            activeSteps = result.activeSteps;
                        }

                        if (result.currentCustomTheme) {
                            currentCustomTheme = result.currentCustomTheme;
                            localStorage.setItem('tp_custom_theme', currentCustomTheme);
                        }

                        if (result.customColorHexValue) {
                            customColorHexValue = result.customColorHexValue;
                            localStorage.setItem('tp_custom_color_hex', customColorHexValue);
                        }

                        renderSteps();
                        applyCustomBgColor();
                        updateLiveTranscript();

                        document.getElementById('sharedLoadBanner').classList.remove('hidden');
                    } else {
                        showToast("Shared transcript could not be found or has expired.", true);
                    }
                } catch (err) {
                    console.error("Fetch share error:", err);
                    showToast("Error retrieving shared cloud document.", true);
                } finally {
                    hideStatusModal();
                }
            }
        }

        async function loadRecentSharedTranscripts() {
            if (!db) return;
            const recentCloudList = document.getElementById('recentCloudList');

            try {
                const publicDataCollection = collection(db, 'artifacts', appId, 'public', 'data', 'transcripts');
                const querySnapshot = await getDocs(publicDataCollection);

                const transcripts = [];
                querySnapshot.forEach((doc) => {
                    transcripts.push({ id: doc.id, ...doc.data() });
                });

                transcripts.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));

                recentCloudList.innerHTML = "";
                
                const topTranscripts = transcripts.slice(0, 10);

                if (topTranscripts.length === 0) {
                    recentCloudList.innerHTML = `<div class="text-center py-4 text-slate-400">No transcripts shared to the public cloud yet. Be the first!</div>`;
                    return;
                }

                topTranscripts.forEach(item => {
                    const row = document.createElement('div');
                    row.className = "p-2.5 rounded-lg border border-slate-100 dark:border-slate-750 bg-slate-50 hover:bg-slate-100 dark:bg-slate-900/50 dark:hover:bg-slate-800 transition-colors flex items-center justify-between cursor-pointer group";
                    
                    const caseLabel = item.case ? `[${item.case}]` : '[No Case]';
                    const customerLabel = item.customerName || 'Anonymous';
                    const subjectLabel = item.subject ? item.subject.substring(0, 35) + '...' : 'No Subject';

                    row.innerHTML = `
                        <div class="flex-1 min-w-0 pr-2">
                            <div class="flex items-center gap-1.5 font-bold text-slate-800 dark:text-slate-200">
                                <span class="text-indigo-600 dark:text-indigo-400">${caseLabel}</span>
                                <span class="truncate">${customerLabel}</span>
                            </div>
                            <p class="text-[11px] text-slate-400 truncate mt-0.5">${subjectLabel}</p>
                        </div>
                        <i class="fa-solid fa-angle-right text-slate-300 group-hover:text-indigo-500 transition-colors"></i>
                    `;

                    row.addEventListener('click', () => {
                        const newUrl = `${window.location.origin}${window.location.pathname}?shareId=${item.id}`;
                        window.history.pushState({ path: newUrl }, '', newUrl);
                        checkUrlForShareId();
                    });

                    recentCloudList.appendChild(row);
                });

            } catch (err) {
                console.error("Error loading list:", err);
                recentCloudList.innerHTML = `<div class="text-center py-4 text-rose-500 font-semibold">Failed to fetch public transcripts.</div>`;
            }
        }

        document.getElementById('cloudShareBtn').addEventListener('click', uploadTranscriptToCloud);
        document.getElementById('refreshCloudListBtn').addEventListener('click', loadRecentSharedTranscripts);
        document.getElementById('closeBannerBtn').addEventListener('click', () => {
            document.getElementById('sharedLoadBanner').classList.add('hidden');
        });

        document.getElementById('copyShareUrlBtn').addEventListener('click', () => {
            const input = document.getElementById('shareUrlInput');
            input.select();
            try {
                document.execCommand('copy');
                showToast("Share link copied to clipboard!");
            } catch (err) {
                showToast("Failed to copy link.", true);
            }
        });

        document.getElementById('closeShareBtn').addEventListener('click', () => document.getElementById('shareLinkModal').classList.add('hidden'));
        document.getElementById('closeShareBtnOk').addEventListener('click', () => document.getElementById('shareLinkModal').classList.add('hidden'));
    </script>

    <script>
        const apiKey = "";
        let activeSteps = ["Analyzed client logs and diagnostic error values", "Cleared faulty user session on backend routing database", "Instructed the client to clear their cache and log back in"];
        let activeStepsStyle = 'bullet';
        const defaultSysDisclaimer = "This transcript and its contents contain confidential customer interactions. Authorized personnel only. Please adhere to your internal compliance standards.";
        let activeDisclaimer = localStorage.getItem('tp_disclaimer') || defaultSysDisclaimer;

        const rawInput = document.getElementById('rawInput');
        const clearRawBtn = document.getElementById('clearRawBtn');
        const aiParseBtn = document.getElementById('aiParseBtn');
        const quickRegexBtn = document.getElementById('quickRegexBtn');
        const demoPresetBtn = document.getElementById('demoPresetBtn');
        
        const fieldSubject = document.getElementById('fieldSubject');
        const fieldName = document.getElementById('fieldName');
        const fieldAccount = document.getElementById('fieldAccount');
        const fieldCase = document.getElementById('fieldCase');
        const fieldPhone = document.getElementById('fieldPhone');
        const fieldEmail = document.getElementById('fieldEmail');
        const fieldIssue = document.getElementById('fieldIssue');
        const fieldResolution = document.getElementById('fieldResolution');
        const fieldNextSteps = document.getElementById('fieldNextSteps');
        const fieldDisclaimer = document.getElementById('fieldDisclaimer');
        
        const stepsContainer = document.getElementById('stepsContainer');
        const newStepInput = document.getElementById('newStepInput');
        const addStepBtn = document.getElementById('addStepBtn');
        const resetFormBtn = document.getElementById('resetFormBtn');

        const transcriptOutput = document.getElementById('transcriptOutput');
        const copyBtn = document.getElementById('copyBtn');
        const copyAltBtn = document.getElementById('copyAltBtn');
        const downloadTextBtn = document.getElementById('downloadTextBtn');
        
        const saveDisclaimerBtn = document.getElementById('saveDisclaimerBtn');
        const resetDisclaimerBtn = document.getElementById('resetDisclaimerBtn');
        
        const styleToggles = document.querySelectorAll('.style-toggle');

        const statusModal = document.getElementById('statusModal');
        const statusTitle = document.getElementById('statusTitle');
        const statusDescription = document.getElementById('statusDescription');
        const statusProgressBar = document.getElementById('statusProgressBar');
        const cancelStatusBtn = document.getElementById('cancelStatusBtn');
        const toastNotification = document.getElementById('toastNotification');
        const toastMessage = document.getElementById('toastMessage');

        const helpBtn = document.getElementById('helpBtn');
        const helpModal = document.getElementById('helpModal');
        const closeHelpBtn = document.getElementById('closeHelpBtn');
        const closeHelpBtnOk = document.getElementById('closeHelpBtnOk');

        const themeToggle = document.getElementById('themeToggle');
        const themeIcon = document.getElementById('themeIcon');
        
        const colorSettingsBtn = document.getElementById('colorSettingsBtn');
        const colorModal = document.getElementById('colorModal');
        const closeColorBtn = document.getElementById('closeColorBtn');
        const closeColorBtnOk = document.getElementById('closeColorBtnOk');
        const themeSelectBtns = document.querySelectorAll('.theme-select-btn');
        const customColorWheel = document.getElementById('customColorWheel');
        const customColorHex = document.getElementById('customColorHex');

        let parseController = null;
        let completenessChartInstance = null;

        const customThemes = {
            slate: { light: '#f8fafc', dark: '#0f172a' },
            indigo: { light: '#eef2ff', dark: '#1e1b4b' },
            forest: { light: '#f0fdf4', dark: '#022c22' },
            rose: { light: '#fff1f2', dark: '#4c0519' },
            dark: { light: '#111827', dark: '#030712' }
        };

        let currentCustomTheme = localStorage.getItem('tp_custom_theme') || 'slate';
        let customColorHexValue = localStorage.getItem('tp_custom_color_hex') || '';

        function applyThemeMode() {
            if (localStorage.getItem('theme') === 'dark' || (!localStorage.getItem('theme') && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
                document.documentElement.classList.add('dark');
                themeIcon.className = "fa-solid fa-sun text-amber-400";
            } else {
                document.documentElement.classList.remove('dark');
                themeIcon.className = "fa-solid fa-moon text-slate-600";
            }
            applyCustomBgColor();
        }

        function applyCustomBgColor() {
            const isDark = document.documentElement.classList.contains('dark');
            let colorHex = '';

            if (currentCustomTheme === 'custom' && customColorHexValue) {
                colorHex = customColorHexValue;
            } else {
                if (!customThemes[currentCustomTheme]) {
                    currentCustomTheme = 'slate';
                }
                colorHex = customThemes[currentCustomTheme][isDark ? 'dark' : 'light'];
            }

            document.body.style.backgroundColor = colorHex;
            
            themeSelectBtns.forEach(btn => {
                if(btn.dataset.theme === currentCustomTheme) {
                    btn.classList.add('border-slate-400', 'dark:border-slate-500');
                    btn.classList.remove('border-transparent');
                } else {
                    btn.classList.remove('border-slate-400', 'dark:border-slate-500');
                    btn.classList.add('border-transparent');
                }
            });

            if (currentCustomTheme === 'custom' && customColorHexValue) {
                customColorWheel.value = customColorHexValue;
                customColorHex.value = customColorHexValue;
            }

            if(completenessChartInstance) updateChartColors();
        }

        themeToggle.addEventListener('click', () => {
            if (document.documentElement.classList.contains('dark')) {
                localStorage.setItem('theme', 'light');
            } else {
                localStorage.setItem('theme', 'dark');
            }
            applyThemeMode();
        });

        colorSettingsBtn.addEventListener('click', () => colorModal.classList.remove('hidden'));
        closeColorBtn.addEventListener('click', () => colorModal.classList.add('hidden'));
        closeColorBtnOk.addEventListener('click', () => colorModal.classList.add('hidden'));

        themeSelectBtns.forEach(btn => {
            btn.addEventListener('click', (e) => {
                currentCustomTheme = e.target.dataset.theme;
                localStorage.setItem('tp_custom_theme', currentCustomTheme);
                applyCustomBgColor();
            });
        });

        customColorWheel.addEventListener('input', (e) => {
            const val = e.target.value;
            customColorHexValue = val;
            currentCustomTheme = 'custom';
            customColorHex.value = val;
            localStorage.setItem('tp_custom_theme', 'custom');
            localStorage.setItem('tp_custom_color_hex', val);
            applyCustomBgColor();
        });

        customColorHex.addEventListener('input', (e) => {
            const val = e.target.value;
            if (/^#[0-9A-F]{6}$/i.test(val)) {
                customColorHexValue = val;
                currentCustomTheme = 'custom';
                customColorWheel.value = val;
                localStorage.setItem('tp_custom_theme', 'custom');
                localStorage.setItem('tp_custom_color_hex', val);
                applyCustomBgColor();
            }
        });

        function initChart() {
            const ctx = document.getElementById('completenessChart').getContext('2d');
            
            completenessChartInstance = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: ['Filled', 'Empty'],
                    datasets: [{
                        data: [0, 11],
                        backgroundColor: ['#6366f1', '#e2e8f0'],
                        borderWidth: 0,
                        hoverOffset: 4
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    cutout: '75%',
                    plugins: {
                        legend: { display: false },
                        tooltip: { enabled: true }
                    }
                }
            });
            updateChartColors();
        }

        function updateChartColors() {
            if(!completenessChartInstance) return;
            const isDark = document.documentElement.classList.contains('dark');
            completenessChartInstance.data.datasets[0].backgroundColor[1] = isDark ? '#334155' : '#e2e8f0';
            completenessChartInstance.update();
        }

        function calculateCompleteness() {
            const fields = [
                fieldSubject.value, fieldName.value, fieldAccount.value, fieldCase.value,
                fieldPhone.value, fieldEmail.value, fieldIssue.value, fieldResolution.value,
                fieldNextSteps.value, fieldDisclaimer.value
            ];
            
            let filledCount = 0;
            fields.forEach(f => { if(f.trim().length > 0) filledCount++; });
            
            if(activeSteps.length > 0 && activeSteps[0].trim() !== "") {
                filledCount++;
            }
            
            const totalFields = 11;
            const emptyCount = totalFields - filledCount;
            
            if(completenessChartInstance) {
                completenessChartInstance.data.datasets[0].data = [filledCount, emptyCount];
                completenessChartInstance.update();
            }
            
            document.getElementById('completenessText').textContent = `${filledCount}/${totalFields} Fields Filled`;
        }

        function initApp() {
            applyThemeMode();
            fieldDisclaimer.value = activeDisclaimer;
            loadDummyData();
            renderSteps();
            updateLiveTranscript();
            initChart();
            calculateCompleteness();
        }

        function loadDummyData() {
            fieldSubject.value = "Unable to access dashboard showing Error Code 403 - Database Timeout";
            fieldName.value = "Johnathan Mercer";
            fieldAccount.value = "Apex Financial Solutions LLC";
            fieldCase.value = "CAS-994021-X9K";
            fieldPhone.value = "+1 (555) 349-2041";
            fieldEmail.value = "jmercer@apexfinancial.com";
            fieldIssue.value = "User encounters constant HTTP 403 Forbidden screens immediately after entering credentials.";
            fieldResolution.value = "Manually cleared lingering orphaned session states in the Redis authentication pool.";
            fieldNextSteps.value = "Provide an updated database access policy briefing and monitor traffic.";
            activeSteps = ["Analyzed application logs", "Isolated an active credential race condition", "Cleared faulty Redis token state"];
        }

        function loadEmailPreset() {
            rawInput.value = `Subject: URGENT: Global Database Lockout\nFrom: Sarah Connor\nTo: support@cloudgridlabs.com\nPhone: +44 (0) 20 7946 0192\nAccount: Skyline Logistics Intl.\nDescription: We are completely locked out of the system.`;
            showToast("Demo raw email copied to paste window!");
            parseLocalRegexOnly();
        }

        clearRawBtn.addEventListener('click', () => {
            rawInput.value = "";
            showToast("Raw text area cleared.");
        });

        function renderSteps() {
            stepsContainer.innerHTML = "";
            activeSteps.forEach((step, idx) => {
                const stepRow = document.createElement('div');
                stepRow.className = "flex items-center gap-2 bg-slate-50 dark:bg-slate-900 px-3 py-2 rounded-lg border border-slate-100 dark:border-slate-800 transition-all group";
                stepRow.innerHTML = `
                    <span class="text-xs text-slate-400 font-mono select-none w-5">${idx + 1}.</span>
                    <input type="text" value="${step.replace(/"/g, '&quot;')}" class="step-edit-input flex-1 bg-transparent border-none p-0 text-sm focus:ring-0 focus:outline-none focus:border-indigo-500" data-index="${idx}">
                    <button type="button" class="delete-step-btn text-slate-400 hover:text-rose-500 opacity-50 group-hover:opacity-100 transition-all px-1" data-index="${idx}">
                        <i class="fa-solid fa-trash-can text-xs"></i>
                    </button>
                `;
                stepsContainer.appendChild(stepRow);
            });

            document.querySelectorAll('.step-edit-input').forEach(input => {
                input.addEventListener('input', (e) => {
                    activeSteps[parseInt(e.target.dataset.index)] = e.target.value;
                    updateLiveTranscript();
                });
            });

            document.querySelectorAll('.delete-step-btn').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    activeSteps.splice(parseInt(e.currentTarget.dataset.index), 1);
                    renderSteps();
                    updateLiveTranscript();
                });
            });
        }

        function addStep() {
            const val = newStepInput.value.trim();
            if (val) {
                activeSteps.push(val);
                newStepInput.value = "";
                renderSteps();
                updateLiveTranscript();
                newStepInput.focus();
            }
        }

        addStepBtn.addEventListener('click', addStep);
        newStepInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                e.preventDefault();
                addStep();
            }
        });

        resetFormBtn.addEventListener('click', () => {
            fieldSubject.value = ""; fieldName.value = ""; fieldAccount.value = "";
            fieldCase.value = ""; fieldPhone.value = ""; fieldEmail.value = "";
            fieldIssue.value = ""; fieldResolution.value = ""; fieldNextSteps.value = "";
            activeSteps = [];
            renderSteps();
            updateLiveTranscript();
            showToast("All fields cleared.");
        });

        saveDisclaimerBtn.addEventListener('click', () => {
            activeDisclaimer = fieldDisclaimer.value.trim();
            localStorage.setItem('tp_disclaimer', activeDisclaimer);
            showToast("Disclaimer preset saved locally!");
            updateLiveTranscript();
        });

        resetDisclaimerBtn.addEventListener('click', () => {
            fieldDisclaimer.value = defaultSysDisclaimer;
            activeDisclaimer = defaultSysDisclaimer;
            localStorage.setItem('tp_disclaimer', defaultSysDisclaimer);
            showToast("Disclaimer restored to factory standards.");
            updateLiveTranscript();
        });

        function compileTranscriptText() {
            let stepsOutput = "N/A";
            if (activeSteps.length > 0) {
                stepsOutput = activeSteps.map((step, idx) => {
                    if (activeStepsStyle === 'bullet') return `- ${step}`;
                    if (activeStepsStyle === 'number') return `${idx + 1}. ${step}`;
                    return step;
                }).join('\n');
            }

            return `Subject: ${fieldSubject.value.trim() || "N/A"}\nCustomer Name: ${fieldName.value.trim() || "N/A"}\nAccount Name: ${fieldAccount.value.trim() || "N/A"}\nCase: ${fieldCase.value.trim() || "N/A"}\nPhone: ${fieldPhone.value.trim() || "N/A"}\nEmail: ${fieldEmail.value.trim() || "N/A"}\nIssue: ${fieldIssue.value.trim() || "N/A"}\nSteps Taken to Resolve:\n${stepsOutput}\nResolution: ${fieldResolution.value.trim() || "N/A"}\nNext Steps (if applicable): ${fieldNextSteps.value.trim() || "N/A"}\nDisclaimer: ${fieldDisclaimer.value.trim() || "N/A"}`;
        }

        function updateLiveTranscript() {
            transcriptOutput.textContent = compileTranscriptText();
            calculateCompleteness();
        }

        const inputFields = [fieldSubject, fieldName, fieldAccount, fieldCase, fieldPhone, fieldEmail, fieldIssue, fieldResolution, fieldNextSteps, fieldDisclaimer];
        inputFields.forEach(field => field.addEventListener('input', updateLiveTranscript));

        function copyTranscript() {
            const textarea = document.createElement('textarea');
            textarea.value = compileTranscriptText();
            textarea.style.position = 'fixed';
            document.body.appendChild(textarea);
            textarea.select();
            try {
                if (document.execCommand('copy')) showToast("Transcript copied successfully!");
                else showToast("Failed to copy transcript.", true);
            } catch (err) {
                showToast("Error copying to clipboard.", true);
            }
            document.body.removeChild(textarea);
        }

        copyBtn.addEventListener('click', copyTranscript);
        copyAltBtn.addEventListener('click', copyTranscript);

        downloadTextBtn.addEventListener('click', () => {
            const blob = new Blob([compileTranscriptText()], { type: 'text/plain' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `${(fieldCase.value.trim() || "transcript").replace(/[^a-z0-9]/gi, '_').toLowerCase()}_note.txt`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showToast("Document saved as TXT!");
        });

        styleToggles.forEach(toggle => {
            toggle.addEventListener('click', (e) => {
                styleToggles.forEach(el => {
                    el.classList.remove('bg-white', 'dark:bg-slate-800', 'shadow-sm', 'text-indigo-600', 'dark:text-indigo-400');
                    el.classList.add('text-slate-500', 'dark:text-slate-400', 'hover:text-slate-700', 'dark:hover:text-slate-300');
                });
                const btn = e.currentTarget;
                btn.classList.remove('text-slate-500', 'dark:text-slate-400', 'hover:text-slate-700');
                btn.classList.add('bg-white', 'dark:bg-slate-800', 'shadow-sm', 'text-indigo-600', 'dark:text-indigo-400');
                activeStepsStyle = btn.dataset.style;
                updateLiveTranscript();
            });
        });

        function parseLocalRegexOnly() {
            const text = rawInput.value.trim();
            if (!text) return showToast("Please paste some content into the raw area first.", true);

            const subjectMatch = text.match(/Subject:\s*(.*)/i);
            const nameMatch = text.match(/(?:From|Customer Name|Name|Client):\s*([^\n<]*)/i);
            const accountMatch = text.match(/(?:Account|Account Name|Company):\s*(.*)/i);
            const caseMatch = text.match(/(?:Case|Case ID|Case Number|Reference|Ticket):\s*(.*)/i);
            const phoneMatch = text.match(/(?:Phone|Contact|Tel):\s*(.*)/i);
            const emailMatch = text.match(/(?:Email|Email Address|Mail):\s*([a-zA-Z0-9._-]+@[a-zA-Z0-9._-]+\.[a-zA-Z0-9_-]+)/i);

            if (subjectMatch) fieldSubject.value = subjectMatch[1].trim();
            if (nameMatch) fieldName.value = nameMatch[1].trim();
            if (accountMatch) fieldAccount.value = accountMatch[1].trim();
            if (caseMatch) fieldCase.value = caseMatch[1].trim();
            if (phoneMatch) fieldPhone.value = phoneMatch[1].trim();
            if (emailMatch) fieldEmail.value = emailMatch[1].trim();

            const lines = text.split('\n');
            const filteredLines = lines.filter(line => !line.toLowerCase().match(/^(subject|from|to|phone|account):/));
            
            if (filteredLines.length > 0) {
                const remainingText = filteredLines.join('\n').trim();
                if (remainingText.length > 10) fieldIssue.value = remainingText;
            }

            renderSteps();
            updateLiveTranscript();
            showToast("Quick extracted details matching common patterns.");
        }

        quickRegexBtn.addEventListener('click', parseLocalRegexOnly);
        demoPresetBtn.addEventListener('click', loadEmailPreset);

        async function parseWithAI() {
            const textToParse = rawInput.value.trim();
            if (!textToParse) return showToast("Please paste some raw text to parse first.", true);

            showStatusModal("Contacting AI Server...", "Establishing connection to Gemini and preparing transcript parsing schemas...", 20);
            parseController = new AbortController();

            const endpointUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
            const payload = {
                contents: [{ parts: [{ text: `Extract values from this communication:\n\n${textToParse}` }] }],
                systemInstruction: { parts: [{ text: `Extract values accurately into the structured format required. Separate actual investigative steps taken into sequential line-by-line arrays. If empty, return "Not Provided".` }] },
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: {
                        type: "OBJECT",
                        properties: {
                            subject: { type: "STRING" }, customerName: { type: "STRING" }, accountName: { type: "STRING" },
                            case: { type: "STRING" }, phone: { type: "STRING" }, email: { type: "STRING" },
                            issue: { type: "STRING" }, stepsTaken: { type: "ARRAY", items: { type: "STRING" } },
                            resolution: { type: "STRING" }, nextSteps: { type: "STRING" }
                        },
                        required: ["subject", "customerName", "accountName", "case", "phone", "email", "issue", "stepsTaken", "resolution", "nextSteps"]
                    }
                }
            };

            let progressInterval = setInterval(() => {
                let currentW = parseFloat(statusProgressBar.style.width);
                if (currentW < 85) statusProgressBar.style.width = `${currentW + 5}%`;
            }, 300);

            try {
                const response = await fetchWithBackoff(endpointUrl, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload), signal: parseController.signal });
                clearInterval(progressInterval);
                statusProgressBar.style.width = "100%";
                
                const textOutput = response.candidates?.[0]?.content?.parts?.[0]?.text;
                if (!textOutput) throw new Error("No payload");

                const result = JSON.parse(textOutput);
                fieldSubject.value = result.subject || ""; fieldName.value = result.customerName || "";
                fieldAccount.value = result.accountName || ""; fieldCase.value = result.case || "";
                fieldPhone.value = result.phone || ""; fieldEmail.value = result.email || "";
                fieldIssue.value = result.issue || ""; fieldResolution.value = result.resolution || "";
                fieldNextSteps.value = result.nextSteps || "";
                
                activeSteps = Array.isArray(result.stepsTaken) && result.stepsTaken.length > 0 ? result.stepsTaken : ["Investigated user incident log timeline"];

                renderSteps();
                updateLiveTranscript();
                hideStatusModal();
                showToast("AI Auto-Parsing complete!");
            } catch (err) {
                clearInterval(progressInterval);
                hideStatusModal();
                if (err.name === 'AbortError') showToast("AI request was canceled by user.");
                else {
                    showToast("AI connection error. Falling back to Quick Local Parser.", true);
                    parseLocalRegexOnly();
                }
            }
        }

        async function fetchWithBackoff(url, options, retries = 5, delay = 1000) {
            try {
                const response = await fetch(url, options);
                if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                return await response.json();
            } catch (error) {
                if (retries > 0 && error.name !== 'AbortError') {
                    await new Promise(resolve => setTimeout(resolve, delay));
                    return fetchWithBackoff(url, options, retries - 1, delay * 2);
                }
                throw error;
            }
        }

        function showStatusModal(title, desc, progressInit) {
            statusTitle.querySelector('span').textContent = title;
            statusDescription.textContent = desc;
            statusProgressBar.style.width = `${progressInit}%`;
            statusModal.classList.remove('hidden');
        }

        function hideStatusModal() { statusModal.classList.add('hidden'); }
        cancelStatusBtn.addEventListener('click', () => { if (parseController) parseController.abort(); hideStatusModal(); });

        let toastTimeout = null;
        function showToast(msg, isError = false) {
            if (toastTimeout) clearTimeout(toastTimeout);
            toastMessage.textContent = msg;
            const checkIcon = toastNotification.querySelector('.h-6');
            if (isError) {
                checkIcon.className = "h-6 w-6 rounded-full bg-rose-500 text-white flex items-center justify-center text-xs";
                checkIcon.innerHTML = `<i class="fa-solid fa-triangle-exclamation"></i>`;
            } else {
                checkIcon.className = "h-6 w-6 rounded-full bg-emerald-500 text-white flex items-center justify-center text-xs";
                checkIcon.innerHTML = `<i class="fa-solid fa-check"></i>`;
            }
            toastNotification.classList.remove('translate-y-12', 'opacity-0', 'pointer-events-none');
            toastNotification.classList.add('translate-y-0', 'opacity-100');
            toastTimeout = setTimeout(() => {
                toastNotification.classList.remove('translate-y-0', 'opacity-100');
                toastNotification.classList.add('translate-y-12', 'opacity-0', 'pointer-events-none');
            }, 3000);
        }

        aiParseBtn.addEventListener('click', parseWithAI);
        helpBtn.addEventListener('click', () => helpModal.classList.remove('hidden'));
        closeHelpBtn.addEventListener('click', () => helpModal.classList.add('hidden'));
        closeHelpBtnOk.addEventListener('click', () => helpModal.classList.add('hidden'));
        helpModal.addEventListener('click', (e) => { if (e.target === helpModal) helpModal.classList.add('hidden'); });

        window.addEventListener('DOMContentLoaded', initApp);
    </script>
</body>
</html>
