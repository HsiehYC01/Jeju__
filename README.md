<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🇰🇷 Jeju Travel App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/Sortable.min.js"></script>
    <style>
        /* 韓系極簡與手機 App 優化樣式 */
        body {
            background-color: #F4F7F9; /* 由藍色與海洋延伸的極簡極淺灰藍底色 */
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }
        /* 隱藏滾動條但保持功能 */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        /* 行程卡片拖曳時的樣式 */
        .sortable-ghost { opacity: 0.4; background-color: #EBF3F9; }
    </style>
</head>
<body class="max-w-md mx-auto min-h-screen shadow-2xl relative bg-[#F4F7F9] overflow-x-hidden flex flex-col pb-20">

    <div id="app" v-cloak class="w-full flex-1 flex flex-col">
        <div class="relative w-full h-[250px] bg-cover bg-left overflow-hidden" style="background-image: url('🇰🇷｜Jeju 🍊'); background-color: #7BB5D8;">
            <div class="absolute inset-0 bg-gradient-to-b from-black/20 via-transparent to-black/40"></div>
            
            <div class="absolute top-8 left-5 text-white">
                <h1 class="text-2xl font-bold tracking-wider flex items-center gap-2">Jeju 🍊</h1>
                <p class="text-xs opacity-90 mt-1">9 Friends • Self-Driving Tour</p>
            </div>

            <div class="absolute bottom-4 right-4 flex items-center bg-white/90 backdrop-blur-md rounded-2xl p-1.5 shadow-lg gap-1 border border-white/40">
                <button @click="currentTab = 'itinerary'" :class="['p-2.5 rounded-xl transition-all duration-300', currentTab === 'itinerary' ? 'bg-[#2B6CB0] text-white' : 'text-slate-600 hover:bg-slate-100']">
                    <i data-lucide="map-pin" class="w-5 h-5"></i>
                </button>
                <button @click="currentTab = 'info'" :class="['p-2.5 rounded-xl transition-all duration-300', currentTab === 'info' ? 'bg-[#2B6CB0] text-white' : 'text-slate-600 hover:bg-slate-100']">
                    <i data-lucide="info" class="w-5 h-5"></i>
                </button>
                <button @click="currentTab = 'shopping'" :class="['p-2.5 rounded-xl transition-all duration-300', currentTab === 'shopping' ? 'bg-[#2B6CB0] text-white' : 'text-slate-600 hover:bg-slate-100']">
                    <i data-lucide="shopping-basket" class="w-5 h-5"></i>
                </button>
                <button @click="currentTab = 'expenses'" :class="['p-2.5 rounded-xl transition-all duration-300', currentTab === 'expenses' ? 'bg-[#2B6CB0] text-white' : 'text-slate-600 hover:bg-slate-100']">
                    <i data-lucide="wallet" class="w-5 h-5"></i>
                </button>
            </div>
        </div>

        <div class="flex-1 bg-[#F4F7F9] rounded-t-[32px] -mt-6 relative z-10 px-4 pt-6 pb-12">
            
            <div v-if="currentTab === 'itinerary'" class="space-y-4">
                <div class="flex gap-3 overflow-x-auto no-scrollbar pb-3">
                    <button v-for="day in daysList" :key="day.id" @click="selectedDay = day.id"
                        :class="['flex flex-col items-center justify-center min-w-[64px] h-[76px] rounded-2xl transition-all duration-300 shadow-sm border border-slate-100', 
                        selectedDay === day.id ? 'bg-[#2B6CB0] text-white scale-105 shadow-md' : 'bg-white text-slate-700']">
                        <span class="text-xs uppercase tracking-wider font-semibold opacity-80">{{ day.weekday }}</span>
                        <span class="text-xl font-bold mt-0.5">{{ day.date }}</span>
                    </button>
                </div>

                <div class="bg-gradient-to-r from-[#EBF3F9] to-[#DFEDF6] rounded-2xl p-4 flex items-center justify-between border border-blue-100/50 shadow-sm">
                    <div class="flex items-center gap-3">
                        <i data-lucide="cloud-sun" class="w-8 h-8 text-[#2B6CB0]"></i>
                        <div>
                            <div class="text-sm font-semibold text-slate-800">濟州島今日天氣</div>
                            <div class="text-xs text-slate-500 mt-0.5">體感溫度: 24°C • 自駕注意防曬</div>
                        </div>
                    </div>
                    <div class="text-right">
                        <div class="text-2xl font-black text-[#2B6CB0]">22°C</div>
                    </div>
                </div>

                <div ref="itineraryList" class="space-y-3 mt-2">
                    <div v-for="(item, index) in filteredItinerary" :key="item.id" :data-id="item.id">
                        
                        <div v-if="index > 0" class="flex items-center justify-center my-3 text-xs font-medium text-slate-400 gap-1.5 py-1 bg-white/50 rounded-full max-w-[160px] mx-auto border border-slate-100">
                            <i :data-lucide="item.transMode === 'car' ? 'car' : item.transMode === 'walk' ? 'footprints' : 'bus'" class="w-3.5 h-3.5 text-[#2B6CB0]"></i>
                            <span>約 {{ item.transTime || '15' }} 分鐘</span>
                        </div>

                        <div v-if="item.category === 'flight'" @click="redirectToMap(item.naverQuery)" class="bg-[#2B6CB0] text-white rounded-2xl p-4 shadow-sm border border-blue-700 active:scale-[0.99] transition-transform cursor-pointer relative overflow-hidden">
                            <div class="absolute -right-6 -bottom-6 opacity-10">
                                <i data-lucide="plane" class="w-24 h-24"></i>
                            </div>
                            <div class="flex justify-between items-center border-b border-white/20 pb-2">
                                <span class="bg-white/20 px-2.5 py-0.5 rounded-full text-[10px] font-bold uppercase tracking-wider">Flight</span>
                                <span class="text-sm font-mono font-bold">{{ item.notes }}</span>
                            </div>
                            <div class="flex items-center justify-between mt-3 px-2">
                                <div class="text-center">
                                    <div class="text-2xl font-black tracking-tight">KHH</div>
                                    <div class="text-xs opacity-80 mt-0.5">高雄小港</div>
                                </div>
                                <div class="flex flex-col items-center flex-1 mx-4">
                                    <span class="text-[10px] font-medium tracking-widest opacity-70">15:50 ► 18:50</span>
                                    <div class="w-full border-t border-dashed border-white/50 my-1 relative">
                                        <i data-lucide="plane" class="w-3.5 h-3.5 absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 bg-[#2B6CB0] px-1"></i>
                                    </div>
                                    <span class="text-[10px] opacity-80">3h 00m</span>
                                </div>
                                <div class="text-center">
                                    <div class="text-2xl font-black tracking-tight">CJU</div>
                                    <div class="text-xs opacity-80 mt-0.5">濟州國際</div>
                                </div>
                            </div>
                        </div>

                        <div v-else class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100 flex items-start gap-3 active:scale-[0.99] transition-transform relative group">
                            <div :class="['w-1 h-12 rounded-full mt-1 shrink-0', getCategoryColor(item.category)]"></div>
                            
                            <div class="flex-1" @click="redirectToMap(item.naverQuery)">
                                <div class="flex items-center justify-between">
                                    <div class="flex items-center gap-1.5">
                                        <span class="text-xs font-mono font-bold text-slate-400 bg-slate-50 px-1.5 py-0.5 rounded">{{ item.time }}</span>
                                        <h3 class="font-bold text-slate-800 text-base">{{ item.title }}</h3>
                                    </div>
                                    <span class="text-[10px] px-2 py-0.5 rounded-full font-medium" :class="getCategoryTagClass(item.category)">
                                        {{ getCategoryName(item.category) }}
                                    </span>
                                </div>
                                
                                <div class="mt-2 space-y-1 text-xs text-slate-500">
                                    <p v-if="item.hours" class="flex items-center gap-1"><i data-lucide="clock" class="w-3 h-3 text-slate-400"></i> {{ item.hours }}</p>
                                    <p v-if="item.ticket" class="flex items-center gap-1"><i data-lucide="ticket" class="w-3 h-3 text-slate-400"></i> {{ item.ticket }}</p>
                                    <p v-if="item.notes" class="flex items-center gap-1"><i data-lucide="bookmark" class="w-3 h-3 text-slate-400 text-blue-500"></i> {{ item.notes }}</p>
                                </div>
                            </div>

                            <div class="flex flex-col items-center justify-between h-14 shrink-0">
                                <button @click.stop="openEditModal(item)" class="text-slate-400 hover:text-slate-600 p-1">
                                    <i data-lucide="edit-3" class="w-4 h-4"></i>
                                </button>
                                <div class="cursor-move text-slate-300 p-1 handle">
                                    <i data-lucide="grip-vertical" class="w-4 h-4"></i>
                                </div>
                            </div>
                        </div>

                    </div>
                </div>

                <button @click="openAddModal('itinerary')" class="fixed bottom-6 right-6 w-14 h-14 bg-[#2B6CB0] text-white rounded-full flex items-center justify-center shadow-xl z-50 active:scale-95 transition-transform">
                    <i data-lucide="plus" class="w-6 h-6"></i>
                </button>
            </div>

            <div v-if="currentTab === 'info'" class="space-y-4">
                <div class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100">
                    <h3 class="font-bold text-slate-800 text-sm mb-3 flex items-center gap-1.5"><i data-lucide="refresh-cw" class="w-4 h-4 text-[#2B6CB0]"></i> 即時匯率換算 (韓元 KRW ➜ 台幣 TWD)</h3>
                    <div class="flex items-center gap-3 bg-slate-50 p-2.5 rounded-xl border border-slate-100">
                        <input type="number" v-model="krwInput" placeholder="輸入韓元" class="bg-transparent w-full font-mono font-bold text-slate-700 outline-none">
                        <span class="text-xs font-bold text-slate-400 shrink-0">₩ KRW</span>
                    </div>
                    <div class="mt-2 text-right text-xs font-medium text-slate-500">
                        約合新台幣：<span class="text-lg font-black text-[#2B6CB0] font-mono">NT$ {{ calcTwd }}</span>
                    </div>
                </div>

                <div class="space-y-3">
                    <div class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100">
                        <h3 class="font-bold text-slate-800 text-sm border-b pb-2 mb-2 flex items-center gap-2"><i data-lucide="plane-takeoff" class="text-blue-500 w-4 h-4"></i> 航班明細</h3>
                        <div class="text-xs space-y-2 text-slate-600">
                            <p class="font-semibold text-slate-700">🛫 去程：虎航 IT670</p>
                            <p>高雄機場 KHH (15:50) ➔ 濟州機場 CJU (18:50) • ⚠️ 13:00 小港機場集合</p>
                            <p class="font-semibold text-slate-700 mt-2">🛬 回程：虎航 IT675</p>
                            <p>濟州機場 CJU (10:50) ➔ 高雄機場 KHH (12:15)</p>
                        </div>
                    </div>
                    
                    <div class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100">
                        <h3 class="font-bold text-slate-800 text-sm border-b pb-2 mb-2 flex items-center gap-2"><i data-lucide="home" class="text-amber-500 w-4 h-4"></i> 住宿飯店資訊</h3>
                        <div class="space-y-3 text-xs">
                            <div class="p-2 bg-slate-50 rounded-xl relative">
                                <p class="font-bold text-slate-700">D1-D2: Hailli 濟州</p>
                                <p class="text-slate-500 mt-1">Susan 8-gil 6, 涯月邑, 濟州特別自治道</p>
                                <button @click="redirectToMap('Hailli济州')" class="mt-2 text-[10px] text-white bg-[#2B6CB0] px-2 py-1 rounded-md inline-flex items-center gap-1">導航 <i data-lucide="navigation" class="w-2.5 h-2.5"></i></button>
                            </div>
                            <div class="p-2 bg-slate-50 rounded-xl relative">
                                <p class="font-bold text-slate-700">D2-D3: Garaji 屋</p>
                                <p class="text-slate-500 mt-1">西歸浦市 安德面 沙溪里 2816-1</p>
                                <button @click="redirectToMap('Garaji屋')" class="mt-2 text-[10px] text-white bg-[#2B6CB0] px-2 py-1 rounded-md inline-flex items-center gap-1">導航 <i data-lucide="navigation" class="w-2.5 h-2.5"></i></button>
                            </div>
                        </div>
                    </div>

                    <div class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100">
                        <h3 class="font-bold text-slate-800 text-sm border-b pb-2 mb-2 flex items-center gap-2"><i data-lucide="car" class="text-emerald-500 w-4 h-4"></i> 租車資訊</h3>
                        <div class="text-xs text-slate-600 space-y-1">
                            <p><span class="font-semibold text-slate-700">租車公司：</span>SK租車接駁車 (5號出口搭乘)</p>
                            <p><span class="font-semibold text-slate-700">車型：</span>KIA EV9（六人座）* 2 台</p>
                            <p><span class="font-semibold text-slate-700">攜帶：</span>國際駕照、台灣駕照、護照</p>
                            <p class="text-rose-500 font-medium">⚠️ 叮嚀：交車記得先拍油表/外觀</p>
                            <button @click="redirectToMap('SK租车')" class="mt-2 text-[10px] text-white bg-[#2B6CB0] px-2 py-1 rounded-md inline-flex items-center gap-1">導航至租車處 <i data-lucide="navigation" class="w-2.5 h-2.5"></i></button>
                        </div>
                    </div>

                    <div class="bg-rose-50 rounded-2xl p-4 shadow-sm border border-rose-100">
                        <h3 class="font-bold text-rose-800 text-sm mb-2 flex items-center gap-2"><i data-lucide="phone-call" class="w-4 h-4"></i> 🚨 韓國緊急求助熱線</h3>
                        <div class="grid grid-cols-2 gap-2 text-xs font-bold text-rose-700">
                            <div class="bg-white p-2 rounded-xl text-center shadow-xs">犯罪報案：112</div>
                            <div class="bg-white p-2 rounded-xl text-center shadow-xs">火災救護：119</div>
                        </div>
                    </div>
                </div>
            </div>

            <div v-if="currentTab === 'shopping'" class="space-y-4">
                <div class="grid grid-cols-2 gap-3">
                    <div v-for="item in shoppingList" :key="item.id" class="bg-white rounded-2xl p-3 shadow-sm border border-slate-100 flex flex-col justify-between">
                        <div class="w-full aspect-square bg-slate-100 rounded-xl overflow-hidden mb-2 flex items-center justify-center relative">
                            <img v-if="item.image" :src="item.image" class="w-full h-full object-cover">
                            <i v-else data-lucide="image" class="w-8 h-8 text-slate-300"></i>
                        </div>
                        <div class="flex items-start justify-between gap-1">
                            <p class="text-sm font-bold text-slate-700 line-clamp-2">{{ item.name }}</p>
                            <button @click="deleteShopping(item.id)" class="text-slate-300 hover:text-rose-500"><i data-lucide="trash-2" class="w-3.5 h-3.5"></i></button>
                        </div>
                    </div>
                </div>
                
                <button @click="openAddModal('shopping')" class="fixed bottom-6 right-6 w-14 h-14 bg-[#2B6CB0] text-white rounded-full flex items-center justify-center shadow-xl z-50 active:scale-95 transition-transform">
                    <i data-lucide="plus" class="w-6 h-6"></i>
                </button>
            </div>

            <div v-if="currentTab === 'expenses'" class="space-y-4">
                <div class="bg-gradient-to-br from-[#2B6CB0] to-[#1A365D] text-white rounded-2xl p-5 shadow-lg relative overflow-hidden">
                    <span class="text-xs uppercase tracking-widest opacity-75">濟州島旅行總花費</span>
                    <h2 class="text-3xl font-black font-mono mt-1">₩ {{ totalExpenses.toLocaleString() }}</h2>
                    <p class="text-xs opacity-80 mt-2">折合新台幣約 NT$ {{ Math.round(totalExpenses * 0.024).toLocaleString() }}</p>
                </div>

                <div class="space-y-2">
                    <div v-for="exp in expensesList" :key="exp.id" class="bg-white rounded-xl p-3.5 shadow-sm border border-slate-100 flex items-center justify-between">
                        <div class="flex items-center gap-3">
                            <div class="p-2 rounded-xl bg-slate-50 text-slate-600">
                                <i :data-lucide="getExpenseIcon(exp.category)" class="w-5 h-5"></i>
                            </div>
                            <div>
                                <h4 class="text-sm font-bold text-slate-800">{{ exp.name }}</h4>
                                <p class="text-[10px] text-slate-400 mt-0.5">{{ exp.date }} • {{ exp.payMethod === 'card' ? '💳 信用卡' : '💵 現金' }}</p>
                            </div>
                        </div>
                        <div class="flex items-center gap-2">
                            <span class="font-mono font-bold text-slate-700 text-sm">₩ {{ exp.amount.toLocaleString() }}</span>
                            <button @click="deleteExpense(exp.id)" class="text-slate-300 hover:text-rose-500 p-1"><i data-lucide="trash-2" class="w-4 h-4"></i></button>
                        </div>
                    </div>
                </div>

                <button @click="openAddModal('expenses')" class="fixed bottom-6 right-6 w-14 h-14 bg-[#2B6CB0] text-white rounded-full flex items-center justify-center shadow-xl z-50 active:scale-95 transition-transform">
                    <i data-lucide="plus" class="w-6 h-6"></i>
                </button>
            </div>

        </div>

        <div v-if="modal.show" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-end justify-center">
            <div class="bg-white w-full max-w-md rounded-t-[28px] p-6 space-y-4 max-h-[85vh] overflow-y-auto animate-slide-up">
                <div class="flex justify-between items-center border-b pb-3">
                    <h3 class="text-lg font-bold text-slate-800">{{ modal.isEdit ? '編輯' : '新增' }}{{ modal.titleType }}</h3>
                    <button @click="modal.show = false" class="text-slate-400"><i data-lucide="x" class="w-5 h-5"></i></button>
                </div>

                <div v-if="modal.type === 'itinerary'" class="space-y-3 text-sm">
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">分類</label>
                        <select v-model="form.category" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                            <option value="spot">景點</option>
                            <option value="hotel">住宿</option>
                            <option value="food">美食</option>
                            <option value="shopping">購物</option>
                            <option value="flight">航班</option>
                            <option value="other">其他</option>
                        </select>
                    </div>
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="block text-xs font-bold text-slate-500 mb-1">出發時間</label>
                            <input type="text" v-model="form.time" placeholder="如 08:30" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-slate-500 mb-1">上個行程過來的交通(分)</label>
                            <input type="text" v-model="form.transTime" placeholder="如 20" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                        </div>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">行程名稱</label>
                        <input type="text" v-model="form.title" placeholder="請輸入景點或美食店名" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">營業時間 / 交通工具方式</label>
                        <input type="text" v-model="form.hours" placeholder="如 11:00-01:00 / 自駕" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">門票資訊</label>
                        <input type="text" v-model="form.ticket" placeholder="如 ₩2,000" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">Naver搜尋關鍵字 (導航用)</label>
                        <input type="text" v-model="form.naverQuery" placeholder="如 Noraba" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">特色 & 備註</label>
                        <textarea v-model="form.notes" rows="2" placeholder="備註清單" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none"></textarea>
                    </div>
                </div>

                <div v-if="modal.type === 'shopping'" class="space-y-3 text-sm">
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">商品名稱</label>
                        <input type="text" v-model="form.shoppingName" placeholder="如 橘子冷麵、大蒜麵包" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">商品圖片</label>
                        <input type="file" @change="handleImageUpload" class="w-full text-xs text-slate-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-xs file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100">
                    </div>
                </div>

                <div v-if="modal.type === 'expenses'" class="space-y-3 text-sm">
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="block text-xs font-bold text-slate-500 mb-1">類別</label>
                            <select v-model="form.expCategory" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                                <option value="transport">交通</option>
                                <option value="food">飲食</option>
                                <option value="stay">住宿</option>
                                <option value="ticket">門票</option>
                                <option value="shopping">購物</option>
                                <option value="other">其他</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-slate-500 mb-1">付款方式</label>
                            <select v-model="form.expPayMethod" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                                <option value="cash">💵 現金</option>
                                <option value="card">💳 信用卡</option>
                            </select>
                        </div>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">項目名稱</label>
                        <input type="text" v-model="form.expName" placeholder="項目名稱" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                    </div>
                    <div class="grid grid-cols-2 gap-2">
                        <div>
                            <label class="block text-xs font-bold text-slate-500 mb-1">金額 (韓元 ₩)</label>
                            <input type="number" v-model="form.expAmount" placeholder="如 30000" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                        </div>
                        <div>
                            <label class="block text-xs font-bold text-slate-500 mb-1">花費日期</label>
                            <input type="date" v-model="form.expDate" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                        </div>
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-500 mb-1">備註</label>
                        <input type="text" v-model="form.expNotes" placeholder="備註" class="w-full bg-slate-50 p-2.5 rounded-xl border border-slate-200 outline-none">
                    </div>
                </div>

                <div class="flex items-center gap-2 pt-2">
                    <button v-if="modal.isEdit" @click="confirmDelete" class="bg-rose-500 text-white font-bold p-2.5 rounded-xl transition-all text-sm shrink-0 flex items-center justify-center">
                        <i data-lucide="trash-2" class="w-5 h-5"></i>
                    </button>
                    <button @click="submitModal" class="flex-1 bg-[#2B6CB0] text-white font-bold py-2.5 rounded-xl hover:bg-blue-700 transition-all text-sm">
                        確定儲存
                    </button>
                </div>
            </div>
        </div>

    </div>

    <script>
        const { createApp, ref, computed, onMounted, nextTick, watch } = Vue;

        createApp({
            setup() {
                const currentTab = ref('itinerary');
                const selectedDay = ref('day1');
                const krwInput = ref('');
                const itineraryList = ref(null);

                // 匯率預估 (1 KRW ≈ 0.024 TWD)
                const calcTwd = computed(() => {
                    if (!krwInput.value) return '0';
                    return Math.round(krwInput.value * 0.024).toLocaleString();
                });

                // 日期導覽列預設資料
                const daysList = ref([
                    { id: 'day1', weekday: 'Mon', date: '29' },
                    { id: 'day2', weekday: 'Tue', date: '30' },
                    { id: 'day3', weekday: 'Wed', date: '1' },
                    { id: 'day4', weekday: 'Thu', date: '2' },
                    { id: 'day5', weekday: 'Fri', date: '3' },
                    { id: 'day6', weekday: 'Sat', date: '4' }
                ]);

                // 核心行程資料（已直接帶入你提供的前兩日詳細行程，以及後續的大地標關鍵字）
                const itineraryData = ref([
                    // Day 1
                    { id: 'it1', day: 'day1', category: 'flight', time: '18:50', title: '抵達濟州機場', hours: '虎航 IT670', ticket: '', notes: '往5號出口走，WowPass換錢，搭SK租車接駁車', naverQuery: '济州机场', transTime: '0', transMode: 'car' },
                    { id: 'it2', day: 'day1', category: 'other', time: '19:30', title: 'SK租車處交車', hours: '08:00-20:00', ticket: 'KIA EV9 * 2台', notes: '帶國際駕照、台灣駕照、護照。⚠️記得先拍油表紀錄！', naverQuery: 'SK租车 济州', transTime: '20', transMode: 'car' },
                    { id: 'it3', day: 'day1', category: 'food', time: '20:30', title: '晚餐：冠軍花蟹1號店 (醬蟹)', hours: '11:00-01:00', ticket: 'IG分享好評', notes: '最後點餐 00:00', naverQuery: '冠军花蟹1号店', transTime: '15', transMode: 'car' },
                    { id: 'it4', day: 'day1', category: 'shopping', time: '21:40', title: 'Olive Young 採購', hours: '營業至 23:00', ticket: '', notes: '歐莉芙洋濟州Jewon店/Yeondong店皆可', naverQuery: '欧莉芙洋济州Jewon店', transTime: '10', transMode: 'car' },
                    { id: 'it5', day: 'day1', category: 'hotel', time: '22:30', title: '入住 Hailli 濟州民宿', hours: 'Check In 16:00後', ticket: '泳池加熱免費', notes: '門口停一台車，記得詢問房東。PS5/洗烘衣機/電子衣櫥。禁菸！', naverQuery: 'Hailli济州', transTime: '16', transMode: 'car' },
                    
                    // Day 2
                    { id: 'it6', day: 'day2', category: 'other', time: '08:00', title: '出門自駕出發', hours: '', ticket: '', notes: '7:00起床吃早餐收行李', naverQuery: 'Hailli济州', transTime: '0', transMode: 'car' },
                    { id: 'it7', day: 'day2', category: 'spot', time: '08:30', title: '9.81 PARK 賽車', hours: '預約 09:00 (玩2-2.5h)', ticket: '$690/人 (已購票)', notes: '前20-30分鐘報到，可先下載App看賽車數據', naverQuery: '9.81 park', transTime: '20', transMode: 'car' },
                    { id: 'it8', day: 'day2', category: 'spot', time: '12:00', title: '協才海水浴場', hours: '停留約30分鐘', ticket: '免費', notes: '如果賽車提早結束可順路去 Seong Isidol 牧場', naverQuery: '挟才海边', transTime: '40', transMode: 'car' },
                    { id: 'it9', day: 'day2', category: 'food', time: '13:00', title: '午餐：NORABA 海鮮拉麵', hours: '10:00-14:00/15:00-16:30', ticket: '備案: Hallim Tongkeun鰻魚', notes: '生醃備案：Mipo家濟州Aewol店', naverQuery: 'Noraba', transTime: '30', transMode: 'car' },
                    { id: 'it10', day: 'day2', category: 'spot', time: '15:00', title: '涯月地網公園散步', hours: '停留30分鐘', ticket: '免費', notes: '漫步海邊步道', naverQuery: 'Aewol Handam公园', transTime: '10', transMode: 'car' },
                    { id: 'it11', day: 'day2', category: 'food', time: '15:30', title: 'Haejigae 海景咖啡廳', hours: '看夕陽看海', ticket: '可刷卡/現金', notes: '若後面要逛街，最晚
