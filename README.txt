================================================================================
                         老有所依 · 社区互助养老时间银行平台
                    (全国大学生数智链大赛参赛作品 · 源代码说明)
================================================================================

一、项目介绍
--------------------------------------------------------------------------------
「老有所依」是一个基于 FISCO BCOS 联盟链的社区互助养老时间银行平台。
志愿者为社区老人提供送餐、陪护、就医陪诊等服务,服务时长经老人确认后
登记上链并计入志愿者的时间银行余额;志愿者可用时长兑换社区超市代金券、
理发、家政等福利。民政监管端可查看全部链上存证记录、异常预警和区块链
可视化图。

核心亮点:
  · 三层防重复上链:数据库查重 + 合约 contentHash 去重 + Mock 链校验
  · 区块链可视化:横滑卡片链展示每笔上链交易(区块号/类型/交易哈希/合约)
  · 时间银行存证:登记→确认→兑换 全流程上链,链上余额与数据库双写一致
  · 角色权限:老人 / 志愿者 / 管理员 / 民政监管


二、技术栈
--------------------------------------------------------------------------------
后端 (backend):
  · Spring Boot 2.7.18  (Java 8+,推荐 JDK 11)
  · Spring Data JPA + Hibernate
  · H2 内存数据库(默认,开箱即用) / MySQL 8.0(可切换)
  · JWT 鉴权 (jjwt 0.9.1)
  · Lombok
  · 区块链抽象层:MockBlockchainService(默认) / FiscoBcosService

前端 (frontend):
  · Vue 3.4 + Vite 5.1
  · Element Plus 2.6
  · Vue Router 4 + Pinia 状态管理 + Axios

智能合约 (contracts):
  · Solidity 0.8 · TimeBank.sol (时间银行存证合约)


三、目录结构
--------------------------------------------------------------------------------
01_源代码/
├── backend/        Spring Boot 后端源码
├── frontend/       Vue3 前端源码(已剔除 node_modules)
├── contracts/      Solidity 智能合约
├── init.sql        MySQL 初始化脚本(建库建表 + 演示账号)
└── README.txt      本说明文件

02_技术路线图/       (请放置项目技术路线图/架构图)
03_效果图/          (请放置系统效果图/截图)


四、环境依赖
--------------------------------------------------------------------------------
必需:
  · JDK 11+ (推荐 JDK 11,兼容 Java 8 语法)
  · Maven 3.6+  (或使用 backend/mvnw,但中文路径可能失败,建议装全局 Maven)
  · Node.js 16+ (推荐 Node 18/20) 与 npm

可选:
  · MySQL 8.0  (默认用 H2 内存库,无需安装)
  · FISCO BCOS 节点 (默认 mock 模式,无需安装)


五、后端启动
--------------------------------------------------------------------------------
【方式一:Maven 命令行(推荐)】
  cd backend
  mvn clean spring-boot:run
  # 服务启动在 http://localhost:8080/api

【方式二:IDEA 打开后运行】
  用 IntelliJ IDEA 打开 backend 目录,等待 Maven 依赖下载完成,
  直接运行 com.laosuoyi.LaosuoyiApplication 主类。

【数据库说明】
  默认 application.yml 使用 H2 内存数据库,JPA ddl-auto=update 会自动建表
  并初始化演示账号,无需任何额外配置即可运行。

  如需切换 MySQL:
    1) 在 MySQL 中执行 01_源代码/init.sql 建库建表
    2) 启动时追加参数:  mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=mysql"
       或 IDEA Run Configuration 的 Program arguments 填入: --spring.profiles.active=mysql
    3) 数据库账号密码见 backend/src/main/resources/application-mysql.yml
       (默认 root / 123456,库名 laosuoyi)

【区块链模式说明】
  application.yml 中 laosuoyi.blockchain.mode:
    mock   → 默认,本地内存模拟链,演示/联调推荐
    fisco  → 接入真实 FISCO BCOS 节点,需配置 RPC 地址和合约地址


六、前端启动
--------------------------------------------------------------------------------
  cd frontend
  npm install          # 首次运行需安装依赖(已剔除 node_modules)
  npm run dev          # 启动 Vite 开发服务器
  # 浏览器打开 http://localhost:5173

  若需打包:
  npm run build        # 产物输出到 frontend/dist


七、演示账号(密码统一为 123456)
--------------------------------------------------------------------------------
  管理员       admin          监管仪表盘 / 全部服务记录
  民政监管     regulator      监管仪表盘 / 全部服务记录
  老人         elder1 (王奶奶)  确认服务
  志愿者       volunteer1 (小李) 登记服务 / 兑换时长

  登录页点击对应演示账号按钮可自动填充账号密码。


八、核心功能演示流程
--------------------------------------------------------------------------------
  1) 志愿者登录 → 登记服务(送餐/陪护等)→ 上链存证,生成区块
  2) 老人登录   → 确认服务 → 志愿者余额增加,链上确认交易
  3) 志愿者登录 → 兑换时长(1小时=100元)→ 链上兑换交易,余额扣减
  4) 管理员登录 → 监管仪表盘 → 查看链上状态 + 区块链可视化卡片链
  5) 重复提交同一笔服务会被三层防护拦截(提示"该服务已登记,不可重复上链")


九、常见问题
--------------------------------------------------------------------------------
Q: 中文路径下 Maven 报错 "Input length=1" ?
A: target 目录编码问题,先删除 backend/target 再编译:
   rm -rf backend/target && mvn clean compile

Q: 前端 npm install 很慢?
A: 可切换淘宝镜像: npm config set registry https://registry.npmmirror.com

Q: 后端启动报端口占用?
A: 杀掉占用 8080 的进程,或修改 application.yml 的 server.port

Q: 浏览器白屏?
A: 确认前后端均已启动;后端 context-path 为 /api,前端 vite.config.js
   已配置代理 /api → http://localhost:8080。


================================================================================
                                  祝比赛顺利!
================================================================================
