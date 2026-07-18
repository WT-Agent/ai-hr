<template>
  <div class="app-container">
    <!-- 右上角常驻分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="share-icon">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享</span>
    </button>

    <header>
      <h1>{{ appTitle }}</h1>
      <p>智能 AI 体验引擎 · 企业 HR 面试套件与胜任力题库</p>
    </header>

    <!-- 活跃动态 -->
    <UserTicker />

    <!-- 核心卡片 / 结果与历史记录展示 -->
    <main class="glass-card input-group">
      <div class="card-tabs">
        <button class="tab-btn" :class="{ active: !showHistory }" @click="showHistory = false">
          面试题库生成
        </button>
        <button class="tab-btn" :class="{ active: showHistory }" @click="showHistory = true">
          历史记录 ({{ historyList.length }})
        </button>
      </div>

      <div v-if="showHistory" class="history-view">
        <div class="history-header">
          <span>本地面试套件历史</span>
          <button v-if="historyList.length > 0" class="clear-all-btn" @click="clearAllHistory">清空全部</button>
        </div>

        <div v-if="historyList.length === 0" class="empty-state">
          <p>暂无历史面试套件记录</p>
        </div>

        <div v-else class="history-grid">
          <div v-for="item in historyList" :key="item.id" class="history-card">
            <div class="h-card-header">
              <span class="h-card-style">{{ item.styleLabel }}</span>
              <span class="h-card-time">{{ item.timestamp }}</span>
            </div>
            
            <div class="h-card-body">
              <div class="h-card-nomad-title">
                <span class="h-city-tag">👔 {{ item.job }}</span>
                <span class="h-score-badge">🚀 胜任度: {{ getAverageScore(item) }}</span>
              </div>

              <!-- 迷你指标条 -->
              <div class="h-mini-metrics">
                <div class="h-mini-item">🎯 技能: {{ item.userScores.skills }}/5</div>
                <div class="h-mini-item">🧠 逻辑: {{ item.userScores.logic }}/5</div>
              </div>

              <p class="h-card-excerpt"><strong>考察概要：</strong>{{ cleanExcerpt(item.output) }}</p>
            </div>

            <div class="h-card-actions">
              <button class="h-action-btn load-btn" @click="selectHistoryItem(item)">
                加载详情
              </button>
              <button class="h-action-btn delete-btn" @click="deleteHistoryRecord(item.id)">
                删除
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <!-- 主测评输入区域 -->
        <div v-if="!result" class="divination-setup">
          <div class="selector-group">
            <label class="selector-label">输入招聘岗位与职级定位</label>
            <input 
              type="text" 
              v-model="inquiryJob" 
              class="city-text-input" 
              placeholder="例如：高级前端工程师、资深产品经理、销售总监、运营专家..."
            />
          </div>

          <div class="selector-group">
            <label class="selector-label">请滑动评估该岗位的胜任力权重</label>
            <div class="score-sliders">
              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🎯 专业技能匹配 (Skills)</span>
                  <span class="slider-value">{{ userScores.skills }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.skills" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🧠 逻辑思维与抗压 (Logic)</span>
                  <span class="slider-value">{{ userScores.logic }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.logic" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🤝 团队协作与沟通 (Teamwork)</span>
                  <span class="slider-value">{{ userScores.team }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.team" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🚀 潜能与学习力 (Potential)</span>
                  <span class="slider-value">{{ userScores.potential }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.potential" class="range-slider" />
              </div>

              <div class="slider-group-item">
                <div class="slider-header">
                  <span class="slider-title">🛡️ 稳定性与忠诚度 (Stability)</span>
                  <span class="slider-value">{{ userScores.stability }} / 5</span>
                </div>
                <input type="range" min="1" max="5" step="1" v-model.number="userScores.stability" class="range-slider" />
              </div>
            </div>
          </div>

          <div class="selector-group">
            <label class="selector-label">输入核心考察要求或业务痛点 (选填)</label>
            <textarea 
              v-model="userInput" 
              placeholder="请输入您想重点防防伪或考察的领域，例如：需要有大型微前端架构经验、能带领5人团队、抗压能力强..."
            ></textarea>
          </div>

          <div class="selector-group">
            <label class="selector-label">选择面试评估流派与视角</label>
            <select v-model="activeStyle" class="style-select">
              <option 
                v-for="style in styleOptions" 
                :key="style.value" 
                :value="style.value"
              >
                {{ style.label }}
              </option>
            </select>
          </div>

          <button 
            class="action-btn" 
            :disabled="!inquiryJob.trim() || loading" 
            @click="handleGenerate"
          >
            {{ loading ? '题库构建中...' : '生成 HR 专业面试套件' }}
          </button>
        </div>

        <!-- 测评结果展现 -->
        <div v-else class="divination-result">
          <!-- Offer Pass 印章解压交互板块 -->
          <div class="stamp-section">
            <div class="stamp-canvas">
              <svg 
                class="stamp-svg" 
                :class="{ stamping: isStamping }" 
                @click="stampOffer" 
                viewBox="0 0 160 160"
              >
                <!-- 绿金双圈录用印章 -->
                <circle cx="80" cy="80" r="70" fill="rgba(16, 185, 129, 0.08)" stroke="#10b981" stroke-width="4" stroke-dasharray="6,3" />
                <circle cx="80" cy="80" r="62" fill="none" stroke="#10b981" stroke-width="2" />
                <text x="80" y="72" font-size="20" font-weight="900" fill="#10b981" text-anchor="middle">OFFER</text>
                <text x="80" y="102" font-size="14" font-weight="bold" fill="#10b981" text-anchor="middle">PASS 录用</text>
              </svg>
              <!-- 浮空 Offer Pass 动效 -->
              <transition-group name="float-up">
                <span 
                  v-for="item in floatingItems" 
                  :key="item.id" 
                  class="floating-merit"
                  :style="{ transform: `translate(${item.x}px, ${item.y}px)` }"
                >
                  {{ item.text }}
                </span>
              </transition-group>
            </div>
            <div class="merit-counter-display">
              累计评估录用：<strong style="color: #10b981;">{{ totalOffers }}</strong>
              <p class="wood-fish-tip">点击上方印章盖戳，Offer Pass +1</p>
            </div>
          </div>

          <!-- 数据对比看板 -->
          <div v-if="aiScores" class="comparison-dashboard">
            <h3 class="dashboard-title">📊 人才胜任力对比 (HR期望 vs AI评估基准)</h3>
            <div class="comparison-grid">
              <div v-for="metric in metricsList" :key="metric.key" class="comparison-row">
                <div class="metric-info">
                  <span class="metric-label">{{ metric.icon }} {{ metric.label }}</span>
                  <span class="metric-scores-text">
                    HR期望: <strong style="color: var(--primary-color)">{{ userScores[metric.key] }}</strong> | 
                    AI基准: <strong style="color: var(--accent-color)">{{ aiScores[metric.key] }}</strong>
                  </span>
                </div>
                <div class="comparison-bars">
                  <!-- 用户评分条 -->
                  <div class="bar-container">
                    <span class="bar-label">期望</span>
                    <div class="bar-bg">
                      <div class="bar-fill user-fill" :style="{ width: userScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                  <!-- AI评分条 -->
                  <div class="bar-container">
                    <span class="bar-label">AI</span>
                    <div class="bar-bg">
                      <div class="bar-fill ai-fill" :style="{ width: aiScores[metric.key] * 20 + '%' }"></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="result-header">
            <span class="result-title">👔 {{ inquiryJob }} 面试套件</span>
            <div class="button-actions">
              <button class="icon-btn" @click="copyText">
                {{ copied ? '已复制套件' : '复制题库' }}
              </button>
              <button class="icon-btn" @click="showShareGuide = true">
                分享朋友圈
              </button>
              <button class="icon-btn" @click="resetReview">
                重新评估
              </button>
            </div>
          </div>

          <div class="ai-response-wrapper">
            <div class="output-content scroll-box" style="text-align: left;">{{ cleanResponseText(result) }}</div>
          </div>
        </div>

        <!-- 加载状态 -->
        <div v-if="loading" class="ai-loading">
          <div class="spinner"></div>
          <p>正在分析冰山胜任力模型、构建 STAR 行为追问与防套话标准...</p>
        </div>

        <!-- 异常提示 -->
        <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
          {{ errorMsg }}
        </div>
      </div>
    </main>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中输入的招聘岗位、候选人考察点以及指标设定等数据，均仅用于实时大模型面试套件生成，我们不在服务器端持久记录您的内容。</p>
          <p>为了在您的浏览器本地保留“Offer Pass 计数”和您的“面试套件卡”历史，应用会使用浏览器的本地存储（localStorage）保存相应状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 企业 HR 面试问题生成服务。使用本应用代表您同意遵守当地有关人工智能生成内容的各项管理条例。</p>
          <p>生成的内容包含胜任力矩阵、5 组行为提问与定薪建议，仅供企业 HR、面试官及招聘团队参考，请结合公司实际情况灵活调整。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content" style="max-width: 420px;">
        <h3>Contact Us</h3>
        <div class="modal-text-content">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过微信或钉钉联系我们：</p>
          <div style="display: flex; gap: 1rem; justify-content: center; margin-top: 0.5rem; margin-bottom: 0.5rem; flex-wrap: wrap;">
            <div style="text-align: center;">
              <img :src="weixinImg" alt="微信二维码" style="width: 130px; height: 130px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.1);" />
              <div style="font-size: 0.75rem; margin-top: 0.25rem; color: var(--text-secondary);">微信</div>
            </div>
            <div style="text-align: center;">
              <img :src="dingtalkImg" alt="钉钉二维码" style="width: 130px; height: 130px; border-radius: 8px; border: 1px solid rgba(255,255,255,0.1);" />
              <div style="font-size: 0.75rem; margin-top: 0.25rem; color: var(--text-secondary);">钉钉</div>
            </div>
          </div>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />

    <!-- 分享引导浮层 -->
    <div v-if="showShareGuide" class="share-guide-overlay" @click="handleShareClose">
      <div class="share-guide-arrow">↗</div>
      <div class="share-guide-content">
        <p>点击右上角菜单 <strong>•••</strong></p>
        <p>选择 <strong>「分享到朋友圈」</strong></p>
        <p class="share-guide-sub">分享这份高效率的人才面试提问工具</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

// 读取动态配置
const appTitle = ref(appConfig.title || '网腾无限AI 企业HR面试问题');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '');

const inquiryJob = ref('');
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);
const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);

const userScores = ref({
  skills: 3,
  logic: 3,
  team: 3,
  potential: 3,
  stability: 3
});

const aiScores = ref<{ skills: number; logic: number; team: number; potential: number; stability: number; } | null>(null);

const styleOptions = [
  { label: '大厂冰山胜任力模型', value: '大厂冰山流派：BEI行为面试法，深挖真实案例与防套话追问技巧。' },
  { label: '九巨擘HR圆桌面试', value: '九巨擘流派：模拟马斯克（第一性原理）、乔布斯（极简与直觉）、秦始皇（执行力与大局观）等9大巨擘化身HR提问。' },
  { label: '压力面试与危机应对', value: '压力面试流派：设置突发业务场景与刁钻提问，考察临场应变与情绪管理。' },
  { label: '文化契合与价值观', value: '文化契合流派：考察长期稳定性、团队融入度、职业操守与价值观对齐。' },
  { label: '初创企业全能多面手', value: '初创多面手流派：关注一专多能、自我驱动、极强落地能力与低管理成本。' }
];

const activeStyle = ref(styleOptions[0].value);

const metricsList = [
  { key: 'skills', label: '专业技能匹配 (Skills)', icon: '🎯' },
  { key: 'logic', label: '逻辑思维与抗压 (Logic)', icon: '🧠' },
  { key: 'team', label: '团队协作与沟通 (Teamwork)', icon: '🤝' },
  { key: 'potential', label: '潜能与学习力 (Potential)', icon: '🚀' },
  { key: 'stability', label: '稳定性与忠诚度 (Stability)', icon: '🛡️' }
] as const;

interface FloatingItem {
  id: number;
  x: number;
  y: number;
  text: string;
}

const floatingItems = ref<FloatingItem[]>([]);
const isStamping = ref(false);
const totalOffers = ref(0);
let floatId = 0;

interface HistoryItem {
  id: string;
  timestamp: string;
  job: string;
  input: string;
  styleLabel: string;
  userScores: { skills: number; logic: number; team: number; potential: number; stability: number; };
  aiScores: { skills: number; logic: number; team: number; potential: number; stability: number; } | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

// 纯前端利用 Web Audio API 动态合成沉稳企业盖章音效
const playStampSound = () => {
  try {
    const AudioContext = window.AudioContext || (window as any).webkitAudioContext;
    if (!AudioContext) return;
    const ctx = new AudioContext();
    const now = ctx.currentTime;
    
    // 沉稳印章撞击低频
    const osc1 = ctx.createOscillator();
    const gain1 = ctx.createGain();
    osc1.frequency.setValueAtTime(220, now);
    osc1.frequency.exponentialRampToValueAtTime(60, now + 0.06);
    gain1.gain.setValueAtTime(0.8, now);
    gain1.gain.exponentialRampToValueAtTime(0.001, now + 0.08);
    osc1.connect(gain1);
    gain1.connect(ctx.destination);
    osc1.start(now);
    osc1.stop(now + 0.1);

    // 机械压扣二次反馈
    const osc2 = ctx.createOscillator();
    const gain2 = ctx.createGain();
    osc2.type = 'square';
    osc2.frequency.setValueAtTime(600, now + 0.03);
    osc2.frequency.exponentialRampToValueAtTime(150, now + 0.08);
    gain2.gain.setValueAtTime(0.3, now + 0.03);
    gain2.gain.exponentialRampToValueAtTime(0.001, now + 0.09);
    osc2.connect(gain2);
    gain2.connect(ctx.destination);
    osc2.start(now + 0.03);
    osc2.stop(now + 0.11);
  } catch (e) {
    console.error('AudioContext error:', e);
  }
};

const stampOffer = () => {
  if (isStamping.value) return;
  isStamping.value = true;
  
  // 触发机械盖章发声
  playStampSound();
  
  // 累加 Offer 评估
  totalOffers.value += 1;
  localStorage.setItem('hr_total_offers', totalOffers.value.toString());
  
  // 生成漂浮文字
  const id = floatId++;
  const x = Math.floor(Math.random() * 40) - 20;
  const y = -45;
  
  floatingItems.value.push({ id, x, y, text: 'Offer Pass +1' });
  
  setTimeout(() => {
    floatingItems.value = floatingItems.value.filter(item => item.id !== id);
  }, 1000);

  setTimeout(() => {
    isStamping.value = false;
  }, 120);
};

const loadHistory = () => {
  try {
    const raw = localStorage.getItem('hr_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
    
    const rawOffers = localStorage.getItem('hr_total_offers');
    totalOffers.value = rawOffers ? parseInt(rawOffers, 10) : 0;
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('hr_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const matched = styleOptions.find(o => o.value === activeStyle.value);
  const styleLabel = matched ? matched.label : '评估流派';

  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    job: inquiryJob.value,
    input: userInput.value,
    styleLabel,
    userScores: { ...userScores.value },
    aiScores: aiScores.value ? { ...aiScores.value } : null,
    output: result.value
  };
  historyList.value.unshift(newItem);
  saveHistory();
};

const deleteHistoryRecord = (id: string) => {
  historyList.value = historyList.value.filter(item => item.id !== id);
  saveHistory();
};

const clearAllHistory = () => {
  if (confirm('确认清空所有历史面试套件记录吗？此操作不可恢复。')) {
    historyList.value = [];
    saveHistory();
  }
};

const selectHistoryItem = (item: HistoryItem) => {
  inquiryJob.value = item.job;
  userInput.value = item.input;
  userScores.value = { ...item.userScores };
  aiScores.value = item.aiScores ? { ...item.aiScores } : null;
  result.value = item.output;
  showHistory.value = false;
};

const getAverageScore = (item: HistoryItem) => {
  const s = item.aiScores || item.userScores;
  const avg = (s.skills + s.logic + s.team + s.potential + s.stability) / 5;
  return avg.toFixed(1);
};

const cleanExcerpt = (text: string) => {
  const cleaned = cleanResponseText(text);
  return cleaned.length > 80 ? cleaned.slice(0, 80) + '...' : cleaned;
};

const parseAIScores = (text: string) => {
  const match = text.match(/\[HR_SCORES\](.*?)\[\/HR_SCORES\]/);
  if (match) {
    const scoreStr = match[1].trim();
    const scores: Record<string, number> = {};
    scoreStr.split(',').forEach(item => {
      const [key, val] = item.split(':');
      if (key && val) {
        scores[key.trim()] = Math.min(5, Math.max(1, parseInt(val.trim(), 10) || 3));
      }
    });
    return {
      skills: scores.skills || 3,
      logic: scores.logic || 3,
      team: scores.team || 3,
      potential: scores.potential || 3,
      stability: scores.stability || 3
    };
  }
  return null;
};

const cleanResponseText = (text: string) => {
  return text.replace(/\[HR_SCORES\].*?\[\/HR_SCORES\]/g, '').trim();
};

onMounted(() => {
  loadHistory();
});

const handleShareClose = () => {
  showShareGuide.value = false;
  localStorage.setItem('shared_fission', 'true');
};

const isLimitReached = computed(() => {
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        taskType: 'text',
        prompt: `起卦主题：${promptTopic.value}。招聘岗位：${inquiryJob.value}。HR期望胜任力指标：专业技能 ${userScores.value.skills}分，逻辑抗压 ${userScores.value.logic}分，团队协作 ${userScores.value.team}分，潜能学习力 ${userScores.value.potential}分，稳定性 ${userScores.value.stability}分。HR补充要求：${userInput.value}。流派倾向：${activeStyle.value}`,
        style: activeStyle.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      
      const parsed = parseAIScores(data.result);
      if (parsed) {
        aiScores.value = parsed;
      } else {
        aiScores.value = { ...userScores.value };
      }

      // 自动存储历史
      addHistoryRecord();
      
      // 累加免费次数
      const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
      localStorage.setItem('free_uses', (currentUses + 1).toString());
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const resetReview = () => {
  inquiryJob.value = '';
  userInput.value = '';
  result.value = '';
  aiScores.value = null;
  errorMsg.value = '';
  userScores.value = { skills: 3, logic: 3, team: 3, potential: 3, stability: 3 };
};

const copyText = async () => {
  try {
    const cleanedText = cleanResponseText(result.value);
    await navigator.clipboard.writeText(cleanedText);
    copied.value = true;
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};
</script>

<style scoped>
.city-text-input {
  width: 100%;
  padding: 0.75rem 1rem;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: var(--text-primary);
  font-family: inherit;
  font-size: 0.9rem;
  outline: none;
  transition: all 0.3s ease;
}

.city-text-input:focus {
  border-color: var(--primary-color);
  box-shadow: 0 0 10px rgba(168, 85, 247, 0.2);
}

/* 评分滑块 */
.score-sliders {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.25rem;
  margin-bottom: 0.5rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.04);
  padding: 1.25rem;
  border-radius: 12px;
}

.slider-group-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  text-align: left;
}

.slider-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.8rem;
  color: var(--text-secondary);
}

.slider-title {
  font-weight: 500;
}

.slider-value {
  color: #10b981;
  font-weight: bold;
}

.range-slider {
  -webkit-appearance: none;
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: rgba(255, 255, 255, 0.1);
  outline: none;
  cursor: pointer;
  transition: background 0.3s;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  background: #10b981;
  cursor: pointer;
  box-shadow: 0 0 10px rgba(16, 185, 129, 0.5);
  transition: transform 0.1s ease;
}

.range-slider::-webkit-slider-thumb:hover {
  transform: scale(1.2);
}

/* Offer Pass 盖章解压区域 */
.stamp-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 1.5rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 16px;
  margin-bottom: 1.5rem;
}

.stamp-canvas {
  position: relative;
  width: 130px;
  height: 130px;
}

.stamp-svg {
  width: 100%;
  height: 100%;
  cursor: pointer;
  transition: transform 0.1s ease-in-out;
}

.stamp-svg:hover {
  filter: drop-shadow(0 0 12px rgba(16, 185, 129, 0.4));
}

.stamp-svg.stamping {
  transform: scale(0.9) translateY(4px);
}

.floating-merit {
  position: absolute;
  left: 50%;
  top: 40%;
  font-size: 0.95rem;
  font-weight: bold;
  color: #10b981;
  text-shadow: 0 0 10px rgba(16, 185, 129, 0.6);
  pointer-events: none;
  white-space: nowrap;
}

/* 浮空渐隐动画 */
.float-up-enter-active {
  animation: floatUpAnim 1s ease-out forwards;
}

@keyframes floatUpAnim {
  0% {
    transform: translate(-50%, -40px);
    opacity: 1;
  }
  100% {
    transform: translate(-50%, -100px);
    opacity: 0;
  }
}

.merit-counter-display {
  font-size: 0.95rem;
  color: var(--text-primary);
  text-align: center;
}

.merit-counter-display strong {
  font-size: 1.3rem;
  text-shadow: 0 0 15px rgba(16, 185, 129, 0.4);
}

.wood-fish-tip {
  font-size: 0.75rem;
  color: var(--text-secondary);
  margin: 0.25rem 0 0 0;
}

/* 对比看板 */
.comparison-dashboard {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 12px;
  padding: 1.25rem;
  margin-bottom: 1.5rem;
  text-align: left;
}

.dashboard-title {
  font-size: 0.95rem;
  margin-top: 0;
  margin-bottom: 1.25rem;
  color: var(--text-primary);
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
  padding-bottom: 0.5rem;
}

.comparison-grid {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.comparison-row {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.metric-info {
  display: flex;
  justify-content: space-between;
  font-size: 0.85rem;
  font-weight: 500;
}

.metric-label {
  color: var(--text-primary);
}

.metric-scores-text {
  color: var(--text-secondary);
}

.comparison-bars {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.bar-container {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.bar-label {
  font-size: 0.7rem;
  color: var(--text-secondary);
  width: 25px;
}

.bar-bg {
  flex: 1;
  height: 8px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 4px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.user-fill {
  background: linear-gradient(90deg, #059669 0%, #10b981 100%);
  box-shadow: 0 0 8px rgba(16, 185, 129, 0.4);
}

.ai-fill {
  background: linear-gradient(90deg, #06b6d4 0%, #3b82f6 100%);
  box-shadow: 0 0 8px rgba(6, 182, 212, 0.4);
}

/* 历史卡片定制：Nomad Style */
.h-card-nomad-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.h-city-tag {
  font-size: 1.05rem;
  font-weight: bold;
  color: var(--text-primary);
}

.h-score-badge {
  background: rgba(16, 185, 129, 0.15);
  color: #10b981;
  padding: 0.2rem 0.5rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: bold;
}

.h-mini-metrics {
  display: flex;
  gap: 0.75rem;
  margin-bottom: 0.5rem;
}

.h-mini-item {
  font-size: 0.75rem;
  color: var(--text-secondary);
  background: rgba(255, 255, 255, 0.04);
  padding: 0.15rem 0.4rem;
  border-radius: 4px;
  border: 1px solid rgba(255, 255, 255, 0.04);
}

/* 分享样式与浮层 */
.share-guide-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(17, 14, 36, 0.9);
  z-index: 1000;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  padding: 2rem;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
}

.share-guide-arrow {
  font-size: 3rem;
  color: var(--primary-color);
  animation: bounce 1s infinite alternate;
  margin-right: 1.5rem;
}

.share-guide-content {
  text-align: center;
  width: 100%;
  margin-top: 2rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.share-guide-content p {
  font-size: 1.2rem;
  margin: 0;
}

.share-guide-sub {
  font-size: 0.9rem;
  color: var(--text-secondary);
  margin-top: 1rem !important;
}

/* 右上角常驻分享按钮 */
.floating-share-btn {
  position: fixed;
  top: 1rem;
  right: 1rem;
  z-index: 99;
  display: flex;
  align-items: center;
  gap: 0.35rem;
  padding: 0.5rem 0.8rem;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 20px;
  color: var(--text-primary);
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  transition: all 0.2s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.floating-share-btn:hover {
  background: rgba(255, 255, 255, 0.15);
  border-color: var(--primary-color);
  box-shadow: 0 4px 16px rgba(168, 85, 247, 0.2);
}

.share-icon {
  width: 14px;
  height: 14px;
}

/* 历史记录选项卡 */
.card-tabs {
  display: flex;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
  margin-bottom: 1.25rem;
  gap: 0.5rem;
}

.tab-btn {
  background: none;
  border: none;
  padding: 0.5rem 1rem;
  color: var(--text-secondary);
  font-size: 0.9rem;
  font-weight: bold;
  cursor: pointer;
  border-bottom: 2px solid transparent;
  transition: all 0.2s ease;
}

.tab-btn:hover {
  color: var(--text-primary);
}

.tab-btn.active {
  color: var(--primary-color);
  border-bottom-color: var(--primary-color);
}

/* 历史卡片网格 */
.history-view {
  text-align: left;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  font-size: 0.85rem;
  color: var(--text-secondary);
}

.clear-all-btn {
  background: none;
  border: none;
  color: var(--accent-color);
  font-size: 0.8rem;
  cursor: pointer;
  transition: opacity 0.2s ease;
}

.clear-all-btn:hover {
  opacity: 0.8;
}

.empty-state {
  text-align: center;
  padding: 3rem 1rem;
  color: var(--text-secondary);
  font-size: 0.9rem;
}

.history-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr));
  gap: 1rem;
}

.history-card {
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 10px;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  transition: all 0.3s ease;
}

.history-card:hover {
  background: rgba(255, 255, 255, 0.04);
  border-color: rgba(168, 85, 247, 0.2);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.h-card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.75rem;
}

.h-card-style {
  background: rgba(168, 85, 247, 0.15);
  color: var(--primary-color);
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
  font-weight: bold;
}

.h-card-time {
  color: var(--text-secondary);
}

.h-card-body {
  flex: 1;
  font-size: 0.85rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.h-card-excerpt {
  color: var(--text-primary);
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  margin: 0;
  white-space: pre-wrap;
}

.h-card-actions {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
  padding-top: 0.75rem;
}

.h-action-btn {
  background: none;
  border: none;
  font-size: 0.8rem;
  font-weight: bold;
  cursor: pointer;
  transition: color 0.2s ease;
}

.load-btn {
  color: var(--primary-color);
}

.load-btn:hover {
  color: var(--primary-hover);
}

.delete-btn {
  color: var(--accent-color);
}

.delete-btn:hover {
  color: #ef4444;
}

.scroll-box {
  max-height: 320px;
  overflow-y: auto;
  padding-right: 0.5rem;
}
</style>
