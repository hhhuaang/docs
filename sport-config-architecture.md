# 运动项目配置架构文档

## 概述

为了提高系统的可扩展性和可维护性，我们将运动项目的成绩计算规则和评分标准从服务类中分离到独立的配置文件中。这种设计使得新增运动项目、修改评分标准变得更加简单和灵活。

## 架构设计

### 1. 配置文件结构

```
src/util/sport-configs.ts  # 主配置文件
```

### 2. 核心接口

#### SportStandard 接口
```typescript
interface SportStandard {
  excellent: number;  // 优秀标准
  good: number;      // 良好标准
  pass: number;      // 合格标准
  unit: string;      // 单位
  scoreType: 'higher_better' | 'lower_better';  // 成绩类型
}
```

#### SportConfig 接口
```typescript
interface SportConfig {
  identifiers: string[];     // 运动类型标识符列表
  smartPeCode: string;      // 智体PE项目代码
  scoreExtraction: {        // 成绩提取配置
    fields: string[];       // 可能的成绩字段名
    unit: string;          // 单位
  };
  standards: GradeGenderStandards;  // 评价标准
  enabled: boolean;         // 是否启用
  description?: string;     // 项目描述
}
```

### 3. 已配置的运动项目

| 项目名称 | 配置键 | 智体PE代码 | 支持年级 | 评价标准 |
|---------|--------|-----------|---------|---------|
| 立定跳远 | standing_long_jump | jump | 6-9年级 | 国家标准 |
| 仰卧起坐 | sit_ups | situp | 3-9年级 | 国家标准 |
| 引体向上 | pull_ups | chin | 6-9年级 | 国家标准 |
| 1分钟跳绳 | rope_skipping | ropeskip | 3-9年级 | 参考标准 |
| 50米跑 | sprint | sprint | 6-9年级 | 国家标准 |
| 实心球 | solid_ball | solidball | 6-9年级 | 参考标准 |
| 开合跳 | jumping_jacks | jumping_jacks | 6-9年级 | 学校标准 |
| 800米跑 | run_800m | run800 | 7-9年级 | 国家标准 |
| 1000米跑 | run_1000m | run1000 | 7-9年级 | 国家标准 |

## 配置示例

### 立定跳远配置
```typescript
standing_long_jump: {
  identifiers: ['standing_long_jump', 'jump', 'tiaoyuan'],
  smartPeCode: 'jump',
  description: '立定跳远测试',
  scoreExtraction: {
    fields: ['distance', 'max_distance', 'jump_distance'],
    unit: '厘米'
  },
  standards: {
    7: { // 初一年级
      male: { excellent: 211, good: 195, pass: 155, unit: '厘米', scoreType: 'higher_better' },
      female: { excellent: 184, good: 170, pass: 140, unit: '厘米', scoreType: 'higher_better' }
    },
    // ... 其他年级
  },
  enabled: true
}
```

## 使用方法

### 1. 获取运动配置
```typescript
import { getSportConfig } from './sport-configs';

const config = getSportConfig('standing_long_jump');
if (config) {
  console.log('找到配置:', config.description);
}
```

### 2. 动态添加新项目
```typescript
import { addSportConfig, SportConfig } from './sport-configs';

const newConfig: SportConfig = {
  identifiers: ['new_sport'],
  smartPeCode: 'newsport',
  scoreExtraction: {
    fields: ['score'],
    unit: '个'
  },
  standards: {
    9: {
      male: { excellent: 100, good: 80, pass: 60, unit: '个', scoreType: 'higher_better' }
    }
  },
  enabled: true,
  description: '新运动项目'
};

addSportConfig('new_sport', newConfig);
```

### 3. 更新现有配置
```typescript
import { updateSportConfig } from './sport-configs';

updateSportConfig('standing_long_jump', {
  description: '更新后的立定跳远测试',
  enabled: false  // 临时禁用
});
```

## 核心功能函数

### 1. 获取配置
- `getSportConfig(sportType: string)` - 根据运动类型获取配置
- `getSportConfigBySmartPeCode(code: string)` - 根据智体PE代码获取配置
- `getEnabledSportConfigs()` - 获取所有启用的配置

### 2. 配置管理
- `addSportConfig(key: string, config: SportConfig)` - 添加新配置
- `updateSportConfig(key: string, updates: Partial<SportConfig>)` - 更新配置
- `toggleSportConfig(key: string, enabled: boolean)` - 启用/禁用配置

## 评分算法

### 1. 成绩类型
- **higher_better**: 成绩越高越好（如跳远、跳绳）
- **lower_better**: 成绩越低越好（如跑步时间）

### 2. 评价等级
- **优秀**: 90-100分
- **良好**: 80-89分
- **合格**: 60-79分
- **不合格**: <60分

### 3. 分数计算
```typescript
// 越高越好的项目
if (score >= standard.excellent) return 100;
if (score >= standard.good) return 85;
if (score >= standard.pass) return 60;
// 不合格区间线性插值
return Math.max(0, Math.min(59, (score / standard.pass) * 59));

// 越低越好的项目（如时间）
if (score <= standard.excellent) return 100;
if (score <= standard.good) return 85;
if (score <= standard.pass) return 60;
// 不合格区间：时间越长分数越低
```

## 扩展指南

### 新增运动项目步骤

1. **定义配置对象**
```typescript
const newSportConfig: SportConfig = {
  identifiers: ['sport_name', 'alias1', 'alias2'],
  smartPeCode: 'smart_pe_code',
  scoreExtraction: {
    fields: ['score_field1', 'score_field2'],
    unit: '单位'
  },
  standards: {
    // 按年级性别定义标准
  },
  enabled: true,
  description: '项目描述'
};
```

2. **添加到配置**
```typescript
// 在 sport-configs.ts 中的 sportConfigs 对象中添加
sport_name: newSportConfig
```

3. **测试验证**
```typescript
const config = getSportConfig('sport_name');
console.log('配置是否正确:', config !== null);
```

### 修改评分标准

直接修改 `sport-configs.ts` 文件中对应项目的 `standards` 字段即可，支持：
- 按年级分组（3-12年级）
- 按性别分组（male/female）
- 混合分组（某些项目男女标准相同）

## 优势特点

1. **高度可扩展**: 新增项目只需添加配置，无需修改业务逻辑
2. **标准化接口**: 所有项目遵循统一的配置格式
3. **灵活映射**: 支持多个标识符映射到同一配置
4. **动态管理**: 支持运行时添加、更新、启用/禁用配置
5. **类型安全**: 完整的TypeScript类型定义
6. **易于维护**: 配置与业务逻辑分离，便于维护和测试

这种架构设计确保了系统的可扩展性，使得新增运动项目变得简单高效，同时保持了代码的清晰性和可维护性。 