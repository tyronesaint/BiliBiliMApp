# BiliBiliMDDylib 主页空白改动说明

## 改动概要

实现哔哩哔哩 App 主页推荐流空白（不显示任何推荐卡片）。

## 改了什么

### 1. 修改的文件

| 文件路径 | 改动内容 |
|---------|---------|
| `BiliBiliMDDylib/Logos/Home/rcmd/Entry/NJRcmdCardHandler.m` | `handleCardData:` 方法第一行改为 `return nil;`，过滤所有推荐卡片 |

**原理**：
- `NJRcmdCardDataItem.m` 在遍历 items 时，会调用 `cardHandler handleCardData:`
- 当 `handleCardData:` 返回 `nil` 时，`NJRcmdCardDataItem.m` 第 58-62 行的 `if (cardData)` 判断会跳过该卡片
- 所有卡片都被过滤后，`newItems` 为空数组，`items` 被清空
- 最终返回的 JSON 中 `data.items` 为空数组，主页显示空白

**不会崩溃**：因为 `NJRcmdCardDataItem.m` 已有 `if (cardData)` 的 nil 保护。

### 2. 新增的文件

| 文件路径 | 说明 |
|---------|------|
| `Makefile` | Theos 编译配置，无需 Xcode 即可编译 dylib |
| `.github/workflows/build-dylib.yml` | GitHub Actions 自动编译工作流 |

## 编译方法

### 本地编译（macOS）

```bash
# 1. 安装 Theos（如果还没装）
sudo mkdir -p /opt
sudo git clone --recursive https://github.com/theos/theos.git /opt/theos

# 2. 进入项目根目录
cd /path/to/BiliBiliMApp

# 3. 编译
export THEOS=/opt/theos
make

# 4. 产物位置
ls .theos/obj/debug/BiliBiliMDDylib.dylib
```

### GitHub Actions 自动编译

推送代码到 `main` 或 `master` 分支，Actions 会自动编译并上传产物。

也可以手动触发：进入仓库 → Actions → Build BiliBiliMDDylib → Run workflow

## 目录结构（改动后）

```
BiliBiliMApp/
├── Makefile                          ← 新增
├── .github/
│   └── workflows/
│       └── build-dylib.yml           ← 新增
├── BiliBiliMDDylib/
│   └── Logos/
│       └── Home/
│           └── rcmd/
│               └── Entry/
│                   └── NJRcmdCardHandler.m   ← 修改（return nil）
│   └── ... (其他文件不变)
```

## 回滚方法

如果需要恢复推荐流显示，只需把 `NJRcmdCardHandler.m` 改回去：

```objc
- (NSDictionary *)handleCardData:(NSMutableDictionary *)cardData {
    // 删除 return nil; 恢复原来的代码
    NSString *cardType = cardData[@"card_type"];
    if (cardType.length == 0) {
        return cardData;
    }
    NJRcmdCardEntry *cardEntry = self.cardEntryDic[cardType];
    if (!cardEntry) {
        return cardData;
    }
    NSDictionary *dataHandled = [cardEntry handleData:cardData];
    return dataHandled;
}
```
