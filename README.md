<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tokyo & Seoul Trip Vibe</title>
  
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            'vibe-bg': '#fcfaf8',
            'vibe-text': '#44403c',
            'vibe-rose': { 100: '#ffe4e6', 500: '#fda4af', 700: '#be123c' },
            'vibe-blue': { 100: '#e0f2fe', 500: '#7dd3fc', 700: '#0369a1' }
          },
          fontFamily: { sans: ['Inter', 'system-ui', 'sans-serif'] }
        }
      }
    }
  </script>
  <style>
    body { -webkit-font-smoothing: antialiased; }
    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    .animate-fadeIn { animation: fadeIn 0.5s ease-out forwards; }
    @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
  </style>
</head>
<body class="bg-vibe-bg text-vibe-text h-screen overflow-hidden">
  
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect } = React;

    // --- 簡單的圖示組件 (SVG) ---
    const Icon = ({ name, size = 20, className = "" }) => {
      const paths = {
        plane: "M2 12h20 M13 2l9 10-9 10m-6-10 6-10-6-10", 
        // 簡化的飛機圖標替代方案，為了縮減代碼使用通用路徑
        home: "M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z",
        mapPin: "M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z M12 10a3 3 0 1 0 0-6 3 3 0 0 0 0 6z",
        calendar: "M19 4H5a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6a2 2 0 0 0-2-2z M16 2v4 M8 2v4 M3 10h18",
        shoppingBag: "M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z M3 6h18 M16 10a4 4 0 0 1-8 0",
        checkSquare: "M9 11l3 3L22 4 M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11",
        info: "M12 22a10 10 0 1 0 0-20 10 10 0 0 0 0 20z M12 16v-4 M12 8h.01",
        coffee: "M18 8h1a4 4 0 0 1 0 8h-1 M2 8h16v9a4 4 0 0 1-4 4H6a4 4 0 0 1-4-4V8z M6 1v3 M10 1v3 M14 1v3",
        utensils: "M3 2v7c0 1.1.9 2 2 2h4a2 2 0 0 0 2-2V2 M7 2v20 M21 15V2v0a5 5 0 0 0-5 5v6c0 1.1.9 2 2 2h3zm0 0v7",
        train: "M19 17h2l-2-5h-3l-2 5h5z M4 17h2l2-5H5l-1 5z M8 17h8 M2 12h20 M4 17v4h2v-4 M18 17v4h2v-4 M6 8h12"
      };
      
      // 使用通用 SVG path 渲染
      const d = paths[name] || paths.info;
      
      return (
        <svg 
          xmlns="http://www.w3.org/2000/svg" 
          width={size} height={size} viewBox="0 0 24 24" 
          fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" 
          className={className}
        >
          {name === 'plane' ? <path d="M22 2L11 13M22 2l-7 20-4-9-9-4 20-7z"/> : 
           name === 'train' ? <rect x="2" y="5" width="20" height="14" rx="2" ry="2"/> :
           <path d={d} />
          }
          {/* 特殊處理火車內部的線條 */}
          {name === 'train' && <path d="M2 10h20M7 15h.01M17 15h.01" />}
        </svg>
      );
    };

    // --- 資料層 ---
    const flightsData = [
      { id: 1, date: '12/20 (六)', type: '去程', airline: '虎航 IT200', dep: '桃園 T1 06:10', arr: '成田 T2 10:35', duration: '3h 25m', color: 'rose' },
      { id: 2, date: '12/24 (三)', type: '中程', airline: '德威 TW244', dep: '成田 T2 14:35', arr: '仁川 T1 17:45', duration: '3h 10m', color: 'blue' },
      { id: 3, date: '12/27 (六)', type: '回程', airline: '華航 CI163', dep: '仁川 T2 20:15', arr: '桃園 T1 22:10', duration: '2h 55m', color: 'rose' },
    ];
    
    const accommodationData = [
      { location: '日本東京 (4晚)', hotel: 'Keisei Richmond Hotel', dates: '12/20 - 12/24', note: '錦糸町' },
      { location: '韓國首爾 (3晚)', hotel: '弘大住宿', dates: '12/24 - 12/27', note: '弘大7號出口' },
    ];

    const itineraryData = [
      { day: 'Day 1', date: '12/20 (六)', theme: '抵達東京 & 池袋', location: '東京', acts: [
        { t: '06:10', c: '桃園飛成田 (IT200)', type: 'plane' },
        { t: '11:00', c: '池袋 Parco (Harbs, Onitsuka)', type: 'shoppingBag' },
        { t: '彈性', c: '太陽城 (3coins)', type: 'shoppingBag' }
      ]},
      { day: 'Day 2', date: '12/21 (日)', theme: '新宿燒肉 & 點燈', location: '東京', acts: [
        { t: '11:00', c: '午餐：六歌仙燒肉', type: 'utensils' },
        { t: '下午', c: '新宿逛街 (Beams)', type: 'shoppingBag' },
        { t: '晚上', c: '新宿/六本木 聖誕點燈', type: 'mapPin' }
      ]},
      { day: 'Day 3', date: '12/22 (一)', theme: '豪德寺貓貓', location: '東京', acts: [
        { t: '上午', c: '豪德寺參拜 (6:00-17:00)', type: 'mapPin' },
        { t: '下午', c: '世田谷散策', type: 'coffee' }
      ]},
      { day: 'Day 4', date: '12/23 (二)', theme: '潮流聖地巡禮', location: '東京', acts: [
        { t: '全日', c: '竹下通、表參道、澀谷', type: 'shoppingBag' }
      ]},
      { day: 'Day 5', date: '12/24 (三)', theme: '飛往首爾', location: '移動', acts: [
        { t: '14:35', c: '成田飛仁川 (TW244)', type: 'plane' },
        { t: '晚餐', c: '弘大烤肉/烤鰻魚', type: 'utensils' }
      ]},
      { day: 'Day 6', date: '12/25 (四)', theme: '聖水洞 & 弘大', location: '首爾', acts: [
        { t: '10:00', c: '聖水洞：海鮮麵、咖啡', type: 'coffee' },
        { t: '晚上', c: '弘大商圈逛街', type: 'shoppingBag' }
      ]},
      { day: 'Day 7', date: '12/26 (五)', theme: '明洞 & 醬蟹', location: '首爾', acts: [
        { t: '08:30', c: '早餐：Egg Drop', type: 'utensils' },
        { t: '白天', c: '明洞逛街', type: 'shoppingBag' },
        { t: '晚餐', c: '醬蟹 (IG推薦)', type: 'utensils' }
      ]},
      { day: 'Day 8', date: '12/27 (六)', theme: '最後衝刺 & 回家', location: '回程', acts: [
        { t: '上午', c: '退房，弘大最後採購', type: 'shoppingBag' },
        { t: '15:30', c: '搭 AREX 去機場', type: 'train' },
        { t: '20:15', c: '仁川飛桃園 (CI163)', type: 'plane' }
      ]}
    ];

    const lists = {
      jp: ['Hoka 毛毛蟲拖鞋', 'Hoka Bondi 8/9', '合利他命', '豪德寺貓咪', '妙義神社貓咪', '3coins行李袋', 'UGG雪靴', '嘴破藥', '巧克力餅乾', '美津濃k14'],
      kr: ['藥局痘痘藥', 'Olive Young餅乾', 'Olive Young黑嘉麗', '生髮精華', 'Verish內衣', '排便飲料', '冬粉泡麵', '免稅菸'],
      pack: ['行動電源', '充電線/轉接頭', '護照/臺幣', '網卡/保險', '行李束帶/袋', '藥品(頭痛/腸胃)', '化妝/保養品', '免洗內褲', '雨傘', '隱形眼鏡']
    };

    const budget = [
      { i: '機票來回', c: 18847 }, { i: '日本交通卡', c: 1991 }, { i: '日本住宿', c: 6628 },
      { i: '韓國住宿', c: 3253 }, { i: '韓國行李托運', c: 1471 }, { i: '網卡/保險', c: 0 }
    ];

    // --- 主要 App 組件 ---
    function App() {
      const [activeTab, setActiveTab] = useState('overview');
      const [openDay, setOpenDay] = useState('Day 1');
      const [activeList, setActiveList] = useState('shopping');

      const TabBtn = ({ id, label, icon }) => (
        <button onClick={() => setActiveTab(id)} 
          className={`flex flex-col items-center justify-center w-full py-2 ${activeTab === id ? 'text-vibe-rose-700' : 'text-gray-400'}`}>
          <div className={activeTab === id ? 'scale-110 transition' : ''}><Icon name={icon} /></div>
          <span className="text-[10px] font-bold mt-1">{label}</span>
        </button>
      );

      const CheckItem = ({ text, k }) => {
        const [checked, setChecked] = useState(localStorage.getItem(k) === 'true');
        const toggle = () => {
          const n = !checked; setChecked(n); localStorage.setItem(k, n);
        };
        return (
          <div onClick={toggle} className={`flex items-center gap-3 p-3 rounded-xl border mb-2 cursor-pointer transition ${checked ? 'bg-vibe-rose-100/50 border-vibe-rose-100' : 'bg-white border-gray-100'}`}>
            <div className={`w-5 h-5 rounded border flex items-center justify-center ${checked ? 'bg-vibe-rose-500 border-vibe-rose-500' : 'border-gray-300'}`}>
              {checked && <div className="text-white text-xs">✓</div>}
            </div>
            <span className={`text-sm ${checked ? 'text-gray-400 line-through' : 'text-vibe-text'}`}>{text}</span>
          </div>
        );
      };

      return (
        <div className="flex justify-center bg-gray-100 min-h-screen font-sans">
          <div className="w-full max-w-md bg-vibe-bg shadow-xl relative flex flex-col h-screen">
            {/* 頂部 */}
            <div className="h-1 bg-gradient-to-r from-vibe-rose-500 to-vibe-blue-500 shrink-0"></div>
            <div className="pt-6 px-4 shrink-0 pb-2">
              <h1 className="text-2xl font-black text-vibe-text">Tokyo <span className="text-gray-300">&</span> Seoul</h1>
              <p className="text-xs text-gray-500 font-medium">Dec 20 - Dec 27, 2025</p>
            </div>

            {/* 內容區 (可滾動) */}
            <main className="flex-1 overflow-y-auto p-4 pb-24 no-scrollbar">
              
              {/* 總覽頁面 */}
              {activeTab === 'overview' && <div className="animate-fadeIn space-y-4">
                <div className="space-y-3">
                  {flightsData.map(f => (
                    <div key={f.id} className="bg-white p-4 rounded-xl shadow-sm border-l-4 relative" style={{borderColor: f.color === 'rose' ? '#fda4af' : '#7dd3fc'}}>
                      <div className="flex justify-between text-xs font-bold text-gray-400 mb-2">
                        <span>{f.type} · {f.date}</span><span>{f.airline}</span>
                      </div>
                      <div className="flex justify-between items-end">
                        <div className="font-bold text-lg">{f.dep.split(' ')[2]}<div className="text-xs font-normal text-gray-500">{f.dep.split(' ')[0]}</div></div>
                        <Icon name="plane" size={16} className="text-gray-300 mb-1 mx-2" />
                        <div className="font-bold text-lg text-right">{f.arr.split(' ')[2]}<div className="text-xs font-normal text-gray-500">{f.arr.split(' ')[0]}</div></div>
                      </div>
                    </div>
                  ))}
                </div>
                <div className="space-y-3 pt-2">
                  <h2 className="font-bold text-gray-400 text-sm">ACCOMMODATION</h2>
                  {accommodationData.map((h, i) => (
                    <div key={i} className="bg-white p-4 rounded-xl shadow-sm">
                      <div className="font-bold text-vibe-rose-700 text-xs mb-1">{h.location}</div>
                      <div className="font-bold mb-1">{h.hotel}</div>
                      <div className="text-xs text-gray-500 flex gap-2"><Icon name="calendar" size={12}/> {h.dates}</div>
                    </div>
                  ))}
                </div>
              </div>}

              {/* 行程頁面 */}
              {activeTab === 'itinerary' && <div className="animate-fadeIn space-y-3">
                {itineraryData.map((day) => (
                  <div key={day.day} className="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
                    <div onClick={() => setOpenDay(openDay === day.day ? null : day.day)} className={`p-4 flex justify-between items-center transition ${openDay === day.day ? 'bg-vibe-rose-50' : ''}`}>
                      <div>
                        <div className="text-xs font-bold text-vibe-rose-500">{day.date}</div>
                        <div className="font-bold text-sm">{day.day} · {day.theme}</div>
                      </div>
                    </div>
                    {openDay === day.day && (
                      <div className="px-4 pb-4 pt-2 bg-gray-50/50 border-t border-gray-100">
                        {day.acts.map((act, i) => (
                          <div key={i} className="flex gap-3 mb-3 last:mb-0">
                            <div className="text-xs text-gray-400 w-10 text-right pt-1 shrink-0">{act.t}</div>
                            <div className="bg-white p-2 rounded-lg shadow-sm text-sm border border-gray-100 w-full flex gap-2 items-start">
                              <span className="text-vibe-rose-500 mt-0.5"><Icon name={act.type} size={14}/></span>
                              {act.c}
                            </div>
                          </div>
                        ))}
                      </div>
                    )}
                  </div>
                ))}
              </div>}

              {/* 清單頁面 */}
              {activeTab === 'lists' && <div className="animate-fadeIn">
                <div className="flex bg-gray-200 rounded-lg p-1 mb-4">
                  <button onClick={() => setActiveList('shopping')} className={`flex-1 py-1.5 rounded-md text-xs font-bold ${activeList === 'shopping' ? 'bg-white shadow text-vibe-rose-700' : 'text-gray-500'}`}>購物</button>
                  <button onClick={() => setActiveList('packing')} className={`flex-1 py-1.5 rounded-md text-xs font-bold ${activeList === 'packing' ? 'bg-white shadow text-vibe-blue-700' : 'text-gray-500'}`}>行李</button>
                </div>
                {activeList === 'shopping' ? (
                  <>
                    <h3 className="text-sm font-bold text-gray-400 mb-2 mt-4">JAPAN</h3>
                    {lists.jp.map((item, i) => <CheckItem key={i} text={item} k={`jp_${i}`} />)}
                    <h3 className="text-sm font-bold text-gray-400 mb-2 mt-6">KOREA</h3>
                    {lists.kr.map((item, i) => <CheckItem key={i} text={item} k={`kr_${i}`} />)}
                  </>
                ) : (
                   lists.pack.map((item, i) => <CheckItem key={i} text={item} k={`pk_${i}`} />)
                )}
              </div>}

              {/* 筆記頁面 */}
              {activeTab === 'notes' && <div className="animate-fadeIn">
                 <div className="bg-yellow-50 p-4 rounded-xl border border-yellow-100 mb-6">
                   <h3 className="font-bold text-yellow-800 text-sm mb-2">⚠️ 待確認</h3>
                   <ul className="text-xs text-yellow-900 list-disc pl-4 space-y-1">
                     <li>網卡 & 入境連結 (VJW/Q-Code)</li>
                     <li>東京妙義神社 (週三休)</li>
                     <li>六本木聖誕點燈</li>
                   </ul>
                 </div>
                 <h3 className="font-bold text-gray-700 mb-3">預算概估</h3>
                 <div className="bg-white p-4 rounded-xl shadow-sm">
                   {budget.map((b, i) => (
                     <div key={i} className="flex justify-between text-sm py-1 border-b border-gray-50 last:border-0">
                       <span className="text-gray-500">{b.i}</span>
                       <span>${b.c.toLocaleString()}</span>
                     </div>
                   ))}
                   <div className="flex justify-between pt-3 font-bold text-vibe-rose-700 mt-2 border-t border-gray-100">
                     <span>總計</span>
                     <span>${budget.reduce((a,b)=>a+b.c,0).toLocaleString()}</span>
                   </div>
                 </div>
              </div>}

            </main>

            {/* 底部導航 */}
            <nav className="bg-white/90 backdrop-blur border-t border-gray-100 p-2 pb-6 shrink-0 flex justify-around">
              <TabBtn id="overview" label="總覽" icon="home" />
              <TabBtn id="itinerary" label="行程" icon="calendar" />
              <TabBtn id="lists" label="清單" icon="checkSquare" />
              <TabBtn id="notes" label="筆記" icon="info" />
            </nav>
          </div>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
