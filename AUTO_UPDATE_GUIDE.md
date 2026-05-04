# AI时事网站自动更新方案

## 当前状态
网站已部署到GitHub Pages，具有完整的社交分享功能。但内容是静态的，需要手动更新。

## 自动更新解决方案

### 方案一：GitHub Actions自动化（推荐）⭐

**优点**：完全免费、自动化程度高、无需服务器
**实现步骤**：

1. **创建GitHub Action工作流文件**
   - 在仓库根目录创建 `.github/workflows/update-news.yml`
   - 配置定时任务（每天凌晨6点执行）

2. **编写Python爬虫脚本**
   ```python
   # scripts/fetch_ai_news.py
   import requests
   from bs4 import BeautifulSoup
   import json
   from datetime import datetime
   
   def fetch_hackernews():
       """从Hacker News获取AI相关新闻"""
       url = "https://hacker-news.firebaseio.com/v0/topstories.json"
       response = requests.get(url)
       # ... 解析和处理逻辑
   
   def fetch_arxiv():
       """从arXiv获取最新AI论文"""
       # ... API调用逻辑
   
   def generate_html(news_data):
       """生成更新后的HTML文件"""
       # ... 模板渲染逻辑
   ```

3. **配置工作流**
   ```yaml
   name: Daily AI News Update
   
   on:
     schedule:
       - cron: '0 22 * * *'  # 每天UTC 22:00 (北京时间6:00)
     workflow_dispatch:  # 允许手动触发
   
   jobs:
     update-news:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         
         - name: Set up Python
           uses: actions/setup-python@v4
           with:
             python-version: '3.9'
             
         - name: Install dependencies
           run: pip install requests beautifulsoup4
           
         - name: Fetch AI news
           run: python scripts/fetch_ai_news.py
           
         - name: Commit and push changes
           run: |
             git config --local user.email "action@github.com"
             git config --local user.name "GitHub Action"
             git add index.html
             git commit -m "Daily news update: $(date)" || exit 0
             git push
   ```

### 方案二：使用RSS聚合服务

**数据源推荐**：
- Hacker News AI标签：`https://hnrss.org/newest?q=ai`
- arXiv CS.AI分类：`http://export.arxiv.org/rss/cs.AI`
- TechCrunch AI标签：`https://techcrunch.com/category/artificial-intelligence/feed/`
- MIT Technology Review AI：`https://www.technologyreview.com/topic/artificial-intelligence/feed/`

**实现方式**：
使用现成的RSS-to-JSON服务（如rss2json.com），在前端JavaScript中直接调用API获取最新内容。

### 方案三：后端服务（高级方案）

**技术栈**：
- 云函数：阿里云FC / 腾讯云SCF / AWS Lambda
- 数据库：MongoDB Atlas（免费层）
- 定时触发器：每天执行一次

**架构**：
```
定时触发器 → 云函数（爬取新闻） → 存储到数据库 → 前端API读取 → 展示
```

## 推荐实施路径

### 第一阶段（立即实施）：前端RSS聚合
修改现有HTML，添加JavaScript代码直接从RSS源获取数据：

```javascript
// 添加到index.html的script部分
async function fetchFromRSS() {
    const rssSources = [
        'https://hnrss.org/newest?q=ai',
        'http://export.arxiv.org/rss/cs.AI'
    ];
    
    // 使用rss2json API转换
    const apiUrl = `https://api.rss2json.com/v1/api.json?rss_url=${encodeURIComponent(rssSources[0])}`;
    const response = await fetch(apiUrl);
    const data = await response.json();
    
    // 更新页面显示
    updateNewsDisplay(data.items);
}

// 页面加载时执行
fetchFromRSS();
// 每30分钟刷新一次
setInterval(fetchFromRSS, 30 * 60 * 1000);
```

### 第二阶段（1周内）：GitHub Actions自动化
按照方案一完整实现，建立稳定的自动化更新流程。

### 第三阶段（长期优化）：混合方案
- GitHub Actions负责每日批量更新
- 前端RSS聚合提供实时性补充
- 用户提交功能（可选）

## 社交分享优化建议

### 1. 分享激励设计
- **分享按钮位置优化**：在文章详情页底部、悬浮侧边栏都放置分享按钮
- **分享文案预设**：为不同平台定制最佳文案格式
- **二维码分享**：生成文章专属二维码，方便微信扫码分享

### 2. 病毒式传播机制
```javascript
// 添加分享统计和奖励
function trackShare(platform) {
    // 记录分享行为
    console.log(`Shared to ${platform}`);
    
    // 可以添加积分系统或徽章
    showShareReward();
}

function showShareReward() {
    // 显示"感谢分享"提示
    // 或者解锁特殊功能
}
```

### 3. SEO优化
已在HTML中添加Open Graph标签，确保分享到社交媒体时显示精美预览：
```html
<meta property="og:title" content="文章标题">
<meta property="og:description" content="文章摘要">
<meta property="og:image" content="缩略图URL">
```

## 下一步行动清单

- [ ] 测试当前版本的分享功能（微信、小红书、微博、Twitter）
- [ ] 添加前端RSS聚合代码（最快见效）
- [ ] 创建GitHub Actions工作流（稳定自动化）
- [ ] 设计分享海报/图片生成功能
- [ ] 添加用户订阅功能（邮件/RSS）
- [ ] 监控访问数据和分享转化率

## 成本估算

| 方案 | 月度成本 | 维护难度 | 可靠性 |
|------|---------|---------|--------|
| GitHub Actions | $0（免费额度内） | 低 | 高 |
| RSS前端聚合 | $0 | 极低 | 中（依赖第三方API） |
| 云函数方案 | ¥0-50（免费额度） | 中 | 高 |

**推荐组合**：GitHub Actions + 前端RSS聚合，零成本且兼顾稳定性和实时性。
