<template>
  <section class="glass-card showcase-container">
    <div class="showcase-header">
      <div class="showcase-title-group">
        <h2 class="showcase-title">面试考察维度与问题案例库 (30 精选样例)</h2>
        <p class="showcase-subtitle">体验不同岗位的人才画像与行为追问，点击“一键同款测评”即可即刻体验</p>
      </div>
      <div class="showcase-badge">人才评估 · 免费体验</div>
    </div>

    <!-- 搜索与分类筛选 -->
    <div class="showcase-filter-bar">
      <div class="category-tabs">
        <button 
          v-for="cat in categories" 
          :key="cat"
          class="category-tab"
          :class="{ active: currentCategory === cat }"
          @click="currentCategory = cat"
        >
          {{ cat }}
        </button>
      </div>
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          type="text"
          placeholder="搜索考察岗位、面试流派或核心关键词..."
          class="search-input"
        />
      </div>
    </div>

    <!-- 样例列表格 Grid -->
    <div class="sample-grid">
      <div 
        v-for="sample in paginatedSamples" 
        :key="sample.id" 
        class="sample-card"
      >
        <div class="sample-card-header">
          <span class="topic-category-tag">{{ sample.category }}</span>
          <span class="job-tag">{{ sample.job }}</span>
        </div>
        <div class="sample-original">
          <span class="sample-label">考察流派：</span><span class="style-name">{{ sample.style }}</span>
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">画像核心：</span>{{ sample.matrix }}
        </div>
        <div class="sample-rewritten">
          <span class="sample-label">核心追问：</span>“{{ sample.question }}”
        </div>
        <div class="sample-card-footer">
          <button class="use-sample-btn" @click="$emit('use-sample', sample.job)">
            一键同款测评
          </button>
        </div>
      </div>
    </div>

    <!-- 空状态提示 -->
    <div v-if="filteredSamples.length === 0" class="empty-showcase">
      未找到匹配的面试案例，请尝试切换分类或重置搜索关键词。
    </div>

    <!-- 分页组件 -->
    <div v-if="filteredSamples.length > pageSize" class="pagination-bar">
      <button 
        class="page-btn" 
        :disabled="currentPage === 1"
        @click="currentPage--"
      >
        上一页
      </button>
      <span class="page-info">第 {{ currentPage }} / {{ totalPages }} 页 (共 {{ filteredSamples.length }} 条)</span>
      <button 
        class="page-btn" 
        :disabled="currentPage === totalPages"
        @click="currentPage++"
      >
        下一页
      </button>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';

defineEmits<{
  (e: 'use-sample', text: string): void;
}>();

const categories = ['全部', '技术研发', '产品设计', '市场销售', '运营管理', '综合职能'];
const currentCategory = ref('全部');
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 6;

interface HrSample {
  id: number;
  category: string;
  job: string;
  style: string;
  matrix: string;
  question: string;
}

// 精选 30 条面试测评与提问案例
const raw30Samples: HrSample[] = [
  {
    id: 1,
    category: '技术研发',
    job: '高级前端架构师',
    style: '大厂冰山胜任力模型',
    matrix: '深度考察前端性能优化、大前端工程化、团队技术选型把控力。',
    question: '请结合过去最成功的重构案例，说明你是如何进行技术指标量化与落地推进的？'
  },
  {
    id: 2,
    category: '技术研发',
    job: '资深后端研发工程师',
    style: '九巨擘HR圆桌面试',
    matrix: '马斯克考察其第一性原理理解，爱因斯坦考察算法底座与逻辑抽象。',
    question: '如果要在服务器资源减少一半的情况下保证系统响应时间减半，你会如何做底层重构？'
  },
  {
    id: 3,
    category: '产品设计',
    job: '资深产品经理',
    style: '初创企业全能多面手',
    matrix: '考察其跨业务线协调、极低资源下的冷启动迭代、极致美学直觉。',
    question: '当你手头没有任何开发资源，而竞争对手已经上线同类功能时，你该如何做敏捷测试？'
  },
  {
    id: 4,
    category: '市场销售',
    job: 'KA大客户销售总监',
    style: '压力面试与危机应对',
    matrix: '考察面对刁钻客诉时的情绪控制、客群开拓与谈判防守能力。',
    question: '如果核心大客户在续约前一天突然宣布因价格问题转投竞品，你接下来的两小时会怎么挽回？'
  },
  {
    id: 5,
    category: '运营管理',
    job: '高级用户增长专家',
    style: '大厂冰山胜任力模型',
    matrix: '考察精细化运营、爆款裂变数据闭环、拉新留存ROI测算。',
    question: '请用STAR原则分享一次通过低成本裂变带来爆发性增长的活动细节与留存数据表现？'
  },
  {
    id: 6,
    category: '综合职能',
    job: '资深人力资源专家',
    style: '文化契合与价值观',
    matrix: '考察其危机调解能力、雇主品牌建设、核心骨干流失防范。',
    question: '当公司核心骨干因为薪酬体系矛盾集体闹离职时，HRBP应如何介入并实施防卫性保留方案？'
  },
  {
    id: 7,
    category: '技术研发',
    job: '人工智能算法专家',
    style: '大厂冰山胜任力模型',
    matrix: '考察大模型微调经验、算力成本控制、端侧落地工程实践。',
    question: '在显存受限的环境下，你是如何通过技术手段降低推理延时的？'
  },
  {
    id: 8,
    category: '产品设计',
    job: 'UIUX设计总监',
    style: '九巨擘HR圆桌面试',
    matrix: '乔布斯考察其极致简约与用户体验细节，柏拉图考察设计理念。',
    question: '如果这个界面只保留一个按钮，你会如何确保它的交互能够传递出品牌的精神内核？'
  },
  {
    id: 9,
    category: '市场销售',
    job: '出海海外市场总监',
    style: '初创企业全能多面手',
    matrix: '考察海外本土化运营、跨文化团队搭建、预算ROI控制。',
    question: '在预算极其有限且当地渠道尚不明朗时，你如何带领三人团队在三个月内打开局面？'
  },
  {
    id: 10,
    category: '运营管理',
    job: '自媒体短视频运营专家',
    style: '压力面试与危机应对',
    matrix: '考察捕捉情绪热点、热点防套路追问、版权危机公关。',
    question: '如果运营的短视频因为突发舆情遭遇大量网暴，你的第一条公关推文会如何定调？'
  },
  {
    id: 11,
    category: '综合职能',
    job: '高级财务分析总监',
    style: '文化契合与价值观',
    matrix: '考察职业操守、合规性风险管控、企业投融资分析。',
    question: '当管理层提出某项业务存在灰色合规擦边球且收益巨大时，你如何做好防卫与专业纠偏？'
  },
  {
    id: 12,
    category: '技术研发',
    job: '安全防御开发工程师',
    style: '大厂冰山胜任力模型',
    matrix: '考察零信任架构设计、应用层防渗透攻击、高危漏洞修复。',
    question: '请描述一次你经历的最具挑战的网络攻击防御战，你是如何快速定位漏洞并止损的？'
  },
  {
    id: 13,
    category: '产品设计',
    job: '高级策略产品经理',
    style: '九巨擘HR圆桌面试',
    matrix: '秦始皇考察其对海量数据大一统治理架构，马斯克考察第一性原理。',
    question: '面对混乱的推荐算法指标，你是如何通过重构底层逻辑实现核心留存率提升的？'
  },
  {
    id: 14,
    category: '市场销售',
    job: '渠道商务总监',
    style: '压力面试与危机应对',
    matrix: '考察返利政策谈判、代理商反水危机控制、渠道忠诚度。',
    question: '大区独家代理商突然宣布代理竞争对手产品，你如何保障本季度的大区销量不受重创？'
  },
  {
    id: 15,
    category: '运营管理',
    job: '私域流量运营专家',
    style: '初创企业全能多面手',
    matrix: '考察社群精细化分层、活跃度召回、高黏性成交链路。',
    question: '面对一个已经沉寂的十万用户微信社群，你如何用一套裂变模型重新激活他们？'
  },
  {
    id: 16,
    category: '综合职能',
    job: '法务合规经理',
    style: '文化契合与价值观',
    matrix: '考察合同避坑审查、劳动争议化解、知识产权保护。',
    question: '在裁员引发的群体劳动纠纷中，法务该如何协同HR制定出法律与情感平衡的谈判话术？'
  },
  {
    id: 17,
    category: '技术研发',
    job: '运维SRE专家',
    style: '压力面试与危机应对',
    matrix: '考察高并发系统故障应急响应、容器云自动化调度、降级容灾。',
    question: '当线上服务在双十一大促高峰期突然雪崩，且主备数据库均不可用时，你的第一应急步骤是什么？'
  },
  {
    id: 18,
    category: '产品设计',
    job: '硬件产品研发经理',
    style: '大厂冰山胜任力模型',
    matrix: '考察硬件开模工艺控制、BOM成本优化、敏捷供应链管理。',
    question: '面对供应链芯片缺货，你如何在最短时间内完成替代设计并在工厂顺利切线？'
  },
  {
    id: 19,
    category: '市场销售',
    job: '品牌公关专家',
    style: '压力面试与危机应对',
    matrix: '考察热点响应、舆情监测、媒体危机公关话术设计。',
    question: '如果公司高管的私人言论在微博被恶意解读并冲上热搜，你如何控制舆论对股价的影响？'
  },
  {
    id: 20,
    category: '运营管理',
    job: '电商运营总监',
    style: '九巨擘HR圆桌面试',
    matrix: '贝索斯考察其顾客至上及物流效率，秦始皇考察供应链集权把控。',
    question: '在物流运输受阻的极端大促下，你如何做到库存精准预测与用户满意度防卫？'
  },
  {
    id: 21,
    category: '综合职能',
    job: '行政综合管理经理',
    style: '初创企业全能多面手',
    matrix: '考察固定资产采购控本、多地办公协调、高情商政企公关。',
    question: '初创公司要在一周内完成五十人异地新办公室的租赁、搬迁与网络调试，你如何做敏捷排期？'
  },
  {
    id: 22,
    category: '技术研发',
    job: '区块链智能合约专家',
    style: '文化契合与价值观',
    matrix: '考察智能合约安全审计、DeFi流动性池设计、代码安全性执念。',
    question: '如何防范智能合约中的重入攻击？你在编写以太坊合约时有何安全防卫原则？'
  },
  {
    id: 23,
    category: '产品设计',
    job: '商业化产品经理',
    style: '大厂冰山胜任力模型',
    matrix: '考察广告变现逻辑、定价策略测算、买家卖家生态天平。',
    question: '如何平衡平台用户体验和商业变现广告位占比？请给出你的量化计算公式。'
  },
  {
    id: 24,
    category: '市场销售',
    job: '海外社交媒体矩阵负责人',
    style: '初创企业全能多面手',
    matrix: '考察海外短视频本土化传播、网红商务BD、红人营销ROI。',
    question: '当与海外网红签署推广合同后对方突然失联，你如何保障本轮大推的线上声量？'
  },
  {
    id: 25,
    category: '运营管理',
    job: '供应链管理专家',
    style: '九巨擘HR圆桌面试',
    matrix: '特斯拉考察其电磁线圈般高速运转供应链，乔布斯考察细节质量。',
    question: '当关键元器件海外封测厂遭遇停电危机，你如何在24小时内调动国内备选代工厂切替？'
  },
  {
    id: 26,
    category: '综合职能',
    job: '高级法务专家',
    style: '九巨擘HR圆桌面试',
    matrix: '秦始皇考察其法家防伪底线治理，柏拉图考察法学理想国构建。',
    question: '面对多国跨境电商的专利诉讼围剿，你如何利用本地司法管辖权进行战略防卫？'
  },
  {
    id: 27,
    category: '技术研发',
    job: '高级测试开发架构师',
    style: '初创企业全能多面手',
    matrix: '考察CI/CD自动化测试流水线、混沌测试工具开发、降本提效。',
    question: '如何评估开发人员提交的代码质量？你开发过哪些工具来自动拦截低质代码？'
  },
  {
    id: 28,
    category: '产品设计',
    job: '高级交互设计师',
    style: '文化契合与价值观',
    matrix: '考察以用户为中心的设计执念、无障碍设计规范、交互极简主义。',
    question: '请分享一次你为了坚持更好的交互体验，与产品及开发据理力争并取得双赢的经历？'
  },
  {
    id: 29,
    category: '市场销售',
    job: '内容营销专家',
    style: '大厂冰山胜任力模型',
    matrix: '考察内容软文引流、粉丝转化模型构建、跨界联名爆款企划。',
    question: '在没有买量预算的前提下，你如何策划一篇实现全网自传播的爆款公关稿？'
  },
  {
    id: 30,
    category: '运营管理',
    job: '仓储物流中心经理',
    style: '压力面试与危机应对',
    matrix: '考察仓库防爆发性爆仓应急响应、第三方物流谈判控本。',
    question: '大促期间仓储中心遭遇突然断电且备用发电机故障，你如何组织人工分拣保证发货时效？'
  }
];

const samples = ref<HrSample[]>(raw30Samples);

const filteredSamples = computed(() => {
  return samples.value.filter(s => {
    const matchCat = currentCategory.value === '全部' || s.category === currentCategory.value;
    const matchQuery = !searchQuery.value.trim() || 
      s.job.includes(searchQuery.value) || 
      s.style.includes(searchQuery.value) || 
      s.matrix.includes(searchQuery.value) ||
      s.question.includes(searchQuery.value);
    return matchCat && matchQuery;
  });
});

const totalPages = computed(() => Math.ceil(filteredSamples.value.length / pageSize) || 1);

const paginatedSamples = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredSamples.value.slice(start, start + pageSize);
});

watch([currentCategory, searchQuery], () => {
  currentPage.value = 1;
});
</script>
