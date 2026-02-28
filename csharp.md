# C# Code Metrics 與 Review Agent 整合

> 更新：2026-02-26

---

## C# 的 Code Metrics 工具層次

```
輕量（免費，內建）
  Visual Studio → Analyze → Calculate Code Metrics
  Roslyn Analyzers → 加進 .csproj，build 時自動跑

中等（免費，需設定）
  SonarQube / SonarCloud → CI pipeline，完整 dashboard

完整（付費）
  NDepend → 最強，82 種 metrics，CQLinq 自訂查詢
  ReSharper / InspectCode → JetBrains，IDE + CI
```

---

## 方式 1：Visual Studio 內建（最快看到結果）

```
Analyze → Calculate Code Metrics for Solution
```

直接產出每個方法的：

| 指標 | 說明 | 警戒值 |
|------|------|--------|
| Cyclomatic Complexity | 決策分支數 | > 10 需注意 |
| Maintainability Index | 0–100 綜合分數 | < 20 危險 |
| Depth of Inheritance | 繼承深度 | > 5 需注意 |
| Class Coupling | 依賴的外部類別數 | > 9 需注意 |
| Lines of Code | 方法行數 | > 50 需注意 |

**Maintainability Index（MI）：**
```
MI = 171 - 5.2×ln(HalsteadVolume) - 0.23×CyclomaticComplexity - 16.2×ln(LOC)

> 85  → 綠，容易維護
20–85 → 黃，中等
< 20  → 紅，難以維護
```

Visual Studio 直接顯示這個數字，不需要手動算。這是 C# 比 TypeScript 多的指標。

---

## 方式 2：Roslyn Analyzers（CI 用，免費）

加進 `.csproj`，build 時自動分析，不需要外部服務：

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.CodeAnalysis.NetAnalyzers" Version="*">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  </PackageReference>

  <!-- SonarSource 的 C# 分析器（免費） -->
  <PackageReference Include="SonarAnalyzer.CSharp" Version="*">
    <PrivateAssets>all</PrivateAssets>
    <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  </PackageReference>
</ItemGroup>
```

設定嚴重度（`.editorconfig`）：

```ini
[*.cs]
# Cyclomatic complexity > 10 → warning（CA1502）
dotnet_diagnostic.CA1502.severity = warning

# Cognitive complexity（SonarAnalyzer S3776）
dotnet_diagnostic.S3776.severity = warning

# Method too long（S138）
dotnet_diagnostic.S138.severity = warning
```

CI 跑法：

```bash
# build 時自動輸出 SARIF 格式
dotnet build --configuration Release \
  /p:RunAnalyzersDuringBuild=true \
  /p:ErrorLog=analysis.sarif
```

---

## 方式 3：SonarCloud（完整 dashboard，免費額度）

```yaml
# .github/workflows/sonar.yml
name: SonarCloud
on: [push, pull_request]
jobs:
  sonar:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with: { fetch-depth: 0 }
      - uses: actions/setup-dotnet@v4
        with: { dotnet-version: '8.x' }
      - run: dotnet tool install --global dotnet-sonarscanner
      - run: |
          dotnet sonarscanner begin \
            /k:"${{ secrets.SONAR_PROJECT_KEY }}" \
            /o:"${{ secrets.SONAR_ORG }}" \
            /d:sonar.token="${{ secrets.SONAR_TOKEN }}" \
            /d:sonar.cs.opencover.reportsPaths="**/coverage.opencover.xml"
      - run: dotnet build --no-incremental
      - run: dotnet test --collect:"XPlat Code Coverage" -- DataCollectionRunSettings.DataCollectors.DataCollector.Configuration.Format=opencover
      - run: dotnet sonarscanner end /d:sonar.token="${{ secrets.SONAR_TOKEN }}"
```

SonarCloud 提供的 C# 指標：
- Cognitive Complexity（比 Cyclomatic 更貼近人類理解難度）
- Duplications
- Security Hotspots（SQL injection、hardcoded secrets 等）
- Technical Debt（估算修復時間）
- Quality Gate（可設定 PR 不通過條件）

---

## 方式 4：NDepend（付費，最強）

NDepend 的核心優勢是 **CQLinq**（用 C# LINQ 語法查詢程式碼）：

```csharp
// 找出 Cyclomatic Complexity > 10 的方法
from m in JustMyCode.Methods
where m.CyclomaticComplexity > 10
orderby m.CyclomaticComplexity descending
select new { m, m.CyclomaticComplexity, m.NbLinesOfCode }

// 找出沒有測試覆蓋但複雜度高的方法（高風險）
from m in JustMyCode.Methods
where m.CyclomaticComplexity > 8
  && m.PercentageCoverage < 50
orderby m.CyclomaticComplexity descending
select new { m, m.CyclomaticComplexity, m.PercentageCoverage }
```

這個查詢語言讓你可以自訂任何 quality gate 條件，是 NDepend 最大的差異化。

---

## 和 Review Agent 的搭配

### 搭配 1：Metrics 作為 LLM 的輸入 Context

在跑 semantic review 之前，先收集 metrics，讓 LLM 有數字依據：

```
PR Diff
  + Metrics Report（CC、MI、LOC per method）
  + Requirements 相關段落
  ↓
LLM Semantic Review

LLM 看到：
"方法 ProcessReport() CC=18，LOC=87，MI=35
 這個方法邏輯複雜，請特別注意邊界條件和錯誤處理"
```

LLM 有了數字，判斷會更有根據，不只是靠語感。

### 搭配 2：Metrics 決定是否觸發深度 Review

不是每個 PR 都需要 LLM review（有成本），用 metrics 當觸發條件：

```yaml
- name: Check Metrics Threshold
  id: check
  run: |
    # 解析 SARIF 或 SonarCloud API
    # 如果有方法 CC > 10 或 MI < 50 → 觸發 LLM review
    python scripts/check_metrics_threshold.py

- name: Semantic LLM Review
  if: steps.check.outputs.needs_review == 'true'
  run: python scripts/semantic_review.py
```

### 搭配 3：Quality Gate 直接 Block PR

SonarCloud / NDepend 的 Quality Gate 是確定性的，不需要 LLM：

```
Quality Gate 條件（範例）：
  新增程式碼 CC > 15 → FAIL（block merge）
  新增程式碼 Coverage < 80% → FAIL
  新增 Critical Security Hotspot → FAIL
  MI < 20 → FAIL
```

這層 gate 通過後，LLM 再做更深的語意分析。

---

## 建議的 C# Review Pipeline

```
PR 建立
  ↓
[Layer 1] dotnet build + dotnet test
  ↓ pass
[Layer 2] Roslyn Analyzers / SonarCloud Quality Gate
  CC > 15 或 MI < 20 → block
  ↓ pass
[Layer 3] Semantic LLM Review
  metrics report + diff + requirements → LLM findings
  ↓
[Layer 4] HITL（高風險：DB schema、auth、security）
```

---

## 工具選擇建議

| 情境 | 推薦 |
|------|------|
| 個人/小專案，快速上手 | Visual Studio 內建 + Roslyn Analyzers |
| 開源或小團隊 | SonarCloud（免費） |
| 企業，需要深度分析 | NDepend（付費） |
| JetBrains 用戶 | ReSharper + InspectCode（CI 用免費版） |

**最小可行設定（從零開始，10 分鐘）：**
1. 加 `SonarAnalyzer.CSharp` 到 `.csproj`
2. 設定 `.editorconfig` 的 severity
3. `dotnet build` 就會看到 warnings

不需要任何外部服務。
