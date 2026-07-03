# 种子城市选择与抽样方案报告（AI 社交「主动匹配」内测）

> 基础池：P1「分身复刻+社交」候选 531,513 人（近30天活跃≥15天 且 私信≥5 或 合盘≥2，女 479,951 / 男 51,119 / 未知 443）。
> 数据窗口 2026-06-03 ~ 2026-07-02，基于 07-02 快照。生成日期 2026-07-03。

## 一、字段口径（ODM 确认）

| 需求 | 选用字段 | 说明 |
|---|---|---|
| 城市 | `dim_cc_user_view_df.city` | **用户最近一次活跃时归属的城市**（定位数据，每日刷新）。P1 内覆盖率 100%，标准中文城市名，脏值仅 0.1%。对同城匹配，"人实际在哪"比自填现居地更可操作 |
| 城市（弃用口径） | `dim_cc_user_file_df.current_place`（档案自填现居地） | 默认值污染严重：全表"北京"355 万 +"北京市 东城区"165 万条堆积（用户建档不改默认值），用它选城北京会被系统性高估，仅作参考 |
| 性别 | `dim_cc_user_view_df.gender` | 权威维表值（dws 表 gender 是 REDIRECT，已改写 JOIN） |
| 年龄 | `dim_cc_user_view_df.birthday` 现算（截至 2026-07-02 周岁） | 表内 age 字段计算时点不明，弃用；birthday 缺失仅 0.08% |
| MBTI | `dim_cc_user_view_df.mbti` | ENUM 标准 16 型，无脏值；P1 全池填写率 30.7%。档案表 file_mbti（自由文本几百种脏值）弃用。数仓无独立 MBTI 测试记录表 |

## 二、Top20 城市分布（按总人数降序）

| 排名 | 城市 | 总人数 | 女 | 男 | 男性占比 |
|---|---|---:|---:|---:|---:|
| 1 | 北京 | 27,942 | 25,334 | **2,594** | 9.3% |
| 2 | 上海 | 26,181 | 23,853 | **2,311** | 8.8% |
| 3 | 广州 | 21,229 | 18,924 | 2,279 | 10.7% |
| 4 | 杭州 | 17,633 | 16,015 | 1,612 | 9.1% |
| 5 | 成都 | 17,050 | 15,431 | 1,611 | 9.4% |
| 6 | 深圳 | 13,525 | 12,218 | 1,291 | 9.5% |
| 7 | 郑州 | 11,919 | 10,776 | 1,134 | 9.5% |
| 8 | 济南 | 10,343 | 9,278 | 1,049 | 10.1% |
| 9 | 南京 | 10,224 | 9,207 | 1,010 | 9.9% |
| 10 | 天津 | 10,208 | 9,170 | 1,034 | 10.1% |
| 11 | 重庆 | 10,151 | 9,264 | 880 | 8.7% |
| 12 | 武汉 | 10,111 | 9,251 | 852 | 8.4% |
| 13 | 西安 | 9,706 | 8,744 | 954 | 9.8% |
| 14 | 长沙 | 8,672 | 7,964 | 701 | 8.1% |
| 15 | 苏州 | 8,123 | 7,314 | 798 | 9.8% |
| 16 | 石家庄 | 8,032 | 7,240 | 786 | 9.8% |
| 17 | 沈阳 | 7,946 | 7,151 | 785 | 9.9% |
| 18 | 昆明 | 6,894 | 6,244 | 646 | 9.4% |
| 19 | 太原 | 6,600 | 5,901 | 696 | 10.5% |
| 20 | 哈尔滨 | 6,376 | 5,729 | 642 | 10.1% |

集中度：Top1 = 5.3%，Top3 = 14.2%，Top10 = 31.3%——分布分散，同城内测必须选头部城市。按男性数排序前 20 与此同集，仅次序微调。

## 三、推荐：北京 + 上海

- **男性绝对量是硬约束**：抽 250 男需要 3-5 倍冗余。北京 2,594 / 上海 2,311 是全国仅有的两个 2,300+ 男性候选城市（北京 10.4 倍冗余、上海 9.2 倍）。
- 总浓度同为前二（27,942 / 26,181），女性侧完全无压力。
- 只选 1 城选北京（男性量、总量双第一）；广州（男 2,279，男性占比 10.7% 头部最高）可作备选第三城。

## 四、推荐城市内画像

### 性别 × 年龄层（人数）

| 城市 | 性别 | <18（排除） | 18-24 | 25-30 | 31-40 | >40 | 未知 | 合计 |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| 北京 | 女 | 780 | 9,003 | 8,468 | 5,840 | 1,242 | 1 | 25,334 |
| 北京 | 男 | 77 | 955 | 877 | 527 | 155 | 3 | 2,594 |
| 上海 | 女 | 721 | 7,277 | 8,571 | 6,022 | 1,261 | 1 | 23,853 |
| 上海 | 男 | 50 | 773 | 796 | 527 | 165 | 0 | 2,311 |

未成年占比约 3%（北京 857 / 上海 771 人），抽样 SQL 硬性排除。排除后成年男性可抽池：**北京 2,514、上海 2,261**。

### MBTI

- 填写率：北京男 36.1%、女 23.5%；上海男 34.5%、女 23.3%。
- 两城两性别 **16 型均有人覆盖**，无脏值。
- 结构特征：NF/NT 主导（北京女 top3：ENFP 807 / INFJ 664 / INFP 651；男 ENFP/INFJ/INFP 各 116），S+P 型稀薄——北京男性 ISTP 仅 12 人、ESFP 22 人。这是测测用户的结构性特征，不是抽样问题。

### 可行性判断

**北京、上海各抽 500 人（男女 5:5）均可行。**

- 男女 5:5：北京成年男 2,514 ÷ 250 = 10 倍冗余，稳。
- 年龄层：男性最薄的 >40 层有 155 人，每层抽 40-70 人无压力。
- MBTI 是唯一真瓶颈：若强制全员有 MBTI，北京男候选骤降到 936（3.7 倍冗余，叠加年龄分层后 >40 层会很紧）；若只求"覆盖尽量多型"，可保 16 型全覆盖但 ISTP/ESFP 男性只能各进 1-3 人。**建议不强制全员有 MBTI：男性侧按"已填优先 + 稀缺型优先"排序，未填者补足名额，入选后引导补测。**

## 五、抽样 SQL（北京，男女各 250，明细版）

规则：排除未成年/生日未知/测试号；同性别内按 4 个成年年龄层轮转铺开，层内 MBTI 已填优先、再按活跃天数降序。换城市改 `city`，改名额改 `<= 250`。

```sql
WITH act AS (
  SELECT user_id, COUNT(DISTINCT pt) AS active_days
  FROM internal.cece_dwh.dws_cc_user_active_di
  WHERE pt BETWEEN '2026-06-03' AND '2026-07-02' AND is_active = 1
  GROUP BY user_id HAVING COUNT(DISTINCT pt) >= 15
),
pm AS (
  SELECT user_id1 AS user_id, COUNT(*) AS pm_cnt
  FROM internal.cece_dwh.dwd_cc_audit_private_message_verify_df
  WHERE dt BETWEEN '2026-06-03' AND '2026-07-02'
  GROUP BY user_id1 HAVING COUNT(*) >= 5
),
hp AS (
  SELECT user_id, SUM(use_cnt) AS hp_cnt
  FROM internal.cece_dwh.dws_cc_traffic_sensor_tools_active_using_cnt_di
  WHERE pt BETWEEN '2026-06-03' AND '2026-07-02' AND tool_type = '合盘'
  GROUP BY user_id HAVING SUM(use_cnt) >= 2
),
p1 AS (
  SELECT a.user_id, a.active_days,
         COALESCE(pm.pm_cnt, 0) AS pm_cnt, COALESCE(hp.hp_cnt, 0) AS hp_cnt
  FROM act a
  LEFT JOIN pm ON a.user_id = pm.user_id
  LEFT JOIN hp ON a.user_id = hp.user_id
  WHERE pm.user_id IS NOT NULL OR hp.user_id IS NOT NULL
),
cand AS (
  SELECT p1.user_id, d.gender, d.city,
         TIMESTAMPDIFF(YEAR, d.birthday, '2026-07-02') AS age,
         CASE WHEN TIMESTAMPDIFF(YEAR, d.birthday, '2026-07-02') <= 24 THEN '18-24'
              WHEN TIMESTAMPDIFF(YEAR, d.birthday, '2026-07-02') <= 30 THEN '25-30'
              WHEN TIMESTAMPDIFF(YEAR, d.birthday, '2026-07-02') <= 40 THEN '31-40'
              ELSE '>40' END AS age_band,
         UPPER(d.mbti) AS mbti,
         p1.active_days, p1.pm_cnt, p1.hp_cnt
  FROM p1
  JOIN internal.cece_dwh.dim_cc_user_view_df d ON p1.user_id = d.user_id
  WHERE d.city = '北京'
    AND d.is_test = 0
    AND d.gender IN ('男', '女')
    AND d.birthday IS NOT NULL
    AND TIMESTAMPDIFF(YEAR, d.birthday, '2026-07-02') BETWEEN 18 AND 100
),
ranked AS (
  SELECT c.*,
         ROW_NUMBER() OVER (
           PARTITION BY gender, age_band
           ORDER BY CASE WHEN mbti IS NOT NULL AND mbti <> '' THEN 0 ELSE 1 END,
                    active_days DESC, pm_cnt DESC, hp_cnt DESC
         ) AS rn_in_band
  FROM cand c
)
SELECT user_id, gender, city, age, age_band, mbti,
       active_days, pm_cnt, hp_cnt
FROM (
  SELECT r.*,
         ROW_NUMBER() OVER (PARTITION BY gender ORDER BY rn_in_band, active_days DESC) AS pick_order
  FROM ranked r
) t
WHERE pick_order <= 250
ORDER BY gender, pick_order
LIMIT 100000
```

外层按 `rn_in_band` 轮转 = 对 4 个年龄层 round-robin，250 人 ≈ 每层 60 出头（>40 层男性 155 人足够）。若想强制 MBTI 16 型全覆盖：先跑一版 `PARTITION BY gender, mbti` 每型锁 2 人，再用本 SQL 补齐余量。

## 六、数据质量备注

1. **city 随最近活跃定位刷新**：天然适配同城匹配，但差旅/搬迁会漂移；正式发名单当天用最新分区重跑一次。
2. **档案自填现居地不可用作主口径**（默认值污染），产品侧若要展示"现居地"需先做默认值剔除与标准化。
3. **MBTI 近 7 成缺失**：内测如需精准 MBTI 配对，建议引导入选者补测。
4. **未成年人已排除**，名单发放前建议再过一遍实名年龄核验。
5. 海外用户 2.9%、城市未知 0.1%、性别未知 0.08%，均不入京沪名单，无影响。
6. 私信数含审核拒绝消息（上一轮备注），发名单前建议对高拒绝率用户二次过滤。
