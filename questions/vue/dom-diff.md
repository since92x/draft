# diff 和 patch

- diff 是比较虚拟 dom，返回 patch 对象，存储连个节点的不同
- patch 是给真实 dom 打补丁

# diff 算法(vue)

- 订阅者 setter
- Dep.notify()
- patch(oldVnode, Vnode)
- isSameVnode(oldVnode, Vnode)
- no: replace => return Vnode
- yes: patchVnode
  -- oldVnode 有子节点, Vnode 没有
  -- oldVnode 没有子节点, Vnode 有
  -- 都只有文本节点
  -- 都有子节点
