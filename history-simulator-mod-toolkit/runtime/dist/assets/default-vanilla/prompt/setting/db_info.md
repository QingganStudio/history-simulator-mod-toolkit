## PostgreSQL数据库表结构信息

### 支持的数据表列表
以下是系统支持的所有数据表名称，请在函数调用中使用这些确切的表名：

1. regimes - 势力表, 游戏中所有可用的国家或政权势力的基本信息
2. parties - 党派表, 游戏中大明国内各个党派（利益集团）的详细信息
3. ministers - 存储游戏中的核心人物「大臣」，用于描述其身份、能力、状态以及与党派的归属关系
4. provinces - 省份表, 游戏中所有省份的详细信息
5. equipments - 装备表, 游戏大明的武器装备产量和库存
6. arm_configs - 陆军配置表,  游戏中所有可用的陆军兵种图鉴
7. ship_configs - 船种配置表, 游戏中所有可用的战船种类图鉴
8. social_classes - 社会阶层表, 当前游戏大明国内各个阶级的详细信息
9. land_tax_configs - 每亩土地每季度收税配置表
10. troops - 军队表，游戏中所有军队的详细信息
11. events - 事件记录以及预测表, 记录游戏中已经发生、即将发生的历史事件
12. finances - 财政表, 记录国库、内帑的库存、每季度收入和每季度支出情况
13. province_land_holdings - 省份土地分布表, 记录各省民田、官田的具体数量
14. buildings - 建筑表，记录被创建的目前还存在的建筑，以及创建时带来的效果

### 详细字段信息

#### 1. regimes (势力表)
- 主键: code_name
- 字段:
    - code_name (String): 势力唯一代号，如 '2001_ming'
    - name (String): 势力名称，通常固定。但若发生政权更迭（如“大明”改为“南明”），或新势力建国（如“后金”改为“大清”），需修改此项
    - current_ruler (String): 当前统治者 (如 '朱由检')，若在推演中领袖驾崩、退位，或起义军领袖战死更换继承人，就需要更改此字段内容
    - corruption (Integer): 【重要数值】腐败度 (0-100)，腐败值达到100时会损失约30%的收入，随着腐败值降低而递减，递减速度逐渐放缓，腐败值越高，诏书越难得到准确执行，该值对游戏难度影响较大，应小幅修改，每次修改幅度建议为1-3。**注意：银两余额在 finances 表，本字段只影响收入倍率，不直接存放银两。**
    - prestige (Integer): 【重要数值】权威 (0-100)，统治者的个人威望，威望值越高，诏书执行的越顺利。该值对游戏难度影响较大，应小幅修改，每次修改幅度建议为1-3，当皇帝击败敌军，或显示皇室风范，展现个人魅力时应增加1-3点权威，当皇帝操作失误，胡言乱语，下罪己诏，体现亲民时，应扣除1-3地点权威
    - currency_policy (Integer): 【重要数值】通货供应水平 (0-100)，该值会因为国家货币政策而变动，且低通货会降低赋税收入。若在推演中海外白银大量流入，或新开金银矿，或完善了纸钞，大铸铜钱等，需增加 currency_policy（市场繁荣）。若于战乱导致富户窖藏白银，或民间私自铸钱严重，滥发纸币导致通货膨胀等实际上没有增加有价值的通货，反而损害经济的行为发生，需减少此值。项会极大影响国家的财政总收入，数值修改幅度必须要小，每次修改值建议为1-3。**注意：银两余额在 finances 表，本字段只影响收入倍率，不直接存放银两。**

#### 2. parties (党派表)
- 主键: code_name
- 字段:
    - code_name (String): 党派代号
    - name (String): 党派名称
    - satisfaction (Integer): 满意度 (0-100)，满意度越高，该党派就会支持玩家的诏书，反之就会阻挠
    - influence (Integer): 影响力 (0-100)，影响力越大，该党派就越有能力造成影响，反之就掀不起风浪
    - rebellion_risk (Integer): 叛乱风险 (0-100)，叛乱风险越高，该党派就有越高概率谋反，做出对玩家不利的造反举措
    - description (String): 党派描述及其行事风格、利益关系

#### 3. ministers (大臣表)
- 主键: code_name
- 外键: party_code_name -> parties.code_name
- **去重铁律**：code_name 是主键，但**大臣 name 在本表全局唯一**。新增大臣前**必须先按 name（姓名）在背景数据中查重**：若该姓名已存在（无论其 code_name 是什么），一律用 op=update 修改既有记录（如调整 position），**严禁重复 create**，也不要为同一人另起 code_name 重建。只有姓名确实不在表中时才用 op=create，新建 code_name 约定为 `[数字序号]_[姓名拼音]`（如 '3001_weizhongxian'）。
- **人设必填**：op=create 新建大臣必须同时提供非空 prompt（该大臣的角色扮演人设提示词：出身、性格、政治立场、说话风格）与 description（给玩家看的人物介绍），缺任一项整条 create 会被系统拒绝，不得只写姓名官职就提交。op=update 不受此约束。
- 字段:
    - code_name (String): 大臣代号，通常用于程序或配置引用，如'3001_weizhongxian'
    - name (String): 大臣的姓名，用于展示给玩家
    - position (Array[String]): 大臣当前担任的职位列表，例如“尚书”“大学士”等，可同时拥有多个职位。这个项目很重要。你可以修改这个字段来调整大臣的职位。如果诏书要求罢免官员填充”赋闲“。
    - stance (String, Nullable): 大臣的政治立场，用于刻画其在政治博弈中的倾向
    - prompt (String): 大臣的角色扮演人设提示词，AI 扮演该大臣对话（私聊、议事）时使用的系统提示词，须写明出身、性格、政治立场、说话风格、自称与称呼习惯。op=create 新建大臣时必填（与 description 同为创建硬性要求）
    - influence (Integer): 影响力（0-100），表示其在朝廷或体系中的话语权和号召力
    - description (String, Nullable): 大臣的人物介绍，用于给玩家展示。你可以修改这个字段来描述大臣的背景、经历、性格等信息。op=create 新建大臣时必填（与 prompt 同为创建硬性要求）。
    - gender (String, Nullable): 性别信息，用于角色设定或事件触发条件，仅包含 male、female
    - party_code_name (String, Nullable): 所属党派唯一标识，表示该大臣当前隶属的党派
    - is_visible (Boolean): 是否对玩家可见，用于控制隐藏人物（例如敌对势力人物）。如果玩家提拔或接触了这个大臣，那么应该设置为true。
    - is_alive (Boolean): 是否存活，用于标识死亡或已出局的大臣了。如果玩家要求处死一个大臣，或者大臣因故死亡，那么这里设置为false。
    - is_system (Boolean): 是否为系统人物，用于区分剧情/系统内置角色与玩家生成角色。如果玩家要求新增了一大臣，例如玩家希望提拔一个目前不在大臣列表中的大臣，那么这里设置为false。

#### 4. provinces (省份表)
- 主键: code_name
- 外键: regime_code_name -> regimes.code_name
- 字段:
  - code_name (String): 代号，如 '1001_beijing'
  - name (String): 省份通俗用名
  - population (Integer): 该省份登记在册的居民总数（不包含流民）
  - public_sentiment (Integer): 民心 (0-100)，代表本省民众对朝廷的支持度。
  - rebellion_risk (Integer): 动乱风险（0-100），动乱越高，镇压效果越差
  - yield_mu (Numeric): 粮食产量 (石/亩)，时间单位为每季度
  - other_tax (Integer): 其他税收
  - disaster_status (String): 灾害状态 (如“旱灾”，“洪水”，“战乱”)，该字段随着推演应随时更改
  - disaster_level (String): 灾害等级 (只能填写这四个值：“正常”，“轻微”，“中等”，“严重”)，当灾祸被解决后应更改回“正常”
  - defence_level (Integer): 城防等级 (0-10)，城防越高则省份越难被攻破
  - description (String): 地区的物产气候描述
  - tax_rate (Numeric): 税收倍率（该值>0)，官府对该省所有田税的控制倍率，如果要给全省加税或免税，应修改此值，0.5为正常值。
  - tax_efficiency (Numeric): 税收效率，（0\%-100\%),表示官府的行政能力可以覆盖该省的多少税收)
  - is_coastal (Boolean): 是否沿海
  - regime_code_name (String): 归属势力唯一标记，如果省份被攻占，那么应该更改此标记至对应的政权。

#### 5. equipments (装备表)
- 主键: code_name
- 字段:
  - code_name (String): 装备代号
  - name (String): 装备名称
  - stock_count (Integer): 库存数量
  - annual_output (Integer): 每季度的产量
  - description (String): 装备描述和简介

#### 6. arm_configs (陆军配置表)
- 主键: code_name
- 字段:
  - code_name (String): 兵种代号
  - name (String): 兵种名称
  - maintenance_cost (Numeric): 维护费（两/季度）
  - equipment_requirements (JSONB): 装备需求 {'6001_melee': 1}，装备不足无法训练对应兵种

#### 7. ship_configs (船种配置表)
- 主键: code_name
- 字段:
  - code_name (String): 船种代号
  - name (String): 船种名称
  - crew_count (Integer): 船员数量
  - light_cannon_count (Integer): 轻炮数量，数量多则战斗力强
  - heavy_cannon_count (Integer): 重炮数量，数量多则战斗力强
  - maintenance_cost (Numeric): 单船维护费（两/季度）

#### 8. social_classes (社会阶层表)
- 主键: code_name
- 字段:
  - code_name (String): 阶层代号
  - name (String): 阶层名称
  - satisfaction (Integer): 满意度 (0-100)
  - influence (Integer): 影响力 (0-100)
  - rebellion_risk (Integer): 叛乱风险 (0-100)
  - description (String): 阶层介绍

#### 9. land_tax_configs (土地税配置表)
- 主键: code_name
- 字段:
  - code_name (String): 田税类型代号
  - name (String): 田税类型名称
  - tax_base (Numeric): 基础田赋，单位为（两/亩/季度）
  - tax_liao (Numeric):  辽饷税率，单位为（两/亩/季度）
  - tax_old (Numeric):  旧饷税率，单位为（两/亩/季度）
  - tax_new (Numeric): 新饷税率，单位为（两/亩/季度）

#### 10. troops (军队表)
- 主键: code_name
- 外键: regime_code_name -> regimes.code_name, location_province_code_name -> provinces.code_name
- 字段:
  - code_name (String): 军队代号
  - name (String): 军队名称（显示名/番号）。创建时填写中文军名；既有部队改名必须走军事命令 military.rename，不得用通用状态工具更新本字段
  - type (String): 类型 ('army'代表陆军 或 'navy'代表海军)
  - exp (Integer): 战斗经验，经验越丰富战斗力越强，但新兵补员后经验会降低，作战和训练可以提升经验
  - morale (Integer): 士气 (0-100)，士气越强战斗力越强，士气越低越容易溃败，士气值为20是维持战斗力的底线
  - loyalty (Integer): 忠诚 (0-100)，忠诚度越低，越容易哗变、投敌，低于20就有极高概率叛变
  - supply (Integer): 补给 (0-100)，补给越低，士气就会下降的越快，补给低于20的军队会开始非战斗减员，补给低于50的军队会受到战斗力减弱的效果。
  - attached_personnel_count (Integer): 海军除水手以外搭载的作战部队数量，缺少作战部队的海军不能进行登陆作战或跳帮作战。
  - extra_cost (Numeric): 军队每人每季度的额外维护费（0-10），包括边军补贴，固定奖金，和吃空饷导致的额外花费。超过10两每季每人的花费被认为是过高而无法实现的。
  - salary_rate (Numeric): 满饷率，数据库值范围0-1（0.70代表70%，报告可展示为百分比）；低于1.0属于欠饷状态，战斗力会降低。如果朝廷下旨持续提高实发比例就增加此值，缩减军饷则减少此值。
  - composition (JSONB): 兵种/船只构成和具体数量，作战时要考虑兵种之间的克制关系 [{'type': 'army/ship/equipment', 'count': 1, 'code_name': 'xxx'}, ...]
  - regime_code_name (String): 所属势力唯一标识，当军队叛变、投降、起义、投诚、被移交时，应该更改此标识为新的所属势力
  - location_province_code_name (String): 驻扎省份唯一标识，当军队移动时，应将此标识改为当前军队所在地

军队落库强约束：
- 凡报告声称战损、裁汰、调动、组建、消亡、投诚或战备变化，必须同步修改 troops 实体或对应字段；事件只记录事实，不能代替军队状态落库。
- 战斗损失、实际裁汰、部队间转编、从旧军拆出新军、整支移防必须区分：战损从参战部队现有编制扣除；实际裁汰不需要目标部队；转编必须明确来源和目标；拆出新军必须同时保留来源军并创建目标军；整支移防必须更新驻地。
- 跨部队转编（一支部队减编、另一支部队增编）必须用原子 split（目标为新部队）或两条共享同一 transfer_id 的 reorganize（目标为既有部队，来源负增量与目标正增量各一条）；禁止用两条无 transfer_id 的独立 reorganize 做转移——来源减编必须在目标处对上账，否则被判减员悬空。
- 抽调部分兵力异地行动时，不得把来源整支部队的 location_province_code_name 改到目的地；应创建独立支队并让兵员、装备随支队转移。
- 对每支被提及部队逐项核对 composition、salary_rate/extra_cost、morale/loyalty/supply/exp、驻地、归属、装备、来源/去向和存亡；同一诏书可能需要多条独立军事命令，不能用士气调整代替编制或饷费变化。
- 玩家主势力部队补发欠饷走 pay_salary；对任意已存在部队的一次性犒赏走 reward，财政金额来自对应 operation.resources，loyalty_delta/morale_delta 至少一项必须大于 0，若目标对应属性均已达 100 则不得规划无效犒赏；持续满饷比例变化走 adjust_readiness.salary_rate；额外月粮、边军补贴或加耗走 adjust_readiness.extra_cost。extra_cost 是两/人/季度而非比例，比例命令须按已知基准折算银两；salary_rate 已为 1.0 不代表额外月粮无需落库。
- 任意势力的部分兵力哗变、投敌使用 military.defect：原子扣减来源 composition、创建新部队并转移随队人员，目标为玩家主势力也不自动收费，有实际安置或赏银支出时才声明 fiscal.debit；玩家主势力整军叛离也使用 defect，原地切换 regime_code_name，保留稳定 code_name，可用 new_name 更换显示番号，不得删除来源后重建。非玩家整军不得走 defect。
- 非玩家整支部队在非战斗结算中投降、投诚、移交或改换阵营使用 military.realign：原地修改 regime_code_name，保留原部队编码、编制、装备、驻地和附属人员，可用 new_name 更换显示番号；转属不自动收费；有实际安置、赏银或补给支出时在 operation.resources 声明 fiscal.debit，自愿归附可省略。玩家主势力部队转出不得使用 realign。战斗结局中的败军投诚由 military.battle participant 的 disposition=surrender 与 surrender_to_regime_code_name 原子结算，不得另发 military.realign。
- 既有部队纯改名（保留 code_name、编制与归属，仅更新显示名/番号）使用 military.rename：原地更新 name，无财政成本。defect/realign 的 new_name 只在其叛离/转属语境更换番号——两者都会改变归属，不得借其表达纯改名。
- 农民军、叛军等自发滋生时也必须创建 troops 实体，并填写真实 regime_code_name 与驻地；不得伪装成朝廷正规征募、不得扣大明财政或中央装备库存。确认其消亡时删除实体，不得把其装备回收到大明库存。
- 既有兵种、部队、所属势力和驻地使用背景数据中的真实 code_name；Harness 支持身份分配的即时单对象创建由服务端分配代号，依赖操作须显式关联创建结果并确认前置成功，不得混用新旧对象；报告不得声称任何未落库的军事结果已经完成。

#### 11. events (事件记录以及预测表)

  - Agent 创建事件时不要提交 status 或 time_label；系统依据 turn、event_type 和 end_turn 自动派生状态与时间标签。status 只用于更新既有普通事件。
- 主键: code_name
- 字段:
  - code_name (String): 事件唯一标识（如 '1', '2', '3'）
  - predicted_occurrence_time (String): 发生时间（如 '崇祯1年春季', '崇祯10年秋季'）
  - predicted_occurrence_turn (Integer): 发生回合数（如 1, 37），如果大于
  - content (String): 可被程序解析的结构化事件内容结构
- 说明:
    - 事件的content格式:
        1. content使用json string输出，例如：'{"状态": "xx", "事件描述": "xx", "预测效果": "xx", "触发人": "xx"}'，需确保其可以被`json.loads()`正常解析。
        2. 参数说明：
            - 状态：未发生/进行中/已发生（三选一）；被 lifecycle_control 取消的长效事件由系统置为"已取消"（终态，与正常完工的"已发生"相区分）
            - 事件描述：玩家可见的中文叙事，用自然语言描述事件内容；禁止出现英文字段名（tax_rate）、表名、code_name 或数值公式，涉及字段一律用中文名（税率、民心、国库等）
            - 预测效果：同事件描述，用中文叙事表达预期后果，不写字段名或内部结构
            - 触发人（必填）：皇帝/其他（二选一，不得省略。"皇帝"（主角称谓）代表该事件是由玩家/诏书导致的；"其他"代表该事件是由皇帝以外的人、势力、历史等因素导致的）
            - 结束回合（可选）：项目完工、线程结束或临时状态恢复的生效回合，仅在状态="进行中"时建议填写。填写后该事件在结束回合前不会被每8回合一次的事件归档压缩删除，会持续留在本表被后续每回合读取——用来承接跨多回合的长期项目（水库、造船、驻防等），避免因归档压缩而被遗忘。普通线程在 end_turn 当回合由系统自动置为"已发生"；机械项目先执行该回合最后一期 on_cycle，再执行 on_complete，只有实际成功才完成，暂停或待补算不能视为已结束，无需手动 update。**换算铁律**：本剧本 1年=4回合；从当前回合起持续 N 回合时，end_turn=当前回合+N-1（如持续三年即12回合，应填本回合+11），明确指定完工回合时直接填该回合，例如第5回合创建、第7回合完工填写 turn=5、end_turn=7，不填6。restore 的 end_turn 必须晚于 turn，不允许同回合应用又恢复；创建回合已结算，恢复回合又在结算前恢复，因此免税等效果需覆盖 N 次结算时填 end_turn=当前回合+N+1；第5回合免税一季填7，第6回合免税结算，第7回合恢复；当前回合1、免税12期填14。仅要求状态存续N回合、不要求N次结算时可用当前回合+N，不能混淆两种期限。既有restore不得移动开始回合，修改后的结束回合必须晚于当前回合。周期财政扣款首期在 start_turn+every_turn，按实际扣款次数确定 end_turn，结束回合先执行最后一期再完成，不得少算一期。换算后对照下方"events表回合和时间对应关系"核对。
    - 事件是否可改**按状态分级**（不是简单看回合先后）：
    1. 已固化的历史事件：predicted_occurrence_turn < 当前回合 **且 状态="已发生"** 的事件，是已成定局的历史事实，**只读，不允许 update/delete/重复 create**。若本回合对此历史有新进展要记录，新增一个 predicted_occurrence_turn=当前回合 的事件。
    2. 过去回合遗留的"预测"/"进行中"事件：predicted_occurrence_turn < 当前回合 但 状态还是"预测"或"进行中"的事件，**尚未定局，可 update/delete**——
        - "预测"事件到了其预定回合应予确认：结果发生则 update 状态为"已发生"；已被后续事件覆盖或不再发生则 delete 清理，避免过时预测长期堆积。
        - "进行中"的跨回合项目可逐回合 update 描述/预测效果；但带 long_term_effect 的 project 事件，其状态与周期作用由系统按 on_start/on_cycle/on_complete 自动维护，不要手动 update 进度或改"已发生"——跨回合线程（只填 end_turn 的 fact）到期由系统自动完结为"已发生"，无需手动 update。
    3. 本回合及未来可能发生的事件：predicted_occurrence_turn >= 当前回合 的事件，可基于本回合诏书与实际推演被删除、修改、新建。
    - 举例：
        临时且到期须恢复原值的效果（如某省免税三年）：用 event_type=restore 事件，填 end_turn 与 restore_targets（每项含 table、record_key、临时绝对值 temporary_data）；系统立即应用临时值、到期自动恢复原值。**不要再用 state_operations 手动把字段改到临时值，也不要另建“到期恢复”的预测事件**。例（三年=3×4=12 回合）：
        {"action":"create","code_name":"evt_shaanxi_tax_relief","turn":1,"event_type":"restore","description":"陕西免税三年，期满自动恢复原税率","end_turn":14,"restore_targets":[{"table":"provinces","record_key":"1006_shaanxi","temporary_data":{"tax_rate":0}}]}
        跨回合施工的长期项目（造船、水利、驻防等）用一条 event_type=project + long_term_effect 事件：一次性启动财政或装备成本写在 Phase 1 业务 operation 的 resources，系统创建 project 时消费密封 claims；周期财政支出使用 on_cycle fiscal.debit，库存消耗和 requirements 禁止进入 Hook。不要拆成“开工事件 + 完工事件”两条。
        周期性资源支出铁律：独立持续性财政支出（不含建设、生产或普通征募的项目总成本）必须在 on_cycle fiscal.debit Hook 中填写每期 source/amount，从下一周期逐期执行；不得计算总成本写入 resources 预扣，不得拆成逐期 fact、裸扣款或 state 余额变化。同一 operation 禁止同时使用 fiscal.debit resources；确有启动费和周期费时拆成两个 operation。
        建设项目有玩家支出时，以及生产和普通征募项目，采用启动时全额预付：已确定的总成本写入 operation.resources，完工不再扣款，不支持按进度分期支付。同一费用若已写入 on_cycle fiscal.debit，应将总额改写到 resources 并移除对应周期扣款，禁止保留周期扣款再追加同额预算；总额不明时不得猜测。不同用途的持续性开支（如每季运输费、驻军补贴）另建周期项目。

events表回合和时间对应关系：
{calendar_turn_mapping}

#### 12. finances (财政表)
- 核心架构: 分为 国库 (Treasury) 和 内帑 (Privacy) 两个独立核算体系。
政府财政就是国库，内帑是皇帝自己的私人库存，你要分清楚这两处，当内容中提到政府财政的时候你应该计算国库的财政，例如提到本回合为100万的财政减少，应该在减少国库总财政之后，在分项（例如田赋、盐课）分别减少数值，但各项减少的总值应该等于总数。你应该分别计算内帑和国库的财政收入，任意一项如果处于赤字就必须造成某些负面影响。
- 主键: code_name
- 字段详解:
- 基础信息: current_turn (当前回合)
- 余额: treasury (国库总余额), privacy (内帑/皇帝私钱总余额)。
- 国库收入项:# 国库收入项目每回合会由程序自动计算加入国库余额，推演无需重复计算
  - salt_tea_tax: 盐茶专卖税每季度收入。
  - transport_tax: 钞关/流通税，交易税等正常商业性收入的每季度收入。
  - ore_tax: 矿银和矿课每季度收入。
  - other_treasury_income: 国库其它每季度的长期收入项目，无法分类的一律归于此项。
- 国库支出项:# 国库支出项目每回合会由程序自动计算加入国库余额，推演无需重复计算
  - salary_cost:每季度供养朱姓皇室宗亲的供奉开支。
  - official_salary:全国文官每季度领取的俸禄，提高该值会轻微减少贪腐，提高官僚满意度；降低该值则会增加贪腐，降低民心，降低官僚满意度
  - other_treasury_cost: 国库其它每季度的长期支出项目，无法分类的国库额外每季度开支全都归于此项。
- 内帑收入项 (皇帝私房钱):# 内帑收入项目每回合会由程序自动计算加入内帑余额，推演无需重复计算
  - trade_tax: 宦官搜刮商贾，强买强卖得来的每季度收入。这项收入越高，就会越打击商业发展，也会提高贪腐。
  - jinhua_tax: 金花银每季度收入，来自于江南的粮食税。
  - taipusi_silver_income: 太仆寺马价银每季度收入。
  - imperial_work_income: 皇庄及内臣经营每季度收入。
  - other_privacy_income: 内帑其它每季度的长期收入项目，无法分类的内帑收入一律归入此类
- 内帑支出项:# 内帑支出项目每回合会由程序自动计算加入内帑约，推演无需重复计算
  - palace_cost: 后宫及内廷每季度开销。
  - guard_cost: 锦衣卫、亲军和全国武将的俸禄每季度开支。
  - engineering_cost: 修建陵寝、宫殿等重大皇家工程每季度开支。
  - other_privacy_cost: 内帑其它每季度的长期支出项目，无法分类的内帑支出一律归入此类

重要提示1: treasury/privacy 的一次性扣款来自业务 operation 的密封 ResourcePlan，周期扣款来自 project 的密封 on_cycle fiscal.debit；Agent Tool 不提供 fiscal.debit。入账和账户调拨分别使用 fiscal.credit / fiscal.transfer。
重要提示2: finances 其他未被接管的数值字段使用 state.increment/set，finances 不支持 create/delete。

#### 13. province_land_holdings (省份土地分布表)
描述：记录各省份内不同性质土地（民田、官田等）的具体亩数。它是连接省份、土地配置与最终财政收入的桥梁。
- 主键: code_name
- 字段:
  - code_name (String): 省份土地唯一代号（由土地所属省份和决定税率的土地性质组合而成）
  - area_mu (Integer): 土地面积（单位：亩）
- 逻辑说明:
  - 一个省份在表中会有多行记录（例如：一行记录北京的民田亩数，另一行记录北京的官田亩数）。
  - 操作建议: 若玩家执行“清丈田亩”或“土地改革”，应调整此表的土地面积（area_mu）字段。

#### 14. buildings (建筑表)
- 主键: code_name
- 外键: building_location -> provinces.code_name
- 字段:
  - code_name (String): 建筑代号
  - name (String): 建筑名称，建筑的实质描述；确需更名时只能通过建筑领域 update 修改
  - building_location (String): 建筑所在地，创建建筑时需要指定为 provinces 表中存在的 code_name。
  - build_effects (List[Object]): 建筑建成时由 Building System 应用、删除时反向撤销的状态变化。创建时必填且至少包含一个效果；更新时省略表示保留，提供时必须是至少包含一个效果的完整替换列表，禁止 [] 或 null。跨回合项目完工时的 building.create 同样适用。**必须使用结构化格式（canonical，与随机事件/国策同构）**：每项包含 type、table_name、column_name、op、value、desc；type 固定为 "database"；op 支持 add（数值叠加，value 为增量，撤销取负）、subtract（数值扣减，value 恒为正数，撤销时 add/subtract 互换）、set（目标字段的绝对值，支持数值或文本，不表示本建筑贡献），同一目标字段不得混用 add/subtract 与 set。对 add/subtract 效果，更新时 value 表示本建筑完整的新贡献，例如贡献从+500扩建到+1000应填 op=add,value=1000，不能填差额500或 set 1000；已有 add/subtract 贡献不得在更新时改成 set。系统先撤旧再应用新贡献。明确的独立 set 效果仍可按绝对值更新。add/subtract 在当前字段值上加减，set 直接替换整个字段；增加/减少多少与设为多少必须区分。set 的 prior_value 由系统维护、不要填写。equipments/troops/parties/ministers/social_classes/regimes 等多记录表必须填写 row_name；provinces 也须显式填写 row_name（未填时执行器回退 building_location，仅防御）；finances 是单例表禁填 row_name。示例——`[{"type":"database","table_name":"equipments","column_name":"annual_output","op":"add","value":1000,"row_name":"6009_flintlock","desc":"燧发枪年产+1000"},{"type":"database","table_name":"regimes","column_name":"prestige","op":"add","value":10,"row_name":"2001_ming","desc":"权威+10"},{"type":"database","table_name":"provinces","column_name":"defence_level","op":"add","value":3,"row_name":"1001_beijing","desc":"本建筑提供城防+3"}]`。其中"收入基数"对应 finances 的 imperial_work_income/trade_tax/salt_tea_tax 等固定收入项，"/季"指该基数每季被 recalculate 消费产出恒定收入（非逐季累加，逐季累加请改用 long_term_effect.on_cycle 并填 end_turn 上界）；长期收益后续由回合核算消费。仍存储在原 JSONB 字段中。
  - 资源成本不属于 buildings 表字段，也不出现在 Phase 2 建筑参数中。财政或装备成本只写在 Phase 1 对应 building operation 的 resources，由 Harness 密封为 claims；即时路径与建档原子执行，跨回合 project 在创建时消费、完工不重复消耗、失败或取消自动退回。
逻辑说明：
-如果要新增一座建筑、工厂、机构、工程等，应创建建筑条目；若要拆除，则删除对应条目；更名、扩建或改造通过建筑领域 update 修改允许的字段，building_location 不可修改。
-如要更新、扩建、改造建筑：本回合即时改造应规划建筑领域更新；跨回合改造应在可执行持续项目的完成节点规划建筑领域更新，不要另建新建筑。具体执行入口以当前 Harness 能力契约为准。
新增建筑的两条黄金法则：
  - **能当期完工**（轻工、资金充足、非大型工程）→ 规划本回合即时创建建筑实体，并明确名称、地点、建筑效果和一次性成本。
  - **需跨回合施工**（工程量大、财政不足、造船/水利/驻防等大型项目）→ 规划可执行持续项目：成本归属于 project operation.resources 并在创建时消费，完成节点只创建建筑实体并应用建筑效果，结束回合设为实际完工回合。
禁止三种"假建"：
  - 禁止只在报告/叙述里说"已建/已设/已成立"却既未即时创建建筑实体，也未在跨回合项目完成节点排期创建。
  - 禁止只改 finances/equipments 等经济效果、漏掉 buildings 表实体本身。
  - 禁止创建无 `on_complete` 的跨回合线程充当建筑（不会产生建筑实体）。
### （适用所有表的）重要提醒
1. 请严格遵守数值字段的取值范围
2. 请使用确切的字段名，不要使用缩写或变体
3. Boolean字段使用true/false
4. 输出数据修改时，code_name必须属于这个表。比如code_name=2014_qiyi，属于table_name=regimes表。


## 推演世界规则
### 价值指导：所获（在假设没有被贪官染指的前提下）
1. 一个普通贪官的家产在5万-20万两之间，众数约为8万两。
2. 魏忠贤的所有家产财宝变卖所得总额一般在200万两-300万两之间。
3. 其余阉党重臣（如崔呈秀等人）的家产总额一般在30-50万两之间。
4. 藩王宗室（如福王）的所有家产总额约在100万两-300万两白银，且拥有大量王田。
5. 江南士绅家族一个家族的家产约在20万两-200万两之间，视家族势力和底蕴而定。
6. 山西晋商家族，一家的财产约在30万两-50万两之间，八大晋商资产的总和约为250万两-400万两之间。

#### 权威贪腐增减
将阉党全部杀尽，贪腐值应当会减少10-15；将东林党全部杀尽，贪腐值也应当会减少10-20。
任何需要花费大量银两的命令或工程都会使得贪腐上升1-2
大量的处死大臣，权威会上升10-20；廷杖、处罚大臣，权威会上升1-2

### 各方势力行为指导：
1. 阉党（附庸皇权的权臣集团）
最高优先级：维持皇帝的信任与宠信，攫取系统内最大化的财富与权力。
决策逻辑：
对上：绝对服从皇权。在任何情况下，极难主动触发“造反”、“逼宫”或“武力抗拒皇权”的行为，他们会通过哄骗和欺瞒来麻痹皇帝；当皇权表露杀机时，大概率采取辩解、乞求或物理自尽等消极应对。
对下/对敌：零和博弈。将任何非本派系的政治势力视为死敌；在拥有皇权默许时，行为无底线，手段残忍，优先使用构陷、株连、抄家等极端手段打击异己。
资源处理：优先满足个人私囊与皇帝的享乐需求，无视底层民众的生存底线与国家的长期宏观经济健康。
2. 文官集团（士绅地主阶级代言人）
最高优先级：捍卫士绅阶层的政治特权与经济豁免权（免税特权）。
决策逻辑：
议事规则：道德绑架。在所有提案与辩论中，强制使用儒家道德标准作为武器，优先否定任何触及工商、农业税制改革的法案。
问责机制：提出问题而非解决问题。面临危机时，首要动作为寻找并指责“替罪羊”（政敌或前线武将），坚决拒绝承担直接的执行责任。
忠诚度弹性：表面极度忠诚，但在大明岌岌可危时，会果断投向新政权以保证士绅特权。
3. 后金/大清
最高优先级：获取生存资源，完成从部落联盟向中央集权帝国的转化。
决策逻辑：
军事行动：极致的实用主义。严格执行“避实击虚”原则，拒绝在劣势条件下进行高损耗的阵地战；军事行动以掠夺人口、物资和削弱敌方经济基础为首要目标。
外部统战：高回报收买。对于被明朝冷落的技术人员、高级将领或知识分子，给予超出常规的信任、财富和地位，快速吸收明朝技术与制度。
内部管理：通过持续的外部战争红利来压制和转移内部各派系（如各旗旗主）的权力斗争，通过不断地掠夺来缓解生产力不足。
4. 流民起义武装
最高优先级：在极端匮乏的环境中维持基本的生理存活，队伍规模必须持续扩张。
决策逻辑：
流动机制：拒绝静态防御。不执着于防守任何固定领土，以获取下一批粮草为唯一导向进行移动。
扩充机制：裹挟。每到达一个新区域，强制破坏当地的经济生产体系，迫使破产平民转化为武装力量，呈滚雪球式发展。
契约精神：零契约。在实力处于劣势或面临绝境时，便会主动请求招安；一旦外部压力解除或粮草断绝，立即单方面撕毁契约重新反叛。
5. 边军/地方军阀
最高优先级：保存核心私人武装力量，获取中央最高额度的财政拨款。
决策逻辑：
交战规则：养寇自重。避免与强敌进行消耗战，友军遇险时大概率“观望”或“缓慢移动”；顺风局极速追击，逆风局优先溃散。
要挟机制：以“哗变”、“欠饷”或“敌军压境”为筹码，持续对中央皇权进行政治勒索和资源索要。
外交倾向：墙头草。始终在多方势力之间保持秘密的通讯渠道，根据各方实力对比，随时调整自己的站位以实现利益最大化。
6. 地方乡绅宗族
最高优先级：维护本宗族的土地所有权、财富积累及在基层的绝对统治力。
决策逻辑：
经济策略：危机套利。在系统发生灾荒或战乱时，进行“囤积居奇”和“土地兼并”，同时通过各种手段隐匿人口，将税收负担转嫁给无特权的底层。
安全策略：结寨自保与顺从。面对弱势的流民武装时，组建私人武装死守堡垒；面对具备压倒性优势的正规军（无论哪个阵营），立即触发“献城劳军”选项以保全资产。
7.皇权集团
最高优先级：拥护皇帝的意志，在皇帝的英明领导下拯救大明。
决策逻辑：
皇权集团被视为是皇帝的亲信，包括皇党大臣和一些被皇帝提拔的中立派，他们将皇帝视为英明圣主，甘愿成为皇帝的利剑。
