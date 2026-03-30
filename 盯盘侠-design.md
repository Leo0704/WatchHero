# 盯盘侠 - 加密货币策略监控应用设计

## 概述

**应用名称**: 盯盘侠

## 设计原则

1. **前端驱动后端**: 先用 mock 数据完成 UI/交互，后端按前端需要实现
2. **YAGNI**: MVP 只实现核心功能，不做过度设计
3. **可靠性优先**: 监控任务自动恢复、WebSocket 断线自动切换轮询
4. **用户可控**: 所有参数用户可自定义，系统提供智能默认值
5. **本地优先**: 所有数据存储本地，不依赖云服务

**技术栈**: Electron + Next.js + 本地 Node.js 服务（ccxt）

**开发模式**: 前端先行，mock 数据跑通完整流程，后端按前端需要实现

**架构说明**:
- Electron 提供跨平台桌面能力和本地数据存储
- Next.js (App Router) 提供 UI 组件和状态管理，虽有 SSR 冗余但与 Electron 集成成熟
- 替代方案（如 electron-vite + React Router）更轻量，但 Next.js 的组件生态和 Zustand 集成成熟
- 若未来需要 SSR（如 Web 版本），Next.js 可直接迁移

**目标平台**: macOS + Windows（跨平台，Electron 实现）

---

## 核心功能模块

### 1. 策略构建器

**流程**: 选指标 → 选预设条件 → 选参数 → 选时间框架 → 选监测对象

**指标支持**:

**趋势类**:

- **EMA（指数移动平均线）**：短期 EMA 上穿长期 EMA（金叉）、短期 EMA 下穿长期 EMA（死叉）、价格上穿 EMA、价格下穿 EMA、EMA 多头排列、EMA 空头排列、EMA 向上倾斜、EMA 向下倾斜、EMA 发散、EMA 收敛、EMA 穿越价格、EMA 与价格背离（共12个）
- **SMA（简单移动平均线）**：短期 SMA 上穿长期 SMA（金叉）、短期 SMA 下穿长期 SMA（死叉）、价格上穿 SMA、价格下穿 SMA、SMA 多头排列、SMA 空头排列、SMA 向上倾斜、SMA 向下倾斜、SMA 发散、SMA 收敛、SMA 穿越价格、SMA 与价格背离（共12个）
- **ADX（平均趋向指数）**：ADX > 阈值、ADX < 阈值、ADX 上升、ADX 下降、+DI 上穿 -DI、+DI 下穿 -DI、+DI > -DI、+DI < -DI、ADX 与价格背离（共9个）
- **Parabolic SAR（抛物线转向指标）**：SAR 在价格下方（多头）、SAR 在价格上方（空头）、SAR 翻转（趋势反转）、SAR 连续在价格下方、SAR 连续在价格上方（共5个）
- **Ichimoku Cloud（一目均衡表）**：转折线上穿基准线、转折线下穿基准线、价格上穿云层、价格下穿云层、价格在云层上方、价格在云层下方、价格在云层内部、延迟线在价格上方、延迟线在价格下方、延迟线上穿价格、延迟线下穿价格、云层由窄变宽、云层由宽变窄、基准线倾斜向上、基准线倾斜向下（共16个）

**动量类**:

- **RSI（相对强弱指数）**：RSI > 70（超买）、RSI < 30（超卖）、RSI > 80（极端超买）、RSI < 20（极端超卖）、RSI 上穿 50（中轴）、RSI 下穿 50（中轴）、RSI 创新高、RSI 创新低、RSI 底背离、RSI 顶背离、RSI 趋势线突破、RSI 穿越阈值（共12个）
- **Stochastic（随机指标）**：K 上穿 D（金叉）、K 下穿 D（死叉）、K > 80（超买区域）、K < 20（超卖区域）、K > D 且两者都在超买区、K < D 且两者都在超卖区、K 穿越 50 中轴、K 创新高、K 创新低、K 底背离、K 顶背离、Stochastic 趋势线突破（共12个）
- **MACD**：DIF 上穿 DEA（金叉）、DIF 下穿 DEA（死叉）、DIF 在零轴上方、DIF 在零轴下方、DIF 上穿零轴、DIF 下穿零轴、MACD 柱由负转正、MACD 柱由正转负、MACD 柱收缩、MACD 柱扩张、MACD 柱创N周期新高、MACD 柱创N周期新低、DIF 与价格底背离、DIF 与价格顶背离（共14个）
- **CCI（顺势指标）**：CCI > +100（强势）、CCI < -100（弱势）、CCI 上穿 +100、CCI 下穿 -100、CCI 穿越零轴向上、CCI 穿越零轴向下、CCI 创N周期新高、CCI 创N周期新低（共8个）
- **ROC（变动率指标）**：ROC > 0（上涨动能）、ROC < 0（下跌动能）、ROC 上穿 0、ROC 下穿 0、ROC 穿越均值、ROC 创N周期新高、ROC 创N周期新低、ROC 与价格背离（共8个）

**波动类**:

- **Bollinger Bands（布林带）**：价格上穿上轨、价格下穿下轨、价格触及上轨、价格触及下轨、价格上穿中轨、价格下穿中轨、价格在中轨上方、价格在中轨下方、带宽收缩（布林带收口）、带宽扩张（布林带开口）、带宽创N周期新窄、带宽创N周期新宽、布林带挤压启动、中轨向上倾斜、中轨向下倾斜（共15个）
- **ATR（平均真实波幅）**：ATR 突破均值（波动加剧）、ATR 创N周期新高、ATR 创N周期新低、ATR 与价格背离、ATR 连续上升、ATR 连续下降、ATR 穿越均值向上、ATR 穿越均值向下（共8个）
- **Keltner Channel（肯特纳通道）**：价格上穿上轨、价格下穿下轨、价格上穿中轨、价格下穿中轨、价格在中轨上方、价格在中轨下方、通道向上倾斜、通道向下倾斜、通道收缩、通道扩张（共10个）
- **Donchian Channel（唐奇安通道）**：价格上穿上轨（突破N周期高点）、价格下穿下轨（跌破N周期低点）、价格触及中轨、通道向上突破、通道向下突破、通道宽度收缩、通道宽度扩张、价格在通道内震荡（共8个）
- **Standard Deviation（标准差）**：波动率突破均值、波动率创N周期新高、波动率创N周期新低、价格偏离达到N个标准差、波动率连续上升、波动率连续下降（共6个）

**成交量类**:

- **OBV（能量潮）**：OBV 创N周期新高、OBV 创N周期新低、OBV 与价格底背离、OBV 与价格顶背离、OBV 上穿均线、OBV 下穿均线、OBV 连续上升、OBV 连续下降（共8个）
- **VWAP（成交量加权平均价）**：价格上穿 VWAP（多头信号）、价格下穿 VWAP（空头信号）、价格在 VWAP 上方、价格在 VWAP 下方、VWAP 向上倾斜、VWAP 向下倾斜、VWAP 与价格背离、价格回撤至 VWAP（共8个）
- **ADL（累积/派发线）**：ADL 与价格底背离、ADL 与价格顶背离、ADL 上穿零轴、ADL 下穿零轴、ADL 创N周期新高、ADL 创N周期新低、ADL 连续上升、ADL 连续下降（共8个）
- **CMF（资金流量指标）**：CMF > 0（资金流入）、CMF < 0（资金流出）、CMF 上穿零轴、CMF 下穿零轴、CMF 穿越均值、CMF 创N周期新高、CMF 创N周期新低、CMF 与价格背离（共8个）
- **Volume（成交量）**：成交量突破N周期均值、成交量创N周期新高、成交量创N周期新低、成交量萎缩、成交量放大、成交量与价格底背离、成交量与价格顶背离、放量上涨、放量下跌、缩量盘整（共10个）

**支撑阻力类**:

- **Pivot Points（轴心点）**：价格上穿轴心点 P（看涨）、价格下穿轴心点 P（看跌）、价格上穿 R1、价格下穿 R1、价格上穿 R2、价格下穿 R2、价格上穿 S1、价格下穿 S1、价格在 R1 与 S1 之间震荡、价格突破 R2 或 S2（强趋势）（共10个）
- **Fibonacci（斐波那契回撤）**：价格触及 23.6% 回撤位（±容差）、价格触及 38.2% 回撤位（±容差）、价格触及 50% 回撤位（±容差）、价格触及 61.8% 回撤位（±容差）、价格触及 78.6% 回撤位（±容差）、价格突破 23.6%（延续至 38.2%）、价格突破 38.2%（延续至 50%）、价格突破 50%（延续至 61.8%）、价格突破 61.8%（延续至 78.6%）、价格突破 78.6%（延续至 100%）、价格反弹至 38.2% 后继续上涨、价格回撤至上一波段 61.8% 附近企稳（共12个，含容差机制）
- **价位标记**：价格触及价位、价格上穿价位、价格下穿价位、价格在价位上方、价格在价位下方、价格与价位形成背离（共6个）

**情绪类**:

- **Fear & Greed Index（情绪指数）**：Fear & Greed > 75（极度贪婪）、Fear & Greed < 25（极度恐惧）、Fear & Greed 由恐惧转贪婪、Fear & Greed 由贪婪转恐惧、Fear & Greed 突破均值、Fear & Greed 创N周期新高/新低（共6个，暂不支持，后续可接入第三方 API）
- **Funding Rate（资金费率）**：Funding Rate > 0（多头支付）、Funding Rate < 0（空头支付）、Funding Rate 突破正向阈值、Funding Rate 突破负向阈值、Funding Rate 由正转负、Funding Rate 由负转正、Funding Rate 创N周期新高、Funding Rate 创N周期新低（共8个）
- **Open Interest（未平仓合约）**：Open Interest 创N周期新高、Open Interest 创N周期新低、Open Interest 与价格底背离、Open Interest 与价格顶背离、Open Interest 上升 + 价格上升、Open Interest 上升 + 价格下跌、Open Interest 下降 + 价格上升、Open Interest 下降 + 价格下跌（共8个）
- **Long/Short Ratio（多空比）**：多空比 > 1（多头占优）、多空比 < 1（空头占优）、多空比突破均值、多空比创N周期新高、多空比创N周期新低、多空比与价格底背离、多空比与价格顶背离、多空比极端值预警（>3或<0.33）（共8个）
- **Liquidation（清算数据）**：清算金额突破N周期均值、多头清算集中爆发、空头清算集中爆发、清算金额创N周期新高、清算金额连续上升、多头清算 > 空头清算、空头清算 > 多头清算、清算密度突破阈值（共8个）

**波动率类**:

- **Historical Volatility（历史波动率）**：HV 突破均值、HV 创N周期新高、HV 创N周期新低、HV 与价格背离、HV 连续上升、HV 连续下降、HV 穿越阈值、HV 突破历史均值（共8个，本地计算）

**条件组合**: 支持嵌套 AND / OR

- 扁平条件组 + 嵌套条件组混合
- 嵌套深度无限制
- 每个条件组内支持多个条件
- 条件组结构示例（JSON 存储）:
  ```json
  {
    "id": "root",
    "type": "group",
    "logic": "AND",
    "children": [
      {
        "id": "leaf-1",
        "type": "leaf",
        "indicator": "RSI",
        "preset": "rsi_oversold",
        "parameters": { "period": 14, "threshold": 30 }
      },
      {
        "id": "group-1",
        "type": "group",
        "logic": "OR",
        "children": [
          {
            "id": "leaf-2",
            "type": "leaf",
            "indicator": "MACD",
            "preset": "macd_golden_cross",
            "parameters": { "fast": 12, "slow": 26, "signal": 9 }
          },
          {
            "id": "leaf-3",
            "type": "leaf",
            "indicator": "MACD",
            "preset": "macd_divergence_bottom",
            "parameters": { "fast": 12, "slow": 26, "signal": 9 }
          }
        ]
      }
    ]
  }
  ```

**策略条件数量上限**: 无硬性限制，建议不超过 20 个（性能考量）

**时间框架**: 5m、15m、30m、1h、4h、6h、12h、1d、1w（用户自选）

**监测对象选择**: 多选模式（搜索 + 热门榜 + 收藏列表）

**参数设置**: 智能推荐范围 + 用户可调整

**策略管理**:
- 名称必填（用户手动输入）
- 标签分类（用户自定义分组名称）
- 策略搜索
- 支持导入/导出
- 手动保存 + 离开页面提示保存 + 定期自动保存草稿（本地）
- 删除确认框（显示策略名称 + 关联监控任务数量）+ 关联监控任务同步停止（软删除）+ 30天内可从回收站恢复（恢复时任务保持 stopped 状态，需手动启动）

---

### 2. 监控面板

**币种数据源**:
- 自定义币种列表
- 币安官方合约热门榜（每 1 小时自动刷新）

**多任务管理**:
- 可同时运行多个监控任务（上限 50 个策略实例）
- 每个策略实例可绑定多个监测对象（币种）
- 任务创建时复制策略配置为独立快照，策略修改不影响已运行任务
- 支持暂停/恢复/编辑/删除
- 重启后自动恢复监控任务（仅限正常退出重启；异常崩溃后需用户确认是否恢复）

**检查频率**: 每根 K 线收线后检查（系统自动适配）

- 市场数据服务统一监听所有活跃任务订阅的时间框架
- 任意时间框架 K 线收线时，自动触发该时间框架下所有任务的信号检查
- 多任务跨不同时间框架时互不干扰，各自按自己的 K 线周期检查

**信号冷却**: 每个任务独立冷却时间（默认 5 分钟，用户可调）

**WebSocket 断线策略**: 指数退避 + 币安限流考虑
- 断线后自动切换 REST API 轮询
- 轮询间隔：5s → 10s → 30s → 最大 120s（指数退避）
- 重连成功自动恢复 WebSocket

**REST API 全局限流器**:
- 50 个任务同时断线不能发起 50 个独立请求
- 使用币安 `/fapi/v1/ticker/price` 批量接口，一次获取所有监控币种价格
- 全局请求队列：每 1 秒最多发送 10 个 REST 请求（币安限制约 1200 weight/分钟）
- 请求打包示例: `GET /fapi/v1/ticker/price?symbol=BTCUSDT&symbol=ETHUSDT&...`

```typescript
class GlobalRateLimiter {
  private queue: Array<() => void> = [];
  private requestsThisSecond = 0;
  private maxPerSecond = 10;
  private timer: NodeJS.Timeout | null = null;

  async schedule<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      const task = async () => {
        try {
          this.requestsThisSecond++;
          const result = await fn();
          resolve(result);
        } catch (e) {
          reject(e);
        } finally {
          // 1 秒后重置计数，并触发下一批处理
          setTimeout(() => {
            this.requestsThisSecond--;
            this.processQueue();
          }, 1000);
        }
      };

      this.queue.push(task);
      this.processQueue();
    });
  }

  private processQueue() {
    // 清除之前的定时器（防止重复触发）
    if (this.timer) clearTimeout(this.timer);

    // 消费队列直到达到上限
    while (this.queue.length > 0 && this.requestsThisSecond < this.maxPerSecond) {
      const next = this.queue.shift()!;
      next();
    }

    // 如果队列还有内容，设置定时器在计数器下降后继续处理
    if (this.queue.length > 0) {
      this.timer = setTimeout(() => {
        this.timer = null;
        this.processQueue();
      }, 100);
    }
  }
}
```

**WebSocket 订阅上限管理**:
- 币安 WebSocket 单连接最多 200 个 stream
- 订阅合并：同一币种多个策略只需订阅一次（按 symbol + timeframe 去重）
- 多连接策略：订阅数超过 150 时（留 buffer）自动创建新连接
- 动态负载均衡：新订阅按 round-robin 分配到各连接

```
订阅数计算:
活跃任务数 × 平均每任务币种数 × 时间框架数 = 总订阅数
例: 50 策略 × 3 币种 × 2 时间框架 = 300 stream（需 2 个 WebSocket 连接）
```

| 阈值 | 行为 |
|------|------|
| ≤ 150 | 单连接 |
| > 150 | 创建第二个连接，分配新订阅 |
| > 300 | 创建第三个连接，以此类推 |
| 单连接 stream > 190 | 触发新连接创建 |

**实时数据**:
- WebSocket 推送 + REST API 轮询备用（断线自动切换）
- 监控面板显示实时价格/涨跌幅
- 点击展开查看详细 OHLC、成交量、指标数值

**界面展示**:
- 列表显示每个任务状态（监控中/已暂停）
- 币种实时数据（价格、涨跌幅、指标数值）
- 最新触发信号实时显示

**即时告警**: 独立于策略的价格/指标告警，可设置多个条件

---

### 3. 回测模块

**币种选择**: 搜索 + 热门榜 + 收藏列表（与监控面板保持一致）

**数据源**: 只支持币安 USDT 永续合约（数据最全，主流）

**同时回测数量**: 默认最多 5 个，用户可调整（范围 1-20），超过 5 个时弹出性能警告

**时间范围**: 用户自定义起止日期

**K 线周期**: 用户自选（1m/5m/15m/30m/1h/4h/6h/12h/1d/1w）

**数据处理**:
- 自动从币安补全缺失数据（通过 ccxt 处理限流）
- 显示下载进度条 + 当前下载币种提示
- 数据复用（监控与回测共用同一份数据，指标优化预计算缓存共用同一份）

**手续费**: 用户可自定义费率

**K 线计算**: 收盘价触发信号，但止损/止盈检查使用 bar 内 high/low（模拟更真实的成交）

**仓位管理**:
- 固定百分比（默认每笔 10% 仓位，用户可调）
- 支持做多 / 做空双向（默认只做多）
- 每次信号开仓至目标仓位比例，不加仓不减仓

**连续信号处理**: 同一策略 + 同一币种，每次信号触发前须先有明确出场（止损/止盈/手动平仓），才能重新开仓。默认只第一次触发开仓，后续忽略。

**止损止盈**:
- 止损：按开仓价计算，固定百分比（默认 -2%，用户可调）。使用 bar 内 high/low 检查是否触发；触发时成交价按止损价（模拟日内极端价格穿透后成交）。
  - 做多持仓：bar.low <= 止损价 → 止损平仓（止损价 = entry_price * (1 - stop_loss_pct)）
  - 做空持仓：bar.high >= 止损价 → 止损平仓（止损价 = entry_price * (1 + stop_loss_pct)）
- 止盈：按开仓价计算，固定百分比（默认 +4%，用户可调）。使用 bar 内 high/low 检查是否触发；触发时成交价按止盈价（模拟日内极端价格穿透后成交）。
  - 做多持仓：bar.high >= 止盈价 → 止盈平仓（止盈价 = entry_price * (1 + take_profit_pct)）
  - 做空持仓：bar.low <= 止盈价 → 止盈平仓（止盈价 = entry_price * (1 - take_profit_pct)）
- 移动止损：止盈触发后启用，做多追踪持仓期内最高价，做空追踪最低价，回落 N% 出（默认 1%，用户可调）。成交价按触发价（做多：highest * (1 - trailing_stop_pct)；做空：lowest * (1 + trailing_stop_pct)）

**滑点**: 固定 0.05%（用户可调）

**交易方向**: 做多、做空、双向（用户自选）

**最优参数判定标准**: 用户自选（总收益率 / 年化收益率 / 夏普比率 / 最大回撤 / 盈亏比 × 胜率）

**回测结果指标**:
- 总收益率
- 年化收益率
- 最大回撤
- 胜率
- 盈亏比
- 交易次数
- 夏普比率

**交易明细**:
- 买入/卖出时间、价格、盈亏
- 手续费、滑点、持仓时长
- 策略名称、币种、信号类型

**回测报告总览**:
- 核心指标
- 权益曲线图（支持缩放、平移、时间范围高亮）
- 交易分布图
- 最大单笔盈利/亏损
- 最长连胜/连负次数

**完成后通知**: 应用内 toast 通知

**交互**:
- 一键保存最优参数到策略 + 确认框（是否同步更新当前监控任务中的策略参数）
- 重新回测/调整参数

---

### 4. 参数优化

**优化算法**: 网格搜索

**计算配置**:
- 最大并发数（默认 3，最高 10，用户可设置）
- 单个组合超时时间（默认 30 秒，用户可设置）
- 可随时暂停/取消

**数据处理**:
- 指标数值预计算 + 缓存（避免重复计算，加速网格搜索）
- 缓存表存储各指标在各参数组合下的数值，参数优化直接查表

**结果展示**:
- 所有参数组合表格（可排序/筛选）
- 2 参数热力图 + 3 参数分解展示
- 收益曲线对比图

**导出**: JSON / CSV

**完成后通知**: 应用内 toast 通知

---

### 5. Telegram 通知

**配置**:
- Bot Token + Chat ID（用户手动输入，加密本地存储）
- 加密方案：AES-256-GCM 加密后存 SQLite，密钥存储在平台安全存储（macOS Keychain / Windows DPAPI）
- 手动测试按钮验证连接
- 首次使用提示引导（不强制）

**推送内容**:
- 币种、信号类型、价格、时间
- 策略名称、当前指标数值

**错误通知**:
- 应用内弹窗 + Telegram 推送
- 发送失败后静默重试 3 次（5s → 10s → 20s 指数退避），3 次失败后应用内提示「可重新推送」，不无限重试

---

### 6. 数据存储

**存储位置**: `app.getPath('userData')`（Electron 跨平台 API，macOS 为 `~/Library/Application Support/盯盘侠/`，Windows 为 `%APPDATA%/盯盘侠/`）

**数据库**: SQLite 本地数据库

**SQLite 性能优化**:
```typescript
// database.ts 初始化配置
db.pragma('journal_mode = WAL');     // WAL 模式：写操作不阻塞读
db.pragma('synchronous = NORMAL');   // 平衡性能与安全性
db.pragma('temp_store = MEMORY');    // 临时表存内存
db.pragma('mmap_size = 268435456');  // 内存映射 256MB，加速读取
```

**高频写入队列**:
- WebSocket 实时 Tick 数据不直接写库，先入内存队列
- 队列每 1 秒或积累 100 条后批量写入（Batch Insert）
- 防止高频写入触发 SQLITE_BUSY 锁表

```typescript
class WriteQueue {
  private queue: TickData[] = [];
  private flushInterval = 1000; // ms

  add(tick: TickData) {
    this.queue.push(tick);
    if (this.queue.length >= 100) this.flush();
  }

  async flush() {
    if (this.queue.length === 0) return;
    const batch = this.queue.splice(0);
    db.transaction(() => {
      batch.forEach(t => db.insert('klines', t));
    });
  }
}
```

**存储内容**:
- 策略配置（命名、标签、条件，JSON 格式存储）
- 监控任务快照（策略配置复制，任务独立运行）
- 信号触发记录
- 历史告警记录
- 回测历史
- 策略回收站（删除后 30 天内可恢复）

**K 线历史数据表**: 统一大表 + 复合索引（避免表数量爆炸）
```
CREATE TABLE klines (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  symbol TEXT NOT NULL,                    -- "BTCUSDT"
  timeframe TEXT NOT NULL,                 -- "1h"
  timestamp INTEGER NOT NULL,               -- Unix ms (PK 的一部分)
  open REAL NOT NULL,
  high REAL NOT NULL,
  low REAL NOT NULL,
  close REAL NOT NULL,
  volume REAL NOT NULL,
  created_at TEXT DEFAULT (datetime('now')),
  UNIQUE(symbol, timeframe, timestamp)
);

CREATE INDEX idx_klines_symbol_timeframe ON klines(symbol, timeframe, timestamp DESC);
CREATE INDEX idx_klines_timestamp ON klines(timestamp DESC);
```
- 优点: 100+ 交易对 x 9 时间框架 = 单表查询，加索引即可
- 增量更新: 记录每个 symbol+timeframe 的 `last_timestamp`
- 默认下载范围: 最近 6 个月（首次下载，为回测/优化提供足够样本）
- 首次下载全量，后续启动增量更新；用户可手动触发全量重刷
- 并发下载: 最多 3 个 symbol 并行，每个 symbol 按时间框架串行下载

**历史数据保留**: 用户自定义（默认 3 个月）+ 手动清理 + 定时自动清理
- 说明: 下载 6 个月但只保留 3 个月是正常的（回测需要近期数据，存储管理节省空间）

**指标缓存**: 内存 LRU 缓存（不回写磁盘，进程重启后重新计算）

```
内存缓存结构:
Map<cacheKey, {
  values: Float64Array,  // 指标值数组
  params: object,         // 参数快照
  lastAccess: number      // LRU 追踪
}>

cacheKey = SHA256(`${symbol}:${timeframe}:${indicatorType}:${paramsHash}`)
```

- 回测/参数优化时，同一 symbol + timeframe + indicator + 参数组合的指标值只计算一次
- 内存缓存上限: 200MB（自动 LRU 驱逐）
- 监控任务使用后即释放，进程重启后重新计算
- 不建议持久化到 SQLite（千万级数据量会导致 I/O 瓶颈）

**数据备份**: 每周自动备份 SQLite，保留最近 4 周；用户可手动导出

**导出**: CSV / JSON（信号记录、回测历史）

---

### 7. 其他

**界面**: 暗色模式（前端决定）

**应用窗口**: 单窗口 + 标签页切换（跨平台桌面应用常见模式）

**新手引导**: 可跳过的引导流程

**自动更新**:
- 使用 `electron-updater` + GitHub Releases
- 检查频率：应用启动时检查 + 每 6 小时轮询
- 更新包格式：`.zip`（便于差分更新），通过 `nsis-diff` 或 `zip-diff` 实现增量更新
- 更新流程：下载 → 验证签名 → 解压 → 下次启动应用时自动替换（`electron-updater` 自动处理）
- 更新失败回滚：启动时检测上一版本备份（`{userData}/backups/`），若更新后应用损坏自动回滚

**异常恢复**: 检测到异常退出时，提示用户是否恢复状态

**日志**: 错误日志记录（普通用户无需关注）

**声音**: 信号触发时系统提示音

**主题**: 单主题（前端决定）

---

### 8. 错误处理与日志

**错误码体系**:

| 错误码 | 范围 | 说明 |
|--------|------|------|
| E1xx | 网络 | E101 连接超时, E102 DNS 解析失败, E103 SSL 错误, E104 连接被拒绝 |
| E2xx | 数据 | E201 K线数据缺失, E202 数据格式异常, E203 数据过期 |
| E3xx | 策略 | E301 条件验证失败, E302 指标计算错误, E303 参数越界 |
| E4xx | 推送 | E401 Telegram 推送失败, E402 Bot Token 无效, E403 Chat ID 无效 |
| E5xx | 回测 | E501 回测数据不足, E502 回测计算异常, E503 参数优化超时 |
| E6xx | 系统 | E601 数据库损坏, E602 内存不足, E603 磁盘空间不足, E604 应用崩溃 |

**日志分级**: Warning + Error

**日志查看**: 分级查看（Error/Warning 分开）+ 日志轮转

**日志保留**: 30 天 + 100MB 双重限制

**日志位置**: `{userData}/logs/`

**渲染进程错误处理**:
```typescript
// React Error Boundary 组件
class ErrorBoundary extends Component {
  state = { hasError: boolean; error: Error | null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, info) {
    logError('E604', error.message, { componentStack: info.componentStack });
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

**磁盘空间预检查**:
- 全量下载 K 线前检查剩余磁盘空间
- 最低要求: 剩余空间 > 预估所需空间 × 1.5
- 空间不足时提示用户清理或调整保留天数

**网络错误展示**: 错误码 + 中文描述 + 操作建议

**WebSocket 断线**: 显示重连状态 + 重连次数 + 最终结果

**API 请求超时**: toast 提示"请求超时，正在重试..."（静默重试 3 次，全部失败后弹出确认框）

**API 限流**: 自动排队处理，等待超过 30 秒时显示倒计时提示

**数据下载失败**: 显示具体失败原因 + 提供手动重试按钮

**回测计算出错**: 显示错误码 + 错误步骤 + 错误数据行 + 可选跳过继续计算

**参数优化出错**: 自动跳过出错组合 + 记录错误 + 完成全部后显示汇总（含出错数量）

**Telegram 推送失败**: 静默重试 3 次（5s → 10s → 20s 指数退避），3 次失败后应用内提示「可重新推送」

**策略验证失败**: 保存时自动验证 + 失败提示具体错误 + 提供修复建议

**告警触发失败**: 应用内提示 + 自动重试 + 记录重试结果

**危险操作**: 二次确认 + 30 秒内可撤销（仅限删除策略、删除监控任务、删除监控对象）

**系统托盘**: 最小化到托盘持续运行
- 单击托盘图标：显示菜单（暂停所有任务/恢复所有任务/打开窗口/退出）
- 双击托盘图标：恢复主窗口

**应用崩溃**: 自动生成错误报告 + 下次启动时询问是否查看/发送

**内存不足**: 提示用户关闭其他应用 + 自动降低并发数

**数据库损坏**: 自动尝试备份恢复 + 失败后提示 + 提供导出剩余数据和重置选项

---

### 9. 部署与打包

**打包目标**:
- macOS: `.dmg` 安装包（Apple Silicon + Intel 通用）
- Windows: `.exe` (NSIS installer)

**macOS 签名与公证**:
- 使用 Apple Developer ID 证书签名（`electron-builder` 的 `mac.sign` 配置）
- 签名后通过 `afterSign` hook 自动提交 Apple 公证（Notarization）
- 开发阶段使用 ad-hoc 签名（跳过公证），不适用于正式分发

**Windows 签名**:
- 使用代码签名证书（EV 或 OV 证书）进行签名（`electron-builder` 的 `win.sign` 配置）
- 正式分发必须签名（Windows SmartScreen 会拦截未签名应用）
- 开发阶段跳过签名

**electron-builder 关键配置**:
- `npmRebuild: false`（使用预编译的 native 模块，不重新编译）
- 移除 `node_modules` 中不必要的 `.ts`、`.md`、类型声明文件
- 最终安装包开启压缩

**安装包体积优化**:
- pnpm + Tree Shaking：确保生产构建时去除未使用代码
- 图表库按需引入（`lightweight-charts` 只 import 需要的模块）
- native 模块（`better-sqlite3`）使用 `@electron/rebuild` 预编译，避免运行时重复编译

**分发渠道**: GitHub Releases（托管更新包 + release notes）

---

### 10. 数据备份与恢复

**自动备份**:
- 触发时机：每周日凌晨 3:00（系统空闲时）
- 备份文件命名：`backup_YYYYMMDD_HHmmss.db`
- 存储路径：`{userData}/backups/`
- 保留策略：自动保留最近 4 周备份，超期自动删除；手动备份不自动清理

**备份内容**: 完整 SQLite 数据库文件（含策略、任务、信号、回测历史、K 线缓存、指标缓存）

**灾难恢复**:
- 设置页提供「从备份恢复」入口
- 用户选择备份文件后执行 `sqlite3 .restore` 恢复
- 恢复后自动重启应用并验证数据一致性
- 恢复失败时保留原数据库，提示用户手动处理

**数据迁移工具**:
- 设置页提供「导出全部数据」：导出为单个 JSON 文件（含策略、任务、设置、回测历史，不含 K 线缓存）
- 设置页提供「导入全部数据」：导入时校验 schema 版本，提示兼容性信息
- 导入导出文件命名：`盯盘侠数据_YYYYMMDD.json`

**应用版本升级（数据迁移）**:
- 启动时检测 `app_settings.db_version` 与代码期望版本是否一致
- 不一致时弹出确认框告知用户需要迁移，征得同意后执行迁移脚本
- 迁移失败时保留原数据库，提示用户手动处理或联系支持

---

### 11. 法律与合规

**用户协议与隐私政策**:
- 首次启动时展示，用户必须点击「同意」才能继续使用
- 协议内容要点：
  - 所有数据存储在用户本地设备，应用不收集任何个人信息
  - 应用不接入任何追踪服务（无埋点、无 analytics）
  - 市场数据来源于币安公开 API，数据归属权归币安所有
  - 历史回测结果仅供参考，不代表未来收益表现
  - 投资有风险，使用本应用造成的任何损失由用户自行承担

**风险提示**:
- 策略创建页、回测页、参数优化页底部添加固定提示语：「历史回测结果仅供参考，不代表未来收益。投资有风险，请谨慎决策。」

**数据来源声明**: 应用内关于页面注明「数据来源：币安」，界面上实时价格标注「数据来源：币安」

---

### 12. 第三方服务容灾

**币安 API**:
- 全部通过 ccxt 处理限流（自动排队 + 请求权重控制）
- WebSocket 断线：5s → 10s → 30s → 最大 120s 指数退避重连
- API 完全不可用时（连续重试失败），对应监控任务显示「数据源异常」状态并自动暂停，用户可手动重试恢复
- 暂不实现备用数据源（依赖单一数据源，简化复杂度）

**Telegram 推送**:
- 推送失败后静默重试 3 次（指数退避：5s → 10s → 20s）
- 3 次失败后应用内提示「Telegram 通知发送失败，可重新推送」
- 不无限重试，避免产生大量无效请求

---

### 13. 性能监控（轻量实现）

**MVP 指标采集**:
- 应用启动时间：记录从进程启动到首屏渲染完成的耗时
- 内存占用：定期采样 `performance.memory`（如有），内存超过 500MB 时写入 Warning 日志
- 回测计算耗时：回测完成后记录计算耗时，供用户查看

**生产环境**: 不采集详细性能数据（符合隐私优先原则），仅通过日志记录异常

**CI/开发阶段性能基准**: 通过注释 benchmark 代码实现，不做线上采集系统

---

### 14. 可访问性（基础保证）

**MVP 保证**:
- 所有交互元素可通过键盘完全操作（Tab 切换焦点，Enter 确认，Esc 取消）
- 所有表单元素和按钮有 `aria-label` 或 `aria-labelledby`
- 颜色对比度符合 WCAG AA 标准（4.5:1 文本，3:1 大文本）
- 屏幕阅读器（VoiceOver / NVDA）可读：图片有 alt 文本，图标按钮有标签

**详细实现要求**:
| 要求 | 实现方式 |
|------|---------|
| 图标按钮 | 所有 `<button>` 添加 `aria-label`（如"新建策略"、"删除"） |
| 颜色对比度 | 暗色模式下文字/背景对比度 ≥ 4.5:1 (WCAG AA) |
| 键盘导航 | 所有交互元素可通过 Tab 聚焦，Enter/Space 激活 |
| 焦点指示 | 聚焦元素显示 2px 蓝色轮廓 (`focus-visible`) |
| 状态通知 | 使用 `aria-live="polite"` 区域播报 Toast / 信号更新 |
| 条件树编辑器 | 键盘方向键导航节点，Tab 切换参数字段 |

**自动化测试**: `axe-core` 集成到 CI，每次 PR 自动运行可访问性检查

**高对比度主题**: v1.1 再实现，MVP 只提供默认暗色主题

---

### 15. 国际化（架构预留，v1.1 实现）

**架构**:
- 使用 `next-intl` 或 `react-intl`
- 翻译文件目录：`src/locales/{lang}/`（如 `src/locales/zh/`、`src/locales/en/`）
- 翻译 key 格式：`module.section.message`（如 `strategy.builder.save`）

**文本提取**: 通过 `i18next` 的 `react-i18next` 配合 `i18next-parser` 自动提取

**动态内容插值**: 使用 ICU MessageFormat（如 `你好，{name}，当前价格 {price}`）

**日期、数字格式化**: 统一使用 `Intl.DateTimeFormat` 和 `Intl.NumberFormat`，封装为 `src/lib/format.ts` 工具函数，全应用调用统一入口

**语言支持**: v1.0 只提供简体中文，v1.1 再支持英文及扩展

---

### 16. 测试与质量保证

**单元测试**:
- 框架：`Vitest`（与 Vite 集成，速度快）
- 覆盖范围：策略引擎条件树遍历、指标计算、回测引擎核心逻辑
- 测试数据：内联在测试文件中的静态 K 线数据（极端行情、边界条件）

**集成测试**:
- 测试数据库使用临时文件，每次测试前重建 schema
- 覆盖范围：数据库 CRUD、IPC handlers、策略 CRUD 全流程

**Mock 服务实现**:
- 环境变量 `NEXT_PUBLIC_USE_MOCK=true` 切换
- `createMockAPI()` 返回实现 `ElectronAPI` 接口的内存对象
- 网络延迟模拟：Mock 函数内 `setTimeout` 随机 100-500ms
- 错误场景模拟：`monitor-mock.ts` 提供 `setNetworkError()`、`setRateLimitError()` 方法用于测试
- Mock 与真实服务切换：零侵入，共享同一类型签名
- 注意: `NEXT_PUBLIC_` 前缀确保变量在构建时被替换，Mock 代码不会打入生产包
- CI 检查: 可通过 `grep -r "NEXT_PUBLIC_USE_MOCK" build/` 验证

**E2E 测试用例**（关键场景）:
1. 创建策略 → 启动监控 → 模拟价格触发信号 → 验证信号记录写入 → 验证 Telegram 通知发送
2. 回测配置 → 执行回测 → 验证结果指标计算正确
3. 参数优化 → 执行优化 → 验证进度回调 → 验证结果排序
4. 应用异常退出 → 重启 → 验证监控任务自动恢复（建议用集成测试替代，E2E 难以可靠模拟崩溃）
5. 删除策略 → 验证关联任务停止 → 验证回收站恢复
6. 回测引擎对照验证: 使用"买入持有"策略，与理论值对照（收益率 = 期末/期初 - 1）

**CI 流程**: GitHub Actions，Vitest 单元测试 + Playwright E2E 测试

---

## 模块依赖关系

```
┌─────────────────────────────────────────────────────────┐
│                     Electron Shell                      │
├─────────────────────────────────────────────────────────┤
│                      Next.js UI                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐    │
│  │ 策略    │ │ 监控    │ │ 回测    │ │ 参数优化    │    │
│  │ 构建器  │ │ 面板    │ │ 模块    │ │             │    │
│  └────┬────┘ └────┬────┘ └────┬────┘ └──────┬──────┘    │
│       │           │           │              │           │
│  ─────┴───────────┴───────────┴──────────────┴────────  │
│                      Mock API Layer                     │
│              (前端先行，mock数据跑通流程)                  │
└─────────────────────────────────────────────────────────┘

后端实现时（按需对接）:
┌─────────────────────────────────────────────────────────┐
│                   Local Node.js Service                  │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────┐    │
│  │ ccxt    │ │ 策略    │ │ 回测    │ │ Telegram    │    │
│  │ 数据源  │ │ 引擎    │ │ 引擎    │ │ 推送服务    │    │
│  └─────────┘ └─────────┘ └─────────┘ └─────────────┘    │
└─────────────────────────────────────────────────────────┘
```

---

## 技术选型

| 领域 | 选择 | 理由 |
|------|------|------|
| 状态管理 | Zustand | 轻量、TypeScript 友好、适合中小型 Electron 应用 |
| UI 组件库 | shadcn/ui + Tailwind CSS | 可定制、暗色模式原生支持、按需引入不打包多余代码 |
| 图表库 | lightweight-charts (TradingView) | 金融级 K 线图，性能好、交互丰富 |
| 数据库 | better-sqlite3 | 同步 API、性能优、Electron 生态首选 |
| Mock 层 | 内存 Mock 实现（实现 ElectronAPI 接口，不依赖 HTTP，前后端无缝切换 |
| IPC 通信 | Electron IPC + 类型化桥接 | type-safe 前后端通信 |
| 数据校验 | Zod | schema 校验，策略条件验证、API 参数校验 |
| 包管理 | pnpm | 速度快、磁盘占用少 |
| 测试框架 | Vitest（单元+集成）+ Playwright（E2E） | 快、TypeScript 原生支持、与 Vite 集成 |
| i18n | next-intl | App Router 原生支持、tree-shakable |
| 自动更新 | electron-updater | 与 GitHub Releases 无缝集成 |

---

## 项目结构

```
盯盘侠/
├── electron/                    # Electron 主进程
│   ├── main.ts                  # 入口，窗口创建、IPC 注册
│   ├── preload.ts               # preload 桥接，暴露安全 API
│   ├── ipc/                     # IPC handlers（按模块拆分）
│   │   ├── strategy.ts          # 策略 CRUD、导入导出
│   │   ├── monitor.ts           # 监控任务管理、信号推送
│   │   ├── backtest.ts          # 回测执行、结果查询
│   │   ├── optimization.ts      # 参数优化
│   │   └── settings.ts          # 应用设置、Telegram 配置
│   ├── services/                # 后端核心服务
│   │   ├── database.ts          # SQLite 初始化、迁移、连接管理
│   │   ├── strategy-engine.ts   # 策略评估引擎（条件树遍历，委托指标计算给 indicators-calc.ts）
│   │   ├── backtest-engine.ts   # 回测引擎（历史数据回放 + 交易模拟）
│   │   ├── optimization-engine.ts # 网格搜索 + 并发调度
│   │   ├── data-fetcher.ts      # ccxt 数据获取 + 增量更新
│   │   ├── websocket.ts         # WebSocket 连接管理 + 断线重连
│   │   ├── telegram.ts          # Telegram Bot 推送
│   │   └── indicator-cache.ts   # 指标预计算 + 缓存管理
│   └── workers/                # Worker Threads（CPU 密集型任务隔离）
│       ├── backtest-worker.ts  # 回测引擎运行在独立线程
│       └── optimization-worker.ts # 参数优化运行在独立线程
│   └── utils/
│       ├── logger.ts            # 分级日志（Warning / Error）
│       └── errors.ts            # 统一错误码定义
├── src/                         # Next.js 前端（App Router）
│   ├── app/                     # 页面路由
│   │   ├── layout.tsx           # 根布局（侧边栏 + 内容区）
│   │   ├── page.tsx             # 首页 / 仪表盘
│   │   ├── strategy/
│   │   │   ├── page.tsx         # 策略列表
│   │   │   └── [id]/
│   │   │       └── page.tsx     # 策略编辑 / 新建
│   │   ├── monitor/
│   │   │   └── page.tsx         # 监控面板
│   │   ├── backtest/
│   │   │   └── page.tsx         # 回测模块
│   │   ├── optimization/
│   │   │   └── page.tsx         # 参数优化
│   │   └── settings/
│   │       └── page.tsx         # 设置页
│   ├── components/
│   │   ├── ui/                  # shadcn/ui 基础组件（Button, Dialog, Toast...）
│   │   ├── layout/              # 布局组件（Sidebar, Header, TabContainer）
│   │   ├── strategy/            # 策略构建器组件
│   │   │   ├── ConditionTree.tsx    # 条件树编辑器
│   │   │   ├── ConditionNode.tsx    # 单个条件节点
│   │   │   ├── IndicatorSelector.tsx # 指标选择
│   │   │   ├── PresetSelector.tsx   # 预设条件选择
│   │   │   ├── ParamEditor.tsx      # 参数编辑（智能范围提示）
│   │   │   └── SymbolPicker.tsx     # 监测对象选择器
│   │   ├── monitor/             # 监控面板组件
│   │   │   ├── TaskList.tsx         # 任务列表
│   │   │   ├── TaskCard.tsx         # 单个任务卡片
│   │   │   ├── SignalFeed.tsx       # 实时信号流
│   │   │   └── AlertEditor.tsx      # 即时告警编辑
│   │   ├── backtest/            # 回测组件
│   │   │   ├── BacktestForm.tsx     # 回测参数表单
│   │   │   ├── ResultDashboard.tsx  # 结果仪表盘
│   │   │   ├── EquityChart.tsx      # 权益曲线图
│   │   │   └── TradeTable.tsx       # 交易明细表
│   │   └── shared/              # 通用组件
│   │       ├── KlineChart.tsx       # K 线图（lightweight-charts）
│   │       ├── SymbolSearch.tsx     # 币种搜索（复用）
│   │       └── ProgressBar.tsx      # 通用进度条
│   ├── stores/                  # Zustand 状态管理
│   │   ├── strategy-store.ts    # 策略列表、当前编辑策略
│   │   ├── monitor-store.ts     # 监控任务、实时价格、信号
│   │   ├── backtest-store.ts    # 回测状态、结果数据
│   │   └── settings-store.ts    # 应用设置、Telegram 配置
│   ├── lib/
│   │   ├── api.ts               # IPC 调用封装（前端统一入口）
│   │   ├── indicators.ts        # 指标元数据定义（名称、参数、预设条件映射）
│   │   ├── indicators-calc.ts   # 指标计算引擎（各指标核心算法实现）
│   │   ├── constants.ts         # 全局常量（时间框架、默认参数等）
│   │   └── utils.ts             # 通用工具函数
│   ├── types/                   # TypeScript 类型定义
│   │   ├── strategy.ts          # 策略、条件树类型
│   │   ├── monitor.ts           # 监控任务、信号类型
│   │   ├── backtest.ts          # 回测、交易类型
│   │   └── common.ts            # 通用类型（Timeframe, Symbol 等）
│   └── mocks/                   # Mock 实现（直接实现 ElectronAPI 接口）
│       ├── api.ts               # createMockAPI() 入口，环境变量切换
│       ├── strategy-mock.ts     # 策略 CRUD（内存 Map）
│       ├── monitor-mock.ts      # 监控任务（定时器模拟实时数据）
│       ├── backtest-mock.ts     # 回测（预设数据 + 模拟延迟）
│       └── data/                # mock 数据（策略示例、K线数据、信号样例）
├── src/locales/                 # 国际化资源（v1.1 实现）
│   └── zh/                      # 简体中文翻译
│       └── messages.json
# 注: 若使用 next-intl App Router 版，messages 建议放 messages/ 目录下
# 而非 src/locales/，以配合 next-intl 的文件路由约定
├── tests/                       # 测试文件
│   ├── unit/                   # 单元测试（Vitest）
│   ├── integration/            # 集成测试
│   └── e2e/                    # E2E 测试（Playwright）
├── package.json
├── electron-builder.yml         # Electron 打包配置
├── tailwind.config.ts
├── tsconfig.json
└── next.config.js
```

**next.config.js 优化**:
```javascript
// next.config.js
output: 'export'  // 静态导出，剥离 Node.js 服务端
                  // Electron 直接加载静态文件，无需 Next.js 服务器
                  // 注意: 不兼容 API Routes，如有用到需移除
```

## 指标计算模块

### 计算引擎架构

```
indicators-calc.ts
├── 移动平均类 (MA)
│   ├── SMA  - 简单移动平均
│   ├── EMA  - 指数移动平均
│   └── WMA  - 加权移动平均
├── 趋势类 (Trend)
│   ├── ADX     - 平均趋向指数
│   ├── Parabolic SAR
│   └── Ichimoku Cloud
├── 动量类 (Momentum)
│   ├── RSI
│   ├── Stochastic
│   ├── MACD
│   ├── CCI
│   └── ROC
├── 波动率类 (Volatility)
│   ├── Bollinger Bands
│   ├── ATR
│   ├── Keltner Channel
│   ├── Donchian Channel
│   └── Standard Deviation
└── 成交量类 (Volume)
    ├── OBV
    ├── VWAP
    ├── ADL
    ├── CMF
    └── Volume
```

### 核心数据类型

```typescript
// K 线数据
interface OHLCV {
  timestamp: number;
  open: number;
  high: number;
  low: number;
  close: number;
  volume: number;
}

// 指标输出类型
type IndicatorValue = number | IndicatorMultiValue;

interface IndicatorMultiValue {
  [key: string]: number;
}

// 指标函数签名
type IndicatorFunc = (candles: OHLCV[], params: Record<string, number>) => IndicatorValue[];
```

### 指标计算公式

#### 1. 移动平均类 (MA)

##### SMA（简单移动平均）
```
SMA(close, period) = Sum(close[i], i=0 to period-1) / period
```
- **参数**: `period` (默认: 14)
- **输出**: 单值
- **注意**: 数据不足 period 根时返回 NaN

##### EMA（指数移动平均）
```
α = 2 / (period + 1)

// 初始值（前 period 根 K 线使用 SMA）
EMA[period] = SMA(close[0..period-1], period)

// 后续使用 EMA 公式
EMA[t] = α * close[t] + (1 - α) * EMA[t-1]

// 或者使用迭代形式:
EMA[t] = EMA[t-1] + α * (close[t] - EMA[t-1])
```
- **参数**: `period` (默认: 12, 26)
- **输出**: 单值
- **注意**: 前 period-1 根 K 线返回 NaN（数据不足）

##### WMA（加权移动平均）
```
// weight[i] 表示第 i 个价格的权重（i 从 0 到 period-1，i=0 是最旧的）
weight[i] = (period - i) * (period + 1) / 2  // 等差权重

WMA(close, period) = Σ(close[i] * weight[i]) / Σ(weight[i])

// 等效简化: Σ(close[i] * (period - i)) / Σ(period - i)
// Σ(period - i) = period * (period + 1) / 2
```
- **参数**: `period` (默认: 14)
- **输出**: 单值
- **注意**: 数据不足 period 根时返回 NaN

#### 2. 趋势类 (Trend)

##### ADX（平均趋向指数）
```
// Directional Movement
+DM[t] = max(high[t] - high[t-1], 0)   // 仅正向变动
-DM[t] = max(low[t-1] - low[t], 0)    // 仅负向变动

// 同向 DM 取最大，异向取 0（避免两者同时为正）
if (+DM[t] > -DM[t]) { -DM[t] = 0 }
if (-DM[t] > +DM[t]) { +DM[t] = 0 }

TR[t] = max(high[t] - low[t], |high[t] - close[t-1]|, |low[t] - close[t-1]|)

// Wilder 平滑（与 ATR 相同）
+DI[t] = WilderSmooth(+DM, TR, period) * 100
-DI[t] = WilderSmooth(-DM, TR, period) * 100

// DX = 趋向指数
DX[t] = 100 * |+DI[t] - -DI[t]| / (+DI[t] + -DI[t])

// ADX = DX 的 Wilder 平滑
ADX[t] = WilderSmooth(DX, period)

// 其中 WilderSmooth(x, period) = (prev * (period-1) + x) / period
```
- **参数**: `period` (默认: 14), `ADX_period` (默认: 14)
- **输出**: `{ adx: number, plusDI: number, minusDI: number }`
- **注意**: +DI > -DI 表示上涨趋势，反之表示下跌趋势；ADX 越高表示趋势越强

##### Parabolic SAR
```
// 初始 SAR = 第一根 K 线的 EP（最低价或最高价）
// 初始 AF = af_start (0.02)

// 上涨趋势:
SAR[t] = SAR[t-1] + AF * (EP - SAR[t-1])
if (high[t] > EP) { EP = high[t]; AF = min(AF + af_step, af_max) }

// 下跌趋势:
SAR[t] = SAR[t-1] - AF * (SAR[t-1] - EP)
if (low[t] < EP) { EP = low[t]; AF = min(AF + af_step, af_max) }

// 趋势反转（ SAR 穿越价格）:
if (上涨 AND SAR[t] > low[t]) → 反转为下跌，SAR = EP，AF = af_start
if (下跌 AND SAR[t] < high[t]) → 反转为上涨，SAR = EP，AF = af_start

EP (极值点): 上行时为周期最高价，下行时为周期最低价
```
- **参数**: `af_start` (默认: 0.02), `af_max` (默认: 0.2), `af_step` (默认: 0.02)
- **输出**: 单值（SAR 数值）
- **注意**: SAR 穿越价格时触发趋势反转，需平仓并反向开仓

##### Ichimoku Cloud（一目均衡表）
```
Tenkan-Sen (转折线) = (9周期最高价 + 9周期最低价) / 2
Kijun-Sen (基准线)  = (26周期最高价 + 26周期最低价) / 2
Senkou Span A (先行线A) = (转折线 + 基准线) / 2，前移 26 周期
Senkou Span B (先行线B) = (52周期最高价 + 52周期最低价) / 2，前移 26 周期
Chikou Span (延迟线) = close，后移 26 周期

云层 = 先行线A 与 先行线B 之间的区域

// 前移/后移说明（关键，容易混淆）
// - 先行线A/B: 当前计算完成后，在时间轴上向前（右侧）移动 displacement 个周期
//   即：在 t 时刻，Senkou Span A 的值实际上代表 t+26 周期的预测
// - 延迟线: t 时刻的 Chikou Span = close[t-26]，即向后看 26 周期
```
- **参数**: `tenkan_period` (9), `kijun_period` (26), `senkou_b_period` (52), `displacement` (26)
- **输出**: `{ tenkan: number, kijun: number, senkouA: number, senkouB: number, chikou: number }`
- **注意**: 云层前移使得 Ichimoku 成为"领先"指标，帮助预判支撑/阻力区域

#### 3. 动量类 (Momentum)

##### RSI（相对强弱指数）
```
gain[t] = close[t] > close[t-1] ? close[t] - close[t-1] : 0
loss[t] = close[t] < close[t-1] ? close[t-1] - close[t] : 0

// 标准 Wilder 平滑法（等效于 EMA）
avgGain[period] = Sum(gain[1..period]) / period  // 初始值使用 SMA
avgLoss[period] = Sum(loss[1..period]) / period

// 后续使用 Wilder 平滑
avgGain[t] = (prev_avgGain * (period - 1) + gain[t]) / period
avgLoss[t] = (prev_avgLoss * (period - 1) + loss[t]) / period

RS[t] = avgGain[t] / avgLoss[t]
RSI[t] = 100 - (100 / (1 + RS[t]))

// 边界处理: 当 avgLoss[t] = 0 时，RSI[t] = 100
```
- **参数**: `period` (默认: 14)
- **输出**: 单值 (0-100)
- **注意**: 除零时返回 100（全部为上涨周期）

##### Stochastic（随机指标）
```
// 原始 %K（未平滑）
K_raw[t] = 100 * (close[t] - lowest(low, k_period)) / (highest(high, k_period) - lowest(low, k_period))

// %K 平滑（使用 SMA）
K[t] = SMA(K_raw, smooth_k)

// %D = SMA(K, d_period)
D[t] = SMA(K, d_period)

// 边界处理:
// - 当 highest == lowest 时（价格横盘），K_raw[t] = 50
// - 数据不足 k_period 根时，K_raw 和 K 返回 NaN
```
- **参数**: `k_period` (默认: 14), `smooth_k` (默认: 3), `d_period` (默认: 3)
- **输出**: `{ k: number, d: number }`
- **注意**: 部分实现使用 `SMA(SMA(K_raw, smooth_k), d_period)` 即双重平滑

##### MACD
```
DIF[t] = EMA(close, fast_period) - EMA(close, slow_period)
DEA[t] = EMA(DIF, signal_period)

// 标准 MACD 柱（部分平台如 TradingView 乘以 2，需兼容）
MACD_Histogram[t] = DIF[t] - DEA[t]
```
- **参数**: `fast_period` (默认: 12), `slow_period` (默认: 26), `signal_period` (默认: 9)
- **输出**: `{ dif: number, dea: number, histogram: number }`
- **注意**: TradingView 等平台使用 `histogram * 2`，可通过配置切换兼容模式

##### CCI（顺势指标）
```
TP[t] = (high[t] + low[t] + close[t]) / 3
SMA_TP[t] = SMA(TP, period)
MeanDeviation[t] = Σ|TP[i] - SMA_TP[t]| / period  (i 从 t-period+1 到 t)
CCI[t] = (TP[t] - SMA_TP[t]) / (0.015 * MeanDeviation[t])

// 边界处理: 当 MeanDeviation = 0 时（TP 完全相等），CCI[t] = 0
```
- **参数**: `period` (默认: 14)
- **输出**: 单值（通常 -100 到 +100，可超出）
- **注意**: 0.015 是 CCI 的标准化常数，使约 70-80% 的值落在 -100 到 +100 之间

##### ROC（变动率指标）
```
ROC[t] = 100 * (close[t] - close[t-period]) / close[t-period]

// 边界处理: 当 close[t-period] = 0 或接近 0 时，ROC 返回 NaN 或极大值
```
- **参数**: `period` (默认: 12)
- **输出**: 单值

#### 4. 波动率类 (Volatility)

##### Bollinger Bands（布林带）
```
中轨 (MB) = SMA(close, period)
上轨 (UB) = MB + k * StdDev(close, period)
下轨 (LB) = MB - k * StdDev(close, period)
带宽 = (UB - LB) / MB * 100
```
- **参数**: `period` (默认: 20), `std_dev` (默认: 2)
- **输出**: `{ upper: number, middle: number, lower: number, bandwidth: number }`

##### ATR（平均真实波幅）
```
TR[t] = max(
  high[t] - low[t],
  |high[t] - close[t-1]|,
  |low[t] - close[t-1]|
)

// 标准 Wilder 平滑 ATR
ATR[period] = Sum(TR[1..period]) / period  // 初始值使用 SMA
ATR[t] = (prev_ATR * (period - 1) + TR[t]) / period  // Wilder 平滑

// 等效表达: ATR[t] = (prev_ATR * period + TR[t] - TR[t-period]) / period
```
- **参数**: `period` (默认: 14)
- **输出**: 单值
- **注意**: Wilder 平滑使 ATR 响应更平滑，避免短期波动干扰

##### Keltner Channel（肯特纳通道）
```
中轨 = EMA(close, ema_period)
上轨 = 中轨 + k * ATR(period)
下轨 = 中轨 - k * ATR(period)
```
- **参数**: `ema_period` (默认: 20), `atr_period` (默认: 10), `k` (默认: 2)
- **输出**: `{ upper: number, middle: number, lower: number }`

##### Donchian Channel（唐奇安通道）
```
上轨 = highest(high, period)
下轨 = lowest(low, period)
中轨 = (上轨 + 下轨) / 2
```
- **参数**: `period` (默认: 20)
- **输出**: `{ upper: number, middle: number, lower: number }`

##### Standard Deviation（标准差）
```
StdDev(close, period) = sqrt(Sum((close[i] - SMA(close, period))^2, i=0 to period-1) / period)
```
- **参数**: `period` (默认: 20)
- **输出**: 单值

#### 5. 成交量类 (Volume)

##### OBV（能量潮）
```
OBV[t] = OBV[t-1] + volume[t] * sign(close[t] - close[t-1])
其中 sign = +1 (上涨), -1 (下跌), 0 (持平)

// 边界处理: 当 close[t] == close[t-1] 时，sign = 0，OBV 不变
```
- **参数**: 无
- **输出**: 单值

##### VWAP（成交量加权平均价）
```
VWAP[t] = Σ(price * volume) / Σ(volume)
其中 price = (high + low + close) / 3
日内重置，从开盘重新累计
```
- **参数**: 无
- **输出**: 单值

##### ADL（累积/派发线）
```
MFV[t] = volume[t] * ((close[t] - low[t]) - (high[t] - close[t])) / (high[t] - low[t])
ADL[t] = ADL[t-1] + MFV[t]

// 边界处理: 当 high[t] == low[t] 时（波动为零），MFV[t] = 0
```
- **参数**: 无
- **输出**: 单值

##### CMF（资金流量指标）
```
MF[t] = volume[t] * ((close[t] - low[t]) - (high[t] - close[t])) / (high[t] - low[t])
CMF[t] = SMA(MF, period) / SMA(volume, period)

// 边界处理: 当 high[t] == low[t] 时，MF[t] = 0
```
- **参数**: `period` (默认: 20)
- **输出**: 单值 (-1 到 +1)
- **参数**: `period` (默认: 20)
- **输出**: 单值 (-1 到 +1)

##### Volume（成交量）
```
// 成交量简单比较
volume_avg[t] = SMA(volume, period)  // 或 EMA
volume_ratio[t] = volume[t] / volume_avg[t]

// 成交量变化率
volume_change[t] = 100 * (volume[t] - volume[t-1]) / volume[t-1]
```
- **参数**: `period` (默认: 20)
- **输出**: `{ ratio: number, change: number }`

### 背离计算

背离是指标与价格走势不一致的情况，用于判断趋势反转。

#### 波段识别算法（关键前置步骤）

背离检测依赖"波段高低点"识别。使用 ZigZag 变体算法：

```
// 局部极值法识别波段
function findSwingPoints(prices: number[], swingSize: number): SwingPoint[] {
  // swingSize: 波段最小长度（默认 5 根 K 线）
  swings = []

  for i = swingSize to length(prices) - swingSize:
    // 检查是否是局部最高点
    if prices[i] == max(prices[i-swingSize : i+swingSize]):
      swings.push({ index: i, type: 'high', value: prices[i] })

    // 检查是否是局部最低点
    if prices[i] == min(prices[i-swingSize : i+swingSize]):
      swings.push({ index: i, type: 'low', value: prices[i] })

  return swings  // 按 index 排序
}

// 简化版：直接使用最近 N 根 K 线的极值
function findRecentSwingLow(candles: OHLCV[], lookback: number): { index: number, value: number } {
  window = candles[-lookback:]
  minIdx = argmin(window.low)
  return { index: length(candles) - lookback + minIdx, value: window.low[minIdx] }
}
```

#### 算法参数
- `lookback_period`: 回溯窗口（默认: 50 根 K 线）
- `confirmation_bars`: 确认所需 K 线数（默认: 3）
- `price_threshold`: 价格变动最小幅度（默认: 0.5%，避免噪音）
- `swing_size`: 波段识别最小长度（默认: 5）

#### 底背离检测伪代码

```
function detectBottomDivergence(candles, indicatorValues, params):
  lookback = params.lookback_period
  confirmBars = params.confirmation_bars
  threshold = params.price_threshold

  // 1. 找到最近波段低点
  recentLow = findRecentSwingLow(candles, lookback)
  prevLow = findPreviousSwingLow(candles, recentLow.index, lookback)

  // 2. 价格条件：创新低
  if recentLow.value >= prevLow.value:
    return NO_DIVERGENCE

  // 3. 价格变动需超过阈值（避免噪音）
  priceChange = (prevLow.value - recentLow.value) / prevLow.value
  if priceChange < threshold:
    return NO_DIVERGENCE

  // 4. 指标条件：未创新低（或更高）
  recentInd = indicatorValues[recentLow.index]
  prevInd = indicatorValues[prevLow.index]
  if recentInd <= prevInd:
    return NO_DIVERGENCE  // 指标也创新低，不是背离

  // 5. 趋势确认：价格下跌趋势中的背离更可靠
  priceSlope = linearSlope(candles.low[-5:], 5)
  if priceSlope >= 0:
    return NO_DIVERGENCE  // 价格不在下跌趋势

  // 6. 等待确认 K 线
  for i = 1 to confirmBars:
    if indicatorValues[-i] <= recentInd:
      return NO_DIVERGENCE  // 指标未持续走强

  return DIVERGENCE_CONFIRMED
```

#### 顶背离检测伪代码

```
function detectTopDivergence(candles, indicatorValues, params):
  // 对称于底背离，方向反转
  lookback = params.lookback_period
  confirmBars = params.confirmation_bars

  recentHigh = findRecentSwingHigh(candles, lookback)
  prevHigh = findPreviousSwingHigh(candles, recentHigh.index, lookback)

  // 价格创新高
  if recentHigh.value <= prevHigh.value:
    return NO_DIVERGENCE

  // 指标未创新高（顶背离）
  recentInd = indicatorValues[recentHigh.index]
  prevInd = indicatorValues[prevHigh.index]
  if recentInd >= prevInd:
    return NO_DIVERGENCE

  // 等待确认
  for i = 1 to confirmBars:
    if indicatorValues[-i] >= recentInd:
      return NO_DIVERGENCE

  return DIVERGENCE_CONFIRMED
```

#### 状态窗口管理
```typescript
interface DivergenceState {
  detected: boolean;        // 是否检测到背离
  confirmed: boolean;       // 是否已确认
  startBar: number;         // 开始检测的 K 线索引
  confirmationBars: number; // 已确认的 K 线数
  priceExtreme: number;     // 价格极值点
  indicatorExtreme: number; // 指标极值点
  direction: 'top' | 'bottom';
}
```

#### 背离类型细分
| 类型 | 价格 | 指标 | 说明 |
|------|------|------|------|
| 常规底背离 | 新低 | 未新低 | 最常见 |
| 隐藏底背离 | 未新低 | 新低 | 趋势延续信号 |
| 常规顶背离 | 新高 | 未新高 | 最常见 |
| 隐藏顶背离 | 未新高 | 新高 | 趋势延续信号 |

### 预设条件与指标计算映射

| 指标 | 预设条件 | 计算公式 |
|------|---------|---------|
| RSI > 70 | RSI(14) > 70 | 直接取 RSI 值与阈值比较 |
| RSI 底背离 | 价格创新低且 RSI 未创新低 | 背离算法 |
| MACD 金叉 | DIF 上穿 DEA | DIF[t-1] < DEA[t-1] AND DIF[t] > DEA[t] |
| Bollinger Band 突破上轨 | close > upper_band | 直接比较 |
| ATR 创新高 | ATR > max(ATR[n]) | 滑动窗口最大值 |
| Stochastic K > 80 | K(14,3) > 80 | 直接取 K 值与阈值比较 |
| CCI > +100 | CCI(14) > 100 | 直接取 CCI 值与阈值比较 |
| OBV 连续上升 | OBV[t] > OBV[t-n] | 连续 N 周期比较 |

### 边界处理与异常值

指标计算中可能产生 NaN、Infinity 等异常值，需统一处理：

```typescript
// 边界处理策略
function sanitizeIndicatorValue(value: number, indicatorType: string): number {
  if (!Number.isFinite(value)) return NaN;  // NaN 和 Infinity 保留

  // 各指标合理范围检查（超出视为异常）
  const INDICATOR_BOUNDS = {
    RSI: { min: 0, max: 100 },
    Stochastic: { min: 0, max: 100 },
    ADX: { min: 0, max: 100 },
    CCI: { min: -500, max: 500 },    // CCI 可超范围，但有限度
    MACD: { min: -1e6, max: 1e6 },   // 基于价格绝对值
    Bollinger_Bands: { min: 0, max: 1e8 },
    ATR: { min: 0, max: 1e8 },
    // ...
  };

  const bounds = INDICATOR_BOUNDS[indicatorType];
  if (bounds && (value < bounds.min || value > bounds.max)) {
    return NaN;
  }
  return value;
}

// 各指标特殊边界处理
const BOUNDARY_HANDLERS = {
  RSI: (value: number) => Math.max(0, Math.min(100, value)),
  Stochastic: (value: number) => Math.max(0, Math.min(100, value)),
  CCI: (value: number) => Math.max(-500, Math.min(500, value)),  // 合理范围
  ADX: (value: number) => Math.max(0, Math.min(100, value)),
  // ... 其他指标
};
```

| 场景 | 原因 | 处理方式 |
|------|------|---------|
| 除零 | ATR/布林带分母为 0 | 返回上一有效值或 NaN |
| 价格全相等 | 波动率为 0 | 返回 0 或 NaN，取决于指标含义 |
| 数据不足 | K 线数少于指标周期 | 返回 NaN，需足够预热期数据 |
| 极端行情 | 价格剧烈波动 | 限制输出范围，避免溢出 |

### 批量计算接口

```typescript
// 指标计算器接口
interface IndicatorCalculator {
  // 单指标计算
  calculate(
    indicatorType: string,
    candles: OHLCV[],
    params: Record<string, number>
  ): Promise<IndicatorValue[]>;

  // 批量指标计算（用于参数优化）
  calculateBatch(
    requests: Array<{
      indicatorType: string;
      candles: OHLCV[];
      params: Record<string, number>;
    }>
  ): Promise<Map<string, IndicatorValue[]>>;

  // 带缓存的计算（优先查缓存，未命中则计算并写入缓存）
  calculateWithCache(
    indicatorType: string,
    candles: OHLCV[],
    params: Record<string, number>,
    cacheKey: string
  ): Promise<IndicatorValue[]>;
}
```

### 内存管理

| 限制项 | 默认值 | 说明 |
|--------|--------|------|
| 单币种 K 线缓存上限 | 10,000 根 | 超出时卸载最旧的 |
| 指标缓存总上限 | 200MB | LRU 驱逐 |
| 指标缓存组合上限 | 50 组参数 | 避免参数爆炸 |
| 计算预热 K 线 | max(指标周期) × 2 | 避免冷启动问题 |

```typescript
// 内存管理策略
const MEMORY_CONFIG = {
  maxKlinesPerSymbol: 10000,
  maxIndicatorCacheMB: 200,
  maxIndicatorParamCombinations: 50,
  warmupBars: (indicatorType: string) => INDICATOR_PERIODS[indicatorType] * 2,
};
```

---

## 数据模型（SQLite Schema）

### 安全规范

### 时间戳与时区规范

| 类型 | 存储格式 | 时区 | 说明 |
|------|---------|------|------|
| K 线 timestamp | Unix ms (INTEGER) | UTC | 币安 API 返回 UTC，无需转换 |
| 数据库 created_at | `datetime('now')` | 交易所时间 (UTC+8) | SQLite 默认本地时区，部署时统一 |
| 日志时间 | ISO 8601 | UTC | 便于聚合分析 |
| UI 展示 | 用户本地时区 | 用户系统设置 | 前端转换 |

**关键原则**:
- **K 线数据**: 统一使用 UTC 存储，与交易所时区一致
- **收线判定**: 以 UTC 0 点为分界线（适用于日线及以下周期）
- **用户展示**: 前端根据用户系统时区转换后显示
- **备份/日志**: 使用 UTC，便于跨时区问题排查

**实现示例**:
```typescript
// 存储 (主进程)
const timestamp = candle.timestamp; // 来自币安，已经是 UTC ms

// 展示 (渲染进程)
const localTime = new Date(timestamp).toLocaleString(); // 自动转用户本地时区
```

**SQL 注入防护**:
- 所有用户输入必须通过参数化查询，禁止拼接 SQL 字符串
- Symbol、Timeframe 等字段需验证格式
- JSON 字段存储前需验证 JSON 格式有效性

**Symbol 格式规范**:
```
内部规范 (Internal):  大写无分隔符    例: BTCUSDT, ETHUSDT
UI 显示格式:          大写带分隔符   例: BTC/USDT, ETH/USDT
Binance API 格式:    同内部格式      例: BTCUSDT (直接使用)

转换边界:
- 用户输入 (BTC/USDT) → 内部转换 (BTCUSDT) → 存储
- 数据库读取 → 内部格式 (BTCUSDT) → UI 显示 (BTC/USDT)
- API 调用 → 直接使用内部格式 (BTCUSDT)

验证正则: /^BTCUSDT|ETHUSDT|SOLUSDT|...$/ (白名单或通用 /^([A-Z]+)(USDT|USDC)$/)
```
- 统一使用 `BTCUSDT` 格式存储，UI 层负责展示时添加 `/`

```typescript
// 正确示例
const stmt = db.prepare('SELECT * FROM strategies WHERE id = ?');
const result = stmt.get(strategyId);

// 错误示例（禁止）
const query = `SELECT * FROM strategies WHERE id = '${userInput}'`; // SQL 注入风险！
```

**JSON 字段使用规范**:
- `tags`: 简单数组，仅存储字符串，用于前端展示筛选，可接受 JSON 解析错误降级
- `conditions`: 复杂条件树，需严格 schema 验证，验证失败拒绝保存
- `symbols`: 简单数组，存储前验证每项符合 symbol 格式正则

```typescript
// JSON 验证示例
function validateConditions(json: string): boolean {
  try {
    const obj = JSON.parse(json);
    return validateConditionSchema(obj); // 递归验证结构
  } catch {
    return false;
  }
}
```

```sql
-- ============================
-- 策略相关
-- ============================

CREATE TABLE strategies (
  id TEXT PRIMARY KEY,                              -- UUID v4
  name TEXT NOT NULL,                                -- 策略名称
  tags TEXT DEFAULT '[]',                            -- JSON array: ["趋势", "BTC"]
  conditions TEXT NOT NULL,                          -- JSON: 条件树（见条件树类型定义）
  timeframe TEXT NOT NULL,                           -- "5m" | "15m" | "30m" | "1h" | "4h" | "6h" | "12h" | "1d" | "1w"
  symbols TEXT NOT NULL,                             -- JSON array: ["BTCUSDT", "ETHUSDT"]
  parameters TEXT DEFAULT '{}',                      -- JSON: 用户自定义参数覆盖
  is_draft INTEGER DEFAULT 0,                       -- 0=已保存, 1=草稿
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at TEXT NOT NULL DEFAULT (datetime('now')),
  deleted_at TEXT                                    -- 软删除时间，NULL=正常
);

CREATE INDEX idx_strategies_deleted ON strategies(deleted_at);
-- 注意: tags 是 JSON 数组文本字段，无法用普通索引查询
-- 标签筛选在应用层用 JSON 函数或内存过滤实现，不在 DB 层索引

-- ============================
-- 监控任务
-- ============================

CREATE TABLE monitor_tasks (
  id TEXT PRIMARY KEY,
  strategy_id TEXT NOT NULL,
  strategy_snapshot TEXT NOT NULL,                   -- JSON: 创建时复制的策略配置快照
  status TEXT NOT NULL DEFAULT 'stopped' CHECK(status IN ('running', 'paused', 'stopped')),
  symbols TEXT NOT NULL,                             -- JSON array
  cooldown_seconds INTEGER DEFAULT 300,
  last_trigger_at TEXT,
  error_count INTEGER DEFAULT 0,                     -- 连续错误计数，用于熔断
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  updated_at TEXT NOT NULL DEFAULT (datetime('now')),
  FOREIGN KEY (strategy_id) REFERENCES strategies(id)
);

-- ============================
-- 信号记录
-- ============================

CREATE TABLE signal_records (
  id TEXT PRIMARY KEY,
  monitor_task_id TEXT NOT NULL,
  strategy_id TEXT NOT NULL,
  strategy_name TEXT NOT NULL,                       -- 冗余存储，避免关联查询
  symbol TEXT NOT NULL,
  signal_type TEXT NOT NULL,                         -- "buy" | "sell" | "custom"
  price REAL NOT NULL,
  indicator_values TEXT,                             -- JSON: 触发时各指标数值快照
  triggered_at TEXT NOT NULL DEFAULT (datetime('now')),
  FOREIGN KEY (monitor_task_id) REFERENCES monitor_tasks(id),
  FOREIGN KEY (strategy_id) REFERENCES strategies(id)
);

CREATE INDEX idx_signal_task ON signal_records(monitor_task_id);
CREATE INDEX idx_signal_time ON signal_records(triggered_at);

-- ============================
-- 即时告警
-- ============================

CREATE TABLE alerts (
  id TEXT PRIMARY KEY,
  symbol TEXT NOT NULL,
  condition TEXT NOT NULL,                           -- JSON: 单个条件定义
  is_active INTEGER DEFAULT 1,
  last_triggered_at TEXT,
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE alert_records (
  id TEXT PRIMARY KEY,
  alert_id TEXT NOT NULL,
  price REAL NOT NULL,
  triggered_at TEXT NOT NULL DEFAULT (datetime('now')),
  FOREIGN KEY (alert_id) REFERENCES alerts(id)
);

-- ============================
-- 回测结果
-- ============================

CREATE TABLE backtest_results (
  id TEXT PRIMARY KEY,
  strategy_id TEXT NOT NULL,
  strategy_name TEXT NOT NULL,
  symbol TEXT NOT NULL,
  timeframe TEXT NOT NULL,
  start_date TEXT NOT NULL,
  end_date TEXT NOT NULL,
  fee_rate REAL DEFAULT 0.001,                    -- 小数: 0.001 = 0.1%
  slippage REAL DEFAULT 0.0005,                  -- 小数: 0.0005 = 0.05%
  position_pct REAL DEFAULT 0.1,                 -- 小数: 0.1 = 10%
  stop_loss_pct REAL DEFAULT 0.02,               -- 小数: 0.02 = 2%
  take_profit_pct REAL DEFAULT 0.04,             -- 小数: 0.04 = 4%
  trailing_stop_pct REAL DEFAULT 0.01,
  trade_direction TEXT DEFAULT 'long',               -- "long" | "short" | "both"
  total_return REAL NOT NULL,                        -- 总收益率
  annualized_return REAL,                            -- 年化收益率
  max_drawdown REAL NOT NULL,                        -- 最大回撤
  win_rate REAL NOT NULL,                            -- 胜率
  profit_factor REAL,                                -- 盈亏比
  trade_count INTEGER NOT NULL,                      -- 交易次数
  sharpe_ratio REAL,                                 -- 夏普比率
  trades TEXT NOT NULL,                              -- JSON: 交易明细数组
  equity_curve TEXT NOT NULL,                        -- JSON: 权益曲线 [{time, equity}]
  created_at TEXT NOT NULL DEFAULT (datetime('now')),
  FOREIGN KEY (strategy_id) REFERENCES strategies(id)
);

CREATE INDEX idx_backtest_strategy ON backtest_results(strategy_id);

-- ============================
-- K线数据缓存（统一大表 + 复合索引）
-- ============================

CREATE TABLE klines (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  symbol TEXT NOT NULL,                    -- "BTCUSDT"
  timeframe TEXT NOT NULL,                 -- "1h"
  timestamp INTEGER NOT NULL,               -- Unix ms
  open REAL NOT NULL,
  high REAL NOT NULL,
  low REAL NOT NULL,
  close REAL NOT NULL,
  volume REAL NOT NULL,
  created_at TEXT DEFAULT (datetime('now')),
  UNIQUE(symbol, timeframe, timestamp)
);

CREATE INDEX idx_klines_symbol_timeframe ON klines(symbol, timeframe, timestamp DESC);
CREATE INDEX idx_klines_timestamp ON klines(timestamp DESC);

-- 数据下载状态追踪
CREATE TABLE kline_sync_status (
  symbol TEXT NOT NULL,
  timeframe TEXT NOT NULL,
  last_timestamp INTEGER NOT NULL DEFAULT 0,         -- 最新已同步的时间戳
  updated_at TEXT NOT NULL DEFAULT (datetime('now')),
  PRIMARY KEY (symbol, timeframe)
);

-- ============================
-- 指标预计算缓存
-- ============================
-- 注: 指标缓存存储于内存（进程重启后清空），不回写磁盘
-- 如需持久化缓存以支持进程恢复，需自行扩展此表

-- ============================
-- 应用设置
-- ============================

CREATE TABLE app_settings (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL                                -- JSON
);

-- 预置设置项:
-- telegram_bot_token  | string (AES-256-GCM 加密，密钥存平台安全存储)
-- telegram_chat_id    | string (AES-256-GCM 加密，密钥存平台安全存储)
-- data_retention_days | number (默认 90)
-- auto_backup_enabled | boolean (默认 true)
-- sound_enabled       | boolean (默认 true)
-- max_concurrent_backtests | number (默认 5)
-- db_version          | number (当前数据库 schema 版本号)

-- ============================
-- 错误日志
-- ============================

CREATE TABLE error_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  level TEXT NOT NULL CHECK(level IN ('warning', 'error')),
  error_code TEXT,                                   -- E101, E401 等
  message TEXT NOT NULL,
  context TEXT,                                      -- JSON: 上下文（堆栈、参数等）
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);

CREATE INDEX idx_error_level ON error_logs(level);
CREATE INDEX idx_error_time ON error_logs(created_at);
```

### 外键级联行为规范

| 父表 | 子表 | 硬删除 (DELETE) | 软删除 (deleted_at) |
|------|------|----------------|-------------------|
| strategies | monitor_tasks | ON DELETE RESTRICT（需先停止并删除任务） | 停止关联任务（status → stopped），保留数据，strategy_snapshot 保持不变 |
| strategies | signal_records | ON DELETE CASCADE | 保留记录（strategy_name 冗余字段保证可读性） |
| strategies | backtest_results | ON DELETE CASCADE | 保留记录（同上） |
| monitor_tasks | signal_records | ON DELETE CASCADE | N/A（监控任务软删除不常见） |

**说明**:
- 软删除策略：关联任务停止但保留，strategy_snapshot 保存删除时的配置快照
- 恢复策略：任务恢复为 stopped 状态，需用户手动启动
- 硬删除策略：必须先删除关联监控任务（ON DELETE RESTRICT）
- signal_records 和 backtest_results 保留 strategy_name 冗余字段，避免关联查询

**实现建议**:
```sql
-- 硬删除策略前必须先清理关联数据
DELETE FROM signal_records WHERE strategy_id = ?;
DELETE FROM backtest_results WHERE strategy_id = ?;
DELETE FROM monitor_tasks WHERE strategy_id = ?;
DELETE FROM strategies WHERE id = ?;
```

---

## 条件树类型定义（TypeScript）

```typescript
// ============================
// 基础类型
// ============================

type Timeframe = '5m' | '15m' | '30m' | '1h' | '4h' | '6h' | '12h' | '1d' | '1w';

type IndicatorType =
  // 趋势类
  'EMA' | 'SMA' | 'ADX' | 'PARABOLIC_SAR' | 'ICHIMOKU'
  // 动量类
  | 'RSI' | 'STOCHASTIC' | 'MACD' | 'CCI' | 'ROC'
  // 波动类
  | 'BOLLINGER_BANDS' | 'ATR' | 'KELTNER_CHANNEL' | 'DONCHIAN_CHANNEL' | 'STD_DEV'
  // 成交量类
  | 'OBV' | 'VWAP' | 'ADL' | 'CMF' | 'VOLUME'
  // 支撑阻力类
  | 'PIVOT_POINTS' | 'FIBONACCI' | 'PRICE_LEVEL'
  // 情绪类
  | 'FEAR_GREED' | 'FUNDING_RATE' | 'OPEN_INTEREST' | 'LONG_SHORT_RATIO' | 'LIQUIDATION'
  // 波动率类
  | 'HISTORICAL_VOLATILITY';
// 注意: FEAR_GREED 暂不支持 MVP，后续接入第三方 API 后再加入

// ============================
// 条件树
// ============================

interface ConditionGroup {
  id: string;
  type: 'group';
  logic: 'AND' | 'OR';
  children: ConditionTreeNode[];
}

interface ConditionLeaf {
  id: string;
  type: 'leaf';
  indicator: IndicatorType;
  preset: string;                                    // 预设条件 ID，如 "rsi_overbought"
  parameters: Record<string, number>;                // 用户调整的参数（统一为 number）
  // 注: 历史原因保留 string 类型，实际使用中所有参数应为 number
  // Fibonacci 的 tolerance 在前端展示为 "±0.5%"，存储时转为 0.005
  signalDirection: 'buy' | 'sell';                   // 触发时产生的信号方向（买入/卖出）
  timeframe?: Timeframe;                             // 覆盖策略全局 timeframe（可选）
}

type ConditionTreeNode = ConditionGroup | ConditionLeaf;

// ============================
// 信号方向语义
// ============================

// ConditionLeaf 的 signalDirection 表示该条件成立时推荐的交易方向
// 条件组（AND/OR）的结果方向由以下规则决定：

信号方向确定规则:
1. 条件组内所有叶节点应具有相同的 signalDirection（由 strategy:validate 校验）
2. 条件组结果为 true 时，信号方向 = 子叶节点的 signalDirection
3. 若条件组内存在不同方向的叶节点，validate 应返回错误

示例:
- RSI 超卖 + MACD 金叉 → 两者的 signalDirection 都应为 'buy'
- 条件树评估结果为 true → 信号方向 = 'buy'

反向信号处理:
- trade_direction='long' 时: 只执行 buy 信号，忽略 sell 信号
- trade_direction='short' 时: 只执行 sell 信号，忽略 buy 信号
- trade_direction='both' 时: buy 信号开多，sell 信号开空

// ============================
// 指标预设条件映射（示例）
// ============================

interface IndicatorPreset {
  id: string;                                        // "rsi_overbought"
  label: string;                                     // "RSI > 70（超买）"
  indicator: IndicatorType;
  defaultParams: Record<string, number | string>;    // { period: 14, threshold: 70 }
  paramRanges: Record<string, { min: number; max: number; step: number; recommended: number }>;
  // 参数范围，用于智能推荐
}

// ============================
// 策略
// ============================

interface Strategy {
  id: string;
  name: string;
  tags: string[];
  conditions: ConditionTreeNode;                     // 条件树根节点
  timeframe: Timeframe;
  symbols: string[];
  parameters: Record<string, number | string>;
  isDraft: boolean;
  createdAt: string;
  updatedAt: string;
  deletedAt: string | null;
}

// ============================
// 监控任务
// ============================

type MonitorTaskStatus = 'running' | 'paused' | 'stopped';

interface MonitorTask {
  id: string;
  strategyId: string;
  strategySnapshot: Strategy;                        // 独立快照
  status: MonitorTaskStatus;
  symbols: string[];
  cooldownSeconds: number;
  lastTriggerAt: string | null;
  errorCount: number;
  createdAt: string;
  updatedAt: string;
}

// ============================
// 信号记录
// ============================

interface SignalRecord {
  id: string;
  monitorTaskId: string;
  strategyId: string;
  strategyName: string;
  symbol: string;
  signalType: 'buy' | 'sell' | 'custom';
  price: number;
  indicatorValues: Record<string, number>;           // 触发时指标快照
  triggeredAt: string;
}

// ============================
// 回测结果
// ============================

interface BacktestTrade {
  entryTime: string;
  exitTime: string;
  entryPrice: number;
  exitPrice: number;
  direction: 'long' | 'short';
  pnl: number;                                       // 盈亏金额
  pnlPct: number;                                    // 盈亏百分比
  fee: number;
  slippage: number;
  holdingPeriod: number;                             // 持仓时长（分钟）
  signalType: string;
}

interface BacktestResult {
  id: string;
  strategyId: string;
  strategyName: string;
  symbol: string;
  timeframe: Timeframe;
  startDate: string;
  endDate: string;
  // 交易参数
  feeRate: number;
  slippage: number;
  positionPct: number;
  stopLossPct: number;
  takeProfitPct: number;
  trailingStopPct: number;
  tradeDirection: 'long' | 'short' | 'both';
  // 核心指标
  totalReturn: number;
  annualizedReturn: number;
  maxDrawdown: number;
  winRate: number;
  profitFactor: number;
  tradeCount: number;
  sharpeRatio: number;
  // 详细数据
  trades: BacktestTrade[];
  equityCurve: Array<{ time: string; equity: number }>;
  createdAt: string;
}

// ============================
// 参数优化结果
// ============================

interface OptimizationResult {
  id: string;
  optimizationRunId: string;
  parameters: Record<string, number>;                 // 该组合的参数值
  // 核心指标（与 BacktestResult 一致）
  totalReturn: number;
  annualizedReturn: number;
  maxDrawdown: number;
  winRate: number;
  profitFactor: number;
  tradeCount: number;
  sharpeRatio: number;
  // 排名依据（由用户选择的评判标准排序）
  rank: number;
}

// ============================
// IPC 通信类型
// ============================

// 前端 → 主进程 的 API 接口定义
interface ElectronAPI {
  // 策略
  strategy: {
    list(): Promise<Strategy[]>;
    get(id: string): Promise<Strategy | null>;
    create(data: Omit<Strategy, 'id' | 'createdAt' | 'updatedAt'>): Promise<Strategy>;
    update(id: string, data: Partial<Strategy>): Promise<Strategy>;
    delete(id: string): Promise<void>;
    restore(id: string): Promise<Strategy>;
    export(ids: string[]): Promise<string>;            // JSON string
    import(data: string): Promise<Strategy[]>;         // JSON string → 解析导入
    validate(conditions: ConditionTreeNode): Promise<{ valid: boolean; errors: string[] }>;
  };
  // 监控
  monitor: {
    list(): Promise<MonitorTask[]>;
    create(strategyId: string, symbols: string[], cooldown?: number): Promise<MonitorTask>;
    update(id: string, data: Partial<MonitorTask>): Promise<MonitorTask>;
    start(id: string): Promise<void>;
    pause(id: string): Promise<void>;
    resume(id: string): Promise<void>;
    stop(id: string): Promise<void>;
    delete(id: string): Promise<void>;
    getSignals(taskId: string, limit?: number): Promise<SignalRecord[]>;
    // 事件监听
    onSignal(callback: (signal: SignalRecord) => void): () => void;  // 返回取消函数
    onPriceUpdate(callback: (data: { symbol: string; price: number; change24h: number }) => void): () => void;
  };
  // 回测
  backtest: {
    run(params: {
      strategyId: string;
      symbol: string;
      timeframe: Timeframe;
      startDate: string;
      endDate: string;
      feeRate?: number;
      slippage?: number;
      positionPct?: number;
      stopLossPct?: number;
      takeProfitPct?: number;
      trailingStopPct?: number;
      tradeDirection?: 'long' | 'short' | 'both';
    }): Promise<string>;                               // 返回 backtest result id
    cancel(id: string): Promise<void>;                 // 取消正在运行的回测
    getResult(id: string): Promise<BacktestResult>;
    listHistory(strategyId?: string): Promise<BacktestResult[]>;
    deleteResult(id: string): Promise<void>;
    // 进度
    onProgress(callback: (data: { id: string; progress: number; stage: string }) => void): () => void;
  };
  // 参数优化
  optimization: {
    run(params: {
      strategyId: string;
      symbol: string;
      timeframe: Timeframe;
      startDate: string;
      endDate: string;
      paramRanges: Record<string, { min: number; max: number; step: number }>;
      maxConcurrent?: number;
      timeout?: number;
    }): Promise<string>;                               // 返回 optimization run id
    getResults(id: string): Promise<OptimizationResult[]>;
    cancel(id: string): Promise<void>;
    // 进度
    onProgress(callback: (data: { id: string; completed: number; total: number }) => void): () => void;
  };
  // 设置
  settings: {
    get(key: string): Promise<any>;
    set(key: string, value: any): Promise<void>;
    getAll(): Promise<Record<string, any>>;
    testTelegram(botToken: string, chatId: string): Promise<{ success: boolean; error?: string }>;
  };
}
```

---

## API 设计（IPC 通道）

前端通过 `window.electronAPI` 统一调用后端，所有接口类型见上方 `ElectronAPI` 定义。

**通道总览**:

| 通道 | 方向 | 说明 |
|------|------|------|
| `strategy:list/get/create/update/delete/restore` | 前端→主进程 | 策略 CRUD + 回收站恢复 |
| `strategy:export/import` | 前端→主进程 | JSON 导入导出 |
| `strategy:validate` | 前端→主进程 | 条件树合法性校验 |
| `monitor:create/update/delete` | 前端→主进程 | 监控任务管理 |
| `monitor:start/pause/resume/stop` | 前端→主进程 | 任务状态切换 |
| `monitor:getSignals` | 前端→主进程 | 查询信号记录 |
| `monitor:onSignal` | 主进程→前端 | 实时信号推送（IPC event） |
| `monitor:onPriceUpdate` | 主进程→前端 | 实时价格更新（IPC event） |
| `backtest:run/cancel` | 前端→主进程 | 启动回测 / 取消回测 |
| `backtest:getResult/listHistory/deleteResult` | 前端→主进程 | 回测结果管理 |
| `backtest:onProgress` | 主进程→前端 | 回测进度回调 |
| `optimization:run/cancel` | 前端→主进程 | 参数优化控制 |
| `optimization:getResults` | 前端→主进程 | 优化结果查询 |
| `optimization:onProgress` | 主进程→前端 | 优化进度回调 |
| `settings:get/set/getAll` | 前端→主进程 | 应用设置读写 |
| `settings:testTelegram` | 前端→主进程 | Telegram 连接测试 |

**数据流示意**:

```
前端 (Next.js / Zustand)              主进程 (Electron)
    │                                       │
    │  window.electronAPI.monitor.start(taskId)
    │  ──────────────────────────────────►  │
    │                                       │  strategy-engine.ts: 遍历条件树
    │                                       │  data-fetcher.ts: 获取K线数据
    │                                       │  websocket.ts: 订阅实时数据
    │                                       │
    │  ◄──────────────────────────────────  │
    │  onSignal({ symbol, price, ... })     │
    │  onPriceUpdate({ symbol, ... })       │
    │                                       │
    │  Zustand store 更新 → UI 重渲染        │
```

**Worker Threads 架构**（CPU 密集型任务隔离）:

```
主进程 (Electron Main)
├── WebSocket 接收 (不阻塞)
├── IPC 通信 (不阻塞)
├── 指标缓存查询 (内存操作)
└── 任务调度
    │
    ├── backtest-worker.ts (Worker Thread)
    │    ├── K 线数据加载
    │    ├── 回测引擎执行
    │    └── 结果通过 parentPort.postMessage 回传
    │
    └── optimization-worker.ts (Worker Thread)
         ├── 网格搜索调度
         ├── 并发回测执行 (每个组合一个 Worker 或串行)
         └── 结果汇总回传
```

**为什么需要 Worker Threads**:
- Node.js 单线程：回测引擎的数组遍历、指标计算会阻塞事件循环
- 阻塞后果：IPC 通信延迟、WebSocket 断线、UI 卡顿
- Worker Thread：独立线程，不阻塞主进程，通过消息传递结果

**通信协议**:
```typescript
// 主进程 → Worker
worker.postMessage({ type: 'RUN_BACKTEST', payload: { strategyId, symbols, ... } });

// Worker → 主进程
parentPort.on('message', (msg) => {
  if (msg.type === 'PROGRESS') updateProgress(msg.payload);
  if (msg.type === 'COMPLETE') handleResult(msg.payload);
  if (msg.type === 'ERROR') handleError(msg.error);
});
```

**preload.ts 桥接方式**:

```typescript
// preload.ts 通过 contextBridge 暴露类型安全的 API
contextBridge.exposeInMainWorld('electronAPI', {
```

**大数据量 IPC 处理**:

Electron IPC 使用结构化克隆算法，对象大小有限制（约 1GB，但过大会导致性能问题）。回测结果（trades 数组、equity_curve）可能包含数千条记录，需分页传输：

```typescript
// 大数据量分页接口
backtest: {
  getTrades(id: string, page: number, pageSize: number): Promise<Trade[]>;
  getEquityCurve(id: string): Promise<EquityPoint[]>;  // equity_curve 通常较小，可一次性返回
  getResult(id: string, options?: { includeTrades: boolean }): Promise<BacktestResult>;
}
```

**IPC 超时处理**:

所有 `ipcRenderer.invoke()` 调用应包装超时：

```typescript
async function invokeWithTimeout<T>(channel: string, args: any[], timeout = 30000): Promise<T> {
  return Promise.race([
    ipcRenderer.invoke(channel, ...args),
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error('IPC timeout')), timeout)
    )
  ]);
}
```

| 场景 | 建议超时 | 原因 |
|------|---------|------|
| 策略 CRUD | 5s | 简单数据库操作 |
| 回测启动 | 10s | 可能涉及大量数据加载 |
| 回测进度回调 | 无超时 | 长时间运行 |
| 回测结果查询 | 30s | 分页获取 |
| Telegram 测试 | 15s | 外部 API 调用 |
  strategy: {
    list: () => ipcRenderer.invoke('strategy:list'),
    create: (data) => ipcRenderer.invoke('strategy:create', data),
    // ...
  },
  monitor: {
    onSignal: (callback) => {
      ipcRenderer.on('monitor:signal', (_, data) => callback(data));
      return () => ipcRenderer.removeAllListeners('monitor:signal');
    },
    // ...
  },
  // ...
});
```

---

## UI 布局设计

### 整体布局 — 侧边栏 + 内容区 + 状态栏

```
┌──────────────────────────────────────────────────────────────┐
│  ◉ ◉ ◉   盯盘侠 v1.0                              — □ ×   │
├────────┬─────────────────────────────────────────────────────┤
│        │                                                     │
│  📊    │  内容区（按路由切换）                                 │
│ 仪表盘  │                                                     │
│        │  ┌─────────────────────────────────────────────┐    │
│  📈    │  │                                             │    │
│ 策略   │  │                                             │    │
│        │  │        当前页面的内容                          │    │
│  📡    │  │                                             │    │
│ 监控   │  │                                             │    │
│        │  │                                             │    │
│  📉    │  │                                             │    │
│ 回测   │  └─────────────────────────────────────────────┘    │
│        │                                                     │
│  ⚙️    │                                                     │
│ 设置   │                                                     │
│        │                                                     │
├────────┴─────────────────────────────────────────────────────┤
│  状态栏: 连接状态 ● | 运行中监控: 3 | 最新信号: BTC/USDT 买入  │
└──────────────────────────────────────────────────────────────┘
```

**侧边栏**: 固定 60px（图标）+ hover 展开到 180px（显示文字），底部固定设置图标

**状态栏**: 始终显示 WebSocket 连接状态、运行中监控数量、最新信号摘要

### 页面路由

| 路由 | 页面 | 说明 |
|------|------|------|
| `/` | 仪表盘 | 运行中监控概览、最近信号、快捷操作入口 |
| `/strategy` | 策略列表 | 搜索/筛选/管理所有策略 |
| `/strategy/[id]` | 策略编辑 | 条件树编辑器、参数配置、币种选择 |
| `/monitor` | 监控面板 | 运行中/已暂停任务、实时数据、信号流 |
| `/backtest` | 回测模块 | 回测配置、结果仪表盘、权益曲线 |
| `/optimization` | 参数优化 | 网格搜索配置、热力图、参数对比 |
| `/settings` | 设置 | Telegram 配置、数据管理、通用设置 |

### 仪表盘页

```
┌─────────────────────────────────────────────────┐
│ 仪表盘                                          │
│                                                 │
│ ┌─ 运行概览 ──────────────────────────────┐     │
│ │  🟢 运行中: 3    ⏸️ 已暂停: 1    📊 策略: 8 │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ┌─ 最近信号 ──────────────────────────────┐     │
│ │ 🔔 BTC/USDT 买入 $65,890 RSI超卖 14:30  │     │
│ │ 🔔 ETH/USDT 卖出 $3,512 MACD死叉 13:15  │     │
│ │ 🔔 SOL/USDT 买入 $142.3 布林带下轨 11:20│     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ┌─ 快捷操作 ──────────────────────────────┐     │
│ │ [+ 新建策略]  [+ 新建监控]  [运行回测]    │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ┌─ 关注币种概览 ──────────────────────────┐     │
│ │ BTC/USDT  $67,234 (+2.3%)              │     │
│ │ ETH/USDT  $3,456  (-0.5%)              │     │
│ │ SOL/USDT  $142.3  (+5.1%)              │     │
│ └─────────────────────────────────────────┘     │
└─────────────────────────────────────────────────┘
```

### 参数优化页

```
┌─────────────────────────────────────────────────┐
│ 参数优化                                         │
│                                                 │
│ ┌─ 优化配置 ──────────────────────────────┐     │
│ │ 策略: [RSI超卖反弹 ▼]                    │     │
│ │ 币种: [BTC/USDT ▼]    周期: [4h ▼]       │     │
│ │ 时间: [2025-01-01] ~ [2025-12-31]        │     │
│ │ 评判标准: [夏普比率 ▼]                    │     │
│ │ 并发数: [3]  超时: [30s]                  │     │
│ │                                         │     │
│ │ 参数范围:                                 │     │
│ │ RSI period: [7] ~ [21]  step: [2]        │     │
│ │ RSI threshold: [20] ~ [40] step: [5]     │     │
│ │                         [开始优化]       │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ┌─ 优化进度 ──────────────────────────────┐     │
│ │ ████████░░░░░░ 56% (28/50)              │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ┌─ 结果排名（按评判标准排序）──────────────┐     │
│ │ #1 period:14 thresh:30 收益+52% 夏普1.6 │     │
│ │ #2 period:10 thresh:25 收益+48% 夏普1.5 │     │
│ │ #3 period:18 thresh:35 收益+41% 夏普1.3 │     │
│ │                                         │     │
│ │ [热力图] [参数对比] [导出 CSV]           │     │
│ └─────────────────────────────────────────┘     │
└─────────────────────────────────────────────────┘
```

### 策略列表页

```
┌─────────────────────────────────────────────────┐
│ 策略管理                    [+ 新建策略] [导入]   │
│ ┌──────────────────────────────────────────────┐ │
│ │ 🔍 搜索策略...    筛选: [全部▼] [标签▼]       │ │
│ └──────────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────────┐ │
│ │ 📈 RSI超卖反弹策略          [趋势] [BTC]      │ │
│ │ 2 个条件 · 4h · 5 个币种 · 监控中             │ │
│ │                        [编辑] [复制] [删除]    │ │
│ ├──────────────────────────────────────────────┤ │
│ │ 📊 MACD金叉+量能突破         [动量] [ETH]      │ │
│ │ 3 个条件 · 1h · 3 个币种 · 已停止             │ │
│ │                        [编辑] [复制] [删除]    │ │
│ └──────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

### 策略编辑页

```
┌─────────────────────────────────────────────────┐
│ ← 返回列表                  [保存草稿] [保存策略] │
│                                                 │
│ 策略名称: [________________]                     │
│ 标签:     [+ 添加标签]                           │
│                                                 │
│ ── 条件设置 ──────────────────────────────       │
│                                                 │
│  ┌─ AND ────────────────────────────────────┐   │
│  │                                          │   │
│  │  ┌─ 条件1 ─────────────────────── [×] ─┐ │   │
│  │  │ RSI  │ < 30 │ period: 14     │ 4h   │ │   │
│  │  └──────────────────────────────────────┘ │   │
│  │                                          │   │
│  │  ┌─ OR ──────────────────────────────┐   │   │
│  │  │  ┌─ 条件2 ─────────────── [×] ─┐ │   │   │
│  │  │  │ MACD │ 金叉 │ fast:12 slow:26│ │ │   │   │
│  │  │  └───────────────────────────────┘ │ │   │   │
│  │  │  ┌─ 条件3 ─────────────── [×] ─┐ │   │   │
│  │  │  │ MACD │ 底背离 │             │ │ │   │   │
│  │  │  └───────────────────────────────┘ │ │   │   │
│  │  └────────────────────────────────────┘ │   │
│  │                                          │   │
│  │  [+ 添加条件] [+ 添加条件组]              │   │
│  └──────────────────────────────────────────┘   │
│                                                 │
│ ── 监测对象 ──────────────────────────────       │
│ 时间框架: [4h ▼]                                │
│ 币种: [BTC/USDT ×] [ETH/USDT ×] [+ 添加]        │
└─────────────────────────────────────────────────┘
```

### 监控面板页

```
┌─────────────────────────────────────────────────┐
│ 监控面板            [+ 新建监控] [即时告警]        │
│                                                 │
│ ┌─ 运行中 (3) ────────────────────────────┐     │
│ │ 🟢 RSI超卖反弹 · BTC/USDT               │     │
│ │ $67,234.56 (+2.3%)  RSI: 28.4          │     │
│ │ 最近信号: 买入 14:30  [暂停] [编辑]      │     │
│ ├─────────────────────────────────────────┤     │
│ │ 🟢 MACD金叉 · ETH/USDT                  │     │
│ │ $3,456.78 (-0.5%)   MACD: 12.3         │     │
│ │ 冷却中 (剩余 2m)    [暂停] [编辑]        │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ┌─ 已暂停 (1) ────────────────────────────┐     │
│ │ ⏸️ 布林带突破 · SOL/USDT                 │     │
│ │ $142.30 (+5.1%)     [恢复] [编辑] [删除] │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ── 最近信号 ──────────────────────────────       │
│ 🔔 BTC/USDT 买入 $65,890 RSI超卖反弹 14:30      │
│ 🔔 ETH/USDT 卖出 $3,512 MACD死叉 13:15          │
└─────────────────────────────────────────────────┘
```

### 回测页

```
┌─────────────────────────────────────────────────┐
│ 回测                                             │
│                                                 │
│ ┌─ 回测配置 ──────────────────────────────┐     │
│ │ 策略: [RSI超卖反弹 ▼]                    │     │
│ │ 币种: [BTC/USDT ▼]    周期: [4h ▼]       │     │
│ │ 起始: [2025-01-01]  结束: [2025-12-31]   │     │
│ │ 手续费: [0.1%]  滑点: [0.05%]            │     │
│ │ 仓位: [10%]  止损: [-2%]  止盈: [+4%]    │     │
│ │ 方向: [做多 ▼]                            │     │
│ │                          [开始回测]       │     │
│ └─────────────────────────────────────────┘     │
│                                                 │
│ ┌─ 回测结果 ──────────────────────────────┐     │
│ │ 总收益: +45.2%  最大回撤: -12.3%         │     │
│ │ 胜率: 58.3%    盈亏比: 1.8              │     │
│ │ 交易次数: 24    夏普比率: 1.45           │     │
│ │                                         │     │
│ │ ┌─ 权益曲线 ──────────────────────┐     │     │
│ │ │          ╱‾‾╲  ╱‾‾‾             │     │     │
│ │ │    ╱‾‾╲╱    ╲╱                  │     │     │
│ │ │   ╱                             │     │     │
│ │ │  ╱                              │     │     │
│ │ │ ╱                               │     │     │
│ │ └─────────────────────────────────┘     │     │
│ │                                         │     │
│ │ [交易明细] [保存参数到策略] [导出]       │     │
│ └─────────────────────────────────────────┘     │
└─────────────────────────────────────────────────┘
```

### 设置页

```
┌─────────────────────────────────────────────────┐
│ 设置                                             │
│                                                 │
│ ── Telegram 通知 ────────────────────────        │
│ Bot Token: [••••••••••••]                        │
│ Chat ID:   [••••••••]                            │
│                              [测试连接]          │
│                                                 │
│ ── 数据管理 ─────────────────────────────        │
│ 历史数据保留: [3个月 ▼]                           │
│ K线缓存: 2.3 GB                     [清理缓存]   │
│ 自动备份: [✓] 每周                                │
│                                      [导出备份]  │
│                                                 │
│ ── 通用 ─────────────────────────────────        │
│ 信号提示音: [✓]                                   │
│ 最大并发回测: [5]                                  │
│ 自动检查更新: [✓]                                  │
└─────────────────────────────────────────────────┘
```

## Mock 数据策略

### 原则

前端开发完全不依赖后端，MSW 拦截所有 IPC 调用，Mock 层和真实 IPC 层共享 `ElectronAPI` 类型定义。

### 前后端切换

通过环境变量 `NEXT_PUBLIC_USE_MOCK=true` 控制是否启用 Mock。

**注意**: Mock 层直接实现 `ElectronAPI` 接口（JavaScript 对象），不走 HTTP 拦截。Electron 的 IPC 通信不是 HTTP 请求，Mock 实现和真实 preload 暴露的 API 共享同一类型签名。

```typescript
// src/lib/api.ts
const api: ElectronAPI = process.env.NEXT_PUBLIC_USE_MOCK === 'true'
  ? createMockAPI()    // 内存 Mock，直接实现 ElectronAPI 接口
  : window.electronAPI; // 真实 Electron IPC（通过 preload.ts 桥接）
```

### Mock 数据内容

| 数据类型 | 内容 | 来源 |
|---------|------|------|
| 策略示例 | 5-10 个预设策略（覆盖各指标类型） | 手写 JSON |
| K线数据 | BTC/ETH/SOL 多个时间框架的历史数据 | 币安真实数据导出 |
| 实时价格 | 随机游走 + 趋势模拟 | `setInterval` 每 2 秒更新 |
| 信号记录 | 预设信号样例 + 基于模拟价格随机触发 | 简单条件判断 |
| 回测结果 | 完整的预计算结果（含交易明细、权益曲线） | 手写 JSON |
| 优化结果 | 多组参数组合结果 | 手写 JSON |

### Mock 实现要点

```typescript
// mocks/monitor-mock.ts
// 实时价格: setInterval 每 2 秒生成随机波动
// 信号触发: 基于模拟价格做简单阈值判断（模拟 RSI < 30 → 触发买入信号）
// 冷却机制: 记录上次触发时间，模拟冷却逻辑
// 事件模拟: 通过回调函数直接调用，模拟 IPC onSignal / onPriceUpdate

// mocks/backtest-mock.ts
// 回测执行: 返回预计算结果，模拟 3 秒延迟 + 分段进度回调（0% → 30% → 60% → 100%）
// 结果查询: 从预设 JSON 数据中返回

// mocks/strategy-mock.ts
// CRUD: 内存 Map 存储，模拟数据库操作
// 导入导出: JSON.stringify / JSON.parse
// 验证: 简单的结构检查（条件树非空、指标类型合法）
```

---

## 策略引擎逻辑

### 评估流程

```
定时触发（每根K线收线后）
  │
  ├─► 获取该 symbol + timeframe 的最新K线
  │     └─► 优先从 WebSocket 缓存读取，缺失则 REST API 补全
  │
  ├─► 计算所需指标（查缓存 → 缓存未命中则实时计算并写入缓存）
  │
  └─► 遍历条件树
        │
        ├─ ConditionGroup (AND): 所有 children 都为 true → true
        ├─ ConditionGroup (OR): 任一 child 为 true → true
        │
        └─ ConditionLeaf: 执行预设条件判断
              │
              ├─ 比较类: current_value op threshold
              │    例: RSI > 70 → rsi_value > 70
              │
              ├─ 穿越类: prev_a < prev_b && curr_a >= curr_b (上穿)
              │    例: EMA金叉 → prev_short_ema < prev_long_ema && curr_short_ema >= curr_long_ema
              │
              ├─ 背离类: 价格趋势方向 vs 指标趋势方向相反
              │    例: RSI底背离 → 价格创新低 但 RSI 未创新低（最近 N 周期内）
              │
              ├─ 倾斜类: 最近 N 周期的线性回归斜率
              │    例: EMA向上倾斜 → slope(ema[last_N]) > 0
              │
              ├─ 创新类: current === max/min(last_N)
              │    例: RSI创新高 → rsi_current === max(rsi[last_N])
              │
              ├─ 发散收敛类: 两线差值绝对值的变化趋势
              │    例: EMA发散 → abs(ema_a - ema_b) 逐周期增大
              │
              ├─ 区域类: price > upper / price < lower
              │    例: 价格在云层上方 → price > max(senkou_span_a, senkou_span_b)
              │
              └─ 翻转类: prev_state !== curr_state
                   例: SAR翻转 → prev_sar < prev_price && curr_sar > curr_price

  根节点结果 = true
    │
    ├─► 检查冷却时间（距上次触发是否超过 cooldown_seconds）
    │
    └─► 未冷却中 → 触发信号
          ├─► 写入 signal_records 表
          ├─► 通过 IPC 推送事件到前端 → Zustand store 更新 → UI 重渲染
          ├─► Telegram 推送（如已配置且启用）
          └─► 系统提示音（如已启用）
```

### 条件判断逻辑分类

| 类型 | 判断方式 | 示例 |
|------|---------|------|
| 比较类 | `indicator_value op threshold` | RSI > 70, ADX < 25 |
| 穿越类 | `prev_a < prev_b && curr_a >= curr_b`（上穿） | EMA金叉, DIF上穿DEA |
| 背离类 | 价格创新高/低但指标未同步 | RSI底背离, MACD顶背离 |
| 倾斜类 | 最近N周期线性回归斜率 | EMA向上倾斜 |
| 创新类 | `curr === max/min(last_N)` | RSI创新高, OBV创新低 |
| 发散收敛类 | 两线差值绝对值趋势 | EMA发散, 布林带收口 |
| 区域类 | `curr > upper` 或 `curr < lower` | 价格在云层上方 |
| 翻转类 | 前一值和当前值状态不同 | SAR翻转, Funding Rate由正转负 |

### 实时监控 vs 回测 条件语义统一

条件判定必须遵循统一的语义，避免实时监控、回测、用户理解三者不一致：

| 条件类型 | 实时监控 | 回测 | 说明 |
|---------|---------|------|------|
| 触及（如布林带触及上轨） | 当前价格 >= 上轨 | bar.close >= 上轨 | 仅用收盘价判断 |
| 突破（如价格突破唐奇安上轨） | 当前价格 > 上轨 | bar.close > 上轨 | 需等 K 线收盘确认 |
| 回撤至（如 VWAP 回撤） | 当前价格 <= VWAP | bar.close <= VWAP | 仅用收盘价 |
| 上穿/下穿（如 EMA 金叉） | curr > prev 且 crossover | bar.close > EMA 且 prev_close <= prev_EMA | 需等收盘确认 |
| 止损/止盈 | - | bar.high/low 触发 | 使用 bar 内极值 |

**关键原则**:
- **实时监控**: 基于当前未完成 K 线的实时价格（来自 WebSocket tick），只能判断"当前价格"是否触及
- **回测**: 基于已完成 K 线，使用 bar.close 判断信号（避免 look-ahead bias）
- **触及类条件**: 实时监控用当前价格，回测用 bar.close（等 K 线收盘确认）
- **穿越类条件**: 实时监控用 prev.crossover(curr)，回测用 bar.close 与上一 bar.close 比较

这样保证：同一策略在回测中的表现 ≈ 实时监控的实际表现（扣除滑点和延迟）

### 回测引擎逻辑

```
回测执行流程:

1. 加载历史K线数据（从 kline_cache 表）
   └─► 按时间排序，逐根遍历

2. 逐根K线执行策略评估

   ┌─────────────────────────────────────────────────────────────────────────┐
   │ 阶段划分（解决执行时机矛盾）:                                           │
   │                                                                         │
   │ bar[t] 收盘:                                                            │
   │   - 检测信号（条件树评估）                                              │
   │   - 若有持仓，SL/TP/移动止损检查使用 bar[t] 的 high/low                 │
   │                                                                         │
   │ bar[t+1] 开盘:                                                          │
   │   - 若上一阶段检测到信号且当时无持仓 → 执行开仓                          │
   │   - 开仓价格 = bar[t+1].open ± 滑点                                     │
   │   - 记录 entry，highest/lowest = entry_price                           │
   │   - 从 bar[t+1] 开始进行 SL/TP/移动止损检查（使用 bar[t+1] 的 high/low）│
   └─────────────────────────────────────────────────────────────────────────┘

   ├─ 无持仓 + 信号触发（bar[t] 收盘时检测）→ 在 bar[t+1] 开盘执行
   │    ├─ 计算开仓价格（bar[t+1].open ± 滑点）
   │    ├─ 计算手续费（按成交价计算）
   │    └─ 记录 entry，highest/lowest = entry_price
   │    注意: 开仓后立即检查 bar[t+1] 是否触发 SL/TP（使用 bar[t+1] 的 high/low）

   ├─ 持仓中 → 止损/止盈/移动止损检查（使用 bar[t] 内部 high/low）
   │    │
   │    ├─ SL/TP 碰撞优先级（悲观执行原则）:
   │    │    ⚠️ 同一 bar 内同时触发止损和止盈时，先执行止损
   │    │    原因: 防止回测收益虚高（宽幅震荡中极为常见）
   │    │    例: 做多时 bar.high >= 止盈价 且 bar.low <= 止损价 → 先触发止损
   │    │
   │    ├─ 做多持仓:
   │    │    ├─ 计算触发标志:
   │    │    │    stop_triggered = bar[t].low <= 止损价
   │    │    │    tp_triggered = bar[t].high >= 止盈价
   │    │    │
   │    │    ├─ 止损优先（悲观原则）:
   │    │    │    if stop_triggered:
   │    │    │      // 成交价 = max(止损价, bar[t].open)（跳空时不利价格）
   │    │    │      exit_price = max(止损价, bar[t].open)
   │    │    │      成交价 = min(exit_price, bar[t].open * (1 - slippage))  // 滑点
   │    │    │
   │    │    ├─ 止盈（止损未触发时）:
   │    │    │    else if tp_triggered:
   │    │    │      exit_price = min(止盈价, bar[t].open)  // 跳空时不利价格
   │    │    │      成交价 = min(exit_price, bar[t].open * (1 - slippage))
   │    │    │      if 移动止损模式B:
   │    │    │        trailing_stop_activated = true，highest = bar[t].high
   │    │    │
   │    │    └─ 移动止损（止盈未触发时）:
   │    │         if trailing_stop_activated && bar[t].low <= highest * (1 - trailing_stop_pct):
   │    │           exit_price = highest * (1 - trailing_stop_pct)
   │    │           成交价 = min(exit_price, bar[t].open * (1 - slippage))
   │    │
   │    ├─ 做空持仓:
   │    │    ├─ 计算触发标志:
   │    │    │    stop_triggered = bar[t].high >= 止损价
   │    │    │    tp_triggered = bar[t].low <= 止盈价
   │    │    │
   │    │    ├─ 止损优先（悲观原则）:
   │    │    │    if stop_triggered:
   │    │    │      exit_price = min(止损价, bar[t].open)  // 跳空时不利价格
   │    │    │      成交价 = max(exit_price, bar[t].open * (1 + slippage))
   │    │    │
   │    │    ├─ 止盈（止损未触发时）:
   │    │    │    else if tp_triggered:
   │    │    │      exit_price = max(止盈价, bar[t].open)  // 跳空时不利价格
   │    │    │      成交价 = max(exit_price, bar[t].open * (1 + slippage))
   │    │    │      if 移动止损模式B:
   │    │    │        trailing_stop_activated = true，lowest = bar[t].low
   │    │    │
   │    │    └─ 移动止损:
   │    │         if trailing_stop_activated && bar[t].high >= lowest * (1 + trailing_stop_pct):
   │    │           exit_price = lowest * (1 + trailing_stop_pct)
   │    │           成交价 = max(exit_price, bar[t].open * (1 + slippage))
   │    │
   │    └─ 反向信号处理（仅 trade_direction='both' 时有效）:
   │         ├─ 当前做多 + 收到卖出信号 → 平多仓，再开空仓
   │         ├─ 当前做空 + 收到买入信号 → 平空仓，再开多仓
   │         └─ trade_direction='long' 或 'short' 时忽略反向信号
   │
   ├─ 平仓后 → 计算平仓价格
   │    ├─ 计算手续费: 开仓手续费 + 平仓手续费（双向收费）
   │    │    开仓费 = entry_price * position_size * fee_rate
   │    │    平仓费 = exit_price * position_size * fee_rate
   │    │    默认 fee_rate = 0.001 (0.1%，币安 USDT 永续合约 maker 费率）
   │    ├─ 计算盈亏 = (exit_price - entry_price) * position_size - total_fees
   │    └─ 记录 trade，reset trailing_stop_activated

3. 遍历完毕 → 计算汇总指标

   ├─ 总收益率 = (最终权益 - 初始资金) / 初始资金
      初始资金: 默认 10,000 USDT（用户可配置）
      仓位: 每笔开仓使用初始资金的固定百分比（默认 10%）
      注意: 仓位基于初始资金（非复利），便于横向比较不同策略

   ├─ 年化收益率 = (1 + 总收益率) ^ (365 / 日历天数) - 1
      注意: 这是简单年化（非复合年化），未考虑期间资金闲置
      日历天数 = 回测结束日期 - 回测开始日期

   ├─ 夏普比率 = (mean(daily_returns) - risk_free_rate) / std(daily_returns) * sqrt(252)
      参数:
      - risk_free_rate: 默认 0.04 / 252（年化 4% 的日利率）
      - daily_returns: 每日收益率序列（每笔交易结束后按日计算权益变化）
      - 分母为 0 时（std=0），夏普比率返回 0
      UI 显示: 显示使用的无风险利率值

   ├─ 最大回撤 = max((peak - trough) / peak)，遍历权益曲线计算滚动峰值
      实现:
      peak = 0
      max_drawdown = 0
      for each equity in equity_curve:
        if equity > peak: peak = equity
        drawdown = (peak - equity) / peak
        if drawdown > max_drawdown: max_drawdown = drawdown

   ├─ 夏普比率 = (mean(daily_returns) - risk_free_rate) / std(daily_returns) * sqrt(252)
      注意:
      - 使用日收益率序列（非简单日收益）
      - risk_free_rate = 年化无风险利率 / 252（如 年化 4% → 日频 0.000159）
      - 分母为 0 时（std=0，即无波动），夏普比率返回 0
      - 年化夏普 = 日夏普 * sqrt(252)

   ├─ 胜率 = 盈利次数 / 总交易次数

   ├─ 盈亏比 = 平均盈利 / 平均亏损
      注意: 亏损为负值，计算时取绝对值

   └─ 最长连胜/连负 = 遍历 trade 列表统计

4. 存储 backtest_results + 通过 IPC 回传结果
```

#### 关键注意事项

| 场景 | 风险 | 处理方式 |
|------|------|---------|
| K 线重生线（Look-ahead） | 信号用当期收盘价判断，立即用当期收盘价成交 | 强制使用下一 bar 开盘价执行 |
| 止损/止盈日内穿透 | 极端行情中 high/low 穿透止损/止盈价 | 成交价按止损/止盈价，不按极端穿透价 |
| 收益率为零/负 | 无交易或全亏损 | 分母为零时返回特定值 |
| 波动率为零 | 无风险资产或恒定价格 | 夏普比率返回 0 |
| 浮点精度 | 多次计算累积误差 | 使用 Decimal.js 或银行家舍入 |

## 用户体验与交互细节

#### 新手引导流程

**形式**: 3 步嵌入式侧边抽屉引导（非模态），用户可随时关闭，不再弹出。

| 步骤 | 内容 | 触发位置 |
|------|------|---------|
| Step 1 | 欢迎页 → "盯盘侠帮你自动盯盘" + 功能亮点简介 | 首次启动时 |
| Step 2 | "创建你的第一个策略" → 引导进入策略编辑页，预填 RSI 超卖反弹示例 | 仪表盘新手卡片区 |
| Step 3 | "配置 Telegram 接收信号" → 跳转设置页 Telegram 区域 | 仪表盘新手卡片区 |

**完成判断**: 创建过至少 1 个策略 → 标记引导完成，不再显示新手卡片。设置页有"重新查看引导"入口。

**状态持久化**: `app_settings` 表存储 `onboarding_completed: boolean` + `onboarding_step: number`。

#### 操作反馈与确认机制

| 操作类型 | 确认方式 | 反馈 | 撤销 |
|---------|---------|------|------|
| 删除策略 | 二次确认对话框（显示策略名 + 关联任务数） | Toast "已删除，30天内可恢复" | 回收站恢复 |
| 删除监控任务 | 二次确认对话框 | Toast "已删除" | 无（任务配置已丢失） |
| 覆盖保存策略 | 若策略已修改但未保存，离开页面弹确认框 | — | 留在页面继续编辑 |
| 清空历史信号 | 二次确认 + 输入"确认清空" | Toast + 进度条 | 无 |
| 重置应用设置 | 二次确认对话框 | Toast "已恢复默认" | 手动重新配置 |
| 全量数据下载 | 底部进度条 + 取消按钮 | 进度条 + 当前币种 | 取消按钮 |
| 回测执行 | 进度条（分阶段）+ 取消按钮 | Toast "回测完成" | — |
| 参数优化 | 进度条 + 完成数/总数 + 取消按钮 | Toast "优化完成" | — |

#### 空状态设计

所有列表页在无数据时展示：

```
┌─────────────────────────────────────┐
│                                     │
│          📈 (插图/图标)              │
│                                     │
│     还没有策略                       │
│     创建你的第一个监控策略吧          │
│                                     │
│     [+ 新建策略]                    │
│     或 [导入策略]                    │
│                                     │
└─────────────────────────────────────┘
```

| 页面 | 空状态文案 | 快捷操作 |
|------|----------|---------|
| 策略列表 | "还没有策略" | [+ 新建策略] [导入示例策略] |
| 监控面板 | "还没有监控任务" | [+ 新建监控] |
| 回测历史 | "还没有回测记录" | [运行第一次回测] |
| 信号记录 | "暂无信号记录" | (启动监控后自动出现) |
| 回收站 | "回收站是空的" | — |

#### 键盘快捷键

| 快捷键 | 功能 | 适用页面 |
|--------|------|---------|
| `Cmd+N` | 新建策略 | 全局 |
| `Cmd+S` | 保存当前策略 | 策略编辑页 |
| `Cmd+Enter` | 运行回测 | 回测页 |
| `Cmd+P` | 快速搜索策略 | 全局（弹出搜索框） |
| `Cmd+,` | 打开设置 | 全局 |
| `Cmd+W` | 关闭当前弹窗/退出编辑 | 弹窗/编辑页 |
| `Space` | 暂停/恢复选中监控任务 | 监控面板（选中任务时） |
| `Delete` | 删除选中项 | 列表页（已选中项） |
| `Cmd+Z` | 撤销条件树操作 | 策略编辑页 |
| `Cmd+Shift+Z` | 重做条件树操作 | 策略编辑页 |

快捷键列表在设置页底部可查看，支持自定义（MVP 后考虑）。

**Undo/Redo 实现**: 使用 Zustand 的 `useUndo` 或 immer + 自定义 middleware，存储条件树的快照而非操作序列。每次条件变更保存到历史栈，最多保留 50 个快照。

---

### 17. 数据安全与隐私

#### 敏感数据加密

| 数据 | 加密方式 | 说明 |
|------|---------|------|
| Telegram Bot Token | AES-256-GCM 加密存 SQLite | 加密密钥存平台安全存储（macOS Keychain / Windows DPAPI） |
| Telegram Chat ID | AES-256-GCM 加密存 SQLite | 同上 |
| 其他设置 | 明文 JSON | 非敏感数据 |

**密钥管理**:
- macOS: 系统 Keychain（`keytar` 库）存储 AES 加密密钥
- Windows: DPAPI（`node-dpapi` 或 `win-dpapi`）保护密钥

**AES-256-GCM 实现细节**:
```typescript
// 加密流程
1. 生成随机 IV (12 bytes)
2. 使用 PBKDF2 从用户主密码派生 256-bit 密钥（若需要）
3. AES-256-GCM 加密: ciphertext = AES_GCM_encrypt(plaintext, key, IV)
4. 存储格式: IV || ciphertext || auth_tag
5. 存储到 SQLite: base64(IV || ciphertext || auth_tag)

// 解密流程
1. 从 SQLite 读取 base64 数据
2. 解析: IV (12 bytes), ciphertext, auth_tag
3. AES-256-GCM 解密: plaintext = AES_GCM_decrypt(ciphertext, key, IV, auth_tag)
4. 验证 auth_tag，失败则抛出异常
```

**Electron 安全配置**:
```javascript
// BrowserWindow 创建时启用安全选项
new BrowserWindow({
  webPreferences: {
    contextIsolation: true,    // 启用上下文隔离
    nodeIntegration: false,    // 禁用 Node.js 集成
    sandbox: true,            // 启用沙箱
    preload: 'preload.js',   // 使用预加载脚本暴露安全 API
    webSecurity: true,       // 启用 Web 安全策略
    allowRunningInsecureContent: false,  // 禁止加载混合内容
  }
});

// CSP (Content Security Policy) 配置建议
// default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';
```
```
- 降级方案: 若平台安全存储不可用:
  - **方案 A（推荐）**: fail closed — 提示用户"Telegram 通知功能暂不可用"，禁用加密配置保存
  - **方案 B**: 要求用户设置应用主密码，从密码派生密钥（PBKDF2）
  - **不推荐**: machine-id + 盐值（本质是混淆，非真正加密，防不住有意窃取）
- 密钥不写入配置文件，不写入数据库明文字段

**加密实现位置**: `electron/services/crypto.ts`

```typescript
// 加密流程
encrypt(plaintext: string): string
  → 密钥 = getKeyFromPlatformSecureStorage() || deriveFromMachineId()
  → iv = crypto.randomBytes(12)
  → cipher = aes-256-gcm(key, iv)
  → return base64(iv + authTag + ciphertext)

decrypt(ciphertext: string): string
  → 解析 iv, authTag, ciphertext
  → 密钥同上
  → return plaintext
```

#### 数据导出安全

- 导出内容**不包含** Telegram Token / Chat ID 等敏感配置
- 导出范围可选:
  - 策略结构（默认包含条件树 + 参数）
  - 策略参数可选脱敏（参数值替换为推荐默认值）
  - 信号记录 / 回测历史（独立导出，不包含策略内部细节）
- 导出文件头标注: `"export_version": "1.0", "sensitive_excluded": true`

#### 更新签名校验

| 检查项 | 方式 |
|--------|------|
| 更新包完整性 | SHA-256 校验和（对比服务端发布的 hash） |
| 更新包来源 | 代码签名（macOS Developer ID / Windows 代码签名证书） |
| 传输安全 | HTTPS only (GitHub Releases) |
| 降级保护 | 不允许降级安装（版本号校验） |

---

### 18. 错误处理与系统健壮性

#### 用户友好的错误呈现

| 错误级别 | 展示方式 | 内容 |
|---------|---------|------|
| 轻微 (Toast) | 右下角 Toast，3 秒自动消失 | 错误码 + 一句话中文描述 |
| 一般 (Dialog) | 居中弹窗，需手动关闭 | 错误码 + 中文描述 + 操作建议 + [查看详情] |
| 严重 (Full) | 全屏错误页 | 错误码 + 描述 + [查看日志] [报告问题] [重试] [重置] |

**[查看详情]** 展开显示技术细节（原始错误信息、堆栈摘要）。

**[报告问题]** 将错误日志复制到剪贴板（不自动发送，用户自行粘贴到 GitHub Issue / Telegram 群）。

#### 日志查看器

设置页内置日志查看器：

```
┌─────────────────────────────────────────────────┐
│ 日志查看器                      [导出] [清理]    │
│ ┌──────────────────────────────────────────────┐ │
│ │ 级别: [全部▼] [Error] [Warning]              │ │
│ │ 时间: [最近24h ▼]   搜索: [____________]     │ │
│ └──────────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────────┐ │
│ │ ⚠️ W 14:30:25 API请求限流，等待15s重试       │ │
│ │    → monitor:fetch_klines | BTC/USDT 4h     │ │
│ │ ✕  E 14:28:10 E401 Telegram推送失败          │ │
│ │    → Chat ID无效 | [重新推送]                │ │
│ │ ⚠️ W 14:25:01 WebSocket断线，切换REST轮询    │ │
│ │    → 重连尝试 1/5                            │ │
│ └──────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

- 筛选: 按级别 (Error / Warning)、时间范围、模块
- 搜索: 全文搜索日志消息
- 操作: [导出 CSV] [清理 N 天前的日志]
- 自动滚动到最新

#### 灾难恢复矩阵

| 场景 | 检测方式 | 恢复策略 |
|------|---------|---------|
| SQLite 数据库损坏 | 启动时 `PRAGMA integrity_check` | 自动尝试从最近备份恢复 → 失败则提示用户选择: 导出剩余数据 / 重置 |
| 配置文件损坏 | 启动时 JSON.parse 校验 | 忽略损坏项，使用默认值，Toast 提示"部分设置已重置" |
| K线缓存数据不一致 | 校验时间戳连续性 | 自动标记不一致范围，下次启动时重新下载该段数据 |
| 应用崩溃后重启 | 检查 `crash_marker` 文件 | 显示"上次异常退出"对话框，提供: [查看崩溃日志] [恢复监控任务] [忽略] |
| 监控任务状态不一致 | 启动时校验任务表 vs 实际运行状态 | 崩溃标记存在时，将所有 `status=running` 的任务重置为 `stopped`，由用户决定是否恢复 |

---

### 19. 性能与可扩展性

#### 大数据量 UI 性能

| 场景 | 数据量级 | 优化策略 |
|------|---------|---------|
| 策略列表 | < 200 条 | 常规渲染，前端内存筛选 |
| 监控任务列表 | ≤ 50 个 | 常规渲染 |
| 信号记录列表 | 数万条 | **虚拟滚动** (`@tanstack/virtual`)，每页加载 100 条，滚动加载 |
| 回测交易明细 | 数千条 | 分页表格 (每页 50 条)，支持跳页 |
| 回测历史列表 | 数百条 | 常规渲染 |
| K线图表 | 数万根 | lightweight-charts 内置按视口渲染 |
| 参数优化结果 | 数百到数千行 | **虚拟滚动**表格 + 分页 |

**搜索与筛选**: 输入 300ms 防抖后执行筛选，避免频繁重渲染。

#### 内存管理

| 组件 | 内存策略 |
|------|---------|
| 指标预计算缓存 | LRU 策略，默认最多缓存 50 组参数 × 10000 条数据点 ≈ 200MB 上限 |
| WebSocket 缓冲 | 环形缓冲区，只保留最近 1000 条 tick 数据（仅用于 UI 显示） |
| 回测执行 | 单次回测数据加载后逐根处理，处理完的 K 线释放。权益曲线数组不超过 50000 点（超过则降采样） |
| 策略引擎 | 每个监控任务独立上下文，指标计算只保留最近 N 根 K 线（N = 指标所需最大周期 × 2） |

**内存监控**: 主进程每 30 秒检查 `process.memoryUsage().heapUsed`，超过 500MB 时:
1. 清理最久未使用的指标缓存
2. Toast 提示用户
3. 超过 800MB 时自动暂停非关键任务

#### 后台资源占用

| 状态 | CPU | 内存 | 策略 |
|------|-----|------|------|
| 前台活跃 | 无限制 | 无限制 | — |
| 最小化到托盘 | ≤ 2% | ≤ 200MB | WebSocket 保持连接，价格推送频率降为每 10 秒一次 |
| 后台长时间运行 | ≤ 1% | ≤ 150MB | 仅在 K 线收线时计算，其余时间 idle |

---

### 20. 品牌与视觉资产

#### 应用图标

- **macOS**: `.icns` 格式，包含 16x16 ~ 1024x1024 全尺寸
- **设计元素**: 机器人 + 放大镜/望远镜 + 上涨箭头组合，体现"自动盯盘"
- **主色调**: 科技蓝 (#3B82F6) + 涨色绿 (#10B981)

#### 启动画面

- macOS 使用 `electron-builder` 的 `splash` 配置
- 显示 2 秒应用 Logo + 加载动画，主进程初始化完成后自动关闭

#### 设计令牌 (Design Tokens)

```typescript
// tailwind.config.ts 中定义的品牌令牌
{
  theme: {
    extend: {
      colors: {
        brand: {
          primary: '#3B82F6',    // 主色（操作按钮、选中态）
          success: '#10B981',    // 涨/买入/成功
          danger: '#EF4444',     // 跌/卖出/危险操作
          warning: '#F59E0B',    // 警告/冷却中
        },
        surface: {
          bg: '#0F172A',         // 主背景
          card: '#1E293B',       // 卡片背景
          elevated: '#334155',   // 浮层背景
          border: '#475569',     // 边框
        },
        text: {
          primary: '#F8FAFC',    // 主文字
          secondary: '#94A3B8',  // 次要文字
          muted: '#64748B',      // 弱化文字
        },
      },
      borderRadius: {
        sm: '4px',
        md: '8px',
        lg: '12px',
      },
      fontSize: {
        xs: '12px',
        sm: '13px',
        base: '14px',
        lg: '16px',
        xl: '20px',
      },
    },
  },
}
```

---

### 21. 测试与质量保证

#### 测试策略

| 层级 | 覆盖目标 | 框架 |
|------|---------|------|
| 单元测试 | 策略引擎核心逻辑 ≥ 80% | Vitest |
| 单元测试 | 回测引擎核心逻辑 ≥ 80% | Vitest |
| 单元测试 | 指标计算函数 ≥ 90% | Vitest |
| 单元测试 | 工具函数 / 数据转换 ≥ 70% | Vitest |
| 组件测试 | 关键交互组件（条件树编辑器、策略表单） | Vitest + Testing Library |
| 集成测试 | IPC 通信（preload → 主进程 → 数据库） | Vitest |
| 端到端测试 | 核心流程（创建策略 → 启动监控 → 收到信号） | Playwright (Electron) |

#### 测试数据集

| 数据集 | 用途 | 内容 |
|--------|------|------|
| 正常行情 | 基本功能验证 | BTC 6个月 4h K线，含趋势+震荡 |
| 极端行情 | 边界测试 | 2020.3.12 暴跌、2021 牛市顶点数据 |
| 数据缺失 | 容错测试 | 刻意删除部分K线，测试数据补全逻辑 |
| 高频震荡 | 性能测试 | 1m K线 1 年数据，测试回测性能 |
| 空白数据 | 空状态测试 | 无任何K线数据 |
| 网络异常 | 错误处理测试 | Mock 超时、限流、断线场景 |

#### CI/CD 检查项

- 每次 PR: `pnpm lint` + `pnpm type-check` + `pnpm test`
- 合并前: `pnpm test:e2e` (Playwright)
- 发布前: `pnpm build` + 手动冒烟测试

---

### 22. 部署与维护

#### 崩溃报告

**方案**: **本地记录为主**，不集成第三方服务（本地优先原则）。

| 环节 | 处理 |
|------|------|
| 崩溃捕获 | Electron `crashReporter` + `process.on('uncaughtException')` |
| 本地存储 | 写入 `{userData}/crash-reports/` |
| 用户通知 | 下次启动弹窗: "上次异常退出" + [查看日志] [复制报告] |
| 分享方式 | 用户手动复制日志文件到 GitHub Issue / Telegram 群 |

**不使用 Sentry 等云服务**，符合本地优先原则。

#### 数据库版本迁移

```typescript
// electron/services/database.ts
const DB_VERSION = 2; // 当前版本号

function migrateDatabase(db: Database, fromVersion: number) {
  for (let v = fromVersion + 1; v <= DB_VERSION; v++) {
    switch (v) {
      case 2:
        // 从 v1 迁移到 v2 的逻辑
        db.exec(`
          ALTER TABLE strategies ADD COLUMN last_modified INTEGER;
          ALTER TABLE monitor_tasks ADD COLUMN last_run INTEGER;
        `);
        break;
    }
  }
}
```

#### 更新机制

| 环节 | 处理 |
|------|------|
| 检查频率 | 启动时检查一次 + 每 24 小时检查一次 |
| 更新源 | GitHub Releases (macOS: zip, Windows: nsis) |
| 下载 | 后台下载，进度存本地，完成后提示用户 |
| 安装 | 用户确认后，macOS 自动替换 app，Windows 关闭重启后安装 |
| 版本号格式 | semver (`1.0.0`)，CI 构建时写入 `package.json` 的 `version` |
| 自动回滚 | 降级安装不允许（版本号校验），出问题时用户需手动重装旧版 |

#### 维护任务

| 任务 | 频率 | 操作 |
|------|------|------|
| 日志压缩/归档 | 每周 | 超过 7 天的日志压缩为 `.gz` |
| 临时文件清理 | 每次启动 | 清理 `Temp/` 下的过期缓存 |
| 缓存清理 | 每月 或 用户主动触发 | 清理 K 线缓存（`indicator_cache` 表） |
| 数据库 Vacuum | 每月 或 每次更新后 | `PRAGMA vacuum` 回收删除行占用的空间 |
| 备份验证 | 每季度 | 尝试验证备份文件完整性（非破坏性） |

#### 安全维护

| 任务 | 频率 | 操作 |
|------|------|------|
| 依赖安全审计 | 每次 `pnpm install` | `pnpm audit` 自动运行 |
| 关键依赖更新 | 按需 | `better-sqlite3` / `ccxt` / `electron` 升级前需手动测试 |
| 版本锁定 | 每次构建 | `pnpm-lock.yaml` 提交到 Git，确保可重复构建 |
| 版本选择 | 主版本号锁定（如 `ccxt@^4.x`），安全补丁自动更新 |
| 安全审计 | 每次 `pnpm install` 后运行 `pnpm audit`，CI 中强制 |
| 关键依赖 | `better-sqlite3` / `ccxt` / `electron` 升级前需手动测试 |
| 原生模块 | `better-sqlite3` 需要 `electron-rebuild`，在 `postinstall` 中处理 |

#### 新手引导流程

**形式**: 3 步嵌入式侧边抽屉引导（非模态），用户可随时关闭，不再弹出。

| 步骤 | 内容 | 触发位置 |
|------|------|---------|
| Step 1 | 欢迎页 → "盯盘侠帮你自动盯盘" + 功能亮点简介 | 首次启动时 |
| Step 2 | "创建你的第一个策略" → 引导进入策略编辑页，预填 RSI 超卖反弹示例 | 仪表盘新手卡片区 |
| Step 3 | "配置 Telegram 接收信号" → 跳转设置页 Telegram 区域 | 仪表盘新手卡片区 |

**完成判断**: 创建过至少 1 个策略 → 标记引导完成，不再显示新手卡片。设置页有"重新查看引导"入口。

**状态持久化**: `app_settings` 表存储 `onboarding_completed: boolean` + `onboarding_step: number`。

#### 操作反馈与确认机制

| 操作类型 | 确认方式 | 反馈 | 撤销 |
|---------|---------|------|------|
| 删除策略 | 二次确认对话框（显示策略名 + 关联任务数） | Toast "已删除，30天内可恢复" | 回收站恢复 |
| 删除监控任务 | 二次确认对话框 | Toast "已删除" | 无（任务配置已丢失） |
| 覆盖保存策略 | 若策略已修改但未保存，离开页面弹确认框 | — | 留在页面继续编辑 |
| 清空历史信号 | 二次确认 + 输入"确认清空" | Toast + 进度条 | 无 |
| 重置应用设置 | 二次确认对话框 | Toast "已恢复默认" | 手动重新配置 |
| 全量数据下载 | 底部进度条 + 取消按钮 | 进度条 + 当前币种 | 取消按钮 |
| 回测执行 | 进度条（分阶段）+ 取消按钮 | Toast "回测完成" | — |
| 参数优化 | 进度条 + 完成数/总数 + 取消按钮 | Toast "优化完成" | — |

#### 空状态设计

所有列表页在无数据时展示：

```
┌─────────────────────────────────────┐
│                                     │
│          📈 (插图/图标)              │
│                                     │
│     还没有策略                       │
│     创建你的第一个监控策略吧          │
│                                     │
│     [+ 新建策略]                    │
│     或 [导入策略]                    │
│                                     │
└─────────────────────────────────────┘
```

| 页面 | 空状态文案 | 快捷操作 |
|------|----------|---------|
| 策略列表 | "还没有策略" | [+ 新建策略] [导入示例策略] |
| 监控面板 | "还没有监控任务" | [+ 新建监控] |
| 回测历史 | "还没有回测记录" | [运行第一次回测] |
| 信号记录 | "暂无信号记录" | (启动监控后自动出现) |
| 回收站 | "回收站是空的" | — |

#### 键盘快捷键

| 快捷键 | 功能 | 适用页面 |
|--------|------|---------|
| `Cmd+N` | 新建策略 | 全局 |
| `Cmd+S` | 保存当前策略 | 策略编辑页 |
| `Cmd+Enter` | 运行回测 | 回测页 |
| `Cmd+P` | 快速搜索策略 | 全局（弹出搜索框） |
| `Cmd+,` | 打开设置 | 全局 |
| `Cmd+W` | 关闭当前弹窗/退出编辑 | 弹窗/编辑页 |
| `Space` | 暂停/恢复选中监控任务 | 监控面板（选中任务时） |
| `Delete` | 删除选中项 | 列表页（已选中项） |
| `Cmd+Z` | 撤销条件树操作 | 策略编辑页 |
| `Cmd+Shift+Z` | 重做条件树操作 | 策略编辑页 |

快捷键列表在设置页底部可查看，支持自定义（MVP 后考虑）。

**Undo/Redo 实现**: 使用 Zustand 的 `useUndo` 或 immer + 自定义 middleware，存储条件树的快照而非操作序列。每次条件变更保存到历史栈，最多保留 50 个快照。

---

### 17. 数据安全与隐私

#### 敏感数据加密

| 数据 | 加密方式 | 说明 |
|------|---------|------|
| Telegram Bot Token | AES-256-GCM 加密存 SQLite | 加密密钥存平台安全存储（macOS Keychain / Windows DPAPI） |
| Telegram Chat ID | AES-256-GCM 加密存 SQLite | 同上 |
| 其他设置 | 明文 JSON | 非敏感数据 |

**密钥管理**:
- macOS: 系统 Keychain（`keytar` 库）存储 AES 加密密钥
- Windows: DPAPI（`node-dpapi` 或 `win-dpapi`）保护密钥

**AES-256-GCM 实现细节**:
```typescript
// 加密流程
1. 生成随机 IV (12 bytes)
2. 使用 PBKDF2 从用户主密码派生 256-bit 密钥（若需要）
3. AES-256-GCM 加密: ciphertext = AES_GCM_encrypt(plaintext, key, IV)
4. 存储格式: IV || ciphertext || auth_tag
5. 存储到 SQLite: base64(IV || ciphertext || auth_tag)

// 解密流程
1. 从 SQLite 读取 base64 数据
2. 解析: IV (12 bytes), ciphertext, auth_tag
3. AES-256-GCM 解密: plaintext = AES_GCM_decrypt(ciphertext, key, IV, auth_tag)
4. 验证 auth_tag，失败则抛出异常
```

**Electron 安全配置**:
```javascript
// BrowserWindow 创建时启用安全选项
new BrowserWindow({
  webPreferences: {
    contextIsolation: true,    // 启用上下文隔离
    nodeIntegration: false,    // 禁用 Node.js 集成
    sandbox: true,            // 启用沙箱
    preload: 'preload.js',   // 使用预加载脚本暴露安全 API
    webSecurity: true,       // 启用 Web 安全策略
    allowRunningInsecureContent: false,  // 禁止加载混合内容
  }
});

// CSP (Content Security Policy) 配置建议
// default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';
```
```
- 降级方案: 若平台安全存储不可用:
  - **方案 A（推荐）**: fail closed — 提示用户"Telegram 通知功能暂不可用"，禁用加密配置保存
  - **方案 B**: 要求用户设置应用主密码，从密码派生密钥（PBKDF2）
  - **不推荐**: machine-id + 盐值（本质是混淆，非真正加密，防不住有意窃取）
- 密钥不写入配置文件，不写入数据库明文字段

**加密实现位置**: `electron/services/crypto.ts`

```typescript
// 加密流程
encrypt(plaintext: string): string
  → 密钥 = getKeyFromPlatformSecureStorage() || deriveFromMachineId()
  → iv = crypto.randomBytes(12)
  → cipher = aes-256-gcm(key, iv)
  → return base64(iv + authTag + ciphertext)

decrypt(ciphertext: string): string
  → 解析 iv, authTag, ciphertext
  → 密钥同上
  → return plaintext
```

#### 数据导出安全

- 导出内容**不包含** Telegram Token / Chat ID 等敏感配置
- 导出范围可选:
  - 策略结构（默认包含条件树 + 参数）
  - 策略参数可选脱敏（参数值替换为推荐默认值）
  - 信号记录 / 回测历史（独立导出，不包含策略内部细节）
- 导出文件头标注: `"export_version": "1.0", "sensitive_excluded": true`

#### 更新签名校验

| 检查项 | 方式 |
|--------|------|
| 更新包完整性 | SHA-256 校验和（对比服务端发布的 hash） |
| 更新包来源 | 代码签名（macOS Developer ID / Windows 代码签名证书） |
| 传输安全 | HTTPS only (GitHub Releases) |
| 降级保护 | 不允许降级安装（版本号校验） |

---

### 18. 错误处理与系统健壮性

#### 用户友好的错误呈现

| 错误级别 | 展示方式 | 内容 |
|---------|---------|------|
| 轻微 (Toast) | 右下角 Toast，3 秒自动消失 | 错误码 + 一句话中文描述 |
| 一般 (Dialog) | 居中弹窗，需手动关闭 | 错误码 + 中文描述 + 操作建议 + [查看详情] |
| 严重 (Full) | 全屏错误页 | 错误码 + 描述 + [查看日志] [报告问题] [重试] [重置] |

**[查看详情]** 展开显示技术细节（原始错误信息、堆栈摘要）。

**[报告问题]** 将错误日志复制到剪贴板（不自动发送，用户自行粘贴到 GitHub Issue / Telegram 群）。

#### 日志查看器

设置页内置日志查看器：

```
┌─────────────────────────────────────────────────┐
│ 日志查看器                      [导出] [清理]    │
│ ┌──────────────────────────────────────────────┐ │
│ │ 级别: [全部▼] [Error] [Warning]              │ │
│ │ 时间: [最近24h ▼]   搜索: [____________]     │ │
│ └──────────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────────┐ │
│ │ ⚠️ W 14:30:25 API请求限流，等待15s重试       │ │
│ │    → monitor:fetch_klines | BTC/USDT 4h     │ │
│ │ ✕  E 14:28:10 E401 Telegram推送失败          │ │
│ │    → Chat ID无效 | [重新推送]                │ │
│ │ ⚠️ W 14:25:01 WebSocket断线，切换REST轮询    │ │
│ │    → 重连尝试 1/5                            │ │
│ └──────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

- 筛选: 按级别 (Error / Warning)、时间范围、模块
- 搜索: 全文搜索日志消息
- 操作: [导出 CSV] [清理 N 天前的日志]
- 自动滚动到最新

#### 灾难恢复矩阵

| 场景 | 检测方式 | 恢复策略 |
|------|---------|---------|
| SQLite 数据库损坏 | 启动时 `PRAGMA integrity_check` | 自动尝试从最近备份恢复 → 失败则提示用户选择: 导出剩余数据 / 重置 |
| 配置文件损坏 | 启动时 JSON.parse 校验 | 忽略损坏项，使用默认值，Toast 提示"部分设置已重置" |
| K线缓存数据不一致 | 校验时间戳连续性 | 自动标记不一致范围，下次启动时重新下载该段数据 |
| 应用崩溃后重启 | 检查 `crash_marker` 文件 | 显示"上次异常退出"对话框，提供: [查看崩溃日志] [恢复监控任务] [忽略] |
| 监控任务状态不一致 | 启动时校验任务表 vs 实际运行状态 | 崩溃标记存在时，将所有 `status=running` 的任务重置为 `stopped`，由用户决定是否恢复 |

---

### 19. 性能与可扩展性

#### 大数据量 UI 性能

| 场景 | 数据量级 | 优化策略 |
|------|---------|---------|
| 策略列表 | < 200 条 | 常规渲染，前端内存筛选 |
| 监控任务列表 | ≤ 50 个 | 常规渲染 |
| 信号记录列表 | 数万条 | **虚拟滚动** (`@tanstack/virtual`)，每页加载 100 条，滚动加载 |
| 回测交易明细 | 数千条 | 分页表格 (每页 50 条)，支持跳页 |
| 回测历史列表 | 数百条 | 常规渲染 |
| K线图表 | 数万根 | lightweight-charts 内置按视口渲染 |
| 参数优化结果 | 数百到数千行 | **虚拟滚动**表格 + 分页 |

**搜索与筛选**: 输入 300ms 防抖后执行筛选，避免频繁重渲染。

#### 内存管理

| 组件 | 内存策略 |
|------|---------|
| 指标预计算缓存 | LRU 策略，默认最多缓存 50 组参数 × 10000 条数据点 ≈ 200MB 上限 |
| WebSocket 缓冲 | 环形缓冲区，只保留最近 1000 条 tick 数据（仅用于 UI 显示） |
| 回测执行 | 单次回测数据加载后逐根处理，处理完的 K 线释放。权益曲线数组不超过 50000 点（超过则降采样） |
| 策略引擎 | 每个监控任务独立上下文，指标计算只保留最近 N 根 K 线（N = 指标所需最大周期 × 2） |

**内存监控**: 主进程每 30 秒检查 `process.memoryUsage().heapUsed`，超过 500MB 时:
1. 清理最久未使用的指标缓存
2. Toast 提示用户
3. 超过 800MB 时自动暂停非关键任务

#### 后台资源占用

| 状态 | CPU | 内存 | 策略 |
|------|-----|------|------|
| 前台活跃 | 无限制 | 无限制 | — |
| 最小化到托盘 | ≤ 2% | ≤ 200MB | WebSocket 保持连接，价格推送频率降为每 10 秒一次 |
| 后台长时间运行 | ≤ 1% | ≤ 150MB | 仅在 K 线收线时计算，其余时间 idle |

---

### 20. 品牌与视觉资产

#### 应用图标

- **macOS**: `.icns` 格式，包含 16x16 ~ 1024x1024 全尺寸
- **设计元素**: 机器人 + 放大镜/望远镜 + 上涨箭头组合，体现"自动盯盘"
- **主色调**: 科技蓝 (#3B82F6) + 涨色绿 (#10B981)

#### 启动画面

- macOS 使用 `electron-builder` 的 `splash` 配置
- 显示 2 秒应用 Logo + 加载动画，主进程初始化完成后自动关闭

#### 设计令牌 (Design Tokens)

```typescript
// tailwind.config.ts 中定义的品牌令牌
{
  theme: {
    extend: {
      colors: {
        brand: {
          primary: '#3B82F6',    // 主色（操作按钮、选中态）
          success: '#10B981',    // 涨/买入/成功
          danger: '#EF4444',     // 跌/卖出/危险操作
          warning: '#F59E0B',    // 警告/冷却中
        },
        surface: {
          bg: '#0F172A',         // 主背景
          card: '#1E293B',       // 卡片背景
          elevated: '#334155',   // 浮层背景
          border: '#475569',     // 边框
        },
        text: {
          primary: '#F8FAFC',    // 主文字
          secondary: '#94A3B8',  // 次要文字
          muted: '#64748B',      // 弱化文字
        },
      },
      borderRadius: {
        sm: '4px',
        md: '8px',
        lg: '12px',
      },
      fontSize: {
        xs: '12px',
        sm: '13px',
        base: '14px',
        lg: '16px',
        xl: '20px',
      },
    },
  },
}
```

---

### 21. 测试与质量保证

#### 测试策略

| 层级 | 覆盖目标 | 框架 |
|------|---------|------|
| 单元测试 | 策略引擎核心逻辑 ≥ 80% | Vitest |
| 单元测试 | 回测引擎核心逻辑 ≥ 80% | Vitest |
| 单元测试 | 指标计算函数 ≥ 90% | Vitest |
| 单元测试 | 工具函数 / 数据转换 ≥ 70% | Vitest |
| 组件测试 | 关键交互组件（条件树编辑器、策略表单） | Vitest + Testing Library |
| 集成测试 | IPC 通信（preload → 主进程 → 数据库） | Vitest |
| 端到端测试 | 核心流程（创建策略 → 启动监控 → 收到信号） | Playwright (Electron) |

#### 测试数据集

| 数据集 | 用途 | 内容 |
|--------|------|------|
| 正常行情 | 基本功能验证 | BTC 6个月 4h K线，含趋势+震荡 |
| 极端行情 | 边界测试 | 2020.3.12 暴跌、2021 牛市顶点数据 |
| 数据缺失 | 容错测试 | 刻意删除部分K线，测试数据补全逻辑 |
| 高频震荡 | 性能测试 | 1m K线 1 年数据，测试回测性能 |
| 空白数据 | 空状态测试 | 无任何K线数据 |
| 网络异常 | 错误处理测试 | Mock 超时、限流、断线场景 |

#### CI/CD 检查项

- 每次 PR: `pnpm lint` + `pnpm type-check` + `pnpm test`
- 合并前: `pnpm test:e2e` (Playwright)
- 发布前: `pnpm build` + 手动冒烟测试

---

### 22. 部署与维护

#### 崩溃报告

**方案**: **本地记录为主**，不集成第三方服务（本地优先原则）。

| 环节 | 处理 |
|------|------|
| 崩溃捕获 | Electron `crashReporter` + `process.on('uncaughtException')` |
| 本地存储 | 写入 `{userData}/crash-reports/` |
| 用户通知 | 下次启动弹窗: "上次异常退出" + [查看日志] [复制报告] |
| 分享方式 | 用户手动复制日志文件到 GitHub Issue / Telegram 群 |

**不使用 Sentry 等云服务**，符合本地优先原则。

#### 数据库版本迁移

```typescript
// electron/services/database.ts
const DB_VERSION = 2; // 当前版本号

function migrateDatabase(db: Database, fromVersion: number) {
  for (let v = fromVersion + 1; v <= DB_VERSION; v++) {
    switch (v) {
      case 2:
        // 示例: v1 → v2 添加 optimization 字段
        db.exec('ALTER TABLE backtest_results ADD COLUMN optimization_id TEXT');
        break;
      // case 3: ...
    }
  }
  db.pragma(`user_version = ${DB_VERSION}`);
}

// 启动时
const currentVersion = db.pragma('user_version')[0].user_version;
if (currentVersion < DB_VERSION) {
  // 1. 备份数据库
  backupDatabase();
  // 2. 执行迁移
  migrateDatabase(db, currentVersion);
  // 3. 验证完整性
  db.pragma('integrity_check');
}
```

**迁移原则**:
- 只增不改删（新增字段/表，不删旧字段）
- 迁移前自动备份
- 迁移失败自动回滚（使用备份）
- `user_version` pragma 追踪当前版本

#### 依赖项管理

| 策略 | 说明 |
|------|------|
| 版本锁定 | `pnpm-lock.yaml` 提交到 Git，确保可重复构建 |
| 版本选择 | 主版本号锁定（如 `ccxt@^4.x`），安全补丁自动更新 |
| 安全审计 | 每次 `pnpm install` 后运行 `pnpm audit`，CI 中强制 |
| 关键依赖 | `better-sqlite3` / `ccxt` / `electron` 升级前需手动测试 |
| 原生模块 | `better-sqlite3` 需要 `electron-rebuild`，在 `postinstall` 中处理 |
