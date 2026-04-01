# 🛠️ 实战指南（Practice Guide）

> 本篇目标：学会在实际项目中使用 Primer React，掌握常见场景的实现方式、最佳实践和问题排查技巧。

---

## 🚀 项目集成

### Next.js 项目集成

```tsx
// app/layout.tsx（App Router）
import {ThemeProvider, BaseStyles} from '@primer/react'

export default function RootLayout({children}: {children: React.ReactNode}) {
  return (
    <html lang="zh-CN">
      <body>
        <ThemeProvider colorMode="auto" preventSSRMismatch>
          <BaseStyles>{children}</BaseStyles>
        </ThemeProvider>
      </body>
    </html>
  )
}
```

**注意事项**：

- Next.js App Router 中必须设置 `preventSSRMismatch` 避免水合错误
- 如果使用 Pages Router，在 `_app.tsx` 中添加 ThemeProvider

### Vite 项目集成

```tsx
// main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import {ThemeProvider, BaseStyles} from '@primer/react'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <ThemeProvider colorMode="auto">
      <BaseStyles>
        <App />
      </BaseStyles>
    </ThemeProvider>
  </React.StrictMode>,
)
```

---

## 📋 常见场景实现

### 场景一：带筛选的操作菜单

```tsx
import {ActionMenu, ActionList, TextInput} from '@primer/react'
import {FilterIcon, CheckIcon} from '@primer/octicons-react'
import {useState} from 'react'

function FilterMenu() {
  const [selected, setSelected] = useState<string[]>([])
  const options = ['Bug', 'Feature', 'Enhancement', 'Documentation']

  return (
    <ActionMenu>
      <ActionMenu.Button leadingVisual={FilterIcon}>
        筛选标签 {selected.length > 0 && `(${selected.length})`}
      </ActionMenu.Button>
      <ActionMenu.Overlay width="medium">
        <ActionList selectionVariant="multiple">
          {options.map(option => (
            <ActionList.Item
              key={option}
              selected={selected.includes(option)}
              onSelect={() => {
                setSelected(prev => (prev.includes(option) ? prev.filter(s => s !== option) : [...prev, option]))
              }}
            >
              {option}
            </ActionList.Item>
          ))}
        </ActionList>
      </ActionMenu.Overlay>
    </ActionMenu>
  )
}
```

### 场景二：确认对话框

```tsx
import {Button, Dialog} from '@primer/react'
import {useState, useCallback, useRef} from 'react'

function ConfirmDeleteDialog() {
  const [isOpen, setIsOpen] = useState(false)
  const returnFocusRef = useRef<HTMLButtonElement>(null)

  const handleClose = useCallback(() => setIsOpen(false), [])

  const handleConfirm = useCallback(() => {
    // 执行删除操作
    console.log('已删除')
    setIsOpen(false)
  }, [])

  return (
    <>
      <Button ref={returnFocusRef} variant="danger" onClick={() => setIsOpen(true)}>
        删除仓库
      </Button>

      {isOpen && (
        <Dialog
          title="确认删除"
          onClose={handleClose}
          returnFocusRef={returnFocusRef}
          footerButtons={[
            {content: '取消', onClick: handleClose},
            {
              content: '确认删除',
              buttonType: 'danger',
              onClick: handleConfirm,
            },
          ]}
        >
          <p>此操作不可撤销。确定要删除该仓库吗？</p>
        </Dialog>
      )}
    </>
  )
}
```

### 场景三：响应式页面布局

```tsx
import {PageLayout, NavList, Heading, Text} from '@primer/react'
import {HomeIcon, RepoIcon, IssueOpenedIcon} from '@primer/octicons-react'

function DashboardPage() {
  return (
    <PageLayout>
      {/* 侧边栏 - 窄屏时折叠 */}
      <PageLayout.Pane position="start" width="small">
        <NavList>
          <NavList.Item href="/dashboard" aria-current="page">
            <NavList.LeadingVisual>
              <HomeIcon />
            </NavList.LeadingVisual>
            首页
          </NavList.Item>
          <NavList.Item href="/repos">
            <NavList.LeadingVisual>
              <RepoIcon />
            </NavList.LeadingVisual>
            仓库
          </NavList.Item>
          <NavList.Item href="/issues">
            <NavList.LeadingVisual>
              <IssueOpenedIcon />
            </NavList.LeadingVisual>
            议题
          </NavList.Item>
        </NavList>
      </PageLayout.Pane>

      {/* 主内容区 */}
      <PageLayout.Content>
        <Heading as="h1">仪表板</Heading>
        <Text as="p">欢迎回来！</Text>
        {/* 更多内容 */}
      </PageLayout.Content>
    </PageLayout>
  )
}
```

### 场景四：数据表格

```tsx
import {DataTable, Table} from '@primer/react'

const repos = [
  {id: 1, name: 'primer-react', language: 'TypeScript', stars: 3200},
  {id: 2, name: 'octicons', language: 'JavaScript', stars: 8100},
  {id: 3, name: 'css', language: 'CSS', stars: 12400},
]

function RepoTable() {
  return (
    <Table.Container>
      <Table.Title as="h2">热门仓库</Table.Title>
      <DataTable
        data={repos}
        columns={[
          {
            header: '名称',
            field: 'name',
            renderCell: row => <a href={`/${row.name}`}>{row.name}</a>,
          },
          {header: '语言', field: 'language'},
          {
            header: '⭐ Stars',
            field: 'stars',
            align: 'end',
            renderCell: row => row.stars.toLocaleString(),
          },
        ]}
      />
    </Table.Container>
  )
}
```

### 场景五：表单验证

```tsx
import {FormControl, TextInput, Textarea, Button, Stack} from '@primer/react'
import {useState} from 'react'

function IssueForm() {
  const [title, setTitle] = useState('')
  const [body, setBody] = useState('')
  const [submitted, setSubmitted] = useState(false)

  const titleError = submitted && !title ? '标题不能为空' : undefined

  return (
    <form
      onSubmit={e => {
        e.preventDefault()
        setSubmitted(true)
        if (title) {
          console.log({title, body})
        }
      }}
    >
      <Stack direction="vertical" gap="normal">
        <FormControl required>
          <FormControl.Label>标题</FormControl.Label>
          <TextInput
            value={title}
            onChange={e => setTitle(e.target.value)}
            validationStatus={titleError ? 'error' : undefined}
            block
          />
          {titleError && <FormControl.Validation variant="error">{titleError}</FormControl.Validation>}
        </FormControl>

        <FormControl>
          <FormControl.Label>描述</FormControl.Label>
          <Textarea value={body} onChange={e => setBody(e.target.value)} placeholder="详细描述问题..." block />
          <FormControl.Caption>支持 Markdown 格式</FormControl.Caption>
        </FormControl>

        <Button type="submit" variant="primary">
          提交议题
        </Button>
      </Stack>
    </form>
  )
}
```

---

## ✨ 最佳实践

### 1. 语义化使用组件

```tsx
// ✅ 使用 Primer 提供的语义组件
import {Heading, Text, Link} from '@primer/react'

<Heading as="h2">页面标题</Heading>
<Text as="p" color="fg.muted">辅助说明文字</Text>
<Link href="/docs">查看文档</Link>

// ❌ 避免使用原生标签 + 手动样式
<h2 style={{fontSize: '20px', fontWeight: 600}}>页面标题</h2>
<p style={{color: '#656d76'}}>辅助说明文字</p>
<a style={{color: '#0969da'}} href="/docs">查看文档</a>
```

### 2. 正确使用 FormControl

```tsx
// ✅ 使用 FormControl 包裹表单元素
<FormControl>
  <FormControl.Label>邮箱</FormControl.Label>
  <TextInput type="email" />
  <FormControl.Caption>我们不会公开你的邮箱</FormControl.Caption>
</FormControl>

// ❌ 不要单独使用 TextInput
<label>邮箱</label>
<TextInput type="email" />
<span>我们不会公开你的邮箱</span>
```

**为什么？** FormControl 自动：

- 关联 `<label>` 和 `<input>`（通过 `htmlFor`/`id`）
- 关联描述文字（通过 `aria-describedby`）
- 关联错误消息（通过 `aria-errormessage`）
- 处理 `required`、`disabled` 状态的无障碍传播

### 3. 避免样式覆盖的反模式

```tsx
// ✅ 使用组件提供的 variant/size props
<Button variant="primary" size="large">确认</Button>

// ⚠️ 仅在没有合适 prop 时使用 className
<Button className={styles.customSpacing}>特殊间距</Button>

// ❌ 避免使用内联样式覆盖组件样式
<Button style={{backgroundColor: 'red', padding: '20px'}}>
  不推荐
</Button>
```

### 4. 处理加载状态

```tsx
// ✅ 使用组件内置的 loading 状态
;<Button loading loadingAnnouncement="正在提交...">
  提交
</Button>

// ✅ 使用 Spinner 组件
import {Spinner} from '@primer/react'

function LoadingState() {
  return (
    <div style={{display: 'flex', justifyContent: 'center', padding: '40px'}}>
      <Spinner size="large" />
    </div>
  )
}
```

### 5. 无障碍最佳实践

```tsx
// ✅ 图标按钮必须有 aria-label
<IconButton icon={XIcon} aria-label="关闭" />

// ✅ 使用语义化的 HTML 结构
<nav aria-label="主导航">
  <NavList>
    <NavList.Item href="/" aria-current="page">首页</NavList.Item>
  </NavList>
</nav>

// ✅ 提供空状态说明
<Blankslate>
  <Blankslate.Visual>
    <BookIcon size="medium" />
  </Blankslate.Visual>
  <Blankslate.Heading>暂无文档</Blankslate.Heading>
  <Blankslate.Description>
    创建你的第一个文档来开始吧
  </Blankslate.Description>
  <Blankslate.PrimaryAction href="/new">
    新建文档
  </Blankslate.PrimaryAction>
</Blankslate>
```

---

## 🔍 问题排查

### 常见问题 1：样式不生效

**症状**：组件渲染了，但没有样式

**原因**：CSS 文件没有被正确加载

**解决方案**：

```tsx
// 确保 CSS 被导入（如果使用 Primer React 的预编译版本）
import '@primer/react/dist/index.css'
```

或者确保构建工具正确处理了 CSS Modules。

### 常见问题 2：SSR 水合不匹配

**症状**：控制台警告 "Text content does not match server-rendered HTML"

**原因**：ThemeProvider 在服务端和客户端解析出不同的颜色模式

**解决方案**：

```tsx
<ThemeProvider
  colorMode="auto"
  preventSSRMismatch  // ← 添加这个 prop
>
```

### 常见问题 3：焦点管理异常

**症状**：关闭对话框后焦点丢失

**原因**：没有设置 `returnFocusRef`

**解决方案**：

```tsx
const buttonRef = useRef<HTMLButtonElement>(null)

<Button ref={buttonRef} onClick={() => setOpen(true)}>打开</Button>

<Dialog
  returnFocusRef={buttonRef}  // ← 指定焦点返回位置
  onClose={() => setOpen(false)}
>
  内容
</Dialog>
```

### 常见问题 4：TypeScript 类型错误

**症状**：使用 `as` prop 时类型报错

**解决方案**：

```tsx
// 确保导入了正确的类型
import type {ButtonProps} from '@primer/react'

// 如果使用自定义组件作为 as，确保它接受 ref
const MyLink = forwardRef<HTMLAnchorElement, {to: string}>((props, ref) => (
  <a ref={ref} href={props.to}>{props.children}</a>
))

<Button as={MyLink} to="/page">链接按钮</Button>
```

---

## 📊 性能优化建议

### 1. 按需导入

```tsx
// ✅ 直接从主入口导入（已支持 Tree-shaking）
import {Button, TextInput} from '@primer/react'

// 框架的 bundler 会自动去除未使用的代码
```

### 2. 避免不必要的重新渲染

```tsx
// ✅ 使用 useCallback 缓存事件处理函数
const handleSelect = useCallback((item: string) => {
  setSelected(item)
}, [])

// ✅ 使用 useMemo 缓存复杂计算
const filteredItems = useMemo(() => items.filter(item => item.includes(query)), [items, query])
```

### 3. 大列表使用虚拟滚动

对于超过 100 项的列表，优先使用 `DataTable`——它内部集成了 `@tanstack/react-virtual` 进行虚拟滚动。

---

## ✅ 本章小结

| 内容     | 要点                                                 |
| -------- | ---------------------------------------------------- |
| 项目集成 | ThemeProvider + BaseStyles 是必需的根组件            |
| 常见场景 | 筛选菜单、确认对话框、响应式布局、数据表格、表单验证 |
| 最佳实践 | 语义化使用、FormControl 包裹、避免内联样式覆盖       |
| 问题排查 | CSS 加载、SSR 水合、焦点管理、TypeScript 类型        |
| 性能优化 | Tree-shaking、useCallback/useMemo、虚拟滚动          |

---

## ➡️ 下一步

继续阅读 [07 - 扩展学习](./07-further-learning.md)，了解相关知识体系和推荐的延伸阅读资源。
