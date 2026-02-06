# D3D11 迁移说明（保持 DLL 导出接口不变）

## 现状约束

这个插件通过 `gmapi` 直接读写 GameMaker 8 的纹理结构：`gm::GMTEXTURE.texture` 的类型是 `IDirect3DTexture8*`，并且绘制链路依赖 GM8 的 D3D8 渲染器。

这意味着 **不能在不改调用方式的前提下，把最终纹理对象改成 D3D11 资源**，否则会破坏 ABI。

## 过时点定位

原实现中最过时/最容易在新环境出问题的部分是：

- 依赖 `d3dx8.h` / `d3dx8.lib`。
- 使用 `D3DXCreateTexture` + `D3DXFillTexture` 创建并上传字形纹理。

`D3DX` 整体已废弃多年，现代系统/工具链经常缺失对应组件。

## 本次改动（兼容性迁移）

在保持导出函数完全不变的前提下，做了以下替换：

1. 删除 `d3dx8` 头文件和库依赖。
2. 改为使用 `IDirect3DDevice8::CreateTexture` 创建 `A8R8G8B8` 纹理。
3. 通过 `IDirect3DTexture8::LockRect/UnlockRect` 直接写入 GDI+ 生成的 BGRA 像素。

这样做等价于将「D3DX 纹理填充」迁移为「底层 D3D API 直接上传」，通常对新驱动环境更稳，同时不改变 DLL 接口。

## 若要进一步“真正 D3D11 化”

若必须走 D3D11，需要同时升级宿主（GM8）渲染接口或增加桥接层（例如把 D3D11 资源再桥接回 D3D8 可见纹理），这已经超出“保持原 DLL 调用方法不变”的范围。
