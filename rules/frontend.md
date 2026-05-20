# 前端规则

> 适用：React + TypeScript + Ant Design 6

## React

- 必须使用函数组件与 Hooks，禁止使用类组件（class component）
- 禁止使用 TypeScript `any` 类型，所有类型必须明确声明
- 禁止页面宽高抖动：Select 与 Input 组件须使用固定宽度，不得因选中内容长度变化导致布局偏移；Table 列宽须明确指定

## Ant Design 6

- 禁止使用已废弃的 API
- Notification 组件禁止使用 `message` 字段，统一使用 `title` 字段
- Drawer 组件禁止使用 `width` 与 `destroyOnClose` 属性，统一使用 `size` 与 `destroyOnHidden` 属性
- 禁止新增 `antd` 的 `List` 组件用法，优先使用原生语义容器元素

## 大规模树结构

必须满足：

- 使用虚拟滚动
- 使用扁平投影（flat projection）
- 使用稳定且唯一的 key
- 展开状态操作的时间复杂度为 O(1)

禁止：

- 深层递归渲染
- 每次渲染时重复创建内联函数（应使用 useCallback 或提取为组件外部函数）
- 每次渲染时重新构建树形数据结构（应在 useMemo 中或仅在数据变更时构建）
