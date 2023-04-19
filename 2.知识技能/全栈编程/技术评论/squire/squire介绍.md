## 为什么使用squire editor
1.之前有团队用squire开发邮件编辑功能，继续沿用是一个考虑因素。
2.squire声称自己是为邮件编辑而生，支持对任意结构的html进行编辑，并且具有良好的跨浏览器兼容性。
3.轻量，对已有工程没有侵入性的改变。
4.在Fastmail中使用，有实际案例。

## squire实现富文本编辑器的原理
1.设置contenteditable属性控制html中输可以入文字。通过控制dom结构及光标，确保浏览器在确定的元素中输入文字。
2.为了保证跨浏览器兼容性，使用js实现了文本编辑的控制操作（换行，删除等），添加/查询/删除格式操作（设置文字颜色，设置列表等）。
3.为了实现撤销与还原，维护了一个history stack。选区也要支持还原撤销。
4.对编辑内容做了一定程度的规整/清理，保证样式与结构的一致性。
5.行内及块级样式，通过使用tag包装 + inline style + class的方式实现。
6.光标等选区操作，通过Web selection api实现。

js实现编辑控制操作及格式操作，需要进行大量Dom Tree查询/操作，以及比较复杂的选区操作。

## 工程实现
Editor 主体
Clean 负责清理、善后工作
Constants 常量
node/ 节点操作
range/ 选区操作
EditorAction 编辑器编辑动作
KeyHandlers 键盘，快捷键处理，

## 核心逻辑、算法
Html层级规划
    行内 inline
    块级 block
    容器级 container

Dom Tree遍历

Dom 查询
    查询节点

Dom 操作
    替换内容
    inline拆分/合并
    block拆分/合并

选区操作
    查询选区
    设置选区向下从text开始，text结束
    设置选区向上从指定的节点开始和结束
    暂存/恢复选区

规整操作
    inline包装
    空block填充br
    整理container结构
    底部添加空行

## 部分注释
Constants.ts

Editor.ts

    mutation:MutationObserver 判断文档是否改变
    getGlobal 获取运行环境
    sanitizeToDOMFragment sanitize html
    setConfig 合并生成最终配置
    createDefaultBlock 创建默认块级结构
    didError 打印错误
    getRoot 获取根节点
    modifyDocument document改编后，重新监听mutation
    fireEvent 触发自定义事件
    destroy 编辑器销毁
    handleEvent 处理自定义事件
    addEventListener 添加自定义事件监听
    removeEventListener 删除自定义事件监听

    createRange 创建选区
    getCursorPosition 获取光标坐标位置
    _moveCursorTo 移动光标至
    moveCursorToStart 移动光标至开始
    moveCursorToEnd 移动光标至末尾
    setSelection 设置选区
    getSelection 获取选区
    enableRestoreSelection ???
    disableRestoreSelection  ???
    restoreSelection 还原选区

    getSelectedText 获取选中的文字
    getPath 获取路径
    _didAddZWS 设置零宽字符
    _removeZWS 删除零宽字符

    _updatePath 更新路径
    _updatePathOnEvent 更新路径

    focus 聚焦
    blur 失焦

    _saveRangeToBookmark 暂时保存选区
    _getRangeAndRemoveBookmark 获取选区

    _docWasChanged 文档已改变

    _recordUndoState 记录可撤销状态
    saveUndoState 记录可撤销状态

    undo 撤销
    redo 还原

    hasFormat 判断是否有格式
    getFontInfo 获取字体信息
    _addFormat 添加格式
    _removeFormat 删除格式
    changeFormat 变更格式

    forEachBlock 遍历块级元素
    modifyBlocks 更改块级

    increaseListLevel 增加列表层级
    decreaseListLevel 减少列表层级

    _ensureBottomLine 检查确保底部线条

    setKeyHandler 设置KeyHandler
    _getHTML 获取html
    _setHTML 设置html
    getHTML 获取html
    setHTML 设置html

    insertElement 插入元素
    insertImage 插入image
    insertHTML 插入html

    escapeHTML 转移html
    insertPlainText 插入字符串

    command 执行命令
    addStyles 增加样式
    makeLink 添加link
    removeLink 删除link

    setFontFace 设置字体
    setFontSize 设置字号
    setTextColour 设置文字颜色
    setHighlightColour 设置高亮颜色

    setTextAlignment 设置字体缩进
    addPre 增加pre块
    removePre 删除pre块

    code 增加code块
    removeCode 删除code块
    toggleCode toggle code块

    removeFormatting 清除格式

    removeAllFormatting 清除所有格式

    increaseQuoteLevel 增加quote块层级
    decreaseQuoteLevel 减少quote快层级

    makeUnorderedList 设置无序列表
    makeOrderedList 设置有序列表
    removeList 删除列表


EditorAction.ts 可以理解封装好的工具函数

    addLinks 添加link
    mergeObjects 合并对象
    decreaseBlockQuoteLevel 减少块级quote层级
    increaseBlockQuoteLevel 增加块级quote层级
    removeBlockQuote 删除块级quote

    makeList 设置列表
    makeUnorderedList 设置无序列表
    makeOrderedList 设置有序列表
    removeList 删除列表

    getListSelection 获取列表选区
    removeZWS 删除零宽字符
    splitBlock 分割块级元素

KeyHandlers.ts

    _onKey 处理键盘事件
    mapKeyTo 映射按键到处理函数
    mapKeyToFormat 映射按键到设置格式
    afterDelete 删除后逻辑
    detachUneditableNode 分离不可编辑节点
    linkifyText 文字增加链接

    handleEnter 处理enter

    keyHandlers
        enter 处理enter
        shift-enter 
        backspace 处理删除
        delete 处理删除
        tab 处理锁进
        shift-tab 处理反缩进
        space 处理空格
        left 处理向左
        right 处理向右

    changeIndentationLevel 改变缩进层级
    toggleList toggle列表

Clean.ts 对文档进行清理

    fontSizes 字体大小
    styleToSemantic 样式语义化
    replaceWithTag 使用tag替换
    replaceStyles 替换样式

    stylesRewriters 样式重写器

    allowedBlock 允许的块级
    blacklist 块级列表

    cleanTree 清理树
    removeEmptyInlines 删除空的inline

    notWSTextNode 是否是空白字符 ？？？
    isLineBreak 是否是行内换行

    cleanupBRs 删除br

Clipboard.ts

    setClipboardData 设置剪切板数据
    _onCut 处理剪切事件
    _onCopy 处理复制事件
    onCopy 处理复制事件

    _monitorShiftKey 检测shift是否按下
    _onPaste 处理粘贴事件
    _onDrop 处理drop事件


node/

    Block.ts

        getBlockWalker 获取块级迭代器
        getPreviousBlock 获取前一个块级
        getNextBlock 获取下一个块级
        isEmptyBlock 是否是空块级

    Category.ts 元素进行分类

        inlineNodeNames 行内节点名
        leafNodeNames 叶子节点名

        resetNodeCategoryCache ？？？

        isLeaf 是否是叶子节点
        getNodeCategory 获取节点分类

        isInline 是否是行内
        isBlock 是否是块级
        isContainer 是否是容器

    MergeSplit.ts

        fixCursor ？？？
        fixContainer ？？？

        split 分割
        _mergeInlines 合并行内
        mergeInlines 合并行内

        mergeWithBlock ？？？
        mergeContainers ？？？

    Node.ts

        createElement 创建元素
        isOrContains 是还是包含
        areAlike 两个元素是否类似
        hasTagAttributes 是否有元素属性

        getNearest 向上获取最近的
        getNodeBeforeOffset 向前位移获取节点
        getNodeAfterOffset 向后位移获取节点

        getLength 获取子节点长度
        empty ？？？
        detach 从父节点上剥离

        replaceWith 同一个父节点下替换

    Path.ts

        getPath 获取路径 ？？？
    TreeIterator.ts

        typeToBitArray 节点类型映射
            TreeIterator 
                nextNode 下一个节点
                previousNode 前一个节点
                previousPONode 反向前一个节点？？？
range/

    Block.ts
        getStartBlockOfRange 选区开始的块级
        getEndBlockOfRange 选区结束的块级
        isContent 是否是内容 ？？？

        rangeDoesStartAtBlockBoundary 选区是否从块级开始
        rangeDoesEndAtBlockBoundary 选区是否在块级结束
        expandRangeToBlockBoundaries 扩展选区到一个块级边界

    Boundaries.ts
        isNodeContainedInRange ？？？
        moveRangeBoundariesDownTree 
        moveRangeBoundariesUpTree
        moveRangeBoundaryOutOf

    InsertDelete.ts
        insertNodeInRange 选区中插入节点
        extractContentsOfRange 从选区中抽取内容
        getAdjacentInlineNode 获取邻近的行内节点
        deleteContentsOfRange 删除选区内容
        insertTreeFragmentIntoRange 选区插入树


## Bug review
1.编辑一段时间，发现文字的样式不对了。

cleanupBRs里下面的逻辑
```
// Recursively examine container nodes and wrap any inline children.
const fixContainer = function (
    container: Node,
    root: Element | DocumentFragment,
): Node {
    const children = container.childNodes;
    let wrapper: HTMLElement | null = null;
    for (let i = 0, l = children.length; i < l; i += 1) {
        const child = children[i];
        const isBR = child.nodeName === 'BR';
        if (!isBR && isInline(child)) {
            if (!wrapper) {
                wrapper = createElement('DIV');
            }
            wrapper.appendChild(child);
            i -= 1;
            l -= 1;
        } else if (isBR || wrapper) {
            if (!wrapper) {
                wrapper = createElement('DIV');
            }
            fixCursor(wrapper, root);
            if (isBR) {
                container.replaceChild(wrapper, child);
            } else {
                container.insertBefore(wrapper, child);
                i += 1;
                l += 1;
            }
            wrapper = null;
        }
        if (isContainer(child)) {
            fixContainer(child, root);
        }
    }
    if (wrapper) {
        container.appendChild(fixCursor(wrapper, root));
    }
    return container;
};
```

2.mobile上，签名里，图片上方删除文字，图片下方会一直增加空行
```
const afterDelete = function (self, range?: Range): void {
    try {
        if (!range) {
            range = self.getSelection();
        }
        let node = range!.startContainer;
        // Climb the tree from the focus point while we are inside an empty
        // inline element
        if (node instanceof Text) {
            node = node.parentNode!;
        }
        let parent = node;
        while (
            isInline(parent) &&
            (!parent.textContent || parent.textContent === ZWS)
        ) {
            node = parent;
            parent = node.parentNode!;
        }
        // If focused in empty inline element
        if (node !== parent) {
            // Move focus to just before empty inline(s)
            range!.setStart(parent, indexOf.call(parent.childNodes, node));
            range!.collapse(true);
            // Remove empty inline(s)
            parent.removeChild(node);
            // Fix cursor in block
            if (!isBlock(parent)) {
                parent = getPreviousBlock(parent, self._root) || self._root;
            }
            fixCursor(parent, self._root);
            // Move cursor into text node
            moveRangeBoundariesDownTree(range!);
        }
        // If you delete the last character in the sole <div> in Chrome,
        // it removes the div and replaces it with just a <br> inside the
        // root. Detach the <br>; the _ensureBottomLine call will insert a new
        // block.
        if (
            node === self._root &&
            (node = node.firstChild!) &&
            node.nodeName === 'BR'
        ) {
            detach(node);
        }
        self._ensureBottomLine();
        self.setSelection(range);
        self._updatePath(range, true);
    } catch (error) {
        self.didError(error);
    }
};
```

```
proto._ensureBottomLine = function (this: Squire): void {
    const root = this._root;
    const last = root.lastElementChild;
    if (!last || last.nodeName !== this._config.blockTag || !isBlock(last)) {
        root.appendChild(this.createDefaultBlock());
    }
};
```

```
const mergeContainers = function (node: Node, root: Element): void {
    const prev = node.previousSibling;
    const first = node.firstChild;
    const isListItem = node.nodeName === 'LI';

    // Do not merge LIs, unless it only contains a UL
    if (isListItem && (!first || !/^[OU]L$/.test(first.nodeName))) {
        return;
    }

    if (prev && areAlike(prev, node)) {
        if (!isContainer(prev)) {
            if (isListItem) {
                const block = createElement('DIV');
                block.appendChild(empty(prev));
                prev.appendChild(block);
            } else {
                return;
            }
        }
        detach(node);
        const needsFix = !isContainer(node);
        prev.appendChild(empty(node));
        if (needsFix) {
            fixContainer(prev, root);
        }
        if (first) {
            mergeContainers(first, root);
        }
    } else if (isListItem) {
        const block = createElement('DIV');
        node.insertBefore(block, first);
        fixCursor(block, root);
    }
};
```

delete的时候，把最后一行加的默认段落又删掉了，其实并不是删掉了，而是移到了gmail signature的外层div的里面的最后