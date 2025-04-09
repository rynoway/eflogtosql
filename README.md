# EFToSQL - Entity Framework Core Log to SQL Converter

一个简单高效的在线工具，用于将 Entity Framework Core 日志转换为格式化的 SQL 查询语句。

## 特性

- 🚀 实时转换 EF Core 日志到 SQL
- 💅 美观的语法高亮显示
- 📋 一键复制 SQL 语句
- 📊 智能表结构分析
- 📝 本地历史记录

## 预览

![eflogtosql 预览](/demo.png)

## 技术栈

- **前端框架**: Vue 3
- **构建工具**: Vite
- **样式**: Tailwind CSS
- **代码高亮**: Prism.js
- **工具库**: Lodash

## 本地开发

### 环境要求

- Node.js >= 16
- npm >= 8

### 安装步骤

1. 克隆仓库
```bash
git clone https://github.com/rynoway/eflogtosql
cd eftosql
```

2. 安装依赖
```bash
npm install
```

3. 启动开发服务器
```bash
npm run dev
```

4. 构建生产版本
```bash
npm run build
```

## 使用说明

1. 将 EF Core 日志粘贴到左侧输入框
2. 点击"转换"按钮
3. 格式化的 SQL 将显示在右侧面板
4. 可以点击"复制"按钮复制 SQL
5. 点击"表结构"按钮查看表结构分析
6. 历史记录会自动保存在本地

## 项目结构

```
eftosql/
├── src/
│   ├── components/     # Vue 组件
│   ├── assets/         # 静态资源
│   ├── App.vue         # 根组件
│   └── main.js         # 入口文件
├── public/             # 公共资源
├── index.html          # HTML 模板
├── vite.config.js      # Vite 配置
└── package.json        # 项目配置
```

## 主要功能

### SQL 转换
- 支持复杂的 EF Core 日志格式
- 智能解析表别名
- 优化字段匹配
- 支持多条 SQL 语句

### 表结构分析
- 自动提取表名和字段
- 分析表间关系
- 显示字段列表

### 用户界面
- 响应式双面板设计
- 动态高度控制
- 溢出处理
- 复制成功反馈

## 许可

[MIT License](LICENSE)
