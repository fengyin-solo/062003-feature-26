<template>
  <div class="trainee-card card" :class="statusClass">
    <div class="card-top">
      <h4>{{ trainee.name }}</h4>
      <span class="badge" :class="trainee.status">{{ statusLabel }}</span>
    </div>

    <div class="bars">
      <div class="bar-row">
        <span>疲劳</span>
        <div class="bar"><div class="fill fatigue" :style="{ width: trainee.fatigue + '%' }"></div></div>
        <span>{{ trainee.fatigue }}</span>
      </div>
      <div class="bar-row">
        <span>压力</span>
        <div class="bar"><div class="fill stress" :style="{ width: trainee.stress + '%' }"></div></div>
        <span>{{ trainee.stress }}</span>
      </div>
    </div>

    <div class="stats-grid">
      <div v-for="key in statKeys" :key="key" class="stat-cell">
        <span class="stat-label">{{ statLabels[key] }}</span>
        <span class="stat-val">
          {{ trainee.stats[key] }}
          <span v-if="growthTrend.stats[key]" class="trend-icon" :class="growthTrend.stats[key]">
            {{ trendIcons[growthTrend.stats[key]] }}
          </span>
        </span>
      </div>
    </div>

    <div v-if="score !== null" class="score">
      综合评分 <strong>{{ score }}</strong>
      <span v-if="trainee.status === 'trainee'" class="debut-hint">
        {{ score >= debutThreshold ? '✓ 可出道' : `需 ${debutThreshold}` }}
      </span>
    </div>

    <div v-if="trainee.illnessDays > 0" class="illness">🤒 休养中 ({{ trainee.illnessDays }}天)</div>
    <div v-if="trainee.fans > 0" class="fans">个人粉丝 {{ trainee.fans.toLocaleString() }}</div>

    <div class="card-section growth-section">
      <div class="section-title">
        <span>📈 成长趋势</span>
        <span class="trend-badge" :class="growthTrend.overall">{{ trendLabels[growthTrend.overall] }}</span>
      </div>
      <div class="trend-bars">
        <div v-for="key in statKeys" :key="key" class="trend-row">
          <span class="trend-label">{{ statLabels[key] }}</span>
          <span class="trend-change" :class="getChangeClass(growthTrend.changes[key])">
            {{ formatChange(growthTrend.changes[key]) }}
          </span>
        </div>
      </div>
    </div>

    <div class="card-section stress-section">
      <div class="section-title">
        <span>💢 压力来源</span>
      </div>
      <div class="stress-list">
        <div v-for="(src, idx) in stressSources" :key="idx" class="stress-item" :class="src.type">
          <span class="stress-label">{{ src.label }}</span>
          <span class="stress-desc">{{ src.desc }}</span>
        </div>
      </div>
    </div>

    <div class="card-section suggestion-section">
      <div class="section-title">
        <span>💡 培养建议</span>
      </div>
      <div class="suggestion-list">
        <div v-for="(sug, idx) in suggestions" :key="idx" class="suggestion-item" :class="sug.priority">
          <span class="suggestion-dot"></span>
          <div class="suggestion-content">
            <span class="suggestion-label">{{ sug.label }}</span>
            <span class="suggestion-desc">{{ sug.desc }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { GAME_CONFIG } from '../config/gameConfig'
import {
  getGrowthTrend,
  getStressSources,
  getTrainingSuggestions,
} from '../utils/gameLogic'

const props = defineProps({
  trainee: Object,
  score: { type: Number, default: null },
  relationships: { type: Object, default: null },
  allTrainees: { type: Array, default: null },
})

const statKeys = GAME_CONFIG.stats
const statLabels = GAME_CONFIG.statLabels
const debutThreshold = GAME_CONFIG.rating.debutScoreThreshold

const trendIcons = {
  up: '↑',
  down: '↓',
  stable: '→',
}

const trendLabels = {
  fast: '快速成长',
  up: '稳步提升',
  stable: '平稳',
  down: '有所下滑',
}

const statusLabel = computed(() => {
  const map = { trainee: '练习生', debuted: '已出道', left: '已离开' }
  return map[props.trainee.status] || props.trainee.status
})

const statusClass = computed(() => ({
  debuted: props.trainee.status === 'debuted',
  left: props.trainee.status === 'left',
  ill: props.trainee.illnessDays > 0,
}))

const growthTrend = computed(() => getGrowthTrend(props.trainee))

const stressSources = computed(() => {
  return getStressSources(props.trainee, props.relationships, props.allTrainees)
})

const suggestions = computed(() => getTrainingSuggestions(props.trainee))

function formatChange(val) {
  if (val === undefined || val === null) return '-'
  if (val > 0) return `+${val}`
  return `${val}`
}

function getChangeClass(val) {
  if (val === undefined || val === null) return 'neutral'
  if (val > 0) return 'positive'
  if (val < 0) return 'negative'
  return 'neutral'
}
</script>

<style scoped>
.trainee-card {
  padding: 1rem;
  transition: border-color 0.2s;
}

.trainee-card.debuted { border-color: var(--accent); }
.trainee-card.left { opacity: 0.5; }
.trainee-card.ill { border-color: var(--warning); }

.card-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.card-top h4 { font-size: 1rem; }

.badge {
  font-size: 0.7rem;
  padding: 0.15rem 0.5rem;
  border-radius: 999px;
  background: var(--bg-secondary);
}

.badge.debuted { background: var(--accent-soft); color: var(--accent); }
.badge.left { background: var(--danger-soft); color: var(--danger); }

.bars { margin-bottom: 0.75rem; }

.bar-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.75rem;
  margin-bottom: 0.25rem;
}

.bar-row span:first-child { width: 28px; color: var(--text-muted); }
.bar-row span:last-child { width: 24px; text-align: right; }

.bar {
  flex: 1;
  height: 6px;
  background: var(--bg-secondary);
  border-radius: 3px;
  overflow: hidden;
}

.fill { height: 100%; border-radius: 3px; transition: width 0.3s; }
.fill.fatigue { background: var(--warning); }
.fill.stress { background: var(--danger); }

.stats-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 0.25rem;
  text-align: center;
}

.stat-cell {
  background: var(--bg-secondary);
  border-radius: 6px;
  padding: 0.3rem 0.1rem;
}

.stat-label { display: block; font-size: 0.65rem; color: var(--text-muted); }
.stat-val { font-weight: 700; font-size: 0.85rem; }

.score {
  margin-top: 0.5rem;
  font-size: 0.85rem;
  color: var(--text-secondary);
}

.debut-hint {
  margin-left: 0.5rem;
  font-size: 0.75rem;
  color: var(--accent);
}

.illness, .fans {
  margin-top: 0.35rem;
  font-size: 0.8rem;
  color: var(--warning);
}

.trend-icon {
  font-size: 0.65rem;
  margin-left: 2px;
}
.trend-icon.up { color: var(--success); }
.trend-icon.down { color: var(--danger); }
.trend-icon.stable { color: var(--text-muted); }

.card-section {
  margin-top: 0.75rem;
  padding-top: 0.75rem;
  border-top: 1px dashed var(--border);
}

.section-title {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--text-secondary);
  margin-bottom: 0.5rem;
}

.trend-badge {
  font-size: 0.65rem;
  padding: 0.1rem 0.4rem;
  border-radius: 999px;
  font-weight: 500;
}
.trend-badge.fast { background: var(--success-soft); color: var(--success); }
.trend-badge.up { background: var(--accent-soft); color: var(--accent); }
.trend-badge.stable { background: var(--bg-secondary); color: var(--text-muted); }
.trend-badge.down { background: var(--danger-soft); color: var(--danger); }

.trend-bars {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 0.25rem;
  text-align: center;
}

.trend-row {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
}

.trend-label {
  font-size: 0.6rem;
  color: var(--text-muted);
}

.trend-change {
  font-size: 0.7rem;
  font-weight: 600;
}
.trend-change.positive { color: var(--success); }
.trend-change.negative { color: var(--danger); }
.trend-change.neutral { color: var(--text-muted); }

.stress-list {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.stress-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.3rem 0.5rem;
  border-radius: 6px;
  background: var(--bg-secondary);
  font-size: 0.7rem;
}

.stress-label { font-weight: 600; }
.stress-desc { color: var(--text-muted); font-size: 0.65rem; }

.stress-item.critical {
  background: var(--danger-soft);
  border-left: 3px solid var(--danger);
}
.stress-item.critical .stress-label { color: var(--danger); }

.stress-item.high {
  background: var(--warning-soft);
  border-left: 3px solid var(--warning);
}
.stress-item.high .stress-label { color: var(--warning-dark); }

.stress-item.fatigue .stress-label,
.stress-item.competition .stress-label,
.stress-item.illness .stress-label {
  color: var(--text-primary);
}

.stress-item.good {
  background: var(--success-soft);
  border-left: 3px solid var(--success);
}
.stress-item.good .stress-label { color: var(--success); }

.suggestion-list {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.suggestion-item {
  display: flex;
  align-items: flex-start;
  gap: 0.5rem;
  padding: 0.4rem 0.5rem;
  border-radius: 6px;
  background: var(--bg-secondary);
}

.suggestion-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  margin-top: 0.3rem;
  flex-shrink: 0;
}

.suggestion-item.high .suggestion-dot { background: var(--danger); }
.suggestion-item.medium .suggestion-dot { background: var(--warning); }
.suggestion-item.low .suggestion-dot { background: var(--text-muted); }

.suggestion-content {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.suggestion-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text-primary);
}

.suggestion-desc {
  font-size: 0.65rem;
  color: var(--text-muted);
  line-height: 1.3;
}
</style>
