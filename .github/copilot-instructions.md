# Unity Spline Utility - AI Coding Guidelines

## Project Overview
Unity Spline工具，用于在场景中创建和编辑平滑曲线路径。支持CatMull-Rom和Bezier样条插值算法，可在运行时和编辑器中实时预览和模拟曲线。

## Tech Stack
- **Unity Version**: 2022.3.62f2c1 (LTS)
- **Render Pipeline**: Universal Render Pipeline (URP) 14.0.12
- **Target Platform**: 2D/3D通用

## Architecture & Code Organization

### Expected Directory Structure
```
Assets/
  Scripts/
    Spline/              # 核心样条算法
      SplineBase.cs      # 样条基类
      CatmullRomSpline.cs
      BezierSpline.cs
    Editor/              # 编辑器工具
      SplineEditor.cs    # Scene视图中的可视化编辑
    Components/          # MonoBehaviour组件
      SplineRenderer.cs  # 运行时曲线渲染
      WaypointManager.cs # 路径点管理
```

### Core Components

#### 1. Spline Algorithm Classes
- 实现CatMull-Rom和Bezier插值算法
- 提供 `GetPoint(float t)` 方法 (t ∈ [0,1])
- 提供 `GetTangent(float t)` 获取切线方向
- 使用 `List<Vector3>` 或 `Vector3[]` 存储控制点
- 所有样条类应继承自 `ISpline` 接口或 `SplineBase` 抽象类

**CatMull-Rom公式参考**:
```csharp
// P(t) = 0.5 * [(2*P1) + (-P0 + P2)*t + (2*P0 - 5*P1 + 4*P2 - P3)*t^2 + (-P0 + 3*P1 - 3*P2 + P3)*t^3]
```

#### 2. Editor Scripts
- 使用 `[CustomEditor]` 特性扩展Inspector
- 在Scene视图使用 `Handles` API绘制控制点和曲线
- 支持通过Handles交互添加/移除/拖动路径点
- 实现 `OnSceneGUI()` 进行可视化编辑
- 使用 `EditorGUILayout` 在Inspector中切换样条类型

#### 3. Runtime Components
- `SplineRenderer`: 使用 `LineRenderer` 或 `Gizmos` 渲染曲线
- `WaypointManager`: MonoBehaviour管理路径点列表
- 支持动态添加/移除路径点并实时更新曲线
- 提供 `GetPositionAtDistance(float distance)` 用于沿曲线移动对象

## Development Conventions

### Coding Standards
- **命名**: 使用PascalCase命名类和公共成员，camelCase命名私有字段
- **序列化**: 使用 `[SerializeField]` 暴露私有字段到Inspector
- **性能**: 缓存插值结果，避免每帧重新计算整条曲线
- **Gizmos**: 使用 `OnDrawGizmos()` 在Scene视图显示调试信息

### Unity-Specific Patterns
```csharp
// ✓ 推荐: 使用SerializeField暴露配置
[SerializeField] private List<Vector3> waypoints = new List<Vector3>();
[SerializeField] private SplineType splineType = SplineType.CatmullRom;
[SerializeField] private int resolution = 20; // 曲线分段数

// ✓ 推荐: 编辑器脚本放在Editor文件夹
#if UNITY_EDITOR
using UnityEditor;
[CustomEditor(typeof(WaypointManager))]
public class WaypointManagerEditor : Editor { }
#endif

// ✓ 推荐: 运行时性能优化
private Vector3[] cachedCurvePoints;
private void UpdateCurveCache() {
    cachedCurvePoints = new Vector3[resolution];
    for (int i = 0; i < resolution; i++) {
        float t = i / (float)(resolution - 1);
        cachedCurvePoints[i] = spline.GetPoint(t);
    }
}
```

### Spline Algorithm Notes
- **CatMull-Rom**: 需要至少4个控制点，曲线通过中间控制点
- **Bezier (Cubic)**: 每段使用4个控制点 (起点、控制点1、控制点2、终点)
- **闭合曲线**: 添加首尾点到控制点列表实现循环
- **采样精度**: 使用自适应采样或固定分段数 (建议20-50段)

## Testing & Debugging
- **Scene测试**: 在 `Assets/Scenes/SampleScene.unity` 中测试样条渲染
- **Editor测试**: 使用Unity Test Framework (`com.unity.test-framework`)
- **可视化调试**: 使用 `Debug.DrawLine()` 或 `Gizmos.DrawLine()` 显示曲线段

## Key Files
- `Packages/manifest.json`: 包依赖管理
- `ProjectSettings/ProjectVersion.txt`: Unity版本锁定
- `Assets/Settings/UniversalRP.asset`: URP渲染设置

## Common Tasks

### 添加新的样条类型
1. 在 `Scripts/Spline/` 创建新的插值算法类
2. 实现 `ISpline` 接口或继承 `SplineBase`
3. 在 `WaypointManager` 中添加切换逻辑
4. 更新Editor脚本支持新类型

### 优化曲线渲染性能
- 使用对象池管理 `LineRenderer` 组件
- 在控制点改变时才重新计算曲线
- 使用 `Mesh` 代替 `LineRenderer` 渲染复杂曲线

## External References
- Unity Splines Package (官方): `com.unity.splines` (可选依赖)
- CatMull-Rom公式: https://en.wikipedia.org/wiki/Centripetal_Catmull%E2%80%93Rom_spline
- Bezier曲线: https://pomax.github.io/bezierinfo/
