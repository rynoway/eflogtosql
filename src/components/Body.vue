<template>
  <main class="flex-grow p-6 bg-gray-50 h-[90vh]">
    <div class="max-w-[95%] mx-auto flex h-[calc(90vh-12rem)]">
      <!-- 主要内容区域 -->
      <div class="flex-grow grid grid-cols-2 gap-6 min-w-0 overflow-hidden">
        <!-- 左侧编辑框 -->
        <div class="w-full min-w-0 flex flex-col overflow-hidden">
          <div class="mb-2 flex-none">
            <label class="block text-sm font-medium text-gray-700">EF Core Log</label>
          </div>
          <div class="flex-1 min-h-0 relative">
            <pre class="editor-panel h-full border border-gray-300"><code
              ref="codeInput"
              class="block w-full h-full outline-none break-all whitespace-pre-wrap"
              contenteditable="true"
              @input="handleInput"
              spellcheck="false"
            ></code></pre>
          </div>
        </div>
        
        <!-- 右侧编辑框 -->
        <div class="w-full min-w-0 flex flex-col overflow-hidden">
          <div class="mb-2 flex-none">
            <label class="block text-sm font-medium text-gray-700">SQL Query</label>
          </div>
          <div class="flex-1 min-h-0 relative">
            <pre class="editor-panel h-full border border-gray-300 margin-top-0"><code
              ref="codeOutput"
              class="language-sql block w-full h-full break-all whitespace-pre-wrap"
              v-html="rightCode"
            ></code></pre>
            <!-- 复制按钮 -->
            <div 
              v-if="rightCode"
              class="absolute top-2 right-2 z-10"
            >
              <button 
                @click="handleCopy"
                class="bg-white/80 hover:bg-white text-gray-500 hover:text-gray-700 p-1.5 rounded-md shadow-sm border border-gray-200 transition-all duration-200 group"
                :class="{ 'text-green-500 border-green-200 bg-green-50/80 hover:bg-green-50': copySuccess }"
                title="复制到剪贴板"
              >
                <svg 
                  v-if="!copySuccess"
                  xmlns="http://www.w3.org/2000/svg" 
                  class="h-5 w-5 transform group-active:scale-90 transition-transform duration-100" 
                  viewBox="0 0 20 20" 
                  fill="currentColor"
                >
                  <path d="M8 3a1 1 0 011-1h2a1 1 0 110 2H9a1 1 0 01-1-1z" />
                  <path d="M6 3a2 2 0 00-2 2v11a2 2 0 002 2h8a2 2 0 002-2V5a2 2 0 00-2-2 3 3 0 01-3 3H9a3 3 0 01-3-3z" />
                </svg>
                <svg
                  v-else
                  xmlns="http://www.w3.org/2000/svg"
                  class="h-5 w-5 transform group-active:scale-90 transition-transform duration-100"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                >
                  <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- 历史记录侧边栏 -->
      <div class="w-96 ml-6 bg-white rounded-lg shadow p-4 h-full overflow-auto shrink-0">
        <h3 class="text-lg font-medium text-gray-900 mb-4">最近记录</h3>
        <div class="space-y-3">
          <div
            v-for="(record, index) in historyRecords.slice(0, 6)"
            :key="index"
            class="p-3 bg-gray-50 rounded-md hover:bg-gray-100 cursor-pointer"
            @click="loadHistory(record)"
          >
            <div class="text-sm text-gray-600">{{ record.timestamp }}</div>
            <div class="text-sm font-medium text-gray-900 truncate">{{ record.preview }}</div>
          </div>
        </div>
      </div>
    </div>
      
    <!-- 按钮组 -->
    <div class="flex justify-center space-x-4 mt-4">
      <button 
        @click="handleConvert"
        class="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 flex items-center"
        :disabled="converting"
      >
        <svg v-if="converting" class="animate-spin -ml-1 mr-2 h-4 w-4 text-white" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
        {{ converting ? '转换中...' : '转换' }}
      </button>
      <button 
        @click="handleReset"
        class="px-6 py-2 bg-gray-600 text-white rounded-md hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
      >
        重置
      </button>
      <button 
        @click="showTableStructure"
        :disabled="!rightCode"
        class="px-6 py-2 bg-purple-600 text-white rounded-md hover:bg-purple-700 focus:outline-none focus:ring-2 focus:ring-purple-500 focus:ring-offset-2"
      >
        表结构
      </button>
    </div>

    <!-- 表结构弹窗 -->
    <TableStructureModal
      v-if="showTableModal"
      :structure="tableStructure"
      @close="showTableModal = false"
    />
  </main>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import Prism from 'prismjs'
import debounce from 'lodash/debounce'
import TableStructureModal from './TableStructureModal.vue'
import 'prismjs/themes/prism.css'
import 'prismjs/components/prism-sql'
import 'prismjs/components/prism-csharp'

const codeInput = ref(null)
const codeOutput = ref(null)
const leftCode = ref('')
const rightCode = ref('')
const converting = ref(false)
const historyRecords = ref([])
const showToast = ref(false)
const showTableModal = ref(false)
const tableStructure = ref([])
const copySuccess = ref(false)

// 处理输入变化
const handleInput = debounce(async (e) => {
  leftCode.value = e.target.textContent || ''
}, 300)

// 高亮输出框内容
const highlightOutput = async () => {
  if (!codeOutput.value) return
  await nextTick()
  Prism.highlightElement(codeOutput.value)
}

// 加载历史记录
const loadHistory = async (record) => {
  if (!record) return
  
  // 设置左侧编辑框内容
  leftCode.value = record.input
  if (codeInput.value) {
    codeInput.value.textContent = record.input
  }
  
  // 设置右侧输出内容
  rightCode.value = record.output
  await highlightOutput()
}

// 保存历史记录
const saveHistory = () => {
  const timestamp = new Date().toLocaleString()
  const preview = leftCode.value.slice(0, 50) + (leftCode.value.length > 50 ? '...' : '')
  
  historyRecords.value.unshift({
    timestamp,
    preview,
    input: leftCode.value,
    output: rightCode.value
  })
  
  // 限制历史记录数量为6条
  if (historyRecords.value.length > 6) {
    historyRecords.value = historyRecords.value.slice(0, 6)
  }
  
  // 保存到localStorage
  localStorage.setItem('history', JSON.stringify(historyRecords.value))
}

// 初始化时加载历史记录
onMounted(() => {
  const savedHistory = localStorage.getItem('history')
  if (savedHistory) {
    try {
      historyRecords.value = JSON.parse(savedHistory)
    } catch (err) {
      console.error('加载历史记录失败:', err)
    }
  }
})

// 转换按钮点击事件
const handleConvert = async () => {
  if (!leftCode.value) return
  
  converting.value = true
  try {
    // 转换逻辑...
    const efLog = leftCode.value
    const sql = convertEFToSQL(efLog)
    const highlightedSql = Prism.highlight(sql, Prism.languages.sql, 'sql')
    rightCode.value = highlightedSql
    
    // 保存历史记录
    saveHistory()
    
  } catch (err) {
    console.error('转换失败:', err)
  } finally {
    converting.value = false
  }
}

// 重置按钮点击事件
const handleReset = () => {
  if (codeInput.value) {
    codeInput.value.textContent = ''
  }
  leftCode.value = ''
  rightCode.value = ''
}

// 复制按钮点击事件
const handleCopy = async () => {
  try {
    // 获取右侧编辑框的纯文本内容
    const content = codeOutput.value?.textContent || '';
    await navigator.clipboard.writeText(content);
    
    // 显示复制成功状态
    copySuccess.value = true;
    setTimeout(() => {
      copySuccess.value = false;
    }, 1500);
  } catch (err) {
    console.error('复制失败:', err);
  }
};

// 表结构按钮点击事件
const showTableStructure = () => {
  if (!rightCode.value) {
    return
  }

  // 获取原始SQL（移除HTML标签）
  const tempDiv = document.createElement('div')
  tempDiv.innerHTML = rightCode.value
  const sql = tempDiv.textContent

  tableStructure.value = analyzeTableStructure(sql)
  showTableModal.value = true
}

// 转换EF Core日志为SQL
const convertEFToSQL = (efLog) => {
  // 按行分割日志
  const lines = efLog.split('\n')
  let sqlParts = []
  let currentSql = []
  let isCollectingSql = false

  for (const line of lines) {
    // 检查是否是命令行开始
    if (line.includes('DbCommand')) {
      isCollectingSql = true
      currentSql = []
      continue
    }

    // 如果正在收集SQL并且不是空行
    if (isCollectingSql && line.trim()) {
      // 如果遇到新的时间戳或其他日志标记，说明SQL结束
      if (line.match(/^\[\d{4}-\d{2}-\d{2}/)) {
        if (currentSql.length > 0) {
          sqlParts.push(formatSql(currentSql.join('\n')))
        }
        isCollectingSql = false
        currentSql = []
        continue
      }

      // 收集SQL行
      currentSql.push(line.trim())
    }
  }

  // 处理最后一个SQL语句
  if (currentSql.length > 0) {
    sqlParts.push(formatSql(currentSql.join('\n')))
  }

  return sqlParts.join('\n\n')
}

// 格式化SQL
const formatSql = (sql) => {
  // 提取所有表名和别名的映射
  const tableAliasMap = new Map()
  const fromRegex = /FROM\s+\[([^\]]+)\]\s+AS\s+\[([^\]]+)\]/gi
  const joinRegex = /JOIN\s+\[([^\]]+)\]\s+AS\s+\[([^\]]+)\]/gi
  
  let match
  while ((match = fromRegex.exec(sql)) !== null) {
    const tableName = match[1]
    const shortTableName = tableName.split('_').pop() // 获取最后一个下划线后的部分
    tableAliasMap.set(match[2], `_${shortTableName}`)
  }
  
  while ((match = joinRegex.exec(sql)) !== null) {
    const tableName = match[1]
    const shortTableName = tableName.split('_').pop()
    tableAliasMap.set(match[2], `_${shortTableName}`)
  }

  // 替换所有别名
  tableAliasMap.forEach((newAlias, oldAlias) => {
    const aliasRegex = new RegExp(`\\[${oldAlias}\\]`, 'g')
    sql = sql.replace(aliasRegex, `[${newAlias}]`)
  })

  // 美化SQL关键字
  const keywords = ['SELECT', 'FROM', 'WHERE', 'ORDER BY', 'GROUP BY', 'HAVING', 'JOIN', 'LEFT JOIN', 'RIGHT JOIN', 'INNER JOIN', 'ON', 'AND', 'OR', 'IN', 'EXISTS', 'NOT EXISTS', 'UNION', 'UNION ALL']
  
  keywords.forEach(keyword => {
    const regex = new RegExp(`\\b${keyword}\\b`, 'gi')
    sql = sql.replace(regex, `\n${keyword}`)
  })

  // 处理子查询和IN子句中的值
  sql = sql.replace(/\(\s*SELECT/gi, '\n(SELECT')
  sql = sql.replace(/IN\s*\(([\s\S]*?)\)/gi, (match, p1) => {
    const values = p1.split(',').map(v => v.trim())
    if (values.length > 5) {
      return `IN (\n  ${values.join(',\n  ')}\n)`
    }
    return match
  })

  // 缩进处理
  const lines = sql.split('\n').map(line => line.trim()).filter(line => line)
  let indentLevel = 0
  const formattedLines = lines.map(line => {
    if (line.includes(')')) indentLevel = Math.max(0, indentLevel - 1)
    const indent = '    '.repeat(indentLevel)
    if (line.includes('(') && !line.includes(')')) indentLevel++
    return indent + line
  })

  return formattedLines.join('\n')
}

// 分析SQL中的表结构
const analyzeTableStructure = (sql) => {
  const tables = []
  const fromRegex = /FROM\s+\[([^\]]+)\]\s+AS\s+\[([^\]]+)\]/gi
  const joinRegex = /JOIN\s+\[([^\]]+)\]\s+AS\s+\[([^\]]+)\]/gi
  const selectRegex = /SELECT\s+([\s\S]+?)(?=\s+FROM)/i

  // 提取SELECT字段
  const selectMatch = sql.match(selectRegex)
  let fields = []
  if (selectMatch) {
    fields = selectMatch[1]
      .split(',')
      .map(field => field.trim())
      .filter(field => field)
      .map(field => {
        // 匹配 [alias].[field] 或 [field] 格式
        const match = field.match(/(?:\[([^\]]+)\]\.)?(?:\[([^\]]+)\]|\[?([^\],\s]+)\]?)/i)
        if (match) {
          const alias = match[1]
          const fieldName = match[2] || match[3]
          return {
            alias: alias || null,
            name: fieldName,
            fullField: field.trim()
          }
        } else {
          console.error(`无法匹配字段：${field}`)
        }
        return null
      })
      .filter(field => field)
  } else {
    console.error('无法匹配SELECT语句:', sql)
  }

  // 提取表名和别名，并关联字段
  const processTable = (match) => {
    if (match) {
      const tableName = match[1]
      const alias = match[2]
      const tableFields = fields.filter(f => f.alias === alias)
      
      tables.push({
        name: tableName,
        alias: alias,
        fields: tableFields.map(field => ({
          name: field.name,
          fullField: field.fullField
        }))
      })
    }
  }

  // 处理FROM子句
  let match
  while ((match = fromRegex.exec(sql)) !== null) {
    processTable(match)
  }

  // 处理JOIN子句
  while ((match = joinRegex.exec(sql)) !== null) {
    processTable(match)
  }

  return tables
}
</script>

<style scoped>
.editor-panel {
  @apply w-full overflow-y-auto overflow-x-hidden bg-gray-50 p-4 rounded-lg;
  font-family: 'Fira Code', monospace;
  max-height: 100%;
  height: 100%;
}

/* 确保编辑器内的文本节点继承字体 */
.editor-panel * {
  font-family: inherit !important;
  white-space: pre-wrap !important;
  word-break: break-all !important;
  word-wrap: break-word !important;
  overflow-wrap: break-word !important;
}

.editor-panel code {
  background: transparent !important;
  padding: 0 !important;
  margin: 0 !important;
  font-size: inherit !important;
  border: none !important;
  box-shadow: none !important;
  width: 100% !important;
  display: block !important;
  max-width: 100% !important;
  height: 100% !important;
}

/* 自定义滚动条样式 */
.editor-panel::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.editor-panel::-webkit-scrollbar-track {
  @apply bg-gray-100;
}

.editor-panel::-webkit-scrollbar-thumb {
  @apply bg-gray-400 rounded-full;
}

.editor-panel::-webkit-scrollbar-thumb:hover {
  @apply bg-gray-500;
}

/* Token 样式覆盖 */
:deep(.token) {
  word-break: break-all !important;
  white-space: pre-wrap !important;
  overflow-wrap: break-word !important;
}

:deep(.token.operator),
:deep(.token.string),
:deep(.token.number) {
  word-break: keep-all !important;
}

/* 复制按钮悬停效果 */
.group-active\:scale-90 {
  transform: scale(0.9);
}
pre{
  margin-top: 0;;
}
</style>
