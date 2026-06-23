<template>
  <div class="home-page vx-grid">
    <header class="topbar">
      <AppBrand />

      <div class="topbar-meta">
        <span class="meta-item">
          <span class="meta-dot" aria-hidden="true"></span>
          PUBLIC OPINION FORECAST
        </span>
        <span class="meta-divider"></span>
        <span class="meta-item meta-version">v0.1 · {{ now }}</span>
        <span class="meta-divider"></span>
        <LanguageSwitcher />
        <a href="https://github.com/whoamiARC/VoxWave" target="_blank" rel="noreferrer" class="repo-link">
          <span class="repo-dot" aria-hidden="true"></span>
          GitHub
        </a>
      </div>
    </header>

    <main class="workspace">
      <section class="hero">
        <div class="hero-copy">
          <span class="hero-eyebrow">
            <span class="hero-eyebrow-bar"></span>
            REAL-TIME OPINION SIMULATION
          </span>
          <h1>社会舆情<br/><span class="hero-accent">预测分析</span></h1>
          <p class="hero-text">
            把一段事件描述、几张背景材料丢进 VoxWave，自动抽取种子词、组织图谱节点、
            启动多智能体模拟，输出传播路径、群体情绪、观点分化、风险拐点与处置建议。
          </p>
          <div class="hero-stats">
            <div class="stat-block">
              <span class="stat-label">SELECTED SEEDS</span>
              <span class="stat-value">{{ String(selectedTags.length).padStart(2, '0') }}</span>
            </div>
            <div class="stat-block">
              <span class="stat-label">ATTACHMENTS</span>
              <span class="stat-value">{{ String(files.length).padStart(2, '0') }}</span>
            </div>
            <div class="stat-block">
              <span class="stat-label">SEED POOL</span>
              <span class="stat-value">{{ String(totalSeedCount).padStart(3, '0') }}</span>
            </div>
            <div class="stat-block stat-block-accent">
              <span class="stat-label">SIM ROUNDS</span>
              <span class="stat-value">— —</span>
            </div>
          </div>
        </div>

        <aside class="hero-panel">
          <div class="hero-panel-head">
            <span class="panel-tag">SYSTEM STATUS</span>
            <span class="panel-pulse"><span></span> LIVE</span>
          </div>
          <ul class="hero-panel-list">
            <li>
              <span class="dot dot-cyan"></span>
              <span class="key">引擎</span>
              <span class="value">{{ ready ? 'READY' : 'IDLE' }}</span>
            </li>
            <li>
              <span class="dot dot-purple"></span>
              <span class="key">种子库</span>
              <span class="value">{{ totalSeedCount }} entries</span>
            </li>
            <li>
              <span class="dot dot-magenta"></span>
              <span class="key">本地时区</span>
              <span class="value">{{ tz }}</span>
            </li>
            <li>
              <span class="dot dot-green"></span>
              <span class="key">多智能体</span>
              <span class="value">OASIS / CAMEL</span>
            </li>
            <li>
              <span class="dot dot-amber"></span>
              <span class="key">记忆图谱</span>
              <span class="value">Zep Cloud</span>
            </li>
            <li>
              <span class="dot dot-cyan"></span>
              <span class="key">LLM</span>
              <span class="value">OpenAI SDK compatible</span>
            </li>
          </ul>
          <div class="hero-panel-footer">
            <span class="footer-bar"></span>
            <span>为校园 · 职场 · 公共安全 · 消费维权 · 平台争议 · 社区治理 而设计</span>
          </div>
        </aside>
      </section>

      <section class="generator-layout" aria-label="舆情种子生成器">
        <div class="selector-panel">
          <div class="panel-heading">
            <div>
              <span class="panel-kicker">SEED ATLAS</span>
              <h2>选择事件要素</h2>
            </div>
            <button type="button" class="quiet-button" @click="clearSeeds" :disabled="selectedTags.length === 0">
              清空标签
            </button>
          </div>

          <div class="seed-toolbar">
            <label class="seed-search" for="seedSearch">
              <span>搜索种子</span>
              <input
                id="seedSearch"
                v-model="seedSearch"
                type="search"
                placeholder="输入关键词，如教育、裁员、暴雨、维权、谣言"
              />
            </label>
            <span class="seed-total">{{ visibleSeedCount }} / {{ totalSeedCount }}</span>
          </div>

          <div class="seed-groups">
            <section
              v-for="group in filteredSeedGroups"
              :key="group.key"
              class="seed-group"
              :style="{ '--group-color': group.color }"
            >
              <div class="group-title-row">
                <span class="group-dot" aria-hidden="true"></span>
                <h3>{{ group.label }}</h3>
                <span class="group-count">{{ getGroupSelectedCount(group.key) }}/{{ group.items.length }}</span>
              </div>

              <div class="chip-list">
                <button
                  v-for="item in group.items"
                  :key="item"
                  type="button"
                  class="seed-chip"
                  :class="{ active: isSelected(group.key, item) }"
                  @click="toggleSeed(group.key, item)"
                >
                  {{ item }}
                </button>
              </div>
            </section>
          </div>

          <p v-if="filteredSeedGroups.length === 0" class="empty-search">
            没有匹配的种子，可以直接在右侧自然语言场景里输入新关键词。
          </p>
        </div>

        <div class="composer-panel">
          <div class="panel-heading">
            <div>
              <span class="panel-kicker">SCENARIO DRAFT</span>
              <h2>整理推演文本</h2>
            </div>
            <button type="button" class="quiet-button" @click="loadExample">
              载入示例
            </button>
          </div>

          <label class="field-label" for="rawPrompt">自然语言场景</label>
          <textarea
            id="rawPrompt"
            v-model="rawPrompt"
            class="prompt-input"
            rows="7"
            placeholder="例如：一名不满被裁员的员工在匿名职场社区发帖，称公司为防止摸鱼偷偷安装监控软件，结果被猎豹VPN反噬导致整个内网瘫痪。"
            :disabled="loading"
          ></textarea>

          <div class="action-row">
            <button type="button" class="secondary-button" @click="extractSeedsFromPrompt" :disabled="!rawPrompt.trim() || loading">
              从文本提取种子
            </button>
            <button type="button" class="secondary-button" @click="generatePrompt" :disabled="!canCompose || loading">
              生成推演提示词
            </button>
            <button type="button" class="ghost-button" @click="clearAll" :disabled="loading">
              重置
            </button>
          </div>

          <div class="selected-strip" v-if="selectedTags.length > 0">
            <span class="selected-label">已选种子</span>
            <button
              v-for="tag in selectedTags"
              :key="tag.key"
              type="button"
              class="selected-pill"
              @click="toggleSeed(tag.groupKey, tag.name)"
            >
              {{ tag.name }}
              <span aria-hidden="true">×</span>
            </button>
          </div>

          <label class="field-label" for="generatedPrompt">最终推演提示词</label>
          <textarea
            id="generatedPrompt"
            v-model="generatedPrompt"
            class="generated-input"
            rows="10"
            placeholder="点击「生成推演提示词」，或直接在这里编辑最终提交给推演引擎的需求。"
            :disabled="loading"
          ></textarea>

          <section class="upload-section">
            <div class="upload-copy">
              <span class="upload-title">补充资料</span>
              <span class="upload-hint">可选：PDF / MD / TXT，用于补充事实背景</span>
            </div>

            <div
              class="upload-zone"
              :class="{ 'drag-over': isDragOver, 'has-files': files.length > 0 }"
              @dragover.prevent="handleDragOver"
              @dragleave.prevent="handleDragLeave"
              @drop.prevent="handleDrop"
              @click="triggerFileInput"
            >
              <input
                ref="fileInput"
                type="file"
                multiple
                accept=".pdf,.md,.markdown,.txt"
                @change="handleFileSelect"
                :disabled="loading"
              />

              <div v-if="files.length === 0" class="upload-empty">
                <span class="upload-plus">+</span>
                <span>拖拽文件到这里，或点击选择</span>
              </div>

              <div v-else class="file-list">
                <div v-for="(file, index) in files" :key="`${file.name}-${index}`" class="file-item">
                  <span class="file-type">{{ getFileType(file.name) }}</span>
                  <span class="file-name">{{ file.name }}</span>
                  <button type="button" class="remove-file" @click.stop="removeFile(index)">移除</button>
                </div>
              </div>
            </div>
          </section>

          <p v-if="notice" class="notice-text">{{ notice }}</p>
          <p v-if="error" class="error-text">{{ error }}</p>

          <button
            type="button"
            class="launch-button"
            @click="startSimulation"
            :disabled="!canSubmit || loading"
          >
            <span class="launch-label">
              <span class="launch-tag">LAUNCH</span>
              <span>{{ loading ? '正在初始化...' : '生成并进入推演' }}</span>
            </span>
            <span class="launch-arrow" aria-hidden="true">›</span>
          </button>
        </div>
      </section>

      <HistoryDatabase />
    </main>

    <footer class="bottom-bar">
      <span>VOXWAVE · PREDICTIVE ANALYTICS v0.1</span>
      <span class="bottom-bar-divider"></span>
      <span>舆情种子生成 · 多智能体模拟 · 风险研判</span>
      <span class="bottom-bar-divider"></span>
      <span>© {{ year }} VoxWave Lab · AGPL-3.0</span>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import HistoryDatabase from '../components/HistoryDatabase.vue'
import LanguageSwitcher from '../components/LanguageSwitcher.vue'
import AppBrand from '../components/AppBrand.vue'

const router = useRouter()

const seedGroups = [
  {
    key: 'scene',
    label: '场景',
    color: '#22d3ee',
    items: [
      '校园', '幼儿园', '中小学', '高校', '培训机构', '家庭', '小区', '社区', '村镇', '街道办', '政务大厅', '派出所',
      '法院', '医院', '药店', '养老院', '福利院', '公司', '工厂', '园区', '写字楼', '商场', '超市', '餐饮店',
      '酒店', '景区', '机场', '高铁站', '地铁', '公交', '高速公路', '物流仓库', '银行网点', '证券营业部',
      '直播间', '电商平台', '匿名职场社区', '本地论坛', '业主群', '粉丝社群', '游戏社区', '海外社媒', '新闻评论区'
    ]
  },
  {
    key: 'role',
    label: '角色',
    color: '#34d399',
    items: [
      '学生', '家长', '老师', '校方', '医生', '护士', '患者', '患者家属', '公司员工', '被裁员员工', 'HR', '企业管理层',
      '基层干部', '执法人员', '监管部门', '平台运营', '客服', '商家', '消费者', '业主', '物业', '司机', '乘客',
      '外卖骑手', '快递员', '主播', '网红', '粉丝', '普通网友', '意见领袖', '媒体记者', '自媒体账号', '专家学者',
      '律师', '公益组织', '维权群体', '投资者', '供应商', '竞品公司', '境外账号', '机器人账号'
    ]
  },
  {
    key: 'event',
    label: '事件',
    color: '#f472b6',
    items: [
      '裁员', '欠薪', '降薪', '加班争议', '工伤', '绩效争议', '职场霸凌', '校园霸凌', '师德争议', '考试作弊',
      '招生争议', '学术不端', '医疗纠纷', '药品安全', '医美事故', '食品安全', '餐饮卫生', '产品质量', '虚假宣传',
      '价格争议', '退费纠纷', '物业矛盾', '房屋质量', '烂尾楼', '租房纠纷', '交通事故', '公共安全事故', '火灾',
      '爆炸', '自然灾害', '暴雨内涝', '地震', '台风', '疫情', '环境污染', '噪音扰民', '数据泄露', '网络攻击',
      '系统宕机', '算法歧视', '账号封禁', '版权纠纷', '明星塌房', '饭圈冲突', '体育争议', '旅游宰客', '政务失误',
      '执法争议', '公共道歉', '谣言传播', '辟谣澄清', '舆情反转', '跨境争议'
    ]
  },
  {
    key: 'action',
    label: '行为',
    color: '#fb923c',
    items: [
      '发帖', '爆料', '实名举报', '匿名投稿', '求助', '投诉', '晒证据', '直播', '录音曝光', '视频曝光', '长文控诉',
      '截图传播', '转发', '评论', '点赞', '收藏', '二创', '剪辑', '玩梗', '抵制', '差评', '刷屏', '举报账号',
      '冲榜', '带节奏', '删帖', '限流', '封号', '道歉', '澄清', '辟谣', '沉默', '冷处理', '回应质疑', '法律函',
      '报警', '行政处罚', '赔偿', '召回', '线下聚集', '舆论监督'
    ]
  },
  {
    key: 'issue',
    label: '议题',
    color: '#a78bfa',
    items: [
      '劳动权益', '薪酬公平', '就业压力', '教育公平', '未成年人保护', '校园安全', '师生关系', '医疗资源', '医患信任',
      '食品安全', '消费权益', '价格透明', '住房民生', '物业服务', '交通安全', '公共秩序', '基层治理', '执法边界',
      '政府公信力', '企业责任', '平台责任', '算法透明', '隐私保护', '数据安全', '网络安全', '监控软件', 'VPN',
      '猎豹VPN', '内网', '摸鱼', '绩效考核', '管理失当', '性别议题', '年龄歧视', '地域偏见', '贫富差距', '环保',
      '动物保护', '民族宗教', '国际关系', '爱国情绪', '饭圈文化', '品牌信任', '危机公关', '信任危机'
    ]
  },
  {
    key: 'platform',
    label: '平台',
    color: '#22d3ee',
    items: [
      '微博', '微信朋友圈', '微信群', '公众号', '视频号', '抖音', '快手', '小红书', '知乎', 'B站', '贴吧', '豆瓣',
      '脉脉', 'Boss直聘社区', '黑猫投诉', '12345平台', '人民网留言板', '本地论坛', '新闻客户端', '今日头条',
      '直播平台', '电商评论区', '闲鱼', '淘宝', '京东', '拼多多', '企业内网', '内部论坛', '邮件列表', 'Telegram',
      'X/Twitter', 'Facebook', 'Instagram', 'YouTube', 'TikTok', 'Reddit'
    ]
  },
  {
    key: 'time',
    label: '时间阶段',
    color: '#60a5fa',
    items: [
      '工作日早高峰', '午休时段', '下班后', '深夜发酵', '周末', '节假日', '开学季', '毕业季', '招聘季', '双十一',
      '春节', '清明', '五一', '暑期', '汛期', '重大会议期间', '考试前后', '发布会前', '事故发生后1小时',
      '事故发生后6小时', '事故发生后24小时', '黄金回应期', '舆情长尾期'
    ]
  },
  {
    key: 'sentiment',
    label: '情绪倾向',
    color: '#f472b6',
    items: [
      '愤怒', '恐慌', '焦虑', '同情', '质疑', '嘲讽', '失望', '震惊', '猎奇', '围观', '支持', '反感',
      '民族情绪', '群体对立', '不信任', '维权诉求', '道德审判', '理性讨论', '疲劳厌倦', '二次愤怒'
    ]
  },
  {
    key: 'spread',
    label: '传播阶段',
    color: '#a78bfa',
    items: [
      '首发爆料', '圈层扩散', '本地发酵', '跨平台搬运', '大V介入', '媒体跟进', '热搜冲榜', '平台推荐放大',
      '谣言混入', '证据补充', '当事人回应', '官方通报', '舆情反转', '对立阵营形成', '线下行动外溢',
      '二次传播', '长尾讨论', '记忆沉淀'
    ]
  },
  {
    key: 'response',
    label: '处置策略',
    color: '#94a3b8',
    items: [
      '快速确认事实', '先行安抚', '主动公开', '分阶段通报', '第三方调查', '专家解读', '当事人沟通', '平台治理',
      '评论区管理', '证据链整理', '公开道歉', '补偿方案', '责任追究', '法律回应', '线下协调', '媒体沟通',
      '谣言澄清', '风险隔离', '复盘整改', '持续更新'
    ]
  },
  {
    key: 'risk',
    label: '风险类型',
    color: '#fbbf24',
    items: [
      '声誉风险', '合规风险', '法律诉讼', '监管问询', '客户流失', '员工流失', '供应链风险', '股价波动',
      '线下聚集', '群体性事件', '次生谣言', '人肉搜索', '网络暴力', '隐私泄露', '境外放大', '跨圈层误读',
      '舆情反噬', '处置失当', '长期信任受损'
    ]
  },
  {
    key: 'region',
    label: '地域层级',
    color: '#34d399',
    items: [
      '一线城市', '新一线城市', '二三线城市', '县城', '乡镇', '农村', '东北', '华北', '华东', '华南',
      '华中', '西南', '西北', '沿海地区', '内陆地区', '高校集中区', '产业园区', '边境地区', '海外华人社区',
      '跨国传播'
    ]
  }
]

const aliasMap = {
  公司: ['企业', '单位', '厂里', '公司里', '雇主'],
  工厂: ['厂区', '制造业', '车间'],
  校园: ['学校', '校内', '大学', '高校'],
  医院: ['医疗机构', '门诊', '急诊'],
  政务大厅: ['办事大厅', '窗口单位', '政务窗口'],
  匿名职场社区: ['匿名社区', '职场社区', '职场论坛'],
  本地论坛: ['地方论坛', '本地贴吧', '同城论坛'],
  业主群: ['业主微信群', '小区群', '居民群'],
  被裁员员工: ['不满被裁员', '被优化员工', '离职员工', '裁员员工', '被辞退员工'],
  企业管理层: ['老板', '高管', '管理层', '公司领导'],
  监管部门: ['监管', '主管部门', '有关部门'],
  意见领袖: ['大V', '博主', 'KOL', '头部账号'],
  自媒体账号: ['营销号', '自媒体', '媒体号'],
  基层干部: ['社区干部', '村干部', '街道干部'],
  裁员: ['被裁', '优化', '裁撤', '裁人', '辞退'],
  欠薪: ['拖欠工资', '工资没发', '讨薪'],
  降薪: ['降工资', '薪资下调'],
  加班争议: ['强制加班', '996', '加班费'],
  校园霸凌: ['校园欺凌', '学生被欺负'],
  医疗纠纷: ['医患纠纷', '看病纠纷', '医疗事故'],
  食品安全: ['吃出异物', '食物中毒', '卫生问题'],
  产品质量: ['质量问题', '产品翻车', '故障频发'],
  数据泄露: ['信息泄露', '隐私泄露', '资料外泄'],
  网络攻击: ['黑客攻击', '被攻击', '入侵'],
  系统宕机: ['瘫痪', '宕机', '停摆', '崩了', '服务中断'],
  暴雨内涝: ['大暴雨', '城市内涝', '积水'],
  谣言传播: ['谣传', '假消息', '不实消息'],
  舆情反转: ['反转', '剧情反转', '真相反相'],
  发帖: ['发文', '帖子', '发到', '发布'],
  爆料: ['曝光', '爆出', '爆料称'],
  晒证据: ['放证据', '贴证据', '截图为证'],
  长文控诉: ['小作文', '长文爆料', '控诉文'],
  质疑: ['不满', '质疑称', '怀疑'],
  澄清: ['回应', '说明', '官方回应'],
  辟谣: ['澄清不实', '回应不实', '辟谣称'],
  删帖: ['删除帖子', '撤稿', '下架内容'],
  道歉: ['致歉', '公开道歉'],
  监控软件: ['监控系统', '监控员工', '电脑监控'],
  摸鱼: ['防止摸鱼', '上班摸鱼'],
  VPN: ['vpn', 'VPN'],
  猎豹VPN: ['猎豹vpn', '猎豹VPN'],
  内网: ['公司内网', '办公网'],
  隐私保护: ['个人隐私', '隐私侵犯', '隐私'],
  劳动权益: ['劳动法', '员工权益', '劳动争议'],
  信息安全: ['安全事故', '网络安全'],
  平台责任: ['平台有没有责任', '平台治理'],
  微信群: ['群聊', '微信群'],
  微博: ['微博', '热搜'],
  抖音: ['短视频', '抖音'],
  B站: ['b站', 'B站', '哔哩哔哩'],
  小红书: ['小红书'],
  脉脉: ['脉脉'],
  'X/Twitter': ['Twitter', 'X平台', '推特'],
  首发爆料: ['最早爆料', '第一条帖子', '源头'],
  大V介入: ['大V转发', '博主介入', '头部账号下场'],
  热搜冲榜: ['上热搜', '冲热搜', '榜单'],
  官方通报: ['通报', '官方发布', '警方通报'],
  黄金回应期: ['黄金4小时', '黄金24小时', '紧急回应'],
  声誉风险: ['口碑风险', '品牌形象受损'],
  线下聚集: ['线下维权', '聚集', '围堵'],
  人肉搜索: ['开盒', '扒身份', '曝光个人信息'],
  境外放大: ['外媒关注', '境外势力', '海外发酵']
}

const seedLookup = seedGroups.reduce((acc, group) => {
  group.items.forEach((name) => {
    const key = `${group.key}:${name}`
    acc[key] = {
      key,
      groupKey: group.key,
      groupLabel: group.label,
      name
    }
  })
  return acc
}, {})

const rawPrompt = ref('')
const generatedPrompt = ref('')
const seedSearch = ref('')
const selectedSeedKeys = ref([])
const files = ref([])
const loading = ref(false)
const error = ref('')
const notice = ref('')
const isDragOver = ref(false)
const fileInput = ref(null)

const now = ref('')
const tz = ref('')
const year = new Date().getFullYear()
const ready = ref(true)
let nowTimer = null

const normalizeText = (value) => String(value || '').toLowerCase()

const selectedTags = computed(() => selectedSeedKeys.value.map((key) => seedLookup[key]).filter(Boolean))
const totalSeedCount = computed(() => seedGroups.reduce((sum, group) => sum + group.items.length, 0))
const filteredSeedGroups = computed(() => {
  const keyword = normalizeText(seedSearch.value.trim())
  if (!keyword) return seedGroups

  return seedGroups
    .map((group) => {
      const groupMatched = normalizeText(group.label).includes(keyword)
      const items = groupMatched
        ? group.items
        : group.items.filter((item) => {
            const aliases = aliasMap[item] || []
            return [item, ...aliases].some((text) => normalizeText(text).includes(keyword))
          })
      return { ...group, items }
    })
    .filter((group) => group.items.length > 0)
})
const visibleSeedCount = computed(() => filteredSeedGroups.value.reduce((sum, group) => sum + group.items.length, 0))
const canCompose = computed(() => rawPrompt.value.trim() !== '' || selectedTags.value.length > 0)
const canSubmit = computed(() => generatedPrompt.value.trim() !== '' || canCompose.value)

const isSelected = (groupKey, name) => selectedSeedKeys.value.includes(`${groupKey}:${name}`)

const toggleSeed = (groupKey, name) => {
  const key = `${groupKey}:${name}`
  if (selectedSeedKeys.value.includes(key)) {
    selectedSeedKeys.value = selectedSeedKeys.value.filter((item) => item !== key)
  } else {
    selectedSeedKeys.value = [...selectedSeedKeys.value, key]
  }
  notice.value = ''
  error.value = ''
}

const getGroupSelectedCount = (groupKey) => {
  return selectedTags.value.filter((tag) => tag.groupKey === groupKey).length
}

const getSelectedByGroup = () => {
  return seedGroups.map((group) => ({
    ...group,
    selected: selectedTags.value.filter((tag) => tag.groupKey === group.key).map((tag) => tag.name)
  }))
}

const extractSeedsFromPrompt = () => {
  const text = rawPrompt.value.trim()
  if (!text) return

  const lowerText = text.toLowerCase()
  const found = []

  seedGroups.forEach((group) => {
    group.items.forEach((item) => {
      const aliases = [item, ...(aliasMap[item] || [])]
      const matched = aliases.some((alias) => {
        const target = alias === alias.toUpperCase() ? text : lowerText
        const keyword = alias === alias.toUpperCase() ? alias : alias.toLowerCase()
        return target.includes(keyword)
      })
      if (matched) found.push(`${group.key}:${item}`)
    })
  })

  selectedSeedKeys.value = Array.from(new Set([...selectedSeedKeys.value, ...found]))
  if (found.length > 0) {
    notice.value = `已从文本中提取 ${new Set(found).size} 个种子词。`
    error.value = ''
    generatePrompt()
  } else {
    notice.value = ''
    error.value = '暂未匹配到种子词，可以手动选择标签后再生成。'
  }
}

const generatePrompt = () => {
  generatedPrompt.value = buildSimulationPrompt()
  notice.value = '已生成结构化推演提示词，可继续编辑后启动。'
  error.value = ''
}

const buildSimulationPrompt = () => {
  const groups = getSelectedByGroup()
  const selectedLines = groups
    .filter((group) => group.selected.length > 0)
    .map((group) => `${group.label}: ${group.selected.join('、')}`)

  const sceneText = groups.find((group) => group.key === 'scene')?.selected.join('、') || '综合社会场景'
  const roleText = groups.find((group) => group.key === 'role')?.selected.join('、') || '相关公众与利益相关方'
  const eventText = groups.find((group) => group.key === 'event')?.selected.join('、') || '待分析事件'
  const platformText = groups.find((group) => group.key === 'platform')?.selected.join('、') || '社交媒体与社区平台'
  const issueText = groups.find((group) => group.key === 'issue')?.selected.join('、') || '公众关注议题'
  const actionText = groups.find((group) => group.key === 'action')?.selected.join('、') || '发布、评论、转发、质疑与回应'
  const timeText = groups.find((group) => group.key === 'time')?.selected.join('、') || '事件发生后的关键传播窗口'
  const sentimentText = groups.find((group) => group.key === 'sentiment')?.selected.join('、') || '多种公众情绪混合演化'
  const spreadText = groups.find((group) => group.key === 'spread')?.selected.join('、') || '从首发到跨平台扩散再到长尾讨论'
  const responseText = groups.find((group) => group.key === 'response')?.selected.join('、') || '事实核验、公开回应、持续更新'
  const riskText = groups.find((group) => group.key === 'risk')?.selected.join('、') || '声誉、合规、次生传播与线下外溢风险'
  const regionText = groups.find((group) => group.key === 'region')?.selected.join('、') || '根据事件属性设定地域扩散范围'
  const sourceText = rawPrompt.value.trim() || '用户未提供自然语言描述，请基于已选种子构建合理的舆情初始事件。'

  return `请作为舆情分析与社会模拟系统，围绕以下事件搭建可推演的舆论场景，并预测不同群体在多平台上的传播、情绪、观点分化与风险演化。

【原始描述】
${sourceText}

【种子词】
${selectedLines.length > 0 ? selectedLines.join('\n') : '未手动选择种子词，请从原始描述中提取。'}

【场景设定】
- 主要场景：${sceneText}
- 核心角色：${roleText}
- 事件触发：${eventText}
- 传播平台：${platformText}
- 关键议题：${issueText}
- 典型行为：${actionText}
- 时间窗口：${timeText}
- 情绪倾向：${sentimentText}
- 传播阶段：${spreadText}
- 处置策略：${responseText}
- 风险类型：${riskText}
- 地域层级：${regionText}

【推演目标】
1. 识别首发叙事、反方叙事、平台扩散路径、关键意见节点与可能的情绪峰值。
2. 模拟不同角色的发帖、评论、转发、质疑、辟谣、道歉或沉默策略。
3. 预测舆情在 6 到 48 小时内的热度变化、风险拐点、误读链条与二次传播点。
4. 输出风险等级、处置优先级、沟通建议与需要进一步核实的事实清单。

【边界要求】
仅用于舆情研判、风险预警和合规处置推演；不要生成煽动攻击、骚扰、人肉搜索或规避监管的操作话术。`
}

const buildSeedDocument = () => {
  const groups = getSelectedByGroup()
  const seedBlocks = groups
    .map((group) => `## ${group.label}\n${group.selected.length > 0 ? group.selected.map((item) => `- ${item}`).join('\n') : '- 未选择'}`)
    .join('\n\n')

  return `# 舆情场景种子

## 原始输入
${rawPrompt.value.trim() || '无'}

${seedBlocks}

## 最终推演提示词
${generatedPrompt.value.trim() || buildSimulationPrompt()}
`
}

const createSeedFile = () => {
  return new File([buildSeedDocument()], '舆情场景种子.txt', { type: 'text/plain' })
}

const loadExample = () => {
  rawPrompt.value = '一名不满被裁员的员工在匿名职场社区（如脉脉、小红书）发帖："公司为了防止摸鱼，偷偷装了监控软件，结果反而被猎豹VPN反噬，把整个内网搞瘫痪了。"'
  extractSeedsFromPrompt()
}

const clearSeeds = () => {
  selectedSeedKeys.value = []
  notice.value = ''
  error.value = ''
}

const clearAll = () => {
  rawPrompt.value = ''
  generatedPrompt.value = ''
  selectedSeedKeys.value = []
  files.value = []
  notice.value = ''
  error.value = ''
  if (fileInput.value) fileInput.value.value = ''
}

const scrollToTop = () => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const triggerFileInput = () => {
  if (!loading.value) fileInput.value?.click()
}

const handleFileSelect = (event) => {
  addFiles(Array.from(event.target.files || []))
  event.target.value = ''
}

const handleDragOver = () => {
  if (!loading.value) isDragOver.value = true
}

const handleDragLeave = () => {
  isDragOver.value = false
}

const handleDrop = (event) => {
  isDragOver.value = false
  if (loading.value) return
  addFiles(Array.from(event.dataTransfer.files || []))
}

const addFiles = (newFiles) => {
  const validFiles = newFiles.filter((file) => {
    const ext = file.name.split('.').pop()?.toLowerCase()
    return ['pdf', 'md', 'markdown', 'txt'].includes(ext)
  })

  const existing = new Set(files.value.map((file) => `${file.name}-${file.size}`))
  const uniqueFiles = validFiles.filter((file) => !existing.has(`${file.name}-${file.size}`))
  files.value.push(...uniqueFiles)

  if (validFiles.length !== newFiles.length) {
    error.value = '已忽略不支持的文件格式，仅支持 PDF、MD、TXT。'
  } else {
    error.value = ''
  }
}

const removeFile = (index) => {
  files.value.splice(index, 1)
}

const getFileType = (filename) => {
  return filename.split('.').pop()?.toUpperCase() || 'FILE'
}

const startSimulation = () => {
  if (!canSubmit.value || loading.value) return

  if (!generatedPrompt.value.trim()) {
    generatedPrompt.value = buildSimulationPrompt()
  }

  loading.value = true
  const uploadFiles = [createSeedFile(), ...files.value]

  import('../store/pendingUpload.js').then(({ setPendingUpload }) => {
    setPendingUpload(uploadFiles, generatedPrompt.value.trim())
    router.push({
      name: 'Process',
      params: { projectId: 'new' }
    })
  }).finally(() => {
    loading.value = false
  })
}

const formatNow = () => {
  const d = new Date()
  const pad = (n) => String(n).padStart(2, '0')
  now.value = `${d.getFullYear()}.${pad(d.getMonth() + 1)}.${pad(d.getDate())} ${pad(d.getHours())}:${pad(d.getMinutes())}`
}

const detectTz = () => {
  try {
    tz.value = Intl.DateTimeFormat().resolvedOptions().timeZone || 'Local'
  } catch (e) {
    tz.value = 'Local'
  }
}

onMounted(() => {
  formatNow()
  detectTz()
  nowTimer = setInterval(formatNow, 30000)
})

onUnmounted(() => {
  if (nowTimer) clearInterval(nowTimer)
})
</script>

<style scoped>
.home-page {
  min-height: 100vh;
  color: #e2e8f0;
  font-family: 'Inter', 'Noto Sans SC', system-ui, sans-serif;
}

.topbar {
  position: sticky;
  top: 0;
  z-index: 30;
  height: 68px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 32px;
  background: rgba(11, 18, 32, 0.85);
  border-bottom: 1px solid #1e293b;
  backdrop-filter: blur(14px);
}

.topbar::after {
  content: '';
  position: absolute;
  inset: auto 0 -1px 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, #22d3ee, #a78bfa, transparent);
  opacity: 0.4;
}

.topbar-meta {
  display: flex;
  align-items: center;
  gap: 14px;
}

.meta-item {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.16em;
  color: #94a3b8;
  text-transform: uppercase;
}

.meta-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #22d3ee;
  box-shadow: 0 0 6px rgba(34, 211, 238, 0.7);
  animation: pulse 1.6s ease-in-out infinite;
}

.meta-version {
  color: #64748b;
}

.meta-divider {
  width: 1px;
  height: 16px;
  background: #1e293b;
}

.repo-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 14px;
  border: 1px solid #1e293b;
  border-radius: 8px;
  background: rgba(15, 23, 42, 0.6);
  color: #e2e8f0;
  text-decoration: none;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.12em;
  transition: border-color 0.2s ease, box-shadow 0.2s ease, color 0.2s ease;
}

.repo-link:hover {
  border-color: rgba(167, 139, 250, 0.6);
  color: #a78bfa;
  box-shadow: 0 0 0 3px rgba(167, 139, 250, 0.1);
}

.repo-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #a78bfa;
  box-shadow: 0 0 6px rgba(167, 139, 250, 0.7);
}

.workspace {
  width: min(1360px, calc(100% - 48px));
  margin: 0 auto;
  padding: 36px 0 28px;
}

.hero {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 360px;
  gap: 28px;
  margin-bottom: 32px;
  padding: 28px;
  border: 1px solid #1e293b;
  border-radius: 16px;
  background:
    radial-gradient(700px 320px at 0% 0%, rgba(34, 211, 238, 0.12), transparent 60%),
    radial-gradient(700px 320px at 100% 100%, rgba(167, 139, 250, 0.12), transparent 60%),
    #0f172a;
  position: relative;
  overflow: hidden;
}

.hero::before {
  content: '';
  position: absolute;
  inset: 0;
  background-image:
    linear-gradient(rgba(148, 163, 184, 0.04) 1px, transparent 1px),
    linear-gradient(90deg, rgba(148, 163, 184, 0.04) 1px, transparent 1px);
  background-size: 32px 32px;
  pointer-events: none;
}

.hero-copy,
.hero-panel {
  position: relative;
  z-index: 1;
}

.hero-eyebrow {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 18px;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.2em;
  color: #22d3ee;
}

.hero-eyebrow-bar {
  width: 28px;
  height: 2px;
  background: linear-gradient(90deg, #22d3ee, transparent);
}

.hero-copy h1 {
  margin: 0;
  font-size: 44px;
  line-height: 1.1;
  font-weight: 800;
  letter-spacing: 0.01em;
  color: #e2e8f0;
}

.hero-accent {
  background: linear-gradient(120deg, #22d3ee 0%, #a78bfa 60%, #f472b6 100%);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
}

.hero-text {
  max-width: 620px;
  margin: 18px 0 0;
  color: #94a3b8;
  font-size: 15px;
  line-height: 1.85;
}

.hero-stats {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 12px;
  margin-top: 26px;
}

.stat-block {
  position: relative;
  padding: 14px 16px;
  border: 1px solid #1e293b;
  border-radius: 12px;
  background: rgba(11, 18, 32, 0.55);
  display: flex;
  flex-direction: column;
  gap: 8px;
  overflow: hidden;
}

.stat-block::before {
  content: '';
  position: absolute;
  inset: 0 0 auto 0;
  height: 2px;
  background: linear-gradient(90deg, #22d3ee, transparent);
  opacity: 0.6;
}

.stat-block-accent {
  background:
    linear-gradient(180deg, rgba(34, 211, 238, 0.08), rgba(167, 139, 250, 0.06));
  border-color: rgba(34, 211, 238, 0.35);
}

.stat-block-accent::before {
  background: linear-gradient(90deg, #22d3ee, #a78bfa);
  opacity: 1;
}

.stat-label {
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.18em;
  color: #64748b;
}

.stat-value {
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 24px;
  font-weight: 700;
  color: #e2e8f0;
  letter-spacing: 0.02em;
}

.hero-panel {
  border: 1px solid #1e293b;
  border-radius: 14px;
  background: rgba(11, 18, 32, 0.7);
  padding: 18px 20px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.hero-panel-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.panel-tag {
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.2em;
  color: #94a3b8;
}

.panel-pulse {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.18em;
  color: #34d399;
}

.panel-pulse span {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #34d399;
  box-shadow: 0 0 8px rgba(52, 211, 153, 0.7);
  animation: pulse 1.4s ease-in-out infinite;
}

.hero-panel-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
}

.hero-panel-list li {
  display: grid;
  grid-template-columns: 12px 1fr auto;
  align-items: center;
  gap: 10px;
  padding: 10px 0;
  border-bottom: 1px dashed #1e293b;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  color: #94a3b8;
  letter-spacing: 0.06em;
}

.hero-panel-list li:last-child {
  border-bottom: 0;
}

.hero-panel-list .key {
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.16em;
  font-size: 10px;
}

.hero-panel-list .value {
  color: #e2e8f0;
  font-weight: 600;
}

.dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.dot-cyan { background: #22d3ee; box-shadow: 0 0 8px rgba(34, 211, 238, 0.6); }
.dot-purple { background: #a78bfa; box-shadow: 0 0 8px rgba(167, 139, 250, 0.6); }
.dot-magenta { background: #f472b6; box-shadow: 0 0 8px rgba(244, 114, 182, 0.6); }
.dot-green { background: #34d399; box-shadow: 0 0 8px rgba(52, 211, 153, 0.6); }
.dot-amber { background: #fbbf24; box-shadow: 0 0 8px rgba(251, 191, 36, 0.6); }

.hero-panel-footer {
  margin-top: 4px;
  padding-top: 14px;
  border-top: 1px solid #1e293b;
  display: flex;
  align-items: center;
  gap: 10px;
  color: #64748b;
  font-size: 11px;
  letter-spacing: 0.06em;
}

.footer-bar {
  flex: 0 0 auto;
  width: 20px;
  height: 2px;
  background: linear-gradient(90deg, #22d3ee, #a78bfa);
}

.generator-layout {
  display: grid;
  grid-template-columns: minmax(0, 1.05fr) minmax(420px, 0.95fr);
  gap: 22px;
  align-items: start;
}

.selector-panel,
.composer-panel {
  border: 1px solid #1e293b;
  border-radius: 14px;
  background:
    linear-gradient(180deg, rgba(15, 23, 42, 0.96), rgba(11, 18, 32, 0.96));
  box-shadow: 0 24px 48px -32px rgba(34, 211, 238, 0.25);
}

.selector-panel {
  padding: 24px;
}

.composer-panel {
  position: sticky;
  top: 92px;
  padding: 24px;
}

.panel-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 20px;
}

.panel-heading h2 {
  margin: 4px 0 0;
  font-size: 22px;
  line-height: 1.3;
  color: #e2e8f0;
}

.panel-kicker {
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.2em;
  color: #22d3ee;
  text-transform: uppercase;
}

.quiet-button,
.ghost-button,
.secondary-button {
  min-height: 36px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 700;
  font-size: 12px;
  letter-spacing: 0.06em;
}

.quiet-button {
  padding: 0 12px;
  border: 1px solid #1e293b;
  background: rgba(15, 23, 42, 0.7);
  color: #94a3b8;
}

.quiet-button:hover {
  border-color: rgba(34, 211, 238, 0.45);
  color: #22d3ee;
}

.quiet-button:disabled,
.ghost-button:disabled,
.secondary-button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.seed-toolbar {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  align-items: end;
  gap: 14px;
  margin-bottom: 20px;
  padding: 14px;
  border-radius: 10px;
  background: rgba(11, 18, 32, 0.7);
  border: 1px solid #1e293b;
}

.seed-search {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.seed-search span {
  color: #64748b;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.seed-search input {
  width: 100%;
  min-height: 38px;
  border: 1px solid #1e293b;
  border-radius: 8px;
  background: rgba(11, 18, 32, 0.85);
  color: #e2e8f0;
  outline: none;
  padding: 0 12px;
  font-size: 13px;
}

.seed-search input::placeholder {
  color: #475569;
}

.seed-search input:focus {
  border-color: rgba(34, 211, 238, 0.55);
  box-shadow: 0 0 0 3px rgba(34, 211, 238, 0.12);
}

.seed-total {
  min-height: 38px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0 12px;
  border-radius: 8px;
  background: rgba(34, 211, 238, 0.12);
  color: #22d3ee;
  border: 1px solid rgba(34, 211, 238, 0.3);
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 12px;
  font-weight: 800;
  white-space: nowrap;
}

.seed-groups {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
}

.seed-group {
  padding-top: 2px;
}

.group-title-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 10px;
}

.group-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--group-color);
  box-shadow: 0 0 8px var(--group-color);
}

.group-title-row h3 {
  margin: 0;
  font-size: 14px;
  font-weight: 700;
  color: #e2e8f0;
}

.group-count {
  margin-left: auto;
  color: #64748b;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  letter-spacing: 0.08em;
}

.chip-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.seed-chip {
  min-height: 30px;
  padding: 0 11px;
  border: 1px solid #1e293b;
  border-radius: 8px;
  background: rgba(11, 18, 32, 0.7);
  color: #cbd5e1;
  cursor: pointer;
  font-size: 12px;
  transition: background 0.18s ease, border-color 0.18s ease, color 0.18s ease, transform 0.18s ease, box-shadow 0.18s ease;
}

.seed-chip:hover {
  transform: translateY(-1px);
  border-color: var(--group-color);
  color: #e2e8f0;
  box-shadow: 0 0 0 3px rgba(34, 211, 238, 0.06);
}

.seed-chip.active {
  color: #0b1220;
  background: var(--group-color);
  border-color: var(--group-color);
  font-weight: 700;
  box-shadow: 0 0 12px color-mix(in srgb, var(--group-color) 50%, transparent);
}

.empty-search {
  margin: 18px 0 0;
  color: #64748b;
  font-size: 13px;
  line-height: 1.6;
}

.field-label {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 18px 0 8px;
  color: #cbd5e1;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.field-label::before {
  content: '';
  width: 14px;
  height: 2px;
  background: linear-gradient(90deg, #22d3ee, transparent);
}

.prompt-input,
.generated-input {
  width: 100%;
  border: 1px solid #1e293b;
  border-radius: 10px;
  background: rgba(11, 18, 32, 0.85);
  color: #e2e8f0;
  resize: vertical;
  outline: none;
  padding: 14px;
  font: 14px/1.72 'Noto Sans SC', 'Inter', system-ui, sans-serif;
}

.prompt-input::placeholder,
.generated-input::placeholder {
  color: #475569;
}

.prompt-input:focus,
.generated-input:focus {
  border-color: rgba(34, 211, 238, 0.55);
  box-shadow: 0 0 0 3px rgba(34, 211, 238, 0.12);
}

.generated-input {
  font-family: 'JetBrains Mono', 'Noto Sans SC', monospace;
  font-size: 12.5px;
  line-height: 1.7;
}

.action-row {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 12px;
}

.secondary-button {
  padding: 0 14px;
  border: 1px solid rgba(34, 211, 238, 0.5);
  background: rgba(34, 211, 238, 0.1);
  color: #22d3ee;
  transition: background 0.18s ease, border-color 0.18s ease, box-shadow 0.18s ease;
}

.secondary-button:hover:not(:disabled) {
  background: rgba(34, 211, 238, 0.2);
  border-color: #22d3ee;
  box-shadow: 0 0 0 3px rgba(34, 211, 238, 0.1);
}

.ghost-button {
  padding: 0 14px;
  border: 1px solid #1e293b;
  background: rgba(15, 23, 42, 0.7);
  color: #94a3b8;
}

.ghost-button:hover:not(:disabled) {
  color: #e2e8f0;
  border-color: rgba(148, 163, 184, 0.5);
}

.selected-strip {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 8px;
  margin-top: 16px;
  padding: 12px;
  border-radius: 10px;
  background: rgba(11, 18, 32, 0.7);
  border: 1px solid #1e293b;
}

.selected-label {
  margin-right: 4px;
  color: #64748b;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.16em;
  text-transform: uppercase;
}

.selected-pill {
  min-height: 26px;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  border: 1px solid rgba(34, 211, 238, 0.4);
  border-radius: 999px;
  background: rgba(34, 211, 238, 0.08);
  color: #22d3ee;
  cursor: pointer;
  padding: 0 10px;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.04em;
  transition: background 0.18s ease, border-color 0.18s ease;
}

.selected-pill:hover {
  background: rgba(34, 211, 238, 0.2);
  border-color: #22d3ee;
}

.upload-section {
  margin-top: 18px;
}

.upload-copy {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 8px;
}

.upload-title {
  color: #cbd5e1;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.upload-hint {
  color: #64748b;
  font-size: 12px;
}

.upload-zone {
  min-height: 116px;
  border: 1px dashed #334155;
  border-radius: 10px;
  background: rgba(11, 18, 32, 0.6);
  cursor: pointer;
  transition: background 0.18s ease, border-color 0.18s ease, box-shadow 0.18s ease;
}

.upload-zone input {
  display: none;
}

.upload-zone.drag-over,
.upload-zone:hover {
  background: rgba(34, 211, 238, 0.06);
  border-color: rgba(34, 211, 238, 0.6);
  box-shadow: inset 0 0 0 1px rgba(34, 211, 238, 0.15);
}

.upload-empty {
  min-height: 116px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  color: #64748b;
  font-size: 13px;
}

.upload-plus {
  width: 28px;
  height: 28px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: linear-gradient(135deg, #22d3ee, #a78bfa);
  color: #0b1220;
  font-weight: 800;
}

.file-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 12px;
}

.file-item {
  display: grid;
  grid-template-columns: auto minmax(0, 1fr) auto;
  align-items: center;
  gap: 10px;
  padding: 8px 10px;
  border-radius: 8px;
  background: rgba(11, 18, 32, 0.85);
  border: 1px solid #1e293b;
}

.file-type {
  min-width: 42px;
  text-align: center;
  border-radius: 6px;
  background: rgba(34, 211, 238, 0.1);
  color: #22d3ee;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 10px;
  font-weight: 800;
  letter-spacing: 0.08em;
  padding: 3px 5px;
}

.file-name {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #cbd5e1;
  font-size: 13px;
}

.remove-file {
  border: 0;
  background: transparent;
  color: #f87171;
  cursor: pointer;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.06em;
}

.notice-text,
.error-text {
  margin: 12px 0 0;
  font-size: 12px;
  line-height: 1.5;
}

.notice-text {
  color: #34d399;
}

.error-text {
  color: #f87171;
}

.launch-button {
  width: 100%;
  min-height: 56px;
  margin-top: 22px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border: 0;
  border-radius: 12px;
  background: linear-gradient(120deg, #22d3ee 0%, #6366f1 50%, #a78bfa 100%);
  color: #0b1220;
  cursor: pointer;
  padding: 0 20px;
  font-size: 15px;
  font-weight: 800;
  letter-spacing: 0.04em;
  position: relative;
  overflow: hidden;
  box-shadow: 0 18px 40px -16px rgba(34, 211, 238, 0.55);
  transition: transform 0.18s ease, box-shadow 0.18s ease;
}

.launch-button::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(120deg, transparent 30%, rgba(255, 255, 255, 0.18) 50%, transparent 70%);
  transform: translateX(-120%);
  transition: transform 0.6s ease;
}

.launch-button:hover:not(:disabled)::after {
  transform: translateX(120%);
}

.launch-button:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 24px 50px -16px rgba(34, 211, 238, 0.7);
}

.launch-button:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

.launch-label {
  display: flex;
  align-items: center;
  gap: 14px;
}

.launch-tag {
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  font-weight: 800;
  letter-spacing: 0.18em;
  padding: 4px 8px;
  border-radius: 6px;
  background: rgba(11, 18, 32, 0.18);
  color: #0b1220;
}

.launch-arrow {
  font-size: 28px;
  line-height: 1;
  font-weight: 700;
}

.bottom-bar {
  margin-top: 28px;
  padding: 18px 32px;
  border-top: 1px solid #1e293b;
  display: flex;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
  font-family: 'JetBrains Mono', ui-monospace, monospace;
  font-size: 11px;
  letter-spacing: 0.14em;
  color: #64748b;
  text-transform: uppercase;
  background: rgba(11, 18, 32, 0.7);
}

.bottom-bar-divider {
  width: 1px;
  height: 14px;
  background: #1e293b;
}

@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(0.85); }
}

@media (max-width: 1180px) {
  .hero,
  .generator-layout {
    grid-template-columns: 1fr;
  }

  .composer-panel {
    position: static;
  }
}

@media (max-width: 760px) {
  .topbar {
    height: auto;
    padding: 14px 18px;
    align-items: flex-start;
    gap: 14px;
    flex-direction: column;
  }

  .topbar-meta {
    flex-wrap: wrap;
    gap: 10px;
  }

  .workspace {
    width: min(100% - 28px, 1440px);
    padding-top: 24px;
  }

  .hero {
    padding: 22px;
  }

  .hero-copy h1 {
    font-size: 34px;
  }

  .hero-stats {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .seed-groups {
    grid-template-columns: 1fr;
  }

  .selector-panel,
  .composer-panel {
    padding: 18px;
  }

  .panel-heading,
  .upload-copy {
    flex-direction: column;
    align-items: flex-start;
  }

  .seed-toolbar {
    grid-template-columns: 1fr;
    align-items: stretch;
  }

  .action-row {
    flex-direction: column;
  }

  .secondary-button,
  .ghost-button {
    width: 100%;
  }

  .bottom-bar {
    padding: 14px 18px;
  }
}
</style>
