# iOS审核隐私合规指南

## 📋 概述

本文档说明国浩中医App在iOS平台上的隐私合规要求和实施建议，确保顺利通过App Store审核。

---

## ✅ 已完成的工作

### 1. 隐私政策文档
- ✅ 详细的隐私政策（privacy-policy.html）
- ✅ 用户注册协议（user-agreement.html）
- ✅ 完整的第三方SDK列表及隐私政策链接
- ✅ 详细的权限使用说明表格
- ✅ iOS平台特别说明章节

### 2. 权限说明覆盖
已详细说明以下权限的用途：
- 相机权限
- 相册/存储权限
- 位置权限
- 麦克风权限
- 通知权限
- 日历权限
- 网络权限
- 读取手机状态
- 振动权限
- WiFi状态

### 3. 第三方SDK清单
已列出所有SDK及其隐私政策：
- 百度地图SDK
- 微信开放平台SDK
- 支付宝SDK
- 腾讯云IM SDK
- 腾讯云音视频通话SDK (TUICallKit)
- Apple登录SDK

---

## 🔧 需要在App中实施的工作

### 1. 隐私政策展示位置

#### 必须展示的位置：
```
1. App首次启动时
   - 显示隐私政策同意弹窗
   - 包含"同意"和"不同意"按钮
   - 提供隐私政策和用户协议的链接

2. 注册/登录页面
   - 在注册按钮下方显示
   - "注册即表示同意《用户协议》和《隐私政策》"
   - 文字可点击跳转到对应页面

3. 设置页面
   - 在"关于我们"或"设置"中添加
   - "隐私政策"菜单项
   - "用户协议"菜单项

4. App Store描述
   - 在应用描述中添加隐私政策链接
   - 链接：https://你的域名/privacy-policy.html
```

#### 实现示例代码：

**首次启动弹窗（Vue 3）：**
```vue
<template>
  <wd-popup v-model="showPrivacy" :close-on-click-modal="false">
    <view class="privacy-popup">
      <view class="privacy-title">隐私政策与用户协议</view>
      <view class="privacy-content">
        <text>欢迎使用国浩中医！我们非常重视您的隐私保护和个人信息安全。</text>
        <text>在使用我们的服务前，请您仔细阅读并充分理解</text>
        <text class="link" @click="openPrivacy">《隐私政策》</text>
        <text>和</text>
        <text class="link" @click="openAgreement">《用户协议》</text>
        <text>的全部内容。</text>
      </view>
      <view class="privacy-buttons">
        <button @click="disagree">不同意</button>
        <button type="primary" @click="agree">同意并继续</button>
      </view>
    </view>
  </wd-popup>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

const showPrivacy = ref(false)

onMounted(() => {
  // 检查是否已同意隐私政策
  const hasAgreed = uni.getStorageSync('privacy_agreed')
  if (!hasAgreed) {
    showPrivacy.value = true
  }
})

const openPrivacy = () => {
  uni.navigateTo({
    url: '/pages/webview/index?url=https://你的域名/privacy-policy.html'
  })
}

const openAgreement = () => {
  uni.navigateTo({
    url: '/pages/webview/index?url=https://你的域名/user-agreement.html'
  })
}

const agree = () => {
  uni.setStorageSync('privacy_agreed', true)
  uni.setStorageSync('privacy_agreed_time', new Date().toISOString())
  showPrivacy.value = false
}

const disagree = () => {
  uni.showModal({
    title: '提示',
    content: '您需要同意隐私政策才能使用本应用',
    showCancel: false,
    success: () => {
      // 可以选择退出应用或重新显示弹窗
      showPrivacy.value = true
    }
  })
}
</script>
```

### 2. 权限申请时机和说明

#### 权限申请最佳实践：

```typescript
// composables/usePermission.ts
export const usePermission = () => {
  /**
   * 申请相机权限
   */
  const requestCamera = async (purpose: string = '拍摄医疗资料') => {
    // iOS需要在Info.plist中配置说明
    // NSCameraUsageDescription: 需要使用相机拍摄病历、处方等医疗资料
    
    return new Promise((resolve, reject) => {
      uni.authorize({
        scope: 'scope.camera',
        success: () => resolve(true),
        fail: () => {
          uni.showModal({
            title: '需要相机权限',
            content: `为了${purpose}，需要您授权相机权限`,
            confirmText: '去设置',
            success: (res) => {
              if (res.confirm) {
                uni.openSetting()
              }
            }
          })
          reject(false)
        }
      })
    })
  }

  /**
   * 申请相册权限
   */
  const requestAlbum = async (purpose: string = '上传医疗图片') => {
    // NSPhotoLibraryUsageDescription: 需要访问相册以上传病历、检查报告等医疗资料
    // NSPhotoLibraryAddUsageDescription: 需要保存处方单、用药指导等医疗文档到相册
    
    return new Promise((resolve, reject) => {
      uni.authorize({
        scope: 'scope.writePhotosAlbum',
        success: () => resolve(true),
        fail: () => {
          uni.showModal({
            title: '需要相册权限',
            content: `为了${purpose}，需要您授权相册权限`,
            confirmText: '去设置',
            success: (res) => {
              if (res.confirm) {
                uni.openSetting()
              }
            }
          })
          reject(false)
        }
      })
    })
  }

  /**
   * 申请位置权限
   */
  const requestLocation = async (purpose: string = '推荐附近医生') => {
    // NSLocationWhenInUseUsageDescription: 需要获取您的位置以推荐附近的医生和诊所，提供本地化医疗服务
    
    return new Promise((resolve, reject) => {
      uni.authorize({
        scope: 'scope.userLocation',
        success: () => resolve(true),
        fail: () => {
          uni.showModal({
            title: '需要位置权限',
            content: `为了${purpose}，需要您授权位置权限`,
            confirmText: '去设置',
            success: (res) => {
              if (res.confirm) {
                uni.openSetting()
              }
            }
          })
          reject(false)
        }
      })
    })
  }

  /**
   * 申请麦克风权限
   */
  const requestMicrophone = async (purpose: string = '语音问诊') => {
    // NSMicrophoneUsageDescription: 需要使用麦克风进行语音问诊、发送语音消息和音视频通话
    
    return new Promise((resolve, reject) => {
      uni.authorize({
        scope: 'scope.record',
        success: () => resolve(true),
        fail: () => {
          uni.showModal({
            title: '需要麦克风权限',
            content: `为了${purpose}，需要您授权麦克风权限`,
            confirmText: '去设置',
            success: (res) => {
              if (res.confirm) {
                uni.openSetting()
              }
            }
          })
          reject(false)
        }
      })
    })
  }

  return {
    requestCamera,
    requestAlbum,
    requestLocation,
    requestMicrophone
  }
}
```

### 3. Info.plist 权限描述配置

在 `manifest.json` 的 iOS 配置中添加：

```json
{
  "app-plus": {
    "distribute": {
      "ios": {
        "privacyDescription": {
          "NSCameraUsageDescription": "需要使用相机拍摄病历、处方、患处照片等医疗资料，用于医生诊断",
          "NSPhotoLibraryUsageDescription": "需要访问相册以上传病历、检查报告等医疗资料",
          "NSPhotoLibraryAddUsageDescription": "需要保存处方单、用药指导等医疗文档到相册",
          "NSLocationWhenInUseUsageDescription": "需要获取您的位置以推荐附近的医生和诊所，提供本地化医疗服务",
          "NSMicrophoneUsageDescription": "需要使用麦克风进行语音问诊、发送语音消息和音视频通话",
          "NSCalendarsUsageDescription": "需要访问日历以添加预约挂号、复诊时间、用药提醒等日程",
          "NSUserTrackingUsageDescription": "我们不会追踪您的信息用于广告目的"
        }
      }
    }
  }
}
```

### 4. App Store 提交清单

#### App Store Connect 配置：

1. **隐私详情（Privacy Details）**
   - 数据类型：健康与健身、联系信息、标识符、使用数据
   - 数据用途：应用功能、分析、产品个性化
   - 是否关联到用户：是
   - 是否用于追踪：否

2. **应用隐私问题回答**
   ```
   Q: 此App是否收集用户数据？
   A: 是

   Q: 收集的数据是否关联到用户身份？
   A: 是

   Q: 是否用于追踪用户？
   A: 否

   Q: 收集哪些类型的数据？
   A: 
   - 健康与健身（诊疗记录、健康数据）
   - 联系信息（姓名、电话、地址）
   - 标识符（用户ID）
   - 使用数据（产品交互、诊断数据）
   ```

3. **应用描述中添加隐私政策链接**
   ```
   隐私政策：https://你的域名/privacy-policy.html
   用户协议：https://你的域名/user-agreement.html
   ```

### 5. 审核注意事项

#### 常见拒审原因及解决方案：

| 拒审原因 | 解决方案 |
|---------|---------|
| 隐私政策链接无法访问 | 确保链接在App内和浏览器都能正常访问 |
| 权限说明不清晰 | 在Info.plist中添加详细的权限使用说明 |
| 首次启动未显示隐私政策 | 添加首次启动隐私政策同意弹窗 |
| 第三方SDK未披露 | 在隐私政策中列出所有SDK及其用途 |
| 收集数据与隐私标签不符 | 确保隐私标签与实际收集的数据一致 |
| 未成年人保护不足 | 添加年龄验证和家长同意机制 |

#### 测试检查清单：

- [ ] 首次启动显示隐私政策弹窗
- [ ] 注册页面有隐私政策链接
- [ ] 设置中可以访问隐私政策
- [ ] 所有权限申请都有明确说明
- [ ] 隐私政策链接可以正常访问
- [ ] 第三方SDK隐私政策链接有效
- [ ] App Store隐私标签已正确填写
- [ ] Info.plist权限描述已配置
- [ ] 未成年人使用有提示和限制

---

## 📱 App Store 隐私标签配置

### 数据收集类型配置：

#### 1. 健康与健身
- **收集的数据**：健康记录、症状、诊断信息
- **用途**：应用功能
- **是否关联到用户**：是
- **是否用于追踪**：否

#### 2. 联系信息
- **收集的数据**：姓名、电话号码、实际地址、电子邮件地址
- **用途**：应用功能、客户支持
- **是否关联到用户**：是
- **是否用于追踪**：否

#### 3. 用户内容
- **收集的数据**：照片或视频、音频数据
- **用途**：应用功能（医疗诊断）
- **是否关联到用户**：是
- **是否用于追踪**：否

#### 4. 标识符
- **收集的数据**：用户ID、设备ID
- **用途**：应用功能、分析
- **是否关联到用户**：是
- **是否用于追踪**：否

#### 5. 使用数据
- **收集的数据**：产品交互、崩溃数据、性能数据
- **用途**：分析、应用功能
- **是否关联到用户**：是
- **是否用于追踪**：否

#### 6. 诊断
- **收集的数据**：崩溃数据、性能数据
- **用途**：应用功能
- **是否关联到用户**：否
- **是否用于追踪**：否

#### 7. 财务信息
- **收集的数据**：支付信息（通过第三方）
- **用途**：应用功能
- **是否关联到用户**：是
- **是否用于追踪**：否

#### 8. 位置
- **收集的数据**：精确位置
- **用途**：应用功能（推荐附近医生）
- **是否关联到用户**：是
- **是否用于追踪**：否

---

## 🔐 数据安全措施

### 已实施的安全措施：

1. **传输安全**
   - 所有网络请求使用HTTPS
   - 敏感数据传输加密

2. **存储安全**
   - 本地数据加密存储
   - 使用Keychain存储敏感信息

3. **访问控制**
   - 用户身份验证
   - 权限最小化原则

4. **第三方SDK安全**
   - 所有SDK来自可信来源
   - 定期更新SDK版本
   - 审查SDK权限需求

---

## 📞 联系方式

如有任何隐私相关问题，请联系：
- 微信客服：guohaozhongyikefu
- 邮箱：（建议添加官方邮箱）

---

## 📝 更新日志

- 2025-02-10：创建iOS审核隐私合规指南
- 2025-02-10：补充详细权限说明和第三方SDK列表
- 2025-02-10：添加iOS平台特别说明章节

---

## ⚠️ 重要提醒

1. **隐私政策必须在App内可访问**：不能仅在网站上提供
2. **首次启动必须显示隐私政策**：用户同意后才能使用
3. **权限申请必须有明确说明**：在申请时告知用户用途
4. **第三方SDK必须披露**：列出所有SDK及其隐私政策
5. **隐私标签必须准确**：与实际收集的数据保持一致
6. **定期更新隐私政策**：功能变更时及时更新

---

## 🎯 下一步行动

1. [ ] 在App中实现隐私政策弹窗
2. [ ] 配置Info.plist权限描述
3. [ ] 添加隐私政策访问入口
4. [ ] 填写App Store隐私标签
5. [ ] 测试所有权限申请流程
6. [ ] 准备审核说明文档
7. [ ] 提交App Store审核
